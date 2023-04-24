# Software Distribution

### Software Distribution

To clarify which files may be distributed with your applications, the following are the files and the distribution terms under which they may be used for the SDK.

Note that open-source projects have the right to include the header files within their distributions, which may then be used with dynamic loading of the NDI libraries.

### Header files. (NDI\_SDK\_DIR\Include\\\*.h) <a href="#_toc131521264" id="_toc131521264"></a>

These files may be distributed with open-source projects under the terms of the MIT license and may be included in open-source projects (see “Dynamic Loading” section for preferred mechanism). However, the requirements of these projects in terms of visual identification of NDI shall be as outlined within the License section above.

### Binary files (NDI\_SDK\_DIR\Bin\\\*.\*) <a href="#_toc131521265" id="_toc131521265"></a>

You may distribute these files within your application if your EULA terms cover the specific requirements of the NDI SDK EULA, and your application covers the terms of the License section above.

### Redistributables (NDI\_SDK\_DIR\Redist\\\*.exe) <a href="#_toc131521266" id="_toc131521266"></a>

You may distribute the NDI redistributable and install them within your own installer. However, you must make all reasonable effort to keep the versions you distribute up to date. You may use the command line with /verysilent to install without any user intervention, but if you do then you must ensure that the terms of the NDI license agreement are fully covered elsewhere in your application.

An alternative is to provide a user link to the NDI provided download of this application at [http://ndi.link/NDIRedistV5](http://ndi.link/NDIRedistV5). At runtime, the location of the NDI runtime DLLs can be determined from the environment variable NDI\_RUNTIME\_DIR\_V5.

### Content Files (NDI\_SDK\_DIR\LOGOS\\\*.\*) <a href="#_toc131521267" id="_toc131521267"></a>

You may distribute all files in this folder as you need and use them in any marketing, product, or web material. Please refer to the guidelines within the “NDI Brand Guidelines” which are included within this folder.

### Utilities <a href="#_toc131521272" id="_toc131521272"></a>

To make the library easy to use, the SDK includes several utilities that can be used to convert between common formats. For instance, conversion between different audio formats is provided as a service.

### Command line tools <a href="#_toc131521273" id="_toc131521273"></a>

There are also several important command line tools within the SDK, including a discovery server implementation, and a command line application that can be used for recording.

{% content-ref url="command-line-tools.md" %}
[command-line-tools.md](command-line-tools.md)
{% endcontent-ref %}
