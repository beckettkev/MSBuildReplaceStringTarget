#Website String Replacement with MSBuild

Sample Targets for MSBuild for Replacing Strings within website files in MSBuild.

First of all, if you haven't done so already, download the MSBuildTasks

  PM> Install-Package MSBuildTasks

Now we need to add a new xml file called Url_Replacements.xml to your SharePoint project, in a sub folder called _Replacements:

	<Replacements>
		<Environment Name="myproject-dev - Web Deploy">
			<UrlBase>https://MyProjectDevAssets.azurewebsites.net</UrlBase>
		</Environment>
		<Environment Name="myproject-test - Web Deploy">
			<UrlBase>https://MyProjectTestAssets.azurewebsites.net</UrlBase>
		</Environment>
	</Replacements>

Obviously amend the Urls in this file to map to the relevant environment base urls for your project. The next stage is to put a special string token in each file, where you want the replacement to take place (e.g. MyProjectUrlBasePath):

	...
	  <link type="text/css" rel="stylesheet" href="MyProjectUrlBasePath/Assets/styles/projstyles_core" />
  	...

Now unload the project by right clicking on it in Visual Studio and then Right click again and edit the *.csproj file to include the following four lines underneath the existing PropertyGroup region:

	<PropertyGroup>
		<MSBuildCommunityTasksPath>$(MSBuildProjectDirectory)\..\.build</MSBuildCommunityTasksPath>
	</PropertyGroup>
	<Import Project="$(MSBuildCommunityTasksPath)\MSBuild.Community.Tasks.targets" />

Now you need to add the Target. The Target runs at the correct point in the SharePoint publishing process to replace the special string value with the base url for the given environment that you are publishing for (so for example, to replace values in views only):

	<Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\Web\Microsoft.Web.Publishing.targets" Condition="false" />
	<Target Name="UrlReplacementsAfterCopy" AfterTargets="CopyAllFilesToSingleFolderForMsdeploy">
		<PropertyGroup>
			<ReplacementsFile>_Replacements\Url_Replacements.xml</ReplacementsFile>
		</PropertyGroup>
		<ItemGroup>
			<ViewFiles Include="$(MSBuildProjectDirectory)\$(_PackageTempDir)\**\*.cshtml" />
		</ItemGroup>

		<Message Text="Removing directory read only locks: '$(MSBuildProjectDirectory)\$(_PackageTempDir)'" Importance="high" />
		<!-- Remove file locks -->
		<Exec Command="attrib -R &quot;$(MSBuildProjectDirectory)\$(_PackageTempDir)&quot;" />
		
		<!-- Fetch the appropriate value from the environments.xml file.. -->
		<Message Text="Going to look for environment node publishing for '$(PublishProfileName)' in file $(ReplacementsFile).." Importance="high" />
		<XmlPeek XmlInputPath="$(ReplacementsFile)" Query="Replacements/Environment[@Name='$(PublishProfileName)']/UrlBase/text()">
			<Output TaskParameter="Result" ItemName="FetchedUrlBaseValue" />
		</XmlPeek>
		<Message Text="Fetched Url Base value of @(FetchedUrlBaseValue) from $(ReplacementsFile) file" Importance="high" />
		<!-- Update all the assembly info files with generated version info -->
		<Message Text="Modifying Views of type &quot;$(MSBuildProjectDirectory)\$(_PackageTempDir)\**\*.cshtml&quot; with ReplacementText is &quot;@(FetchedUrlBaseValue)&quot;." Importance="high" />
		<FileUpdate Files="@(ViewFiles)" Regex="MyProjectUrlBasePath" ReplacementText="@(FetchedUrlBaseValue)" />
	</Target>
	
The important thing to notice here is we are pulling in the Microsoft.Web.Publishing.targets to allow us to hook into the web publish process (MS Deploy). Once the temporary files have been copied over in preperation for the publish, we can do our string replacement (AfterTargets="CopyAllFilesToSingleFolderForMsdeploy").

That's it, save the project file and reload it in visual studio.
