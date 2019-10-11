# How to build a Nuget that includes referenced DLLS
As the Nuget documentation states

*"If your library is composed of multiple assemblies that aren't independently useful, then it's fine to combine them into one package. Using the previous example, if Parser.dll contains code that's used only by Utilities.dll, then it's fine to keep Parser.dll in the same package."*

That makes sense but the documentation doesn't come straight out and tell you how to do this. If you are reading the dotnet cli documenation it is even more confusing. It need not be.  Here's an example of how to do it.

There are three projects in this solution. There is *nulib* which is the nuget package class library. There is "inclib" which class library that nulib depends upon. Finally their is nutest, a simple console app that will use a class and method exposed by nulib which will in turn use a class and method exposed in inclib. The result is written to the console and simply says "Something".

Start buy simply developing and testing your class libraries in your project. Use a project reference to include use inclib inside of nulib in the usual way like so:

`  <ItemGroup>
    <ProjectReference Include="..\inclib\inclib.csproj" />
  </ItemGroup>`

The next step is to create a nuspec file. In theory we should be able to use the Nuget metadata in the .csproj file for nulib, but it seems to be missing the ability to define files. So run the following command in the folder containing the nulib project:

`nuget spec`

This creates a file called nulib.nuspec which will look like this
~~~~
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2019</copyright>
    <tags>Tag1 Tag2</tags>
  </metadata>
</package>
~~~~

This is just a template. Fill in the necessary details and add a list of files that you want included in your nuget package. The result looks like this:

~~~~
<?xml version="1.0"?>
<package >
  <metadata>
    <id>test.nulib</id>
    <title>Test package</title>
    <version>1.0.0</version>
    <authors>Glenn</authors>
    <owners>Glenn</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Just testing including multipe assemblies</description>
    <copyright>Copyright 2019</copyright>
    <tags>Totaly worthless</tags>
  </metadata>
  <files>
    <file src="bin\Debug\netstandard2.0\*.*" target="lib\netstandard2.0" />
    <file src="..\inclib\bin\Debug\netstandard2.0\*.*" target="lib\netstandard2.0" />
  </files>
</package>
~~~~

At this point we can create the nuget package itself by running this command in the folder containing the nulib project. I'm using the dotnet cli for this so that we don't need to add the nuget executable to our build machines.

`dotnet pack /p:NuspecFile=nulib.nuspec`

The result will be a test.nulib.1.0.0.nupkg being created. If, like most people, you need to version your nuget packages a build time in a CI/CD process, you will need to edit version element before running the pack command.

I tested this by pushing my nuget package to an internal repository and adding it to nutet in the normal way

`   <ItemGroup>
    <PackageReference Include="test.nulib" Version="1.0.0" />
  </ItemGroup>`


Also, it's a good idea to examine your nuget package with the Nuget Package Explorer which is available on the Microsoft Store app for free. 
