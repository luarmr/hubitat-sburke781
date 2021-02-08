/**
 *  Mitsubishi Electric KumoCloud Parent Driver
 *
 *  Copyright 2021 Simon Burke
 *
 *  Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
 *  in compliance with the License. You may obtain a copy of the License at:
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software distributed under the License is distributed
 *  on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License
 *  for the specific language governing permissions and limitations under the License.
 *
 *  Change History:
 *
 *    Date        Who            What
 *    ----        ---            ----
 *    2021-02-07  Simon Burke    Created Parent Driver
 * 
 */
metadata {
	definition (name: "KumoCloud Parent Driver", namespace: "simnet", author: "Simon Burke") {
        
        capability "Refresh"
        
        attribute "authCode", "string"
	}

	preferences {
		input(name: "BaseURL", type: "string", title:"KumoCloud Base URL", description: "Enter the base URL for the Mitsubishi Electric KumoCloud Service", defaultValue: "https://geo-c.kumocloud.com", required: true, displayDuringSetup: true)
		input(name: "UserName", type: "string", title:"KumoCloud Username / Email", description: "Username / Email used to authenticate on Mitsubishi Electric KumoCloud", displayDuringSetup: true)
		input(name: "Password", type: "password", title:"KumoCloud Account Password", description: "Password for authenticating on Mitsubishi Electric KumoCloud", displayDuringSetup: true)
        input(name: "DebugLogging", type: "bool", title:"Enable Debug Logging", displayDuringSetup: true, defaultValue: false)
        input(name: "WarnLogging", type: "bool", title:"Enable Warning Logging", displayDuringSetup: true, defaultValue: true)
        input(name: "ErrorLogging", type: "bool", title:"Enable Error Logging", displayDuringSetup: true, defaultValue: true)
        input(name: "InfoLogging", type: "bool", title:"Enable Description Text (Info) Logging", displayDuringSetup: true, defaultValue: false)
    }

}


def refresh() {
  debugLog("refresh: Refresh process called")
  // Authenticate with KumoCloud Service and
  //   record Authentication Code for use in future communications  
  setAuthCode()
  
  //createChildACUnits()
  
}


def initialize() {
    if ("${UserName}" != "" && "${Password}" != "" && "${BaseURL}" != "") { refresh() }
}

//Authentication

def setAuthCode() {
      
    def bodyJson = "{ \"username\": \"${UserName}\", \"password\": \"${Password}\", \"AppVersion\": \"2.2.0\" }"
    def headers = [:] 

    headers.put("Accept-Encoding", "gzip, deflate, br")
    headers.put("Connection", "keep-alive")
    headers.put("Accept", "application/json, text/plain, */*") 
    headers.put("DNT", "1")
    headers.put("User-Agent", "")
    headers.put("Content-Type", "application/json;charset=UTF-8")
    headers.put("Origin", "https://app.kumocloud.com")
    headers.put("Sec-Fetch-Site", "same-site")
    headers.put("Sec-Fetch-Mode", "cors")
    headers.put("Sec-Fetch-Dest", "empty")
    headers.put("Referer", "https://app.kumocloud.com")
    headers.put("Accept-Language", "en-US,en;q=0.9")
    
    def postParams = [
        uri: "${BaseURL}/login",
        headers: headers,
        contentType: "application/json; charset=UTF-8",
        body : bodyJson
	]
           
	try {
        
        httpPost(postParams)
        { resp -> 
            debugLog("setAuthCode: ${resp.data}")
            def newAuthCode = "";
            
            debugLog("setAuthCode: ContextKey - ${resp?.data?.login?.value}");
            newAuthCode = "${resp?.data?.login?.value}";
            
            if (newAuthCode != "") {
                //sendEvent(name: "authCode", value : newAuthCode)
                state.authCode = newAuthCode
                debugLog("setAuthCode: New authentication code value has been set")
            }
            else {debugLog("setAuthCode: New authentication code was NOT set")}
        }
            
	}
	catch (Exception e) {
        errorLog("setAuthCode: Unable to query Mitsubishi Electric MELCloud: ${e}")
	}

}

def getAuthCode() { return state.authCode }

def deriveChildDNI(childDeviceId, childDeviceType) {

    return "${device.deviceNetworkId}-id${childDeviceId}-type${childDeviceType}"
}

def findChildDevice(childDeviceId, childDeviceType) {
	getChildDevices()?.find { it.deviceNetworkId == deriveChildDNI(childDeviceId, childDeviceType)}
}

def getBaseURL() { return BaseURL }

def createChildDevice(childDeviceId, childDeviceName, childDeviceType) {
    debugLog("createChildDevice: Creating Child Device: ${childDeviceId}, ${childDeviceName}, ${childDeviceType}")
    
	def childDevice = findChildDevice(childDeviceId, childDeviceType)
    
    if (childDevice == null) {
        childDevice = addChildDevice("simnet", "MELCloud AC Unit", deriveChildDNI(childDeviceId, childDeviceType), [label: "${device.displayName} - ${childDeviceName}"])
        infoLog("createChildDevice: New MEL Air Conditioning Child Device created -  ${device.displayName} - ${childDeviceName}")
	}
    else {
      debugLog("createChildDevice: child device ${childDevice.deviceNetworkId} already exists")
	}
}

//Utility methods
def debugLog(debugMessage) {
	if (DebugLogging == true) {log.debug(debugMessage)}	
}

def errorLog(errorMessage) {
    if (ErrorLogging == true) { log.error(errorMessage)}  
}

def infoLog(infoMessage) {
    if(InfoLogging == true) {log.info(infoMessage)}    
}

def warnLog(warnMessage) {
    if(WarnLogging == true) {log.warn(warnMessage)}    
}
