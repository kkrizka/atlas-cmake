---
title: "The Package CMakeLists.txt"
teaching: 20
exercises: 20
questions:
- "How do I compile my package?"
objectives:
- "Use ATLAS CMake macros to specify an executable and library targets."
- "Add ATLAS packages as dependencies."
keypoints:
- "Use ATLAS macros. They specify a structure to your `build/` directory that makes your life easy!"
---

In this section, we will write two `CMakeLists.txt` files; one for the `JetSelectionHelper` package and one for the `AnalysisPayload` package. The contents will tell CMake how to compile each package. The former will teach you how to define a library and the latter how to define an executable program.

The approach we will take is to slowly construct the `CMakeLists.txt` incrementaly adding blocks and running `make` each time. Quite often, this will result in an error. We will then analyze the error and implement the next block that fixes it.

# Compiling JetSelectionHelper

Start by creating a new file at `source/JetSelectionHelper/CMakeLists.txt` with the following contents. The `ATLAS_SUBDIR` line identifies your directory as an ATLAS package with the name `JetSelectionHelper`.

~~~cmake
# The name of the package
ATLAS_SUBDIR(JetSelectionHelper)
~~~
{: .source}

Let's see what this will do. At this point, you need to run `cmake ../source` again for CMake to learn about the new file. After this, you only need to run `make` to process any changes.

~~~
# Run from build/ directory
cmake ../source
make
~~~
{: .language-bash}

~~~
# Boring CMake output
Scanning dependencies of target Package_JetSelectionHelper_tests
[  0%] Built target Package_JetSelectionHelper_tests
Scanning dependencies of target atlas_tests
[  0%] Built target atlas_tests
Scanning dependencies of target Package_JetSelectionHelper
[100%] Built package JetSelectionHelper
JetSelectionHelper: Package build succeeded
[100%] Built target Package_JetSelectionHelper
~~~
{: .output}

You can see that now the package is being picked up by make. Great! However your binary is not being compiled. Not so great. This is because we have not defined what should be compile using what source code. This is done using the `ATLAS_ADD_LIBRARY` command. It is the equivalent of the standard CMake `ADD_LIBRARY` command, but with convenience added in for handling dependencies.

Add the following line to tell CMake that you want to compile a library called `JetSelectionHelperLib` using the source file `src/JetSelectionHelper.cxx`. It also copies your header files, that will be used by other packages depending on `JetSelectionHelperLib` (ie: `AnalysisPayload`) to a central place.

~~~cmake
# Add binary
ATLAS_ADD_LIBRARY ( JetSelectionHelperLib JetSelectionHelper/JetSelectionHelper.h src/JetSelectionHelper.cxx
		  PUBLIC_HEADERS JetSelectionHelper )
~~~
{: .source}

Let's see what this will do. Note that you only need to run `make` this time. CMake already knows about your `JetSelectionHelper/CMakeLists.txt` from the previous step, allowing `make` to automatically look for updates.

~~~bash
# Run from build/ directory
make
~~~
{: .language-bash}

~~~
Scanning dependencies of target JetSelectionHelperHeaderInstall
[ 25%] Generating ../x86_64-centos7-gcc8-opt/include/JetSelectionHelper
[ 25%] Built target JetSelectionHelperHeaderInstall
Scanning dependencies of target JetSelectionHelperLib
[ 50%] Building CXX object JetSelectionHelper/CMakeFiles/JetSelectionHelperLib.dir/src/JetSelectionHelper.cxx.o
In file included from /home/kkrizka/Bootcamp/v1-gitlab-finished-code/source/JetSelectionHelper/src/JetSelectionHelper.cxx:1:
/home/kkrizka/Bootcamp/v1-gitlab-finished-code/source/JetSelectionHelper/JetSelectionHelper/JetSelectionHelper.h:1:10: fatal error: xAODJet/Jet.h: No such file or directory
 #include "xAODJet/Jet.h"
          ^~~~~~~~~~~~~~~
compilation terminated.
make[2]: *** [JetSelectionHelper/CMakeFiles/JetSelectionHelperLib.dir/src/JetSelectionHelper.cxx.o] Error 1
make[1]: *** [JetSelectionHelper/CMakeFiles/JetSelectionHelperLib.dir/all] Error 2
make: *** [all] Error 2
~~~
{: .output}

