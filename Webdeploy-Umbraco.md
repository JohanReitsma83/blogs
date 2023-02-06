# Cleanup your Umbraco Application with no FTP Access!

## Who are we? 
At my company, a web agency the .NET developers build custom websites based on Umbraco. All of our sites have a common configuration and libraries but are customized for every client of ours. We started using Umbraco 7 and now we support V10. The reason not to use V11 is mainly that it's not Lont Time Supported version, but that's a completely different story for a later blog.

## Why we don't want FTP access!
Our company's biggest issue with our Umbraco environments is FTP access for our Support employees and our Developers.

Mainly the issue is because some developers do changes using FTP on production without committing it back to version control, or without Pull Request validation and correct testing. And then with our official release mechanism using Azure Pipelines, the changes are gone or will break the application if a new release will be executed. The customer is angry and a developer needs to find out what happened. But no history :( or need to revert the wrong change that isn't tested. So this is why we want to remove access to our Website of the Umbraco Application by FTP.

If it's in version control it's in history and there is one truth!

## But how do you do this? 
There are a couple of solutions, for example, custom scripts, or creating a new instance (version folder online) for each deployment, but in our case, we want a single instance online so the server keeps clean. And that's why we use a tool from Microsoft called MS Web Deploy and a publish profile you can configure in visual studio. I'm not gone explain how to install it, but you can read more about this here:  https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/visual-studio-web-deployment/deploying-to-iis

With a publish profile we can deploy it using visual studio or by command line. We use the command line in our azure pipeline to publish to a specific profile. This is great, but a profile has 2 options for files and folders: .
- Keep files on disk that don't exist in the source control
- Remove all files that are on disk but not in source control.

And now the issue with our setup comes to play. We can keep all the files on disk, but what if a DLL is not used, it remains in your Bin folder and is still used in the startup of your project or can have a conflict. Or assets that are removed from source control still are available on disk on the server. A mess occurs :( But if we clean up everything automatically that's not in version control all our user-generated content will be removed. The Media for Umbraco for example, or form uploads.
In the past, we kept all the files on disk and could manually remove files using FTP. But this gives a developer the wrong access :(

# How we changed to remove all files but with an exclusion!
We could make the Media folder and App_Data folder a virtual folder on another part of the disk on the IIS server. But this means moving files in production environments and configuration changes for the clients and possible issues with the environment.

So if you're setting up the first time for Umbraco keep in mind how to set up your IIS instance. But the other solution is to make some changes in your deployment profile. Here you can specify to keep all files on the server, but we want only files from our GIT repository and one exclusion of the Media folder. You can do that by adding skip rules for deployment.

Its called MsDeploySkipRules, for Microsoft reference to the Documentation you can read it here: https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/visual-studio-publish-profiles?view=aspnetcore-7.0
For a Umbraco we use the following Deployment Script that seems to be working. I'm gonna explain what you need to do in the script for excluding some folder for removal.

## the script
```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
https://go.microsoft.com/fwlink/?LinkID=208121.
-->
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <AutoParameterizationWebConfigConnectionStrings>False</AutoParameterizationWebConfigConnectionStrings>
    <AfterAddIisSettingAndFileContentsToSourceManifest>AddCustomSkipRules</AfterAddIisSettingAndFileContentsToSourceManifest> <!-- Required to add skip rules-->
    <AllowUntrustedCertificate>True</AllowUntrustedCertificate> 
    <IncludeSetACLProviderOnDestination>False</IncludeSetACLProviderOnDestination> <!-- We don't set folder rights, this will remove our current folder rights if set to true -->
    <WebPublishMethod>MSDeploy</WebPublishMethod>
    <LastUsedBuildConfiguration>Release</LastUsedBuildConfiguration>
    <LastUsedPlatform>Any CPU</LastUsedPlatform>
    <SiteUrlToLaunchAfterPublish></SiteUrlToLaunchAfterPublish>
    <LaunchSiteAfterPublish>true</LaunchSiteAfterPublish>
    <ExcludeApp_Data>true</ExcludeApp_Data> <!--App Data can be excluded from deployment -->
    <MSDeployServiceURL>https://url-server/msdeploy.axd</MSDeployServiceURL> <!-- Url to server where you want to deploy to -->
    <DeployIisAppPath>exampledomain.com</DeployIisAppPath> <!-- We use domain names for our deployment target -->
    <RemoteSitePhysicalPath />
    <SkipExtraFilesOnServer>false</SkipExtraFilesOnServer> <!-- if set to false it will remove all files not present in the build -->
    <MSDeployPublishMethod>WMSVC</MSDeployPublishMethod>
    <EnableMSDeployBackup>true</EnableMSDeployBackup>
    <EnableMsDeployAppOffline>false</EnableMsDeployAppOffline>
  </PropertyGroup>
  <PropertyGroup> <!-- This looks like it is needed for the Custom Skip Rules -->
    <UseMsDeployExe>true</UseMsDeployExe>
  </PropertyGroup>
  <ItemGroup>
    <MSDeployParameterValue Include="$(DeployParameterPrefix)umbracoDbDSN-Web.config Connection String" />
  </ItemGroup>
  <Target Name="AddCustomSkipRules"> <!-- Here we define the custom skip rules-->
    <Message Text="Adding Custom Skip Rules" /> <!-- Apply a message to console -->
    <ItemGroup>
      <MsDeploySkipRules Include="SkipFilesInFilesFolder"> <!-- Skip File in A specific folder -->
        <SkipAction>Delete</SkipAction> <!-- Don't delete -->
        <ObjectName>filePath</ObjectName>
        <AbsolutePath>$(_DestinationContentPath)\\Media\\.*</AbsolutePath> <!--The root path of media -->
        <Apply>Destination</Apply>
      </MsDeploySkipRules>
      <MsDeploySkipRules Include="SkipFoldersInFilesFolders"> <!-- Here we define subfolders within media and what to do -->
        <SkipAction>
        </SkipAction>
        <ObjectName>dirPath</ObjectName> 
        <AbsolutePath>$(_DestinationContentPath)\\Media\\.*\\*</AbsolutePath>
        <Apply>Destination</Apply>
      </MsDeploySkipRules>
    </ItemGroup>
  </Target>
</Project>
```

the following rules are important:
This is required for setting skip rules, here you specify a unique name to use as a target.
```
<AfterAddIisSettingAndFileContentsToSourceManifest>AddCustomSkipRules</AfterAddIisSettingAndFileContentsToSourceManifest> <!-- Required to add skip rules-->
```


The following line is set to not override our permissions of the folders

```
<IncludeSetACLProviderOnDestination>False</IncludeSetACLProviderOnDestination>
```

App_Data can be excluded from cleaning, with the following rule
```
<ExcludeApp_Data>true</ExcludeApp_Data> <!--App Data can be excluded from deployment -->
```

We used to have this on True, but we want to have a clean enviroment because if we upgrade stuff or rename assemblies then you don't need to fix this on a server. So set it to False to keep it clean
```
<SkipExtraFilesOnServer>false</SkipExtraFilesOnServer>
```

In my case U needed to set this rule below for the skip rules to be used
```
  <PropertyGroup> <!-- This looks like it is needed for the Custom Skip Rules -->
    <UseMsDeployExe>true</UseMsDeployExe>
  </PropertyGroup>
  ```
  
And here we go with the skip rules, we define 2 rules, one for the root of the media folder and the second for all the subfolders within the media folder.

```
<Target Name="AddCustomSkipRules"> <!-- Here we define the custom skip rules-->
    <Message Text="Adding Custom Skip Rules" /> <!-- Apply a message to console -->
    <ItemGroup>
      <MsDeploySkipRules Include="SkipFilesInFilesFolder"> <!-- Skip File in A specific folder -->
        <SkipAction>Delete</SkipAction> <!-- Don't delete -->
        <ObjectName>filePath</ObjectName>
        <AbsolutePath>$(_DestinationContentPath)\\Media\\.*</AbsolutePath> <!--The root path of media -->
        <Apply>Destination</Apply>
      </MsDeploySkipRules>
      <MsDeploySkipRules Include="SkipFoldersInFilesFolders"> <!-- Here we define subfolders within media and what to do -->
        <SkipAction>
        </SkipAction>
        <ObjectName>dirPath</ObjectName> 
        <AbsolutePath>$(_DestinationContentPath)\\Media\\.*\\*</AbsolutePath>
        <Apply>Destination</Apply>
      </MsDeploySkipRules>
    </ItemGroup>
  </Target>
  ```
  
  And now we have a deployment profile that is cleaning all files that needs to be removed except user generated files. 
  
 Note: this is now only done for media, apply your own exclusions for license files and other files you don't have in source control.
 
