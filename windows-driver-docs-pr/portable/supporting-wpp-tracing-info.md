---
description: Supporting WPP Tracing
title: Supporting WPP Tracing
ms.date: 04/20/2017
---

# Supporting WPP Tracing


Windows Software Trace PreProcessor (WPP), provides an efficient real-time event logging mechanism. WPP is the recommended way to log trace and error messages in a WPD driver.

WPP traces include the system timestamp and can be used to measure performance, for example, by measuring the elapsed time between function calls.

## <span id="Sample_Drivers_that_Support_WPP_Tracing"></span><span id="sample_drivers_that_support_wpp_tracing"></span><span id="SAMPLE_DRIVERS_THAT_SUPPORT_WPP_TRACING"></span>Sample Drivers that Support WPP Tracing


The following sample drivers show how to use WPP tracing:

-   WpdBasicHardwareDriver
-   WpdServiceSampleDriver

## <span id="Transitioning_from_OutputDebugString_to_WPP"></span><span id="transitioning_from_outputdebugstring_to_wpp"></span><span id="TRANSITIONING_FROM_OUTPUTDEBUGSTRING_TO_WPP"></span>Transitioning from OutputDebugString to WPP


The WpdHelloWorldDriver, WpdWudfSampleDriver, and the WpdMultiTransportDriver log error messages by using a CHECK\_HR function that wraps around the **OutputDebugString** method. While this method is easy to use during development, it requires an active debugger connection in order to view the traces. The OutputDebugString method should be used minimally in final code for a release. WPP tracing is much more lightweight, flexible, and preferable for a range of tracing applications: logging errors for diagnosing failures, tracing code execution during development, for example.

In order to transition a driver from using **OutputDebugString** to use WPP, complete the following steps:

1.  Remove the original CHECK\_HR() function definition (it can be found in *WpdHelloWorldDriver.cpp* or *WpdWudfSampleDriver.cpp* for the WPD samples) and its declaration in *Stdafx.h*.
2.  Update the *Stdafx.h* file with the WPP macros, which are described later in this topic.
3.  Add a RUN\_WPP directive to the source files for your driver.
4.  Add the WPP initialization and cleanup routines to DllMain method for your driver.
5.  For each driver source file, add a *\#include &lt;filename&gt;.tmh*after the existing include statements.
6.  Rebuild your driver. Because you replaced the CHECK\_HR function with an equivalent WPP macro that has the same signature, no changes are needed for any code that uses CHECK\_HR.

The WPP preprocessor processes *Stdafx.h* for tracing macros, and generates a Trace Message Header (tmh) file for each .CPP file in the obj folders.

If you inspect the preprocessor output for each .CPP file that calls CHECK\_HR (generated by using make *WpdBaseDriver.pp*), you notice that all CHECK\_HR trace statements have been wrapped by WPP function calls, while the tracing text and WPP functions are defined in *WpdBaseDriver.tmh*.

The most common WPP-related compilation errors you can encounter are due to mismatched format identifiers and parameters (for example, if your format string contains an extra %d that does not match an argument) or missing entries in the WPP\_CONTROL\_GUIDS macro.

## <span id="updating_the_stdafx.h_file"></span><span id="UPDATING_THE_STDAFX.H_FILE"></span>Updating the Stdafx.h File


The following code example contains the updates to make to *Stdafx.h* if your driver currently supports **OutputDebugString**.

