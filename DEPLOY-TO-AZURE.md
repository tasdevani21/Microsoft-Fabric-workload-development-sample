# Building the Fabric Workload
## Update Configuration Files
1. In the Backend/src/Packages/manifest folder, update the following fields in `ManifestPackageRelease.nuspec` file.

```xml
  <files>
    <!-- Include the XML and FE files -->
    <file src="WorkloadManifest.xml" target="BE\WorkloadManifest.xml" />
    <file src="Item1.xml" target="BE\Item1.xml" />
    <file src="..\..\..\..\Frontend\Package\*" target="FE" />
    <file src="..\..\..\..\Frontend\Package\assets\**" target="FE\assets" />
  </files> 
```
| File | Description |
| ---- | ----------- |
| `WorkloadManifest.xml` | The XML file that defines the workload. `src` path and file can be updated. However, **the `target` path and file name must remain the same.** |
| `Item1.xml` | The XML file that defines the sample workload. `src` path and file can be updated. However, **the `target` path has to be `BE\<whatever-file-name>`** |
| `..\..\..\..\Frontend\Package\*` | The path to the Frontend package files. This path is relative to the `manifest` folder. |
| `..\..\..\..\Frontend\Package\assets\**` | The path to the Frontend package assets. This path is relative to the `manifest` folder. |

2. In the Backend/src/Packages/manifest folder, update the following fields in `WorkloadManifest.xml` file.

| Field | Description |
| ----- | ----------- |
| `WorkloadName` | The name of the workload. This is the name that will be displayed in Fabric. |
| `WorkloadVersion` | The version of the workload. This is the version that will be displayed in Fabric. |
| `AppId` | The application ID of the App Registration in Entra. |
| `RedirectUri` | The redirect URI of the App Registration in Entra. |
| `ResourceId` | The resource ID of the App Registration in Entra. This is the ID that will be used to authenticate to Fabric. |
| `ServiceEndpoint.Name` | The name of the service endpoint. Create one for the Frontend and one for the Backend. |
| `ServiceEndpoint.Url` | The URL to connect to the Frontend and Backend. This has to be a verified domain in the same tenant. |

## Build the Backend
Open a terminal and navigate to the `Backend` folder and following command to build the Backend.

```bash
dotnet build -c Release
```

## Upload the Nuget Package
Upload the Nuget package to the Fabric tenant. The package is located in the `Backend/src/bin/Release` folder. Example `ManifestPackage.1.0.0.nupkg`.

> You need to have Fabric Admin access to the tenant to upload the package.

## Deploying to Multiple Environments
Since an Azure Tenant can have only one Fabric instance, it can be challenging to deploy the workload to multiple environments. To overcome this, we can use different configuration files. This allows us to create a unique package for each environment.

1. For each environment, you will need to create one of each of the following files and update as required:
   - `Item1.xml`: Example: `SampleWorkload.dev.xml`
   - `ManifestPackage.nuspec`: Example: `ManifestPackage.dev.nuspec`
   - `WorkloadManifest.xml`: Example: `WorkloadManifest.dev.xml`

2. Update the csproj file to include the configuration files during build. This is done by adding the following lines to a `PropertyGroup` section of the csproj file. 

**Example:**
```xml
  <PropertyGroup>
    <BuildConfiguration Condition="'$(BuildConfiguration)'==''">dev</BuildConfiguration>
    <CopyNugetPackage Condition="'$(CopyNugetPackage)'==''">true</CopyNugetPackage> 
    <NuspecFile>Packages\manifest\ManifestPackage.$(BuildConfiguration).nuspec</NuspecFile>
  </PropertyGroup>
```

3. Run the `Build` command with the flag `-p:BuildConfiguration=<environment>`. This is done by adding the following lines to the `Build` command in the pipeline.
```bash
  dotnet build -c Release -p:BuildConfiguration=dev
```
4. Upload the Nuget package to the Fabric tenant.