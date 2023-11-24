# zabbix-vmware-snapshot

## Description

Zabbix template to check the snapshot state for VMware VMs

Tested on:
* vCenter 7/8
* Zabbix 6.4


## What does this template do?

<b>Discover VM Snapshots</b> - LLD rule performs virtual machine discovery via vmware.vm.discovery[{$VMWARE.URL}]

<b>{#VM.NAME} Snapshot State</b> - Item prototype pulls snapshot info via vmware.vm.snapshot.get[{$VMWARE.URL},{#VM.UUID}]

<b>{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Age</b> - Item prototype calculates snapshot age via regex + javascript pre-prossesing from parent

<b>{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Count</b> - Item prototype pulls snapshot count via regex pre-processing from parent

<b>{#VM.NAME} Snapshot State: {#VM.NAME} Snapshot Oldest Date</b> - Item prototype pulls oldest snapshot created date via regex pre-processing from parent

<b>[{#VM.NAME}]: VM has more than {$SNAP_COUNT_CRIT} open snapshots</b> - Trigger protptype using a pre-defined threshold macro. Raises a HIGH severity alert

<b>[{#VM.NAME}]: VM has more than {$SNAP_COUNT_WARN} open snapshots</b> - Trigger protptype using a pre-defined threshold macro. Raises a WARNING severity alert. Depends on previous.

<b>[{#VM.NAME}]: VM has snapshots older than {$SNAP_AGE_CRIT} (Oldest snapshot date: {ITEM.VALUE2})</b> - Trigger protptype using a pre-defined threshold macro. Raises a HIGH severity alert

<b>[{#VM.NAME}]: VM has snapshots older than {$SNAP_AGE_WARN} (Oldest snapshot date: {ITEM.VALUE2})</b> - Trigger protptype using a pre-defined threshold macro. Raises a WARNING severity alert. Depends on previous.


## Setup instructions

1. Create a service account on your vCenter server with scope global and read-only permissions
2. Import yaml template to your zabbix server
3. Check/change pre-defined template macros. This can also be done per host.
4. Attach template to your vCenter host. Make sure required macros are filled out


## Required template macros
<pre>
{$VMWARE.URL} = VMware service (vCenter or ESX hypervisor) SDK URL (https://servername/sdk)
{$VMWARE.USERNAME} = VMware service user name
{$VMWARE.PASSWORD} = VMware service {$USERNAME} user password
{$SNAP_AGE_CRIT} = Max number of days since since snapshot was opened before triggering a HIGH alert (Default: 3d)
{$SNAP_AGE_WARN} = Max number of days since since snapshot was opened before triggering a WARNING alert (Default: 1d)
{$SNAP_COUNT_CRIT} = Max number of snapshots before triggering a HIGH alert (Default: 3)
{$SNAP_COUNT_WARN} = Max number of snapshots before triggering a WARNING alert (Default: 1)
</pre>

