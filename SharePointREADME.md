# MSBuildReplaceStringTarget
Sample Targets for MSBuild for Replacing Strings within website files in MSBuild. 

First of all download the MSBuildTasks

	PM> Install-Package MSBuildTasks

Now we need to add a new xml file called Url_Replacements.xml to your SharePoint project, in a sub folder called _Replacements:

	<Replacements>
		<Environment Name="DevTenancy">
			<UrlBase>https://MyProjectDevAssets.azurewebsites.net</UrlBase>
		</Environment>
		<Environment Name="TestTenancy">
			<UrlBase>https://MyProjectTestAssets.azurewebsites.net</UrlBase>
		</Environment>
	</Replacements>

Obviously amend the Urls in this file to map to the relevant environment base urls for your project. The next stage is to put a special string token in each file, where you want the replacement to take place (e.g. MyProjectUrlBasePath):

	...
	<SharePoint:ScriptLink language="javascript" name="suitelinks.js" OnDemand="true" runat="server" Localizable="false"></SharePoint:ScriptLink>
	<SharePoint:CustomJSUrl runat="server"></SharePoint:CustomJSUrl>
	<SharePoint:SoapDiscoveryLink runat="server"></SharePoint:SoapDiscoveryLink>

	<link type="text/css" rel="stylesheet" href="MyProjectUrlBasePath/Assets/styles/projstyles_core" />
	...

Now unload the project by right clicking on it in Visual Studio and then Right click again and edit the *.csproj file to include the following four lines underneath the existing PropertyGroup region:

	<PropertyGroup>
		<MSBuildCommunityTasksPath>$(MSBuildProjectDirectory)\..\.build</MSBuildCommunityTasksPath>
	</PropertyGroup>
	<Import Project="$(MSBuildCommunityTasksPath)\MSBuild.Community.Tasks.targets" />
	
Now you need to add the Target. The Target runs at the correct point in the SharePoint publishing process to replace the special string value with the base url for the given environment that you are publishing for (so for example, to replace values in masterpages only):
	
	<Target Name="AfterLayout" Condition="$(Configuration) != 'Debug' AND $(Configuration) != 'Release'">
		<PropertyGroup>
			<ReplacementsFile>_Replacements\Url_Replacements.xml</ReplacementsFile>
		</PropertyGroup>
		<ItemGroup>
			<MasterPageFiles Include="$(LayoutPath)**\*.master" />
		</ItemGroup>
		
		<!-- Remove directory readonly locks -->
		<Exec Command="attrib -R &quot;$(LayoutPath)My.Project.Name&quot;" />
		
		<!-- Fetch the appropriate value from the environments.xml file.. -->
		<Message Text="Going to look for Environment node with Name ='$(Configuration)' in file $(ReplacementsFile).." Importance="high" />
		<XmlPeek XmlInputPath="$(ReplacementsFile)" Query="Replacements/Environment[@Name='$(Configuration)']/UrlBase/text()">
			<Output TaskParameter="Result" ItemName="FetchedUrlBaseValue" />
		</XmlPeek>
		<Message Text="Fetched Url Base value of @(FetchedUrlBaseValue) from $(ReplacementsFile) file" Importance="high" />
		<!-- Update all the assembly info files with generated version info -->
		<Message Text="Modifying MasterPages under &quot;$(LayoutPath)&quot;." />
		<FileUpdate Files="@(MasterPageFiles)" Regex="MyProjectUrlBasePath" ReplacementText="@(FetchedUrlBaseValue)" />
		<Message Text="MyProjectUrlBasePath to @(FetchedUrlBaseValue) Replacement..." />
	</Target>

Save the change and reload the project.
