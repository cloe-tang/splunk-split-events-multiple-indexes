# splunk-split-events-multiple-indexes
The purpose of this lab is to test events from single log file sending into different indexer and index name based on certain filter criteria. 

## Architecture
![image](https://github.com/user-attachments/assets/bada3dd0-79e7-457f-8686-fef08e55a426)

In this scenario, we are using windows event logs. Events with EventCode 4624 will go to indexer 1 index winsecevent. Events with EventCode 4797|4634|4672 will go to indexer 2 index wineventlog

## Pre-requisites 
1. Have the indexes created in the indexer
2. Have the HF setup to receive logs from the UF and forward it to the indexer

## Part 1 - Setup Universal Forwarder
Step 1: Configure Splunk Universal Forwarder to monitor Windows Event Logs - Security. Copy and paste the following into C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```
[WinEventLog] 
_meta = hf_proxy::meta_test
host = WIN2K16_DC
index = winsecevent

[WinEventLog://Security]
disabled = 0
```

Step 2: Configure Universal Forwarder to send to heavy forwarder
```
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <HF IP address>:9997

[tcpout-server://<HF IP address>:9997]
```
Step 2: Restart the SplunkUniversalForwarder Service

## Part 2 - Configure Heavy Forwarder
Step 1: Create a file transform.conf under /opt/splunk/etc/system/local. Copy and paste the following configuration
```
[defaultRouting]
REGEX=.
DEST_KEY=_TCP_ROUTING
FORMAT=defaultGroup

[wmifilter1]
REGEX=.  
DEST_KEY=_TCP_ROUTING
FORMAT=winEvent1Group
        
[wmifilter2]
REGEX=(?m)^EventCode=(4624) 
DEST_KEY=_TCP_ROUTING
FORMAT=winEvent2Group
    
[changeindex] 
REGEX=(?m)^EventCode=(4797|4634|4672)
DEST_KEY = _MetaData:Index
FORMAT = wineventlog
```
wmifilter1 route event to winEvent1Group output (indexer 1 index winsecevent).
wmifilter2 filters event with EventCode 4624 and route event to winEvent2 Group output (indexer 2 index winsecevent).
changeindex filters event with EventCode 4797|4634|4672 and change the index to wineventlog

Step 2: Create a file props.conf under /opt/splunk/etc/system/local. Copy and paste the following configuration
```
[WinEventLog:Security]
TRANSFORM-wmi=changeindex,wmifilter1,wmifilter2
```
changeindex is the first to make sure splunk changes the index to wineventlog for event with EventCode 4797|4634|4672 before sending to the indexer. 

Step 3: Modify the output.conf under /opt/splunk/etc/system/local
```
[tcpout]
defaultGroup = default-autolb-group
indexAndForward = 0

[tcpout:default-autolb-group]
server = 10.202.11.42:9997

[tcpout:winEvent1Group]
server = 10.202.11.42:9997  ## Indexer 1

[tcpout:winEvent2Group]
server = 10.202.20.60:9997  ## Indexer 2

[tcpout-server://10.202.11.42:9997]
```
Step 4: Restart Splunk Service

## Part 3 - Verify result in both indexer 1 and indexer 2
Indexer 1 is only receiving events with EventCode 4797|4634|4672
![image](https://github.com/user-attachments/assets/daaa982e-d4cc-4ace-8ac4-07fd1faca69a)

Indexer 2 is only receiving events with EventCode 4624
![image](https://github.com/user-attachments/assets/f7636129-5048-4ab9-86eb-e83d7c7f3d6a)



