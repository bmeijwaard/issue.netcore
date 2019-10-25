### Behaviour

We have 3 App Service Plans, each with 1 Web App.
TEST, ACPT and PROD.
All three environments are created using ARM templates. There is no difference between each environment in the template.
The project has 2 builds, one for TEST and one for ACPT/PROD. They each have their own release, so TEST, ACPT and PROD. We uze Azure Devops for our pipeline.

After inspection, all the build and releases are identical.

TEST works perfectly fine. PROD and ACPT will not start.

When doing FTP deploy (using filezilla) from a clean publish to folder from VS2019 we still see this issue.

When copying the working site content from TEST via FTP and uploading it to the PROD using FTP. Still the same issue (altough TEST works).

When creating a new App Service Plan and Web App in Azure and doing a 'import profile', still the same issue.

Deleted and recreated the App Service Plan for PROD usign the ARM template in the build street, no luck.

Hosting the build on a Windows Server VM with ISS, does WORK.

When creating the a clean test project I choose a NetCore2.2 MVC application, and added a Class assembly with NetStandard2. 
Referenced it to the MVC project. FTP published this to the PROD Web App, but this does WORK.
Added all nuget dependencies from the original solution and it still WORKS.

And various other github / stackoverflow / msdn suggestions to similar issues.

Starting the app will return some kind of error catched in stdout:

This is weird, since the folder exists and the files are there. 
```
Error:
  An assembly specified in the application dependencies manifest (MemberPlatform.Api.deps.json) was not found:
    package: 'Microsoft.Win32.SystemEvents', version: '4.5.0'
    path: 'runtimes/win/lib/netcoreapp2.0/Microsoft.Win32.SystemEvents.dll'
```

or when forcing version by copying the correct version, or removing the runtimeTargets (in the deps.json file) for this pacakge the following occurs:

```
Error:
  An assembly specified in the application dependencies manifest (MemberPlatform.Api.deps.json) was not found:
    package: 'System.Diagnostics.PerformanceCounter', version: '4.5.0'
    path: 'runtimes/win/lib/netcoreapp2.0/System.Diagnostics.PerformanceCounter.dll'
```

Sometimes it says (probably meaning the above):

```
Invalid runtimeconfig.json [D:\home\site\wwwroot\MemberPlatform.Api.runtimeconfig.json] [D:\home\site\wwwroot\MemberPlatform.Api.runtimeconfig.dev.json]
```

Toying around with some of the .csproj properties also causes (which is weird because the path does exist:

```
Unhandled Exception: System.IO.FileNotFoundException: Error reading the D:\home\site\wwwroot\ directory.
   at System.IO.FileSystemWatcher.StartRaisingEvents()
   at System.IO.FileSystemWatcher.StartRaisingEventsIfNotDisposed()
   at System.IO.FileSystemWatcher.set_EnableRaisingEvents(Boolean value)
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.TryEnableFileSystemWatcher()
   at Microsoft.Extensions.FileProviders.Physical.PhysicalFilesWatcher.CreateFileChangeToken(String filter)
   at Microsoft.Extensions.FileProviders.PhysicalFileProvider.Watch(String filter)
   at Microsoft.Extensions.Configuration.FileConfigurationProvider.<.ctor>b__0_0()
   at Microsoft.Extensions.Primitives.ChangeToken.OnChange(Func`1 changeTokenProducer, Action changeTokenConsumer)
   at Microsoft.Extensions.Configuration.FileConfigurationProvider..ctor(FileConfigurationSource source)
   at Microsoft.Extensions.Configuration.Json.JsonConfigurationSource.Build(IConfigurationBuilder builder)
   at Microsoft.Extensions.Configuration.ConfigurationBuilder.Build()
   at Microsoft.AspNetCore.Hosting.WebHostBuilder.BuildCommonServices(AggregateException& hostingStartupErrors)
   at Microsoft.AspNetCore.Hosting.WebHostBuilder.Build()
   at MemberPlatform.Api.Program.Main(String[] args) in C:\Projects\fifpro-mp\src\backend\MemberPlatform\ui\MemberPlatform.Api\Program.cs:line 17
   at MemberPlatform.Api.Program.<Main>(String[] args)
```

The options I tried:
`<AspNetCoreModuleName>AspNetCoreModule</AspNetCoreModuleName>`
`<AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>`

or

`<AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>`

or 

`<ResolveNuGetPackages>true</ResolveNuGetPackages>`
`<RestoreProjectStyle>PackageReference</RestoreProjectStyle>`

or

`<RuntimeFrameworkVersion>2.2.0</RuntimeFrameworkVersion>`

`<RuntimeFrameworkVersion>2.2.6</RuntimeFrameworkVersion>`

`<RuntimeFrameworkVersion>2.2.7</RuntimeFrameworkVersion>`

------------------------

Web app .csproj:

```
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.2</TargetFramework>
    <LangVersion>7.2</LangVersion>
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
    <DocumentationFile>bin\Debug\netcoreapp2.2\api.xml</DocumentationFile>
    <WarningsAsErrors>NU1605</WarningsAsErrors>
    <NoWarn>1701;1702;1591</NoWarn>
    <WarningLevel>3</WarningLevel>
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|AnyCPU'">
    <DocumentationFile>bin\Release\netcoreapp2.2\api.xml</DocumentationFile>
    <WarningsAsErrors>NU1605</WarningsAsErrors>
    <NoWarn>1701;1702;1591</NoWarn>
    <WarningLevel>3</WarningLevel>
  </PropertyGroup>
 

  <ItemGroup>
    <PackageReference Include="AutoMapper" Version="9.0.0" />
    <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.7.1" />
    <PackageReference Include="Microsoft.AspNetCore.App" />
    <PackageReference Include="Microsoft.AspNetCore.Razor.Design" Version="2.2.0" />
    <PackageReference Include="Microsoft.Azure.KeyVault" Version="3.0.4" />
    <PackageReference Include="Microsoft.Extensions.Configuration.AzureKeyVault" Version="2.2.0" />
    <PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="2.2.0" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.2.3" />
    <PackageReference Include="Microsoft.Win32.SystemEvents" Version="4.6.0" />
    <PackageReference Include="Nager.Country" Version="1.0.4" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="4.0.1" />
    <PackageReference Include="Swashbuckle.AspNetCore.Swagger" Version="4.0.1" />
  </ItemGroup>
 

  <ItemGroup>
    <ProjectReference Include="..\..\common\MemberPlatform.I18n\MemberPlatform.I18n.csproj" />
    <ProjectReference Include="..\..\core\MemberPlatform.Core.Contracts\MemberPlatform.Core.Contracts.csproj" />
    <ProjectReference Include="..\..\core\MemberPlatform.Mapping\MemberPlatform.Mapping.csproj" />
  </ItemGroup>

  <ItemGroup>
    <Content Update="appsettings.json">
      <CopyToOutputDirectory>Never</CopyToOutputDirectory>
    </Content>
    <Content Update="web.config">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>

  <ItemGroup>
    <Folder Include="wwwroot\static" />
  </ItemGroup>

</Project>

