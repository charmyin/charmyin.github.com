---
layout: post
title: "Debug Python programs remotely in eclipse"
description: "How to debug python program in eclipse remotely"
category: [python]
tags: [python, debug]
---

---------------------------------------

##Remote Debugger

In PyDev you can debug a remote program (a file that is not launched from within Eclipse).

The steps to debug an external program are:

    Start the remote debugger server
    Go to the debug perspective
    Start the external program with the file 'pydevd.py' in its pythonpath
    Call pydevd.settrace()
    Let's see a simple 'step-by-step' example on how this works:

###1. Start the remote debugger server:

To start the remote debugger server, you have to click the green button pointed by '1' in the image below. After doing that, it will show a message in the console (indicated by '2') to confirm that the server is listening for incoming connections.

**Note:** Those buttons should be present at the debug perspective and they can be enabled in other perspectives through Window > Customize perspective > Command groups availability > PyDev debug.


![RemoteDebugger]({{ site.url }}/assets/images/2014-11-09/remotedebugger1.png)

**Image**: Remote Debugger Server

###2. Go to the debug perspective:

This is needed because it has no actual 'signal' that it reached a breakpoint when doing remote debugging. So, if you already have it open, just cycle to it with Ctrl+F8. Otherwise, go to the menu: window > Open Perspective > Other > Debug.

Note that there should appear a process named 'Debug Server' in the debug view (see '1' in the image below).


![RemoteDebugger]({{ site.url }}/assets/images/2014-11-09/remotedebugger2.png)

Image: Debug perspective

###3. Make sure pydevd.py is in your pythonpath:

This file is included in the org.python.pydev plugin. So, you'll have to add it to the pythonpath. It's exact location will depend upon the eclipse location and the plugin version, being something like:

    eclipse/plugins/org.python.pydev_x.x.x/pysrc/pydevd.py

(so, the container folder must be in your pythonpath). If you choose to execute it from another machine, you need to copy all the files within that folder to the target machine in order to be able to debug it (if the target machine does not have the same paths as the client machine, the file pydevd_file_utils.py must be edited to properly translate the paths from one machine to the other - see comments on that file).

###4. Call pydevd.settrace():

 Now that the pydevd.py module is already on your pythonpath, you can use the template provided: 'pydevd' to make the call: import pydevd;pydevd.settrace(). When that call is reached, it will automatically suspend the execution and show the debugger.


![RemoteDebugger]({{ site.url }}/assets/images/2014-11-09/remotedebugger3.png)

Image: pydevd.settrace called

###Django remote debugging with auto-reload

By default, PyDev will add a --noreload flag when creating a Django run configuration, so that it works with the default debugger, but it's also possible to debug an application with auto-reload provided that some steps are followed to enable PyDev support in that case.

To do that, edit the launch that PyDev created (run > run configurations > PyDev Django) and remove the noreload flag and edit your manage.py so that the lines:

      #Add pydevd to the PYTHONPATH (may be skipped if that path is already added in the PyDev configurations)
      import sys;sys.path.append(r'path_to\pydev\plugins\org.python.pydev\pysrc')

      import pydevd

      pydevd.patch_django_autoreload(patch_remote_debugger=True, patch_show_console=True)
      are added BEFORE the if _name_ == "_main_".

With that change, the breakpoints should be gotten whenever a run is done (note that from now on, launches should only be done in 'regular' mode from now on and the debug server must be already started in the Eclipse side).

To disable the debugging, those lines must be removed from manage.py.

An interesting thing to note is that when you kill the 'parent django' process from Eclipse, the subprocesses it created won't be terminated at the same time, but they should be terminated when a code-change is done (in which case the parent process would create a new 'reload process', if it was still alive).

Note that the patch_show_console=True will make a separate window be shown for each process (and not be shown in Eclipse itself) – it has a benefit that you'll be able to stop the process with Ctrl+C in that window and the downside that you won't have the output in the Eclipse console.

### Important Notes

**NOTE 1:** the settrace() function can have an optional parameter to specify the host where the remote debugger is listening. E.g.: pydevd.settrace('10.0.0.1')

**NOTE 2:** the settrace() function can have optional parameters to specify that all the messages printed to stdout or stderr should be passed to the server to show. E.g.: pydevd.settrace(stdoutToServer=True, stderrToServer=True)

**NOTE 3:** You can have the running program in one machine and PyDev on another machine, but if the paths are not exactly the same, some adjustments have to be done in the target machine:

Aside from passing the files in eclipse/plugins/org.python.pydev_x.x.x/pysrc to your target machine, the file pydevd_file_utils.py must be edited to make the path translations from the client machine to the server machine and vice-versa. See the comments on that file for detailed instructions on setting the path translations.
