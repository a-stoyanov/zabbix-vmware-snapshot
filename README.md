# zabbix-vmware-snapshot

## Description

Zabbix template to check the snapshot state for VMware VMs

Tested on:
* vCenter 7/8
* Zabbix 6.4

## Requirements:
* Zabbix v6.4 or later
* Zabbix template VMware (https://www.zabbix.com/integrations/vmware)

## Setup:

1. Create a service account on your vCenter server with scope global and read-only permissions
2. Import yaml template to your zabbix server
3. Attach template to your vCenter host. Make sure required macros are filled out

## Required macros:

|Macro|Default Value|Description|
|-----|-------------|-----------|
|{$VMWARE.URL}|N/A|VMware service (vCenter or ESX hypervisor) SDK URL (https://servername/sdk)|
|{$VMWARE.USERNAME}|N/A|VMware service user name|
|{$VMWARE.PASSWORD}|N/A|VMware service user pass|
|{$SNAP_AGE_CRIT}|3d|Max number of days since snapshot was opened before triggering a HIGH alert|
|{$SNAP_AGE_WARN}|1d|Max number of days since snapshot was opened before triggering a WARNING alert|
|{$SNAP_COUNT_CRIT}|3|Max number of snapshots before triggering a HIGH alert|
|{$SNAP_COUNT_WARN}|1|Max number of snapshots before triggering a WARNING alert|

## LLD rules:

|Name|Description|Type|Key and additional info|
|----|-----------|----|----|
|Discover VM Snapshots|Discovery of guest VM/snapshots|Simple Check|vmware.vm.discovery[{$VMWARE.URL}], Update interval: 1h|

## Item prototypes:

|Name|Description|Type|Key and additional info|
|----|-----------|----|----|
|{#VM.NAME} Snapshot State|Get snapshot info|Text|vmware.vm.snapshot.get[{$VMWARE.URL},{#VM.UUID}], Update interval: 1m|
|{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Age|Calculate snapshot age via regex + javascript pre-prossesing from parent|Dependant item|vmware.vm.snapshot.age[{#VM.NAME}]|
|{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Count|Get snapshot count via jsonpath pre-processing from parent|Dependant item|vmware.vm.snapshot.count[{#VM.NAME}]|
|{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Oldest Date|Get oldest snapshot created date via regex pre-processing from parent|Dependant item|vmware.vm.snapshot.oldestdate[{#VM.NAME}]|

## Trigger prototypes:
|Name|Description|Expression|Severity|
|----|-----------|----------|--------|
|[{#VM.NAME}]: VM has more than {$SNAP_COUNT_CRIT} open snapshots|Raise alert when number of snapshots is over threshold|last(/VMware Snapshot/vmware.vm.snapshot.count[{#VM.NAME}])>{$SNAP_COUNT_CRIT}|High|
|[{#VM.NAME}]: VM has more than {$SNAP_COUNT_WARN} open snapshots|Raise alert when number of snapshots is over threshold|last(/VMware Snapshot/vmware.vm.snapshot.count[{#VM.NAME}])>{$SNAP_COUNT_WARN}|Warning|
|[{#VM.NAME}]: VM has snapshots older than {$SNAP_AGE_CRIT} (Oldest snapshot date: {ITEM.VALUE2})|Raise alert when oldest snapshot age is over threshold|last(/VMware Snapshot/vmware.vm.snapshot.age[{#VM.NAME}])>{$SNAP_AGE_CRIT} and last(/VMware Snapshot/vmware.vm.snapshot.oldestdate[{#VM.NAME}])<>0|High|
|[{#VM.NAME}]: VM has snapshots older than {$SNAP_AGE_WARN} (Oldest snapshot date: {ITEM.VALUE2})|Raise alert when oldest snapshot age is over threshold|last(/VMware Snapshot/vmware.vm.snapshot.age[{#VM.NAME}])>{$SNAP_AGE_WARN} and last(/VMware Snapshot/vmware.vm.snapshot.oldestdate[{#VM.NAME}])<>0|Warning|
