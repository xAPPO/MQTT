IN PROGRESS

This will very shortly - not yet be updated for alpha 5 


# MQTT

MQTT client for Hubitat

The licence for alpha5 has been updated to be restrictive whilst in testing.  I intend the final version to be under a much less restrictive licence although it will likely require my express written permission to publish any derivate code and no commercial offering will be allowed.

You may not redistribute this code in any form to any other person. It is provided to members of a restricted alpha test group for the purpose of testing and feedback only.  You may use and modify this code for your own personal use only.  You may NOT fork this repository (please).

Pre Release notes for alpha5 
11th January 2020

This read me is only partially updated for alpha 5 - at this time .  Before installing please backup first though and as always use at your own risk, there is no warranty or accepted liability as to this app being suitable for any purpose whatsoever.   

You must install the MQTT app and the MQTT client driver.  The previous (alpha4) two drivers for 'MQTT switch' and 'MQTT dimmer' are no longer required and are deprecated / will no longer function.   Instead you can now use any of the 24 Hubitat virtual drivers.  The MQTT text driver is still available (optional) but is likely not required.

You may lose/have to recreate any virtual devices that you have that import devices from MQTT into HE.  If this is a concern contact me first as there is a way around it. 

This read me will be updated again very shortly with the new features for alpha5 - meanwhile here are previous version instructions that are still valid.  alpha5 feature additions are discussed in the Hubitat community alpha topic of which you are a member.


# Features

a) Enabling inbuilt HE devices*  to publish and be controllable through MQTT either using a compliant homie3 topic or alternatively a simplified minimal topic structure.   The simplified topic is the replacement for the previous 'Hubitat' topic structure. It removes most static topics from the homie3 tree and publishes mainly state changes, many of which can be updated by publishing to the state topic but with '/set' appended to the topic.  In a minimal form the topic is no longer compliant with the homie3 or homie4 specification.

b) Enables automatic discovery and selected inclusion and control of HE devices* and sensors into other controllers supporting the homie3 protocol (promoted by openHAB).

c) Enables automatic discovery and selected inclusion and control of HE devices* and sensors into other controllers using the Home Assistant discovery protocol (promoted by HA) 

d) Enable automatic discovery and selected inclusion and control of external devices* and sensors into HE that are using the homie3 protocol.

e) Enables automatic discovery and selected inclusion of external devices* and sensors into HE that are using the Home Assitant statestream protocol.  HA statestream does not support 'control' so a small automation script is provided for HA that adds this control capability for control back to HA too.  This supports switches and lights and can be extended to other devices by adding more automation scripts to HA.

f) Enables existing MQTT devices* to be 'mirrored' as any of the HE virtual devices and controlled within HE. All the topics are configurable as are the state (attribute) values..  Later referred to as 'manual' or 'adhoc' devices.

Athoms' Homey controller also supports MQTT and the homie3 protocol. They work really well together automatically discovering each others devices and enabling realtime synchronisation and control between the controllers. Two HE's could be setup this way too but HubConnect is the recommended option.  HubConnect virtual devices of course work with this MQTT app too.

#I have not yet tested the just released openHAB 2.5 with homie3 discovery but it may still need work. If you do use OH 2.5 (earlier versions will not work) then let me know how it goes.

*N.B.This version currently supports 'switch' (onoff), 'switchLevel' (dim) colour, most sensors and a lot of other device types too - and will be expanded to include others as needed (requested).  

# Future Features:

1) Support of many more device capabilities.

2) Support Home Assistant MQTT discovery protocol bidirectionally - HE devices are already auto discovered by HA but support devices  advertising using this protocol for discovery by HE. (Update: May not implement this latter support for HA Discovery > HE as there are virtually no such devices and HA doesn't support exporting it's devcies this way either)

3) Support for multiple homie discovered devices.

4) Support for JSON payloads

5) Support multiple MQTT brokers (considering)

6) Improved auto matching of device capabilities between different controllers

# Instructions: 
(preliminary)

# Initial setup:

1) Install both the main app MQTT and the device driver for MQTT Client and optionally MQTT text.
2) From 'Devices' "Add Virtual Device" selecting the "MQTT client" device driver, name the broker device as you wish, make up a Device Network ID e.g. the IP of the broker.  Do NOT create two MQTT client drivers as this causes known issues.
3) Configure this device to access your existing MQTT broker using the 'Preferences' section on the next screen in the device driver then click 'Done'.
You must enter the IP (or URL) e.g. tcp://192.168.1.78:1883 ,  username / password are only required if your broker needs them.  NB include  tcp:// at the start of the entry.
4) Launch the MQTT app, it has three setup pages each with multiple sections , the first page covers individual devices and the second and third pages are optional. The second page is for importing existing MQTT devices into HE and the third for automatic discovery of devices into HE using either homie or HA statestream 'discovery'. Configuring the second and third pages is optional.

On the first page click 'Configuration' and from the 'MQTT Broker' dropdown select the MQTT client device you just configured in step 3

Give you hub a name in 'Hub Name'

