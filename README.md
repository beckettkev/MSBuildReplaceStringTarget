# MSBuildReplaceStringTarget
Sample Targets for MSBuild for Replacing Strings in MSBuild.


	<PropertyGroup>
		<MSBuildCommunityTasksPath>$(MSBuildProjectDirectory)\..\.build</MSBuildCommunityTasksPath>
	</PropertyGroup>
	<Import Project="$(MSBuildCommunityTasksPath)\MSBuild.Community.Tasks.targets" />
	
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
		<FileUpdate Files="@(MasterPageFiles)" Regex="AzureUrlBasePath" ReplacementText="@(FetchedUrlBaseValue)" />
		<Message Text="AzureUrlBasePath to @(FetchedUrlBaseValue) Replacement..." />
	</Target>
