# CEF to Microsoft Sentinel CommonSecurityLog
----
This pack was written to convert CEF based logs into the Microsoft CommonSecurityLog format for ingestion into Microsoft Sentinel. 

The pack follows the mapping spec from Microsoft outlined here: https://learn.microsoft.com/en-us/azure/sentinel/cef-name-mapping to rename any CEF fields into their respecitive CommonSecurityLog format. 


## Requirements Section

Before you begin, ensure that you have met the following requirements:

* Setup a Microsoft destination CommonSecurityLog webhook outlined in guide here: https://docs.cribl.io/stream/usecase-azure-webhook/
* Format your logs into CEF format. Many vendors such as Palo Alto, Trend Micro, etc have logging settings within their products that allow you to write and output logs into CEF.
* Use the Cribl syslog pre-processor pack on ingest.

## How it works
* We split the CSV Syslog _raw field into it's own fields. 
* We then use a rename function to look up the CEF field name and find it's CommonSecurityLog field name from within the cef-to-common-security-log.csv mapping file in the knowledge pack.
* Numerify any values that are numbers. 
* (Optional) If the DeviceVendor input is Palo Alto Networks, through any fields that start with PAN into a field called: "AdditionalEvents".  

## Sample outputs

Example Trend Micro CEF input: 

```
{
  "_raw": "<14>Sep 18 14:58:29 logstream syslog: CEF:0|Trend Micro|Deep Security Agent|10.2.229|6001200|AppControl detectOnly|6|cn1=202 cn1Label=Host ID dvc=192.168.33.128 TrendMicroDsTenant=Primary TrendMicroDsTenantId=0 fileHash=80D4AC182F97D2AB48EE4310AC51DA5974167C596D133D64A83107B9069745E0 suser=root suid=0 act=detectOnly filePath=/home/user1/Desktop/Directory1//heartbeatSync.sh fsize=20 aggregationType=0 repeatCount=1 cs1=notWhitelisted cs1Label=actionReason cs2=0CC9713BA896193A527213D9C94892D41797EB7C cs2Label=sha1 cs3=7EA8EF10BEB2E9876D4D7F7E5A46CF8D cs3Label=md5",
  "_time": 1631995140,
  "cribl_breaker": [
    "Break on newlines",
    "ndjson"
  ],
  "message": "CEF:0|Trend Micro|Deep Security Agent|10.2.229|6001200|AppControl detectOnly|6|cn1=202 cn1Label=Host ID dvc=192.168.33.128 TrendMicroDsTenant=Primary TrendMicroDsTenantId=0 fileHash=80D4AC182F97D2AB48EE4310AC51DA5974167C596D133D64A83107B9069745E0 suser=root suid=0 act=detectOnly filePath=/home/user1/Desktop/Directory1//heartbeatSync.sh fsize=20 aggregationType=0 repeatCount=1 cs1=notWhitelisted cs1Label=actionReason cs2=0CC9713BA896193A527213D9C94892D41797EB7C cs2Label=sha1 cs3=7EA8EF10BEB2E9876D4D7F7E5A46CF8D cs3Label=md5",
  "severity": 6,
  "facility": 1,
  "host": "logstream",
  "appname": "syslog",
  "severityName": "info",
  "facilityName": "user"
}
```

Example Trend Micro CEF convereted into CommonSecurityLog output:
```
{
  "TrendMicroDsTenant": "Primary",
  "TrendMicroDsTenantId": 0,
  "aggregationType": 0,
  "repeatCount": 1,
  "LogSeverity": 6,
  "Computer": "logstream",
  "DeviceFacility": "user",
  "DeviceVendor": "Trend Micro",
  "DeviceProduct": "Deep Security Agent",
  "DeviceVersion": "10.2.229",
  "DeviceEventClassID": 6001200,
  "Activity": "AppControl detectOnly",
  "DeviceCustomNumber1": 202,
  "DeviceCustomNumber1Label": "Host ID",
  "DeviceAddress": "192.168.33.128",
  "FileHash": "80D4AC182F97D2AB48EE4310AC51DA5974167C596D133D64A83107B9069745E0",
  "SourceUserName": "root",
  "SourceUserID": 0,
  "DeviceAction": "detectOnly",
  "FilePath": "home/user1/Desktop/Directory1//heartbeatSync.sh",
  "FileSize": 20,
  "DeviceCustomString1": "notWhitelisted",
  "DeviceCustomString1Label": "actionReason",
  "DeviceCustomString2": "0CC9713BA896193A527213D9C94892D41797EB7C",
  "DeviceCustomString2Label": "sha1",
  "DeviceCustomString3": "7EA8EF10BEB2E9876D4D7F7E5A46CF8D",
  "DeviceCustomString3Label": "md5",
  "TimeGenerated": "2022-09-18T14:58:29.000Z",
  "Type": "CommonSecurityLog",
}
```