Next an important step that is required for speed:
The MQTT app can only interact with HE devices that you give it permission to use.  To make it possible for the app to know and match the names of all your devices on the first page click 'Configuration' and then at the bottom click this button "IMPORTANT: Enable all your devices for app access<" and enable all devices by clicking the top option. Please as you use the MQTT continue to check that all devices remain enabled - especially as you add new devices.  This speeds up the app considerably rather than the fallback of having to loop through every device individually.

For the time being ignore all other options and select 'Next' on the first and then second page and 'Done' on the third.
.. you should see something similar to this in the log.

Log:

	info MQTT: ================== Startup complete ==================
	info MQTT:     0 Hubitat devices enabled on MQTT
	info MQTT: ==================================================
	info MQTT: Skipping HA stateStream MQTT discovery
	info MQTT: Skipping homie MQTT discovery
	info MQTT> Connected as Hubitat_Development to MQTT broker tcp://192.168.1.78:1883
	info MQTT client alpha 5 initialised
	info MQTT> Resetting MQTT connection
	info MQTT> Log Level set to 2
	info MQTT: Hubitat hub name is : Hubitat/Development
	info MQTT alpha 5 Initialized
	info MQTT: MQTT Installed
	
Inparticular check that you get the "Connected as Hubitat_xxxx to MQTT broker" success result and no other errors, otherwise nothing will work !

Now choose from the key features listed above which feature(s) a) b) c) d) e) or f) you are wishing to setup the MQTT app for.
I recommend choosing just one for now preferably a) as it is the easiest.

Now follow the matching option a)-f) below that you need.....

Lastly several of the less obvious app configuration options are discussed right at the end of these instructions in 'Additional Notes'.  Please read those too.
__________________________________________________________________________________________________________________________


# a) Enabling inbuilt existing HE devices* to publish and be controllable through MQTT

  This is used to expose selected existing HE devices onto MQTT so they report their status and can be controlled via MQTT

 From 'Apps' run the MQTT app again and select how you would like to use to publish your devices to MQTT.
 You have two choices - the standard "homie3 protocol" compatible format or a reduced (simplified) version of this.
 The previous alpha4 'Hubitat Basic' topic structure is now deprecated and replaced by the simplified option above.
 
homie3:  and simplified topic

	homie/hubitat_hubname/deviceName/capability/<value>
	
	homie/hubitat_hubname/deviceName/capability/set/<value>
	
	NB. - To change the state of a HE device (if permitted) use the state topic with '/set' appended at the end of the topic, as shown above.
	
homie3: full version

	includes additional configuration topics required for homie3 compliance

When enabling the homie protocol there is an additional '..retain homie states' option. Enabling this causes the last reported state of your devices to be retained on your MQTT broker. Any new device device connecting to MQTT will receive these states , although these may be old (residual) values.  If you disable it any new device connecting to MQTT will get no information until your device updates its state, but the value will be known to be current.  This should normally be left enabled.

Now enable which of your HE devices that you wish to publish to MQTT by selecting them in the provided dropdowns. A dropdown is provided for each capability**. If you have no devices of that capability on HE then the dropdown will not be shown

HE Switch Devices > MQTT

HE Dimmer Devices > MQTT

click 'Update then 'Next' and 'Next' and then 'Done'

These devices will now update status changes to MQTT and be controllable from MQTT, using /set appended topic.

** A quick explanation on 'capabilities': When you select a device in a capability dropdown that device will publish just the drop downs capabilities to MQTT - not all the capabilities it might have.  So only some of the devices 'capabilities' might be reported to MQTT. Therefore make sure all the capabilities that you wish are enabled for that device, by selecting it in several drop downs, or to select every capability a device has use the 'Everything (all capabilities/attributes) drop down at the top and select your device.  For example some motion sensors provide motion, temperature, battery and light sensors. That's 4 capabilities, each individually selectable.
__________________________________________________________________________________________________________________________
# b) Enable automatic discovery of HE devices by other controllers (e.g OpenHAB) using homie3 protocol

In Configuration >> MQTT Publish Formats.
    Enable "homie 3 protocol"
    Enable "Complete and Compliant homietopics" is enabled
    Enable "..retain homie states"

Enable the HE devices you wish to be discoverable using homie by following a) above.

Configure your other controllers 'discovery option to point at at HE's device homie topic.

homie/hubname/#

__________________________________________________________________________________________________________________________
# c) Enable automatic discovery of HE devices by other controllers (e.g HA) using Home Assistant Discovery protocol

First enable HA MQTT discovery on your other controller.
If using Home Assistant ensure it has 'discovery' enabled and and note the topic that it is using (within the mqtt section of HA's configuration.yaml file).  You will need to restart HA if you edit configuration.yaml

	discovery: true
	discovery_prefix: homeassistant
	
Do NOT use the same topic name (base_topic) that HA is publishing statestream to - see e) below.
  
In Configuration >> MQTT Publish Formats.
    Enable "homie 3 protocol" ***
    Enable "Home Assistant MQTT discovery protocol" (requires homie3 publish enabled)
    
