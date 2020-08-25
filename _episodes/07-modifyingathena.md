---
title: "Modyfing an Athena Package"
teaching: 20
exercises: 20
questions:
- "How do I modify a package inside the athena repository?"
objectives:
- "Learn about sparse git checkouts."
- "Learn about `package_filters.txt`."
keypoints:
- "Don't compile the entire athena repository. Use either sparse checkouts or `package_filters.txt` to specify what packages you want to compile."
---

In this section, we will learn how to modify a package in the athena repository. Part of making changes is compiling them for testing.

Start by pointing your web browser to the [official; athena repository](https://gitlab.cern.ch/atlas/athena) and making a fork. As we learned yesterday, you should always make a fork if you want to modify other people's code. One you made your fork, made a clone of it. The last line tells git to checkout the `21.2` branch. This is the base for the release that we are working with.

~~~bash
# Run from source/ directory
git clone ssh://git@gitlab.cern.ch:7999/${USER}/athena.git
cd athena
git checkout 21.2
~~~
{: .source}

Do not try to compile your project now. The athena repository contains many packages. Compiling it will take forever. Instead we will look at two ways to copmile only specific packages:
- `package_filters.txt`, which is an ATLAS CMake addition to take a list of packages you want to compile,
- sparce checkouts, which is a git feature to only checkout specific files.

# The `package_filters.txt` Method



The `ATLAS_PROJECT` command, that finds all of your packages, can be tune by the use of a `package_filters.txt` file. This file can contain specific instructions for what packages to ignore and which ones to compile. Create a `sources/package_filters.txt` file with the following contents.

~~~
+ athena/Reconstruction/Jet/JetCalibTools
- athena/.*
+ .*
~~~
{: .source}

Then clone your fork into the `source/` directory. Make sure to change `kkrizka` to your lxplus username.

The first column is whether to compile a package (`+`) or to ignore it (`-`). You can use regular expressions to select multiple packages. In this example, the first line says we want to compile `JetCalibTools` package, the second line says ignore all athena packages not already listed and the third line says compile all other packages.

Now you need to re-run CMake, telling it explicitely where your package filter file is located.

~~~
# Run from build/ directory
cmake -DATLAS_PACKAGE_FILTER_FILE=../source/package_filters.txt ../source
~~~
{: .language-bash}

Now run `make`. It should finish quickly!


# The Sparse Checkout Method
The sparse checkout is a git feature that allows you to only "checkout" files that you want to the file system. In our case, we will only checkout the `JetCalibTools` package. As you will see, it is a more hidden feature of git and requires some manual work.

The first step is to enable sparse checkout functionality inside your athena clone.

~~~
# Run from source/athena directory
git config core.sparsecheckout true
~~~
{: .language-bash}

Then define the list of directories or files that you want to be explicitely checked out. This list is stored inside the `.git/info/sparse-checkout` file. The `.git/` folder is inside the root of your git repository and contains all of your settings and repository history.

~~~
# Run from source/athena directory
echo Reconstruction/Jet/JetCalibTools/ > .git/info/sparse-checkout
~~~
{: .language-bash}

Finally tell git to redo the checkout of your branch. You should now only have the `Reconstruction/Jet/JetCalibTools/` package leftover.

~~~
# Run from source/athena directory
git checkout 21.2
ls
~~~
{: .language-bash}

~~~
Reconstruction
~~~
{: .output}


Finally you need to re-run CMake to register the new package and compile using make.

~~~
# Run from build/ directory
cmake ../source
make
~~~
{: .language-bash}




{% include links.md %}