```

Class assembly dependencies:

```
<PackageReference Include="AutoMapper" Version="9.0.0" />
<PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="7.0.0" />
<PackageReference Include="Dapper" Version="2.0.4" />
<PackageReference Include="DocumentFormat.OpenXml" Version="2.9.1" />
<PackageReference Include="EFCore.BulkExtensions" Version="2.6.0" />
<PackageReference Include="Elastique.DapperProvider" Version="1.0.2" />
<PackageReference Include="Elastique.StatusLibrary" Version="1.0.1" />
<PackageReference Include="HtmlSanitizer" Version="4.0.217" />
<PackageReference Include="LinqKit.Microsoft.EntityFrameworkCore" Version="1.1.16" />
<PackageReference Include="MailKit" Version="2.3.0" />
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="2.2.0" />
<PackageReference Include="Microsoft.AspNetCore.Http.Abstractions" Version="2.2.0" />
<PackageReference Include="Microsoft.AspNetCore.Http.Features" Version="2.2.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity" Version="2.2.0" />
<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="2.2.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="2.2.6" />
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="2.2.6" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Relational" Version="2.2.6" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="2.2.6" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.2.6" />
<PackageReference Include="Microsoft.Extensions.Configuration" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Configuration.Binder" Version="2.2.4" />
<PackageReference Include="Microsoft.Extensions.Configuration.EnvironmentVariables" Version="2.2.4" />
<PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.DependencyInjection.Abstractions" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Http" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Identity.Core" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Identity.Stores" Version="2.2.0" />
<PackageReference Include="Microsoft.Extensions.Options" Version="2.2.0" />
<PackageReference Include="Nager.Country" Version="1.0.4" />
<PackageReference Include="Newtonsoft.Json" Version="12.0.2" />
<PackageReference Include="protobuf-net" Version="2.4.0" />
<PackageReference Include="SharpZipLib" Version="1.2.0" />
<PackageReference Include="StackExchange.Redis" Version="2.0.601" />
<PackageReference Include="System.Drawing.Common" Version="4.5.1" />
<PackageReference Include="System.Drawing.Primitives" Version="4.3.0" />
<PackageReference Include="System.ServiceModel.Primitives" Version="4.6.0" />
<PackageReference Include="WindowsAzure.Storage" Version="9.3.3" />
<PackageReference Include="Z.EntityFramework.Plus.EFCore" Version="2.0.7" />
<PackageReference Include="ZXing.Net" Version="0.16.4" />
```


----------------
.deps.json content: 
```
{
  "runtimeTarget": {
    "name": ".NETCoreApp,Version=v2.2",
    "signature": "742be9589cf7bfa6008b3d134323892d120682ca"
  },
  "compilationOptions": {
    "defines": [
      "TRACE",
      "RELEASE",
      "NETCOREAPP",
      "NETCOREAPP2_2"
    ],
    "languageVersion": "7.2",
    "platform": "",
    "allowUnsafe": false,
    "warningsAsErrors": false,
    "optimize": true,
    "keyFile": "",
    "emitEntryPoint": true,
    "xmlDoc": true,
    "debugType": "portable"
  },
  "targets": {
    ".NETCoreApp,Version=v2.2": {
      "MemberPlatform.Api/1.0.0": {
        "dependencies": {
          "AutoMapper": "9.0.0",
          "MemberPlatform.Core.Contracts": "1.0.0",
          "MemberPlatform.I18n": "1.0.0",
          "MemberPlatform.Mapping": "1.0.0",
          "Microsoft.ApplicationInsights.AspNetCore": "2.7.1",
          "Microsoft.AspNetCore.App": "2.2.0",
          "Microsoft.AspNetCore.Razor.Design": "2.2.0",
          "Microsoft.Azure.KeyVault": "3.0.4",
          "Microsoft.Extensions.Configuration.AzureKeyVault": "2.2.0",
          "Microsoft.Extensions.Configuration.UserSecrets": "2.2.0",
          "Microsoft.NETCore.App": "2.2.0",
          "Microsoft.VisualStudio.Web.CodeGeneration.Design": "2.2.3",
          "Nager.Country": "1.0.4",
          "Swashbuckle.AspNetCore": "4.0.1",
          "Swashbuckle.AspNetCore.Swagger": "4.0.1"
        },
        "runtime": {
          "MemberPlatform.Api.dll": {}
        },
        "compile": {
          "MemberPlatform.Api.dll": {}
        }
      },
      "AngleSharp/0.9.11": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Dynamic.Runtime": "4.3.0",
          "System.IO": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Net.Requests": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Text.Encoding.Extensions": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/AngleSharp.dll": {
            "assemblyVersion": "0.9.9.0",
            "fileVersion": "0.9.9.0"
          }
        },
        "compile": {
          "lib/netstandard1.0/AngleSharp.dll": {}
        }
      },
      "AutoMapper/9.0.0": {
        "dependencies": {
          "Microsoft.CSharp": "4.5.0",
          "System.Reflection.Emit": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/AutoMapper.dll": {
            "assemblyVersion": "9.0.0.0",
            "fileVersion": "9.0.0.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/AutoMapper.dll": {}
        }
      },
      "AutoMapper.Extensions.Microsoft.DependencyInjection/7.0.0": {
        "dependencies": {
          "AutoMapper": "9.0.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/AutoMapper.Extensions.Microsoft.DependencyInjection.dll": {
            "assemblyVersion": "0.0.0.0",
            "fileVersion": "7.0.0.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/AutoMapper.Extensions.Microsoft.DependencyInjection.dll": {}
        }
      },
      "Dapper/2.0.4": {
        "dependencies": {
          "System.Reflection.Emit.Lightweight": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/Dapper.dll": {
            "assemblyVersion": "2.0.0.0",
            "fileVersion": "2.0.4.31833"
          }
        },
        "compile": {
          "lib/netstandard2.0/Dapper.dll": {}
        }
      },
      "DocumentFormat.OpenXml/2.9.1": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "System.IO.Packaging": "4.5.0",
          "System.Runtime.Serialization.Xml": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.3/DocumentFormat.OpenXml.dll": {
            "assemblyVersion": "2.9.1.0",
            "fileVersion": "2.9.1.2"
          }
        },
        "compile": {
          "lib/netstandard1.3/DocumentFormat.OpenXml.dll": {}
        }
      },
      "EFCore.BulkExtensions/2.6.0": {
        "dependencies": {
          "FastMember": "1.5.0",
          "Microsoft.EntityFrameworkCore": "2.2.6",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6",
          "Microsoft.EntityFrameworkCore.Sqlite": "2.2.6",
          "System.Data.SqlClient": "4.6.1"
        },
        "runtime": {
          "lib/netstandard2.0/EFCore.BulkExtensions.dll": {
            "assemblyVersion": "2.6.0.0",
            "fileVersion": "2.6.0.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/EFCore.BulkExtensions.dll": {}
        }
      },
      "Elastique.DapperProvider/1.0.2": {
        "dependencies": {
          "Dapper": "2.0.4"
        },
        "runtime": {
          "lib/netstandard2.0/Elastique.DapperProvider.dll": {
            "assemblyVersion": "1.0.2.0",
            "fileVersion": "1.0.2.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Elastique.DapperProvider.dll": {}
        }
      },
      "Elastique.StatusLibrary/1.0.1": {
        "runtime": {
          "lib/netstandard2.0/Elastique.StatusLibrary.dll": {
            "assemblyVersion": "1.0.1.0",
            "fileVersion": "1.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Elastique.StatusLibrary.dll": {}
        }
      },
      "FastMember/1.5.0": {
        "runtime": {
          "lib/netcoreapp2.0/FastMember.dll": {
            "assemblyVersion": "1.5.0.0",
            "fileVersion": "1.5.0.0"
          }
        },
        "compile": {
          "lib/netcoreapp2.0/FastMember.dll": {}
        }
      },
      "HtmlSanitizer/4.0.217": {
        "dependencies": {
          "AngleSharp": "0.9.11"
        },
        "runtime": {
          "lib/netstandard2.0/HtmlSanitizer.dll": {
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.0.217.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/HtmlSanitizer.dll": {}
        }
      },
      "LinqKit.Microsoft.EntityFrameworkCore/1.1.16": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore": "2.2.6"
        },
        "runtime": {
          "lib/netstandard2.0/LinqKit.Microsoft.EntityFrameworkCore.dll": {
            "assemblyVersion": "1.1.16.0",
            "fileVersion": "1.1.16.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/LinqKit.Microsoft.EntityFrameworkCore.dll": {}
        }
      },
      "MailKit/2.3.0": {
        "dependencies": {
          "MimeKit": "2.3.0",
          "System.Net.NameResolution": "4.3.0",
          "System.Net.Security": "4.3.2",
          "System.Runtime.Serialization.Primitives": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/MailKit.dll": {
            "assemblyVersion": "2.3.0.0",
            "fileVersion": "2.3.0.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/MailKit.dll": {}
        }
      },
      "Microsoft.ApplicationInsights/2.10.0": {
        "dependencies": {
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Runtime.InteropServices": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.ApplicationInsights.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.31626"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.ApplicationInsights.dll": {}
        }
      },
      "Microsoft.ApplicationInsights.AspNetCore/2.7.1": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "Microsoft.ApplicationInsights.DependencyCollector": "2.10.0",
          "Microsoft.ApplicationInsights.PerfCounterCollector": "2.10.0",
          "Microsoft.ApplicationInsights.WindowsServer": "2.10.0",
          "Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel": "2.10.0",
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.Extensions.Configuration.Json": "2.2.0",
          "Microsoft.Extensions.Logging.ApplicationInsights": "2.10.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Net.NameResolution": "4.3.0",
          "System.Text.Encodings.Web": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.ApplicationInsights.AspNetCore.dll": {
            "assemblyVersion": "2.7.1.0",
            "fileVersion": "2.7.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.ApplicationInsights.AspNetCore.dll": {}
        }
      },
      "Microsoft.ApplicationInsights.DependencyCollector/2.10.0": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "Microsoft.Extensions.DiagnosticAdapter": "2.2.0",
          "Microsoft.Extensions.PlatformAbstractions": "1.1.0",
          "NETStandard.Library": "2.0.3",
          "System.Data.SqlClient": "4.6.1",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Diagnostics.StackTrace": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.6/Microsoft.AI.DependencyCollector.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.32157"
          }
        },
        "compile": {
          "lib/netstandard1.6/Microsoft.AI.DependencyCollector.dll": {}
        }
      },
      "Microsoft.ApplicationInsights.PerfCounterCollector/2.10.0": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.PlatformAbstractions": "1.1.0",
          "System.Diagnostics.PerformanceCounter": "4.5.0",
          "System.Runtime.Serialization.Json": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0",
          "System.Threading.Thread": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.AI.PerfCounterCollector.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.32157"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AI.PerfCounterCollector.dll": {}
        }
      },
      "Microsoft.ApplicationInsights.WindowsServer/2.10.0": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "Microsoft.ApplicationInsights.DependencyCollector": "2.10.0",
          "Microsoft.ApplicationInsights.PerfCounterCollector": "2.10.0",
          "Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel": "2.10.0",
          "NETStandard.Library": "2.0.3",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Runtime.Serialization.Json": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.6/Microsoft.AI.WindowsServer.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.32157"
          }
        },
        "compile": {
          "lib/netstandard1.6/Microsoft.AI.WindowsServer.dll": {}
        }
      },
      "Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/2.10.0": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "System.Diagnostics.Process": "4.3.0",
          "System.IO.FileSystem.AccessControl": "4.3.0",
          "System.Net.NetworkInformation": "4.3.0",
          "System.Net.Requests": "4.3.0",
          "System.Net.WebHeaderCollection": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Threading.Thread": "4.3.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.AI.ServerTelemetryChannel.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.31626"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AI.ServerTelemetryChannel.dll": {}
        }
      },
      "Microsoft.Azure.KeyVault/3.0.4": {
        "dependencies": {
          "Microsoft.Azure.KeyVault.WebKey": "3.0.4",
          "Microsoft.Rest.ClientRuntime": "2.3.20",
          "Microsoft.Rest.ClientRuntime.Azure": "3.3.18",
          "Newtonsoft.Json": "12.0.2",
          "System.Net.Http": "4.3.4"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Azure.KeyVault.dll": {
            "assemblyVersion": "3.0.4.0",
            "fileVersion": "3.0.419.36903"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Azure.KeyVault.dll": {}
        }
      },
      "Microsoft.Azure.KeyVault.WebKey/3.0.4": {
        "dependencies": {
          "Microsoft.Rest.ClientRuntime": "2.3.20",
          "Microsoft.Rest.ClientRuntime.Azure": "3.3.18",
          "Newtonsoft.Json": "12.0.2",
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Net.Http": "4.3.4",
          "System.Runtime": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Cng": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Azure.KeyVault.WebKey.dll": {
            "assemblyVersion": "3.0.4.0",
            "fileVersion": "3.0.419.36903"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Azure.KeyVault.WebKey.dll": {}
        }
      },
      "Microsoft.Azure.Services.AppAuthentication/1.0.1": {
        "dependencies": {
          "Microsoft.IdentityModel.Clients.ActiveDirectory": "3.14.2",
          "NETStandard.Library": "2.0.3",
          "System.Diagnostics.Process": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.4/Microsoft.Azure.Services.AppAuthentication.dll": {
            "assemblyVersion": "1.0.1.0",
            "fileVersion": "1.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard1.4/Microsoft.Azure.Services.AppAuthentication.dll": {}
        }
      },
      "Microsoft.CodeAnalysis.CSharp.Workspaces/2.8.0": {
        "dependencies": {
          "Microsoft.CodeAnalysis.CSharp": "2.8.0",
          "Microsoft.CodeAnalysis.Workspaces.Common": "2.8.0"
        },
        "runtime": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.CSharp.Workspaces.dll": {
            "assemblyVersion": "2.8.0.0",
            "fileVersion": "2.8.0.62830"
          }
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.CSharp.Workspaces.dll": {}
        }
      },
      "Microsoft.CodeAnalysis.Workspaces.Common/2.8.0": {
        "dependencies": {
          "Microsoft.CodeAnalysis.Common": "2.8.0",
          "System.Composition": "1.0.31",
          "System.Diagnostics.Contracts": "4.3.0",
          "System.Linq.Parallel": "4.3.0",
          "System.ObjectModel": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading.Tasks.Parallel": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.Workspaces.dll": {
            "assemblyVersion": "2.8.0.0",
            "fileVersion": "2.8.0.62830"
          }
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.Workspaces.dll": {}
        }
      },
      "Microsoft.Data.Sqlite.Core/2.2.6": {
        "dependencies": {
          "SQLitePCLRaw.core": "1.1.12"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Data.Sqlite.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Data.Sqlite.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore/2.2.6": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore.Abstractions": "2.2.6",
          "Microsoft.EntityFrameworkCore.Analyzers": "2.2.6",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Remotion.Linq": "2.2.0",
          "System.Collections.Immutable": "1.5.0",
          "System.ComponentModel.Annotations": "4.5.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Interactive.Async": "3.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore.Abstractions/2.2.6": {
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Abstractions.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Abstractions.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore.Analyzers/2.2.6": {},
      "Microsoft.EntityFrameworkCore.InMemory/2.2.6": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore": "2.2.6"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.InMemory.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.InMemory.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore.Relational/2.2.6": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore": "2.2.6"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Relational.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Relational.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore.Sqlite/2.2.6": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore.Sqlite.Core": "2.2.6",
          "SQLitePCLRaw.bundle_green": "1.1.12"
        }
      },
      "Microsoft.EntityFrameworkCore.Sqlite.Core/2.2.6": {
        "dependencies": {
          "Microsoft.Data.Sqlite.Core": "2.2.6",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6",
          "Microsoft.Extensions.DependencyModel": "2.1.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Sqlite.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Sqlite.dll": {}
        }
      },
      "Microsoft.EntityFrameworkCore.SqlServer/2.2.6": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6",
          "System.Data.SqlClient": "4.6.1"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.SqlServer.dll": {
            "assemblyVersion": "2.2.6.0",
            "fileVersion": "2.2.6.19169"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.SqlServer.dll": {}
        }
      },
      "Microsoft.Extensions.Configuration.AzureKeyVault/2.2.0": {
        "dependencies": {
          "Microsoft.Azure.KeyVault": "3.0.4",
          "Microsoft.Azure.Services.AppAuthentication": "1.0.1",
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.AzureKeyVault.dll": {
            "assemblyVersion": "2.2.0.0",
            "fileVersion": "2.2.0.18315"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.AzureKeyVault.dll": {}
        }
      },
      "Microsoft.Extensions.Configuration.Binder/2.2.4": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Binder.dll": {
            "assemblyVersion": "2.2.0.0",
            "fileVersion": "2.2.0.19081"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Binder.dll": {}
        }
      },
      "Microsoft.Extensions.Configuration.EnvironmentVariables/2.2.4": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.EnvironmentVariables.dll": {
            "assemblyVersion": "2.2.0.0",
            "fileVersion": "2.2.0.19081"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.EnvironmentVariables.dll": {}
        }
      },
      "Microsoft.Extensions.Logging.ApplicationInsights/2.10.0": {
        "dependencies": {
          "Microsoft.ApplicationInsights": "2.10.0",
          "Microsoft.Extensions.Logging": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.ApplicationInsights.dll": {
            "assemblyVersion": "2.10.0.0",
            "fileVersion": "2.10.0.41092"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.ApplicationInsights.dll": {}
        }
      },
      "Microsoft.Extensions.PlatformAbstractions/1.1.0": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "System.Reflection.TypeExtensions": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.3/Microsoft.Extensions.PlatformAbstractions.dll": {
            "assemblyVersion": "1.1.0.0",
            "fileVersion": "1.1.0.21115"
          }
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.Extensions.PlatformAbstractions.dll": {}
        }
      },
      "Microsoft.IdentityModel.Clients.ActiveDirectory/3.14.2": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "System.Runtime.Serialization.Json": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.3/Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll": {
            "assemblyVersion": "3.14.2.11",
            "fileVersion": "3.14.40721.918"
          },
          "lib/netstandard1.3/Microsoft.IdentityModel.Clients.ActiveDirectory.dll": {
            "assemblyVersion": "3.14.2.11",
            "fileVersion": "3.14.40721.918"
          }
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll": {},
          "lib/netstandard1.3/Microsoft.IdentityModel.Clients.ActiveDirectory.dll": {}
        }
      },
      "Microsoft.Rest.ClientRuntime/2.3.20": {
        "dependencies": {
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Rest.ClientRuntime.dll": {
            "assemblyVersion": "2.0.0.0",
            "fileVersion": "2.3.20.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Rest.ClientRuntime.dll": {}
        }
      },
      "Microsoft.Rest.ClientRuntime.Azure/3.3.18": {
        "dependencies": {
          "Microsoft.Rest.ClientRuntime": "2.3.20",
          "NETStandard.Library": "2.0.3",
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard1.4/Microsoft.Rest.ClientRuntime.Azure.dll": {
            "assemblyVersion": "3.0.0.0",
            "fileVersion": "3.3.18.0"
          }
        },
        "compile": {
          "lib/netstandard1.4/Microsoft.Rest.ClientRuntime.Azure.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration/2.2.3": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.VisualStudio.Web.CodeGeneration.EntityFrameworkCore": "2.2.3"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.Contracts/2.2.3": {
        "dependencies": {
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Contracts.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Contracts.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.Core/2.2.3": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.VisualStudio.Web.CodeGeneration.Templating": "2.2.3",
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Core.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Core.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.Design/2.2.3": {
        "dependencies": {
          "Microsoft.VisualStudio.Web.CodeGenerators.Mvc": "2.2.3"
        },
        "runtime": {
          "lib/netstandard2.0/dotnet-aspnet-codegenerator-design.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/dotnet-aspnet-codegenerator-design.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.EntityFrameworkCore/2.2.3": {
        "dependencies": {
          "Microsoft.VisualStudio.Web.CodeGeneration.Core": "2.2.3"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.EntityFrameworkCore.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.EntityFrameworkCore.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.Templating/2.2.3": {
        "dependencies": {
          "Microsoft.AspNetCore.Razor.Language": "2.2.0",
          "Microsoft.CodeAnalysis.CSharp": "2.8.0",
          "Microsoft.VisualStudio.Web.CodeGeneration.Utils": "2.2.3"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Templating.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Templating.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGeneration.Utils/2.2.3": {
        "dependencies": {
          "Microsoft.CodeAnalysis.CSharp.Workspaces": "2.8.0",
          "Microsoft.VisualStudio.Web.CodeGeneration.Contracts": "2.2.3",
          "Newtonsoft.Json": "12.0.2",
          "NuGet.Frameworks": "4.7.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Utils.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGeneration.Utils.dll": {}
        }
      },
      "Microsoft.VisualStudio.Web.CodeGenerators.Mvc/2.2.3": {
        "dependencies": {
          "Microsoft.VisualStudio.Web.CodeGeneration": "2.2.3"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGenerators.Mvc.dll": {
            "assemblyVersion": "2.2.3.0",
            "fileVersion": "2.2.3.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.VisualStudio.Web.CodeGenerators.Mvc.dll": {}
        }
      },
      "Microsoft.Win32.Primitives/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        }
      },
      "Microsoft.Win32.SystemEvents/4.5.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Microsoft.Win32.SystemEvents.dll": {
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "runtimeTargets": {
          "runtimes/win/lib/netcoreapp2.0/Microsoft.Win32.SystemEvents.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.6.26515.6"
          }
        }
      },
      "MimeKit/2.3.0": {
        "dependencies": {
          "Portable.BouncyCastle": "1.8.5",
          "System.Data.Common": "4.3.0",
          "System.Globalization.Extensions": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Text.Encoding.CodePages": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/MimeKit.dll": {
            "assemblyVersion": "2.3.0.0",
            "fileVersion": "2.3.0.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/MimeKit.dll": {}
        }
      },
      "Nager.Country/1.0.4": {
        "runtime": {
          "lib/netstandard2.0/Nager.Country.dll": {
            "assemblyVersion": "1.0.4.0",
            "fileVersion": "1.0.4.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Nager.Country.dll": {}
        }
      },
      "Newtonsoft.Json/12.0.2": {
        "runtime": {
          "lib/netstandard2.0/Newtonsoft.Json.dll": {
            "assemblyVersion": "12.0.0.0",
            "fileVersion": "12.0.2.23222"
          }
        },
        "compile": {
          "lib/netstandard2.0/Newtonsoft.Json.dll": {}
        }
      },
      "NuGet.Frameworks/4.7.0": {
        "dependencies": {
          "NETStandard.Library": "2.0.3"
        },
        "runtime": {
          "lib/netstandard1.6/NuGet.Frameworks.dll": {
            "assemblyVersion": "4.7.0.5",
            "fileVersion": "4.7.0.5148"
          }
        },
        "compile": {
          "lib/netstandard1.6/NuGet.Frameworks.dll": {}
        }
      },
      "Pipelines.Sockets.Unofficial/2.0.22": {
        "dependencies": {
          "System.Buffers": "4.5.0",
          "System.IO.Pipelines": "4.5.2",
          "System.Runtime.CompilerServices.Unsafe": "4.5.2"
        },
        "runtime": {
          "lib/netcoreapp2.2/Pipelines.Sockets.Unofficial.dll": {
            "assemblyVersion": "1.0.0.0",
            "fileVersion": "2.0.22.26858"
          }
        },
        "compile": {
          "lib/netcoreapp2.2/Pipelines.Sockets.Unofficial.dll": {}
        }
      },
      "Portable.BouncyCastle/1.8.5": {
        "runtime": {
          "lib/netstandard2.0/BouncyCastle.Crypto.dll": {
            "assemblyVersion": "1.8.5.0",
            "fileVersion": "1.8.5.50"
          }
        },
        "compile": {
          "lib/netstandard2.0/BouncyCastle.Crypto.dll": {}
        }
      },
      "protobuf-net/2.4.0": {
        "dependencies": {
          "System.ServiceModel.Primitives": "4.6.0"
        },
        "runtime": {
          "lib/netcoreapp2.1/protobuf-net.dll": {
            "assemblyVersion": "2.4.0.0",
            "fileVersion": "2.4.0.8641"
          }
        },
        "compile": {
          "lib/netcoreapp2.1/protobuf-net.dll": {}
        }
      },
      "Remotion.Linq/2.2.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.Linq.Queryable": "4.0.1",
          "System.ObjectModel": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/Remotion.Linq.dll": {
            "assemblyVersion": "2.2.0.0",
            "fileVersion": "2.2.0.30000"
          }
        },
        "compile": {
          "lib/netstandard1.0/Remotion.Linq.dll": {}
        }
      },
      "runtime.debian.8-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/debian.8-x64/native/_._": {
            "rid": "debian.8-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.fedora.23-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/fedora.23-x64/native/_._": {
            "rid": "fedora.23-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.fedora.24-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/fedora.24-x64/native/_._": {
            "rid": "fedora.24-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.native.System.Data.SqlClient.sni/4.5.0": {
        "dependencies": {
          "runtime.win-arm64.runtime.native.System.Data.SqlClient.sni": "4.4.0",
          "runtime.win-x64.runtime.native.System.Data.SqlClient.sni": "4.4.0",
          "runtime.win-x86.runtime.native.System.Data.SqlClient.sni": "4.4.0"
        }
      },
      "runtime.native.System.Net.Security/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0"
        }
      },
      "runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "dependencies": {
          "runtime.debian.8-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.fedora.23-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.fedora.24-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.opensuse.13.2-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.opensuse.42.1-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.rhel.7-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.ubuntu.14.04-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.ubuntu.16.04-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2",
          "runtime.ubuntu.16.10-x64.runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        }
      },
      "runtime.opensuse.13.2-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/opensuse.13.2-x64/native/_._": {
            "rid": "opensuse.13.2-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.opensuse.42.1-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/opensuse.42.1-x64/native/_._": {
            "rid": "opensuse.42.1-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/osx.10.10-x64/native/_._": {
            "rid": "osx.10.10-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.rhel.7-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/rhel.7-x64/native/_._": {
            "rid": "rhel.7-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.ubuntu.14.04-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/ubuntu.14.04-x64/native/_._": {
            "rid": "ubuntu.14.04-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.ubuntu.16.04-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/ubuntu.16.04-x64/native/_._": {
            "rid": "ubuntu.16.04-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.ubuntu.16.10-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
        "runtimeTargets": {
          "runtime/ubuntu.16.10-x64/native/_._": {
            "rid": "ubuntu.16.10-x64",
            "assetType": "native"
          }
        }
      },
      "runtime.win-arm64.runtime.native.System.Data.SqlClient.sni/4.4.0": {
        "runtimeTargets": {
          "runtimes/win-arm64/native/sni.dll": {
            "rid": "win-arm64",
            "assetType": "native",
            "fileVersion": "4.6.25512.1"
          }
        }
      },
      "runtime.win-x64.runtime.native.System.Data.SqlClient.sni/4.4.0": {
        "runtimeTargets": {
          "runtimes/win-x64/native/sni.dll": {
            "rid": "win-x64",
            "assetType": "native",
            "fileVersion": "4.6.25512.1"
          }
        }
      },
      "runtime.win-x86.runtime.native.System.Data.SqlClient.sni/4.4.0": {
        "runtimeTargets": {
          "runtimes/win-x86/native/sni.dll": {
            "rid": "win-x86",
            "assetType": "native",
            "fileVersion": "4.6.25512.1"
          }
        }
      },
      "SharpZipLib/1.2.0": {
        "runtime": {
          "lib/netstandard2.0/ICSharpCode.SharpZipLib.dll": {
            "assemblyVersion": "1.2.0.246",
            "fileVersion": "1.2.0.246"
          }
        },
        "compile": {
          "lib/netstandard2.0/ICSharpCode.SharpZipLib.dll": {}
        }
      },
      "SQLitePCLRaw.bundle_green/1.1.12": {
        "dependencies": {
          "SQLitePCLRaw.core": "1.1.12",
          "SQLitePCLRaw.lib.e_sqlite3.linux": "1.1.12",
          "SQLitePCLRaw.lib.e_sqlite3.osx": "1.1.12",
          "SQLitePCLRaw.lib.e_sqlite3.v110_xp": "1.1.12",
          "SQLitePCLRaw.provider.e_sqlite3.netstandard11": "1.1.12"
        },
        "runtime": {
          "lib/netcoreapp/SQLitePCLRaw.batteries_green.dll": {
            "assemblyVersion": "1.1.12.351",
            "fileVersion": "1.0.0.0"
          },
          "lib/netcoreapp/SQLitePCLRaw.batteries_v2.dll": {
            "assemblyVersion": "1.1.12.351",
            "fileVersion": "1.0.0.0"
          }
        },
        "compile": {
          "lib/netcoreapp/SQLitePCLRaw.batteries_green.dll": {},
          "lib/netcoreapp/SQLitePCLRaw.batteries_v2.dll": {}
        }
      },
      "SQLitePCLRaw.core/1.1.12": {
        "dependencies": {
          "NETStandard.Library": "2.0.3"
        },
        "runtime": {
          "lib/netstandard1.1/SQLitePCLRaw.core.dll": {
            "assemblyVersion": "1.1.12.351",
            "fileVersion": "1.0.0.0"
          }
        },
        "compile": {
          "lib/netstandard1.1/SQLitePCLRaw.core.dll": {}
        }
      },
      "SQLitePCLRaw.lib.e_sqlite3.linux/1.1.12": {
        "runtimeTargets": {
          "runtimes/alpine-x64/native/libe_sqlite3.so": {
            "rid": "alpine-x64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-arm/native/libe_sqlite3.so": {
            "rid": "linux-arm",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-arm64/native/libe_sqlite3.so": {
            "rid": "linux-arm64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-armel/native/libe_sqlite3.so": {
            "rid": "linux-armel",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-musl-x64/native/libe_sqlite3.so": {
            "rid": "linux-musl-x64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-x64/native/libe_sqlite3.so": {
            "rid": "linux-x64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/linux-x86/native/libe_sqlite3.so": {
            "rid": "linux-x86",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          }
        }
      },
      "SQLitePCLRaw.lib.e_sqlite3.osx/1.1.12": {
        "runtimeTargets": {
          "runtimes/osx-x64/native/libe_sqlite3.dylib": {
            "rid": "osx-x64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          }
        }
      },
      "SQLitePCLRaw.lib.e_sqlite3.v110_xp/1.1.12": {
        "runtimeTargets": {
          "runtimes/win-x64/native/e_sqlite3.dll": {
            "rid": "win-x64",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/win-x86/native/e_sqlite3.dll": {
            "rid": "win-x86",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          },
          "runtimes/win8-arm/native/e_sqlite3.dll": {
            "rid": "win8-arm",
            "assetType": "native",
            "fileVersion": "0.0.0.0"
          }
        }
      },
      "SQLitePCLRaw.provider.e_sqlite3.netstandard11/1.1.12": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "SQLitePCLRaw.core": "1.1.12"
        },
        "runtime": {
          "lib/netstandard1.1/SQLitePCLRaw.provider.e_sqlite3.dll": {
            "assemblyVersion": "1.1.12.351",
            "fileVersion": "1.0.0.0"
          }
        },
        "compile": {
          "lib/netstandard1.1/SQLitePCLRaw.provider.e_sqlite3.dll": {}
        }
      },
      "StackExchange.Redis/2.0.601": {
        "dependencies": {
          "Pipelines.Sockets.Unofficial": "2.0.22",
          "System.Diagnostics.PerformanceCounter": "4.5.0",
          "System.IO.Pipelines": "4.5.2",
          "System.Threading.Channels": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/StackExchange.Redis.dll": {
            "assemblyVersion": "2.0.0.0",
            "fileVersion": "2.0.601.3402"
          }
        },
        "compile": {
          "lib/netstandard2.0/StackExchange.Redis.dll": {}
        }
      },
      "Swashbuckle.AspNetCore/4.0.1": {
        "dependencies": {
          "Swashbuckle.AspNetCore.Swagger": "4.0.1",
          "Swashbuckle.AspNetCore.SwaggerGen": "4.0.1",
          "Swashbuckle.AspNetCore.SwaggerUI": "4.0.1"
        },
        "runtime": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.dll": {
            "assemblyVersion": "4.0.1.0",
            "fileVersion": "4.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.dll": {}
        }
      },
      "Swashbuckle.AspNetCore.Swagger/4.0.1": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Formatters.Json": "2.2.0"
        },
        "runtime": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.Swagger.dll": {
            "assemblyVersion": "4.0.1.0",
            "fileVersion": "4.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.Swagger.dll": {}
        }
      },
      "Swashbuckle.AspNetCore.SwaggerGen/4.0.1": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.ApiExplorer": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0",
          "Microsoft.AspNetCore.Mvc.DataAnnotations": "2.2.0",
          "Swashbuckle.AspNetCore.Swagger": "4.0.1"
        },
        "runtime": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.SwaggerGen.dll": {
            "assemblyVersion": "4.0.1.0",
            "fileVersion": "4.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.SwaggerGen.dll": {}
        }
      },
      "Swashbuckle.AspNetCore.SwaggerUI/4.0.1": {
        "dependencies": {
          "Microsoft.AspNetCore.Routing": "2.2.0",
          "Microsoft.AspNetCore.StaticFiles": "2.2.0",
          "Microsoft.Extensions.FileProviders.Embedded": "2.2.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.SwaggerUI.dll": {
            "assemblyVersion": "4.0.1.0",
            "fileVersion": "4.0.1.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Swashbuckle.AspNetCore.SwaggerUI.dll": {}
        }
      },
      "System.Collections.Immutable/1.5.0": {},
      "System.Composition/1.0.31": {
        "dependencies": {
          "System.Composition.AttributedModel": "1.0.31",
          "System.Composition.Convention": "1.0.31",
          "System.Composition.Hosting": "1.0.31",
          "System.Composition.Runtime": "1.0.31",
          "System.Composition.TypedParts": "1.0.31"
        }
      },
      "System.Composition.AttributedModel/1.0.31": {
        "dependencies": {
          "System.Reflection": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/System.Composition.AttributedModel.dll": {
            "assemblyVersion": "1.0.31.0",
            "fileVersion": "4.6.24705.1"
          }
        },
        "compile": {
          "lib/netstandard1.0/System.Composition.AttributedModel.dll": {}
        }
      },
      "System.Composition.Convention/1.0.31": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Composition.AttributedModel": "1.0.31",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/System.Composition.Convention.dll": {
            "assemblyVersion": "1.0.31.0",
            "fileVersion": "4.6.24705.1"
          }
        },
        "compile": {
          "lib/netstandard1.0/System.Composition.Convention.dll": {}
        }
      },
      "System.Composition.Hosting/1.0.31": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Composition.Runtime": "1.0.31",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.ObjectModel": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/System.Composition.Hosting.dll": {
            "assemblyVersion": "1.0.31.0",
            "fileVersion": "4.6.24705.1"
          }
        },
        "compile": {
          "lib/netstandard1.0/System.Composition.Hosting.dll": {}
        }
      },
      "System.Composition.Runtime/1.0.31": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/System.Composition.Runtime.dll": {
            "assemblyVersion": "1.0.31.0",
            "fileVersion": "4.6.24705.1"
          }
        },
        "compile": {
          "lib/netstandard1.0/System.Composition.Runtime.dll": {}
        }
      },
      "System.Composition.TypedParts/1.0.31": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Composition.AttributedModel": "1.0.31",
          "System.Composition.Hosting": "1.0.31",
          "System.Composition.Runtime": "1.0.31",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0"
        },
        "runtime": {
          "lib/netstandard1.0/System.Composition.TypedParts.dll": {
            "assemblyVersion": "1.0.31.0",
            "fileVersion": "4.6.24705.1"
          }
        },
        "compile": {
          "lib/netstandard1.0/System.Composition.TypedParts.dll": {}
        }
      },
      "System.Configuration.ConfigurationManager/4.5.0": {
        "dependencies": {
          "System.Security.Cryptography.ProtectedData": "4.5.0",
          "System.Security.Permissions": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/System.Configuration.ConfigurationManager.dll": {
            "assemblyVersion": "4.0.1.0",
            "fileVersion": "4.6.26515.6"
          }
        }
      },
      "System.Data.Common/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        }
      },
      "System.Data.SqlClient/4.6.1": {
        "dependencies": {
          "Microsoft.Win32.Registry": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0",
          "System.Text.Encoding.CodePages": "4.5.0",
          "runtime.native.System.Data.SqlClient.sni": "4.5.0"
        },
        "runtime": {
          "lib/netcoreapp2.1/System.Data.SqlClient.dll": {
            "assemblyVersion": "4.5.0.1",
            "fileVersion": "4.6.27618.1"
          }
        },
        "runtimeTargets": {
          "runtimes/unix/lib/netcoreapp2.1/System.Data.SqlClient.dll": {
            "rid": "unix",
            "assetType": "runtime",
            "assemblyVersion": "4.5.0.1",
            "fileVersion": "4.6.27618.1"
          },
          "runtimes/win/lib/netcoreapp2.1/System.Data.SqlClient.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.5.0.1",
            "fileVersion": "4.6.27618.1"
          }
        },
        "compile": {
          "ref/netcoreapp2.1/System.Data.SqlClient.dll": {}
        }
      },
      "System.Diagnostics.PerformanceCounter/4.5.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.Win32.Registry": "4.5.0",
          "System.Configuration.ConfigurationManager": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/System.Diagnostics.PerformanceCounter.dll": {
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "runtimeTargets": {
          "runtimes/win/lib/netcoreapp2.0/System.Diagnostics.PerformanceCounter.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "compile": {
          "ref/netstandard2.0/System.Diagnostics.PerformanceCounter.dll": {}
        }
      },
      "System.Diagnostics.Process/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.Win32.Primitives": "4.3.0",
          "Microsoft.Win32.Registry": "4.5.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Text.Encoding.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Threading.Thread": "4.3.0",
          "System.Threading.ThreadPool": "4.3.0",
          "runtime.native.System": "4.3.0"
        },
        "runtimeTargets": {
          "runtime/linux/lib/_._": {
            "rid": "linux",
            "assetType": "runtime"
          },
          "runtime/osx/lib/_._": {
            "rid": "osx",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Drawing.Common/4.5.1": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.Win32.SystemEvents": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/System.Drawing.Common.dll": {
            "assemblyVersion": "4.0.0.1",
            "fileVersion": "4.6.26919.2"
          }
        },
        "runtimeTargets": {
          "runtimes/unix/lib/netcoreapp2.0/System.Drawing.Common.dll": {
            "rid": "unix",
            "assetType": "runtime",
            "assemblyVersion": "4.0.0.1",
            "fileVersion": "4.6.26919.2"
          },
          "runtimes/win/lib/netcoreapp2.0/System.Drawing.Common.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.0.0.1",
            "fileVersion": "4.6.26919.2"
          }
        },
        "compile": {
          "ref/netstandard2.0/System.Drawing.Common.dll": {}
        }
      },
      "System.Drawing.Primitives/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0"
        }
      },
      "System.Interactive.Async/3.2.0": {
        "runtime": {
          "lib/netstandard2.0/System.Interactive.Async.dll": {
            "assemblyVersion": "3.2.0.0",
            "fileVersion": "3.2.0.702"
          }
        },
        "compile": {
          "lib/netstandard2.0/System.Interactive.Async.dll": {}
        }
      },
      "System.IO.FileSystem.AccessControl/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.IO.FileSystem": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Security.AccessControl": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        },
        "compile": {
          "ref/netstandard1.3/System.IO.FileSystem.AccessControl.dll": {}
        }
      },
      "System.IO.Packaging/4.5.0": {
        "runtime": {
          "lib/netstandard2.0/System.IO.Packaging.dll": {
            "assemblyVersion": "4.0.3.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "compile": {
          "ref/netstandard2.0/System.IO.Packaging.dll": {}
        }
      },
      "System.Linq.Parallel/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        }
      },
      "System.Linq.Queryable/4.0.1": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0"
        }
      },
      "System.Net.Http/4.3.4": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Globalization.Extensions": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.OpenSsl": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Security.Cryptography.X509Certificates": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "runtime.native.System": "4.3.0",
          "runtime.native.System.Net.Http": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Net.NameResolution/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Principal.Windows": "4.5.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "runtime.native.System": "4.3.0"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Net.NetworkInformation/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.Win32.Primitives": "4.3.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Net.Sockets": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Principal.Windows": "4.5.0",
          "System.Threading": "4.3.0",
          "System.Threading.Overlapped": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Threading.Thread": "4.3.0",
          "System.Threading.ThreadPool": "4.3.0",
          "runtime.native.System": "4.3.0"
        },
        "runtimeTargets": {
          "runtime/linux/lib/_._": {
            "rid": "linux",
            "assetType": "runtime"
          },
          "runtime/osx/lib/_._": {
            "rid": "osx",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Net.Primitives/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Handles": "4.3.0"
        }
      },
      "System.Net.Requests/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Net.Http": "4.3.4",
          "System.Net.Primitives": "4.3.0",
          "System.Net.WebHeaderCollection": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Net.Security/4.3.2": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.Win32.Primitives": "4.3.0",
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Globalization.Extensions": "4.3.0",
          "System.IO": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Claims": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.OpenSsl": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Security.Cryptography.X509Certificates": "4.3.0",
          "System.Security.Principal": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Threading.ThreadPool": "4.3.0",
          "runtime.native.System": "4.3.0",
          "runtime.native.System.Net.Security": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Net.Sockets/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.IO": "4.3.0",
          "System.Net.Primitives": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        }
      },
      "System.Net.WebHeaderCollection/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0"
        }
      },
      "System.Private.ServiceModel/4.6.0": {
        "dependencies": {
          "System.Reflection.DispatchProxy": "4.5.0",
          "System.Security.Cryptography.Xml": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "runtime": {
          "lib/netstandard2.0/System.Private.ServiceModel.dll": {
            "assemblyVersion": "4.6.0.0",
            "fileVersion": "4.600.19.47001"
          }
        }
      },
      "System.Reflection.DispatchProxy/4.5.0": {},
      "System.Runtime.CompilerServices.Unsafe/4.5.2": {
        "runtime": {
          "lib/netcoreapp2.0/System.Runtime.CompilerServices.Unsafe.dll": {
            "assemblyVersion": "4.0.4.1",
            "fileVersion": "4.6.26919.2"
          }
        },
        "compile": {
          "ref/netstandard2.0/System.Runtime.CompilerServices.Unsafe.dll": {}
        }
      },
      "System.Runtime.Serialization.Json/4.3.0": {
        "dependencies": {
          "System.IO": "4.3.0",
          "System.Private.DataContractSerialization": "4.3.0",
          "System.Runtime": "4.3.0"
        }
      },
      "System.Security.Cryptography.ProtectedData/4.5.0": {
        "runtime": {
          "lib/netstandard2.0/System.Security.Cryptography.ProtectedData.dll": {
            "assemblyVersion": "4.0.3.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "runtimeTargets": {
          "runtimes/win/lib/netstandard2.0/System.Security.Cryptography.ProtectedData.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.0.3.0",
            "fileVersion": "4.6.26515.6"
          }
        }
      },
      "System.ServiceModel.Primitives/4.6.0": {
        "dependencies": {
          "System.Private.ServiceModel": "4.6.0"
        },
        "runtime": {
          "lib/netcoreapp2.1/System.ServiceModel.Primitives.dll": {
            "assemblyVersion": "4.6.0.0",
            "fileVersion": "4.600.19.47001"
          },
          "lib/netcoreapp2.1/System.ServiceModel.dll": {
            "assemblyVersion": "4.0.0.0",
            "fileVersion": "4.600.19.47001"
          }
        },
        "compile": {
          "ref/netcoreapp2.1/System.ServiceModel.Primitives.dll": {},
          "ref/netcoreapp2.1/System.ServiceModel.dll": {}
        }
      },
      "System.Text.Encoding.CodePages/4.5.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Runtime.CompilerServices.Unsafe": "4.5.2"
        },
        "runtime": {
          "lib/netstandard2.0/System.Text.Encoding.CodePages.dll": {
            "assemblyVersion": "4.1.1.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "runtimeTargets": {
          "runtimes/win/lib/netcoreapp2.0/System.Text.Encoding.CodePages.dll": {
            "rid": "win",
            "assetType": "runtime",
            "assemblyVersion": "4.1.1.0",
            "fileVersion": "4.6.26515.6"
          }
        },
        "compile": {
          "ref/netstandard2.0/System.Text.Encoding.CodePages.dll": {}
        }
      },
      "System.Threading.Overlapped/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Handles": "4.3.0"
        },
        "runtimeTargets": {
          "runtime/unix/lib/_._": {
            "rid": "unix",
            "assetType": "runtime"
          },
          "runtime/win/lib/_._": {
            "rid": "win",
            "assetType": "runtime"
          }
        }
      },
      "System.Threading.ThreadPool/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0",
          "System.Runtime.Handles": "4.3.0"
        }
      },
      "WindowsAzure.Storage/9.3.3": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "Newtonsoft.Json": "12.0.2"
        },
        "runtime": {
          "lib/netstandard1.3/Microsoft.WindowsAzure.Storage.dll": {
            "assemblyVersion": "9.3.2.0",
            "fileVersion": "9.3.2.0"
          }
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.WindowsAzure.Storage.dll": {}
        }
      },
      "Z.EntityFramework.Plus.EFCore/2.0.7": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6"
        },
        "runtime": {
          "lib/netstandard2.0/Z.EntityFramework.Plus.EFCore.dll": {
            "assemblyVersion": "2.0.7.0",
            "fileVersion": "2.0.7.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/Z.EntityFramework.Plus.EFCore.dll": {}
        }
      },
      "ZXing.Net/0.16.4": {
        "dependencies": {
          "NETStandard.Library": "2.0.3"
        },
        "runtime": {
          "lib/netstandard2.0/zxing.dll": {
            "assemblyVersion": "0.16.4.0",
            "fileVersion": "0.16.4.0"
          }
        },
        "compile": {
          "lib/netstandard2.0/zxing.dll": {}
        }
      },
      "MemberPlatform.Cmon/1.0.0": {
        "dependencies": {
          "HtmlSanitizer": "4.0.217",
          "MemberPlatform.I18n": "1.0.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Drawing.Common": "4.5.1",
          "System.Drawing.Primitives": "4.3.0",
          "System.ServiceModel.Primitives": "4.6.0",
          "ZXing.Net": "0.16.4"
        },
        "runtime": {
          "MemberPlatform.Cmon.dll": {}
        },
        "compile": {
          "MemberPlatform.Cmon.dll": {}
        }
      },
      "MemberPlatform.Core/1.0.0": {
        "dependencies": {
          "AutoMapper": "9.0.0",
          "DocumentFormat.OpenXml": "2.9.1",
          "Elastique.DapperProvider": "1.0.2",
          "LinqKit.Microsoft.EntityFrameworkCore": "1.1.16",
          "MemberPlatform.Cmon": "1.0.0",
          "MemberPlatform.Core.Contracts": "1.0.0",
          "MemberPlatform.Entities": "1.0.0",
          "MemberPlatform.I18n": "1.0.0",
          "MemberPlatform.Persistence": "1.0.0",
          "MemberPlatform.Persistence.Contracts": "1.0.0",
          "MemberPlatform.Store.Contracts": "1.0.0",
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.EntityFrameworkCore.InMemory": "2.2.6",
          "Microsoft.Extensions.Http": "2.2.0",
          "Microsoft.Extensions.Identity.Core": "2.2.0",
          "Newtonsoft.Json": "12.0.2",
          "SharpZipLib": "1.2.0"
        },
        "runtime": {
          "MemberPlatform.Core.dll": {}
        },
        "compile": {
          "MemberPlatform.Core.dll": {}
        }
      },
      "MemberPlatform.Core.Contracts/1.0.0": {
        "dependencies": {
          "MailKit": "2.3.0",
          "MemberPlatform.Cmon": "1.0.0",
          "MemberPlatform.Entities": "1.0.0",
          "MemberPlatform.Store.Contracts": "1.0.0",
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.Identity.Core": "2.2.0",
          "WindowsAzure.Storage": "9.3.3",
          "protobuf-net": "2.4.0"
        },
        "runtime": {
          "MemberPlatform.Core.Contracts.dll": {}
        },
        "compile": {
          "MemberPlatform.Core.Contracts.dll": {}
        }
      },
      "MemberPlatform.Entities/1.0.0": {
        "dependencies": {
          "MemberPlatform.Cmon": "1.0.0",
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.AspNetCore.Identity.EntityFrameworkCore": "2.2.0",
          "Microsoft.EntityFrameworkCore": "2.2.6",
          "Microsoft.Extensions.Identity.Stores": "2.2.0"
        },
        "runtime": {
          "MemberPlatform.Entities.dll": {}
        },
        "compile": {
          "MemberPlatform.Entities.dll": {}
        }
      },
      "MemberPlatform.I18n/1.0.0": {
        "dependencies": {
          "Elastique.StatusLibrary": "1.0.1"
        },
        "runtime": {
          "MemberPlatform.I18n.dll": {}
        },
        "resources": {
          "es-ES/MemberPlatform.I18n.resources.dll": {
            "locale": "es-ES"
          },
          "fr-FR/MemberPlatform.I18n.resources.dll": {
            "locale": "fr-FR"
          }
        },
        "compile": {
          "MemberPlatform.I18n.dll": {}
        }
      },
      "MemberPlatform.Mapping/1.0.0": {
        "dependencies": {
          "AutoMapper": "9.0.0",
          "AutoMapper.Extensions.Microsoft.DependencyInjection": "7.0.0",
          "MemberPlatform.Core": "1.0.0",
          "MemberPlatform.Persistence": "1.0.0",
          "MemberPlatform.Store": "1.0.0",
          "Microsoft.AspNetCore.Authentication.JwtBearer": "2.2.0",
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.Configuration.EnvironmentVariables": "2.2.4",
          "Microsoft.Extensions.Configuration.Json": "2.2.0",
          "Microsoft.Extensions.Configuration.UserSecrets": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0"
        },
        "runtime": {
          "MemberPlatform.Mapping.dll": {}
        },
        "compile": {
          "MemberPlatform.Mapping.dll": {}
        }
      },
      "MemberPlatform.Persistence/1.0.0": {
        "dependencies": {
          "Dapper": "2.0.4",
          "EFCore.BulkExtensions": "2.6.0",
          "LinqKit.Microsoft.EntityFrameworkCore": "1.1.16",
          "MemberPlatform.Cmon": "1.0.0",
          "MemberPlatform.Entities": "1.0.0",
          "MemberPlatform.Persistence.Contracts": "1.0.0",
          "Microsoft.AspNetCore.Authentication.JwtBearer": "2.2.0",
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.AspNetCore.Identity.EntityFrameworkCore": "2.2.0",
          "Microsoft.EntityFrameworkCore": "2.2.6",
          "Microsoft.EntityFrameworkCore.InMemory": "2.2.6",
          "Microsoft.EntityFrameworkCore.SqlServer": "2.2.6",
          "Z.EntityFramework.Plus.EFCore": "2.0.7"
        },
        "runtime": {
          "MemberPlatform.Persistence.dll": {}
        },
        "compile": {
          "MemberPlatform.Persistence.dll": {}
        }
      },
      "MemberPlatform.Persistence.Contracts/1.0.0": {
        "dependencies": {
          "MemberPlatform.Cmon": "1.0.0",
          "MemberPlatform.Core.Contracts": "1.0.0",
          "MemberPlatform.Entities": "1.0.0",
          "Microsoft.EntityFrameworkCore": "2.2.6",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6"
        },
        "runtime": {
          "MemberPlatform.Persistence.Contracts.dll": {}
        },
        "compile": {
          "MemberPlatform.Persistence.Contracts.dll": {}
        }
      },
      "MemberPlatform.Store/1.0.0": {
        "dependencies": {
          "MemberPlatform.Cmon": "1.0.0",
          "MemberPlatform.Core.Contracts": "1.0.0",
          "MemberPlatform.Store.Contracts": "1.0.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "StackExchange.Redis": "2.0.601",
          "WindowsAzure.Storage": "9.3.3",
          "protobuf-net": "2.4.0"
        },
        "runtime": {
          "MemberPlatform.Store.dll": {}
        },
        "compile": {
          "MemberPlatform.Store.dll": {}
        }
      },
      "MemberPlatform.Store.Contracts/1.0.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Features": "2.2.0",
          "WindowsAzure.Storage": "9.3.3",
          "protobuf-net": "2.4.0"
        },
        "runtime": {
          "MemberPlatform.Store.Contracts.dll": {}
        },
        "compile": {
          "MemberPlatform.Store.Contracts.dll": {}
        }
      },
      "Microsoft.AspNet.WebApi.Client/5.2.6": {
        "dependencies": {
          "Newtonsoft.Json": "12.0.2",
          "Newtonsoft.Json.Bson": "1.0.1"
        },
        "compile": {
          "lib/netstandard2.0/System.Net.Http.Formatting.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Diagnostics": "2.2.0",
          "Microsoft.AspNetCore.HostFiltering": "2.2.0",
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.AspNetCore.Routing": "2.2.0",
          "Microsoft.AspNetCore.Server.IIS": "2.2.0",
          "Microsoft.AspNetCore.Server.IISIntegration": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Https": "2.2.0",
          "Microsoft.Extensions.Configuration.CommandLine": "2.2.0",
          "Microsoft.Extensions.Configuration.EnvironmentVariables": "2.2.4",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0",
          "Microsoft.Extensions.Configuration.Json": "2.2.0",
          "Microsoft.Extensions.Configuration.UserSecrets": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Logging.Configuration": "2.2.0",
          "Microsoft.Extensions.Logging.Console": "2.2.0",
          "Microsoft.Extensions.Logging.Debug": "2.2.0",
          "Microsoft.Extensions.Logging.EventSource": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Antiforgery/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.DataProtection": "2.2.0",
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.WebUtilities": "2.2.0",
          "Microsoft.Extensions.ObjectPool": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Antiforgery.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.App/2.2.0": {
        "dependencies": {
          "Microsoft.AspNet.WebApi.Client": "5.2.6",
          "Microsoft.AspNetCore": "2.2.0",
          "Microsoft.AspNetCore.Antiforgery": "2.2.0",
          "Microsoft.AspNetCore.Authentication": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Cookies": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Facebook": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Google": "2.2.0",
          "Microsoft.AspNetCore.Authentication.JwtBearer": "2.2.0",
          "Microsoft.AspNetCore.Authentication.MicrosoftAccount": "2.2.0",
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0",
          "Microsoft.AspNetCore.Authentication.OpenIdConnect": "2.2.0",
          "Microsoft.AspNetCore.Authentication.Twitter": "2.2.0",
          "Microsoft.AspNetCore.Authentication.WsFederation": "2.2.0",
          "Microsoft.AspNetCore.Authorization": "2.2.0",
          "Microsoft.AspNetCore.Authorization.Policy": "2.2.0",
          "Microsoft.AspNetCore.Connections.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.CookiePolicy": "2.2.0",
          "Microsoft.AspNetCore.Cors": "2.2.0",
          "Microsoft.AspNetCore.Cryptography.Internal": "2.2.0",
          "Microsoft.AspNetCore.Cryptography.KeyDerivation": "2.2.0",
          "Microsoft.AspNetCore.DataProtection": "2.2.0",
          "Microsoft.AspNetCore.DataProtection.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.DataProtection.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Diagnostics": "2.2.0",
          "Microsoft.AspNetCore.Diagnostics.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore": "2.2.0",
          "Microsoft.AspNetCore.Diagnostics.HealthChecks": "2.2.0",
          "Microsoft.AspNetCore.HostFiltering": "2.2.0",
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Server.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Html.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Connections": "1.1.0",
          "Microsoft.AspNetCore.Http.Connections.Common": "1.1.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Http.Features": "2.2.0",
          "Microsoft.AspNetCore.HttpOverrides": "2.2.0",
          "Microsoft.AspNetCore.HttpsPolicy": "2.2.0",
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.AspNetCore.Identity.EntityFrameworkCore": "2.2.0",
          "Microsoft.AspNetCore.Identity.UI": "2.2.0",
          "Microsoft.AspNetCore.JsonPatch": "2.2.0",
          "Microsoft.AspNetCore.Localization": "2.2.0",
          "Microsoft.AspNetCore.Localization.Routing": "2.2.0",
          "Microsoft.AspNetCore.MiddlewareAnalysis": "2.2.0",
          "Microsoft.AspNetCore.Mvc": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Analyzers": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ApiExplorer": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Cors": "2.2.0",
          "Microsoft.AspNetCore.Mvc.DataAnnotations": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Formatters.Json": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Formatters.Xml": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Localization": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Razor": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Razor.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Razor.ViewCompilation": "2.2.0",
          "Microsoft.AspNetCore.Mvc.RazorPages": "2.2.0",
          "Microsoft.AspNetCore.Mvc.TagHelpers": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ViewFeatures": "2.2.0",
          "Microsoft.AspNetCore.NodeServices": "2.2.0",
          "Microsoft.AspNetCore.Owin": "2.2.0",
          "Microsoft.AspNetCore.Razor": "2.2.0",
          "Microsoft.AspNetCore.Razor.Design": "2.2.0",
          "Microsoft.AspNetCore.Razor.Language": "2.2.0",
          "Microsoft.AspNetCore.Razor.Runtime": "2.2.0",
          "Microsoft.AspNetCore.ResponseCaching": "2.2.0",
          "Microsoft.AspNetCore.ResponseCaching.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.ResponseCompression": "2.2.0",
          "Microsoft.AspNetCore.Rewrite": "2.2.0",
          "Microsoft.AspNetCore.Routing": "2.2.0",
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Server.HttpSys": "2.2.0",
          "Microsoft.AspNetCore.Server.IIS": "2.2.0",
          "Microsoft.AspNetCore.Server.IISIntegration": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Core": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Https": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets": "2.2.0",
          "Microsoft.AspNetCore.Session": "2.2.0",
          "Microsoft.AspNetCore.SignalR": "1.1.0",
          "Microsoft.AspNetCore.SignalR.Common": "1.1.0",
          "Microsoft.AspNetCore.SignalR.Core": "1.1.0",
          "Microsoft.AspNetCore.SignalR.Protocols.Json": "1.1.0",
          "Microsoft.AspNetCore.SpaServices": "2.2.0",
          "Microsoft.AspNetCore.SpaServices.Extensions": "2.2.0",
          "Microsoft.AspNetCore.StaticFiles": "2.2.0",
          "Microsoft.AspNetCore.WebSockets": "2.2.0",
          "Microsoft.AspNetCore.WebUtilities": "2.2.0",
          "Microsoft.CodeAnalysis.Razor": "2.2.0",
          "Microsoft.EntityFrameworkCore": "2.2.6",
          "Microsoft.EntityFrameworkCore.Abstractions": "2.2.6",
          "Microsoft.EntityFrameworkCore.Analyzers": "2.2.6",
          "Microsoft.EntityFrameworkCore.Design": "2.2.0",
          "Microsoft.EntityFrameworkCore.InMemory": "2.2.6",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6",
          "Microsoft.EntityFrameworkCore.SqlServer": "2.2.6",
          "Microsoft.EntityFrameworkCore.Tools": "2.2.0",
          "Microsoft.Extensions.Caching.Abstractions": "2.2.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.Caching.SqlServer": "2.2.0",
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.Configuration.CommandLine": "2.2.0",
          "Microsoft.Extensions.Configuration.EnvironmentVariables": "2.2.4",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0",
          "Microsoft.Extensions.Configuration.Ini": "2.2.0",
          "Microsoft.Extensions.Configuration.Json": "2.2.0",
          "Microsoft.Extensions.Configuration.KeyPerFile": "2.2.0",
          "Microsoft.Extensions.Configuration.UserSecrets": "2.2.0",
          "Microsoft.Extensions.Configuration.Xml": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.DiagnosticAdapter": "2.2.0",
          "Microsoft.Extensions.Diagnostics.HealthChecks": "2.2.0",
          "Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Composite": "2.2.0",
          "Microsoft.Extensions.FileProviders.Embedded": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0",
          "Microsoft.Extensions.FileSystemGlobbing": "2.2.0",
          "Microsoft.Extensions.Hosting": "2.2.0",
          "Microsoft.Extensions.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Http": "2.2.0",
          "Microsoft.Extensions.Identity.Core": "2.2.0",
          "Microsoft.Extensions.Identity.Stores": "2.2.0",
          "Microsoft.Extensions.Localization": "2.2.0",
          "Microsoft.Extensions.Localization.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Configuration": "2.2.0",
          "Microsoft.Extensions.Logging.Console": "2.2.0",
          "Microsoft.Extensions.Logging.Debug": "2.2.0",
          "Microsoft.Extensions.Logging.EventSource": "2.2.0",
          "Microsoft.Extensions.Logging.TraceSource": "2.2.0",
          "Microsoft.Extensions.ObjectPool": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Extensions.Options.ConfigurationExtensions": "2.2.0",
          "Microsoft.Extensions.Options.DataAnnotations": "2.2.0",
          "Microsoft.Extensions.Primitives": "2.2.0",
          "Microsoft.Extensions.WebEncoders": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0",
          "System.IO.Pipelines": "4.5.2"
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.DataProtection": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Extensions.WebEncoders": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Cookies/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Cookies.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Core/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Core.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Facebook/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Facebook.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Google/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Google.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.JwtBearer/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication": "2.2.0",
          "Microsoft.IdentityModel.Protocols.OpenIdConnect": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.JwtBearer.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.MicrosoftAccount/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.MicrosoftAccount.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.OAuth/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication": "2.2.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.OAuth.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.OpenIdConnect/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0",
          "Microsoft.IdentityModel.Protocols.OpenIdConnect": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.OpenIdConnect.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.Twitter/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.OAuth": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.Twitter.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authentication.WsFederation/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication": "2.2.0",
          "Microsoft.IdentityModel.Protocols.WsFederation": "5.3.0",
          "System.IdentityModel.Tokens.Jwt": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authentication.WsFederation.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authorization/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authorization.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Authorization.Policy/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Authorization": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Authorization.Policy.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Connections.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Features": "2.2.0",
          "System.IO.Pipelines": "4.5.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Connections.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.CookiePolicy/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.CookiePolicy.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Cors/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Cors.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Cryptography.Internal/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Cryptography.Internal.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Cryptography.KeyDerivation/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Cryptography.Internal": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.0/Microsoft.AspNetCore.Cryptography.KeyDerivation.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.DataProtection/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Cryptography.Internal": "2.2.0",
          "Microsoft.AspNetCore.DataProtection.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Win32.Registry": "4.5.0",
          "System.Security.Cryptography.Xml": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.DataProtection.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.DataProtection.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.DataProtection.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.DataProtection.Extensions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.DataProtection": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.DataProtection.Extensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Diagnostics/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Diagnostics.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.WebUtilities": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Reflection.Metadata": "1.6.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Diagnostics.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Diagnostics.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Diagnostics.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Diagnostics.HealthChecks/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.Diagnostics.HealthChecks": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Diagnostics.HealthChecks.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.HostFiltering/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.HostFiltering.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Hosting/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.EnvironmentVariables": "2.2.4",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0",
          "Microsoft.Extensions.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Reflection.Metadata": "1.6.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Hosting.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Hosting.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Server.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.Hosting.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Hosting.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Hosting.Server.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Features": "2.2.0",
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Hosting.Server.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Html.Abstractions/2.2.0": {
        "dependencies": {
          "System.Text.Encodings.Web": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Html.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.WebUtilities": "2.2.0",
          "Microsoft.Extensions.ObjectPool": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Http.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Features": "2.2.0",
          "System.Text.Encodings.Web": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Http.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http.Connections/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authorization.Policy": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Connections.Common": "1.1.0",
          "Microsoft.AspNetCore.Routing": "2.2.0",
          "Microsoft.AspNetCore.WebSockets": "2.2.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "lib/netcoreapp2.2/Microsoft.AspNetCore.Http.Connections.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http.Connections.Common/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Connections.Abstractions": "2.2.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Buffers": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Http.Connections.Common.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http.Extensions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0",
          "System.Buffers": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Http.Extensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Http.Features/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Http.Features.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.HttpOverrides/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.HttpOverrides.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.HttpsPolicy/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.HttpsPolicy.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Identity/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Cookies": "2.2.0",
          "Microsoft.AspNetCore.Cryptography.KeyDerivation": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Identity.Core": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Identity.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Identity.EntityFrameworkCore/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6",
          "Microsoft.Extensions.Identity.Stores": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Identity.EntityFrameworkCore.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Identity.UI/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Identity": "2.2.0",
          "Microsoft.AspNetCore.Mvc": "2.2.0",
          "Microsoft.AspNetCore.StaticFiles": "2.2.0",
          "Microsoft.Extensions.FileProviders.Embedded": "2.2.0",
          "Microsoft.Extensions.Identity.Stores": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Identity.UI.Views.V3.dll": {},
          "lib/netstandard2.0/Microsoft.AspNetCore.Identity.UI.Views.V4.dll": {},
          "lib/netstandard2.0/Microsoft.AspNetCore.Identity.UI.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.JsonPatch/2.2.0": {
        "dependencies": {
          "Microsoft.CSharp": "4.5.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.JsonPatch.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Localization/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Localization.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Localization.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Localization.Routing/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Localization": "2.2.0",
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Localization.Routing.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.MiddlewareAnalysis/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.MiddlewareAnalysis.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Analyzers": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ApiExplorer": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Cors": "2.2.0",
          "Microsoft.AspNetCore.Mvc.DataAnnotations": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Formatters.Json": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Localization": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Razor.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.RazorPages": "2.2.0",
          "Microsoft.AspNetCore.Mvc.TagHelpers": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ViewFeatures": "2.2.0",
          "Microsoft.AspNetCore.Razor.Design": "2.2.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Analyzers/2.2.0": {
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.ApiExplorer/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.ApiExplorer.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Core/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.Authorization.Policy": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.ResponseCaching.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Routing": "2.2.0",
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.DependencyModel": "2.1.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "System.Diagnostics.DiagnosticSource": "4.5.0",
          "System.Threading.Tasks.Extensions": "4.5.1"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Core.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Cors/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Cors": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Cors.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.DataAnnotations/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0",
          "Microsoft.Extensions.Localization": "2.2.0",
          "System.ComponentModel.Annotations": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.DataAnnotations.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Formatters.Json/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.JsonPatch": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Formatters.Json.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Formatters.Xml/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Formatters.Xml.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Localization/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Localization": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Razor": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.Localization": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Localization.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Razor/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Razor.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ViewFeatures": "2.2.0",
          "Microsoft.AspNetCore.Razor.Runtime": "2.2.0",
          "Microsoft.CodeAnalysis.CSharp": "2.8.0",
          "Microsoft.CodeAnalysis.Razor": "2.2.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.FileProviders.Composite": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Razor.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Razor.Extensions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Razor.Language": "2.2.0",
          "Microsoft.CodeAnalysis.Razor": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Razor.Extensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.Razor.ViewCompilation/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.AspNetCore.Mvc.RazorPages": "2.2.0"
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.RazorPages/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Razor": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.RazorPages.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.TagHelpers/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.Razor": "2.2.0",
          "Microsoft.AspNetCore.Razor.Runtime": "2.2.0",
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.FileSystemGlobbing": "2.2.0",
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.TagHelpers.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Mvc.ViewFeatures/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Antiforgery": "2.2.0",
          "Microsoft.AspNetCore.Diagnostics.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Html.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Core": "2.2.0",
          "Microsoft.AspNetCore.Mvc.DataAnnotations": "2.2.0",
          "Microsoft.AspNetCore.Mvc.Formatters.Json": "2.2.0",
          "Microsoft.Extensions.WebEncoders": "2.2.0",
          "Newtonsoft.Json.Bson": "1.0.1"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Mvc.ViewFeatures.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.NodeServices/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Console": "2.2.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.NodeServices.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Owin/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Owin.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Razor/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Html.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Razor.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Razor.Design/2.2.0": {
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Razor.Language/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Razor.Language.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Razor.Runtime/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Html.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Razor": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Razor.Runtime.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.ResponseCaching/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.ResponseCaching.Abstractions": "2.2.0",
          "Microsoft.Extensions.Caching.Memory": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.ResponseCaching.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.ResponseCaching.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.ResponseCaching.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.ResponseCompression/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.1/Microsoft.AspNetCore.ResponseCompression.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Rewrite/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Rewrite.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Routing/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.Routing.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.ObjectPool": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.2/Microsoft.AspNetCore.Routing.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Routing.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Routing.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.HttpSys/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0",
          "Microsoft.Win32.Registry": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Server.HttpSys.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.IIS/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.Connections.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "System.IO.Pipelines": "4.5.2",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Server.IIS.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.IISIntegration/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authentication.Core": "2.2.0",
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.AspNetCore.HttpOverrides": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Buffers": "4.5.0",
          "System.IO.Pipelines": "4.5.2",
          "System.Memory": "4.5.1",
          "System.Numerics.Vectors": "4.5.0",
          "System.Runtime.CompilerServices.Unsafe": "4.5.2",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Server.IISIntegration.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.Kestrel/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Core": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Https": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Server.Kestrel.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.Kestrel.Core/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.WebUtilities": "2.2.0",
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Microsoft.Net.Http.Headers": "2.2.0",
          "System.Memory": "4.5.1",
          "System.Numerics.Vectors": "4.5.0",
          "System.Runtime.CompilerServices.Unsafe": "4.5.2",
          "System.Security.Cryptography.Cng": "4.5.0",
          "System.Threading.Tasks.Extensions": "4.5.1"
        },
        "compile": {
          "lib/netcoreapp2.1/Microsoft.AspNetCore.Server.Kestrel.Core.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.Kestrel.Https/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Core": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.1/Microsoft.AspNetCore.Server.Kestrel.Https.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Connections.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.1/Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.Session/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.DataProtection": "2.2.0",
          "Microsoft.AspNetCore.Http.Abstractions": "2.2.0",
          "Microsoft.Extensions.Caching.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.Session.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SignalR/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Connections": "1.1.0",
          "Microsoft.AspNetCore.SignalR.Core": "1.1.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.SignalR.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SignalR.Common/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Connections.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Buffers": "4.5.0"
        },
        "compile": {
          "lib/netcoreapp2.2/Microsoft.AspNetCore.SignalR.Common.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SignalR.Core/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Authorization": "2.2.0",
          "Microsoft.AspNetCore.SignalR.Common": "1.1.0",
          "Microsoft.AspNetCore.SignalR.Protocols.Json": "1.1.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "System.Reflection.Emit": "4.3.0",
          "System.Threading.Channels": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.SignalR.Core.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SignalR.Protocols.Json/1.1.0": {
        "dependencies": {
          "Microsoft.AspNetCore.SignalR.Common": "1.1.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.SignalR.Protocols.Json.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SpaServices/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Mvc.TagHelpers": "2.2.0",
          "Microsoft.AspNetCore.Mvc.ViewFeatures": "2.2.0",
          "Microsoft.AspNetCore.NodeServices": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.SpaServices.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.SpaServices.Extensions/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.SpaServices": "2.2.0",
          "Microsoft.AspNetCore.StaticFiles": "2.2.0",
          "Microsoft.AspNetCore.WebSockets": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.SpaServices.Extensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.StaticFiles/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Hosting.Abstractions": "2.2.0",
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.WebEncoders": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.StaticFiles.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.WebSockets/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Http.Extensions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Net.WebSockets.WebSocketProtocol": "4.5.1"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.WebSockets.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.AspNetCore.WebUtilities/2.2.0": {
        "dependencies": {
          "Microsoft.Net.Http.Headers": "2.2.0",
          "System.Text.Encodings.Web": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.AspNetCore.WebUtilities.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.CodeAnalysis.Analyzers/1.1.0": {
        "compileOnly": true
      },
      "Microsoft.CodeAnalysis.Common/2.8.0": {
        "dependencies": {
          "Microsoft.CodeAnalysis.Analyzers": "1.1.0",
          "System.AppContext": "4.3.0",
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Collections.Immutable": "1.5.0",
          "System.Console": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.FileVersionInfo": "4.3.0",
          "System.Diagnostics.StackTrace": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Dynamic.Runtime": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO.Compression": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Metadata": "1.6.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.Numerics": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.X509Certificates": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Text.Encoding.CodePages": "4.5.0",
          "System.Text.Encoding.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Threading.Tasks.Parallel": "4.3.0",
          "System.Threading.Thread": "4.3.0",
          "System.ValueTuple": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0",
          "System.Xml.XDocument": "4.3.0",
          "System.Xml.XPath.XDocument": "4.3.0",
          "System.Xml.XmlDocument": "4.3.0"
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.CodeAnalysis.CSharp/2.8.0": {
        "dependencies": {
          "Microsoft.CodeAnalysis.Common": "2.8.0"
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.CodeAnalysis.CSharp.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.CodeAnalysis.Razor/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Razor.Language": "2.2.0",
          "Microsoft.CodeAnalysis.CSharp": "2.8.0",
          "Microsoft.CodeAnalysis.Common": "2.8.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.CodeAnalysis.Razor.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.CSharp/4.5.0": {
        "compileOnly": true
      },
      "Microsoft.DotNet.PlatformAbstractions/2.1.0": {
        "dependencies": {
          "System.AppContext": "4.3.0",
          "System.Collections": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.InteropServices.RuntimeInformation": "4.3.0"
        },
        "compile": {
          "lib/netstandard1.3/Microsoft.DotNet.PlatformAbstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.EntityFrameworkCore.Design/2.2.0": {
        "dependencies": {
          "Microsoft.CSharp": "4.5.0",
          "Microsoft.EntityFrameworkCore.Relational": "2.2.6"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.EntityFrameworkCore.Design.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.EntityFrameworkCore.Tools/2.2.0": {
        "dependencies": {
          "Microsoft.EntityFrameworkCore.Design": "2.2.0"
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Caching.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Caching.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Caching.Memory/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Caching.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Caching.Memory.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Caching.SqlServer/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Caching.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Data.SqlClient": "4.6.1"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Caching.SqlServer.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.CommandLine/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.CommandLine.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.FileExtensions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.FileExtensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.Ini/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Ini.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.Json/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Json.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.KeyPerFile/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.KeyPerFile.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.UserSecrets/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Json": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.UserSecrets.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Configuration.Xml/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.Configuration.FileExtensions": "2.2.0",
          "System.Security.Cryptography.Xml": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Configuration.Xml.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.DependencyInjection/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netcoreapp2.0/Microsoft.Extensions.DependencyInjection.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.DependencyInjection.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.DependencyInjection.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.DependencyModel/2.1.0": {
        "dependencies": {
          "Microsoft.DotNet.PlatformAbstractions": "2.1.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Dynamic.Runtime": "4.3.0",
          "System.Linq": "4.3.0"
        },
        "compile": {
          "lib/netstandard1.6/Microsoft.Extensions.DependencyModel.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.DiagnosticAdapter/2.2.0": {
        "dependencies": {
          "System.Diagnostics.DiagnosticSource": "4.5.0"
        },
        "compile": {
          "lib/netcoreapp2.0/Microsoft.Extensions.DiagnosticAdapter.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Diagnostics.HealthChecks/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions": "2.2.0",
          "Microsoft.Extensions.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Diagnostics.HealthChecks.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.FileProviders.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.FileProviders.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.FileProviders.Composite/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.FileProviders.Composite.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.FileProviders.Embedded/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.FileProviders.Embedded.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.FileProviders.Physical/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileSystemGlobbing": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.FileProviders.Physical.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.FileSystemGlobbing/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.FileSystemGlobbing.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Hosting/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration": "2.2.0",
          "Microsoft.Extensions.DependencyInjection": "2.2.0",
          "Microsoft.Extensions.FileProviders.Physical": "2.2.0",
          "Microsoft.Extensions.Hosting.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Hosting.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Hosting.Abstractions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.FileProviders.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Hosting.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Http/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Http.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Identity.Core/2.2.0": {
        "dependencies": {
          "Microsoft.AspNetCore.Cryptography.KeyDerivation": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.ComponentModel.Annotations": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Identity.Core.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Identity.Stores/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Identity.Core": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "System.ComponentModel.Annotations": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Identity.Stores.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Localization/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Localization.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Localization.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Localization.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Localization.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.Abstractions/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.Abstractions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.Configuration/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Options.ConfigurationExtensions": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.Configuration.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.Console/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.Logging": "2.2.0",
          "Microsoft.Extensions.Logging.Configuration": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.Console.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.Debug/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Logging": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.Debug.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.EventSource/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Logging": "2.2.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.EventSource.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Logging.TraceSource/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Logging": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Logging.TraceSource.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.ObjectPool/2.2.0": {
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.ObjectPool.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Options/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Primitives": "2.2.0",
          "System.ComponentModel.Annotations": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Options.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Options.ConfigurationExtensions/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Configuration.Abstractions": "2.2.0",
          "Microsoft.Extensions.Configuration.Binder": "2.2.4",
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Options.ConfigurationExtensions.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Options.DataAnnotations/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.ComponentModel.Annotations": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Options.DataAnnotations.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.Primitives/2.2.0": {
        "dependencies": {
          "System.Memory": "4.5.1",
          "System.Runtime.CompilerServices.Unsafe": "4.5.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.Primitives.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Extensions.WebEncoders/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.DependencyInjection.Abstractions": "2.2.0",
          "Microsoft.Extensions.Options": "2.2.0",
          "System.Text.Encodings.Web": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Extensions.WebEncoders.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.JsonWebTokens/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Tokens": "5.3.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.JsonWebTokens.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Logging/5.3.0": {
        "dependencies": {
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Logging.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Protocols/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Logging": "5.3.0",
          "Microsoft.IdentityModel.Tokens": "5.3.0",
          "System.Collections.Specialized": "4.3.0",
          "System.Diagnostics.Contracts": "4.3.0",
          "System.Net.Http": "4.3.4"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Protocols.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Protocols.OpenIdConnect/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Protocols": "5.3.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Dynamic.Runtime": "4.3.0",
          "System.IdentityModel.Tokens.Jwt": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Protocols.OpenIdConnect.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Protocols.WsFederation/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Protocols": "5.3.0",
          "Microsoft.IdentityModel.Tokens.Saml": "5.3.0",
          "Microsoft.IdentityModel.Xml": "5.3.0",
          "System.Xml.XmlDocument": "4.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Protocols.WsFederation.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Tokens/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Logging": "5.3.0",
          "Newtonsoft.Json": "12.0.2",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.InteropServices.RuntimeInformation": "4.3.0",
          "System.Runtime.Serialization.Xml": "4.3.0",
          "System.Security.Claims": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.X509Certificates": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Tokens.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Tokens.Saml/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Tokens": "5.3.0",
          "Microsoft.IdentityModel.Xml": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Tokens.Saml.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.IdentityModel.Xml/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.Tokens": "5.3.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.IdentityModel.Xml.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.Net.Http.Headers/2.2.0": {
        "dependencies": {
          "Microsoft.Extensions.Primitives": "2.2.0",
          "System.Buffers": "4.5.0"
        },
        "compile": {
          "lib/netstandard2.0/Microsoft.Net.Http.Headers.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.NETCore.App/2.2.0": {
        "dependencies": {
          "Microsoft.NETCore.DotNetHostPolicy": "2.2.0",
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "NETStandard.Library": "2.0.3"
        },
        "compile": {
          "ref/netcoreapp2.2/Microsoft.CSharp.dll": {},
          "ref/netcoreapp2.2/Microsoft.VisualBasic.dll": {},
          "ref/netcoreapp2.2/Microsoft.Win32.Primitives.dll": {},
          "ref/netcoreapp2.2/System.AppContext.dll": {},
          "ref/netcoreapp2.2/System.Buffers.dll": {},
          "ref/netcoreapp2.2/System.Collections.Concurrent.dll": {},
          "ref/netcoreapp2.2/System.Collections.Immutable.dll": {},
          "ref/netcoreapp2.2/System.Collections.NonGeneric.dll": {},
          "ref/netcoreapp2.2/System.Collections.Specialized.dll": {},
          "ref/netcoreapp2.2/System.Collections.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.Annotations.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.DataAnnotations.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.EventBasedAsync.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.Primitives.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.TypeConverter.dll": {},
          "ref/netcoreapp2.2/System.ComponentModel.dll": {},
          "ref/netcoreapp2.2/System.Configuration.dll": {},
          "ref/netcoreapp2.2/System.Console.dll": {},
          "ref/netcoreapp2.2/System.Core.dll": {},
          "ref/netcoreapp2.2/System.Data.Common.dll": {},
          "ref/netcoreapp2.2/System.Data.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.Contracts.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.Debug.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.DiagnosticSource.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.FileVersionInfo.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.Process.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.StackTrace.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.TextWriterTraceListener.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.Tools.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.TraceSource.dll": {},
          "ref/netcoreapp2.2/System.Diagnostics.Tracing.dll": {},
          "ref/netcoreapp2.2/System.Drawing.Primitives.dll": {},
          "ref/netcoreapp2.2/System.Drawing.dll": {},
          "ref/netcoreapp2.2/System.Dynamic.Runtime.dll": {},
          "ref/netcoreapp2.2/System.Globalization.Calendars.dll": {},
          "ref/netcoreapp2.2/System.Globalization.Extensions.dll": {},
          "ref/netcoreapp2.2/System.Globalization.dll": {},
          "ref/netcoreapp2.2/System.IO.Compression.Brotli.dll": {},
          "ref/netcoreapp2.2/System.IO.Compression.FileSystem.dll": {},
          "ref/netcoreapp2.2/System.IO.Compression.ZipFile.dll": {},
          "ref/netcoreapp2.2/System.IO.Compression.dll": {},
          "ref/netcoreapp2.2/System.IO.FileSystem.DriveInfo.dll": {},
          "ref/netcoreapp2.2/System.IO.FileSystem.Primitives.dll": {},
          "ref/netcoreapp2.2/System.IO.FileSystem.Watcher.dll": {},
          "ref/netcoreapp2.2/System.IO.FileSystem.dll": {},
          "ref/netcoreapp2.2/System.IO.IsolatedStorage.dll": {},
          "ref/netcoreapp2.2/System.IO.MemoryMappedFiles.dll": {},
          "ref/netcoreapp2.2/System.IO.Pipes.dll": {},
          "ref/netcoreapp2.2/System.IO.UnmanagedMemoryStream.dll": {},
          "ref/netcoreapp2.2/System.IO.dll": {},
          "ref/netcoreapp2.2/System.Linq.Expressions.dll": {},
          "ref/netcoreapp2.2/System.Linq.Parallel.dll": {},
          "ref/netcoreapp2.2/System.Linq.Queryable.dll": {},
          "ref/netcoreapp2.2/System.Linq.dll": {},
          "ref/netcoreapp2.2/System.Memory.dll": {},
          "ref/netcoreapp2.2/System.Net.Http.dll": {},
          "ref/netcoreapp2.2/System.Net.HttpListener.dll": {},
          "ref/netcoreapp2.2/System.Net.Mail.dll": {},
          "ref/netcoreapp2.2/System.Net.NameResolution.dll": {},
          "ref/netcoreapp2.2/System.Net.NetworkInformation.dll": {},
          "ref/netcoreapp2.2/System.Net.Ping.dll": {},
          "ref/netcoreapp2.2/System.Net.Primitives.dll": {},
          "ref/netcoreapp2.2/System.Net.Requests.dll": {},
          "ref/netcoreapp2.2/System.Net.Security.dll": {},
          "ref/netcoreapp2.2/System.Net.ServicePoint.dll": {},
          "ref/netcoreapp2.2/System.Net.Sockets.dll": {},
          "ref/netcoreapp2.2/System.Net.WebClient.dll": {},
          "ref/netcoreapp2.2/System.Net.WebHeaderCollection.dll": {},
          "ref/netcoreapp2.2/System.Net.WebProxy.dll": {},
          "ref/netcoreapp2.2/System.Net.WebSockets.Client.dll": {},
          "ref/netcoreapp2.2/System.Net.WebSockets.dll": {},
          "ref/netcoreapp2.2/System.Net.dll": {},
          "ref/netcoreapp2.2/System.Numerics.Vectors.dll": {},
          "ref/netcoreapp2.2/System.Numerics.dll": {},
          "ref/netcoreapp2.2/System.ObjectModel.dll": {},
          "ref/netcoreapp2.2/System.Reflection.DispatchProxy.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Emit.ILGeneration.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Emit.Lightweight.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Emit.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Extensions.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Metadata.dll": {},
          "ref/netcoreapp2.2/System.Reflection.Primitives.dll": {},
          "ref/netcoreapp2.2/System.Reflection.TypeExtensions.dll": {},
          "ref/netcoreapp2.2/System.Reflection.dll": {},
          "ref/netcoreapp2.2/System.Resources.Reader.dll": {},
          "ref/netcoreapp2.2/System.Resources.ResourceManager.dll": {},
          "ref/netcoreapp2.2/System.Resources.Writer.dll": {},
          "ref/netcoreapp2.2/System.Runtime.CompilerServices.VisualC.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Extensions.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Handles.dll": {},
          "ref/netcoreapp2.2/System.Runtime.InteropServices.RuntimeInformation.dll": {},
          "ref/netcoreapp2.2/System.Runtime.InteropServices.WindowsRuntime.dll": {},
          "ref/netcoreapp2.2/System.Runtime.InteropServices.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Loader.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Numerics.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Serialization.Formatters.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Serialization.Json.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Serialization.Primitives.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Serialization.Xml.dll": {},
          "ref/netcoreapp2.2/System.Runtime.Serialization.dll": {},
          "ref/netcoreapp2.2/System.Runtime.dll": {},
          "ref/netcoreapp2.2/System.Security.Claims.dll": {},
          "ref/netcoreapp2.2/System.Security.Cryptography.Algorithms.dll": {},
          "ref/netcoreapp2.2/System.Security.Cryptography.Csp.dll": {},
          "ref/netcoreapp2.2/System.Security.Cryptography.Encoding.dll": {},
          "ref/netcoreapp2.2/System.Security.Cryptography.Primitives.dll": {},
          "ref/netcoreapp2.2/System.Security.Cryptography.X509Certificates.dll": {},
          "ref/netcoreapp2.2/System.Security.Principal.dll": {},
          "ref/netcoreapp2.2/System.Security.SecureString.dll": {},
          "ref/netcoreapp2.2/System.Security.dll": {},
          "ref/netcoreapp2.2/System.ServiceModel.Web.dll": {},
          "ref/netcoreapp2.2/System.ServiceProcess.dll": {},
          "ref/netcoreapp2.2/System.Text.Encoding.Extensions.dll": {},
          "ref/netcoreapp2.2/System.Text.Encoding.dll": {},
          "ref/netcoreapp2.2/System.Text.RegularExpressions.dll": {},
          "ref/netcoreapp2.2/System.Threading.Overlapped.dll": {},
          "ref/netcoreapp2.2/System.Threading.Tasks.Dataflow.dll": {},
          "ref/netcoreapp2.2/System.Threading.Tasks.Extensions.dll": {},
          "ref/netcoreapp2.2/System.Threading.Tasks.Parallel.dll": {},
          "ref/netcoreapp2.2/System.Threading.Tasks.dll": {},
          "ref/netcoreapp2.2/System.Threading.Thread.dll": {},
          "ref/netcoreapp2.2/System.Threading.ThreadPool.dll": {},
          "ref/netcoreapp2.2/System.Threading.Timer.dll": {},
          "ref/netcoreapp2.2/System.Threading.dll": {},
          "ref/netcoreapp2.2/System.Transactions.Local.dll": {},
          "ref/netcoreapp2.2/System.Transactions.dll": {},
          "ref/netcoreapp2.2/System.ValueTuple.dll": {},
          "ref/netcoreapp2.2/System.Web.HttpUtility.dll": {},
          "ref/netcoreapp2.2/System.Web.dll": {},
          "ref/netcoreapp2.2/System.Windows.dll": {},
          "ref/netcoreapp2.2/System.Xml.Linq.dll": {},
          "ref/netcoreapp2.2/System.Xml.ReaderWriter.dll": {},
          "ref/netcoreapp2.2/System.Xml.Serialization.dll": {},
          "ref/netcoreapp2.2/System.Xml.XDocument.dll": {},
          "ref/netcoreapp2.2/System.Xml.XPath.XDocument.dll": {},
          "ref/netcoreapp2.2/System.Xml.XPath.dll": {},
          "ref/netcoreapp2.2/System.Xml.XmlDocument.dll": {},
          "ref/netcoreapp2.2/System.Xml.XmlSerializer.dll": {},
          "ref/netcoreapp2.2/System.Xml.dll": {},
          "ref/netcoreapp2.2/System.dll": {},
          "ref/netcoreapp2.2/WindowsBase.dll": {},
          "ref/netcoreapp2.2/mscorlib.dll": {},
          "ref/netcoreapp2.2/netstandard.dll": {}
        },
        "compileOnly": true
      },
      "Microsoft.NETCore.DotNetAppHost/2.2.0": {
        "compileOnly": true
      },
      "Microsoft.NETCore.DotNetHostPolicy/2.2.0": {
        "dependencies": {
          "Microsoft.NETCore.DotNetHostResolver": "2.2.0"
        },
        "compileOnly": true
      },
      "Microsoft.NETCore.DotNetHostResolver/2.2.0": {
        "dependencies": {
          "Microsoft.NETCore.DotNetAppHost": "2.2.0"
        },
        "compileOnly": true
      },
      "Microsoft.NETCore.Platforms/2.2.0": {
        "compileOnly": true
      },
      "Microsoft.NETCore.Targets/2.0.0": {
        "compileOnly": true
      },
      "Microsoft.Win32.Registry/4.5.0": {
        "dependencies": {
          "System.Security.AccessControl": "4.5.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "ref/netstandard2.0/Microsoft.Win32.Registry.dll": {}
        },
        "compileOnly": true
      },
      "NETStandard.Library/2.0.3": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0"
        },
        "compileOnly": true
      },
      "Newtonsoft.Json.Bson/1.0.1": {
        "dependencies": {
          "NETStandard.Library": "2.0.3",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard1.3/Newtonsoft.Json.Bson.dll": {}
        },
        "compileOnly": true
      },
      "runtime.native.System/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0"
        },
        "compileOnly": true
      },
      "runtime.native.System.IO.Compression/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0"
        },
        "compileOnly": true
      },
      "runtime.native.System.Net.Http/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0"
        },
        "compileOnly": true
      },
      "runtime.native.System.Security.Cryptography.Apple/4.3.0": {
        "dependencies": {
          "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.Apple": "4.3.0"
        },
        "compileOnly": true
      },
      "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.Apple/4.3.0": {
        "compileOnly": true
      },
      "System.AppContext/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Buffers/4.5.0": {
        "compileOnly": true
      },
      "System.Collections/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Collections.Concurrent/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Collections.NonGeneric/4.3.0": {
        "dependencies": {
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Collections.Specialized/4.3.0": {
        "dependencies": {
          "System.Collections.NonGeneric": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Globalization.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.ComponentModel.Annotations/4.5.0": {
        "compileOnly": true
      },
      "System.Console/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.IO": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Text.Encoding": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.Contracts/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.Debug/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.DiagnosticSource/4.5.0": {
        "compileOnly": true
      },
      "System.Diagnostics.FileVersionInfo/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Reflection.Metadata": "1.6.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.StackTrace/4.3.0": {
        "dependencies": {
          "System.IO.FileSystem": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Metadata": "1.6.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.Tools/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Diagnostics.Tracing/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Dynamic.Runtime/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Linq.Expressions": "4.3.0",
          "System.ObjectModel": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Globalization/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Globalization.Calendars/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Globalization": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Globalization.Extensions/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Globalization": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0"
        },
        "compileOnly": true
      },
      "System.IdentityModel.Tokens.Jwt/5.3.0": {
        "dependencies": {
          "Microsoft.IdentityModel.JsonWebTokens": "5.3.0",
          "Microsoft.IdentityModel.Tokens": "5.3.0",
          "Newtonsoft.Json": "12.0.2"
        },
        "compile": {
          "lib/netstandard2.0/System.IdentityModel.Tokens.Jwt.dll": {}
        },
        "compileOnly": true
      },
      "System.IO/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.IO.Compression/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Buffers": "4.5.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "runtime.native.System": "4.3.0",
          "runtime.native.System.IO.Compression": "4.3.0"
        },
        "compileOnly": true
      },
      "System.IO.FileSystem/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.IO.FileSystem.Primitives/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.IO.Pipelines/4.5.2": {
        "compile": {
          "ref/netstandard1.3/System.IO.Pipelines.dll": {}
        },
        "compileOnly": true
      },
      "System.Linq/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Linq.Expressions/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Linq": "4.3.0",
          "System.ObjectModel": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Emit.Lightweight": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Memory/4.5.1": {
        "compileOnly": true
      },
      "System.Net.WebSockets.WebSocketProtocol/4.5.1": {
        "compile": {
          "ref/netstandard2.0/System.Net.WebSockets.WebSocketProtocol.dll": {}
        },
        "compileOnly": true
      },
      "System.Numerics.Vectors/4.5.0": {
        "compileOnly": true
      },
      "System.ObjectModel/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Private.DataContractSerialization/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Emit.Lightweight": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Text.Encoding.Extensions": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0",
          "System.Xml.XDocument": "4.3.0",
          "System.Xml.XmlDocument": "4.3.0",
          "System.Xml.XmlSerializer": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.IO": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.Emit/4.3.0": {
        "dependencies": {
          "System.IO": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.Emit.ILGeneration/4.3.0": {
        "dependencies": {
          "System.Reflection": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.Emit.Lightweight/4.3.0": {
        "dependencies": {
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.Extensions/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Reflection": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.Metadata/1.6.0": {
        "compileOnly": true
      },
      "System.Reflection.Primitives/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Reflection.TypeExtensions/4.3.0": {
        "dependencies": {
          "System.Reflection": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Resources.ResourceManager/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Globalization": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0"
        },
        "compileOnly": true
      },
      "System.Runtime.Extensions/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.Handles/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.InteropServices/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Handles": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.InteropServices.RuntimeInformation/4.3.0": {
        "dependencies": {
          "System.Reflection": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Threading": "4.3.0",
          "runtime.native.System": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.Numerics/4.3.0": {
        "dependencies": {
          "System.Globalization": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.Serialization.Primitives/4.3.0": {
        "dependencies": {
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Runtime.Serialization.Xml/4.3.0": {
        "dependencies": {
          "System.IO": "4.3.0",
          "System.Private.DataContractSerialization": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Serialization.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Security.AccessControl/4.5.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Security.Principal.Windows": "4.5.0"
        },
        "compile": {
          "ref/netstandard2.0/System.Security.AccessControl.dll": {}
        },
        "compileOnly": true
      },
      "System.Security.Claims/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Security.Principal": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Algorithms/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.Numerics": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "runtime.native.System.Security.Cryptography.Apple": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Cng/4.5.0": {
        "compile": {
          "ref/netcoreapp2.1/System.Security.Cryptography.Cng.dll": {}
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Csp/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.IO": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Encoding/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.Collections.Concurrent": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.OpenSsl/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.Numerics": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Pkcs/4.5.0": {
        "dependencies": {
          "System.Security.Cryptography.Cng": "4.5.0"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Primitives/4.3.0": {
        "dependencies": {
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.X509Certificates/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.Globalization.Calendars": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.Handles": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Runtime.Numerics": "4.3.0",
          "System.Security.Cryptography.Algorithms": "4.3.0",
          "System.Security.Cryptography.Cng": "4.5.0",
          "System.Security.Cryptography.Csp": "4.3.0",
          "System.Security.Cryptography.Encoding": "4.3.0",
          "System.Security.Cryptography.OpenSsl": "4.3.0",
          "System.Security.Cryptography.Primitives": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "runtime.native.System": "4.3.0",
          "runtime.native.System.Net.Http": "4.3.0",
          "runtime.native.System.Security.Cryptography.OpenSsl": "4.3.2"
        },
        "compileOnly": true
      },
      "System.Security.Cryptography.Xml/4.5.0": {
        "dependencies": {
          "System.Security.Cryptography.Pkcs": "4.5.0",
          "System.Security.Permissions": "4.5.0"
        },
        "compile": {
          "ref/netstandard2.0/System.Security.Cryptography.Xml.dll": {}
        },
        "compileOnly": true
      },
      "System.Security.Permissions/4.5.0": {
        "dependencies": {
          "System.Security.AccessControl": "4.5.0"
        },
        "compile": {
          "ref/netstandard2.0/System.Security.Permissions.dll": {}
        },
        "compileOnly": true
      },
      "System.Security.Principal/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Security.Principal.Windows/4.5.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0"
        },
        "compile": {
          "ref/netstandard2.0/System.Security.Principal.Windows.dll": {}
        },
        "compileOnly": true
      },
      "System.Text.Encoding/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Text.Encoding.Extensions/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0",
          "System.Text.Encoding": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Text.Encodings.Web/4.5.0": {
        "compile": {
          "lib/netstandard2.0/System.Text.Encodings.Web.dll": {}
        },
        "compileOnly": true
      },
      "System.Text.RegularExpressions/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Threading/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Threading.Channels/4.5.0": {
        "compile": {
          "lib/netcoreapp2.1/System.Threading.Channels.dll": {}
        },
        "compileOnly": true
      },
      "System.Threading.Tasks/4.3.0": {
        "dependencies": {
          "Microsoft.NETCore.Platforms": "2.2.0",
          "Microsoft.NETCore.Targets": "2.0.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Threading.Tasks.Extensions/4.5.1": {
        "compileOnly": true
      },
      "System.Threading.Tasks.Parallel/4.3.0": {
        "dependencies": {
          "System.Collections.Concurrent": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tracing": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Threading.Tasks": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Threading.Thread/4.3.0": {
        "dependencies": {
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.ValueTuple/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Xml.ReaderWriter/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.IO.FileSystem": "4.3.0",
          "System.IO.FileSystem.Primitives": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Runtime.InteropServices": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Text.Encoding.Extensions": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading.Tasks": "4.3.0",
          "System.Threading.Tasks.Extensions": "4.5.1"
        },
        "compileOnly": true
      },
      "System.Xml.XDocument/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Diagnostics.Tools": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Xml.XmlDocument/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Text.Encoding": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Xml.XmlSerializer/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Reflection": "4.3.0",
          "System.Reflection.Emit": "4.3.0",
          "System.Reflection.Emit.ILGeneration": "4.3.0",
          "System.Reflection.Extensions": "4.3.0",
          "System.Reflection.Primitives": "4.3.0",
          "System.Reflection.TypeExtensions": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Text.RegularExpressions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0",
          "System.Xml.XmlDocument": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Xml.XPath/4.3.0": {
        "dependencies": {
          "System.Collections": "4.3.0",
          "System.Diagnostics.Debug": "4.3.0",
          "System.Globalization": "4.3.0",
          "System.IO": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0"
        },
        "compileOnly": true
      },
      "System.Xml.XPath.XDocument/4.3.0": {
        "dependencies": {
          "System.Diagnostics.Debug": "4.3.0",
          "System.Linq": "4.3.0",
          "System.Resources.ResourceManager": "4.3.0",
          "System.Runtime": "4.3.0",
          "System.Runtime.Extensions": "4.3.0",
          "System.Threading": "4.3.0",
          "System.Xml.ReaderWriter": "4.3.0",
          "System.Xml.XDocument": "4.3.0",
          "System.Xml.XPath": "4.3.0"
        },
        "compileOnly": true
      }
    }
  },
  "libraries": {
    "MemberPlatform.Api/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "AngleSharp/0.9.11": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9+4CpMbQPNjpcaeeqMIKXfqfZ6FsDBaiYO/hws+p6l5zS2kcvUDImutew/jxCiORLofgRgDi3MB9WbOPYyMujA==",
      "path": "anglesharp/0.9.11",
      "hashPath": "anglesharp.0.9.11.nupkg.sha512"
    },
    "AutoMapper/9.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xCqvoxT4HIrNY/xlXG9W+BA/awdrhWvMTKTK/igkGSRbhOhpl3Q8O8Gxlhzjc9JsYqE7sS6AxgyuUUvZ6R5/Bw==",
      "path": "automapper/9.0.0",
      "hashPath": "automapper.9.0.0.nupkg.sha512"
    },
    "AutoMapper.Extensions.Microsoft.DependencyInjection/7.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-szI4yeRIM7GWe9JyekW0dKYehPB0t6M+I55fPeCebN6PhS7zQZa0eG3bgOnOx+eP3caSNoE7KEJs2rk7MLsh8w==",
      "path": "automapper.extensions.microsoft.dependencyinjection/7.0.0",
      "hashPath": "automapper.extensions.microsoft.dependencyinjection.7.0.0.nupkg.sha512"
    },
    "Dapper/2.0.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-DrNPDqK3nR4RpwAPX2R0R34YhlT+k5Fbw7UnqxQ2aH+wSWvEIj+fCDXB8RJWgw6fGFZLavq7nuJ6J/eAqSrIpw==",
      "path": "dapper/2.0.4",
      "hashPath": "dapper.2.0.4.nupkg.sha512"
    },
    "DocumentFormat.OpenXml/2.9.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Z3d9A++hVZfwmOme1f3npTGNFwNPjrkrsbxwfPpDo9siwMeNhwceesyk0nFQ5gamR8wT5Tp3FgVBsASTnFO0mQ==",
      "path": "documentformat.openxml/2.9.1",
      "hashPath": "documentformat.openxml.2.9.1.nupkg.sha512"
    },
    "EFCore.BulkExtensions/2.6.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-lQCuLnvqhEkGbf/mrYPbu+zEtIK0VK4zPXC0tKHBUc+paKNoIL5C2ucj+fyd3P8UQ4lPEElV1i7hb7C9wGRTGg==",
      "path": "efcore.bulkextensions/2.6.0",
      "hashPath": "efcore.bulkextensions.2.6.0.nupkg.sha512"
    },
    "Elastique.DapperProvider/1.0.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-zRl9SAzg8jrXcKZGRPzK+o67C2Q+GyjIvvLhQGvSspz8plAqCRhLzzDGjzSxoq52rG5CXw8yRTsHiKng8zp6ug==",
      "path": "elastique.dapperprovider/1.0.2",
      "hashPath": "elastique.dapperprovider.1.0.2.nupkg.sha512"
    },
    "Elastique.StatusLibrary/1.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-p2OD9+z3twaZuUoO/j/hqNkZcINs7K+YloWg0lsC4PxSvj1lm0u6TTgzEooLp7ajzl59skodij6efK3Lur6DmQ==",
      "path": "elastique.statuslibrary/1.0.1",
      "hashPath": "elastique.statuslibrary.1.0.1.nupkg.sha512"
    },
    "FastMember/1.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Db80coUbys8VgmcrR9mxUR+EInTpweoLM0GFJXIBuj8+PTgqV+ygq+EOM3HME83tob1+RR9XlBcEkpfFyypdHg==",
      "path": "fastmember/1.5.0",
      "hashPath": "fastmember.1.5.0.nupkg.sha512"
    },
    "HtmlSanitizer/4.0.217": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-QJON8mKjhTxWM+gbkYZ88Q5/gN3xxGNj8JHXij6Btkm16LivupWitdLP30XSH4iDIFVKNrvC0eYULPfiP0ZVlw==",
      "path": "htmlsanitizer/4.0.217",
      "hashPath": "htmlsanitizer.4.0.217.nupkg.sha512"
    },
    "LinqKit.Microsoft.EntityFrameworkCore/1.1.16": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-CI6wcf1C6vXp3rGwp7brpxBJjVS/vDoHswmX55Rhe13VdhEunHAI7iDCmxMgAYO7s6rarYTjWkNMu6UU66YVdA==",
      "path": "linqkit.microsoft.entityframeworkcore/1.1.16",
      "hashPath": "linqkit.microsoft.entityframeworkcore.1.1.16.nupkg.sha512"
    },
    "MailKit/2.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-+B+d8jJWx2nmypSiTqul/pKGRYbQ49SFSSTUOIIIXG4BlrMDs7WPRW/F7En+hjJAmz4eoFxyp7oFgfTkDUL0OA==",
      "path": "mailkit/2.3.0",
      "hashPath": "mailkit.2.3.0.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qwWKmevApRZmNUqSTiISgcGXkxnS7pxJGiqPkQT2LbrWKQr/CqjbIgzuXXkyHhvOfY39Cw7ZLmIltKbTmIFzig==",
      "path": "microsoft.applicationinsights/2.10.0",
      "hashPath": "microsoft.applicationinsights.2.10.0.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights.AspNetCore/2.7.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-XhFAwdhpmWG8upMg6ayvERlBYWzTzHcBZ0yYLk+Rs0OoOWH2RABov000Sbr2XNt0fpTPVUapjVMUv7VbODNE0g==",
      "path": "microsoft.applicationinsights.aspnetcore/2.7.1",
      "hashPath": "microsoft.applicationinsights.aspnetcore.2.7.1.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights.DependencyCollector/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-m9N41b6CuBclBPRuCu8UA1J7WPmzcU00wl1kgAxZIembrvxYAZXfuY9CUX8PfWMxr5zslRgy5QPjHo7XK02PXw==",
      "path": "microsoft.applicationinsights.dependencycollector/2.10.0",
      "hashPath": "microsoft.applicationinsights.dependencycollector.2.10.0.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights.PerfCounterCollector/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9Pr7OLAEcdt/V2xXrLH6Bob/CivRYEAiIxMI90myPna70JBN8uOLSC0xENVFdSV/YibAnt2iXhfZHzQ2+7u/sA==",
      "path": "microsoft.applicationinsights.perfcountercollector/2.10.0",
      "hashPath": "microsoft.applicationinsights.perfcountercollector.2.10.0.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights.WindowsServer/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oX6ROAts2RmrBmtDyFt+8jdE5FIWW3EkANp1wnIpEhuwbnIXhrLNuRte2cQ+PMh9/DpD9oK7KiOn6GfyuZ3kAA==",
      "path": "microsoft.applicationinsights.windowsserver/2.10.0",
      "hashPath": "microsoft.applicationinsights.windowsserver.2.10.0.nupkg.sha512"
    },
    "Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZvpYOpWc5VySLoOJEMaXd6C+ka6aYG9Qn8Mso58SiydZO6nJV3DTNU1I3W8dZtSHUq2oLECA+yarUVt030SxEw==",
      "path": "microsoft.applicationinsights.windowsserver.telemetrychannel/2.10.0",
      "hashPath": "microsoft.applicationinsights.windowsserver.telemetrychannel.2.10.0.nupkg.sha512"
    },
    "Microsoft.Azure.KeyVault/3.0.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-WyNaquOQ1Czl7FsB2rxXoCq+Hywmlc8lTmiu6jtGfyS6NszKzV0uGRcAF1KC75YRB0tD8Yx6WXdwlGPpbB73QQ==",
      "path": "microsoft.azure.keyvault/3.0.4",
      "hashPath": "microsoft.azure.keyvault.3.0.4.nupkg.sha512"
    },
    "Microsoft.Azure.KeyVault.WebKey/3.0.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-2D4+DRdmGYsfsE+n8pY7NVKnMG60HRsinjQN62KpAa6qNeJ66daVGJhnhVcSqT1l78+u8SduaDMiS7OgYOS0tQ==",
      "path": "microsoft.azure.keyvault.webkey/3.0.4",
      "hashPath": "microsoft.azure.keyvault.webkey.3.0.4.nupkg.sha512"
    },
    "Microsoft.Azure.Services.AppAuthentication/1.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oSVQ+YLgn0PfDLBRwOtHrkw06Pc6HvqIVrtFFWMg/1eafY+Hbrm1CSXnzXKXJwqUUv0ynvuwqzjN+IFR3g588w==",
      "path": "microsoft.azure.services.appauthentication/1.0.1",
      "hashPath": "microsoft.azure.services.appauthentication.1.0.1.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.CSharp.Workspaces/2.8.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-EJWaxi2bI47iEZen/nZkJEDZCrP9Oj3PJtMwBv34Z0ZvvdSkpgsdqlHSud8d5vC53LnCXLfBLewfqHcILDVSDw==",
      "path": "microsoft.codeanalysis.csharp.workspaces/2.8.0",
      "hashPath": "microsoft.codeanalysis.csharp.workspaces.2.8.0.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.Workspaces.Common/2.8.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-tJlJ99SD8bHBAXShOG/pXQ1K118cnsF01obEf9aAtdgLbw3yEPahZ7qvWeGMjrheUhvOsSkv/wTKYg9euKa8MQ==",
      "path": "microsoft.codeanalysis.workspaces.common/2.8.0",
      "hashPath": "microsoft.codeanalysis.workspaces.common.2.8.0.nupkg.sha512"
    },
    "Microsoft.Data.Sqlite.Core/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-JuHLktt4mb5jSTKr+lq4OeuG8tUcCdk/9Gg9kX955+BNcIpSrDqLTcktBOyOIWWUrqBp1zjEbd+/u/9rF57H1w==",
      "path": "microsoft.data.sqlite.core/2.2.6",
      "hashPath": "microsoft.data.sqlite.core.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-vmrMrjiCO7FkuFJwt/MHl6pk6cSXPtB1miPMtn0KTO7WqwaQ2WQ4gWpC/m753PzVriH2X2kIadWrd9SJb7KVww==",
      "path": "microsoft.entityframeworkcore/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Abstractions/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-4CrelpMMLszWqi0gFEYPhbsinXCQ2Vw6rA190IIwKY7THge/ckOwj6QIQKOu3Lmxj4khpzs+b6QADbpRRnOIaQ==",
      "path": "microsoft.entityframeworkcore.abstractions/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.abstractions.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Analyzers/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-BpNllkfZABCvMAaIL/pSLxTYYZQqiVesSG6xQpvelrlEfC0s9PS217Sq5Apn/zYW8ALtGoVEY12TblHrZ4SRRA==",
      "path": "microsoft.entityframeworkcore.analyzers/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.analyzers.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.InMemory/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-PREu4MsAGz3NEheW4TX4e8Bl4l34OrA4R4KFyWb3r475QRS1/ul4vK1mUzWF/fEwJgx7n1h1tvNZW7I04sl5fw==",
      "path": "microsoft.entityframeworkcore.inmemory/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.inmemory.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Relational/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-AlB4Gg1nplg6fwieCLixphmYkhwM1SSHecE43oDizAjFUSs7vlL5mlwf620X4SB037pVg+naxhBEtF53TGa6yQ==",
      "path": "microsoft.entityframeworkcore.relational/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.relational.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Sqlite/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-U4ifHn7YJsj31waXLbXwR/HMdQnp/v+bBGw5VfSRqVtO10npQa3LBOXWZTaocXgJBoIX0ADI2ZkfjX+hb0CnJA==",
      "path": "microsoft.entityframeworkcore.sqlite/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.sqlite.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Sqlite.Core/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FRwNvIjcoDaMGExrrGb02ULgY0dhkk70OuY7Bc61wK4rOVHweLiCAaJJ3y3s9w604lM1lBcTWmEilFincRJRhw==",
      "path": "microsoft.entityframeworkcore.sqlite.core/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.sqlite.core.2.2.6.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.SqlServer/2.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-rNnCA7ntlpITeYl1p3lAeS0kyHhETlanTghfpoDgyGIoxyUm2VsI2eyzL6EMYrqWnhAOwx3HzP0/50XRM/0CHw==",
      "path": "microsoft.entityframeworkcore.sqlserver/2.2.6",
      "hashPath": "microsoft.entityframeworkcore.sqlserver.2.2.6.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.AzureKeyVault/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-aTt0fNYOVehKBrflWUBDZFTV51K986Y4LOIIEjyhk3LJ+VEwajJijKLNy5fsfddWUUDRxkNRr5gIFeUAV8OaaQ==",
      "path": "microsoft.extensions.configuration.azurekeyvault/2.2.0",
      "hashPath": "microsoft.extensions.configuration.azurekeyvault.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.Binder/2.2.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TNTCeQwiVsBc9p4C+xPG+CmdaLVAwp+o9pHqYxtlTcXIA8ma7hCqTrXIjn7tPUEGufNEQ48sH1mfjO08jLjMdQ==",
      "path": "microsoft.extensions.configuration.binder/2.2.4",
      "hashPath": "microsoft.extensions.configuration.binder.2.2.4.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.EnvironmentVariables/2.2.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Os7uRhp9xwawY5w2tgXw/86YrmAJZl6aHiQVdS9boWybTWPkJOvXXrQ3AGwuldN1W/r+cfnwRe2ePGeFO4zlzg==",
      "path": "microsoft.extensions.configuration.environmentvariables/2.2.4",
      "hashPath": "microsoft.extensions.configuration.environmentvariables.2.2.4.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.ApplicationInsights/2.10.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-22Se64sxtqhpH1RlhbyfuQ1EfIy92nNu/ctzlsO5sb4b3l856YzO+2Ng7xFsQKxPv9oOi7FP++xLMNK6VVs/NA==",
      "path": "microsoft.extensions.logging.applicationinsights/2.10.0",
      "hashPath": "microsoft.extensions.logging.applicationinsights.2.10.0.nupkg.sha512"
    },
    "Microsoft.Extensions.PlatformAbstractions/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-H6ZsQzxYw/6k2DfEQRXdC+vQ6obd6Uba3uGJrnJ2vG4PRXjQZ7seB13JdCfE72abp8E6Fk3gGgDzfJiLZi5ZpQ==",
      "path": "microsoft.extensions.platformabstractions/1.1.0",
      "hashPath": "microsoft.extensions.platformabstractions.1.1.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Clients.ActiveDirectory/3.14.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TNsJJMiRnkeby1ovThVoV9yFsPWjAdluwOA+Nf0LtSsBVVrKQv8Qp4kYOgyNwMVj+pDwbhXISySk+4HyHVWNZQ==",
      "path": "microsoft.identitymodel.clients.activedirectory/3.14.2",
      "hashPath": "microsoft.identitymodel.clients.activedirectory.3.14.2.nupkg.sha512"
    },
    "Microsoft.Rest.ClientRuntime/2.3.20": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-bw/H1nO4JdnhTagPHWIFQwtlQ6rb2jqw5RTrqPsPqzrjhJxc7P6MyNGdf4pgHQdzdpBSNOfZTEQifoUkxmzYXQ==",
      "path": "microsoft.rest.clientruntime/2.3.20",
      "hashPath": "microsoft.rest.clientruntime.2.3.20.nupkg.sha512"
    },
    "Microsoft.Rest.ClientRuntime.Azure/3.3.18": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-pCtem10PRQYvzRiwJVInsccsqB0NrTjW83NF3zWk1LpN3IS0AneZKq89RyogDT7mRMT1Li/mLY8N8kU6RAiK0g==",
      "path": "microsoft.rest.clientruntime.azure/3.3.18",
      "hashPath": "microsoft.rest.clientruntime.azure.3.3.18.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-wc71c9HWTeXy9/w9O4Yr2LKmdJjVyIoJ/XQX8/6uma4EAVU25RLtUWlvhA0gpgFw9Kf1TkCv70x+CbKnRw/d8Q==",
      "path": "microsoft.visualstudio.web.codegeneration/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.Contracts/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-wXlpxDfRD5aPypa0p0UE97tkRQvAz9D9FfA2GITzr+LlGIpybyGnxkwGVp0Vha1Ibr0kJG0HdnqfeHME/WuAcQ==",
      "path": "microsoft.visualstudio.web.codegeneration.contracts/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.contracts.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.Core/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-APdPavBUYcGPBaW4rjxBVRePWmg0ZzhIRymOzvLFWUtzfvJKw1+8PaCzsH7Uvl+felm0L1UVQwBx1Do0R7j7Xg==",
      "path": "microsoft.visualstudio.web.codegeneration.core/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.core.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.Design/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xH50cYOU+infRq4KikBuu2qeXcwW4tE0D5TDfKLuLrEtDm90aXI+0qygPsqyISf+lOW7L7rQ64BH/dRYkK3c3Q==",
      "path": "microsoft.visualstudio.web.codegeneration.design/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.design.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.EntityFrameworkCore/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-N9S7TeFZjXzNY9OVbz4OFw9cM9oEeMaCnuLFhetNioy/wPwZbgglrctAEYxfDbvocQ17YCAVR2EMRbYHNDHyVg==",
      "path": "microsoft.visualstudio.web.codegeneration.entityframeworkcore/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.entityframeworkcore.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.Templating/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-sW2lHnOoL1xFnSm/2zSedeUoQPlbhPfWjSuUYsxYUj/N5QmLmH98ZLaqP26k6Om/heR6Gux/veXI96yM1Parow==",
      "path": "microsoft.visualstudio.web.codegeneration.templating/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.templating.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGeneration.Utils/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-/r/y+XpOpbCwN/M/HopjfGTDRlmixTd4G6HG6FaBkD/YF3T1u+4WMRVtuB6zz7aw571HmX+6UokEa6HJSwkPDA==",
      "path": "microsoft.visualstudio.web.codegeneration.utils/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegeneration.utils.2.2.3.nupkg.sha512"
    },
    "Microsoft.VisualStudio.Web.CodeGenerators.Mvc/2.2.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-0gVuA4KUCHFM4M/9SjG+7j7BzZ7SW/BufF97Q78i2VV8JBbQXc/5Rf6YUG1VGW2fwSEOl9+S26utEGS+86GGGw==",
      "path": "microsoft.visualstudio.web.codegenerators.mvc/2.2.3",
      "hashPath": "microsoft.visualstudio.web.codegenerators.mvc.2.2.3.nupkg.sha512"
    },
    "Microsoft.Win32.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9ZQKCWxH7Ijp9BfahvL2Zyf1cJIk8XYLF6Yjzr2yi0b2cOut/HQ31qf1ThHAgCc3WiZMdnWcfJCgN82/0UunxA==",
      "path": "microsoft.win32.primitives/4.3.0",
      "hashPath": "microsoft.win32.primitives.4.3.0.nupkg.sha512"
    },
    "Microsoft.Win32.SystemEvents/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LuI1oG+24TUj1ZRQQjM5Ew73BKnZE5NZ/7eAdh1o8ST5dPhUnJvIkiIn2re3MwnkRy6ELRnvEbBxHP8uALKhJw==",
      "path": "microsoft.win32.systemevents/4.5.0",
      "hashPath": "microsoft.win32.systemevents.4.5.0.nupkg.sha512"
    },
    "MimeKit/2.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-BDokp2b2OBvhK423LQmGYMsA7K8jW6FCgv/waNUx/FbIB5KdvmgfGUCcQDF9VRQnVQ88Ikst7u3du2hStg3jlg==",
      "path": "mimekit/2.3.0",
      "hashPath": "mimekit.2.3.0.nupkg.sha512"
    },
    "Nager.Country/1.0.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LCJYz2N2qpKVp07dFqp2EtNDGQO1058OCnhEm2HlCkgIdGRioqKqLMQ1P73f+GwSJS4SlkVGq/Pz6859eCQmsA==",
      "path": "nager.country/1.0.4",
      "hashPath": "nager.country.1.0.4.nupkg.sha512"
    },
    "Newtonsoft.Json/12.0.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-rTK0s2EKlfHsQsH6Yx2smvcTCeyoDNgCW7FEYyV01drPlh2T243PR2DiDXqtC5N4GDm4Ma/lkxfW5a/4793vbA==",
      "path": "newtonsoft.json/12.0.2",
      "hashPath": "newtonsoft.json.12.0.2.nupkg.sha512"
    },
    "NuGet.Frameworks/4.7.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qbXaB76XYUVLocLBs8Z9TS/ERGK2wm797feO+0JEPFvT7o7MRadOR77mqaSD4J1k8G+DlZQyq+MlkCuxrkr3ag==",
      "path": "nuget.frameworks/4.7.0",
      "hashPath": "nuget.frameworks.4.7.0.nupkg.sha512"
    },
    "Pipelines.Sockets.Unofficial/2.0.22": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-KBrLHtOjnSSgVtNLA3FPxR8+/7+NOjo3CDtUKDKlvFvGiE+x82pEzOgf4J/dRePlTVCaLQzfjssqom6GegNiPg==",
      "path": "pipelines.sockets.unofficial/2.0.22",
      "hashPath": "pipelines.sockets.unofficial.2.0.22.nupkg.sha512"
    },
    "Portable.BouncyCastle/1.8.5": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-EaCgmntbH1sOzemRTqyXSqYjB6pLH7VCYHhhDYZ59guHSD5qPwhIYa7kfy0QUlmTRt9IXhaXdFhNuBUArp70Ng==",
      "path": "portable.bouncycastle/1.8.5",
      "hashPath": "portable.bouncycastle.1.8.5.nupkg.sha512"
    },
    "protobuf-net/2.4.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-j37MD1p1s9NdX8P5+IaY2J9p2382xiL1VP3mxYu0g+G/kf2YM2grFa1jJPO+0WDJNl1XhNPO0Q5yBEcbX77hBQ==",
      "path": "protobuf-net/2.4.0",
      "hashPath": "protobuf-net.2.4.0.nupkg.sha512"
    },
    "Remotion.Linq/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-fK/76UmpC0FXBlGDFVPLJHQlDLYnGC+XY3eoDgCgbtrhi0vzbXDQ3n/IYHhqSKqXQfGw/u04A1drWs7rFVkRjw==",
      "path": "remotion.linq/2.2.0",
      "hashPath": "remotion.linq.2.2.0.nupkg.sha512"
    },
    "runtime.debian.8-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7VSGO0URRKoMEAq0Sc9cRz8mb6zbyx/BZDEWhgPdzzpmFhkam3fJ1DAGWFXBI4nGlma+uPKpfuMQP5LXRnOH5g==",
      "path": "runtime.debian.8-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.debian.8-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.fedora.23-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-0oAaTAm6e2oVH+/Zttt0cuhGaePQYKII1dY8iaqP7CvOpVKgLybKRFvQjXR2LtxXOXTVPNv14j0ot8uV+HrUmw==",
      "path": "runtime.fedora.23-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.fedora.23-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.fedora.24-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-G24ibsCNi5Kbz0oXWynBoRgtGvsw5ZSVEWjv13/KiCAM8C6wz9zzcCniMeQFIkJ2tasjo2kXlvlBZhplL51kGg==",
      "path": "runtime.fedora.24-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.fedora.24-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.native.System.Data.SqlClient.sni/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-AJfX7owAAkMjWQYhoml5IBfXh8UyYPjktn8pK0BFGAdKgBS7HqMz1fw5vdzfZUWfhtTPDGCjgNttt46ZyEmSjg==",
      "path": "runtime.native.system.data.sqlclient.sni/4.5.0",
      "hashPath": "runtime.native.system.data.sqlclient.sni.4.5.0.nupkg.sha512"
    },
    "runtime.native.System.Net.Security/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-M2nN92ePS8BgQ2oi6Jj3PlTUzadYSIWLdZrHY1n1ZcW9o4wAQQ6W+aQ2lfq1ysZQfVCgDwY58alUdowrzezztg==",
      "path": "runtime.native.system.net.security/4.3.0",
      "hashPath": "runtime.native.system.net.security.4.3.0.nupkg.sha512"
    },
    "runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-QR1OwtwehHxSeQvZKXe+iSd+d3XZNkEcuWMFYa2i0aG1l+lR739HPicKMlTbJst3spmeekDVBUS7SeS26s4U/g==",
      "path": "runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.opensuse.13.2-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-I+GNKGg2xCHueRd1m9PzeEW7WLbNNLznmTuEi8/vZX71HudUbx1UTwlGkiwMri7JLl8hGaIAWnA/GONhu+LOyQ==",
      "path": "runtime.opensuse.13.2-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.opensuse.13.2-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.opensuse.42.1-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-1Z3TAq1ytS1IBRtPXJvEUZdVsfWfeNEhBkbiOCGEl9wwAfsjP2lz3ZFDx5tq8p60/EqbS0HItG5piHuB71RjoA==",
      "path": "runtime.opensuse.42.1-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.opensuse.42.1-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-6mU/cVmmHtQiDXhnzUImxIcDL48GbTk+TsptXyJA+MIOG9LRjPoAQC/qBFB7X+UNyK86bmvGwC8t+M66wsYC8w==",
      "path": "runtime.osx.10.10-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.osx.10.10-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.rhel.7-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-vjwG0GGcTW/PPg6KVud8F9GLWYuAV1rrw1BKAqY0oh4jcUqg15oYF1+qkGR2x2ZHM4DQnWKQ7cJgYbfncz/lYg==",
      "path": "runtime.rhel.7-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.rhel.7-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.ubuntu.14.04-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7KMFpTkHC/zoExs+PwP8jDCWcrK9H6L7soowT80CUx3e+nxP/AFnq0AQAW5W76z2WYbLAYCRyPfwYFG6zkvQRw==",
      "path": "runtime.ubuntu.14.04-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.ubuntu.14.04-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.ubuntu.16.04-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xrlmRCnKZJLHxyyLIqkZjNXqgxnKdZxfItrPkjI+6pkRo5lHX8YvSZlWrSI5AVwLMi4HbNWP7064hcAWeZKp5w==",
      "path": "runtime.ubuntu.16.04-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.ubuntu.16.04-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.ubuntu.16.10-x64.runtime.native.System.Security.Cryptography.OpenSsl/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-leXiwfiIkW7Gmn7cgnNcdtNAU70SjmKW3jxGj1iKHOvdn0zRWsgv/l2OJUO5zdGdiv2VRFnAsxxhDgMzofPdWg==",
      "path": "runtime.ubuntu.16.10-x64.runtime.native.system.security.cryptography.openssl/4.3.2",
      "hashPath": "runtime.ubuntu.16.10-x64.runtime.native.system.security.cryptography.openssl.4.3.2.nupkg.sha512"
    },
    "runtime.win-arm64.runtime.native.System.Data.SqlClient.sni/4.4.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LbrynESTp3bm5O/+jGL8v0Qg5SJlTV08lpIpFesXjF6uGNMWqFnUQbYBJwZTeua6E/Y7FIM1C54Ey1btLWupdg==",
      "path": "runtime.win-arm64.runtime.native.system.data.sqlclient.sni/4.4.0",
      "hashPath": "runtime.win-arm64.runtime.native.system.data.sqlclient.sni.4.4.0.nupkg.sha512"
    },
    "runtime.win-x64.runtime.native.System.Data.SqlClient.sni/4.4.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-38ugOfkYJqJoX9g6EYRlZB5U2ZJH51UP8ptxZgdpS07FgOEToV+lS11ouNK2PM12Pr6X/PpT5jK82G3DwH/SxQ==",
      "path": "runtime.win-x64.runtime.native.system.data.sqlclient.sni/4.4.0",
      "hashPath": "runtime.win-x64.runtime.native.system.data.sqlclient.sni.4.4.0.nupkg.sha512"
    },
    "runtime.win-x86.runtime.native.System.Data.SqlClient.sni/4.4.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-YhEdSQUsTx+C8m8Bw7ar5/VesXvCFMItyZF7G1AUY+OM0VPZUOeAVpJ4Wl6fydBGUYZxojTDR3I6Bj/+BPkJNA==",
      "path": "runtime.win-x86.runtime.native.system.data.sqlclient.sni/4.4.0",
      "hashPath": "runtime.win-x86.runtime.native.system.data.sqlclient.sni.4.4.0.nupkg.sha512"
    },
    "SharpZipLib/1.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-zvWa/L02JHNatdtjya6Swpudb2YEHaOLHL1eRrqpjm71iGRNUNONO5adUF/9CHbSJbzhELW1UoH4NGy7n7+3bQ==",
      "path": "sharpziplib/1.2.0",
      "hashPath": "sharpziplib.1.2.0.nupkg.sha512"
    },
    "SQLitePCLRaw.bundle_green/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-U5lZv+E0JBCG5uQngaRgyIAlbzIwRczb0m46XJfLGXovtfVOaMNRY/oGyKAJjdexVrfqLRd+JyopGMySpAZRGQ==",
      "path": "sqlitepclraw.bundle_green/1.1.12",
      "hashPath": "sqlitepclraw.bundle_green.1.1.12.nupkg.sha512"
    },
    "SQLitePCLRaw.core/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-S4hr+tE275ran5jyKFW/FYPG6Bz6nsHUp9H8chqKxzk21PxJadLd9LnvLe6LMRP/IqY5+LOIIDQF3m/2iDlZ7Q==",
      "path": "sqlitepclraw.core/1.1.12",
      "hashPath": "sqlitepclraw.core.1.1.12.nupkg.sha512"
    },
    "SQLitePCLRaw.lib.e_sqlite3.linux/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Tn/YE1VxWtHa4MQ9KC//ptIw6cLAUh+xXSdpX7MyGINmb4/igqyx0IOEq5WeH/+cuI/EnDtdWAOn98eMQnKsTQ==",
      "path": "sqlitepclraw.lib.e_sqlite3.linux/1.1.12",
      "hashPath": "sqlitepclraw.lib.e_sqlite3.linux.1.1.12.nupkg.sha512"
    },
    "SQLitePCLRaw.lib.e_sqlite3.osx/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qfl1ljn6NOQDyM2i9JDZc6xekHoC+Fqe4GzuhWFCS6siI7lLInw09HHSZRqyimV36vjdQYnyBBFKSn53rSOYkA==",
      "path": "sqlitepclraw.lib.e_sqlite3.osx/1.1.12",
      "hashPath": "sqlitepclraw.lib.e_sqlite3.osx.1.1.12.nupkg.sha512"
    },
    "SQLitePCLRaw.lib.e_sqlite3.v110_xp/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-YfmaVhcEyAGU6BZ7NQiYYfCHKsCYjldwsafiFKArzqeM8MHuhfqft1Fjdv7ncukXrvKsHXhCrzJzKEMwPXiSSg==",
      "path": "sqlitepclraw.lib.e_sqlite3.v110_xp/1.1.12",
      "hashPath": "sqlitepclraw.lib.e_sqlite3.v110_xp.1.1.12.nupkg.sha512"
    },
    "SQLitePCLRaw.provider.e_sqlite3.netstandard11/1.1.12": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qjz6Ad1Q5hiI8imCiG5Mpa/w8E8+rAk3SRJdX54uEOo5nPywiN1H0jmMZO+ID0nPibQA3yjlAHt5/GcLW9Iftg==",
      "path": "sqlitepclraw.provider.e_sqlite3.netstandard11/1.1.12",
      "hashPath": "sqlitepclraw.provider.e_sqlite3.netstandard11.1.1.12.nupkg.sha512"
    },
    "StackExchange.Redis/2.0.601": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-aEZQui10BTYL787+GCRlcT5qZe4ioyU2NKIO20ppDFbWE9bUuQLIq2FWQyvVcD+e/a88JFusT77KaQ8C7UDP9g==",
      "path": "stackexchange.redis/2.0.601",
      "hashPath": "stackexchange.redis.2.0.601.nupkg.sha512"
    },
    "Swashbuckle.AspNetCore/4.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Zq6lkFieuFPNDwXwQ+e8i5zy2VMrexcRFU8mQORxqIc8r7Y+qKX63vg57yL1HeGCINHQGGzxGfw2rP63IeEqhg==",
      "path": "swashbuckle.aspnetcore/4.0.1",
      "hashPath": "swashbuckle.aspnetcore.4.0.1.nupkg.sha512"
    },
    "Swashbuckle.AspNetCore.Swagger/4.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-rqzS3vHrjz+tR5j0nZOKZyaMTDfLGbVYkwMq205aYuGbsiGwbOlNU0Q8lq4Q0ptQPMKVkUf8XouCIdJ3qpK17w==",
      "path": "swashbuckle.aspnetcore.swagger/4.0.1",
      "hashPath": "swashbuckle.aspnetcore.swagger.4.0.1.nupkg.sha512"
    },
    "Swashbuckle.AspNetCore.SwaggerGen/4.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ztAj0T1U+2AqQNA8b+nr8yVkDW9XzNaAfez6d1jO13sdn2A/JW5Syn9TThsakrHxYNLt6y6aQCXbyBfQXpcQwA==",
      "path": "swashbuckle.aspnetcore.swaggergen/4.0.1",
      "hashPath": "swashbuckle.aspnetcore.swaggergen.4.0.1.nupkg.sha512"
    },
    "Swashbuckle.AspNetCore.SwaggerUI/4.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-d2U7NyV0e4UhyCzAVK9QHm0iz2QoVPUa9XzJ/Gr0rn/jBZWFpVLvigKv0vxFzO2E793sY605+4h885gvCdKSxQ==",
      "path": "swashbuckle.aspnetcore.swaggerui/4.0.1",
      "hashPath": "swashbuckle.aspnetcore.swaggerui.4.0.1.nupkg.sha512"
    },
    "System.Collections.Immutable/1.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-EXKiDFsChZW0RjrZ4FYHu9aW6+P4MCgEDCklsVseRfhoO0F+dXeMSsMRAlVXIo06kGJ/zv+2w1a2uc2+kxxSaQ==",
      "path": "system.collections.immutable/1.5.0",
      "hashPath": "system.collections.immutable.1.5.0.nupkg.sha512"
    },
    "System.Composition/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-I+D26qpYdoklyAVUdqwUBrEIckMNjAYnuPJy/h9dsQItpQwVREkDFs4b4tkBza0kT2Yk48Lcfsv2QQ9hWsh9Iw==",
      "path": "system.composition/1.0.31",
      "hashPath": "system.composition.1.0.31.nupkg.sha512"
    },
    "System.Composition.AttributedModel/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-NHWhkM3ZkspmA0XJEsKdtTt1ViDYuojgSND3yHhTzwxepiwqZf+BCWuvCbjUt4fe0NxxQhUDGJ5km6sLjo9qnQ==",
      "path": "system.composition.attributedmodel/1.0.31",
      "hashPath": "system.composition.attributedmodel.1.0.31.nupkg.sha512"
    },
    "System.Composition.Convention/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GLjh2Ju71k6C0qxMMtl4efHa68NmWeIUYh4fkUI8xbjQrEBvFmRwMDFcylT8/PR9SQbeeL48IkFxU/+gd0nYEQ==",
      "path": "system.composition.convention/1.0.31",
      "hashPath": "system.composition.convention.1.0.31.nupkg.sha512"
    },
    "System.Composition.Hosting/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-fN1bT4RX4vUqjbgoyuJFVUizAl2mYF5VAb+bVIxIYZSSc0BdnX+yGAxcavxJuDDCQ1K+/mdpgyEFc8e9ikjvrg==",
      "path": "system.composition.hosting/1.0.31",
      "hashPath": "system.composition.hosting.1.0.31.nupkg.sha512"
    },
    "System.Composition.Runtime/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-0LEJN+2NVM89CE4SekDrrk5tHV5LeATltkp+9WNYrR+Huiyt0vaCqHbbHtVAjPyeLWIc8dOz/3kthRBj32wGQg==",
      "path": "system.composition.runtime/1.0.31",
      "hashPath": "system.composition.runtime.1.0.31.nupkg.sha512"
    },
    "System.Composition.TypedParts/1.0.31": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-0Zae/FtzeFgDBBuILeIbC/T9HMYbW4olAmi8XqqAGosSOWvXfiQLfARZEhiGd0LVXaYgXr0NhxiU1LldRP1fpQ==",
      "path": "system.composition.typedparts/1.0.31",
      "hashPath": "system.composition.typedparts.1.0.31.nupkg.sha512"
    },
    "System.Configuration.ConfigurationManager/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-UIFvaFfuKhLr9u5tWMxmVoDPkFeD+Qv8gUuap4aZgVGYSYMdERck4OhLN/2gulAc0nYTEigWXSJNNWshrmxnng==",
      "path": "system.configuration.configurationmanager/4.5.0",
      "hashPath": "system.configuration.configurationmanager.4.5.0.nupkg.sha512"
    },
    "System.Data.Common/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-lm6E3T5u7BOuEH0u18JpbJHxBfOJPuCyl4Kg1RH10ktYLp5uEEE1xKrHW56/We4SnZpGAuCc9N0MJpSDhTHZGQ==",
      "path": "system.data.common/4.3.0",
      "hashPath": "system.data.common.4.3.0.nupkg.sha512"
    },
    "System.Data.SqlClient/4.6.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7MTJhBFt4/tSJ2VRpG+TQNPcDdbCnWuoYzbe0kD5O3mQ7aZ2Q+Q3D9Y5Y3/aper5Gp3Lrs+8edk9clvxyPYXPw==",
      "path": "system.data.sqlclient/4.6.1",
      "hashPath": "system.data.sqlclient.4.6.1.nupkg.sha512"
    },
    "System.Diagnostics.PerformanceCounter/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-JUO5/moXgchWZBMBElgmPebZPKCgwW8kY3dFwVJavaNR2ftcc/YjXXGjOaCjly2KBXT7Ld5l/GTkMVzNv41yZA==",
      "path": "system.diagnostics.performancecounter/4.5.0",
      "hashPath": "system.diagnostics.performancecounter.4.5.0.nupkg.sha512"
    },
    "System.Diagnostics.Process/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-J0wOX07+QASQblsfxmIMFc9Iq7KTXYL3zs2G/Xc704Ylv3NpuVdo6gij6V3PGiptTxqsK0K7CdXenRvKUnkA2g==",
      "path": "system.diagnostics.process/4.3.0",
      "hashPath": "system.diagnostics.process.4.3.0.nupkg.sha512"
    },
    "System.Drawing.Common/4.5.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GiyeGi/v4xYDz1vCNFwFvhz9k1XddOG7VD3jxRqzRBCbTHji+s3HxxbxtoymuK4OadEpgotI8zQ5+GEEH9sUEQ==",
      "path": "system.drawing.common/4.5.1",
      "hashPath": "system.drawing.common.4.5.1.nupkg.sha512"
    },
    "System.Drawing.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-1QU/c35gwdhvj77fkScXQQbjiVAqIL3fEYn/19NE0CV/ic5TN5PyWAft8HsrbRd4SBLEoErNCkWSzMDc0MmbRw==",
      "path": "system.drawing.primitives/4.3.0",
      "hashPath": "system.drawing.primitives.4.3.0.nupkg.sha512"
    },
    "System.Interactive.Async/3.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-C07p0dAA5lGqYUPiPCK3paR709gqS4aMDDsje0v0pvffwzLaxmsn5YQTfZbyNG5qrudPx+BCxTqISnncQ3wIoQ==",
      "path": "system.interactive.async/3.2.0",
      "hashPath": "system.interactive.async.3.2.0.nupkg.sha512"
    },
    "System.IO.FileSystem.AccessControl/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-crjUsNiUm4n6uwPC4K6ya43tu6+pDo8CPQRj/mkVzsAYKLeP26PhhkJuUFSP0TYW1wEe8ZajPi7Mw+WPm3AmfQ==",
      "path": "system.io.filesystem.accesscontrol/4.3.0",
      "hashPath": "system.io.filesystem.accesscontrol.4.3.0.nupkg.sha512"
    },
    "System.IO.Packaging/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-OjtW5pGp1E7KE3ontBrzy+nSFJCM6pcbVDdo3gg4DauTfdtZKdeKvMVlADF4fSY+OfXWUp4qCvOSxIltt37LbA==",
      "path": "system.io.packaging/4.5.0",
      "hashPath": "system.io.packaging.4.5.0.nupkg.sha512"
    },
    "System.Linq.Parallel/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-td7x21K8LalpjTWCzW/nQboQIFbq9i0r+PCyBBCdLWWnm4NBcdN18vpz/G9hCpUaCIfRL+ZxJNVTywlNlB1aLQ==",
      "path": "system.linq.parallel/4.3.0",
      "hashPath": "system.linq.parallel.4.3.0.nupkg.sha512"
    },
    "System.Linq.Queryable/4.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Yn/WfYe9RoRfmSLvUt2JerP0BTGGykCZkQPgojaxgzF2N0oPo+/AhB8TXOpdCcNlrG3VRtsamtK2uzsp3cqRVw==",
      "path": "system.linq.queryable/4.0.1",
      "hashPath": "system.linq.queryable.4.0.1.nupkg.sha512"
    },
    "System.Net.Http/4.3.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-aOa2d51SEbmM+H+Csw7yJOuNZoHkrP2XnAurye5HWYgGVVU54YZDvsLUYRv6h18X3sPnjNCANmN7ZhIPiqMcjA==",
      "path": "system.net.http/4.3.4",
      "hashPath": "system.net.http.4.3.4.nupkg.sha512"
    },
    "System.Net.NameResolution/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-AFYl08R7MrsrEjqpQWTZWBadqXyTzNDaWpMqyxhb0d6sGhV6xMDKueuBXlLL30gz+DIRY6MpdgnHWlCh5wmq9w==",
      "path": "system.net.nameresolution/4.3.0",
      "hashPath": "system.net.nameresolution.4.3.0.nupkg.sha512"
    },
    "System.Net.NetworkInformation/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-zNVmWVry0pAu7lcrRBhwwU96WUdbsrGL3azyzsbXmVNptae1+Za+UgOe9Z6s8iaWhPn7/l4wQqhC56HZWq7tkg==",
      "path": "system.net.networkinformation/4.3.0",
      "hashPath": "system.net.networkinformation.4.3.0.nupkg.sha512"
    },
    "System.Net.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qOu+hDwFwoZPbzPvwut2qATe3ygjeQBDQj91xlsaqGFQUI5i4ZnZb8yyQuLGpDGivEPIt8EJkd1BVzVoP31FXA==",
      "path": "system.net.primitives/4.3.0",
      "hashPath": "system.net.primitives.4.3.0.nupkg.sha512"
    },
    "System.Net.Requests/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-OZNUuAs0kDXUzm7U5NZ1ojVta5YFZmgT2yxBqsQ7Eseq5Ahz88LInGRuNLJ/NP2F8W1q7tse1pKDthj3reF5QA==",
      "path": "system.net.requests/4.3.0",
      "hashPath": "system.net.requests.4.3.0.nupkg.sha512"
    },
    "System.Net.Security/4.3.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xT2jbYpbBo3ha87rViHoTA6WdvqOAW37drmqyx/6LD8p7HEPT2qgdxoimRzWtPg8Jh4X5G9BV2seeTv4x6FYlA==",
      "path": "system.net.security/4.3.2",
      "hashPath": "system.net.security.4.3.2.nupkg.sha512"
    },
    "System.Net.Sockets/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-m6icV6TqQOAdgt5N/9I5KNpjom/5NFtkmGseEH+AK/hny8XrytLH3+b5M8zL/Ycg3fhIocFpUMyl/wpFnVRvdw==",
      "path": "system.net.sockets/4.3.0",
      "hashPath": "system.net.sockets.4.3.0.nupkg.sha512"
    },
    "System.Net.WebHeaderCollection/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-XZrXYG3c7QV/GpWeoaRC02rM6LH2JJetfVYskf35wdC/w2fFDFMphec4gmVH2dkll6abtW14u9Rt96pxd9YH2A==",
      "path": "system.net.webheadercollection/4.3.0",
      "hashPath": "system.net.webheadercollection.4.3.0.nupkg.sha512"
    },
    "System.Private.ServiceModel/4.6.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-8iD5mvu76p3L9tUzmh1Iwgh51XRGIBKfKZhnfUoORjqDCabKugQOmne0SWzuU0ksNW1d6fTn6zEFmig8AbzI3g==",
      "path": "system.private.servicemodel/4.6.0",
      "hashPath": "system.private.servicemodel.4.6.0.nupkg.sha512"
    },
    "System.Reflection.DispatchProxy/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-+UW1hq11TNSeb+16rIk8hRQ02o339NFyzMc4ma/FqmxBzM30l1c2IherBB4ld1MNcenS48fz8tbt50OW4rVULA==",
      "path": "system.reflection.dispatchproxy/4.5.0",
      "hashPath": "system.reflection.dispatchproxy.4.5.0.nupkg.sha512"
    },
    "System.Runtime.CompilerServices.Unsafe/4.5.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-wprSFgext8cwqymChhrBLu62LMg/1u92bU+VOwyfBimSPVFXtsNqEWC92Pf9ofzJFlk4IHmJA75EDJn1b2goAQ==",
      "path": "system.runtime.compilerservices.unsafe/4.5.2",
      "hashPath": "system.runtime.compilerservices.unsafe.4.5.2.nupkg.sha512"
    },
    "System.Runtime.Serialization.Json/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-CpVfOH0M/uZ5PH+M9+Gu56K0j9lJw3M+PKRegTkcrY/stOIvRUeonggxNrfBYLA5WOHL2j15KNJuTuld3x4o9w==",
      "path": "system.runtime.serialization.json/4.3.0",
      "hashPath": "system.runtime.serialization.json.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.ProtectedData/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-wLBKzFnDCxP12VL9ANydSYhk59fC4cvOr9ypYQLPnAj48NQIhqnjdD2yhP8yEKyBJEjERWS9DisKL7rX5eU25Q==",
      "path": "system.security.cryptography.protecteddata/4.5.0",
      "hashPath": "system.security.cryptography.protecteddata.4.5.0.nupkg.sha512"
    },
    "System.ServiceModel.Primitives/4.6.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LwFQqlATMZh5bdL2bZmEhuYGgTeb+GxT0WWkYqA2A+bywYM5Oq3wouHSigm1kRQwk4iQjrfsk5wxBdjmK1ozRQ==",
      "path": "system.servicemodel.primitives/4.6.0",
      "hashPath": "system.servicemodel.primitives.4.6.0.nupkg.sha512"
    },
    "System.Text.Encoding.CodePages/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-S0wEUiKcLvRlkFUXca8uio1UQ5bYQzYgOmOKtCqaBQC3GR9AJjh43otcM32IGsAyvadFTaAMw9Irm6dS4Evfng==",
      "path": "system.text.encoding.codepages/4.5.0",
      "hashPath": "system.text.encoding.codepages.4.5.0.nupkg.sha512"
    },
    "System.Threading.Overlapped/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-m3HQ2dPiX/DSTpf+yJt8B0c+SRvzfqAJKx+QDWi+VLhz8svLT23MVjEOHPF/KiSLeArKU/iHescrbLd3yVgyNg==",
      "path": "system.threading.overlapped/4.3.0",
      "hashPath": "system.threading.overlapped.4.3.0.nupkg.sha512"
    },
    "System.Threading.ThreadPool/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-SrCOmTvvOOCmJT4BBxdZcwhz6XEMsjBGZvGSVorOxdCznaUNeVotEjDfXGIZ8gcGo00qgPbTV6puHDgcFYl6Iw==",
      "path": "system.threading.threadpool/4.3.0",
      "hashPath": "system.threading.threadpool.4.3.0.nupkg.sha512"
    },
    "WindowsAzure.Storage/9.3.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-VUV8j6dzkU/7vDgk5k+ob5g7nnch2fNByr0p9aOxMGFGk+tAnTehrZ4qnClF04CVozP1GNN2zrnbsxCmr+iZBg==",
      "path": "windowsazure.storage/9.3.3",
      "hashPath": "windowsazure.storage.9.3.3.nupkg.sha512"
    },
    "Z.EntityFramework.Plus.EFCore/2.0.7": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-78MwYBielbyKTlzD9Ao9olQ15AnnPVmrBL78Gmpx0C3Nl0KuZjPm5x3lynYFEivjZ4MBay9zKOj2TV2awP57Pg==",
      "path": "z.entityframework.plus.efcore/2.0.7",
      "hashPath": "z.entityframework.plus.efcore.2.0.7.nupkg.sha512"
    },
    "ZXing.Net/0.16.4": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9Qaq+ovAlJ5SL+1HiIzNXOLObj0iCPAytDtsa7QVSn7qx/CspfnlnleGYLeCiESTHBwW1ERxI1MBc2veFsK1sg==",
      "path": "zxing.net/0.16.4",
      "hashPath": "zxing.net.0.16.4.nupkg.sha512"
    },
    "MemberPlatform.Cmon/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Core/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Core.Contracts/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Entities/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.I18n/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Mapping/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Persistence/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Persistence.Contracts/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Store/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "MemberPlatform.Store.Contracts/1.0.0": {
      "type": "project",
      "serviceable": false,
      "sha512": ""
    },
    "Microsoft.AspNet.WebApi.Client/5.2.6": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-owAlEIUZXWSnkK8Z1c+zR47A0X6ykF4XjbPok4lQKNuciUfHLGPd6QnI+rt/8KlQ17PmF+I4S3f+m+Qe4IvViw==",
      "path": "microsoft.aspnet.webapi.client/5.2.6",
      "hashPath": "microsoft.aspnet.webapi.client.5.2.6.nupkg.sha512"
    },
    "Microsoft.AspNetCore/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Bs75iht4lXS8uVWy/Cbsr9i0m2jRtnrfPEWU+6t0dQTZcJEfF9b7G2F7XvstLFWkAKSgYRzFkAwi/KypY0Qtew==",
      "path": "microsoft.aspnetcore/2.2.0",
      "hashPath": "microsoft.aspnetcore.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Antiforgery/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-fVQsSXNZz38Ysx8iKwwqfOLHhLrAeKEMBS5Ia3Lh7BJjOC2vPV28/yk08AovOMsB3SNQPGnE7bv+lsIBTmAkvw==",
      "path": "microsoft.aspnetcore.antiforgery/2.2.0",
      "hashPath": "microsoft.aspnetcore.antiforgery.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.App/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-L3W3kgOOU5+2Tdtnzywcs4/a3XFbwcM7Ghvr2uWnhLUvBithluWlGI+0/lXFrDysXaRMLSRJdExSLuSJJQYuTg==",
      "path": "microsoft.aspnetcore.app/2.2.0",
      "hashPath": "microsoft.aspnetcore.app.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-b0R9X7L6zMqNsssKDvhYHuNi5x0s4DyHTeXybIAyGaitKiW1Q5aAGKdV2codHPiePv9yHfC9hAMyScXQ/xXhPw==",
      "path": "microsoft.aspnetcore.authentication/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-VloMLDJMf3n/9ic5lCBOa42IBYJgyB1JhzLsL68Zqg+2bEPWfGBj/xCJy/LrKTArN0coOcZp3wyVTZlx0y9pHQ==",
      "path": "microsoft.aspnetcore.authentication.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Cookies/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Iar9VFlBHkZGdSG9ZUTmn6Q8Qg+6CtW5G/TyJI2F8B432TOH+nZlkU7O0W0byow6xsxqOYeTviSHz4cCJ3amfQ==",
      "path": "microsoft.aspnetcore.authentication.cookies/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.cookies.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Core/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-XlVJzJ5wPOYW+Y0J6Q/LVTEyfS4ssLXmt60T0SPP+D8abVhBTl+cgw2gDHlyKYIkcJg7btMVh383NDkMVqD/fg==",
      "path": "microsoft.aspnetcore.authentication.core/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.core.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Facebook/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-SOc/wjoBntSWVZ6uG0R/TqQ0xmxu2H1PhkuYxINYpkUB7s3cQQuRDyZtJIdQonzpWVwBRj0ImwktiMaBF/7ihQ==",
      "path": "microsoft.aspnetcore.authentication.facebook/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.facebook.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Google/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-norGVE0KRIT0pdNKhlLlsMi/7O69ACpx2RSj8rMHCoMRETCYH4PTqUbHI1kkfAGNUtcuQ8VIGIXSa1ZdGKWcdA==",
      "path": "microsoft.aspnetcore.authentication.google/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.google.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.JwtBearer/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FnyoLdiIo8KDobVcDuUYYFSbQYp1OR8vSMIOcW6M5+dtF9TC6XvCCS8Ook+DSbqUj6HPxwOIKa5BeIZm1/EpMw==",
      "path": "microsoft.aspnetcore.authentication.jwtbearer/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.jwtbearer.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.MicrosoftAccount/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-troBjvJAMK7P2Vb5sDOzCztq9vR8BJtajDznam2XuQai7kLh5z7cmkB+2zMin+K/HzNjqItJSuSyuaK2PoZ8nA==",
      "path": "microsoft.aspnetcore.authentication.microsoftaccount/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.microsoftaccount.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.OAuth/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-i33SSdJd0g3ENRnHczgzrOlru3ciPsyYHMgAh90sbURS8wuBx0Y4xXfRQcYfu1W0/uiHQO832KNb/ICINWqLzA==",
      "path": "microsoft.aspnetcore.authentication.oauth/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.oauth.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.OpenIdConnect/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-y4iu3vMFnkMTaqT9mCJhD3XUMfavNP0CoOeNOHd7ArqZfgzs3GqAPcBc8Ld6mK2u5OOva8C6bhnQfRu9z0qJKQ==",
      "path": "microsoft.aspnetcore.authentication.openidconnect/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.openidconnect.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.Twitter/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-wKfJeBL+13duv0o4q9zp4pW7UopBHaLafnq2GiIJTcu1x3RR/1N4sRIIppLSIJdulgM1XfNOivlIE2FEfZpmog==",
      "path": "microsoft.aspnetcore.authentication.twitter/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.twitter.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authentication.WsFederation/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TIBkO7Tx8uWXNL5Z7/6+iKdhTS+D9dpJMNcmiVxrAJUqxL4EWGHNqJyUp5yqI76GmbrT4GD23T3cUsSuCi7E0A==",
      "path": "microsoft.aspnetcore.authentication.wsfederation/2.2.0",
      "hashPath": "microsoft.aspnetcore.authentication.wsfederation.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authorization/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-/L0W8H3jMYWyaeA9gBJqS/tSWBegP9aaTM0mjRhxTttBY9z4RVDRYJ2CwPAmAXIuPr3r1sOw+CS8jFVRGHRezQ==",
      "path": "microsoft.aspnetcore.authorization/2.2.0",
      "hashPath": "microsoft.aspnetcore.authorization.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Authorization.Policy/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-aJCo6niDRKuNg2uS2WMEmhJTooQUGARhV2ENQ2tO5443zVHUo19MSgrgGo9FIrfD+4yKPF8Q+FF33WkWfPbyKw==",
      "path": "microsoft.aspnetcore.authorization.policy/2.2.0",
      "hashPath": "microsoft.aspnetcore.authorization.policy.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Connections.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Aqr/16Cu5XmGv7mLKJvXRxhhd05UJ7cTTSaUV4MZ3ynAzfgWjsAdpIU8FWuxwAjmVdmI8oOWuVDrbs+sRkhKnA==",
      "path": "microsoft.aspnetcore.connections.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.connections.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.CookiePolicy/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Kn9CvhNsxRup/5SJfD4/YP3AbFwLJX8u3tKKyQszjUIvjE7M6lU93W44zlqBxltS94gTdLmo2ixPWDNeZthi1w==",
      "path": "microsoft.aspnetcore.cookiepolicy/2.2.0",
      "hashPath": "microsoft.aspnetcore.cookiepolicy.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Cors/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LFlTM3ThS3ZCILuKnjy8HyK9/IlDh3opogdbCVx6tMGyDzTQBgMPXLjGDLtMk5QmLDCcP3l1TO3z/+1viA8GUg==",
      "path": "microsoft.aspnetcore.cors/2.2.0",
      "hashPath": "microsoft.aspnetcore.cors.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Cryptography.Internal/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GXmMD8/vuTLPLvKzKEPz/4vapC5e0cwx1tUVd83ePRyWF9CCrn/pg4/1I+tGkQqFLPvi3nlI2QtPtC6MQN8Nww==",
      "path": "microsoft.aspnetcore.cryptography.internal/2.2.0",
      "hashPath": "microsoft.aspnetcore.cryptography.internal.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Cryptography.KeyDerivation/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-NCY0PH3nrFYbhqiq72rwWsUXlV4OAE0MOukvGvIBOTnEPMC1yVL42k1DXLnaIu+c0yfMAxIIG9Iuaykp9BQQQw==",
      "path": "microsoft.aspnetcore.cryptography.keyderivation/2.2.0",
      "hashPath": "microsoft.aspnetcore.cryptography.keyderivation.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.DataProtection/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-G6dvu5Nd2vjpYbzazZ//qBFbSEf2wmBUbyAR7E4AwO3gWjhoJD5YxpThcGJb7oE3VUcW65SVMXT+cPCiiBg8Sg==",
      "path": "microsoft.aspnetcore.dataprotection/2.2.0",
      "hashPath": "microsoft.aspnetcore.dataprotection.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.DataProtection.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-seANFXmp8mb5Y12m1ShiElJ3ZdOT3mBN3wA1GPhHJIvZ/BxOCPyqEOR+810OWsxEZwA5r5fDRNpG/CqiJmQnJg==",
      "path": "microsoft.aspnetcore.dataprotection.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.dataprotection.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.DataProtection.Extensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Goo1xU9WJnEJ0dKDgYFF+hFQqRMLKjf9zc8Bu3PaBdGncR7QwDMeFIkO7FEM6izaC38QjYrs1Q5AsmljkPyOrw==",
      "path": "microsoft.aspnetcore.dataprotection.extensions/2.2.0",
      "hashPath": "microsoft.aspnetcore.dataprotection.extensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Diagnostics/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RobNuZecn/eefWVApOE+OWAZXCdgfzm8pB7tBvJkahsjWfn1a+bLM9I2cuKlp/9aFBok1O/oDXlgYSvaQYu/yg==",
      "path": "microsoft.aspnetcore.diagnostics/2.2.0",
      "hashPath": "microsoft.aspnetcore.diagnostics.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Diagnostics.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-pva9ggfUDtnJIKzv0+wxwTX7LduDx6xLSpMqWwdOJkW52L0t31PI78+v+WqqMpUtMzcKug24jGs3nTFpAmA/2g==",
      "path": "microsoft.aspnetcore.diagnostics.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.diagnostics.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-xAIXyVmrTcVIJ38/j0TVMRlChC25k+cEAeSYotWhAnho3urzf1EfhoyyNdVytZbbBskue5i6XBL8gA1vlp5KGg==",
      "path": "microsoft.aspnetcore.diagnostics.entityframeworkcore/2.2.0",
      "hashPath": "microsoft.aspnetcore.diagnostics.entityframeworkcore.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Diagnostics.HealthChecks/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RNmdLy9yncTprony49cuwhyTKoROpVflGM+pKlHA1671F00QUsjoY1Oi6xoa9XsUrfRDRYlxbt2CHYCMLzMh7Q==",
      "path": "microsoft.aspnetcore.diagnostics.healthchecks/2.2.0",
      "hashPath": "microsoft.aspnetcore.diagnostics.healthchecks.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.HostFiltering/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-JSX6ZlVWDkokZ+xCKDhUVQNqbmFn1lHQNzJc8K4Y/uTUocZS83+b/8Q7y/yx3oJ362etGMVy0keAvmCdqbP8nA==",
      "path": "microsoft.aspnetcore.hostfiltering/2.2.0",
      "hashPath": "microsoft.aspnetcore.hostfiltering.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Hosting/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7t4RbUGugpHtQmzAkc9fpDdYJg6t/jcB2VVnjensVYbZFnLDU8pNrG0hrekk1DQG7P2UzpSqKLzDsFF0/lkkbw==",
      "path": "microsoft.aspnetcore.hosting/2.2.0",
      "hashPath": "microsoft.aspnetcore.hosting.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Hosting.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ubycklv+ZY7Kutdwuy1W4upWcZ6VFR8WUXU7l7B2+mvbDBBPAcfpi+E+Y5GFe+Q157YfA3C49D2GCjAZc7Mobw==",
      "path": "microsoft.aspnetcore.hosting.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.hosting.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Hosting.Server.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-1PMijw8RMtuQF60SsD/JlKtVfvh4NORAhF4wjysdABhlhTrYmtgssqyncR0Stq5vqtjplZcj6kbT4LRTglt9IQ==",
      "path": "microsoft.aspnetcore.hosting.server.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.hosting.server.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Html.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Y4rs5aMEXY8G7wJo5S3EEt6ltqyOTr/qOeZzfn+hw/fuQj5GppGckMY5psGLETo1U9hcT5MmAhaT5xtusM1b5g==",
      "path": "microsoft.aspnetcore.html.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.html.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-YogBSMotWPAS/X5967pZ+yyWPQkThxhmzAwyCHCSSldzYBkW5W5d6oPfBaPqQOnSHYTpSOSOkpZoAce0vwb6+A==",
      "path": "microsoft.aspnetcore.http/2.2.0",
      "hashPath": "microsoft.aspnetcore.http.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Nxs7Z1q3f1STfLYKJSVXCs1iBl+Ya6E8o4Oy1bCxJ/rNI44E/0f6tbsrVqAWfB7jlnJfyaAtIalBVxPKUPQb4Q==",
      "path": "microsoft.aspnetcore.http.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.http.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http.Connections/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZcwAM9rE5yjGC+vtiNAK0INybpKIqnvB+/rntZn2/CPtyiBAtovVrEp4UZOoC31zH5t0P78ix9gLNJzII/ODsA==",
      "path": "microsoft.aspnetcore.http.connections/1.1.0",
      "hashPath": "microsoft.aspnetcore.http.connections.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http.Connections.Common/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-mYk5QUUjyXQmlyDHWDjkLYDArt97plwe6KsDsNVhDEQ+HgZMKGjISyM6YSA7BERQNR25kXBTbIYfSy1vePGQgg==",
      "path": "microsoft.aspnetcore.http.connections.common/1.1.0",
      "hashPath": "microsoft.aspnetcore.http.connections.common.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http.Extensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-2DgZ9rWrJtuR7RYiew01nGRzuQBDaGHGmK56Rk54vsLLsCdzuFUPqbDTJCS1qJQWTbmbIQ9wGIOjpxA1t0l7/w==",
      "path": "microsoft.aspnetcore.http.extensions/2.2.0",
      "hashPath": "microsoft.aspnetcore.http.extensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Http.Features/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ziFz5zH8f33En4dX81LW84I6XrYXKf9jg6aM39cM+LffN9KJahViKZ61dGMSO2gd3e+qe5yBRwsesvyqlZaSMg==",
      "path": "microsoft.aspnetcore.http.features/2.2.0",
      "hashPath": "microsoft.aspnetcore.http.features.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.HttpOverrides/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-pOlLQyNKQduGbtbgB55RyTHFeshSfKi3DmofrVjk+UBQjyp+Tm0RNNJFQf+sv34hlFsel+VnD79QyO9Zk/c3oA==",
      "path": "microsoft.aspnetcore.httpoverrides/2.2.0",
      "hashPath": "microsoft.aspnetcore.httpoverrides.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.HttpsPolicy/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-0EmmwzAkWEPCC8rpg9nGfcOiitIOYkZ13f+b5ED7AAZvz/ZwkdWbeMarGf77lSyA+Mb9O/iAt4LWup0RRMVOJw==",
      "path": "microsoft.aspnetcore.httpspolicy/2.2.0",
      "hashPath": "microsoft.aspnetcore.httpspolicy.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Identity/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-F16BKeS96wKhyIyhaFR7m8kRIwIvPUW9Dx7IlGWmu2IIwnUDCdo+2z7IrWKA8r77pZQ1UE9kYcBPg5456YdAIA==",
      "path": "microsoft.aspnetcore.identity/2.2.0",
      "hashPath": "microsoft.aspnetcore.identity.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Identity.EntityFrameworkCore/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-PGJ8f8sE9vbnyPJpSCMYAjh1itkM8uL9QnkO5lQSSJGeyG4b1+zNoLS+leJgjGnlkTzgWPffc4OuqH7wsYahWw==",
      "path": "microsoft.aspnetcore.identity.entityframeworkcore/2.2.0",
      "hashPath": "microsoft.aspnetcore.identity.entityframeworkcore.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Identity.UI/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-T4B/Uaqd4u7jN6XDHbEBTZO002HquQKU49V+PvWEGKoiJBgZ96JskDr/NsfgVin8n8/bRSx+4A1WwlkMDKcNBg==",
      "path": "microsoft.aspnetcore.identity.ui/2.2.0",
      "hashPath": "microsoft.aspnetcore.identity.ui.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.JsonPatch/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-o9BB9hftnCsyJalz9IT0DUFxz8Xvgh3TOfGWolpuf19duxB4FySq7c25XDYBmBMS+sun5/PsEUAi58ra4iJAoA==",
      "path": "microsoft.aspnetcore.jsonpatch/2.2.0",
      "hashPath": "microsoft.aspnetcore.jsonpatch.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Localization/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-+PGX1mEfq19EVvskBBb9XBQrXZpZrh6hYhX0x3FkPTEqr+rDM2ZmsEwAAMRmzcidmlDM1/7cyDSU/WhkecU8tA==",
      "path": "microsoft.aspnetcore.localization/2.2.0",
      "hashPath": "microsoft.aspnetcore.localization.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Localization.Routing/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-kjheDUpXWaGOH8bUQafFAkUvw74xoe0Y2hojgeYaAg5LKvaFUwupkz8wgyhfSbLdejxEQJ6PsA7Zq/AcdPoIUQ==",
      "path": "microsoft.aspnetcore.localization.routing/2.2.0",
      "hashPath": "microsoft.aspnetcore.localization.routing.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.MiddlewareAnalysis/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GISp0KoVyJ4llqkmUOWFbOb7g/rOABlsf0Nt8a4eanY71XfUCM0dqBaMct3IUE3KWUvjhKPACQimxgMjPcF7pA==",
      "path": "microsoft.aspnetcore.middlewareanalysis/2.2.0",
      "hashPath": "microsoft.aspnetcore.middlewareanalysis.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-noun9xcrEvOs/ubczt2OluY9/bOOM2erv1D/gyyYtfS2sfyx2uGknUIAWoqmqc401TvQDysyx8S4M9j5zPIVBw==",
      "path": "microsoft.aspnetcore.mvc/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ET6uZpfVbGR1NjCuLaLy197cQ3qZUjzl7EG5SL4GfJH/c9KRE89MMBrQegqWsh0w1iRUB/zQaK0anAjxa/pz4g==",
      "path": "microsoft.aspnetcore.mvc.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Analyzers/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Wxxt1rFVHITp4MDaGQP/wyl+ROVVVeQCTWI6C8hxI8X66C4u6gcxvelqgnmsn+dISMCdE/7FQOwgiMx1HxuZqA==",
      "path": "microsoft.aspnetcore.mvc.analyzers/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.analyzers.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.ApiExplorer/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-iSREQct43Xg2t3KiQ2648e064al/HSLPXpI5yO9VPeTGDspWKHW23XFHRKPN1YjIQHHfBj8ytXbiF0XcSxp5pg==",
      "path": "microsoft.aspnetcore.mvc.apiexplorer/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.apiexplorer.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Core/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ALiY4a6BYsghw8PT5+VU593Kqp911U3w9f/dH9/ZoI3ezDsDAGiObqPu/HP1oXK80Ceu0XdQ3F0bx5AXBeuN/Q==",
      "path": "microsoft.aspnetcore.mvc.core/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.core.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Cors/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oINjMqhU7yzT2T9AMuvktlWlMd40i0do8E1aYslJS+c5fof+EMhjnwTh6cHN1dfrgjkoXJ/gutxn5Qaqf/81Kg==",
      "path": "microsoft.aspnetcore.mvc.cors/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.cors.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.DataAnnotations/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-WOw4SA3oT47aiU7ZjN/88j+b79YU6VftmHmxK29Km3PTI7WZdmw675QTcgWfsjEX4joCB82v7TvarO3D0oqOyw==",
      "path": "microsoft.aspnetcore.mvc.dataannotations/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.dataannotations.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Formatters.Json/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ScWwXrkAvw6PekWUFkIr5qa9NKn4uZGRvxtt3DvtUrBYW5Iu2y4SS/vx79JN0XDHNYgAJ81nVs+4M7UE1Y/O+g==",
      "path": "microsoft.aspnetcore.mvc.formatters.json/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.formatters.json.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Formatters.Xml/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-4pUhKtqhaNqSeMRRyEw1kGjg/pNLczzd4VAsanMGI539sCdkl1JBaoFojZb1helVdUvX9a1Jo+lYXq0lnwB/GQ==",
      "path": "microsoft.aspnetcore.mvc.formatters.xml/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.formatters.xml.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Localization/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-H1L4pP124mrN6duwOtNVIJUqy4CczC2/ah4MXarRt9ZRpJd2zNp1j3tJCgyEQpqai6zNVP6Vp2ZRMQcNDcNAKA==",
      "path": "microsoft.aspnetcore.mvc.localization/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.localization.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Razor/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TXvEOjp3r6qDEjmDtv3pXjQr/Zia9PpoGkl1MyTEqKqrUehBTpAdCjA8APXFwun19lH20OuyU+e4zDYv9g134w==",
      "path": "microsoft.aspnetcore.mvc.razor/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.razor.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Razor.Extensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Sei/0moqBDQKaAYT9PtOeRtvYgHQQLyw/jm3exHw2w9VdzejiMEqCQrN2d63Dk4y7IY0Irr/P9JUFkoVURRcNw==",
      "path": "microsoft.aspnetcore.mvc.razor.extensions/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.razor.extensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.Razor.ViewCompilation/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-dys8ggIDth3g5GBpCfeayU9sNg6Z9IbKFKOuaXbVaAiZQUd+Egk9op4NLHpqfR9Ey2HGw+u87LYC55bhEeOpag==",
      "path": "microsoft.aspnetcore.mvc.razor.viewcompilation/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.razor.viewcompilation.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.RazorPages/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GsMs4QKCf5VgdGZq9/nfAVkMJ/8uE4ie0Iugv4FtxbHBmMdpPQQBfTFKoUpwMbgIRw7hzV8xy2HPPU5o58PsdQ==",
      "path": "microsoft.aspnetcore.mvc.razorpages/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.razorpages.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.TagHelpers/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-hsrm/dLx7ztfWV+WEE7O8YqEePW7TmUwFwR7JsOUSTKaV9uSeghdmoOsYuk0HeoTiMhRxH8InQVE9/BgBj+jog==",
      "path": "microsoft.aspnetcore.mvc.taghelpers/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.taghelpers.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Mvc.ViewFeatures/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-dt7MGkzCFVTAD5oesI8UeVVeiSgaZ0tPdFstQjG6YLJSCiq1koOUSHMpf0PASGdOW/H9hxXkolIBhT5dWqJi7g==",
      "path": "microsoft.aspnetcore.mvc.viewfeatures/2.2.0",
      "hashPath": "microsoft.aspnetcore.mvc.viewfeatures.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.NodeServices/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ML+s+nv/ri3MxM4vXjTK3S4K925TGklSKH74VOkCqWQF9ki5yuYcyxaWTUsCyAXliw+N8HMNmW++uU81JngDDg==",
      "path": "microsoft.aspnetcore.nodeservices/2.2.0",
      "hashPath": "microsoft.aspnetcore.nodeservices.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Owin/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-h9QIdnrH7fOTQgUwjz/v0fDk8j8JNtUB233gYFtngt7jLoVc7vfMEGs9rnOWh8ubz+JdrMt7UBrva07af4Smxw==",
      "path": "microsoft.aspnetcore.owin/2.2.0",
      "hashPath": "microsoft.aspnetcore.owin.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Razor/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-V54PIyDCFl8COnTp9gezNHpUNHk7F9UnerGeZy3UfbnwYvfzbo+ipqQmSgeoESH8e0JvKhRTyQyZquW2EPtCmg==",
      "path": "microsoft.aspnetcore.razor/2.2.0",
      "hashPath": "microsoft.aspnetcore.razor.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Razor.Design/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-VLWK+ZtMMNukY6XjxYHc7mz33vkquoEzQJHm/LCF5REVxIaexLr+UTImljRRJBdUDJluDAQwU+59IX0rFDfURA==",
      "path": "microsoft.aspnetcore.razor.design/2.2.0",
      "hashPath": "microsoft.aspnetcore.razor.design.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Razor.Language/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-IeyzVFXZdpUAnWKWoNYE0SsP1Eu7JLjZaC94jaI1VfGtK57QykROz/iGMc8D0VcqC8i02qYTPQN/wPKm6PfidA==",
      "path": "microsoft.aspnetcore.razor.language/2.2.0",
      "hashPath": "microsoft.aspnetcore.razor.language.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Razor.Runtime/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7YqK+H61lN6yj9RiQUko7oaOhKtRR9Q/kBcoWNRemhJdTIWOh1OmdvJKzZrMWOlff3BAjejkPQm+0V0qXk+B1w==",
      "path": "microsoft.aspnetcore.razor.runtime/2.2.0",
      "hashPath": "microsoft.aspnetcore.razor.runtime.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.ResponseCaching/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-MEBP1UwGD7X1vhO43LN5KhZDt4HMTX7u1YA0nq7HR6IDRhWczHczJPDu3GbL01IMdb03hyT/glJIv8PI5zKtnA==",
      "path": "microsoft.aspnetcore.responsecaching/2.2.0",
      "hashPath": "microsoft.aspnetcore.responsecaching.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.ResponseCaching.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-CIHWEKrHzZfFp7t57UXsueiSA/raku56TgRYauV/W1+KAQq6vevz60zjEKaazt3BI76zwMz3B4jGWnCwd8kwQw==",
      "path": "microsoft.aspnetcore.responsecaching.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.responsecaching.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.ResponseCompression/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RvSstOhebIMDdRLd4iWjA6z2o2kGGwEYGPajvTXwndOA3TZpWH3FOIV4L7mehN/HoKrbTbX5vZ54ZFDwWoAFKA==",
      "path": "microsoft.aspnetcore.responsecompression/2.2.0",
      "hashPath": "microsoft.aspnetcore.responsecompression.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Rewrite/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-jztwQxyn4CcWZj/1mQtFiZ5+pIWYltHIXk5ykyrXMjO6qaKVvc+mlffSUCQ0AOl3vH7vxsZnda8poHwVaT0QIA==",
      "path": "microsoft.aspnetcore.rewrite/2.2.0",
      "hashPath": "microsoft.aspnetcore.rewrite.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Routing/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-jAhDBy0wryOnMhhZTtT9z63gJbvCzFuLm8yC6pHzuVu9ZD1dzg0ltxIwT4cfwuNkIL/TixdKsm3vpVOpG8euWQ==",
      "path": "microsoft.aspnetcore.routing/2.2.0",
      "hashPath": "microsoft.aspnetcore.routing.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Routing.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-lRRaPN7jDlUCVCp9i0W+PB0trFaKB0bgMJD7hEJS9Uo4R9MXaMC8X2tJhPLmeVE3SGDdYI4QNKdVmhNvMJGgPQ==",
      "path": "microsoft.aspnetcore.routing.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.routing.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.HttpSys/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-tei37PK4c6CKd7tGgAOkpbePwu8WLjqsEfiAfLbaMXnmp7o30bzcIxtraTrjvq2SpRAFA9p6WwUbmyqQxXPcfQ==",
      "path": "microsoft.aspnetcore.server.httpsys/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.httpsys.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.IIS/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-6NEwFAJFrnZ0f5eJB1ReIpgPM1ZRDj3IE3Rda01nD3vJANCyJFjZ4SGW3Ckn1AmMi225fGflWzpCKLb7/l43jw==",
      "path": "microsoft.aspnetcore.server.iis/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.iis.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.IISIntegration/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-iVjgAg+doTTrTFCOq6kZRpebXq94YGCx9efMIwO5QhwdY/sHAjfrVz2lXzji63G96YjJVK3ZRrlpgS2fd49ABw==",
      "path": "microsoft.aspnetcore.server.iisintegration/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.iisintegration.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.Kestrel/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-D0vGB8Tp0UNMiAhT+pwAVeqDDx2OFrfpu/plwm0WhA+1DZvTLc99eDwGISL6LAY8x7a12lhl9w7/m+VdoyDu8Q==",
      "path": "microsoft.aspnetcore.server.kestrel/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.kestrel.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.Kestrel.Core/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-F6/Vesd3ODq/ISbHfcvfRf7IzRtTvrNX8VA36Knm5e7bteJhoRA2GKQUVQ+neoO1njLvaQKnjcA3rdCZ6AF6cg==",
      "path": "microsoft.aspnetcore.server.kestrel.core/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.kestrel.core.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.Kestrel.Https/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-nEH5mU6idUYS3/+9BKw2stMOM25ZdGwIH4P4kyj6PVkMPgQUTkBQ7l/ScPkepdhejcOlPa+g3+M4dYsSYPUJ8g==",
      "path": "microsoft.aspnetcore.server.kestrel.https/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.kestrel.https.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.Kestrel.Transport.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-j1ai2CG8BGp4mYf2TWSFjjy1pRgW9XbqhdR4EOVvrlFVbcpEPfXNIPEdjkcgK+txWCupGzkFnFF8oZsASMtmyw==",
      "path": "microsoft.aspnetcore.server.kestrel.transport.abstractions/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.kestrel.transport.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Server.Kestrel.Transport.Sockets/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qTACI0wePgAKCH+YKrMgChyfqJpjwgGZEtSuwBw6TjWLQ66THGasleia/7EZz2t2eAjwWxw8RA/D8ODrBqpj9A==",
      "path": "microsoft.aspnetcore.server.kestrel.transport.sockets/2.2.0",
      "hashPath": "microsoft.aspnetcore.server.kestrel.transport.sockets.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.Session/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-lOjJVh293AKaOEPi1MIC1/G9gOVZMrve2a05o56oslK6bo0PMgMB17rmPomvqrJAjMdlWZ/MGdN2y78Z9wzWTw==",
      "path": "microsoft.aspnetcore.session/2.2.0",
      "hashPath": "microsoft.aspnetcore.session.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SignalR/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-V5X5XkeAHaFyyBOGPrddVeqTNo6zRPJNS5PRhlzEyBXiNG9AtqUbMyWFdZahQyMiIWJau550z59A4kdC9g5I9A==",
      "path": "microsoft.aspnetcore.signalr/1.1.0",
      "hashPath": "microsoft.aspnetcore.signalr.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SignalR.Common/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TyLgQ4y4RVUIxiYFnHT181/rJ33/tL/NcBWC9BwLpulDt5/yGCG4EvsToZ49EBQ7256zj+R6OGw6JF+jj6MdPQ==",
      "path": "microsoft.aspnetcore.signalr.common/1.1.0",
      "hashPath": "microsoft.aspnetcore.signalr.common.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SignalR.Core/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-mk69z50oFk2e89d3F/AfKeAvP3kvGG7MHG4ErydZiUd3ncSRq0kl0czq/COn/QVKYua9yGr2LIDwuR1C6/pu8Q==",
      "path": "microsoft.aspnetcore.signalr.core/1.1.0",
      "hashPath": "microsoft.aspnetcore.signalr.core.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SignalR.Protocols.Json/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-BOsjatDJnvnnXCMajOlC0ISmiFnJi/EyJzMo0i//5fZJVCLrQ4fyV/HzrhhAhSJuwJOQDdDozKQ9MB9jHq84pg==",
      "path": "microsoft.aspnetcore.signalr.protocols.json/1.1.0",
      "hashPath": "microsoft.aspnetcore.signalr.protocols.json.1.1.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SpaServices/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-hUAsOd45CQbUV47b/c5wp6uKM0Fa6MXekFHbRb+jEPjzmrxLPn9nAKK1dYmyMAqSBRL8c6zVCWQk+TOP7eGs/A==",
      "path": "microsoft.aspnetcore.spaservices/2.2.0",
      "hashPath": "microsoft.aspnetcore.spaservices.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.SpaServices.Extensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RvzzubzGPD+dGCfKVVtAvyIsnWpAWOA/x1n6fGLwICPER7Ze6budQGFPdZ7yuXTwtTMRvHa4O4AaGLG1XmoXGw==",
      "path": "microsoft.aspnetcore.spaservices.extensions/2.2.0",
      "hashPath": "microsoft.aspnetcore.spaservices.extensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.StaticFiles/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-byZDrjir6Co5EoWbraQyG0qbPCUG6XgGYQstipMF9lucOAjq/mqnIyt8B8iMWnin/ghZoOln9Y01af4rUAwOhA==",
      "path": "microsoft.aspnetcore.staticfiles/2.2.0",
      "hashPath": "microsoft.aspnetcore.staticfiles.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.WebSockets/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZpOcg2V0rCwU9ErfDb9y3Hcjoe7rU42XlmUS0mO4pVZQSgJVqR+DfyZtYd5LDa11F7bFNS2eezI9cBM3CmfGhw==",
      "path": "microsoft.aspnetcore.websockets/2.2.0",
      "hashPath": "microsoft.aspnetcore.websockets.2.2.0.nupkg.sha512"
    },
    "Microsoft.AspNetCore.WebUtilities/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9ErxAAKaDzxXASB/b5uLEkLgUWv1QbeVxyJYEHQwMaxXOeFFVkQxiq8RyfVcifLU7NR0QY0p3acqx4ZpYfhHDg==",
      "path": "microsoft.aspnetcore.webutilities/2.2.0",
      "hashPath": "microsoft.aspnetcore.webutilities.2.2.0.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.Analyzers/1.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-HS3iRWZKcUw/8eZ/08GXKY2Bn7xNzQPzf8gRPHGSowX7u7XXu9i9YEaBeBNKUXWfI7qjvT2zXtLUvbN0hds8vg==",
      "path": "microsoft.codeanalysis.analyzers/1.1.0",
      "hashPath": "microsoft.codeanalysis.analyzers.1.1.0.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.Common/2.8.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-06AzG7oOLKTCN1EnoVYL1bQz+Zwa10LMpUn7Kc+PdpN8CQXRqXTyhfxuKIz6t0qWfoatBNXdHD0OLcEYp5pOvQ==",
      "path": "microsoft.codeanalysis.common/2.8.0",
      "hashPath": "microsoft.codeanalysis.common.2.8.0.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.CSharp/2.8.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RizcFXuHgGmeuZhxxE1qQdhFA9lGOHlk0MJlCUt6LOnYsevo72gNikPcbANFHY02YK8L/buNrihchY0TroGvXQ==",
      "path": "microsoft.codeanalysis.csharp/2.8.0",
      "hashPath": "microsoft.codeanalysis.csharp.2.8.0.nupkg.sha512"
    },
    "Microsoft.CodeAnalysis.Razor/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-2qL0Qyu5qHzg6/JzF80mLgsqn9NP/Q0mQwjH+Z+DiqcuODJx8segjN4un2Tnz6bEAWv8FCRFNXR/s5wzlxqA8A==",
      "path": "microsoft.codeanalysis.razor/2.2.0",
      "hashPath": "microsoft.codeanalysis.razor.2.2.0.nupkg.sha512"
    },
    "Microsoft.CSharp/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-kaj6Wb4qoMuH3HySFJhxwQfe8R/sJsNJnANrvv8WdFPMoNbKY5htfNscv+LHCu5ipz+49m2e+WQXpLXr9XYemQ==",
      "path": "microsoft.csharp/4.5.0",
      "hashPath": "microsoft.csharp.4.5.0.nupkg.sha512"
    },
    "Microsoft.DotNet.PlatformAbstractions/2.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9KPDwvb/hLEVXYruVHVZ8BkebC8j17DmPb56LnqRF74HqSPLjCkrlFUjOtFpQPA2DeADBRTI/e69aCfRBfrhxw==",
      "path": "microsoft.dotnet.platformabstractions/2.1.0",
      "hashPath": "microsoft.dotnet.platformabstractions.2.1.0.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Design/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-mcsUEzg1bWvPgj/isz7aabDy41x+x8WBTmSF+JFsDGe3K5ZElWT2FSr3LLmkXk/5BLWJ3f9SDe0YR55u3ZgHrw==",
      "path": "microsoft.entityframeworkcore.design/2.2.0",
      "hashPath": "microsoft.entityframeworkcore.design.2.2.0.nupkg.sha512"
    },
    "Microsoft.EntityFrameworkCore.Tools/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-F253CmzpL7eXFKpK++/GIVbyVMZyXYq388osdkggsA1eL7c8ZGwHho0jE3LGA+L6WuXm6KbwQMtnt15zZAqzzA==",
      "path": "microsoft.entityframeworkcore.tools/2.2.0",
      "hashPath": "microsoft.entityframeworkcore.tools.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Caching.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-spsJkYo8gGJapaxTSQFN/wqA+ghpJMLwB4ZyTB+fSdpd7AmMFP/YSpIcGmczcw4KggpxLGhLk7lCkSIlgvHaqQ==",
      "path": "microsoft.extensions.caching.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.caching.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Caching.Memory/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-yFs44RzB2Pzfoj4uk+mEz3MTTQKyeWb8gDhv5GyVPfHnLv0eQhGwzbw/5WpxAcVyOgG/H3/0ULY6g0/7/B+r7w==",
      "path": "microsoft.extensions.caching.memory/2.2.0",
      "hashPath": "microsoft.extensions.caching.memory.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Caching.SqlServer/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-hDAunudTCNyVb22W+ctToi9T3mcrix2L+GfnuhbIcbzgXVyUGMULUJmb2D5ElIJKkcGxkC/lM1aBMgHsSFFZcA==",
      "path": "microsoft.extensions.caching.sqlserver/2.2.0",
      "hashPath": "microsoft.extensions.caching.sqlserver.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-nOP8R1mVb/6mZtm2qgAJXn/LFm/2kMjHDAg/QJLFG6CuWYJtaD3p1BwQhufBVvRzL9ceJ/xF0SQ0qsI2GkDQAA==",
      "path": "microsoft.extensions.configuration/2.2.0",
      "hashPath": "microsoft.extensions.configuration.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-65MrmXCziWaQFrI0UHkQbesrX5wTwf9XPjY5yFm/VkgJKFJ5gqvXRoXjIZcf2wLi5ZlwGz/oMYfyURVCWbM5iw==",
      "path": "microsoft.extensions.configuration.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.configuration.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.CommandLine/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-4kJIGOSRqD1Ccqerst4t/zsNs51plR7BIxbdKO1J/9rL+2DuNT+ieAuEv+HROelqTam3yOpKFR7TtHBt3oLpOA==",
      "path": "microsoft.extensions.configuration.commandline/2.2.0",
      "hashPath": "microsoft.extensions.configuration.commandline.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.FileExtensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-H1qCpWBC8Ed4tguTR/qYkbb3F6DI5Su3t8xyFo3/5MzAd8PwPpHzgX8X04KbBxKmk173Pb64x7xMHarczVFQUA==",
      "path": "microsoft.extensions.configuration.fileextensions/2.2.0",
      "hashPath": "microsoft.extensions.configuration.fileextensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.Ini/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-uEDasBxY7m0GJseqHD8QhfiznxDMhxN9YE3j01Es6eks42rRm3yL8ZMbRxuEjyKqGZqjjt+Vr297/nKcg0eOow==",
      "path": "microsoft.extensions.configuration.ini/2.2.0",
      "hashPath": "microsoft.extensions.configuration.ini.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.Json/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-jUDdmLyFmLf9V3mqnMzSAzAv4QigJ67tZh5Q7HBXeBnESL2UyeesNG6jSBti+b63JpxZf+EDyn+anx3gyrNxug==",
      "path": "microsoft.extensions.configuration.json/2.2.0",
      "hashPath": "microsoft.extensions.configuration.json.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.KeyPerFile/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-qK7vVxtUrpxdQPhvjF3RVYkcV86q/QfMBWqvvXAKYYkQ+H/4GXxk5cbPaSWdMZB5YU1GBEFBuZg9MZxDRvPJkg==",
      "path": "microsoft.extensions.configuration.keyperfile/2.2.0",
      "hashPath": "microsoft.extensions.configuration.keyperfile.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.UserSecrets/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-2/N2xo6/sNbVshnKktmq5lwaQbsAR2SrzCVrJEeMP8OKZVI7SzT8P6/WXZF8/YC7dTYsMe3nrHzgl1cF9i5ZKQ==",
      "path": "microsoft.extensions.configuration.usersecrets/2.2.0",
      "hashPath": "microsoft.extensions.configuration.usersecrets.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Configuration.Xml/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-toCFesKf2KZgRtb6T7tulnJv3IBVL+Gqd4KE3ebQZ20wA2Z5Rp6A44MsRGZ1ollmihzkxxBDavVfgufFeji3Sw==",
      "path": "microsoft.extensions.configuration.xml/2.2.0",
      "hashPath": "microsoft.extensions.configuration.xml.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.DependencyInjection/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-MZtBIwfDFork5vfjpJdG5g8wuJFt7d/y3LOSVVtDK/76wlbtz6cjltfKHqLx2TKVqTj5/c41t77m1+h20zqtPA==",
      "path": "microsoft.extensions.dependencyinjection/2.2.0",
      "hashPath": "microsoft.extensions.dependencyinjection.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.DependencyInjection.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-f9hstgjVmr6rmrfGSpfsVOl2irKAgr1QjrSi3FgnS7kulxband50f2brRLwySAQTADPZeTdow0mpSMcoAdadCw==",
      "path": "microsoft.extensions.dependencyinjection.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.dependencyinjection.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.DependencyModel/2.1.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-nS2XKqi+1A1umnYNLX2Fbm/XnzCxs5i+zXVJ3VC6r9t2z0NZr9FLnJN4VQpKigdcWH/iFTbMuX6M6WQJcTjVIg==",
      "path": "microsoft.extensions.dependencymodel/2.1.0",
      "hashPath": "microsoft.extensions.dependencymodel.2.1.0.nupkg.sha512"
    },
    "Microsoft.Extensions.DiagnosticAdapter/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Av0QGyboU9hYcprPduZg8Ny4mtp1Z0xOfZGCiBhYMh6a0loNomZ74U1P9EJUBksT2ZJd0+hh/pOQIVdAJ8+AbA==",
      "path": "microsoft.extensions.diagnosticadapter/2.2.0",
      "hashPath": "microsoft.extensions.diagnosticadapter.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Diagnostics.HealthChecks/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-p9njfetdebuplBCkIJPqyxsUIOBf/7B/RhPXZnFjh+/wqWNRqhP/1s18q1me9XP0l8uCD8TqJRPC+L0MCoUGRA==",
      "path": "microsoft.extensions.diagnostics.healthchecks/2.2.0",
      "hashPath": "microsoft.extensions.diagnostics.healthchecks.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-cO6f4csTakJXuLWnU/p5mfQInyNq5sSi4mS2YtQZcGoHynU6P/TD6gjqt1TRnVfwuZLw3tmmw2ipFrHbBUqWew==",
      "path": "microsoft.extensions.diagnostics.healthchecks.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.diagnostics.healthchecks.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.FileProviders.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-EcnaSsPTqx2MGnHrmWOD0ugbuuqVT8iICqSqPzi45V5/MA1LjUNb0kwgcxBGqizV1R+WeBK7/Gw25Jzkyk9bIw==",
      "path": "microsoft.extensions.fileproviders.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.fileproviders.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.FileProviders.Composite/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Az/RxWB+UlyVN/TvQFaGXx8XAXVZN5WQnnuJOsjwBzghSJc1i8zqNjIypPHOedcuIXs2XSWgOSL6YQ3BlCnoJA==",
      "path": "microsoft.extensions.fileproviders.composite/2.2.0",
      "hashPath": "microsoft.extensions.fileproviders.composite.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.FileProviders.Embedded/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-6e22jnVntG9JLLowjY40UBPLXkKTRlDpFHmo2evN8lwZIpO89ZRGz6JRdqhnVYCaavq5KeFU2W5VKPA5y5farA==",
      "path": "microsoft.extensions.fileproviders.embedded/2.2.0",
      "hashPath": "microsoft.extensions.fileproviders.embedded.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.FileProviders.Physical/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-tbDHZnBJkjYd9NjlRZ9ondDiv1Te3KYCTW2RWpR1B0e1Z8+EnFRo7qNnHkkSCixLdlPZzhjlX24d/PixQ7w2dA==",
      "path": "microsoft.extensions.fileproviders.physical/2.2.0",
      "hashPath": "microsoft.extensions.fileproviders.physical.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.FileSystemGlobbing/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZSsHZp3PyW6vk37tDEdypjgGlNtpJ0EixBMOfUod2Thx7GtwfFSAQXUQx8a8BN8vfWKGGMbp7jPWdoHx/At4wQ==",
      "path": "microsoft.extensions.filesystemglobbing/2.2.0",
      "hashPath": "microsoft.extensions.filesystemglobbing.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Hosting/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-PxZPd5QbWr8+3JN2segEaD7IAYI+mR8ZmMqgo6GOk+E+UKnRcbC3RSQgJrZYuWVQwJCvdxesO5e64LSHC1zC8g==",
      "path": "microsoft.extensions.hosting/2.2.0",
      "hashPath": "microsoft.extensions.hosting.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Hosting.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-+k4AEn68HOJat5gj1TWa6X28WlirNQO9sPIIeQbia+91n03esEtMSSoekSTpMjUzjqtJWQN3McVx0GvSPFHF/Q==",
      "path": "microsoft.extensions.hosting.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.hosting.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Http/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-hZ8mz6FgxSeFtkHzw+Ad0QOt2yjjpq4WaG9itnkyChtXYTrDlbkw3af2WJ9wdEAAyYqOlQaVDB6MJSEo8dd/vw==",
      "path": "microsoft.extensions.http/2.2.0",
      "hashPath": "microsoft.extensions.http.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Identity.Core/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-/C+Valwg8IeUwDIunusittHivA9iyf82Jr1yeUFWO2zH2mDMMeYgjRyDLZqfL/7Vq94PEQsgv1XAaDfAX8msMw==",
      "path": "microsoft.extensions.identity.core/2.2.0",
      "hashPath": "microsoft.extensions.identity.core.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Identity.Stores/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-WY6os4m5LcG7XXHQK1vQggjdNFs7h6CsidVLOzPjG7Cb1zwRYKzfRT/pSUD40JNGvVp4oNENjLPvu/30ufIGNw==",
      "path": "microsoft.extensions.identity.stores/2.2.0",
      "hashPath": "microsoft.extensions.identity.stores.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Localization/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-3nBQLeBrcd4Rgd9vQi4gF5NgAWxnQrHekjjwlgww4wyLNfJDizjiex2resOLoAuAgy3y2IIAWjOpbr0UKR2ykw==",
      "path": "microsoft.extensions.localization/2.2.0",
      "hashPath": "microsoft.extensions.localization.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Localization.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FQzXG/lYR9UOM2zHpqsjTRpp3EghIYo3FCsQpfmtbp+glPaU0WXZfNmMjyqBRmMj1Sq93fPnC+G9zzYRauuRQA==",
      "path": "microsoft.extensions.localization.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.localization.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Nxqhadc9FCmFHzU+fz3oc8sFlE6IadViYg8dfUdGzJZ2JUxnCsRghBhhOWdM4B2zSZqEc+0BjliBh/oNdRZuig==",
      "path": "microsoft.extensions.logging/2.2.0",
      "hashPath": "microsoft.extensions.logging.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.Abstractions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-B2WqEox8o+4KUOpL7rZPyh6qYjik8tHi2tN8Z9jZkHzED8ElYgZa/h6K+xliB435SqUcWT290Fr2aa8BtZjn8A==",
      "path": "microsoft.extensions.logging.abstractions/2.2.0",
      "hashPath": "microsoft.extensions.logging.abstractions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.Configuration/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ukU1mQGX9+xBsEzpNd13yl4deFVYI+fxxnmKpOhvNZsF+/trCrAUQh+9QM5pPGHbfYkz3lLQ4BXfKCP0502dLw==",
      "path": "microsoft.extensions.logging.configuration/2.2.0",
      "hashPath": "microsoft.extensions.logging.configuration.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.Console/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-1eGgcOJ++PMxW6sn++j6U7wsWvhEBm/5ScqBUUBGLRE8M7AHahi9tsxivDMqEXVM3F0/pshHl3kEpMXtw4BeFg==",
      "path": "microsoft.extensions.logging.console/2.2.0",
      "hashPath": "microsoft.extensions.logging.console.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.Debug/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-JjqWtshxUujSnxslFccCRAaH8uFOciqXkYdRw+h5MwpC4sUc+ju9yZzvVi6PA5vW09ckv26EkasEvXrofGiaJg==",
      "path": "microsoft.extensions.logging.debug/2.2.0",
      "hashPath": "microsoft.extensions.logging.debug.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.EventSource/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oOa5H+vdNgpsxE6vgtX4U/godKtX2edVi+QjlWb2PBQfavGIQ3WxtjxN+B0DQAjwBNdV4mW8cgOiDEZ8KdR7Ig==",
      "path": "microsoft.extensions.logging.eventsource/2.2.0",
      "hashPath": "microsoft.extensions.logging.eventsource.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Logging.TraceSource/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-2laIg/Mf1OhhduUKVN3//j+sYceyUocgGC/ySx6cnZFeNf2mezs32TmRZyzfkQAZQ6azlo/0wTxi8BgIVUyRYA==",
      "path": "microsoft.extensions.logging.tracesource/2.2.0",
      "hashPath": "microsoft.extensions.logging.tracesource.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.ObjectPool/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-gA8H7uQOnM5gb+L0uTNjViHYr+hRDqCdfugheGo/MxQnuHzmhhzCBTIPm19qL1z1Xe0NEMabfcOBGv9QghlZ8g==",
      "path": "microsoft.extensions.objectpool/2.2.0",
      "hashPath": "microsoft.extensions.objectpool.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Options/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-UpZLNLBpIZ0GTebShui7xXYh6DmBHjWM8NxGxZbdQh/bPZ5e6YswqI+bru6BnEL5eWiOdodsXtEz3FROcgi/qg==",
      "path": "microsoft.extensions.options/2.2.0",
      "hashPath": "microsoft.extensions.options.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Options.ConfigurationExtensions/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-d4WS6yVXaw43ffiUnHj8oG1t2B6RbDDiQcgdA+Eq//NlPa3Wd+GTJFKj4OM4eDF3GjVumGr/CEVRS/jcYoF5LA==",
      "path": "microsoft.extensions.options.configurationextensions/2.2.0",
      "hashPath": "microsoft.extensions.options.configurationextensions.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Options.DataAnnotations/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Xk7okx/u+ZQb8xvz71FwVmfZjwDh2DWrovhtQXprWE16KqaP8bs6A8wb0h9nTSFh9rcFDVeo42d47iduu01XvQ==",
      "path": "microsoft.extensions.options.dataannotations/2.2.0",
      "hashPath": "microsoft.extensions.options.dataannotations.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.Primitives/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-azyQtqbm4fSaDzZHD/J+V6oWMFaf2tWP4WEGIYePLCMw3+b2RQdj9ybgbQyjCshcitQKQ4lEDOZjmSlTTrHxUg==",
      "path": "microsoft.extensions.primitives/2.2.0",
      "hashPath": "microsoft.extensions.primitives.2.2.0.nupkg.sha512"
    },
    "Microsoft.Extensions.WebEncoders/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-V8XcqYcpcdBAxUhLeyYcuKmxu4CtNQA9IphTnARpQGhkop4A93v2XgM3AtaVVJo3H2cDWxWM6aeO8HxkifREqw==",
      "path": "microsoft.extensions.webencoders/2.2.0",
      "hashPath": "microsoft.extensions.webencoders.2.2.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.JsonWebTokens/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-5LW5VYvGZLvrbEGxyaE6dSQhT1B5frnpwX/c4/PWrNXeuJ6GkYmiOPf2u5Iwk1qQXPTvDedwEfnBg+i/0cFAyA==",
      "path": "microsoft.identitymodel.jsonwebtokens/5.3.0",
      "hashPath": "microsoft.identitymodel.jsonwebtokens.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Logging/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-o+bBauEMOi6ZI0MlJEC69Sw9UPwKLFmN+lD942g9UCx5pfiLFvJBKp8OPmxtGFL02ZxzXCIUyhyKn85izBDsnQ==",
      "path": "microsoft.identitymodel.logging/5.3.0",
      "hashPath": "microsoft.identitymodel.logging.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Protocols/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-o2Fx9cYQHKtOyVrCXB41kEmny1Zvm+fqXNTD5heB9yPY0C+qYm7fo1yCvtHaH2JPEersGW0iS2dE0s65kWkVEw==",
      "path": "microsoft.identitymodel.protocols/5.3.0",
      "hashPath": "microsoft.identitymodel.protocols.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Protocols.OpenIdConnect/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-NihXp2JT3fRbTq6AOQhEQT8TuJzhUNg9TOeK+TxlkkvanllWFF0gfXH5hTRn9Qn68HJQXtp/mtLbCWzi+4bCSg==",
      "path": "microsoft.identitymodel.protocols.openidconnect/5.3.0",
      "hashPath": "microsoft.identitymodel.protocols.openidconnect.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Protocols.WsFederation/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-6nGUoC+foCQ2UTsRD/Z6TLgsghuX10tunLXxuLE+LljW9H1oANqAQWrP8DNP++nfXke+qu1zVi6yBl6MMK/Dfg==",
      "path": "microsoft.identitymodel.protocols.wsfederation/5.3.0",
      "hashPath": "microsoft.identitymodel.protocols.wsfederation.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Tokens/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-/piauST4FL0qzVI6oqLWxqhFReg12KwVGy0jRlnVOpGMeOVSKdtNVtHsN/hARc25hOOPEp9WKMce5ILzyMx/tQ==",
      "path": "microsoft.identitymodel.tokens/5.3.0",
      "hashPath": "microsoft.identitymodel.tokens.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Tokens.Saml/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-XS6zgN7jKG7QDqG3fV9BRADs8HmRJ6vJDKVBPFFly9MCkS6KMFps4hBdBJ5ycPrXtPBfnISCLiGLHP54blCvWw==",
      "path": "microsoft.identitymodel.tokens.saml/5.3.0",
      "hashPath": "microsoft.identitymodel.tokens.saml.5.3.0.nupkg.sha512"
    },
    "Microsoft.IdentityModel.Xml/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-i4uFRjipeRXGhyfHmJaZ3PkOQIWhwxBJABNDWNaxcwUvramMCWYRLE1P3g4sLjiw8zXehH6eZwxww8F+dB7/+g==",
      "path": "microsoft.identitymodel.xml/5.3.0",
      "hashPath": "microsoft.identitymodel.xml.5.3.0.nupkg.sha512"
    },
    "Microsoft.Net.Http.Headers/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-iZNkjYqlo8sIOI0bQfpsSoMTmB/kyvmV2h225ihyZT33aTp48ZpF6qYnXxzSXmHt8DpBAwBTX+1s1UFLbYfZKg==",
      "path": "microsoft.net.http.headers/2.2.0",
      "hashPath": "microsoft.net.http.headers.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.App/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7z5l8Jp324S8bU8+yyWeYHXUFYvKyiI5lqS1dXgTzOx1H69Qbf6df12kCKlNX45LpMfCMd4U3M6p7Rl5Zk7SLA==",
      "path": "microsoft.netcore.app/2.2.0",
      "hashPath": "microsoft.netcore.app.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.DotNetAppHost/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-DrhaKInRKKvN6Ns2VNIlC7ZffLOp9THf8cO6X4fytPRJovJUbF49/zzx4WfgX9E44FMsw9hT8hrKiIqDSHvGvA==",
      "path": "microsoft.netcore.dotnetapphost/2.2.0",
      "hashPath": "microsoft.netcore.dotnetapphost.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.DotNetHostPolicy/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FJie7IoPZFaPgNDxhZGmDBQP/Bs5vPdfca/G2Wf9gd6LIvMYkZcibtmJwB4tcf4KXkaOYfIOo4Cl9sEPMsSzkw==",
      "path": "microsoft.netcore.dotnethostpolicy/2.2.0",
      "hashPath": "microsoft.netcore.dotnethostpolicy.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.DotNetHostResolver/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-spDm3AJYmebthDNhzY17YLPtvbc+Y1lCLVeiIH1uLJ/hZaM+40pBiPefFR8J1u66Ndkqi8ipR2tEbqPnYnjRhw==",
      "path": "microsoft.netcore.dotnethostresolver/2.2.0",
      "hashPath": "microsoft.netcore.dotnethostresolver.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.Platforms/2.2.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-T/J+XZo+YheFTJh8/4uoeJDdz5qOmOMkjg6/VL8mHJ9AnP8+fmV/kcbxeXsob0irRNiChf+V0ig1MCRLp/+Kog==",
      "path": "microsoft.netcore.platforms/2.2.0",
      "hashPath": "microsoft.netcore.platforms.2.2.0.nupkg.sha512"
    },
    "Microsoft.NETCore.Targets/2.0.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-odP/tJj1z6GylFpNo7pMtbd/xQgTC3Ex2If63dRTL38bBNMwsBnJ+RceUIyHdRBC0oik/3NehYT+oECwBhIM3Q==",
      "path": "microsoft.netcore.targets/2.0.0",
      "hashPath": "microsoft.netcore.targets.2.0.0.nupkg.sha512"
    },
    "Microsoft.Win32.Registry/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-+FWlwd//+Tt56316p00hVePBCouXyEzT86Jb3+AuRotTND0IYn0OO3obs1gnQEs/txEnt+rF2JBGLItTG+Be6A==",
      "path": "microsoft.win32.registry/4.5.0",
      "hashPath": "microsoft.win32.registry.4.5.0.nupkg.sha512"
    },
    "NETStandard.Library/2.0.3": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-st47PosZSHrjECdjeIzZQbzivYBJFv6P2nv4cj2ypdI204DO+vZ7l5raGMiX4eXMJ53RfOIg+/s4DHVZ54Nu2A==",
      "path": "netstandard.library/2.0.3",
      "hashPath": "netstandard.library.2.0.3.nupkg.sha512"
    },
    "Newtonsoft.Json.Bson/1.0.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-5PYT/IqQ+UK31AmZiSS102R6EsTo+LGTSI8bp7WAUqDKaF4wHXD8U9u4WxTI1vc64tYi++8p3dk3WWNqPFgldw==",
      "path": "newtonsoft.json.bson/1.0.1",
      "hashPath": "newtonsoft.json.bson.1.0.1.nupkg.sha512"
    },
    "runtime.native.System/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-c/qWt2LieNZIj1jGnVNsE2Kl23Ya2aSTBuXMD6V7k9KWr6l16Tqdwq+hJScEpWER9753NWC8h96PaVNY5Ld7Jw==",
      "path": "runtime.native.system/4.3.0",
      "hashPath": "runtime.native.system.4.3.0.nupkg.sha512"
    },
    "runtime.native.System.IO.Compression/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-b+V9JC/Ii3sR659flBeaBJww111425tgjcDS1k+hqV4sGh9FALRDBvJnDtQ895gAzpPTUOFDHdqaZ2Et7BpZMg==",
      "path": "runtime.native.system.io.compression/4.3.0",
      "hashPath": "runtime.native.system.io.compression.4.3.0.nupkg.sha512"
    },
    "runtime.native.System.Net.Http/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZVuZJqnnegJhd2k/PtAbbIcZ3aZeITq3sj06oKfMBSfphW3HDmk/t4ObvbOk/JA/swGR0LNqMksAh/f7gpTROg==",
      "path": "runtime.native.system.net.http/4.3.0",
      "hashPath": "runtime.native.system.net.http.4.3.0.nupkg.sha512"
    },
    "runtime.native.System.Security.Cryptography.Apple/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-DloMk88juo0OuOWr56QG7MNchmafTLYWvABy36izkrLI5VledI0rq28KGs1i9wbpeT9NPQrx/wTf8U2vazqQ3Q==",
      "path": "runtime.native.system.security.cryptography.apple/4.3.0",
      "hashPath": "runtime.native.system.security.cryptography.apple.4.3.0.nupkg.sha512"
    },
    "runtime.osx.10.10-x64.runtime.native.System.Security.Cryptography.Apple/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-kVXCuMTrTlxq4XOOMAysuNwsXWpYeboGddNGpIgNSZmv1b6r/s/DPk0fYMB7Q5Qo4bY68o48jt4T4y5BVecbCQ==",
      "path": "runtime.osx.10.10-x64.runtime.native.system.security.cryptography.apple/4.3.0",
      "hashPath": "runtime.osx.10.10-x64.runtime.native.system.security.cryptography.apple.4.3.0.nupkg.sha512"
    },
    "System.AppContext/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-fKC+rmaLfeIzUhagxY17Q9siv/sPrjjKcfNg1Ic8IlQkZLipo8ljcaZQu4VtI4Jqbzjc2VTjzGLF6WmsRXAEgA==",
      "path": "system.appcontext/4.3.0",
      "hashPath": "system.appcontext.4.3.0.nupkg.sha512"
    },
    "System.Buffers/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-pL2ChpaRRWI/p4LXyy4RgeWlYF2sgfj/pnVMvBqwNFr5cXg7CXNnWZWxrOONLg8VGdFB8oB+EG2Qw4MLgTOe+A==",
      "path": "system.buffers/4.5.0",
      "hashPath": "system.buffers.4.5.0.nupkg.sha512"
    },
    "System.Collections/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-3Dcj85/TBdVpL5Zr+gEEBUuFe2icOnLalmEh9hfck1PTYbbyWuZgh4fmm2ysCLTrqLQw6t3TgTyJ+VLp+Qb+Lw==",
      "path": "system.collections/4.3.0",
      "hashPath": "system.collections.4.3.0.nupkg.sha512"
    },
    "System.Collections.Concurrent/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ztl69Xp0Y/UXCL+3v3tEU+lIy+bvjKNUmopn1wep/a291pVPK7dxBd6T7WnlQqRog+d1a/hSsgRsmFnIBKTPLQ==",
      "path": "system.collections.concurrent/4.3.0",
      "hashPath": "system.collections.concurrent.4.3.0.nupkg.sha512"
    },
    "System.Collections.NonGeneric/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LE/oChpRvkSi3U25u0KnJcI44JeDZ1QJCyN4qFDx2uusEypdqR24w7lKYw21eYe5esuCBuc862wRmpF63Yy1KQ==",
      "path": "system.collections.nongeneric/4.3.0",
      "hashPath": "system.collections.nongeneric.4.3.0.nupkg.sha512"
    },
    "System.Collections.Specialized/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Epx8PoVZR0iuOnJJDzp7pWvdfMMOAvpUo95pC4ScH2mJuXkKA2Y4aR3cG9qt2klHgSons1WFh4kcGW7cSXvrxg==",
      "path": "system.collections.specialized/4.3.0",
      "hashPath": "system.collections.specialized.4.3.0.nupkg.sha512"
    },
    "System.ComponentModel.Annotations/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-UxYQ3FGUOtzJ7LfSdnYSFd7+oEv6M8NgUatatIN2HxNtDdlcvFAf+VIq4Of9cDMJEJC0aSRv/x898RYhB4Yppg==",
      "path": "system.componentmodel.annotations/4.5.0",
      "hashPath": "system.componentmodel.annotations.4.5.0.nupkg.sha512"
    },
    "System.Console/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-DHDrIxiqk1h03m6khKWV2X8p/uvN79rgSqpilL6uzpmSfxfU5ng8VcPtW4qsDsQDHiTv6IPV9TmD5M/vElPNLg==",
      "path": "system.console/4.3.0",
      "hashPath": "system.console.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.Contracts/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-eelRRbnm+OloiQvp9CXS0ixjNQldjjkHO4iIkR5XH2VIP8sUB/SIpa1TdUW6/+HDcQ+MlhP3pNa1u5SbzYuWGA==",
      "path": "system.diagnostics.contracts/4.3.0",
      "hashPath": "system.diagnostics.contracts.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.Debug/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-ZUhUOdqmaG5Jk3Xdb8xi5kIyQYAA4PnTNlHx1mu9ZY3qv4ELIdKbnL/akbGaKi2RnNUWaZsAs31rvzFdewTj2g==",
      "path": "system.diagnostics.debug/4.3.0",
      "hashPath": "system.diagnostics.debug.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.DiagnosticSource/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-eIHRELiYDQvsMToML81QFkXEEYXUSUT2F28t1SGrevWqP+epFdw80SyAXIKTXOHrIEXReFOEnEr7XlGiC2GgOg==",
      "path": "system.diagnostics.diagnosticsource/4.5.0",
      "hashPath": "system.diagnostics.diagnosticsource.4.5.0.nupkg.sha512"
    },
    "System.Diagnostics.FileVersionInfo/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-omCF64wzQ3Q2CeIqkD6lmmxeMZtGHUmzgFMPjfVaOsyqpR66p/JaZzManMw1s33osoAb5gqpncsjie67+yUPHQ==",
      "path": "system.diagnostics.fileversioninfo/4.3.0",
      "hashPath": "system.diagnostics.fileversioninfo.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.StackTrace/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-BiHg0vgtd35/DM9jvtaC1eKRpWZxr0gcQd643ABG7GnvSlf5pOkY2uyd42mMOJoOmKvnpNj0F4tuoS1pacTwYw==",
      "path": "system.diagnostics.stacktrace/4.3.0",
      "hashPath": "system.diagnostics.stacktrace.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.Tools/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-UUvkJfSYJMM6x527dJg2VyWPSRqIVB0Z7dbjHst1zmwTXz5CcXSYJFWRpuigfbO1Lf7yfZiIaEUesfnl/g5EyA==",
      "path": "system.diagnostics.tools/4.3.0",
      "hashPath": "system.diagnostics.tools.4.3.0.nupkg.sha512"
    },
    "System.Diagnostics.Tracing/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-rswfv0f/Cqkh78rA5S8eN8Neocz234+emGCtTF3lxPY96F+mmmUen6tbn0glN6PMvlKQb9bPAY5e9u7fgPTkKw==",
      "path": "system.diagnostics.tracing/4.3.0",
      "hashPath": "system.diagnostics.tracing.4.3.0.nupkg.sha512"
    },
    "System.Dynamic.Runtime/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-SNVi1E/vfWUAs/WYKhE9+qlS6KqK0YVhnlT0HQtr8pMIA8YX3lwy3uPMownDwdYISBdmAF/2holEIldVp85Wag==",
      "path": "system.dynamic.runtime/4.3.0",
      "hashPath": "system.dynamic.runtime.4.3.0.nupkg.sha512"
    },
    "System.Globalization/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-kYdVd2f2PAdFGblzFswE4hkNANJBKRmsfa2X5LG2AcWE1c7/4t0pYae1L8vfZ5xvE2nK/R9JprtToA61OSHWIg==",
      "path": "system.globalization/4.3.0",
      "hashPath": "system.globalization.4.3.0.nupkg.sha512"
    },
    "System.Globalization.Calendars/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GUlBtdOWT4LTV3I+9/PJW+56AnnChTaOqqTLFtdmype/L500M2LIyXgmtd9X2P2VOkmJd5c67H5SaC2QcL1bFA==",
      "path": "system.globalization.calendars/4.3.0",
      "hashPath": "system.globalization.calendars.4.3.0.nupkg.sha512"
    },
    "System.Globalization.Extensions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FhKmdR6MPG+pxow6wGtNAWdZh7noIOpdD5TwQ3CprzgIE1bBBoim0vbR1+AWsWjQmU7zXHgQo4TWSP6lCeiWcQ==",
      "path": "system.globalization.extensions/4.3.0",
      "hashPath": "system.globalization.extensions.4.3.0.nupkg.sha512"
    },
    "System.IdentityModel.Tokens.Jwt/5.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-EdcMk+36u9gQtbwTiPQ7ckIfiADBwOmCZ6rGD2rfkaozIdW1t7vbXk/FPVAu2r9KgCQZ5245Z+P0YMM/0Q0G2g==",
      "path": "system.identitymodel.tokens.jwt/5.3.0",
      "hashPath": "system.identitymodel.tokens.jwt.5.3.0.nupkg.sha512"
    },
    "System.IO/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-3qjaHvxQPDpSOYICjUoTsmoq5u6QJAFRUITgeT/4gqkF1bajbSmb1kwSxEA8AHlofqgcKJcM8udgieRNhaJ5Cg==",
      "path": "system.io/4.3.0",
      "hashPath": "system.io.4.3.0.nupkg.sha512"
    },
    "System.IO.Compression/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-YHndyoiV90iu4iKG115ibkhrG+S3jBm8Ap9OwoUAzO5oPDAWcr0SFwQFm0HjM8WkEZWo0zvLTyLmbvTkW1bXgg==",
      "path": "system.io.compression/4.3.0",
      "hashPath": "system.io.compression.4.3.0.nupkg.sha512"
    },
    "System.IO.FileSystem/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-3wEMARTnuio+ulnvi+hkRNROYwa1kylvYahhcLk4HSoVdl+xxTFVeVlYOfLwrDPImGls0mDqbMhrza8qnWPTdA==",
      "path": "system.io.filesystem/4.3.0",
      "hashPath": "system.io.filesystem.4.3.0.nupkg.sha512"
    },
    "System.IO.FileSystem.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-6QOb2XFLch7bEc4lIcJH49nJN2HV+OC3fHDgsLVsBVBk3Y4hFAnOBGzJ2lUu7CyDDFo9IBWkSsnbkT6IBwwiMw==",
      "path": "system.io.filesystem.primitives/4.3.0",
      "hashPath": "system.io.filesystem.primitives.4.3.0.nupkg.sha512"
    },
    "System.IO.Pipelines/4.5.2": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-NOC/SO4gSX6t0tB25xxDPqPEzkksuzW7NVFBTQGAkjXXUPQl7ZtyE83T7tUCP2huFBbPombfCKvq1Ox1aG8D9w==",
      "path": "system.io.pipelines/4.5.2",
      "hashPath": "system.io.pipelines.4.5.2.nupkg.sha512"
    },
    "System.Linq/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-5DbqIUpsDp0dFftytzuMmc0oeMdQwjcP/EWxsksIz/w1TcFRkZ3yKKz0PqiYFMmEwPSWw+qNVqD7PJ889JzHbw==",
      "path": "system.linq/4.3.0",
      "hashPath": "system.linq.4.3.0.nupkg.sha512"
    },
    "System.Linq.Expressions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-PGKkrd2khG4CnlyJwxwwaWWiSiWFNBGlgXvJpeO0xCXrZ89ODrQ6tjEWS/kOqZ8GwEOUATtKtzp1eRgmYNfclg==",
      "path": "system.linq.expressions/4.3.0",
      "hashPath": "system.linq.expressions.4.3.0.nupkg.sha512"
    },
    "System.Memory/4.5.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-sDJYJpGtTgx+23Ayu5euxG5mAXWdkDb4+b0rD0Cab0M1oQS9H0HXGPriKcqpXuiJDTV7fTp/d+fMDJmnr6sNvA==",
      "path": "system.memory/4.5.1",
      "hashPath": "system.memory.4.5.1.nupkg.sha512"
    },
    "System.Net.WebSockets.WebSocketProtocol/4.5.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-FquLjdb/0CeMqb15u9Px6TwnyFl306WztKWu6sKKc5kWPYMdpi5BFEkdxzGoieYFp9UksyGwJnCw4KKAUfJjrw==",
      "path": "system.net.websockets.websocketprotocol/4.5.1",
      "hashPath": "system.net.websockets.websocketprotocol.4.5.1.nupkg.sha512"
    },
    "System.Numerics.Vectors/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-QQTlPTl06J/iiDbJCiepZ4H//BVraReU4O4EoRw1U02H5TLUIT7xn3GnDp9AXPSlJUDyFs4uWjWafNX6WrAojQ==",
      "path": "system.numerics.vectors/4.5.0",
      "hashPath": "system.numerics.vectors.4.5.0.nupkg.sha512"
    },
    "System.ObjectModel/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-bdX+80eKv9bN6K4N+d77OankKHGn6CH711a6fcOpMQu2Fckp/Ft4L/kW9WznHpyR0NRAvJutzOMHNNlBGvxQzQ==",
      "path": "system.objectmodel/4.3.0",
      "hashPath": "system.objectmodel.4.3.0.nupkg.sha512"
    },
    "System.Private.DataContractSerialization/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-yDaJ2x3mMmjdZEDB4IbezSnCsnjQ4BxinKhRAaP6kEgL6Bb6jANWphs5SzyD8imqeC/3FxgsuXT6ykkiH1uUmA==",
      "path": "system.private.datacontractserialization/4.3.0",
      "hashPath": "system.private.datacontractserialization.4.3.0.nupkg.sha512"
    },
    "System.Reflection/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-KMiAFoW7MfJGa9nDFNcfu+FpEdiHpWgTcS2HdMpDvt9saK3y/G4GwprPyzqjFH9NTaGPQeWNHU+iDlDILj96aQ==",
      "path": "system.reflection/4.3.0",
      "hashPath": "system.reflection.4.3.0.nupkg.sha512"
    },
    "System.Reflection.Emit/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-228FG0jLcIwTVJyz8CLFKueVqQK36ANazUManGaJHkO0icjiIypKW7YLWLIWahyIkdh5M7mV2dJepllLyA1SKg==",
      "path": "system.reflection.emit/4.3.0",
      "hashPath": "system.reflection.emit.4.3.0.nupkg.sha512"
    },
    "System.Reflection.Emit.ILGeneration/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-59tBslAk9733NXLrUJrwNZEzbMAcu8k344OYo+wfSVygcgZ9lgBdGIzH/nrg3LYhXceynyvTc8t5/GD4Ri0/ng==",
      "path": "system.reflection.emit.ilgeneration/4.3.0",
      "hashPath": "system.reflection.emit.ilgeneration.4.3.0.nupkg.sha512"
    },
    "System.Reflection.Emit.Lightweight/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-oadVHGSMsTmZsAF864QYN1t1QzZjIcuKU3l2S9cZOwDdDueNTrqq1yRj7koFfIGEnKpt6NjpL3rOzRhs4ryOgA==",
      "path": "system.reflection.emit.lightweight/4.3.0",
      "hashPath": "system.reflection.emit.lightweight.4.3.0.nupkg.sha512"
    },
    "System.Reflection.Extensions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-rJkrJD3kBI5B712aRu4DpSIiHRtr6QlfZSQsb0hYHrDCZORXCFjQfoipo2LaMUHoT9i1B7j7MnfaEKWDFmFQNQ==",
      "path": "system.reflection.extensions/4.3.0",
      "hashPath": "system.reflection.extensions.4.3.0.nupkg.sha512"
    },
    "System.Reflection.Metadata/1.6.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-COC1aiAJjCoA5GBF+QKL2uLqEBew4JsCkQmoHKbN3TlOZKa2fKLz5CpiRQKDz0RsAOEGsVKqOD5bomsXq/4STQ==",
      "path": "system.reflection.metadata/1.6.0",
      "hashPath": "system.reflection.metadata.1.6.0.nupkg.sha512"
    },
    "System.Reflection.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-5RXItQz5As4xN2/YUDxdpsEkMhvw3e6aNveFXUn4Hl/udNTCNhnKp8lT9fnc3MhvGKh1baak5CovpuQUXHAlIA==",
      "path": "system.reflection.primitives/4.3.0",
      "hashPath": "system.reflection.primitives.4.3.0.nupkg.sha512"
    },
    "System.Reflection.TypeExtensions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7u6ulLcZbyxB5Gq0nMkQttcdBTx57ibzw+4IOXEfR+sXYQoHvjW5LTLyNr8O22UIMrqYbchJQJnos4eooYzYJA==",
      "path": "system.reflection.typeextensions/4.3.0",
      "hashPath": "system.reflection.typeextensions.4.3.0.nupkg.sha512"
    },
    "System.Resources.ResourceManager/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-/zrcPkkWdZmI4F92gL/TPumP98AVDu/Wxr3CSJGQQ+XN6wbRZcyfSKVoPo17ilb3iOr0cCRqJInGwNMolqhS8A==",
      "path": "system.resources.resourcemanager/4.3.0",
      "hashPath": "system.resources.resourcemanager.4.3.0.nupkg.sha512"
    },
    "System.Runtime/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-JufQi0vPQ0xGnAczR13AUFglDyVYt4Kqnz1AZaiKZ5+GICq0/1MH/mO/eAJHt/mHW1zjKBJd7kV26SrxddAhiw==",
      "path": "system.runtime/4.3.0",
      "hashPath": "system.runtime.4.3.0.nupkg.sha512"
    },
    "System.Runtime.Extensions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-guW0uK0fn5fcJJ1tJVXYd7/1h5F+pea1r7FLSOz/f8vPEqbR2ZAknuRDvTQ8PzAilDveOxNjSfr0CHfIQfFk8g==",
      "path": "system.runtime.extensions/4.3.0",
      "hashPath": "system.runtime.extensions.4.3.0.nupkg.sha512"
    },
    "System.Runtime.Handles/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-OKiSUN7DmTWeYb3l51A7EYaeNMnvxwE249YtZz7yooT4gOZhmTjIn48KgSsw2k2lYdLgTKNJw/ZIfSElwDRVgg==",
      "path": "system.runtime.handles/4.3.0",
      "hashPath": "system.runtime.handles.4.3.0.nupkg.sha512"
    },
    "System.Runtime.InteropServices/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-uv1ynXqiMK8mp1GM3jDqPCFN66eJ5w5XNomaK2XD+TuCroNTLFGeZ+WCmBMcBDyTFKou3P6cR6J/QsaqDp7fGQ==",
      "path": "system.runtime.interopservices/4.3.0",
      "hashPath": "system.runtime.interopservices.4.3.0.nupkg.sha512"
    },
    "System.Runtime.InteropServices.RuntimeInformation/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-cbz4YJMqRDR7oLeMRbdYv7mYzc++17lNhScCX0goO2XpGWdvAt60CGN+FHdePUEHCe/Jy9jUlvNAiNdM+7jsOw==",
      "path": "system.runtime.interopservices.runtimeinformation/4.3.0",
      "hashPath": "system.runtime.interopservices.runtimeinformation.4.3.0.nupkg.sha512"
    },
    "System.Runtime.Numerics/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-yMH+MfdzHjy17l2KESnPiF2dwq7T+xLnSJar7slyimAkUh/gTrS9/UQOtv7xarskJ2/XDSNvfLGOBQPjL7PaHQ==",
      "path": "system.runtime.numerics/4.3.0",
      "hashPath": "system.runtime.numerics.4.3.0.nupkg.sha512"
    },
    "System.Runtime.Serialization.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Wz+0KOukJGAlXjtKr+5Xpuxf8+c8739RI1C+A2BoQZT+wMCCoMDDdO8/4IRHfaVINqL78GO8dW8G2lW/e45Mcw==",
      "path": "system.runtime.serialization.primitives/4.3.0",
      "hashPath": "system.runtime.serialization.primitives.4.3.0.nupkg.sha512"
    },
    "System.Runtime.Serialization.Xml/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-nUQx/5OVgrqEba3+j7OdiofvVq9koWZAC7Z3xGI8IIViZqApWnZ5+lLcwYgTlbkobrl/Rat+Jb8GeD4WQESD2A==",
      "path": "system.runtime.serialization.xml/4.3.0",
      "hashPath": "system.runtime.serialization.xml.4.3.0.nupkg.sha512"
    },
    "System.Security.AccessControl/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-vW8Eoq0TMyz5vAG/6ce483x/CP83fgm4SJe5P8Tb1tZaobcvPrbMEL7rhH1DRdrYbbb6F0vq3OlzmK0Pkwks5A==",
      "path": "system.security.accesscontrol/4.5.0",
      "hashPath": "system.security.accesscontrol.4.5.0.nupkg.sha512"
    },
    "System.Security.Claims/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-P/+BR/2lnc4PNDHt/TPBAWHVMLMRHsyYZbU1NphW4HIWzCggz8mJbTQQ3MKljFE7LS3WagmVFuBgoLcFzYXlkA==",
      "path": "system.security.claims/4.3.0",
      "hashPath": "system.security.claims.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Algorithms/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-W1kd2Y8mYSCgc3ULTAZ0hOP2dSdG5YauTb1089T0/kRcN2MpSAW1izOFROrJgxSlMn3ArsgHXagigyi+ibhevg==",
      "path": "system.security.cryptography.algorithms/4.3.0",
      "hashPath": "system.security.cryptography.algorithms.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Cng/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-WG3r7EyjUe9CMPFSs6bty5doUqT+q9pbI80hlNzo2SkPkZ4VTuZkGWjpp77JB8+uaL4DFPRdBsAY+DX3dBK92A==",
      "path": "system.security.cryptography.cng/4.5.0",
      "hashPath": "system.security.cryptography.cng.4.5.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Csp/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-X4s/FCkEUnRGnwR3aSfVIkldBmtURMhmexALNTwpjklzxWU7yjMk7GHLKOZTNkgnWnE0q7+BCf9N2LVRWxewaA==",
      "path": "system.security.cryptography.csp/4.3.0",
      "hashPath": "system.security.cryptography.csp.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Encoding/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-1DEWjZZly9ae9C79vFwqaO5kaOlI5q+3/55ohmq/7dpDyDfc8lYe7YVxJUZ5MF/NtbkRjwFRo14yM4OEo9EmDw==",
      "path": "system.security.cryptography.encoding/4.3.0",
      "hashPath": "system.security.cryptography.encoding.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.OpenSsl/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-h4CEgOgv5PKVF/HwaHzJRiVboL2THYCou97zpmhjghx5frc7fIvlkY1jL+lnIQyChrJDMNEXS6r7byGif8Cy4w==",
      "path": "system.security.cryptography.openssl/4.3.0",
      "hashPath": "system.security.cryptography.openssl.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Pkcs/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-TGQX51gxpY3K3I6LJlE2LAftVlIMqJf0cBGhz68Y89jjk3LJCB6SrwiD+YN1fkqemBvWGs+GjyMJukl6d6goyQ==",
      "path": "system.security.cryptography.pkcs/4.5.0",
      "hashPath": "system.security.cryptography.pkcs.4.5.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Primitives/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-7bDIyVFNL/xKeFHjhobUAQqSpJq9YTOpbEs6mR233Et01STBMXNAc/V+BM6dwYGc95gVh/Zf+iVXWzj3mE8DWg==",
      "path": "system.security.cryptography.primitives/4.3.0",
      "hashPath": "system.security.cryptography.primitives.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.X509Certificates/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-t2Tmu6Y2NtJ2um0RtcuhP7ZdNNxXEgUm2JeoA/0NvlMjAhKCnM1NX07TDl3244mVp3QU6LPEhT3HTtH1uF7IYw==",
      "path": "system.security.cryptography.x509certificates/4.3.0",
      "hashPath": "system.security.cryptography.x509certificates.4.3.0.nupkg.sha512"
    },
    "System.Security.Cryptography.Xml/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-i2Jn6rGXR63J0zIklImGRkDIJL4b1NfPSEbIVHBlqoIb12lfXIigCbDRpDmIEzwSo/v1U5y/rYJdzZYSyCWxvg==",
      "path": "system.security.cryptography.xml/4.5.0",
      "hashPath": "system.security.cryptography.xml.4.5.0.nupkg.sha512"
    },
    "System.Security.Permissions/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-9gdyuARhUR7H+p5CjyUB/zPk7/Xut3wUSP8NJQB6iZr8L3XUXTMdoLeVAg9N4rqF8oIpE7MpdqHdDHQ7XgJe0g==",
      "path": "system.security.permissions/4.5.0",
      "hashPath": "system.security.permissions.4.5.0.nupkg.sha512"
    },
    "System.Security.Principal/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-I1tkfQlAoMM2URscUtpcRo/hX0jinXx6a/KUtEQoz3owaYwl3qwsO8cbzYVVnjxrzxjHo3nJC+62uolgeGIS9A==",
      "path": "system.security.principal/4.3.0",
      "hashPath": "system.security.principal.4.3.0.nupkg.sha512"
    },
    "System.Security.Principal.Windows/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-U77HfRXlZlOeIXd//Yoj6Jnk8AXlbeisf1oq1os+hxOGVnuG+lGSfGqTwTZBoORFF6j/0q7HXIl8cqwQ9aUGqQ==",
      "path": "system.security.principal.windows/4.5.0",
      "hashPath": "system.security.principal.windows.4.5.0.nupkg.sha512"
    },
    "System.Text.Encoding/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-BiIg+KWaSDOITze6jGQynxg64naAPtqGHBwDrLaCtixsa5bKiR8dpPOHA7ge3C0JJQizJE+sfkz1wV+BAKAYZw==",
      "path": "system.text.encoding/4.3.0",
      "hashPath": "system.text.encoding.4.3.0.nupkg.sha512"
    },
    "System.Text.Encoding.Extensions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-YVMK0Bt/A43RmwizJoZ22ei2nmrhobgeiYwFzC4YAN+nue8RF6djXDMog0UCn+brerQoYVyaS+ghy9P/MUVcmw==",
      "path": "system.text.encoding.extensions/4.3.0",
      "hashPath": "system.text.encoding.extensions.4.3.0.nupkg.sha512"
    },
    "System.Text.Encodings.Web/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-Xg4G4Indi4dqP1iuAiMSwpiWS54ZghzR644OtsRCm/m/lBMG8dUBhLVN7hLm8NNrNTR+iGbshCPTwrvxZPlm4g==",
      "path": "system.text.encodings.web/4.5.0",
      "hashPath": "system.text.encodings.web.4.5.0.nupkg.sha512"
    },
    "System.Text.RegularExpressions/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-RpT2DA+L660cBt1FssIE9CAGpLFdFPuheB7pLpKpn6ZXNby7jDERe8Ua/Ne2xGiwLVG2JOqziiaVCGDon5sKFA==",
      "path": "system.text.regularexpressions/4.3.0",
      "hashPath": "system.text.regularexpressions.4.3.0.nupkg.sha512"
    },
    "System.Threading/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-VkUS0kOBcUf3Wwm0TSbrevDDZ6BlM+b/HRiapRFWjM5O0NS0LviG0glKmFK+hhPDd1XFeSdU1GmlLhb2CoVpIw==",
      "path": "system.threading/4.3.0",
      "hashPath": "system.threading.4.3.0.nupkg.sha512"
    },
    "System.Threading.Channels/4.5.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-MEH06N0rIGmRT4LOKQ2BmUO0IxfvmIY/PaouSq+DFQku72OL8cxfw8W99uGpTCFf2vx2QHLRSh374iSM3asdTA==",
      "path": "system.threading.channels/4.5.0",
      "hashPath": "system.threading.channels.4.5.0.nupkg.sha512"
    },
    "System.Threading.Tasks/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-LbSxKEdOUhVe8BezB/9uOGGppt+nZf6e1VFyw6v3DN6lqitm0OSn2uXMOdtP0M3W4iMcqcivm2J6UgqiwwnXiA==",
      "path": "system.threading.tasks/4.3.0",
      "hashPath": "system.threading.tasks.4.3.0.nupkg.sha512"
    },
    "System.Threading.Tasks.Extensions/4.5.1": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-WSKUTtLhPR8gllzIWO2x6l4lmAIfbyMAiTlyXAis4QBDonXK4b4S6F8zGARX4/P8wH3DH+sLdhamCiHn+fTU1A==",
      "path": "system.threading.tasks.extensions/4.5.1",
      "hashPath": "system.threading.tasks.extensions.4.5.1.nupkg.sha512"
    },
    "System.Threading.Tasks.Parallel/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-cbjBNZHf/vQCfcdhzx7knsiygoCKgxL8mZOeocXZn5gWhCdzHIq6bYNKWX0LAJCWYP7bds4yBK8p06YkP0oa0g==",
      "path": "system.threading.tasks.parallel/4.3.0",
      "hashPath": "system.threading.tasks.parallel.4.3.0.nupkg.sha512"
    },
    "System.Threading.Thread/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-OHmbT+Zz065NKII/ZHcH9XO1dEuLGI1L2k7uYss+9C1jLxTC9kTZZuzUOyXHayRk+dft9CiDf3I/QZ0t8JKyBQ==",
      "path": "system.threading.thread/4.3.0",
      "hashPath": "system.threading.thread.4.3.0.nupkg.sha512"
    },
    "System.ValueTuple/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-cNLEvBX3d6MMQRZe3SMFNukVbitDAEpVZO17qa0/2FHxZ7Y7PpFRpr6m2615XYM/tYYYf0B+WyHNujqIw8Luwg==",
      "path": "system.valuetuple/4.3.0",
      "hashPath": "system.valuetuple.4.3.0.nupkg.sha512"
    },
    "System.Xml.ReaderWriter/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-GrprA+Z0RUXaR4N7/eW71j1rgMnEnEVlgii49GZyAjTH7uliMnrOU3HNFBr6fEDBCJCIdlVNq9hHbaDR621XBA==",
      "path": "system.xml.readerwriter/4.3.0",
      "hashPath": "system.xml.readerwriter.4.3.0.nupkg.sha512"
    },
    "System.Xml.XDocument/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-5zJ0XDxAIg8iy+t4aMnQAu0MqVbqyvfoUVl1yDV61xdo3Vth45oA2FoY4pPkxYAH5f8ixpmTqXeEIya95x0aCQ==",
      "path": "system.xml.xdocument/4.3.0",
      "hashPath": "system.xml.xdocument.4.3.0.nupkg.sha512"
    },
    "System.Xml.XmlDocument/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-lJ8AxvkX7GQxpC6GFCeBj8ThYVyQczx2+f/cWHJU8tjS7YfI6Cv6bon70jVEgs2CiFbmmM8b9j1oZVx0dSI2Ww==",
      "path": "system.xml.xmldocument/4.3.0",
      "hashPath": "system.xml.xmldocument.4.3.0.nupkg.sha512"
    },
    "System.Xml.XmlSerializer/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-MYoTCP7EZ98RrANESW05J5ZwskKDoN0AuZ06ZflnowE50LTpbR5yRg3tHckTVm5j/m47stuGgCrCHWePyHS70Q==",
      "path": "system.xml.xmlserializer/4.3.0",
      "hashPath": "system.xml.xmlserializer.4.3.0.nupkg.sha512"
    },
    "System.Xml.XPath/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-v1JQ5SETnQusqmS3RwStF7vwQ3L02imIzl++sewmt23VGygix04pEH+FCj1yWb+z4GDzKiljr1W7Wfvrx0YwgA==",
      "path": "system.xml.xpath/4.3.0",
      "hashPath": "system.xml.xpath.4.3.0.nupkg.sha512"
    },
    "System.Xml.XPath.XDocument/4.3.0": {
      "type": "package",
      "serviceable": true,
      "sha512": "sha512-jw9oHHEIVW53mHY9PgrQa98Xo2IZ0ZjrpdOTmtvk+Rvg4tq7dydmxdNqUvJ5YwjDqhn75mBXWttWjiKhWP53LQ==",
      "path": "system.xml.xpath.xdocument/4.3.0",
      "hashPath": "system.xml.xpath.xdocument.4.3.0.nupkg.sha512"
    }
  }
}
```