*** You must enable this as HA reads the MQTT payloads from these homie topics
In the HA Discovery Topic field enter the discovery_prefix value that HA is using (e.g. homeassistant if as above).

Click through 'Next' 'Next' 'Done' 

Devices that are enabled for MQTT within HE will automatically appear as entities in your Home Assistant front end. You may to have press ctrl F5 (or sometimes restart HA) to refresh the HA interface.  Alternatively devices may initially appear as 'unused entities' which you can check by clicking the 'hamburger' icon in the top right hand of recent HA builds.  These devices are bidirectionally realtime synched with HE.  

When HA starts up it checks MQTT for discovered devices. If you want them to persist over HA restarts then ensure you set "Home Assistant MQTT discovered devices" to 'Remember'" in the MQTT app. If it is unchecked then HA will purge these devices when restarted , although it might warn you with yellow boxes the first time it notices they have gone.

When the MQTT app has fully started it sets the $status = ready topic.  HA monitors this and at this stage HA MQTT devices will change from 'unavailable' to being functional. Should the MQTT app not complete startup ($status = init) or the MQTT broker topics not be available to HA then related devices will show 'unavailable'
__________________________________________________________________________________________________________________________

# d) Enable automatic discovery and selected inclusion of devices* using the homie3 protocol (promoted by openHAB)

    On the third page of the app entitled  
    "MQTT Discovery Protocols > HE"
        'homie' and enable the homie 3 protocol
    	Enter the top level name of the homie 'device' you wish to use in 'Homie Device Topic Name'
    	e.g. for homie/myDevice enter 'myDevice'
    	All nodes within that device will be scanned.
    click 'Done'
    
    e.g. if a homie topic looks like homie/mainsys/hallway/onoff then you would enter mainsys as 'device' as that is the device identifier (mainsys probably has many other nodes like hallway)
    
    Discovery may take a minute or so depending on how many nodes there are - the log will show you when it is complete.
    
    Next time you enter page three the 'homie' button will be green and show how many devices were discovered and dropdowns will be present that allows you to choose which of these 'homie' devices should be imported into HE.  Once enabled and 'Done' the app will create child devices for those that are enabled, attempting to choose an appropriate virtual device type.  You can manually alter the device type later in HE's Devices menu if needed.
    
    NB You can only imoprt one other homie device and that device must be within your existing homie topic tree.
    
# e)  Enable automatic discovery and selected inclusion of devices* using the Home Assistant statestream protocol

Firstly configure HA to use MQTT statestream by including the following in your HA configuration.yaml (choose your own base_topic: name)

	mqtt_statestream:

  	  base_topic: HAStateStream50             # << your choice of topic name
  
  	  publish_attributes: true
  
Add the included "automations.yaml" to your automations.yaml file in HA , or if you don't use this then to configuration.yaml.
This enables bidirectional control via statestream (which is normally a status only topic).

    On the third page of the app entitled  
    "MQTT Discovery Protocols > HE"
        "Home Assitant statestream" enable this
     	"Home Assitant Statestream Topic"  enter the topic name that your HA software is publishing to e.g HAStateStream50
     Do NOT uss the same topic name (discovery_topic) that HA is itself discovering devices from (normally 'homeassistant') - see C) above

Click 'Done'

The app will then start up which may take a minute or so and automatically discover all the devices you have asked it too.
The log will inform you when Home Assitant Discovery has completed.
After completion the app will then populate the discovered device dropdown lists so you can now select which devices you are interested in.  This you can see by revisiting the third page of the app where 'Home Assistant' will now be green with a count of how many devices were found. Clicking 'Home Assistant' reveals all the drop down selectors.
Once enabled and 'Done' the app will create child devices for those that are enabled, attempting to choose an appropriate virtual device type.  You can manually alter the device type later in HE's Devices menu if needed.

__________________________________________________________________________________________________________________________
# f) Enable MQTT devices* to be 'mirrored' as virtual devices and controlled within HE. 

Also known as 'manual or 'adhoc' devices this is the most involved option and requires you to create virtual MQTT child devices , using any of HE's 24 inbuilt virtual drivers. Then you configure that device to map to the MQTT topics that represent it's state. This is all done on the second page of the app entitled "Virtual MQTT Data"

HE can't (despite request !) create a dropdown for all 'Virtual Devices' so on this page you mustct a device typeat teh top ou mange devices within each category by a second dropdown.
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

# Forget Enabled Devices

Enabling this will purge your drop down lists from devices that have been discovered and enabled in these lists at restart. They will be re-discovered at startup but not enabled.
	
# Complete and Compliant Homie Topics	

Please leave this set ON for the time being.  If you experience a lot of 'error' messages at startup then please toggle this off and then 'on' again and click 'next' through to restart.  This option will be used once the 'Hubitat/' topic is deprecated to publish a minimal and non complient homie/ tree in it's palce - for simplicity.

# Retain homie states

Makes the homie state topics retained on the MQTT broker (normal).  NB homie /set topics should NOT be published retained.
