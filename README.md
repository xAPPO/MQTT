# MQTT
MQTT client for Hubitat


Release notes for alpha2  
14th May 2019
(first broader release)

# Features

a) Enabling inbuilt HE devices*  to publish and be controllable through MQTT either using a basic topic structure or a limited homie3 structure (or both)

b) Enable MQTT devices* to be 'mirrored'  as virtual devices and controlled within HE. All the topics are configurable as are the state values..

c) Enable automatic discovery and selected inclusion of devices* using the homie3 protocol (promoted by openHAB)

d) Enable automatic discovery and selected inclusion of devices* using the Home Assistant statestream protocol (offered by HA) - a small  automation script is provided for HA that enables the statestream protocol for control > HA too.

*N.B.This version currently supports 'switch' (onoff) and 'switchLevel' (dim) capabilities only - but will be expanded to include others

# Future Features:

1) Support of many more device capabilities beyond 'onoff' and 'dim'
2) Add sensor support for reporting of sensor values - (let me know which ones you use)
3) Support Home Assistant MQTT discovery protocol bidirectionally - HE devices auto discovered by HA and devices advertising using this protocol discoverable by HE
4) Know Issue: currently,  at startup, HE devices do not publish their current state to MQTT. They do on change. [FIXED]
5) Better support for decimal maxBrightness values
6) Support for multiple homie discovered devices.
7) Support for JSON payloads
8) More complete support for homie3 specification from Hubitat - enough so openHAB# discovery is happy.

  #I have not yet tried openHAB homie3 discovery but I suspect it will not work currently.  You will need at least openHAB 2.5 milestone 1 build, maybe later, and for various stability reasons I can't recommend you upgrade to that - and definately not post milestone 1 builds.  Also I suspect I need to better support some topics in the homie3 specification ($nodes $type $properties and maybe more)


# Instructions: 
(preliminary)

# Initial setup:

1) Install both the main app MQTT and the device drivers for MQTT Client, MQTT Dimmer and MQTT Switch
2) From 'Devices' "Add Virtual Device" selecting the "MQTT client" device driver, name the broker device as you wish, make up a Device Network ID e.g. the IP of the broker.
3) Configure this device to access your existing MQTT broker using the 'Preferences' section on the next screen in the device driver then click 'Done'
You must enter the IP (or URL) e.g. tcp://192.168.1.78:1883 ,  username / password are only required if your broker needs them.
4) Launch the MQTT app, it has two setup pages , the first covering individual devices and the second for 'discovery'
On the first page select from the 'MQTT Broker' dropdown the MQTT client device you just configured in step 3
For the time being ignore all other options and select 'Next' on the first page and 'Done' on the second 
.. you should see something similar to this in the log.

Log:

	info MQTT: ================== Startup complete ==================
	info MQTT:     0 Hubitat devices enabled on MQTT
	info MQTT: ==================================================
	info MQTT: Total startup time will be around 0 seconds
	info MQTT: Skipping HA stateStream MQTT discovery
	info MQTT: Skipping homie MQTT discovery
	info MQTT> Connected as Hubitat_Development to MQTT broker tcp://192.168.1.78:1883
	info MQTT client alpha2 initialised
	info MQTT> Resetting MQTT connection
	info MQTT> Log Level set to 2
	info MQTT: Hubitat hub name is : Hubitat/Development
	info MQTT alpha2 Initialized
	info MQTT: MQTT Installed
	
Inparticular check that you get the "Connected as Hubitat_xxxx to MQTT broker" success result, otherwise nothing will work.

Now choose from the key features listed above which feature(s) a) b) c) or d) you are trying to setup the MQTT app for.
I recommend choosing just one for now.  a) is the easiest.

Then follow the option a)-d) below that you need.....

Lastly two app configuration options re. 'LogLevel' and importantly 'Deleting MQTT discovered devices' are discussed right at the end of these instructions in 'Additional Notes'.  Please read those too.
__________________________________________________________________________________________________________________________
# a) Enabling inbuilt 'real' HE devices*  to publish and be controllable through MQTT
 
 From 'Apps' run the MQTT app again and select the way (topic structure) that you would like to use to publish your devices to MQTT.
 You have two choices - a 'Hubitat basic MQTT' topic structure or a "homie3 protocol"  one.
 N.B. Before final release I may remove 'Hubitat Basic' as the 'homie3' topic structure is much the same
 
Hubitat Basic:
 
	Hubitat/HubName/deviceName/capability/<value>           that reports current state
	
	Hubitat/HubName/deviceName/capability/set/<value>       that allows you to send commands to make a state change

homie3:

	homie/hubitat_hubname/deviceName/capability/<value>
	
	homie/hubitat_hubname/deviceName/capability/set/<value>
	
plus several required configuration topics

Now enable which HE devices that you wish to publish to MQTT by selecting them in the two dropdowns

HE Switch Devices > MQTT

HE Dimmer Devices > MQTT

click 'Update then 'Next' then 'Done'

These devices will now update status changes to MQTT and be controllable from MQTT, using /set appended topic.
__________________________________________________________________________________________________________________________
# b) Enable MQTT devices* to be 'mirrored' as virtual devices and controlled within HE.

This requires you create virtual MQTT devices , currently two are provided "MQTT Switch" and "MQTT Dimmer".