```ManagedCPlusPlus
#define MYDRIVER_TRACING_ID      L"Microsoft\\WPD\\ServiceSampleDriver"

#define WPP_CONTROL_GUIDS \
    WPP_DEFINE_CONTROL_GUID(ServiceSampleDriverCtlGuid,(f0cc34b3,a482,4dc0,b978,b5cf42aec4fd), \
        WPP_DEFINE_BIT(TRACE_FLAG_ALL)                                      \
        WPP_DEFINE_BIT(TRACE_FLAG_DEVICE)                                   \
        WPP_DEFINE_BIT(TRACE_FLAG_DRIVER)                                   \
        WPP_DEFINE_BIT(TRACE_FLAG_QUEUE)                                    \
        )

#define WPP_LEVEL_FLAGS_LOGGER(lvl,flags) \
           WPP_LEVEL_LOGGER(flags)

#define WPP_LEVEL_FLAGS_ENABLED(lvl, flags) \
           (WPP_LEVEL_ENABLED(flags) && WPP_CONTROL(WPP_BIT_ ## flags).Level >= lvl)

//
// This comment block is scanned by the trace preprocessor to define our
// TraceEvents function.
//
// begin_wpp config
// FUNC Trace{FLAG=TRACE_FLAG_ALL}(LEVEL, MSG, ...);
// FUNC TraceEvents(LEVEL, FLAGS, MSG, ...);
// end_wpp

//
// This comment block is scanned by the trace preprocessor to define our
// CHECK_HR function.
//
//
// begin_wpp config
// USEPREFIX (CHECK_HR,"%!STDPREFIX!");
// FUNC CHECK_HR{FLAG=TRACE_FLAG_ALL}(hrCheck, MSG, ...);
// USESUFFIX (CHECK_HR, " hr= %!HRESULT!", hrCheck);
// end_wpp

//
// PRE macro: The name of the macro includes the condition arguments FLAGS and EXP
//            define in FUNC above
//
#define WPP_FLAG_hrCheck_PRE(FLAGS, hrCheck) {if(hrCheck != S_OK) {

//
// POST macro
// The name of the macro includes the condition arguments FLAGS and EXP
//            define in FUNC above
#define WPP_FLAG_hrCheck_POST(FLAGS, hrCheck) ; } }

//
// The two macros below are for checking if the event should be logged and for
// choosing the logger handle to use when calling the ETW trace API
//
#define WPP_FLAG_hrCheck_ENABLED(FLAGS, hrCheck) WPP_FLAG_ENABLED(FLAGS)
#define WPP_FLAG_hrCheck_LOGGER(FLAGS, hrCheck) WPP_FLAG_LOGGER(FLAGS)
```

## <span id="Adding_the_WPP_Initialization_and_Cleanup_Routines_to_DllMain"></span><span id="adding_the_wpp_initialization_and_cleanup_routines_to_dllmain"></span><span id="ADDING_THE_WPP_INITIALIZATION_AND_CLEANUP_ROUTINES_TO_DLLMAIN"></span>Adding the WPP Initialization and Cleanup Routines to DllMain


After you update the *Sources* file, update the DllMain routine for the driver, as shown in the following code example:

```ManagedCPlusPlus
// DLL Entry Point
extern "C" BOOL WINAPI DllMain(HINSTANCE hInstance, DWORD dwReason, LPVOID lpReserved)
{    
if(DLL_PROCESS_ATTACH == dwReason)    
{        
g_hInstance = hInstance;              
//        
// Initialize tracing.        
//        
WPP_INIT_TRACING(MYDRIVER_TRACING_ID);    
}    
else if (DLL_PROCESS_DETACH == dwReason)    
{        
//        
// Cleanup tracing.        
//        
WPP_CLEANUP();    
}    
return _AtlModule.DllMain(dwReason, lpReserved);
}
```

## <span id="Including_the_Trace_Message_Header_files"></span><span id="including_the_trace_message_header_files"></span><span id="INCLUDING_THE_TRACE_MESSAGE_HEADER_FILES"></span>Including the Trace Message Header files


After you update the DllMain routine, update each .CPP source file and include the corresponding trace-message header file. The following code example shows the inclusion of the *WpdBaseDriver.tmh* file in *WpdBaseDriver.cpp*.

```ManagedCPlusPlus
//
// WpdBaseDriver.cpp
//#include "stdafx.h"
#include "WpdBaseDriver.h"
// Include the WPP generated Trace Message Header (tmh) file for this .cpp
#include "WpdBaseDriver.tmh"
```

## <span id="related_topics"></span>Related topics


[WDK and Visual Studio build environment](../devtest/wdk-and-visual-studio-build-environment.md)

 

