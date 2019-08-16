---
title: "Introduction to ATLAS Offline Software"
teaching: 10
exercises: 10
questions:
- "What is atlas/athena?"
objectives:
- "Introduce the athena repository."
- "Define the concept of a *package*."
- "Define the contect of a *project*."
- "Describe what a release means."
keypoints:
- "Athena is a collection of common packages that are compiled and released regulariy."
- "Your analysis project is just an external package."
---

In this episodes, we hope to introduce the organization of the ATLAS software (athena).

# What is Athena?
All of the central ATLAS software is stored in a public repository called [Athena](https://gitlab.cern.ch/atlas/athena). It contains all of the code used for simulation, data reconstruction, triggering, calibration and object identification.

This lesson will not touch any of this source code in detail. How to develop new Athena software is covered in the [official ATLAS documentation](https://atlassoftwaredocs.web.cern.ch/athena/). Instead we will focus on *using* the pre-compiled versions of a few athena packages.

# Athena Packages
The Athena repository contains several packages. A single package is a collection of related libraries, programs and other supporting files that are maintained by a dedicated group of developers. For example, the `Reconstruction/Jet/JetCalibTools/JetCalibTools` package contains all of the code necessary to calibrate jets.

Each package, for example `Package`, is a directory with the following structure:
- `Package/`: All of the header files, where `Package` is replaced with the package's name
- `Root/`: All of the source code of the library
- `util/`: Any executables that are part of the package
- `share/`: Supporting files (ie: example settings, small calibration files)
- `CMakeLists.txt`: The file that tells cmake how this package is organized.

The structure itself is very flexible. Technically, only the `CMakeLists.txt` file is required to be there. However maintaining consistency across the experiment is very important for code readability.

You can also have packages outside of the athena repository. For example, you can think of your analysis code as an external package.

# Projects and Releases
The entire athena repository is huge and has many dependencies between its packages. New code is being added all the time. To keep everything stable, there is a complex release procedure for "public" code.

First of all, there is the concept of an athena project. An athena project is a collection of packages required to accomplish a certain task. For example, if you only care to analyze events, then you don't need all of the infrastrucure used during detector operation. Reducing the package list to only what is required has several advantages:
* Reduce the size of the compiled release. This is useful if you want to create a container that can run on your laptop.
* Reduce the amount of validation required. Every release has to be validated to ensure that no bugs have been introduced. Certain projects are released less often, giving 

Purpose
Athena	Functional for event generation, simulation, digitisation reconstruction and DAOD derivations (but rarely validated for anything except digi+reco)
AthenaP1	Used to run the High Level Trigger and online monitoring at P1
AthGeneration	For event generation
AthSimulation	For full Geant4 simulation
AthDerivation	For generating derived AODs from AOD and Athena based analysis
AthAnalysisBase	Athena based analysis
AnalysisBase	Non-athena ROOT based analysis


The entire athena repository is compiled every night and regular releases are copied to CVMFS for user's convenience. 

Compiling the entire athena repository takes a very long time. Every user should not do it them



{% include links.md %}

