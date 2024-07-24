# splunk-split-events-multiple-indexes
The purpose of this lab is to test events from single log file sending into different indexes based on certain filter criteria. In this scenario, we are using windows event logs. Events with EventCode 4624 will go to index 1. Events with EventCode 4797|4634|4672 will go to index 2
