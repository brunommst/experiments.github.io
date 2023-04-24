# NDI-Find

### NDI-Find

This is provided to locate sources available on the network and is normally used in conjunction with NDI-Receive. Internally, it uses a cross-process P2P mDNS implementation to locate sources on the network. (It commonly takes a few seconds to locate all the sources available, since this requires other running machines to send response messages.)

Although discovery uses mDNS, the client is entirely self-contained; Bonjour (etc.) are not required. mDNS is a P2P system that exchanges located network sources and provides a highly robust and bandwidth-efficient way to perform discovery on a local network.

On mDNS initialization (often done using the NDI-Find SDK), a few seconds might elapse before all sources on the network are located. Be aware that some network routers might block mDNS traffic between network segments.

Creating the find instance is very similar to the other APIs – one fills out a NDIlib\_find\_create\_t structure to describe the device that is needed. It is possible to specify a NULL creation parameter in which case default parameters are used.

If you wish to specify the parameters manually, then the member values are as follows:

| Supported Values             |                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| show\_local\_sources (bool)  | This flag tells the finder whether it should locate and report NDI send sources that are running on the current local machine.                                                                                                                                                                                                                                                                                          |
| p\_groups (const char\*)     | This parameter specifies groups for which this NDI finder will report sources. A full description of this parameter and what a NULL default value means is provided in the description of the NDI-Send SDK.                                                                                                                                                                                                             |
| p\_extra\_ips (const char\*) | <p>This parameter will specify a comma separated list of IP addresses that will be queried for NDI sources and added to the list reported by NDI find.</p><p>These IP addresses need not be on the local network and can be in any IP visible range. NDI find will be able to find and report any number of NDI sources running on remote machines and will correctly observe them coming online and going offline.</p> |

Once you have a handle to the NDI find instance, you can recover the list of current sources by calling NDIlib\_find\_get\_current\_sources at any time. This will _immediately_ return with the current list of located sources.

The pointer returned by NDIlib\_find\_get\_current\_sources is owned by the finder instance, so there is no reason to free it. It will be retained until the next call to NDIlib\_find\_get\_current\_sources, or until the NDIlib\_find\_destroy function is destroyed.

You can call NDIlib\_find\_wait\_for\_sources to wait until the set of network sources has been changed; this takes a time-out in milliseconds. If a new source is found on the network, or one has been removed before this time has elapsed, the function will return true immediately. If no new sources are seen before the time has elapsed, it will return false.

The following code will create an NDI-Find instance, and then list the current available sources. It uses NDIlib\_find\_wait\_for\_sources to sleep until new sources are found on the network and, when they are seen, calls NDIlib\_find\_get\_current\_sources to get the current list of sources:

// Create the descriptor of the object to create

NDIlib\_find\_create\_t find\_create;

find\_create.show\_local\_sources = true;

find\_create.p\_groups = NULL;

// Create the instance

NDIlib\_find\_instance\_t pFind = NDIlib\_find\_create\_v2(\&find\_create);

if (!pFind)

/\* Error \*/;

while (true) // You would not loop forever of course !

{

// Wait up till 5 seconds to check for new sources to be added or removed

if (!NDIlib\_find\_wait\_for\_sources(pFind, 5000))

{

// No new sources added!

printf("No change to the sources found.\n");

}

else

{

// Get the updated list of sources

uint32\_t no\_sources = 0;

const NDIlib\_source\_t\* p\_sources =

NDIlib\_find\_get\_current\_sources(pFind, \&no\_sources);

// Display all the sources.

printf("Network sources (%u found).\n", no\_sources);

for (uint32\_t i = 0; i < no\_sources; i++)

printf("%u. %s\n", i + 1, p\_sources\[i].p\_ndi\_name);

}

}

// Destroy the finder when you’re all done finding things

NDIlib\_find\_destroy(pFind);

It is important to understand that mDNS discovery might take some time to locate all network sources. This means that an ‘early’ return to NDIlib\_find\_get\_current\_sources might not include all the sources on the network; these will be added (or removed) as additional or new sources are discovered. It commonly takes a few seconds to discover all sources on a network.

For applications that wish to list the current sources in a user interface menu, the recommended approach would be to create an NDIlib\_find\_instance\_t instance when your user interface is opened and then – each time you wish to display the current list of available sources – call NDIlib\_find\_get\_current\_sources.
