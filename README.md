# JNC

Java NETCONF Client.

[![Build Status](https://travis-ci.org/btisystems/JNC.svg?branch=master)](https://travis-ci.org/btisystems/JNC)


## Overivew

JNC (Java NETCONF Client) is the name of both a Java library for NETCONF client
code and a Java output format plugin for pyang, an extensible YANG validator
and converter in python. You need an installation of pyang to use the JNC
plugin. Get it here: http://code.google.com/p/pyang/

The JNC plugin can be used to generate Java class hierarchies from YANG data
models. Together with the JNC library, these generated Java classes may be used
as the foundation for a NETCONF client (AKA manager) written in Java. The JNC
library is distributed along with the JNC plugin script.

JNC uses Ganymed SSH-2 to communicate with NETCONF servers/agents, you may get
it from here: http://www.cleondris.ch/opensource/ssh2/ or by using aptitude
(http://manpages.ubuntu.com/manpages/lucid/man8/aptitude.8.html):
sudo aptitude install libganymed-ssh2-java

This readme proposes two different ways of getting started. If you're running
Linux and don't want to use eclipse, feel free to skip the following section.


## Get started using eclipse

Make sure you have pyang installed, and that jnc.py is in the plugins directory
of your installation. For instructions on how to use pyang, please see the
pyang man page.

[pyang](http://www.yang-central.org/twiki/pub/Main/YangTools/pyang.1.html)

Create an eclipse project for JNC: open the "New Java Project" dialog, uncheck
the "Use default location" checkbox and choose the jnc folder as the location
instead, then Finish.

Now you need to add ganymed ssh2 to the project. Right click on the project
folder in the Package Explorer, choose Build Path->Add External Archives and
select your ganymed ssh2 Jar file.

If you don't want to run the tests, just delete the test source folder.
Otherwise, you need to add JUnit 4 to the build path. This may be done by
opening one of the test files, placing the marker on one of the lines which
have errors, pressing Ctrl+1, choosing "Fix project setup..." and then OK when
eclipse proposes to add the library. You should now be able to run the tests
by right clicking on the test source folder and selecting Run As -> JUnit test.

The next step is to generate some Java classes using the JNC pyang plugin and
write a client that uses them together with the JNC library. Please see the
user manual for instructions on how to do this.


## Get started in Linux (without eclipse):

Add the JNC plugin to your existing pyang installation. This may be done in one
of the following three ways:

1. Add jnc.py to pyang/plugins in your pyang installation,
2. Add the location of jnc.py to the $PYANG_PLUGINPATH environment variable, or
3. Use the --plugindir option of pyang each time you want to use JNC

If more than one of these approaches is used, you will end up with optparse
conflicts so please choose one and stick with it. From here on, we will assume
that you went for (1), but using (2) or (3) should be anologous.

JNC can be used to generate Java classes from a YANG file of your choice.
There are a collection of yang files in the 'examples/yang' directory. To
generate classes for a YANG file, open a terminal, change directory to where
you want the classes to be generated, launch pyang with the jnc format,
specifying the output folder and the yang model file.

For example, to generate the classes for simple.yang to the 'examples'
directory with base package gen.simple, type:

     $ pyang -f jnc --jnc-output src/gen/simple yang/simple.yang

There should now be a newly generated 'src' folder in the current directory,
containing a directory structure with the generated classes. Note that 'src' is
special treated so that it does not become part of the package names of the
generated classes.

To get more detailed information on how the generation proceeds the --jnc-debug
or --jnc-verbose options can be used. Rerunning JNC silently overwrites any old
classes in the output directory.

To actually use the generated classes, you need to compile Java client code
using the JNC library. It might be convenient to make a Jar file with the JNC
library for this purpose. There are several ways to do this:

1. Run the **ant** script located in the jnc directory: `ant clean jar`
2. Build the jar using **maven** from the jnc directory: `mvn clean package`
3. Manually: Open a terminal and enter the following:

          cd jnc
          mkdir -p lib bin
          cd lib
          wget http://repo1.maven.org/maven2/ch/ethz/ganymed/ganymed-ssh2/262/ganymed-ssh2-262.jar
          cd ..
          javac -cp lib/ganymed-ssh2-262.jar:. -d bin -sourcepath src src/main/java/com/tailf/jnc/*.java
          cd bin
          jar cvf ../lib/JNC.jar *
          
4. Set up a project in eclipse and export the Jar (see previous section).

If the ant script fails to run, the most probable reason is that ganymed ssh2
is not installed in the expected way. Check that the ganymed.dir variable
points to the directory containing your ganymed ssh2 Jar file, and that the
classpath in the compile target corresponds to the correct build.

To run the JNC tests, see the previous section or run them with JUnit 4 from
commandline if you so prefer. The tests resides in the jnc/test folder.

Please see the user manual for tutorials and further information on how to
write a client and how to run it against a server using JNC.


## High level description

The different types of generated files are:

Root class  -- This class has the name of the prefix of the YANG module, and
               contains fields with the prefix and namespace as well as methods
               that enables the JNC library to use the other generated classes
               when interacting with a NETCONF server.

YangElement -- Each YangElement corresponds to a container or a list in the
               YANG model. They represent tree nodes of a configuration and
               provides methods to modify the configuration in accordance with
               the YANG model that they were generated from.
               The top-level containers or lists in the YANG model will have
               their corresponding YangElement classes generated in the output
               directory together with the root class. Their respective
               subcontainers and sublists are generated in subpackages with
               names corresponding to the name of the parent container or list.

YangTypes   -- For each derived type in the YANG model, a class is generated to
               the root of the output directory. The derived type may either
               extend another derived type class, or the JNC type class
               corresponding to a built-in YANG type.

Packageinfo -- For each package in the generated Java class hierarchy, a
               package-info.java file is generated, which can be useful when
               generating javadoc for the hierarchy.

Schema file -- If enabled, an XML file containing structured information about
               the generated Java classes is generated. It contains tagpaths,
               namespace, primitive-type and other useful meta-information.

The typical use case for these classes is as part of a JAVA network management
system (EMS), to enable retrieval and/or storing of configurations on NETCONF
agents/servers with specific capabilities.


## Release History
2012-06-04

Emil starts working at Tail-f and reads up on the NETCONF RFC 6241 and YANG RFC
6020. You should too if you intend to contribute to this project!

2012-06-12

JPyang is born as a few lines of python code that integrates with pyang are
written.

2012-06-20

Empty initial commit of the repository. It will contain the source code for
JPyang once it has been decided that it will be open source rather than
proprietary to tail-f. The plugin itself is just the single jpyang.py script.

2012-07-06

New tests for the INM and ConfM library classes are written.

2012-07-13

Work on a new structure for the JPyang code begins, using classes to represent
methods and organize functionality.

2012-07-16

Unit tests for JPyang functions and class methods are added to pyang.

1012-07-27

The ConfM.xs classes are replaced by new internal representations of the YANG
built in classes.

2012-08-03

The INM and ConfM libraries are merged into the new JNC library, with better
support for YANG.

2012-08-16

JPyang is now fully object oriented, with method generator classes for all
relevant statements. This means that the generated classes should import all
JNC, java.util, java.math and generated classes that they use, and no other.

2012-08-20

The JPyang project repository is cleaned up, removing all non-comprehensible
files and adding a new README and ant build files.

2012-08-23

Meeting at Tail-f about the future of the project. The marketing and
product management VP seems inclined to release as open source once the most
basic functionality has been tested and the documentation is complete.

2012-08-28

JPyang is renamed to JNC. So now there is both a JNC library and a
JNC pyang plugin.

2012-08-29

The GitHub repository is renamed to JNC and its readme is updated.

2012-09-20

Support for notifications added, and there is only one thing
remaining until it is possible to generate fully compliant junos.yang classes!

2012-10-17

JNC is finally made open source! There are some changes remaining
before this will be publicly announced however.

## License
Copyright 2012 Tail-f Systems AB

See [License File](LICENSE).
