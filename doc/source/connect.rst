How to start BAC0
===================================================
Define a bacnet network
----------------------------------------

Once imported, BAC0 will rely on a 'network' variable that will connect to the BACnet network you want to reach. This variable will be tied to a network interface (that can be a network card or a VPN connection) and all the traffice will pass on this variable.

More than one network variable can be created but only one connection by interface is supported.

Typically, we'll call this variable 'bacnet' to illustrate that it represents the network. But you can call it like you want.

This variable will also be passed to some functions when you will define a device for example. As the device needs to know on which network it can be found.

When creating the connection to the network, BAC0 needs to know the ip network of the interface on which it will work. It also needs to know the subnet mask (as BACnet operations often use broadcast messages).If you don't provide one, BAC0 will try to detect the interface for you.

.. note::
    If you use ios, you will need to provide a ip manually. The script is unable to detect the subnet mask yet. You will also have to modify bacpypes and allow 'ios' so it
    can be installed on pythonista.

By default, if Bokeh, Pandas and Flask are installed, using the connect script will launch the complete version. But you can also use the lite version if you want something simple.
    
Example::

    import BAC0
    bacnet = BAC0.connect()
    # or specify the IP you want to use / bacnet = BAC0.connect(ip='192.168.1.10/24')
    # by default, it will attempt an internet connection and use the network adapter
    # connected to the internet.
    # Specifying the network mask will allow the usage of a local broadcast address
    # like 192.168.1.255 instead of the global broadcast address 255.255.255.255
    # which could be blocked in some cases.
    # You can also use :
    # bacnet = BAC0.lite() to force the script to load only minimum features.
    # Please note that if Bokeh, Pandas or Flask are not installed, using connect() will in fact call the lite version.


Lite vs Complete
*****************

Lite
.............

Use Lite if you only want to interact with some devices without using the web 
interface or the live trending features. 
On small devices like Raspberry Pi on which Numpy and Pandas are not installed, 
it will run without problem.

To do so, use the syntax::

    bacnet = BAC0.lite(ip='xxx.xxx.xxx.xxx/mask')

On a device without all the module sufficient to run the "complete" mode, using
this syntax will also run BAC0 in "Lite" mode::

    bacnet = BAC0.connect()

> Device ID 
> 
> It's possible to define the device ID you want in your BAC0 instance by
> using the `deviceId` argument
    
Complete
............

Complete will launch a web server with bokeh trending features. You will be able to 
access the server from another computer if you want.

To do so, use the syntax::

    bacnet = BAC0.connect(ip='xxx.xxx.xxx.xxx/mask')

And log to the web server pointing your browser to http://localhost:8111

.. note::
   To run BAC0 in "complete" mode, you need to install supplemental packages :
       * flask
       * flask-bootstrap
       * bokeh
       * pandas (numpy)
   To install bokeh, using "conda install bokeh" works really well. User will also needs to "pip install" everything else.

.. note::
   To run BAC0 in "complete" mode using a RaspberryPi_, I strongly recommend using the package
   berryconda_. This will install Pandas, numpy, already compiled for the Pi and give you access
   to the "conda" tool. You'll then be able to "conda install bokeh" and everythin will be working fine. If you try
   to "pip install pandas" you will face issues as the RPi will have to compile the source and it is
   a hard taks for a so small device. berryconda_ gives access to a great amount of packages already
   compiled for the Raspberry Pi.


Use BAC0 on a different subnect (Foreign Device)
***************************************************
In some situations (like using BAC0 with a VPN using TUN) your BAC0 instance
will run on a different subnet than the BACnet/IP network.

BAC0 support being used as a foreign device to cover those cases.

You must register to a BBMD (BACnet Broadcast Management Device) that will organize
broadcast messages so they can be sent through diferent subnet and be available for BAC0.

To do so, use the syntax::

    my_ip = '10.8.0.2/24'
    bbmdIP = '192.168.1.2:47808'
    bbmdTTL = 900
    bacnet = BAC0.connect(ip='xxx.xxx.xxx.xxx/mask', bbdmAddress=bbmdIP, bbmdTTL=bbmdTTL)
    
