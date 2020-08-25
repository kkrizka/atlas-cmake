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

Start by removing the work tree of your submodule from your local git repository.
~~~shell
git submodule deinit JetSelectionHelper
~~~
{: .language-bash}

To remove the reference to the submodule, you need manually edit the `.gitmodule` file and delete the following section.
~~~
[submodule "JetSelectionHelper"]
        path = JetSelectionHelper
        url = https://gitlab.cern.ch/usatlas-computing-bootcamp/JetSelectionHelper.git
~~~
{: .source}

Finally remove the directory itself from git's index.

~~~shell
git rm JetSelectionHelper
~~~
{: .language-bash}

After these three steps, your git repository will no longer know about the `JetSelectionHelper` submodule.

Next make a fork of the [JetSelectionHelper repository](https://gitlab.cern.ch/usatlas-computing-bootcamp-2020/JetSelectionHelper). You will be making a few modifications. Add it as a submodule under the `source/` directory. Don't forget to replace `${USER}` with your GitLab username!
~~~shell
git submodule add ssh://git@gitlab.cern.ch:7999/${USER}/JetSelectionHelper.git source/JetSelectionHelper
~~~
{: .language-bash}


The last step is to move the `AnalysisPayload` codebase into a new package with the same name. This can be done simply by using the `git mv` command that you might have learned about yesterday. The following block of commands will move everything into the structure described earlier.

~~~shell
mkdir source/AnalysisPayload
mkdir source/AnalysisPayload/util
git mv AnalysisPayload.cxx source/AnalysisPayload/util/
git rm CMakeLists.txt
rm source/JetSelectionHelper/CMakeLists.txt
~~~
{: .language-bash}

The list of changes to your repository should look like the following. Make sure to commit everything before moving to the next step! Don't worry about the modified content (missing `CMakeLists.txt`) inside the `JetSelectionHelper` for now.

```bash
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   .gitmodules
#       deleted:    CMakeLists.txt
#       renamed:    AnalysisPayload.cxx -> source/AnalysisPayload/util/AnalysisPayload.cxx
#       renamed:    JetSelectionHelper -> source/JetSelectionHelper
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#   (commit or discard the untracked or modified content in submodules)
#
#       modified:   source/JetSelectionHelper (modified content)
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

