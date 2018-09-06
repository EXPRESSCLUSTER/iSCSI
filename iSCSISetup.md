# Setup a iSCSI target on Linux and a iSCSI initiator on Windows Server(CUI)

## Abstract
- This guide provides how to setup a iSCSI target and a initiator.

- A target is placed on CentOS.

- A initiator is placed on Windows Server.


## System Configuration
- Linux OS: CentOS 7.4.1708
- Windows OS: Windows Server 2016 R4


- iSCSI target: targetcli version 2.1.fb46
- iSCSI initiator: Microsoft iSCSI Initiator Version 10.0 Build 17134


## Target setup

1. Install iSCSI target configuration tool
    
    ```bat
    $ yum install -y targetcli
    ```
    
2. Set target.service auto-startup

    ```bat
    $ systemctl enable target.service
    ```
    
3. Create a device used as iSCSI target

    - I created LVM logical volume "LogVol00" in LVM volume group "VolGroup01".
    - After this part, I use LVM as iSCSI target.
    
    
4. Define a backstore

    - Register the LVM as a block device
    
    ```bat
    $ targetcli /backstores/block create name=<block device name> dev=/dev/VolGroup01/LogVol00
    ```
    - Confirm a target configuration
    
    ```bat
    $ targetcli ls
    ```
    
5. Define IQN (iSCSI Qualified Name)

    - Please confirm by yourself how to name IQN properly.
    
    ```bat
    $ targetcli /iscsi create iqn.2018-09.com.iscsi01:target01
    ```
    
6. Connect IQN with a backstore

    ```bat
    $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/luns create /backstores/block/<block device name>
    ```
    
7. Define ACL (Access Control List)

    ```bat
    $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/acls create <initiator IQN>
    ```
    
    - How to confirm initiator IQN is shown below (Windows command).
    
    - "iqn.xxx.xxx.xxx:xxx" is initiator IQN.
    
    ```bat
    > iscsicli
    Microsoft iSCSI Initiator Version 10.0 Build 17134
    
    [iqn.xxx.xxx.xxx:xxx] Enter command or ^C to exit
    ```
    
8. Define IP address

    ```bat
    $ targetcli /iscsi/iqn.2018-09.com.iscsi01:target01/tpg1/portals create <IP address> 3260
    ```
    

## Initiator setup

1. Set msiscsi auto-startup

    - The space after "=" is necessary.
    
    ```bat
    > sc config msiscsi start= auto
    > sc start msiscsi
    ```
   
2. Add iSCSI target

    ```bat
    > iscsicli AddTargetPortal <IP address> <port(default 3260)>
    ```
    
3. Display target list

    ```bat
    > iscsicli ListTargets
    ```
    
4. Login target

    ```bat
    > iscsicli QLoginTarget <IQN>
    ```
    
5. Set persistent connection

    ```bat
    > iscsicli PersistentLoginTarget <IQN> T * * * * * * * * * * * * * * * 0
    ```
    
6. Display persistent connection

    ```bat
    > iscsicli ListPersistentTargets
    ```
        
## Create partition

- Configure disk partitions using "diskpart"
    
    ```bat
    > diskpart
    ```
    
- Display disk list
    
    ```bat
    DISKPART> list disk
    ```
    
- Create partition

    ```bat
    DISKPART> select disk <disk number>
    DISKPART> online disk
    DISKPART> create partition primary size=<size(MB)>
    DISKPART> format fs=ntfs quick
    DISKPART> assign letter=<drive letter>
    ```


## References

- http://ossfan.net/setup/linux-28.html

- http://yoshifumi.hateblo.jp/entry/20080930/p1

- https://www.upken.jp/kb/hyper-v-server-with-iscsi.html