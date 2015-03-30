Compiling Naga from sources.

# Introduction #

How to compile naga from the sources.


# Details #

First, check out naga trunk from the subversion repository.  This can either be done using a tool or on the commandline.

If done from the command line, use: `svn checkout http://naga.googlecode.com/svn/trunk/ <target directory>`.
This will download a read-only copy of the naga repository.

### Source ###

The naga source code itself is available under src/main, and compiles with Java 1.5 without any dependencies whatsoever.

### Tests ###

The naga test suite is located in src/test.

It has dependecies on easymock, so you need to download easymock with classextensions from easymock.org (follow the instructions on http://www.easymock.org/ and remember that easymockclassextension.jar has a dependency on cglib).

### Build Scripts ###

Naga provides an ant build script that will build both relase and debug jars as well as generating javadoc. The resulting jars will be written to the `_DIST` directory, and the javadoc is available in `_TEMP/docs`.
