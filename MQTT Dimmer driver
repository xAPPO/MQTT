metadata 
{
	definition(name: "MQTT Dimmer", namespace: "ukusa", author: "Kevin Hawkins")
	{
		capability "Switch"
		capability "Switch Level"
		//capability "Refresh"
		capability "Telnet"  // temporary kludge to filter devices need capability "MQTT"

		command "setStateTopic", ["String"]
		command "setStateCmdTopic", ["String"]
		command "setLevelTopic", ["String"]
		command "setLevelCmdTopic", ["String"]
		command "setType",["String"]
		command "setLevel", ["String","String"]  // Duplicate of inbuilt one with duration ?
		command "setMaxBrightness", ["String","String"]
		command "setDeviceType", ["String","String"]
		command "toLevel"
		command "getStateTopics", ["String"]
		
		command "on"
		command "off"
		command "toON"
		command "toOFF"
		command "setStateType", ["String", "String"]
	}
}

   // TODO consider making MQTT devices driver children again


preferences{
		//page(name: "mainPage", title: "... not required for automatically discovered devices ") //errors
	section ("... not required for automatically discovered devices ")  // TODO does not display
	{
		input name: "TopicState", type: "text", title: "State topic", description: "", required: false, displayDuringSetup: true
		input name: "TopicCmd", type: "text", title: "Command topic", description: "", required: false, displayDuringSetup: true
		input name: "TopicLevel", type: "text", title: "Level topic", description: "", required: false, displayDuringSetup: true
		input name: "TopicCmdLevel", type: "text", title: "Level command topic", description: "", required: false, displayDuringSetup: true		
		input name: "StateON", type: "text", title: "State on is..", description: "e.g. &nbsp ON &nbsp On &nbsp true &nbsp 1 &nbsp yes", required: false, displayDuringSetup: true
		input name: "StateOFF", type: "text", title: "State off is..", description: "e.g. &nbsp OFF &nbsp Off &nbsp false &nbsp 0 &nbsp no", required: false, displayDuringSetup: true
		input name: "maxBrightness", type: "number", title: "Maximum brightness level", description: " e.g. &nbsp 1.0 &nbsp 5.0  &nbsp 100 &nbsp 255", required: false, displayDuringSetup: true
	}
}

def installed()
{
	initialize()
}

def updated()
{
	if (state.type==null||state.type=="not configured"||state.type=="manual") {
		if (settings?.TopicState != null) {
			state.type = "manual"
			device.deviceNetworkId = 'MQTT:Internal '+ settings?.TopicState
		}
		else if (settings?.TopicLevel != null) {   // there is no state topic for DNI
			state.type = "manual"
			device.deviceNetworkId = 'MQTT:Internal '+ settings?.TopicLevel
		}
		else {
			log.error device.name + " has no state or level topic configured and one at least is required for MQTT to synchronise"
			state.type="not configured"
		}
	}
	
	if (state.type=="manual"){
		if (settings?.TopicState != null) state.stateTopic =settings?.TopicState
			else log.warn device.name + " was setup but has no OnOff topic configured"
		if (settings?.TopicCmd != null) state.commandTopic=settings?.TopicCmd
		if (settings?.StateON != null) state.stateTypeON = settings?.StateON
		if (settings?.StateOFF != null) state.stateTypeOFF = settings?.StateOFF
		if (settings?.TopicLevel != null) state.LevelTopic = settings?.TopicLevel
			else log.warn device.name + " was setup but has no level topic configured"
		if (settings?.TopicCmdLevel != null) state.LevelCmdTopic = settings?.TopicCmdLevel
		if (settings?.maxBrightness != null) state.maxBrightness = settings?.maxBrightness
	}
	getStateTopics()  //update map and MQTT topic subscriptions
	initialize()
}

def initialize()
{
	refresh()
}

def parse(String description) // shouldn't get called
{
	log.warn "Msg: Description is $description"
}

def on() {
    //log.debug "child On " + state.commandTopic
	if (state.type=="manual") sendEvent(name: "changeState", value: state.stateTypeON,  data: [topic: state.commandTopic])
	else parent.sendPayload(state.commandTopic, state.stateTypeON)
}

def off() {
    //log.debug "child Off "+ state.commandTopic
	if (state.type=="manual") sendEvent(name: "changeState", value: state.stateTypeOFF,  data: [topic: state.commandTopic])
	else parent.sendPayload(state.commandTopic, state.stateTypeOFF)
}

