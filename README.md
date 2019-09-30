# MQTT

MQTT client for Hubitat

The licence for alpha4 has been updated to be restrictive whilst in testing.  I intend the final version to be under a much less restrictive licence although it will likely require my express written permission to publish any derivate code and no commercial offering will be allowed.

Pre Release notes for alpha4  
30th September 2019

This is the pre8 'Adventurer' release of alpha 4 - at this time it is still a work in progress but reasonably useable I hope. Please backup first though and use at your own risk.   You must install the app and the three drivers. They have all changed.

You may lose/have to recreate any virtual devices that you have that import devices from MQTT into HE.  If this is a concern contact me first as there is a way around it. 

This read me will be updated again very shortly with the new features for alpha4 - meanwhile here are the V3 instructions.


# Features

a) Enabling inbuilt HE devices*  to publish and be controllable through MQTT either using a basic topic structure or a compliant homie3 structure (or both)

b) Enable MQTT devices* to be 'mirrored'  as virtual devices and controlled within HE. All the topics are configurable as are the state values..

c) Enable automatic discovery and selected inclusion of devices* and sensors using the homie3 protocol (promoted by openHAB)

d) Enable automatic discovery and selected inclusion of devices* and sensors using the Home Assistant statestream protocol (offered by HA) - a small  automation script is provided for HA that enables the statestream protocol for control > HA too.

e) Enable automatic discovery and selected inclusion of Hubitat devices* and sensors into Home Assistant using the HA MQTT Discovery protocol.

f) Athoms' Homey controller also supports MQTT and the homie3 protocol. They work really well together automatically discovering each others devices and enabling realtime synchronisation and control between the controllers. Two HE's could work this way too but HubConnect is probably an easier option.  HubConnect virtual devices of course work with this MQTT app too.

*N.B.This version currently supports 'switch' (onoff), 'switchLevel' (dim) colour and sensor capabilities only - but will be expanded to include others.  Sensors are still a work in progress.

# Future Features:

1) Support of many more device capabilities beyond 'onoff' and 'dim' and 'color'
2) Expand sensor support for better mapping of sensor data to devices.
3) Support Home Assistant MQTT discovery protocol bidirectionally - HE devices auto discovered by HA and devices advertising using this protocol discoverable by HE. (may drop this latter support for HA Discovery > HE)
4) Know Issue: currently,  at startup, HE devices do not publish their current state to MQTT. They do on change. [FIXED]
5) Better support for decimal maxBrightness values
6) Support for multiple homie discovered devices.
7) Support for JSON payloads
8) More complete support for homie3 specification from Hubitat - enough so openHAB# discovery is happy. [DONE]
9) Support multiple MQTT brokers (considering)

  #I have not yet tried openHAB homie3 discovery but I suspect it may not work currently.  You will need at least openHAB 2.5 milestone 1 build, maybe later, and for various stability reasons I can't recommend you upgrade to that - and definately not post milestone 1 builds. This is an openHAB issue rather than this app.  
  #Update -  OpenHAB 2.5M3 (milestone 3) has been released. Huge changes have happened in OH between 2.5M1 and 2.5M3 so I look forward to seeing how my app works with OH 2.5M3. Please let me know


# Instructions: 
(preliminary)

# Initial setup:

1) Install both the main app MQTT and the device drivers for MQTT Client, MQTT Dimmer and MQTT Switch
2) From 'Devices' "Add Virtual Device" selecting the "MQTT client" device driver, name the broker device as you wish, make up a Device Network ID e.g. the IP of the broker.
3) Configure this device to access your existing MQTT broker using the 'Preferences' section on the next screen in the device driver then click 'Done'.
You must enter the IP (or URL) e.g. tcp://192.168.1.78:1883 ,  username / password are only required if your broker needs them.
4) Launch the MQTT app, it has two setup pages each with multiple sections , the first page covers individual devices and the second for 'discovery'. The second page is optional.
On the first page click 'Configuration' and from the 'MQTT Broker' dropdown select the MQTT client device you just configured in step 3
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
	info MQTT client alpha3 initialised
	info MQTT> Resetting MQTT connection
	info MQTT> Log Level set to 2
	info MQTT: Hubitat hub name is : Hubitat/Development
	info MQTT alpha3 Initialized
	info MQTT: MQTT Installed
	
Inparticular check that you get the "Connected as Hubitat_xxxx to MQTT broker" success result and no other errors, otherwise nothing will work.

Now choose from the key features listed above which feature(s) a) b) c) or d) you are trying to setup the MQTT app for.
I recommend choosing just one for now.  a) is the easiest.

Then follow the option a)-d) below that you need.....

Lastly two app configuration options re. 'LogLevel' and importantly 'Deleting MQTT discovered devices' are discussed right at the end of these instructions in 'Additional Notes'.  Please read those too.
__________________________________________________________________________________________________________________________
# a) Enabling inbuilt 'real' HE devices*  to publish and be controllable through MQTT
 
 From 'Apps' run the MQTT app again and select the way (topic structure) that you would like to use to publish your devices to MQTT.
 You have two choices - a 'Hubitat basic MQTT' topic structure or a "homie3 protocol" compatible one.
 N.B. Before final release I may remove 'Hubitat Basic' as the 'homie3' topic structure is much the same
 
Hubitat Basic:
 
	Hubitat/HubName/deviceName/capability/<value>           that reports current state
	
	Hubitat/HubName/deviceName/capability/set/<value>       that allows you to send commands to make a state change

homie3:

	homie/hubitat_hubname/deviceName/capability/<value>
	
	homie/hubitat_hubname/deviceName/capability/set/<value>
	
plus several required configuration topics

