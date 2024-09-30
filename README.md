# MegaRAID-udev

This project provides a way to access drives exposed from a LSI/Avago/Broadcom
MegaRAID storage adapter using the same terminology used by those adapters
(i.e., controller, enclosures, slots, virtual drives, etc). Symlinks are added
under `/dev/megaraid` that point to the actual OS devices.

## Requirements

* Python3 (tested with Python 3.11)
* StorCli tool -- Latest version can be found at Broadcom's web site:
  1. Visit https://www.broadcom.com/support/download-search
     (or click "Support and Services" -> "Document, Downloads and Support" ->
     "Support Documents and Downloads")
  2. In the search form, do this:
     1. Check the **Include Legacy Products** checkbox
     2. Under *Product Group*, select **Storage Adapters, Controllers, and ICs**
     3. Under *Product Family*, select **Host Bus Adapters**
     4. Under *Product Name*, type **storcli**
     5. Leave *Asset Type* empty
     6. Click *Search*
  3. Look for **Management Software and Tools** node and expand it
  4. Under that node, look for the most recent StorCli download available for
  your adapter
     1. For SAS 3.5 adapters (SAS31xx-based, e.g.: 9361-8i), as of Sep/2024, the
     latest StorCli available is the Phase32 release (`STORCLI_SAS3.5_P32.zip`)
  5. Download the zip file, extract it and follow the instructions. For
  Debian-based systems install the `.deb` package found under the Ubuntu folder.

Note: newer adapters use a newer StorCli2 tool which hasn't been tested

## Installation

0. Ensure udev is installed and running on your system (it should be as it's pretty common)
1. Copy the contents of the `/etc` directory to your device
2. Ensure `/etc/udev/megaraid_id` script is executable

```shell
sudo chmod 755 /etc/udev/megaraid_id
```

3. Trigger udev to read the new rules

```shell
sudo udevadm trigger
```

## Examples

In this example, there's only a single drive set as JBOD attached to controller 0:

```
root@host:~# /opt/MegaRAID/storcli/storcli64 /c0 /eall /sall show jbod
[...]
------------------------------------------------------------------------------------------
EID:Slt DID State Intf Med       Size SeSz Model                   Vendor   Port
------------------------------------------------------------------------------------------
252:4    23 JBOD  SATA SSD 931.512 GB 512B Samsung SSD 860 EVO 1TB ATA      Port 4 - 7 x1
------------------------------------------------------------------------------------------
[...]
root@host:~# ls -l /dev/megaraid/c0/e252
total 0
lrwxrwxrwx 1 root root 12 Sep 30 11:39 s4 -> ../../../sdb
```

Next, there are 5 virtual drives attached to the same controller (controller 0):

```
[...]
------------------------------------------------------------------------
DG/VD TYPE  State Access Consist Cache Cac sCC       Size Name
------------------------------------------------------------------------
1/0   RAID1 Optl  RW     Yes     RWBD  -   ON  931.000 GB VM Store
2/1   RAID1 Optl  RW     Yes     RWBD  -   ON  931.000 GB Backup
0/2   RAID1 Optl  RW     Yes     RWBD  -   ON    1.818 TB New VM Store
------------------------------------------------------------------------
[...]
root@host:~# ls -l /dev/megaraid/c0
total 0
drwxr-xr-x 2 root root 60 Sep 30 11:39 e252
lrwxrwxrwx 1 root root  9 Sep 30 11:39 v0 -> ../../sda
lrwxrwxrwx 1 root root  9 Sep 30 11:39 v1 -> ../../sdc
lrwxrwxrwx 1 root root  9 Sep 30 11:39 v2 -> ../../sdd
```