Sample Palo Alto Networks CEF input: 

```
{
	"_raw": "Mar 21 15:00:21 xxx.xx.x.xx 4581 <14>1 2023-03-16T20:46:50.869Z stream-logfwd20-587718190-03011242-xynu-harness-zpqg logforwarder - panwlogs - CEF:0|Palo Alto Networks|LF|2.0|TRAFFIC|end|3|ProfileToken=xxxxx dtz=UTC rt=Mar 21 2023 15:00:21 deviceExternalId=xxxxxxxxxxxxx PanOSApplicationContainer= PanOSApplicationRisk=5 PanOSApplicationSubcategory=file-sharing PanOSApplicationTechnology=peer-to-peer PanOSCaptivePortal=false PanOSCortexDataLakeTenantID=xxxxxxxxxxxxx PanOSDestinationDeviceClass= PanOSDestinationDeviceOS= dntdom=paloaltonetwork duser=xxxxx duid= PanOSInboundInterfaceDetailsPort=0 PanOSInboundInterfaceDetailsSlot=0 PanOSInboundInterfaceDetailsType=unknown PanOSInboundInterfaceDetailsUnit=0 PanOSIsClienttoServer=false PanOSIsContainer=false PanOSIsDecryptMirror=false PanOSIsDecrypted=false PanOSIsDecryptedLog=false PanOSIsDecryptedPayloadForward=false PanOSIsDuplicateLog=false PanOSIsEncrypted=false PanOSIsIPV6=false PanOSIsInspectionBeforeSession=true PanOSIsMptcpOn=false PanOSIsNonStandardDestinationPort=false PanOSIsPacketCapture=false PanOSIsPhishing=false PanOSIsPrismaNetwork=false PanOSIsPrismaUsers=false PanOSIsProxy=false PanOSIsReconExcluded=false PanOSIsSaaSApplication=false PanOSIsServertoClient=false PanOSIsSourceXForwarded=false PanOSIsSystemReturn=false PanOSIsTransaction=false PanOSIsTunnelInspected=false PanOSIsURLDenied=false PanOSLogExported=false PanOSLogForwarded=true PanOSLogSource=firewall PanOSLogSourceTimeZoneOffset= PanOSNAT=false PanOSNonStandardDestinationPort=0 PanOSOutboundInterfaceDetailsPort=0 PanOSOutboundInterfaceDetailsSlot=0 PanOSOutboundInterfaceDetailsType=unknown PanOSOutboundInterfaceDetailsUnit=0 PanOSSDWANFECRatio=0.0 PanOSSanctionedStateOfApp=false PanOSSessionOwnerMidx=false PanOSSessionTracker=16 PanOSSourceDeviceClass= PanOSSourceDeviceOS= sntdom=xxxxx suser=xxxxx xxxxx suid= PanOSTunneledApplication=tunneled-app PanOSUsers=xxxxx\\xxxxx xxxxx PanOSVirtualSystemID=1 PanOSApplicationCategory=peer2peer PanOSConfigVersion=10.0 start=Feb 27 2021 20:16:17 src=xxx.xx.x.xx dst=xxx.xx.x.xx sourceTranslatedAddress=xxx.xx.x.xx destinationTranslatedAddress=xxx.xx.x.xx cs1=deny-attackers cs1Label=Rule suser0=xxxxx\\xxxxx xxxxx duser0=paloaltonetwork\\xxxxx app=fileguri cs3=vsys1 cs3Label=VirtualLocation cs4=untrust cs4Label=FromZone cs5=ethernet4Zone-test1 cs5Label=ToZone deviceInboundInterface=unknown deviceOutboundInterface=unknown cs6=rs-logging cs6Label=LogSetting cn1=25596 cn1Label=SessionID cnt=1 spt=22871 dpt=27092 sourceTranslatedPort=24429 destinationTranslatedPort=14744 proto=tcp act=deny PanOSBytes=1370294 out=400448 in=969846 cn2=314 cn2Label=PacketsTotal PanOSSessionStartTime=Mar 16 2023 20:15:48 cn3=56 cn3Label=SessionDuration cs2=custom-category cs2Label=URLCategory externalId=xxxxxxxxxxxxx PanOSSourceLocation=east-coast PanOSDestinationLocation=BR PanOSPacketsSent=194 PanOSPacketsReceived=120 reason=unknown PanOSDGHierarchyLevel1=11 PanOSDGHierarchyLevel2=0 PanOSDGHierarchyLevel3=0 PanOSDGHierarchyLevel4=0 PanOSVirtualSystemName= dvchost=xxxxx cat=unknown PanOSSourceUUID= PanOSDestinationUUID= PanOSIMSI=0 PanOSIMEI= PanOSParentSessionID=0 PanOSParentStarttime=Mar 16 2023 20:15:40 PanOSTunnel=GRE PanOSEndpointAssociationID=-3746994889972252628 PanOSChunksTotal=1945 PanOSChunksSent=323 PanOSChunksReceived=1622 PanOSRuleUUID=017e4d76-2003-47f4-8afc-1d35c808c615 PanOSHTTP2Connection=469139 PanOSLinkChangeCount=0 PanOSSDWANPolicyName= PanOSLinkSwitches= PanOSSDWANCluster= PanOSSDWANDeviceType= PanOSSDWANClusterType= PanOSSDWANSite= PanOSDynamicUserGroupName=dynug-4 PanOSX-Forwarded-ForIP=xxx.xx.x.xx PanOSSourceDeviceCategory=N-Phone PanOSSourceDeviceProfile=n-profile PanOSSourceDeviceModel=Nexus PanOSSourceDeviceVendor=Google PanOSSourceDeviceOSFamily=LG-H790 PanOSSourceDeviceOSVersion=Android v6 PanOSSourceDeviceHost=pan-301 PanOSSourceDeviceMac=839147449905 PanOSDestinationDeviceCategory=N-Phone PanOSDestinationDeviceProfile=n-profile PanOSDestinationDeviceModel=Nexus PanOSDestinationDeviceVendor=Google PanOSDestinationDeviceOSFamily=H1511 PanOSDestinationDeviceOSVersion=Android v7 PanOSDestinationDeviceHost=pan-355 PanOSDestinationDeviceMac=530589561221 PanOSContainerID=1873cc5c-0d31 PanOSContainerNameSpace=pns_default PanOSContainerName=pan-dp-77754f4 PanOSSourceEDL= PanOSDestinationEDL= PanOSGPHostID=xxxxxxxxxxxxxx PanOSEndpointSerialNumber=xxxxxxxxxxxxxx PanOSSourceDynamicAddressGroup= aqua_dag PanOSDestinationDynamicAddressGroup= PanOSHASessionOwner=session_owner-4 PanOSTimeGeneratedHighResolution=Feb 27 2021 20:16:18 PanOSNSSAINetworkSliceType=0 PanOSNSSAINetworkSliceDifferentiator=1bca5",
	"_time": 1679410821,
	"cribl_breaker": [
		"Break on newlines",
		"noBreak1MB",
		"noBreak1MB"
	],
	"message": "1 2023-03-16T20:46:50.869Z stream-logfwd20-587718190-03011242-xynu-harness-zpqg logforwarder - panwlogs - CEF:0|Palo Alto Networks|LF|2.0|TRAFFIC|end|3|ProfileToken=xxxxx dtz=UTC rt=Mar 21 2023 15:00:21 deviceExternalId=xxxxxxxxxxxxx PanOSApplicationContainer= PanOSApplicationRisk=5 PanOSApplicationSubcategory=file-sharing PanOSApplicationTechnology=peer-to-peer PanOSCaptivePortal=false PanOSCortexDataLakeTenantID=xxxxxxxxxxxxx PanOSDestinationDeviceClass= PanOSDestinationDeviceOS= dntdom=paloaltonetwork duser=xxxxx duid= PanOSInboundInterfaceDetailsPort=0 PanOSInboundInterfaceDetailsSlot=0 PanOSInboundInterfaceDetailsType=unknown PanOSInboundInterfaceDetailsUnit=0 PanOSIsClienttoServer=false PanOSIsContainer=false PanOSIsDecryptMirror=false PanOSIsDecrypted=false PanOSIsDecryptedLog=false PanOSIsDecryptedPayloadForward=false PanOSIsDuplicateLog=false PanOSIsEncrypted=false PanOSIsIPV6=false PanOSIsInspectionBeforeSession=true PanOSIsMptcpOn=false PanOSIsNonStandardDestinationPort=false PanOSIsPacketCapture=false PanOSIsPhishing=false PanOSIsPrismaNetwork=false PanOSIsPrismaUsers=false PanOSIsProxy=false PanOSIsReconExcluded=false PanOSIsSaaSApplication=false PanOSIsServertoClient=false PanOSIsSourceXForwarded=false PanOSIsSystemReturn=false PanOSIsTransaction=false PanOSIsTunnelInspected=false PanOSIsURLDenied=false PanOSLogExported=false PanOSLogForwarded=true PanOSLogSource=firewall PanOSLogSourceTimeZoneOffset= PanOSNAT=false PanOSNonStandardDestinationPort=0 PanOSOutboundInterfaceDetailsPort=0 PanOSOutboundInterfaceDetailsSlot=0 PanOSOutboundInterfaceDetailsType=unknown PanOSOutboundInterfaceDetailsUnit=0 PanOSSDWANFECRatio=0.0 PanOSSanctionedStateOfApp=false PanOSSessionOwnerMidx=false PanOSSessionTracker=16 PanOSSourceDeviceClass= PanOSSourceDeviceOS= sntdom=xxxxx suser=xxxxx xxxxx suid= PanOSTunneledApplication=tunneled-app PanOSUsers=xxxxx\\xxxxx xxxxx PanOSVirtualSystemID=1 PanOSApplicationCategory=peer2peer PanOSConfigVersion=10.0 start=Feb 27 2021 20:16:17 src=xxx.xx.x.xx dst=xxx.xx.x.xx sourceTranslatedAddress=xxx.xx.x.xx destinationTranslatedAddress=xxx.xx.x.xx cs1=deny-attackers cs1Label=Rule suser0=xxxxx\\xxxxx xxxxx duser0=paloaltonetwork\\xxxxx app=fileguri cs3=vsys1 cs3Label=VirtualLocation cs4=untrust cs4Label=FromZone cs5=ethernet4Zone-test1 cs5Label=ToZone deviceInboundInterface=unknown deviceOutboundInterface=unknown cs6=rs-logging cs6Label=LogSetting cn1=25596 cn1Label=SessionID cnt=1 spt=22871 dpt=27092 sourceTranslatedPort=24429 destinationTranslatedPort=14744 proto=tcp act=deny PanOSBytes=1370294 out=400448 in=969846 cn2=314 cn2Label=PacketsTotal PanOSSessionStartTime=Mar 16 2023 20:15:48 cn3=56 cn3Label=SessionDuration cs2=custom-category cs2Label=URLCategory externalId=xxxxxxxxxxxxx PanOSSourceLocation=east-coast PanOSDestinationLocation=BR PanOSPacketsSent=194 PanOSPacketsReceived=120 reason=unknown PanOSDGHierarchyLevel1=11 PanOSDGHierarchyLevel2=0 PanOSDGHierarchyLevel3=0 PanOSDGHierarchyLevel4=0 PanOSVirtualSystemName= dvchost=xxxxx cat=unknown PanOSSourceUUID= PanOSDestinationUUID= PanOSIMSI=0 PanOSIMEI= PanOSParentSessionID=0 PanOSParentStarttime=Mar 16 2023 20:15:40 PanOSTunnel=GRE PanOSEndpointAssociationID=-3746994889972252628 PanOSChunksTotal=1945 PanOSChunksSent=323 PanOSChunksReceived=1622 PanOSRuleUUID=017e4d76-2003-47f4-8afc-1d35c808c615 PanOSHTTP2Connection=469139 PanOSLinkChangeCount=0 PanOSSDWANPolicyName= PanOSLinkSwitches= PanOSSDWANCluster= PanOSSDWANDeviceType= PanOSSDWANClusterType= PanOSSDWANSite= PanOSDynamicUserGroupName=dynug-4 PanOSX-Forwarded-ForIP=xxx.xx.x.xx PanOSSourceDeviceCategory=N-Phone PanOSSourceDeviceProfile=n-profile PanOSSourceDeviceModel=Nexus PanOSSourceDeviceVendor=Google PanOSSourceDeviceOSFamily=LG-H790 PanOSSourceDeviceOSVersion=Android v6 PanOSSourceDeviceHost=pan-301 PanOSSourceDeviceMac=839147449905 PanOSDestinationDeviceCategory=N-Phone PanOSDestinationDeviceProfile=n-profile PanOSDestinationDeviceModel=Nexus PanOSDestinationDeviceVendor=Google PanOSDestinationDeviceOSFamily=H1511 PanOSDestinationDeviceOSVersion=Android v7 PanOSDestinationDeviceHost=pan-355 PanOSDestinationDeviceMac=530589561221 PanOSContainerID=1873cc5c-0d31 PanOSContainerNameSpace=pns_default PanOSContainerName=pan-dp-77754f4 PanOSSourceEDL= PanOSDestinationEDL= PanOSGPHostID=xxxxxxxxxxxxxx PanOSEndpointSerialNumber=xxxxxxxxxxxxxx PanOSSourceDynamicAddressGroup= aqua_dag PanOSDestinationDynamicAddressGroup= PanOSHASessionOwner=session_owner-4 PanOSTimeGeneratedHighResolution=Feb 27 2021 20:16:18 PanOSNSSAINetworkSliceType=0 PanOSNSSAINetworkSliceDifferentiator=1bca5",
	"severity": 6,
	"facility": 1,
	"host": "xxxxx",
	"severityName": "info",
	"facilityName": "user"
}
```

