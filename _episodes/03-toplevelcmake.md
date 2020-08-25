---
title: "The Top-Level CMakeLists.txt"
teaching: 20
exercises: 10
questions:
- "How do I tell CMake about all my packages?"
objectives:
- "Setup an ATLAS release"
- "Build a top-level workspace that identifies your packages"
keypoints:
- "Copy-paste the generic `CMakeLists.txt` from the ATLAS Software Documentation and place it inside the `source/` directory."
---

Now that we have reorganized our code, it is time to try to write the `CMakeLists.txt` that use some useful ATLAS macros. By using the ATLAS macros, you simplify your life by automating certain tasks like dependency look-up, placing all your binaries in convenient places and preparing an archive that can be uploaded to the grid.

In this section, we will write the top-level `CMakeLists.txt` that will setup your environment and include all of your custom packages. In practice, this is a generic file that can be copied from the ATLAS Software Documentation. However today we will write it ourselves, to understand what it is doing.

# Writing the CMakeLists.txt

Start by opening the `source/CMakeLists.txt` file that contains our current CMake build instructions and removing all of the contents. Then start by telling CMake that you will be using command available only in version 3.14 and beyond.

~~~cmake
# Set the minimum required CMake version:
cmake_minimum_required( VERSION 3.14 FATAL_ERROR )
~~~
{: .source}

The next step tries to automatically determine what ATLAS project you are using. By automating this task, you can easily transition between different projects to test your code in different environments. Remember, out-of-source builds are awesome for this!

The way this block works is by looping over all known projects (hardcoded as the `_parentProjectNames` list) and seeing if the `ProjectName_DIR` environmental variables has been set. After you run `source release_setup.sh` in your Docker image or `asetup AnalysisBase,21.2.125` when using CVMFS, this variable will contain the installation path of the project.

~~~cmake
# Try to figure out what project is our parent. Just using a hard-coded list
# of possible project names. Basically the names of all the other
# sub-directories inside the Projects/ directory in the repository.
set( _parentProjectNames Athena AthenaP1 AnalysisBase AthAnalysis
   AthSimulation AthDerivation AnalysisTop )
set( _defaultParentProject AnalysisBase )
foreach( _pp ${_parentProjectNames} )
   if( NOT "$ENV{${_pp}_DIR}" STREQUAL "" )
      set( _defaultParentProject ${_pp} )
      break()
   endif()
endforeach()

# Set the parent project name based on the previous findings:
set( ATLAS_PROJECT ${_defaultParentProject}
   CACHE STRING "The name of the parent project to build against" )

# Clean up:
unset( _parentProjectNames )
unset( _defaultParentProject )
~~~
{: .source}

Once we know which ATLAS project is being used, we tell CMake about it using the standard `FIND_PACKAGE` command. This configures all of the paths to the central packages and loads the CMake macros that we will use in the next section.

~~~cmake
# Find the AnalysisBase project. This is what, amongst other things, pulls
# in the definition of all of the "atlas_" prefixed functions/macros.
find_package( ${ATLAS_PROJECT} REQUIRED )
~~~
{: .source}

The next line uses a custom ATLAS macro to make writing unit tests easier. We will not dive into that here.

~~~cmake
# Set up CTest. This makes sure that per-package build log files can be
# created if the user so chooses.
atlas_ctest_setup()
~~~
{: .source}

The next step is how CMake learns what local packages you have in your repository. The `ATLAS_PROJECT` macro scans through all of the subdirectories, looking for ones containing a `CMakeLists.txt` file. All such directories are then defined as packages and included in the compilation.

The macro also provides a manual way to control packages that are compiled through a `package_filters.txt` file. We will not go into that here, but it is very useful when having a clone of the athena repository in your workspace.

~~~cmake
# With this CMake will look for "packages"
# in the current repository and all of its submodules, respecting the
# "package_filters.txt" file, and set up the build of those packages.
atlas_project( UserAnalysis 1.0.0
   USE ${ATLAS_PROJECT} ${${ATLAS_PROJECT}_VERSION} )
~~~
{: .source}

The ATLAS CMake framework creates a convenient setup script that puts all of your binaries into the relevant paths (ie: `PATH` and `LD_LIBRARY_PATH`). This way you can run the programs from any place you want, without caring where they are installed. The following set of lines copies the setup script into your build directory.

~~~cmake
# Set up the runtime environment setup script. This makes sure that the
# project's "setup.sh" script can set up a fully functional runtime environment,
# including all the externals that the project uses.
lcg_generate_env( SH_FILE ${CMAKE_BINARY_DIR}/${ATLAS_PLATFORM}/env_setup.sh )
install( FILES ${CMAKE_BINARY_DIR}/${ATLAS_PLATFORM}/env_setup.sh
   DESTINATION . )
~~~
{: .source}

The final step is to setup CPack. This is a CMake module that can generate distributable packages from your compiled binaries. This is used by Panda, the grid job management tool, to upload your code to the cloud when running over many big files.
~~~cmake
# Set up CPack. This call makes sure that an RPM or TGZ file can be created
# from the built project. Used by Panda to send the project to the grid worker
# nodes.
atlas_cpack_setup()
~~~
{: .source}

# Building your Workspace
Start by removing the `JetSelectionHelper/

Now that we have written the top-level `CMakeLists.txt` file, it is time to try to compile everything. Make sure that you have loaded the AnalysisBase release using the `release_setup.sh` script. Run the following commands inside the `build/` directory. The first command runs CMake to create a `Makefile` that can compile your entire workspace. The second command configures your environment with the necessary paths, as described earlier.

~~~shell
# Run from build/ directory
cmake ../source
source x86_64-centos7-gcc8-opt/setup.sh
~~~
{: .language-bash}

If you wrote the `CMakeLists.txt` file correctly, you should see a bunch of output saying random dependencies have been found and end with the following.

~~~
# Lots of boring text

-- Configuring done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/atlas/Bootcamp/v4-gitmodule-submodule-jetselector-simplecmake/build
~~~
{: .output}

Now let's try to compile our source code by running `make`. You should quickly see the following:
~~~bash
make
~~~
{: .language-bash}

~~~
Scanning dependencies of target atlas_tests
Built target atlas_tests
~~~
{: .output}

That was quick! We haven't written our package's `CMakeLists.txt` files, so it does not compile them. That's what we will do next.

> ## Restoring Your Environment
>
> When you come back to your workspace at a later date, what commands do you run to setup the environment?
>
> > ## Solution
> >
> > You need to run two commands. One to setup the ATLAS project and the second to configure the enviroment of your workspace. You do not have to rerun `cmake`.
> >
> > ~~~bash
> > # Run from build/ directory
> > source ~/release_setup.sh 
> > source x86_64-centos7-gcc8-opt/setup.sh 
> > ~~~
> > {: .language-bash}
> >
> > ~~~
> > Configured GCC from: /opt/lcg/gcc/8.3.0-cebb0/x86_64-centos7/bin/gcc
> > Configured AnalysisBase from: /usr/AnalysisBase/21.2.125/InstallArea/x86_64-centos7-gcc8-opt
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}


{% include links.md %}

