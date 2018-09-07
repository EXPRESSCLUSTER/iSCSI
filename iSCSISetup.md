# Setup an iSCSI target on Linux and an iSCSI initiator on Windows Server(CUI)

## Introduction
- This guide provides how to setup an iSCSI target and an initiator.

- The target is installed on CentOS.

- The initiator is installed on Windows Server.


## System Configuration
- Linux OS: CentOS 7.4.1708
- Windows OS: Windows Server 2016 R4


- iSCSI target: targetcli version 2.1.fb46
- iSCSI initiator: Microsoft iSCSI Initiator Version 10.0 Build 17134

## figure

## Target setup

1. Install iSCSI target configuration tool
    
    ```bat
    $ yum install -y targetcli
    ```
    
2. Set iSCSI target startup type

    ```bat
    $ systemctl enable target.service
    ```
    
3. Create a LVM logical volume for iSCSI target virtual volume

    e.g. LVM logical volume"LogVol00" in LVM volume group "VolGroup01".
    
    
4. Define a backstore

    Register the LVM as a block device
    
    ```bat
    $ targetcli /backstores/block create name=<block device name> dev=<device path>
    
    e.g. $ targetcli /backstores/block create name=lun1 dev=/dev/VolGroup01/LogVol00
    ```
    Confirm a target configuration
    
    ```bat
    $ targetcli ls
    ```
    
5. Define target IQN (iSCSI Qualified Name)
    
    Please look up how to name target IQN in the URL below.
    - https://docs.vmware.com/jp/VMware-vSphere/6.5/com.vmware.vsphere.storage.doc/GUID-686D92B6-A2B2-4944-8718-F1B74F6A2C53.html
    
    
    ```bat
    $ targetcli /iscsi create <target IQN>
    
    e.g. $ targetcli /iscsi create iqn.2018-09.com.iscsi01:target01
    ```
    
6. Connect target IQN with a backstore

    ```bat
    $ targetcli /iscsi/<target IQN>/tpg1/luns create /backstores/block/<block device name>
    
    e.g. $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/luns create /backstores/block/lun1
    ```
    
7. Define ACL (Access Control List)

    ```bat
    $ targetcli /iscsi/<target IQN>/tpg1/acls create <initiator IQN>
    
    e.g. $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/acls create iqn.1991-05.com.microsoft:vserver1
    ```
    
        On **Windows**, please confirm initiator IQN.
    
        "iqn.xxx.xxx.xxx:xxx" is initiator IQN.
    
        ```bat
        > iscsicli
        Microsoft iSCSI Initiator Version 10.0 Build 17134
    
        [iqn.xxx.xxx.xxx:xxx] Enter command or ^C to exit
        ```
    
8. Define IP address

    ```bat
    $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/portals create <IP address> <port>
    
    e.g. $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/portals create 192.168.100.100 3260
    ```
    

## Initiator setup

1. Set iSCSI initiator startup type 

    **NOTE:** The space after "=" is necessary.
    
    ```bat
    > sc config msiscsi start= auto
    > sc start msiscsi
    ```
   
2. Add iSCSI target

    ```bat
    > iscsicli AddTargetPortal <IP address> <port>
    ```
    
3. Display target list

    ```bat
    > iscsicli ListTargets
    ```
    
4. Login target

    ```bat
    > iscsicli QLoginTarget <target IQN>
    ```
    
5. Set persistent connection

    ```bat
    > iscsicli PersistentLoginTarget <targetIQN> T * * * * * * * * * * * * * * * 0
    ```
    
6. Display persistent connection list

    ```bat
    > iscsicli ListPersistentTargets
    ```
  
## Appendix: How to create partition      
### Create partition

1. Configure disk partitions using "diskpart"
    
    ```bat
    > diskpart
    ```
    
2. Display disk list
    
    ```bat
    DISKPART> list disk
    ```
    
3. Create partition

    ```bat
    DISKPART> select disk <disk number>
    DISKPART> online disk
    DISKPART> create partition primary size=<size(MB)>
    DISKPART> format fs=ntfs quick
    DISKPART> assign letter=<drive letter>
    ```


### References

- http://ossfan.net/setup/linux-28.html

- http://yoshifumi.hateblo.jp/entry/20080930/p1

- https://www.upken.jp/kb/hyper-v-server-with-iscsi.html
