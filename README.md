# webdeploy-umbraco

For deploying Umbraco we use the web deploy functionality of Microsoft. Because we use it mainly for Umbraco V7 and V8 we can't remove all the files in the root directory. We are stuck at FTP access to remove old files like assets and assemblies that aren't needed anymore. The solution could be to make the Media folder and App_Data folder a virtual folder on another part of the disk on the server. But this means moving files in production environments and configuration changes for the clients and possible issues with the environment. 

So if you're setting up the first time for Umbraco keep in mind how to set up your IIS instance. But the other solution is to make some changes in your deployment profile. Here you can specify to keep all files on the server, but we want only files from our GIT repository and one exclusion of the Media folder. You can do that by adding skip rules for deployment.

Its called MsDeploySkipRules, for Microsoft reference to the Documentation you can read it here:
https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/visual-studio-publish-profiles?view=aspnetcore-7.0

For a Umbraco we use the following Deployment Script that seems to be working.
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
```
<AfterAddIisSettingAndFileContentsToSourceManifest>AddCustomSkipRules</AfterAddIisSettingAndFileContentsToSourceManifest> <!-- Required to add skip rules-->
```
