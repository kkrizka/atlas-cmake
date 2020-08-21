---
title: "Your Workspace"
teaching: 20
exercises: 20
questions:
- "How should I organize my code?"
objectives:
- "Learn about the standard structure for an ATLAS workspace."
keypoints:
- "Use three directories: `source/` for code, `build/` for binaries and `run/` for output."
---

# The Structure

When working on your analysis, you will most likely be editing, compiling and running code from several packages. There is a recommended way to organize your code. The structure is as follows:

- `source/`: Place all your packages here, one per directory.
- `source/CMakeLists.txt`: The top-level `CMakeLists.txt` file loading all of the ATLAS infrastructure and finding all of your packages.
- `build/`: Compiled binaries.
- `run/`: Place to run your analysis from (ie: where the results will be placed).

This is only a recommendation. You can take other approaches, if you like. For example, I like to append the release version to the `build` directory name. This allows me to easily revert back to an older releases, in case I want to understand a change.

> ## Multiple Build Directories
>
> What feature of CMake allows you to have multiple build directories?
>
> > ## Solution
> >
> > This is an advantage of the `out-of-source` builds used in CMake. You can separate your source code from the binaries, allowing you to have several variations on the compiler in parallel.
> {: .solution}
{: .challenge}

# Transitioning Your Current Project
At the end of the last day, you should be left with the following structure.

~~~shell
ls
~~~
{: .language-bash}

~~~
AnalysisPayload.cxx  CMakeLists.txt  JetSelectionHelper/  README.md		     
~~~
{: .output}

Our goal is to convert it into the structure similar to the one outline above. We will do this using git command, meaning that your whole change history will remain intact.

Start by creating the necessary directories.

~~~shell
mkdir source/
mkdir build/
mkdir run/
~~~
{: .language-bash}

Then you should move your `JetSelectionHelper` package into the `source/` directory. Unfortunately, the version of git inside the docker image is quite old (a common theme with ATLAS software, stability is key for our experiment!) and does not yet include the ability to move git submodules. There is a manual way to do this by editing several files inside `.git/`, but that is outside the scope of this tutorial. Instead we will remove the old `JetSelectionHelper` submodule and re-add it as `source/JetSelectionHelper`. Since the `JetSelectionHelper` lives inside its own repository, you will not lose any of the change history by doing so.

To remove the existing submodule, you need manually edit the `.gitmodule` file and delete the following section.
~~~
[submodule "JetSelectionHelper"]
        path = JetSelectionHelper
        url = https://gitlab.cern.ch/usatlas-computing-bootcamp/JetSelectionHelper.git
~~~
{: .source}

From `.git/config`, remove the following section.

~~~
[submodule "JetSelectionHelper"]
        url = https://gitlab.cern.ch/usatlas-computing-bootcamp/JetSelectionHelper.git
~~~
{: .source}

Finally remove the directory itself from git's index.

By using the `--cached` option, you only remove the directory from the index while maintaining the copy on the file system. Just in case you forgot to commit some changes.

~~~shell
git rm --cached JetSelectionHelper
~~~
{: .language-bash}

After these three steps, your git repository will no longer know about the `JetSelectionHelper` submodule.

> ## Deleting submodules in git 1.8 and beyond
>
> If you are using a newer version of git, you can delete a submodule in a single step.
> ~~~shell
> git submodule deinit JetSelectionHelper
> ~~~
> {: .language-bash}
{: .callout}


Next make a fork of the [JetSelectionHelper repository](https://gitlab.cern.ch/usatlas-computing-bootcamp/JetSelectionHelper). You will be making a few modifications. Add it as a submodule under the `source/` directory.
~~~shell
git submodule add ssh://git@gitlab.cern.ch:7999/kkrizka/JetSelectionHelper.git source/JetSelectionHelper
~~~
{: .language-bash}


The last step is to move the `AnalysisPayload` codebase into a new package with the same name. This can be done simply by using the `git mv` command that you might have learned about yesterday. The following block of commands will move everything into the structure described earlier.

~~~shell
mkdir source/AnalysisPayload
mkdir source/AnalysisPayload/util
git mv AnalysisPayload.cxx source/AnalysisPayload/util/
git mv CMakeLists.txt source/
~~~
{: .language-bash}

After this, everything should be set to go! The list of changes to your repository should look like the following. Make sure to commit everything before moving to the next step!

```bash
[bash][atlas]:v4-gitmodule-submodule-jetselector-simplecmake > git status .
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   .gitmodules
#       deleted:    JetSelectionHelper
#       renamed:    CMakeLists.txt -> source/CMakeLists.txt
#       renamed:    AnalysisPayload.cxx -> source/AnalysisPayload/util/AnalysisPayload.cxx
#       new file:   source/JetSelectionHelper
#
```

> ## Tracking the build/ Directory
>
> Notice that we did not at any point add the `build/` directory to the repository index. You should never version the contents this directory. Why?
>
> > ## Solution
> >
> > There are two important reasons:
> > - Git reduces the history size by storying only the differences between file versions. For binary files, which is what `build/` mostly contains, finding the differences is more complicated. Thus by default, will store the entire binary file anytime you make a change. This will blow up the size of your repository by a lot.
> > - Some of the files contain information custom to the user's environment (ie: compiler, location of libraries). By versioning these files, you will clutter the history by tracking changes that are not relevent to the execution of the code.
> >
> > If you ever commit the `build/` directory, bad things will happen to you!
> {: .solution}
{: .challenge}




{% include links.md %}