From 'Devices' select 'Add Virtual Device' and choose one of these 'user' devices and name it.
I recommend prefixing the name with a special character e.g. ~ as this will allow them to group together and be easily identifiable when selecting them later.

For the time being type in any random set of characters for the 'Device Network Id' - it will be renamed automatically later.

Now 'Save' the device and you will be presented with another screen, scroll down to the 'Preferences' section.

If you selected 'MQTT Switch" there are 4 preferences to configure.  MQTT Dimmer requires 3 more.

MQTT Switch devices:		State Topic, Command Topic, StateON, StateOFF  

MQTT Dimmer  devices:		as above plus Level Topic, Level Command Topic, Maximum Brightness Level

State Topic is the topic on MQTT where the device reports its current onoff state

StateON is what the device posts as its payload when the device is ON (case sensitive) 		e.g. On on ON True true 1 yes etc.

StateOFF is what the device posts as its payload when the device is OFF (case sensitive)  	e.g. Off off OFF False false 0 no etc.

Command Topic is the topic you send a command to to make the device change state


Level Topic is the topic on MQTT where the device reports its current level

Level Command Topic is the topic you send a command to to make the device change level

Maximum Brightness Level is the value of full brightness for this device e.g. 100 255 1.0 a decimal point is significant as it will 
support real numbers rather than integers.
	N.B. Hubitat uses 0-100 for level internally so everything will get scaled accordingly
	
If your device does not offer some of these options you may leave them blank, however either 'State Topic' or 'Level Topic' is required as it is used to create the Device Network ID.
	
Once these parameters are entered click 'Save Preferences' followed by 'Save Device'

Then you have to restart the MQTT app.

   This next step is a bit of annoyance but currently necessary. From the 'MQTT Manual Devices' dropdown select the virtual MQTT device(s) that you have just added. 
   
   This dropdown also includes 'discovered' devices, you want to select just the virtual ones.  If you did prefix them with a special character e.g. ~ it will be much more obvious which these devices are.
   
   Click through 'Next' 'Done'.
   (I will look into eliminating this extra step)
   
   
The current values for the state will be updated and control should be available from HE
__________________________________________________________________________________________________________________________
# c) Enable automatic discovery and selected inclusion of devices* using the homie3 protocol (promoted by openHAB)

# d)  Enable automatic discovery and selected inclusion of devices* using the Home Assistant statestream protocol

To use either or both of these discovery protocols just enable them on the second page of the app configuration

MQTT Discovery:

	HA statestream
	
	homie 3 protocol
	
The HA statestream protocol needs to know the name of the HA statestream topic that HA has been configured to publish on.
Configure this using "HA Statestream topic"

The homie 3 protocol needs to know which device within the homie topic tree to discover. Currently only one homie device can be discovered.
Configure this under "homie device topic name"

e.g. if a homie topic looks like homie/mainsys/hallway/onoff then you would enter mainsys here as that is the device identifier (mainsys has sub nodes like hallway)

Both of these are configured on the second page of the app configuration (click 'Next')

Click 'Done'

The app will then start up which will take around a minute and automatically discover all the devices you have asked it too. 
It will then populate the discovered device dropdown lists so you can later select which devices you are interested in.

Now restart the app on the second page you will see four lists... choose the MQTT devices you wish to 'mirror' within Hubitat.

Discovered homie switches

Discovered homie dimmers

Discovered HA switches

Discovered HA lights

After selecting you devices and clicking 'Done' all these will be added to Hubitat, providing realtime status and control.

# For Home Assistant Users ONLY

HA users will need to include the following in your HA installation.

Configure HA to use MQTT statestream by including the following in your configuration.yaml (choose your own base_topic: name)

	mqtt_statestream:

  	  base_topic: HAStateStream50
  
  	  publish_attributes: true
  
Add the included "automations.yaml" to your automations.yaml file in HA , or if you don't use this then to configuration.yaml.
This enables bidirectional control via statestream

__________________________________________________________________________________________________________________________

# Additional Notes:

After sucessful configuration and Startup you should see something like the following in the log at 'INFO' level

	info MQTT: ================ Startup complete ================
	info MQTT:     Discovered 70 HA light devices
	info MQTT:     Discovered 47 HA switch devices
	info MQTT:     Discovered 13 homie dim devices
	info MQTT:     Discovered 10 homie onoff devices
	info MQTT:     23 Hubitat devices enabled on MQTT
	info MQTT: ==================================================


There are two other app configuration options to be aware of.
 
# LogLevel:

LogLevel can be set to limit the logging entries the app posts. It is recommended to set it at level 'INFO'
but it can be minimised to "WARN".	Any log messages that show in red as 'ERROR' need investigation (and reporting)
 
# Purge Discovered Devices

WARNING: Setting this will delete all MQTT 'discovered devices' when you click 'Done'

Sounds ominous eh, it's really not so drastic.  Switching this ON will clear all the HE devices that were added through 'discovery'. 
They can be recreated by re-running the app. However the unique internal device ID for the device will then change and so the newly created devices will be just that 'new'.
This really only has an impact on HE internal references to the original device e.g. Dashboard devices and RuleMachine rules. You will have to recreate these.
The main use is to clear out all your discovered devices but once running happily you should leave this set to OFF. 
You can always disable devices on the second page of the app config and individually delete previously created devices manually.
	
	
	

	
	
