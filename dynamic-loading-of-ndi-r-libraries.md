# Dynamic Loading of NDI® Libraries

At times you might prefer not to link directly against the NDI® libraries, loading them dynamically at run time instead. This can be of value in Open-Source projects.

There is a structure that contains all the NDI entry points for a particular SDK version; calling a single-entry point in the library will recover all these functions. The basic procedure is relatively simple, and an example is provided with the SDK.

### Locating the Library <a href="#_toc131521276" id="_toc131521276"></a>

You can of course include the NDI runtime within your application folder. Alternatively, on Windows you can install the NDI runtime and use an environment variable to locate it on disk. If you are unable to locate the library on disk, you may ask users to perform a download from a standardized URL. System dependent #defines are provided to make this a simple process:

NDILIB\_LIBRARY\_NAME is a C #define to represent the dynamic library name (as, for example, the dynamic library Processing.NDI.Lib.x64.dll).

NDILIB\_REDIST\_FOLDER is a C #define variable that references an environment variable to the installed NDI runtime library (for example, C:\Program Files\NDI\NDI 5 Runtime\\).

NDILIB\_REDIST\_URL is a C #define for a URL where the redistributable for your platform may be downloaded (for example, [http://ndi.link/NDIRedistV5](http://ndi.link/NDIRedistV5)).

On the Mac, it is not possible to specify global environment variables and so there is no standard way for the application to provide to specify a path. For this reason, the redistributable on MacOS is installed within /usr/local/lib.

It is our intent to make the process of cross platform loading of the run-times easier for the next version of NDI.

### Recovering the Function Pointers <a href="#_toc131521277" id="_toc131521277"></a>

Once you have located the library, you can look for a single exported function NDIlib\_v5\_load(). This function returns a structure of type NDIlib\_v5 that gives you a reference to every NDI function.

### Calling NDI functions <a href="#_toc131521278" id="_toc131521278"></a>

Once you have a pointer to NDIlib\_v5, you can replace every function with a simple new reference. For instance, to initialize a sender you can replace a call to NDIlib\_find\_create\_v2 in the following way:

```
NDIlib_find_create_v2(...) becomes p_NDILib->NDIlib_find_create_v2(...)// Some code
```
