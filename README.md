AntFlexLib: Ant library for Flex Applications 
=============================================


AntFlexLib is an library for ANT dedicated for Flex application. AntFlexLib is presented as xml, the language use for ANT build files.


This library handles: 
 	_ asdoc generation
	- unit tests (Flexunit)
	- prod/dev environments
	- build history
	- swf / swc compilation	


It is build as a real library using macrodef feature which will made AntFlexLib easy to use into a continuous integration process or simply for a application that use source versionning control.


Workspace
---------

The only constraint for AntFlexLib is a specific project organization. However this one correspond to those generaly used (it is compatible with Flash Builder projects). This one is the following:

/MyApplication
	/build/
	/config/	
	/src/
	/bin/
	/docs/
	/tests/
	

You can handle sub components or several top applications using the same build:


/MyApplication
	/build/
	/main1/	
		/bin/		
		/config/
		/docs/
		/src/
		/tests/		
	/main2/
		/bin/
		/config/
		/docs/
		/src/
		/tests/				
	/bin/
	/libs/
		/lib1/
			/bin/
			/config/
			/docs/
			/src/
			/tests/				

Directory details
-----------------


build: contains the build information:
		- build.xml: (any name is ok) build ant file to call using the ant program
		- build.properties: local properties of the application. Use to define the environment (dev or prod) ant the ant tasks file to use (however, by default AntFlexLib contains the tasks files to prevend from dependencies issues)
		
config: flex configuration files:
			- app.xml flex compilation configuration files
			- doc.xml flex asdoc configuration files

doc: generated asdoc.The asdoc is automaticaly ziped to prevend from polluting the workspace with a lot of files (that generaly pollutate also the version control system). can be uzipped on the integration server

src: sources of the application. Should contains at least the mxml file corresponding to the application name givent to AntFlexLib.


AntFlexLib functions
--------------------


"functions" are macrodef.



prepare-swf ({@name}, {@path}): handle the deployment of a swf for the project {@name} into {@path}
				- perform the unit tests if they exists (tests directory exists)
				- generate the documentation (doc directory exists)
				- compile the swf
				- generate the build.info into {@path}
				
				if the tests failed, stop the process

				
prepare-swc ({@name}, {@path}): same as prepare-swf but for swc



dispose-bin ({@path}): delete all intermediary file generated at compilation (link-report, cache). Should be call before deploying to prevend from links issues



The following methods are utils method used by the two main methods (prepare-swf and prepare-swc)


build-info ({@path}): generate a build info file into {@path}/build.info
tests ({@path}): perform unit tests of {@path} found into {@path}/tests/ (flexunit)
docs ({@path}): generate the documention of {@path} into {@path}/docs/
ccompc ({@name}, {@path}): generate the swc  {@path}/bin/{@name}.swc
cmxmlc ({@name}, {@path}): generate the swf {@path}/bin/{@name}.swf
prepare-bin ({@path}): generate sequentially docs and tests



differences between prod and dev environment
--------------------------------------------


compilation configuration
	- use {$ANTFLEXLIB_HOME}/config.{$build.mode}.xml (see the files for details)
	- always generate documentation and perform tests in prod mode
	- pass by documentation and tests as default behavior in dev mode. Possiblity of forcing these behaviors using force.tests and/or force.docs into local build.properties file
	- generate the build.info file in prod mode


Example of AntFlexLib:



~/foobar/build/build.properties:

ANTFLEXLIB_HOME = /usr/share/antflexlib/
FLEX_HOME = /usr/share/flex/

build.mode = dev
force.tests = true
force.docs = false


~/foobar/build/build.xml:



<?xml version="1.0" encoding="UTF-8"?>
<project basedir="./"  name="FooBar Make" default="FooBar">

	<property file="./build.properties" />
	<import file="${ANTFLEXLIB_HOME}build/build.lib.xml" />

	<target name="Foo">
		<prepare-swc path="../lib/foo/" name="hello" />
	</target>

	<target name="Bar">
		<prepare-swc path="../lib/bar/" name="word" />
	</target>
	
	<target name="FooBar" depends="DisposeAll, Hello, Word">
		<prepare-swf path="../" name="helloword" />
	</target>

	<target name="DisposeAll">
		<echo message="dispose all builds" />

		<dispose-bin path="../lib/foo/" />
		<dispose-bin path="../lib/bar/" />
		<dispose-bin path="../" />
	</target>
</project>

~/foobar/config/app.xml

<?xml version="1.0" encoding="utf-8"?>
<flex-config>
	<!--Compiler options -->
	<compiler>
		<source-path append="true">
			<path-element>../src/</path-element>
		</source-path>
		<library-path append="true">
			<path-element>../../lib/foo/bin/</path-element>
			<path-element>../../lib/bar/bin/</path-element>
		</library-path>
	</compiler>
	
	<!--insert here all link reports no to recompile 
	<load-externs></load-externs>
	-->
</flex-config>


~/foobar/config/docs.xml

<?xml version="1.0" encoding="utf-8"?>
<flex-config xmlns="http://www.adobe.com/2006/flex-config">
	<compiler>
		<source-path append="true">
			<path-element>../src/</path-element>
		</source-path>
		<library-path append="true">
			<path-element>../../lib/foo/bin/</path-element>
			<path-element>../../lib/bar/bin/</path-element>
		</library-path>
	</compiler>
	<doc-sources>
		<!--  path is relative to the main ant script -->
		<path-element>../main/src/</path-element>
	</doc-sources>
</flex-config>



then just run:
ant ~/foobar/build/build.xml