Here we have our first error, `fatal error: xAODJet/Jet.h: No such file or directory` , telling us that we are missing something. Without any arguments, `ATLAS_ADD_LIBRARY` thinks that our library does not depend on any extra packages. But we are using quite a few central packages, like `xAODJet` for storing the jet representation. To find the name of the package, just look at the directory where the header file is stored.

The dependency can be added by using the `LINK_LIBRARIES` option with a list of packages. Update the call to `ATLAS_ADD_LIBRARY` to look like this:

~~~cmake
ATLAS_ADD_LIBRARY ( JetSelectionHelperLib JetSelectionHelper/JetSelectionHelper.h src/JetSelectionHelper.cxx
		  PUBLIC_HEADERS JetSelectionHelper
		  LINK_LIBRARIES xAODEventInfo
				 xAODRootAccess
				 xAODJet)
~~~
{: .source}

Now run `make` again and you should see it complete sucessfully.

~~~bash
# Run from build/ directory
make
~~~
{: .language-bash}

~~~
# Boring output from cmake detecting an updated and being automatically rerun
# Boring compilation output
Scanning dependencies of target JetSelectionHelperLib
[ 25%] Building CXX object JetSelectionHelper/CMakeFiles/JetSelectionHelperLib.dir/src/JetSelectionHelper.cxx.o
[ 50%] Linking CXX shared library ../x86_64-centos7-gcc8-opt/lib/libJetSelectionHelperLib.so
Detaching debug info of libJetSelectionHelperLib.so into libJetSelectionHelperLib.so.dbg
[ 50%] Built target JetSelectionHelperLib
Scanning dependencies of target JetSelectionHelperHeaderInstall
[ 75%] Generating ../x86_64-centos7-gcc8-opt/include/JetSelectionHelper
[ 75%] Built target JetSelectionHelperHeaderInstall
Scanning dependencies of target Package_JetSelectionHelper
[100%] Built package JetSelectionHelper
JetSelectionHelper: Package build succeeded
[100%] Built target Package_JetSelectionHelper
~~~
{: .output}

# Compiling AnalysisPayload

Now that our helper library is compiled, let's turn our attention to the executable that runs our analysis. Start by creating a `CMakaLists.txt` file. This time, we will use the `ATLAS_ADD_EXECUTABLE` command to declare an executable target. It has the same syntax as `ATLAS_ADD_LIBRARY`, but creates a binary instead of a library.

Try writing the `CMakeLists.txt` file on your own! Don't forget that our `AnalysisPayload` program depends on functions from the `JetSelectionHelper` package.

> ## Solution
>
> The contents of `../source/AnalysisPayload/CMakeLists.txt` should be the following:
>
> ~~~cmake
> # The name of the package
> ATLAS_SUBDIR(AnalysisPayload)
>
> # Add binary
> ATLAS_ADD_EXECUTABLE ( AnalysisPayload util/AnalysisPayload.cxx
>                      LINK_LIBRARIES  xAODEventInfo
>                                      xAODRootAccess
>                                      xAODJet
>                                      JetSelectionHelperLib)
> ~~~
> {: .source}
{: .solution}

On success, you should see the following when you try to build.

~~~bash
# Run from build/ directory
cmake ../source
make
~~~
{: .language-bash}

~~~
# Boring CMake output
# Boring output
Scanning dependencies of target AnalysisPayload
[ 42%] Building CXX object AnalysisPayload/CMakeFiles/AnalysisPayload.dir/util/AnalysisPayload.cxx.o
[ 57%] Linking CXX executable ../x86_64-centos7-gcc8-opt/bin/AnalysisPayload
# Boring Output
~~~
{: .output}

Now try running it. Note that the executable is in your `PATH`, so you can call it directly. Prefereably from within your `run/` directory.

~~~bash
# Run from run/ directory
AnalysisPayload
~~~
{: .language-bash}

We now have a working package! It is a good time to checkpoint and commit all of your changes.



{% include links.md %}

