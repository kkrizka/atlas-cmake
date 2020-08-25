---
title: "Calibrate Your Jets"
teaching: 20
exercises: 30
questions:
- "How do you use Athena tools?"
objectives:
- "Learn about Athena tools."
- "Use the JetCalibTool from the Package to calibrate your jets."
keypoints:
- "There are nice official tools to apply calibrations."
---

In this session, we'll learn how to use the `JetCalibrationTool` to calibrate our jets. This will showcase the usefulness of having central packages accomplishing some of the common tasks.

# Using The Calibration Tools
The first step is to implement the jet calibration tool inside our `AnalysisPayload` program. This is done using the `JetCalibTools` package, maintained by the JetEtMiss group. They are the ones responsible for deriving the calibrations and systematics, among many things. You can find more information about the tool on the [ApplyJetCalibrationR21](https://twiki.cern.ch/twiki/bin/view/AtlasProtected/ApplyJetCalibrationR21) twiki page. But don't take this as an example of a standard ATLAS documentation. The JetEtMiss group is exceptionally good at documenting their work!

The Athena framework provides a common interface for definining tools. They provide functionality for
- Garbage collection via smart pointers.
- Share instances between different parts of code via "public tools". This reduces memory usage, as each instance can load sizeable calibration tables!
- Abstract interface to allow for different tool implementation.
- Configurability via properties, accessible via C++ code or Python configuration files (callled job options).

## Initialize the Tool
Start by including the necessary header files to the beginning of your `AnalysisPayload.cxx`. We will go over what code you are including next.

~~~c++
// jet calibration
#include <AsgTools/AnaToolHandle.h>
#include <JetCalibTools/IJetCalibrationTool.h>
~~~
{: .source}

To use a tool, we wrap it inside the `asg::AnaToolHandle` class. This acts as a smart pointer, with ability to look for existing instances of the tool during initialization. We will refer to this object as a tool handle. This is a templated class, where the template is an interface class. The interface class determines what functions the tool should provide, without actually implementing them. This allows one to easily drop in alternate implementations. For the `JetCalibrationTool`, the interface class is `IJetCalibrationTool`. Add the following line near the beginning of your `main` function.

~~~c++
asg::AnaToolHandle<IJetCalibrationTool> JetCalibrationTool_handle;
~~~
{: .source}

So far, we have only declared the tool. We still need to initialize it and create an instance. As part of the initialization, you need to
- tell it what implemtation to use,
- configure any properties,
- instantiate the object.

Start by telling the tool handle that we want to use the `JetCalibrationTool` implementation with the name "MyCalibrationTool". The name is important for tool reuse. If another tool handle already exists with the same name, the instance of the tool is shared.
~~~c++
JetCalibrationTool_handle.setTypeAndName("JetCalibrationTool/MyCalibrationTool");
~~~
{: .source}

Next let's configure the tool itself using the latest and greatest available calibration. You can find what the fields mean in the [JetCalibrationTool](https://twiki.cern.ch/twiki/bin/view/AtlasProtected/ApplyJetCalibrationR21) documentation.
~~~c++
JetCalibrationTool_handle.setProperty("JetCollection","AntiKt4EMTopo"                                                  );
JetCalibrationTool_handle.setProperty("ConfigFile"   ,"JES_MC16Recommendation_Consolidated_EMTopo_Apr2019_Rel21.config");
JetCalibrationTool_handle.setProperty("CalibSequence","JetArea_Residual_EtaJES_GSC_Smear"                              );
JetCalibrationTool_handle.setProperty("CalibArea"    ,"00-04-82"                                                       );
JetCalibrationTool_handle.setProperty("IsData"       ,false                                                            );
~~~
{: .source}

Finally, create an instance of the `JetCalibrationTool` class and configure it using the information specified above.
~~~c++
JetCalibrationTool_handle.retrieve();
~~~
{: .source}

## Use the Tool
We next need to loop over all jets in each event and apply the calibration to them before storing them. Locate your jet loop, and add a call to our `JetCalibrationTool_handle` to calibrate each jet.

~~~c++
// calibrate the jet
xAOD::Jet *calibratedjet;
JetCalibrationTool_handle->calibratedCopy(*jet,calibratedjet);
~~~
{: .source}

Once you have calibrated it, save that instead of the original `jet` object.
~~~c++
jets_raw.push_back(*calibratedjet);

// perform kinematic selections and store in vector of "selected jets"
if(jet_selector.isJetGood(calibratedjet)){
  jets_kin.push_back(*calibratedjet);
}
~~~
{: .source}

Finally delete the pointer to de-allocate the memory associated with the `calibratedjet`.
~~~c++
// cleanup
delete calibratedjet;
~~~
{: .source}

> ## Memory Management Quiz
>
> Why do we need to delete the `calibratedjet` object?
>
> > ## Solution
> >
> > We store our calibrated jets in side a `std::vector<xAOD::Jet>`, which is a list of jet objects and not pointers. Thus when we add a jet to the list, we allocate new memory with a copy of the value pointed to by `calibratedjet`.
> >
> > When writing your analysis code, you might want to take a look at shallow copies. These will store a copy of your `jet` object, but only allocate memory for attributes that you change. Thus being more memory efficient.
> {: .solution}
{: .challenge}


> ## Multiple Histograms
>
> Modify the `AnalysisPayload` program to store the histograms for both calibrated and uncalibrated jet. You might want to create a new class called `JetHistograms` that manages the two jet histograms for a given jet collection.
>
{: .challenge}

# Updating the Dependenciy List
`AnalysisPayload` now uses code from two central packages; `AsgTools` and `JetCalibToolsLib`. We need to update our executable target to let CMake know that these are now dependencies.


> ## Solution
>
> Your `AnalysisPayload/CMakeLists.txt` should now contain the following.
>
> ~~~cmake
> ATLAS_ADD_EXECUTABLE ( AnalysisPayload util/AnalysisPayload.cxx
>                      LINK_LIBRARIES  xAODEventInfo
>                                      xAODRootAccess
>                                      xAODJet
>                                      JetSelectionHelperLib
>                                      AsgTools
>                                      JetCalibToolsLib)
> ~~~
> {: .source}
>
{: .solution}


# Look at the Results
Before running your new code, make a copy of your old results. We will be compare the new histograms with the old ones to understand the effect of calibration.

```shell
cp myOutputFile.root uncalibOutputFile.root
```

Then compile your new code and run!



{% include links.md %}