Discovering devices on a network
*********************************
BACnet protocole relies on "whois" and "iam" messages to search and find devices. Typically, 
those are broadcast messages that are sent to the network so every device listening will be 
able to answer to whois requests by a iam request.

By default, BAC0 will use "local broadcast" whois message. This mean that in some situation,
you will not see by default the global network. Local broadcast will not traverse subnets and 
won't propagate to MSTP network behind BACnet/IP-BACnet/MSTP router that are on the same subnet
than BAC0.

This is done on purpose because using "global broadcast" by default will create a great amount
of traffic on big BACnet network when all devices will send their "iam" response at the same
time.

Instead, it is recommended to be careful and try to find devices on BACnet networks one at a time.
For that though, you have to "already know" what is on your network. Which is not always the case.
This is why BAC0 will still be able to issue global broadcast whois request if explicitly told to do so.

The recommended function to use is ::

    bacnet.discover(networks=['listofnetworks'], limits=(0,4194303), global_broadcast=False)
    # networks can be a list of integers, a simple integer, or 'known'
    # By default global_broadcast is set to False 
    # By default, the limits are set to any device instance, user can choose to request only a
    # range of device instances (1000,1200) for instance


This function will trigger the whois function and get you results. It will also emit a special request
named 'What-si-network-number' to try to learn the network number actually in use for BAC0. As this function
have been added in the protocole 2008, it may not be available on all networks.

BAC0 will store all network number found in the property named `bacnet.known_network_numbers`. User can then 
use this list to work with discover and find everything on the network without issuing global broadcasts.
To make a discover on known networks, use ::

    bacnet.discover(networks='known')

Also, all found devices can be seen in the property `bacnet.discoveredDevices`. This list is filled with all
the devices found when issuing whois requests.

BAC0 also provide a special functions to get a device table with details about the found devices. This function
will try to read on the network for the manufacturer name, the object name, and other informations to present 
all the devices in a pandas dataframe. This is for presentation purposes and if you want to explore the network, 
I recommend using discover. 

Devices dataframe ::

    bacnet.devices

..note::
    WARNING. `bacnet.devices` may in some circumstances, be a bad choice when you want to discover
    devices on a network. A lot of read requests are made to look for manufacturer, object name, etc
    and if a lot of devices are on the network, it is recommended to use whois() and start from there.

BAC0 also support the 'Who-Is-Router-To-Network' request so you can ask the network and you will see the address
of the router for this particular BACnet network. The request 'Initialize-Router-Table' will be triggered on the 
reception of the 'I-Am-Router-To-Network' answer.

Once BAC0 will know which router leads to a network, the requests for the network inside the network will be 
sent directly to the router as unicast messages. For example ::

    # if router for network 3 is 192.168.1.2
    bacnet.whois('3:*') 
    # will send the request to 192.168.1.2, even if by default, a local broadcast would sent the request
    # to 192.168.1.255 (typically with a subnet 255.255.255.0 or /24)

Ping devices (monitoring feature)
**********************************
BAC0 includes a way to ping constantly the devices that have been registered. 
This way, when devices go offline, BAC0 will disconnect them until they come back
online. This feature can be disabled if required when declaring the network ::

    bacnet = BAC0.lite(ping=False)
    
By default, the feature is activated.

When reconnecting after being disconnected, a complete rebuild of the device is done.
This way, if the device have changed (a download have been done and point list changed)
new points will be available. Old one will not.

..note::
    WARNING. When BAC0 disconnects a device, it will try to save the device to SQL.

Routing Table
***************
BACnet communication trough different networks is made possible by the different 
routers creating "routes" between the subnet where BAC0 live and the other networks.
When a network discovery is made by BAC0, informations about the detected routes will
be saved (actually by the bacpypes stack itself) and for reference, BAC0 offers a way 
to extract the information ::

    bacnet.routing_table

This will return a dict with all the available information about the routes in this form : 

bacnet.routing_table
Out[5]: {'192.168.211.3': Source Network: None | Address: 192.168.211.3 | Destination Networks: {303: 0} | Path: (1, 303)}

.. _berryconda : https://github.com/jjhelmus/berryconda  
.. _RaspberryPi : http://www.raspberrypi.org