When enabling the homie protocol there is an additional '..retain homie states' option. Enabling this causes the last reported state of your devices to be retained on your MQTT broker. Any new device device connecting to MQTT will receive these states , although these may be old (residual) values.  If you disable it any new device connecting to MQTT will get no information until your device updates its state, but the value will be known to be current. 

Now enable which HE devices that you wish to publish to MQTT by selecting them in the two dropdowns

HE Switch Devices > MQTT

HE Dimmer Devices > MQTT

click 'Update then 'Next' then 'Done'

These devices will now update status changes to MQTT and be controllable from MQTT, using /set appended topic.
__________________________________________________________________________________________________________________________
# b) Enable MQTT devices* to be 'mirrored' as virtual devices and controlled within HE.

This requires you create virtual MQTT devices , currently two are provided "MQTT Switch" and "MQTT Dimmer".

From 'Devices' select 'Add Virtual Device' and choose one of these 'user' devices and name it.
I recommend prefixing the name with a special character e.g. ~ as this will allow them to group together in lists and be easily identifiable when selecting them later.

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

Maximum Brightness Level is the value of full brightness for this device e.g. 100 255 1.0 a decimal point is significant as it will then
use real numbers rather than integers.
	N.B. Hubitat uses 0-100 for level internally so everything will get scaled accordingly
	
If your device does not offer some of these options you may leave them blank, however either 'State Topic' or 'Level Topic' is required as it is used to create the Device Network ID.
	
Once these parameters are entered click 'Save Preferences' followed by 'Save Device'

Then you have to restart the MQTT app.

   This next additional step is a bit of annoyance but currently necessary. From the 'MQTT Manual Devices' dropdown select the virtual MQTT device(s) that you have just added. 
   
   This dropdown also includes 'discovered' devices, but you want to select just the virtual ones.  If you did prefix them with a special character when creating them e.g. ~  then it will be much more obvious which these devices are.
   
   Click through 'Next' 'Done'.
   (I will look into eliminating this additional step)
   
   
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

The homie 3 protocol needs to know which device within the homie topic tree to discover. Currently only one homie 'device' can be discovered although it further exposes all of it's nodes.
Configure this under "homie device topic name"

e.g. if a homie topic looks like homie/mainsys/hallway/onoff then you would enter mainsys here as that is the device identifier (mainsys has sub nodes like hallway)

Both of these are configured on the second page of the app configuration (click 'Next')

Click 'Done'

The app will then start up which will take a couple of minutes and automatically discover all the devices you have asked it too. 
It will then populate the discovered device dropdown lists so you can later select which devices you are interested in.

Now restart the app,  on the second page you will see four lists... choose the MQTT devices you wish to 'mirror' within Hubitat.

Discovered homie switches

Discovered homie dimmers

Discovered HA switches

Discovered HA lights

After selecting you devices and clicking 'Done' all these will be added to Hubitat, providing realtime status and control.

# For Home Assistant Users ONLY

HA users will need to include the following in your HA installation.

Configure HA to use MQTT statestream by including the following in your configuration.yaml (choose your own base_topic: name)

	mqtt_statestream:

  	  base_topic: HAStateStream50             # << your choice of topic name
  
  	  publish_attributes: true
  
Add the included "automations.yaml" to your automations.yaml file in HA , or if you don't use this then to configuration.yaml.
This enables bidirectional control via statestream

__________________________________________________________________________________________________________________________

__________________________________________________________________________________________________________________________
# e) Enable automatic discovery by Home Assitant of Hubitat devices (switch, dim, color, sensors currently)

To enable the advertising of Hubitat devices for discovery by HA.

First ensure HA has 'discovery' enabled and and note the topic that it is using (within the mqtt section of HA's configuration.yaml file).  You will need to restart HA if you edit configuration.yaml

	discovery: true
	discovery_prefix: homeassistant
  
Cick MQTT Publish Formats on the MQTT app first page
Enable HA MQTT discovery protocol
Enable homie 3 protocol on the MQTT app first page. (You must enable this as HA reads MQTT payloads from these homie topics)
In the HA Discovery Topic field enter the discovery_prefix value that HA is using (e.g. homeassistant if as above).
Click through 'Next' Done' 

Devices (onoff and dim only currently) that are enabled for MQTT within Hubitat will automatically just appear as 'switches' and 'lights' in your Home Assistant front end. You may to have press ctrl F5 to refresh the HA interface , and they may initially appear as 'unused entities' which you can check by clicking the 'hamburger' icon in the top right hand of recent HA builds.  These devices are bidirectionally realtime synched with Hubitat.  

When HA starts up it checks MQTT for discovered devices. If you want them to persist over HA restarts then ensure you check the 'retain' checkbox under the 'HA Discovery Topic" name in the MQTT app. If it is unchecked then HA will purge these devices when restarted , although it might warn you with yellow boxes the first time it notices they have gone.

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

LogLevel (within Configuration) can be set to limit the logging entries the app posts. It is recommended to set it at level 'INFO'
but it can be minimised to "WARN".	Any log messages that show in red as 'ERROR' need investigation (and reporting)
 
# Purge Discovered Devices

WARNING: Setting this will delete all MQTT 'discovered devices' when you click 'Done'

Sounds ominous eh, it's really not so drastic.  Switching this ON will clear all the HE devices that were added through 'discovery'. 
They can be recreated by re-running the app. However the unique internal device ID for the device will then change and so the newly created devices will be just that 'new'.
This really only has an impact on HE internal references to the original device e.g. Dashboard devices and RuleMachine rules. You will have to recreate these.
The main use is to clear out all your discovered devices but once running happily you should leave this set to OFF. 
You can always disable devices on the second page of the app config and individually delete previously created devices manually.
	
	
	

	
	