def toON() {  // avoids publish loop
	sendEvent(name: "switch", value: "on", isStateChange: true)
}

def toOFF() {  // avoids publish loop
	sendEvent(name: "switch", value: "off", isStateChange: true)
}

def setStateType (stateOFF,stateON)
    {
	//log.trace "Setting OFF State Type to " + stateOFF
    state.stateTypeOFF = stateOFF
    state.stateTypeON = stateON
}

def setLevel(value, duration=1)
{
	sendEvent(name: "level", value: value)
	float newLevel
	if (state.maxBrightness == '1'|| state.maxBrightness == "1.0")  //TODO need to be careful on accuracies and a loop or creep on value updates - solve by breaking the loop
	{
		newLevel= value.toInteger()/100  // TODO messy  doesnt work transparentlywith different devices
		if (state.type=="manual") sendEvent(name: "changeLevel", value: newLevel.toString(),data: [topic: state.commandTopic, dimTopic: state.LevelCmdTopic])
		else parent.sendPayload(state.LevelCmdTopic,newLevel.toString())
	}
	else 
	{
		newLevel= (value.toInteger() * state.maxBrightness.toInteger()) / 100 
		if (state.type=="manual") sendEvent(name: "changeLevel", value: newLevel.toString(),data: [topic: state.commandTopic, dimTopic: state.LevelCmdTopic])
		else parent.sendPayload(state.LevelCmdTopic,newLevel.toInteger().toString())
	}
}

def toLevel(value,duration=1)
{
	sendEvent(name: "level", value: value, isStateChange: true)
}

def refresh()
{
	// Maybe refresh the device by resubscribing to the MQTT broker ?
	// parent.sendDeviceEvent(device.deviceNetworkId, "refresh")
}

def sendStateTopics()
{
		//log.info "MQTT subscribe topics for manual adhoc devices "
	    if (state.LevelTopic!=null)  sendEvent([name: "topic", value: state.LevelTopic])  // if have it
	    if (state.stateTopic!=null)  sendEvent([name: "topic", value: state.stateTopic])  // if have it
}

def getStateTopics()
{
	// needed to create lookup to DNI of device to match MQTT state changes for adhoc/manual devices
	if (state.LevelTopic!=null) 
	{
		if (state.stateTopic==null) 
		{	
			stateTopic = state.LevelTopic  // substitute state into level
			sendEvent (name: "mapTopic" , value: device, data: [level: state.LevelTopic, state: stateTopic, valueMax: state.maxBrightness, stateON: state.stateTypeON, stateOFF: state.stateTypeOFF])
		}
	else  //have both state and level topics
		{  
			sendEvent(name: "mapTopic", value: device, data: [level: state.LevelTopic, state: state.stateTopic, valueMax: state.maxBrightness, stateON: state.stateTypeON, stateOFF: state.stateTypeOFF])
		}
	}
	else if (state.stateTopic!=null) 
	{
		if (state.LevelTopic==null) 
		{
			levelTopic = state.stateTopic  // substitute state into level
			sendEvent(name: "mapTopic", value: device, data: [level: levelTopic, state: state.stateTopic, valueMax: state.maxBrightness, stateON: state.stateTypeON, stateOFF: state.stateTypeOFF])
		}
	}
	runIn(10, "sendStateTopics")  
	// this helps avoids incoming MQTT messages before the mappings have completed
	// TODO maybe find a better way of knowing when can safely run - each adhoc device adds 500ms so this only copes with about 20 devices
	// could use a calculated value

}

def setStateTopic(String topic)
{
	state.stateTopic=topic
}

def setType (String system)
{
	state.type=system
}

def setStateCmdTopic(String topic)
{
	state.commandTopic=topic
}
def setLevelTopic(String topic)
{
	state.LevelTopic=topic   //TODO change to state.levelTopic
}

def setLevelCmdTopic(String topic)
{
	state.LevelCmdTopic=topic  //TODO change to state.levelCmdTopic
}

def setMaxBrightness(String maxBrightness)
{
	//log.debug "Max brightness set to " + maxBrightness
	state.maxBrightness=maxBrightness
}

def setDeviceType(String devType)  // TODO maybe use this for homie HA etc
{
	//log.debug "set deviceType to " + devType
	state.deviceType=devType
}