Example Palo Alto Networks Traffic log converted into CommonSecurityLog output:
```
{
  "ProfileToken": "xxxxx",
  "start": "Feb 27 2021 20:16:17",
  "suser0": "xxxxx\\xxxxx xxxxx",
  "duser0": "paloaltonetwork\\xxxxx",
  "LogSeverity": 3,
  "Computer": "xxxxx",
  "DeviceFacility": "user",
  "DeviceVendor": "Palo Alto Networks",
  "DeviceProduct": "LF",
  "DeviceVersion": 2,
  "DeviceEventClassID": "TRAFFIC",
  "Activity": "end",
  "DeviceTimeZone": "UTC",
  "ReceiptTime": "Mar 21 2023 15:00:21",
  "DeviceExternalID": "xxxxxxxxxxxxx",
  "DestinationNTDomain": "paloaltonetwork",
  "DestinationUserName": "xxxxx",
  "DestinationUserId": 0,
  "SourceNTDomain": "xxxxx",
  "SourceUserName": "xxxxx xxxxx",
  "SourceUserID": 0,
  "SourceIP": "xxx.xx.x.xx",
  "DestinationIP": "xxx.xx.x.xx",
  "SourceTranslatedAddress": "xxx.xx.x.xx",
  "DestinationTranslatedAddress": "xxx.xx.x.xx",
  "DeviceCustomString1": "deny-attackers",
  "DeviceCustomString1Label": "Rule",
  "ApplicationProtocol": "fileguri",
  "DeviceCustomString3": "vsys1",
  "DeviceCustomString3Label": "VirtualLocation",
  "DeviceCustomString4": "untrust",
  "DeviceCustomString4Label": "FromZone",
  "DeviceCustomString5": "ethernet4Zone-test1",
  "DeviceCustomString5Label": "ToZone",
  "DeviceInboundInterface": "unknown",
  "DeviceOutboundInterface": "unknown",
  "DeviceCustomString6": "rs-logging",
  "DeviceCustomString6Label": "LogSetting",
  "DeviceCustomNumber1": 25596,
  "DeviceCustomNumber1Label": "SessionID",
  "EventCount": 1,
  "SourcePort": 22871,
  "DestinationPort": 27092,
  "SourceTranslatedPort": 24429,
  "DestinationTranslatedPort": 14744,
  "Protocol": "tcp",
  "DeviceAction": "deny",
  "SentBytes": 400448,
  "ReceivedBytes": 969846,
  "DeviceCustomNumber2": 314,
  "DeviceCustomNumber2Label": "PacketsTotal",
  "DeviceCustomNumber3": 56,
  "DeviceCustomNumber3Label": "SessionDuration",
  "DeviceCustomString2": "custom-category",
  "DeviceCustomString2Label": "URLCategory",
  "ExtID": "xxxxxxxxxxxxx",
  "Reason": "unknown",
  "DeviceName": "xxxxx",
  "DeviceEventCategory": "unknown",
  "AdditionalExtensions": "PanOSApplicationContainer=;PanOSApplicationRisk=5;PanOSApplicationSubcategory=file-sharing;PanOSApplicationTechnology=peer-to-peer;PanOSCaptivePortal=false;PanOSCortexDataLakeTenantID=xxxxxxxxxxxxx;PanOSDestinationDeviceClass=;PanOSDestinationDeviceOS=;PanOSInboundInterfaceDetailsPort=0;PanOSInboundInterfaceDetailsSlot=0;PanOSInboundInterfaceDetailsType=unknown;PanOSInboundInterfaceDetailsUnit=0;PanOSIsClienttoServer=false;PanOSIsContainer=false;PanOSIsDecryptMirror=false;PanOSIsDecrypted=false;PanOSIsDecryptedLog=false;PanOSIsDecryptedPayloadForward=false;PanOSIsDuplicateLog=false;PanOSIsEncrypted=false;PanOSIsIPV6=false;PanOSIsInspectionBeforeSession=true;PanOSIsMptcpOn=false;PanOSIsNonStandardDestinationPort=false;PanOSIsPacketCapture=false;PanOSIsPhishing=false;PanOSIsPrismaNetwork=false;PanOSIsPrismaUsers=false;PanOSIsProxy=false;PanOSIsReconExcluded=false;PanOSIsSaaSApplication=false;PanOSIsServertoClient=false;PanOSIsSourceXForwarded=false;PanOSIsSystemReturn=false;PanOSIsTransaction=false;PanOSIsTunnelInspected=false;PanOSIsURLDenied=false;PanOSLogExported=false;PanOSLogForwarded=true;PanOSLogSource=firewall;PanOSLogSourceTimeZoneOffset=;PanOSNAT=false;PanOSNonStandardDestinationPort=0;PanOSOutboundInterfaceDetailsPort=0;PanOSOutboundInterfaceDetailsSlot=0;PanOSOutboundInterfaceDetailsType=unknown;PanOSOutboundInterfaceDetailsUnit=0;PanOSSDWANFECRatio=0.0;PanOSSanctionedStateOfApp=false;PanOSSessionOwnerMidx=false;PanOSSessionTracker=16;PanOSSourceDeviceClass=;PanOSSourceDeviceOS=;PanOSTunneledApplication=tunneled-app;PanOSUsers=xxxxx\\xxxxx xxxxx;PanOSVirtualSystemID=1;PanOSApplicationCategory=peer2peer;PanOSConfigVersion=10.0;PanOSBytes=1370294;PanOSSessionStartTime=Mar 16 2023 20:15:48;PanOSSourceLocation=east-coast;PanOSDestinationLocation=BR;PanOSPacketsSent=194;PanOSPacketsReceived=120;PanOSDGHierarchyLevel1=11;PanOSDGHierarchyLevel2=0;PanOSDGHierarchyLevel3=0;PanOSDGHierarchyLevel4=0;PanOSVirtualSystemName=;PanOSSourceUUID=;PanOSDestinationUUID=;PanOSIMSI=0;PanOSIMEI=;PanOSParentSessionID=0;PanOSParentStarttime=Mar 16 2023 20:15:40;PanOSTunnel=GRE;PanOSEndpointAssociationID=-3746994889972252628;PanOSChunksTotal=1945;PanOSChunksSent=323;PanOSChunksReceived=1622;PanOSRuleUUID=017e4d76-2003-47f4-8afc-1d35c808c615;PanOSHTTP2Connection=469139;PanOSLinkChangeCount=0;PanOSSDWANPolicyName=;PanOSLinkSwitches=;PanOSSDWANCluster=;PanOSSDWANDeviceType=;PanOSSDWANClusterType=;PanOSSDWANSite=;PanOSDynamicUserGroupName=dynug-4;PanOSX-Forwarded-ForIP=xxx.xx.x.xx;PanOSSourceDeviceCategory=N-Phone;PanOSSourceDeviceProfile=n-profile;PanOSSourceDeviceModel=Nexus;PanOSSourceDeviceVendor=Google;PanOSSourceDeviceOSFamily=LG-H790;PanOSSourceDeviceOSVersion=Android v6;PanOSSourceDeviceHost=pan-301;PanOSSourceDeviceMac=839147449905;PanOSDestinationDeviceCategory=N-Phone;PanOSDestinationDeviceProfile=n-profile;PanOSDestinationDeviceModel=Nexus;PanOSDestinationDeviceVendor=Google;PanOSDestinationDeviceOSFamily=H1511;PanOSDestinationDeviceOSVersion=Android v7;PanOSDestinationDeviceHost=pan-355;PanOSDestinationDeviceMac=530589561221;PanOSContainerID=1873cc5c-0d31;PanOSContainerNameSpace=pns_default;PanOSContainerName=pan-dp-77754f4;PanOSSourceEDL=;PanOSDestinationEDL=;PanOSGPHostID=xxxxxxxxxxxxxx;PanOSEndpointSerialNumber=xxxxxxxxxxxxxx;PanOSSourceDynamicAddressGroup=aqua_dag;PanOSDestinationDynamicAddressGroup=;PanOSHASessionOwner=session_owner-4;PanOSTimeGeneratedHighResolution=Feb 27 2021 20:16:18;PanOSNSSAINetworkSliceType=0;PanOSNSSAINetworkSliceDifferentiator=1bca5",
  "TimeGenerated": "2023-03-21T15:00:21.000Z",
  "Type": "CommonSecurityLog",
  "cribl_pipe": "azure_sentinel-common-security-log"
}
```

## Using The Pack

Here's the instructions to use the pack: 

1. Add the pack to your Cribl Stream or Edge instance via github or file upload.
2. Setup a CommonSecurityLog Microsoft Sentinel webhook destination outlined in the guide here: https://docs.cribl.io/stream/usecase-azure-webhook/.
3. Configure a Syslog input that recieves CEF formatted logs and apply the Syslog pre-processing pack https://github.com/criblpacks/cribl-syslog-input as the pre-processing filter in the source configuration. 
4. Configure a route that acccepts the source from step 3, and routes it through this pack and set the destination to your Microsoft CommonSecurityLog webhook. 


## Release Notes

### Version 0.0.1 - 2023-03-21
This is the first release. It's been mainly tested against Palo Alto and Trend Micro based CEF logs but may need some refinement against other sources. 


## Contact
To contact us please email kbrunette@cribl.ip and dgleich@cribl.io. 


## License
TThis Pack uses the following license: [`Apache 2.0`](https://www.apache.org/licenses/LICENSE-2.0).
