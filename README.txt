Ivy Integration Project

PURPOSE
=======

This project automatically integrates Ivy into your project. It will
add into your project the correct Ivy settings and add in all the extra
tools that you need for doing your builds. Right now these include:

* Checkstyle - Verifies your code is formatted to the standard Java
  coding standards.
* PMD - Reads through your source code for possible programming errors
* FindBugs - Reads through your class files for possible programming
  errors.
* CPD - Copy Paste Detector - Checks code for large amounts of copy
  and pasted code.

It also includes the Ant Contrib tasks
(http://ant-contrib.sourceforge.net). These extentions help make it
easier to do such things are loop through a set of files and includes
if/then/else logic.

This project was setup for Subversion. The idea is to create a Subversion
project (like trunk/ivy.dir), then have other projects use svn:externals to
include the ivy.dir project into their project. You can do something similar
with Perforce when creating a view. And Git sub-tree merges.

This project makes integration into Jenkins so much easier to do, and
the structure of this project works very closely with Jenkins. For
example, the Ivy cache has been redefined to allow each Jenkins executor
to have its own separate Ivy cache. This way, one build can clean the
cache without affecting a parallel build.

The newly defined macros for the Jars, Wars, and Ears automatically
embed the Jenkins build information into the MANIFEST.MF file (as well
as the Ivy version information. They also will produce a pom.xml file
from the ivy.xml file, so you can use mvn deploy:deploy-file to deploy
the built jars, wars, and ears bak into your Maven repository, and keep
it in Maven format. These commands will also embed the pom.xml into the
jars, wars, and ears under the META-INF directory just like Maven does.

AUTHOR
======

David Weintraub <mailto:david@weintraub.name>

COPYRIGHT
=========

Copyright (c) 2010 by David Weintraub. All rights reserved. This program
is covered by the open source BMAB license.

The BMAB (Buy me a beer) license allows you to use all code for whatever
reason you want with these three caveats:

1.  If you make any modifications in the code, please consider sending
    them to me, so I can put them into my code.

2.  Give me attribution and credit on this program.

3.  If you're in town, buy me a beer. Or, a cup of coffee which is what
    I'd prefer. Or, if you're feeling really spendthrify, you can buy me
    lunch. I promise to eat with my mouth closed and to use a napkin
    instead of my sleeves.

IMPLEMENTATION
==============

* Get this sub-project into your code. Define this package as a project
  inside your repository. Then use your repository's method of including
  subprojects into your checkout.
  * With Subversion, use the svn:externals property
  * With Git use the sub-tree merge 
    (https://help.github.com/articles/working-with-subtree-merge)
    or Git Submodules (http://git-scm.com/book/en/Git-Tools-Submodules).
  * With Perforce, define the Ivy.dir project as a sub-project of your view.

* Add in the following two lines at the beginning of your build.xml file:

    <property name="ivy.dir" value="${basedir}/ivy.dir"/>
    <import file="${basedir}"/>

* Add in the Ivy XML Namespace by putting this in your project line:

    <project name="project_name" default="package" basedir="."
	xmlns:ivy="http://ant.apache.org/ivy"/>

Ant that's all there is to it. The import will import the Ivy settings
and the various build tools that we use. The xmlns line will add in a
separate namespace for the Ivy tasks.

IVY TASKS
=========

The following Ivy tasks have been imported into your build. There are
other Ivy tasks, but these are the basic ones.

Ivy tasks are documented at
http://ant.apache.org/ivy/history/latest-milestone/ant.html

* <ivy:settings/> - Setup Ivy. This is handled by the import.

* <ivy:resolve/> - Resolves the Ivy dependencies and will download the
  jars into your $HOME directory for use. This must be the first Ivy
  task you call.

* <ivy:cachepath/> - Creates a Path ID that you can use in Java compiles.
  The jars are not downloaded into your workspace.

* <ivy:cachedileset/> - Creates a fileset that can be used in your project.
  The jars are not downloaded into your workspace.

* <ivy:retrieve/> - Retrieves the dependencies and put them into your
  workspace as you've specified. This is good for downloading the
  required jars for wars and ears.

* <ivy:cleancache/> - Add to your clean target. This will clean out the
  ivy cache.

* <ivy:report/> - Will produce a set of reports on your Ivy
  dependencies. These dependencies can be placed in HTML format, graph
  ML format, and graphviz format.


OTHER TASKS
===========

These are new tasks created for the various build tools

* pmd
* findbugs
* cpd
* checkstyle
* jacoco.coverage (Used to add coverage to JUnit tests)
* jacoco.report (Used to create a coverage report)


STANDARD TARGETS
================

We will be standardizing on Maven Goal names as our targets. These
should be the standard ones used:

* all         Jenkins Target. Cleans everything and runs everything
* checkstyle  Check Source Code Styles
* clean       Cleans all built artifacts and the Ivy Cache
* generate-sources Generate any required Java sources
* compile     Compiles the Java source code
* cpd         Detects copy and pasting in code
* findbugs    Run the findbugs report for Jenkins
* ivy.report  Produced the Ivy Dependency Report
* javadoc     Produce the Javadocs
* package     Packaging of the artifact
* pmd         Runs PMD Bug Detector Against Source
* site        Produces a whole slew of reports
* test-compile Compile the JUnit tests
* test        Run the JUnit tests

Default target: package

STANDARD DIRECTORY STRUCTURE
============================

There are several Macros that expect the standard Maven directory
structure. The exception will be a directory called ${basedir}/archive.
Any files in this directory will be automatically archived by the
Jenkins CI system.

* src/main/java - All current java source files

* src/main/resources - All files here are placed in the root of jarfiles
  and in the WEB-INF directory of warfiles.

* src/main/webapps - All files here are placed in the root of the
  warfiles.

* src/main/resource/WEB-INF/application.xml - application.xml file

* src/test/java - Source code for all Java tests

* src/test/resources - Non-Java files needed for tests. Will be copied
  into the root of the test classes directory.

* target - Scratch directory for all files not in the repository

* target/archive - All files here will be archived into the build

* target/maven - This is a rather interesting directory. When you
  create a jar, you should place it in a subdirectory in this directory.
  When that jar is created, two pom files: pom.xml and pom-snaphost.xml
  are also created in that directory. These poms are identical except
  that the revision of the pom-snapshot.xml will have "-SNAPSHOT" appended
  to it.

  The purpose of this is to allow you to use mvn deploy:deploy-file to
  deploy the jar to either your snapshot repository or to your release
  repository. All you have to do is specify the correct parameters and
  choose the correct pom.

  HOW I USE IT: I have Jenkins do a build on each change, and after each
  build I use "mvn deploy:deploy-file" to deploy the jar into our snapshot
  directory. When approved, I will also deploy that jar into our release
  repository. This means that Build #20 is our release, but the last build
  might be Build #25 which contains features you need. In that case, you can
  specify the SNAPSHOT version which would be Build #25.


JENKINS EXPECTED OUTPUT
=======================

* target/findbugs.xml     - Findbugs Output
* target/apidocs/         - Location of all Javadocs
* target/surfire-tests/   - All tests in XML format
* target/pmd.xml          - PMD output
* target/cpd.xml          - CPD output
* target/checkstyles.xml  - Checkstyles output
* target/archive/         - File to save for this build


BUILD WORKINGS
==============
All builds should work via a basic checkout. The only tools required
should be:

* Ant version 1.8.x
* JDK version 1.6 or better
* Subversion client version 1.5.x or better

A user should be able to do a checkout and build. The default target
should be "package" which will do the basic build developers want.

The build.xml should not depend upon a build.properties file, and should
work with no changes. The location of tools such as the Java compiler
should not have to be specified.

Developers, however, can use a build.properties file to change the
default behavior of properties for debugging purposes. For example, if a
developer sets "javac.listfiles" to "true", the <javac> task will list
files it is compiling.

By default, <javac> should not be forking except in rare circumstances
where a build won't work otherwise.

By default, JARS, EARS, and WARS should contain the Jenkins build
information and the build date in the MANIFEST.INF file. The
build-sample.xml file will show you how to do this.

There is a sample build.xml file at build-sample.xml that will help you
layout the new format.

PROPERTY NAMES
==============

Property names will be all lower case and each part will be separated by
periods. For example, main.test.dir and not mainTestDir. If a property
is involved with a particular task, the first part of the name should be
the name of the task, and the part after the dot is the parameter for
that task. For example, ivy.log sets the value of the "log" field for
the various Ivy tasks. ivy.cleancache will be used to see whether or not
to run the <ivy:cleancache> task. jar.destfile will be the name of the
jar's destination file. javac.verbose will be the  setting for the
verbose parameter for the <javac> task.

The exception will be for main and test properties. For example, we
should call the destdir for the <javac> task javac.destdir. However,
that will cause problems since the test classes and main classes should
be in two separate directories. Therefore, we will use "main" or "test"
as the first part of the property  name instead of "javac".

DON'T USE	MAIN		TEST
==============  ==============  ================
javac.srcdir	main.srcdir	test.srcdir
javac.destdir	main.destdir	test.destdir
javac.classpath main.classpath  test.classpath

FILES
=====
* ivy.tasks.xml - Basic Ivy task integration (plus other tools)
* build.xml - Sample build.xml file
* build.template.properties - A template file for build.properties
* ivy.template.xml - A template file for creating your ivy.xml file

USING STANDARD MACROS

JAVAC.MACRO:

This will automatically set -XLint and Xmaxwarns for pickup by
Jenkins CI. You can turn Xlint off by setting the "lint" parameter
to "false":

    <javac.macro destdir="${main.destdir}">
	<src path="${main.srcdir}"/>
    </javac>

    <javac.macro destdir="${main.destdir}"
	lint="false">>
	<src path="${main.srcdir}"/>
    </javac>
	
JAR.MACRO:

This will automatically create a pom.xml file in
${basedir}/target/archive, and also put that pom.xml
into your META-INF/maven directory. It will also put
Jenkins build information and Maven information into
your MANIFEST.MF file:

    <jar.macro destfile="${target.dir}/{$jar.name}.jar">
	<fileset dir="${main.destdir}"/>
	<fileset dir="${main.resouces}"/>
    </jar.macro>

Other macros <war.macro> and <ear.macro> are similar. The
<zip.macro> creates a version.txt file that contains the
Jenkins build information and places it in the root of the zip.

USING THE OTHER TASKS
=====================

FINDBUGS:

    <findbugs.macro>
    	<auxClasspath refid="main.classpath"/>
	<class location="${main.destdir}"/>
	<sourcePath="${main.srcdir}"/>
    </findbugs.macro>
or
    <findbugs.macro output="xml"
	outputFile="${targetdir}/findbugs.xml>
    	<auxClasspath refid="main.classpath"/>
	<class location="${main.destdir}"/>
	<sourcePath="${main.srcdir}"/>
    </findbugs.macro>

PMD:

    <pmd.macro>
    	<fileset dir="${main.srcdir}"/>
    </pmd.macro>
or
    <pmd.macro file.type="xml"
    	file.name="${target.dir}/pmd.xml">
    	<fileset dir="${main.srcdir}"/>
    </pmd.macro>

CHECKSTYLE:

    <checkstyle.macro>
    	<fileset dir="${main.srcdir}"/>
    </checkstyle.macro>
or
    <checktyle.macro formatter.type="xml"
	formatter.toFile="${target.dir}/checkstyle.xml">
    	<fileset dir="${main.srcdir}"/>
    </checkstyle.macro>

CPD:

    <cpd.macro>
	<fileset dir="${main.srcdir}"/>
    </cpd.macro>
or
    <cpd.macro format="xml"
	outputfile="${target.dir}/cpd.xml"
	minimumtokencount="100"
	language="java"
	ignoreLiterals="true"
	ignoreIdentifiers="true">
	<fileset dir="${main.srcdir}"/>
    </cpd.macro>

JACOCO:

Coverage:

    <jacoco.coverage>
    	[Your JUnit task definition]
    </jacoco.coverage>
or

    <jacoco.coverage
	destfile="${target.dir}/jacoco/jacoco.exec"
	append="false">
    	[Your JUnit task definition]
    </jacoco.coverage>

Reporting:

    <jacoco.report>
    	<classfiles>
	    <fileset dir="${main.destdir}"/>
	</classfiles>
	<sourcefiles>
	    <fileset dir="${main.srcdir}"/>
	</sourcefiles>
    </jacoco.report>
or
    <jacoco.report
	jacoco.report.name="JUnit Coverage Report"
	jacoco.executiondate.file="${target.dir}/jacoco/jacoco.exec"
	jacoco.html.destdir="${target.dir}/jacoco/html"
	jacoco.xml.destfile="${target.dir}/jacoco/jacoco.xml">
    	<classfiles>
	    <fileset dir="${main.destdir}"/>
	</classfiles>
	<sourcefiles>
	    <fileset dir="${main.srcdir}"/>
	</sourcefiles>
    </jacoco.report>
