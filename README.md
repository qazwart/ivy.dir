Ivy Integration Project
=======================

--

> ***Note***: This document is in [Markdown](http://daringfireball.net/projects/markdown/) format. Markdown is a *markup* language (like HTML), but is specifically written to be readable even if you you are looking at this file in plain text.

> Markdown has become a favorite markup language and is used in Wordpress sites, [Stackexchange] (http://stackexchange.com), and [GitHub] (http://github.com).

> However, this file will look best using a Markdown editor. On Windows, you can use [MarkdownPad for Windows] (http://markdownpad.com) or [MD Pad] (http://apps.codigobit.info/2012/06/markdown-pad-markdown-textfile-editor.html). 

> On the Mac, you can use [Markdown Pro] (http://www.markdownpro.com) (which is available from the App Store) or [marked] (http://markedapp.com). If you want a *free* Markdown editor, try [Mou] (http://mouapp.com).

> If you use Eclipse, you can use the [gfm_viewer] (https://github.com/satyagraha/gfm_viewer/blob/master/README.md) Markdown viewer for Eclipse.

--

PURPOSE
-------

This project automatically integrates Ivy into your project. It will
add into your project the correct Ivy settings and add in all the extra
tools that you need for doing your builds. Right now these include:

* **Checkstyle** - Verifies your code is formatted to the standard Java
  coding standards.
* **PMD** - Reads through your source code for possible programming errors
* **FindBugs** - Reads through your class files for possible programming
  errors.
* **CPD** - Copy Paste Detector - Checks code for large amounts of copy
  and pasted code.

It also includes the [Ant Contrib tasks]
(http://ant-contrib.sourceforge.net). These extensions help make it
easier to do such things are loop through a set of files and includes
`if/then/else` logic.


IMPLEMENTATION
--------------

#####Add in the following two lines at the beginning of your build.xml file:

    <property name="ivy.dir" value="${basedir}/ivy.dir"/>
    <import file="${basedir}"/>

#####Add in the Ivy XML Namespace by putting this in your project line:

    <project name="project_name" default="package" basedir="."
	xmlns:ivy="http://ant.apache.org/ivy"/>

Ant that's all there is to it. The import will import the Ivy settings
and the various build tools that we use. The xmlns line will add in a
separate namespace for the Ivy tasks.

IVY TASKS
---------

The following Ivy tasks have been imported into your build. There are
other Ivy tasks, but these are the basic ones.

Ivy tasks are documented at
http://ant.apache.org/ivy/history/latest-milestone/ant.html

* `<ivy:settings/>` - Setup Ivy. This is handled by the import.

* `<ivy:resolve/>` - Resolves the Ivy dependencies and will download the
  jars into your $HOME directory for use. This must be the first Ivy
  task you call.

* `<ivy:cachepath/>` - Creates a Path ID that you can use in Java compiles.
  The jars are not downloaded into your workspace.

* `<ivy:cachedileset/>` - Creates a fileset that can be used in your project.
  The jars are not downloaded into your workspace.

* `<ivy:retrieve/>` - Retrieves the dependencies and put them into your
  workspace as you've specified. This is good for downloading the
  required jars for wars and ears.

* `<ivy:cleancache/>` - Add to your clean target. This will clean out the
  ivy cache.

* `<ivy:report/>` - Will produce a set of reports on your Ivy
  dependencies. These dependencies can be placed in HTML format, graph
  ML format, and graphviz format.


OTHER TASKS
-----------

These are new tasks created for the various build tools

* `pmd`
* `findbugs`
* `cpd`
* `checkstyle`


STANDARD TARGETS
----------------

We will be standardizing on Maven Goal names as our targets. These
should be the standard ones used:

* `all`         Jenkins Target. Cleans everything and runs everything
* `checkstyle`  Check Source Code Styles
* `clean`       Cleans all built artifacts and the Ivy Cache
* `compile`     Compiles the Java source code
* `cpd`         Detects copy and pasting in code
* `findbugs`    Run the findbugs report for Jenkins
* `ivy.report`  Produced the Ivy Dependency Report
* `javadoc`     Produce the Javadocs
* `package`     Packaging of the artifact
* `pmd`         Runs PMD Bug Detector Against Source
* `site`        Produces a whole slew of reports
* `test`        Run the JUnit tests

Default target: package

STANDARD DIRECTORY STRUCTURE
----------------------------

We will standardize on the Maven directory structure. Except there will
be a directory called `${basedir}/archive`. Any files in this directory
will be automatically archived by the Jenkins CI system.


* `src/main/java`: All current java source files

* `src/main/resources`: All files here are placed in the root of jarfiles and in the WEB-INF directory of warfiles.

* `src/main/webapps`: All files here are placed in the root of the
  warfiles.

* `src/main/resource/WEB-INF/application.xml`: application.xml file

* `src/test/java`: Source code for all Java tests

* `src/test/resources`: Non-Java files needed for tests. Will be copied
  into the root of the test classes directory.

* `target`: Scratch directory for all files not in the repository

* `target/archive`: All files here will be archived into the build


JENKINS EXPECTED OUTPUT
-----------------------

* `target/findbugs.xml`     - Findbugs Output
* `target/javadocs/`        - Location of all Javadocs
* `target/surfire-tests/`   - All tests in XML format
* `target/pmd.xml`          - PMD output
* `target/cpd.xml`          - CPD output
* `target/checkstyles.xml`  - Checkstyles output
* `target/archive/`         - File to save for this build
* `target/maven/`		  - Where the Maven distributed jars are stored.
                            There should be a sub directory for each jar file that is being deployed to Maven


How the Build Should Work
-------------------------

All builds should work via a basic checkout. The only tools required should be:

* Ant version 1.8.x
* JDK version 1.6 or better
* Subversion client version 1.5.x or better

A user should be able to do a checkout and build. The default target should be "package" which will do the basic build developers want.

The `build.xml` should not depend upon a `build.properties` file, and should work with no changes. The location of tools such as the Java compiler should not have to be specified.

Developers, however, can use a `build.properties` file to change the default behavior of properties for debugging purposes. For example, if a
developer sets `javac.listfiles = true`, the `<javac>` task will list
files it is compiling.

By default, `<javac>` should not be forking except in rare circumstances where a build won't work otherwise.

By default, JARS, EARS, and WARS should contain the Jenkins build information and the build date in the MANIFEST.INF file. The `build.xml` file in the `ivy.dir` directory will show you how to do this.


FILES
-----
* `ivy.tasks.xml` - Basic Ivy task integration (plus other tools)
* `build.xml` - Sample build.xml file
* `build.template.properties` - A template file for build.properties
* `ivy.template.xml` - A template file for creating your ivy.xml file

USING THE OTHER TASKS
---------------------

####FINDBUGS:

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

####PMD:

    <pmd.macro>
    	<fileset dir="${main.srcdir}"/>
    </pmd.macro>

or

    <pmd.macro file.type="xml"
    	file.name="${target.dir}/pmd.xml">
    	<fileset dir="${main.srcdir}"/>
    </pmd.macro>

####CHECKSTYLE:

    <checkstyle.macro>
    	<fileset dir="${main.srcdir}"/>
    </checkstyle.macro>

or

    <checktyle.macro formatter.type="xml"
    	formatter.toFile="${target.dir}/checkstyle.xml">
    	<fileset dir="${main.srcdir}"/>
    </checkstyle.macro>

####CPD:

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

####JACOCO:

***Coverage***:

    <jacoco.coverage>
    	[Your JUnit task definition]
    </jacoco.coverage>

or

    <jacoco.coverage
	    destfile="${target.dir}/jacoco/jacoco.exec"
	    append="false">
    	    [Your JUnit task definition]
    </jacoco.coverage>

***Reporting***:

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
