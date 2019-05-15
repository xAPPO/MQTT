metadata 
{
	definition(name: "MQTT Switch", namespace: "ukusa", author: "Kevin Hawkins")
	{
		capability "Switch"
		//capability "Refresh"
		capability "Telnet"  // temporary kludge
		
		command "setStateTopic", ["String"]
		command "setStateCmdTopic", ["String"]
		command "setType",["String"]
		command "on"
		command "off"
		command "toON"
		command "toOFF"
		command "setStateType", ["String", "String"]
		command "getStateTopics", ["String"]
	}
}
preferences{
	section ("... not required for automatically discovered devices ")  // TODO does not display
	{
		input name: "TopicState", type: "text", title: "State topic", description: "", required: false, displayDuringSetup: true
		input name: "TopicCmd", type: "text", title: "Command topic", description: "", required: false, displayDuringSetup: true
		input name: "StateON", type: "text", title: "State on is..", description: "e.g. &nbsp ON &nbsp On &nbsp true &nbsp 1 &nbsp yes", required: false, displayDuringSetup: true
		input name: "StateOFF", type: "text", title: "State off is..", description: "e.g. &nbsp OFF &nbsp Off &nbsp false &nbsp 0 &nbsp no", required: false, displayDuringSetup: true
	}
}

   // TODO consider making MQTT devices driver children again

def installed()
{
	initialize()
}

def updated()
{
	if (state.type==null||state.type=="manual") 
	if (settings?.TopicState != null) {
		state.type = "manual"
		state.stateTopic =settings?.TopicState
		device.deviceNetworkId = 'MQTT:Internal '+ settings?.TopicState
	if (settings?.TopicCmd != null)   state.commandTopic=settings?.TopicCmd
	if (settings?.StateON != null) state.stateTypeON = settings?.StateON
	if (settings?.StateOFF != null) state.stateTypeOFF = settings?.StateOFF
	if (settings?.TopicLevel != null) state.LevelTopic = settings?.StateON
	if (settings?.TopicCmdLevelCmd != null) state.LevelCmdTopic = settings?.StateOFF
	}
	initialize()
}

def initialize()
{
	refresh()
}


def parse(String description)  // shouldn't get called
{
	log.warn "Msg: Description is $description"
}

def on() {
	sendEvent(name: "switch", value: "on", isStateChange: true)
	//parent.sendPayload(state.commandTopic,"true")
	if (state.type=="manual") sendEvent(name: "changeState", value: state.stateTypeON,  data: [topic: state.commandTopic])
	else parent.sendPayload(state.commandTopic, state.stateTypeON) 
}

def off() {
	sendEvent(name: "switch", value: "off", isStateChange: true)
	//parent.sendPayload(state.commandTopic,"false")
	if (state.type=="manual") sendEvent(name: "changeState", value: state.stateTypeOFF,  data: [topic: state.commandTopic])
	else parent.sendPayload(state.commandTopic,state.stateTypeOFF) 

}

def toON() {  // avoids publish loop
	sendEvent(name: "switch", value: "on", isStateChange: true)
}

def toOFF() {  // avoids publish loop
	sendEvent(name: "switch", value: "off", isStateChange: true)
}

def setStateType (stateOFF,stateON)
    {
    state.stateTypeOFF = stateOFF
    state.stateTypeON = stateON
}

def refresh()
{
	// Maybe refresh the device by resubscribing to the MQTT broker ?
	//parent.sendDeviceEvent(device.deviceNetworkId, "refresh")
}

def getStateTopics()
{
	// needed to create DNI of device to match MQTT state changes
	sendEvent([name: "topic", value: state.stateTopic])
}

def setStateTopic(String topic)
{
	state.stateTopic=topic
}

def setType(String system)
{
	state.type = system
}

def setStateCmdTopic(String topic)
{
	state.commandTopic=topic
}


