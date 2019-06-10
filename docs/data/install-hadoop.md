Installing and Maintaining HDFS
===============================

[Hadoop Distributed File System](http://hadoop.apache.org/) (HDFS) is a scalable, reliable distributed file system developed in the Apache project. It is based on the map-reduce framework and design of the Google file system. The OSG distribution of Hadoop includes all components needed to operate a multi-terabyte storage site.

The purpose of this document is to provide Hadoop-based Storage Element administrators the information on how to prepare,
install and validate OSG storage based on the Hadoop Distributed File System (HDFS).
The OSG supports a patched version HDFS from Cloudera's CDH5 distribution of HDFS
(<https://www.cloudera.com/products/open-source/apache-hadoop/key-cdh-components.html>).

!!! note
    The OSG only supports HDFS on EL7 hosts

Before Starting
---------------

Before starting the installation process, consider the following points (consulting [the Reference section below](#reference) as needed):

-   **User IDs:** If they do not exist already, the installation will create the Linux users `hdfs` and `zookeeper` on all nodes
    as well as `hadoop` and `mapred` on the NameNodes
-   **Firewall:** In the OSG, HDFS is intended to run as an internal service without any direct, external access to any of the nodes.
    For more information on the ports used for communication between the various HDFS nodes, see the 
    [Cloudera documentation](https://www.cloudera.com/documentation/cdh/5-0-x/CDH5-Installation-Guide/cdh5ig_ports_cdh5.html).

As with all OSG software installations, there are some one-time (per host) steps to prepare in advance:

- Ensure the host has [a supported operating system](/release/supported_platforms)
- Obtain root access to the host
- Prepare the [required Yum repositories](/common/yum)

Designing Your HDFS Cluster
---------------------------

There are several important components to an HDFS installation:

-   **NameNode**: The NameNode functions as the directory server and coordinator of the HDFS cluster.
    It houses all the meta-data for the hadoop cluster.
-   **Secondary NameNode (optional)**: This is a secondary machine that periodically merges updates to the HDFS file
    system back into the `fsimage`.
    It must share a directory with the primary NameNode to exchange filesystem checkpoints.
    An HDFS installation with a Secondary NameNode dramatically improves startup and restart times.
-   **DataNode**: You will have many DataNodes. Each DataNode stores large blocks of files to for the hadoop cluster.
-   **Client**: This is a documentation shorthand that refers to any machine with the hadoop client commands or
    [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) mount.

Installing HDFS
---------------

An OSG HDFS installation consists of HDFS and other support software (e.g., Gratia accounting).
To simplify installation, OSG provides convenience RPMs that install all required software.

1. Clean yum cache:

        ::console
        root@host # yum clean all --enablerepo=*

1. Update software:

        :::console
        root@host # yum update
    This command will update **all** packages

1. Install the relevant packages based on the node you are installing:

    | If you are installing a(n)... | Then run the following command...             |
    | :---------------------------- | :--------------------------------             |
    | Primary NameNode              | `yum install osg-se-hadoop-namenode`          |
    | Secondary NameNode            | `yum install osg-se-hadoop-secondarynamenode` |
    | DataNode                      | `yum install osg-se-hadoop-datanode`          |


Upgrading HDFS
--------------

This section will guide you through the process to upgrade a HDFS 2.0.0 installation from OSG 3.3 to the HDFS 2.6.0
from OSG 3.4.

!!! warning
    The upgrade process will involve downtime for your HDFS cluster. Please plan accordingly.

!!! note
    The OSG only offers HDFS 2.6.0 for EL7 hosts.

The upgrade process occurs in several steps:

1. [Preparing for the upgrade](#preparing-for-the-upgrade)
1. [Updating to OSG 3.4](#updating-to-osg-34)
1. [Upgrading the Primary NameNode](#upgrading-the-primary-namenode)
1. [Upgrading the DataNodes](#upgrading-the-datanodes)
1. [Upgrading the Secondary NameNode](#upgrading-the-secondary-namenode)
1. [Finalizing the upgrade](#finalizing-the-upgrade)

### Preparing for the upgrade ###

Before upgrading, backup your configuration data and HDFS metadata.

1. Put your Primary NameNode into safe mode:

        :::console
        root@primary-namenode # hdfs dfsadmin -safemode enter
        Safe mode is ON

1. Save a clean copy of your HDFS namespace:

        :::console
        root@primary-namenode # hdfs dfsadmin -saveNamespace
        Save namespace successful

1. Shutdown the HDFS services on all of your HDFS nodes (see [this section](#running-services) for instructions).

1. On the Primary NameNode, verify that your NameNode service is off:

        :::console
        root@primary-namenode # /etc/init.d/hadoop-hdfs-namenode status

    This command should indicate that your NameNode service is not running.

1. Find the location of the directory with the HDFS metadata:

        :::console
        root@primary-namenode # grep -C1 dfs.namenode.name.dir /etc/hadoop/conf/hdfs-site.xml

    And look for the value of `dfs.namenode.name.dir`:

        :::xml
        <property>
         <name>dfs.namenode.name.dir</name>
          <value>file:///var/lib/dfs/nn,file:///home/hadoop/dfs/nn</value>

1. Backup the directory that appears in the output using your backup method of choice.
   If more than one directory appears in the list (as in the example above), choose the most convenient directory.
   All of the directories in the list will have the same contents.

### Updating to OSG 3.4 ###

Once your HDFS services have been turned off and the HDFS metadata has been backed up, update each node to OSG 3.4 by
following the instructions in [this section](/release/release_series/#updating-from-old).

### Upgrading the Primary NameNode ###

To upgrade your Primary NameNode, update all relevant packages then run the upgrade command.

1. Clear the yum cache:

        :::console
        root@primary-namenode # yum clean all --enablerepo=*

1. Update the HDFS RPMs:

        :::console
        root@primary-namenode # yum update osg-se-hadoop-namenode --enablerepo-osg-upcoming

1. Perform the upgrade command:

        :::console
        root@primary-namenode # /etc/init.d/hadoop-hdfs-namenode upgrade

    This will start the upgrade process for the HDFS metadata on your primary namenode.
    You can follow the process by running

        :::console
        root@primary-namenode # tail -f /var/log/hadoop-hdfs/hadoop-hdfs-namenode-<hostname>.log

### Upgrading the DataNodes ###

Once the Primary NameNode has completed its upgrade process, start the process of upgrading each of your DataNodes.

1. Clear the yum cache:

        :::console
        root@datanode # yum clean all --enablerepo=*

1. Update the HDFS RPMs:

        :::console
        root@datanode # yum update osg-se-hadoop-datanode --enablerepo-osg-upcoming

1. Start the DataNode service:

        :::console
        root@datanode # /etc/init.d/hadoop-hdfs-datanode start

1. After all the DataNodes have been brought back up, the Primary NameNode should exit safe mode automatically.
   On the Primary NameNode, run the following command to verify is no longer in safe mode:

        :::console
        root@primary-namenode # hdfs dfsadmin -safemode get
        Safe mode is OFF

   

### Upgrading the Secondary NameNode ###

!!! note
    This section only applies to sites with a Secondary NameNode.
    If you do not run a Secondary NameNode, skip to the [next section](#finalizing-the-upgrade).

Once the Primary NameNode has exited safe mode, start the process of upgrading your Secondary NameNode.

1. Clear the yum cache:

        :::console
        root@secondary-namenode # yum clean all --enablerepo=*

1. Update the HDFS RPMs:

        :::console
        root@secondary-namenode # yum update osg-se-hadoop-secondarynamenode --enablerepo-osg-upcoming

1. Start the Secondary NameNode service:

        :::console
        root@secondary-namenode # /etc/init.d/hadoop-hdfs-secondarynamenode start

### Finalizing the upgrade ###

1. Verify that the HDFS cluster is running correctly by following the instructions in [this section](#validation_1).

1. Finalize the upgrade from the Primary NameNode:

        :::console
        root@primary-namenode # hdfs dfsadmin -finalizeUpgrade
        Finalize upgrade successful

Configuring HDFS
----------------

!!! note
    Needed by: Hadoop NameNode, Hadoop DataNodes, Hadoop client, GridFTP

Hadoop configuration is needed by every node in the hadoop cluster. However, in most cases, you can do the configuration once and copy it to all nodes in the cluster (possibly using your favorite configuration management tool). Special configuration for various special components is given in the below sections.

Hadoop configuration is stored in `/etc/hadoop/conf`. However, by default, these files are mostly blank. OSG provides a sample configuration in `/etc/hadoop/conf.osg` with most common values filled in. You will need to copy these into `/etc/hadoop/conf` before they become active. Please let us know if there are any common values that should be added/changed across the whole grid. You will likely need to modify `hdfs-site.xml` and `core-site.xml`. Review all the settings in these files, but listed below are common settings to modify:

| File            | Setting                    | Example                          | Comments                                                                                  |
|-----------------|----------------------------|----------------------------------|-------------------------------------------------------------------------------------------|
| `core-site.xml` | fs.default.name            | hdfs://namenode.domain.tld.:9000 | This is the address of the NameNode                                                       |
| `core-site.xml` | hadoop.tmp.dir             | /data/scratch                    | Scratch temp directory used by Hadoop                                                     |
| `core-site.xml` | hadoop.log.dir             | /var/log/hadoop-hdfs             | Log directory used by Hadoop                                                              |
| `core-site.xml` | dfs.umaskmode              | 002                              | umask for permissions used by default                                                     |
| `hdfs-site.xml` | dfs.block.size             | 134217728                        | Block size: 128MB by default                                                              |
| `hdfs-site.xml` | dfs.replication            | 2                                | Default replication factor. Generally the same as dfs.replication.min/max                 |
| `hdfs-site.xml` | dfs.datanode.du.reserved   | 100000000                        | How much free space hadoop will reserve for non-Hadoop usage                              |
| `hdfs-site.xml` | dfs.datanode.handler.count | 20                               | Number of server threads for DataNodes. Increase if you have many more client connections |
| `hdfs-site.xml` | dfs.namenode.handler.count | 40                               | Number of server threads for NameNodes. Increase if you need more connections             |
| `hdfs-site.xml` | dfs.http.address           | namenode.domain.tld.:50070       | Web address for dfs health monitoring page                                                |

See <http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml> for more parameters to configure.

!!! note
    NameNodes must have a `/etc/hosts_exclude` present

#### Special NameNode instructions for brand new installs

If this is a new installation (%RED%and only if this is a brand new installation%ENDCOLOR%), you should run the following command as the `hdfs` user. (Otherwise, be sure to `chown` your storage directory to hdfs after running):

``` console
hadoop namenode -format
```

This will initialize the storage directory on your NameNode

### (optional) FUSE Client Configuration ###

A FUSE mount is required on any node that you would like to use standard POSIX-like commands on the Hadoop filesystem. FUSE (or "File system in User SpacE") is a way to access HDFS using typical UNIX directory commands (i.e., POSIX-like access). Note that not all advanced functions of a full POSIX-compliant file system are necessarily available.

FUSE is typically installed as part of this installation, but, if you are running a customized or non-standard system, make sure that the fuse kernel module is installed and loaded with `modprobe fuse`.

You can add the FUSE to be mounted at boot time by adding the following line to `/etc/fstab`:

``` file
hadoop-fuse-dfs# %RED%/mnt/hadoop%ENDCOLOR% fuse server=%RED%namenode.host%ENDCOLOR%,port=9000,rdbuffer=131072,allow_other 0 0
```

Be sure to change the `/mnt/hadoop` mount point and `namenode.host` to match your local configuration. To match the help documents, we recommend using `/mnt/hadoop` as your mountpoint.

Once your `/etc/fstab` is updated, to mount FUSE run:

``` console
root@host # mkdir /mnt/hadoop
root@host # mount /mnt/hadoop
```

When mounting the HDFS FUSE mount, you will see the following harmless warnings printed to the screen:

``` console
# mount /mnt/hadoop
INFO fuse_options.c:162 Adding FUSE arg /mnt/hadoop
INFO fuse_options.c:110 Ignoring option allow_other
```

If you have troubles mounting FUSE refer to [Running FUSE in Debug Mode](#running-fuse-in-debug-mode) in the Troubleshooting section.

### Creating VO and User Areas ###

!!! note
    Grid Users are needed by GridFTP nodes. VO areas are common to all nodes.

For this package to function correctly, you will have to create the users needed for grid operation. Any user that can be authenticated should be created.

For grid-mapfile users, each line of the grid-mapfile is a certificate/user pair. Each user in this file should be created on the server.

Note that these users must be kept in sync with the authentication method.

Prior to starting basic day-to-day operations, it is important to create dedicated areas for each VO and/or user. This is similar to user management in simple UNIX filesystems. Create (and maintain) usernames and groups with UIDs and GIDs on **all nodes**. These are maintained in basic system files such as `/etc/passwd` and `/etc/group`.

!!! note
    In the examples below It is assumed a FUSE mount is set to `/mnt/hadoop`. As an alternative `hadoop fs` commands could have been used.

For clean HDFS operations and filesystem management:

(a) Create top-level VO subdirectories under `/mnt/hadoop`.

Example:

``` console
root@host # mkdir /mnt/hadoop/cms
root@host # mkdir /mnt/hadoop/dzero
root@host # mkdir /mnt/hadoop/sbgrid
root@host # mkdir /mnt/hadoop/fermigrid
root@host # mkdir /mnt/hadoop/cmstest
root@host # mkdir /mnt/hadoop/osg
```

(b) Create individual top-level user areas, under each VO area, as needed.

``` console
root@host # mkdir -p /mnt/hadoop/cms/store/user/tanyalevshina
root@host # mkdir -p /mnt/hadoop/cms/store/user/michaelthomas
root@host # mkdir -p /mnt/hadoop/cms/store/user/brianbockelman
root@host # mkdir -p /mnt/hadoop/cms/store/user/douglasstrain
root@host # mkdir -p /mnt/hadoop/cms/store/user/abhisheksinghrana
```

(c) Adjust username:group ownership of each area.

``` console
root@host # chown -R cms:cms /mnt/hadoop/cms
root@host # chown -R sam:sam /mnt/hadoop/dzero

root@host # chown -R michaelthomas:cms /mnt/hadoop/cms/store/user/michaelthomas
```

### GridFTP Configuration ###

gridftp-hdfs reads the Hadoop configuration file to learn how to talk to Hadoop.
By now, you should have followed the instruction for installing hadoop as detailed in the previous section as well as
created the proper users/directories.

The default settings in `/etc/gridftp.conf` along with `/etc/gridftp.d/gridftp-hdfs.conf` are used by the init.d script
and should be ok for most installations.
The file `/etc/gridftp-hdfs/gridftp-debug.conf` is used by `/usr/bin/gridftp-hdfs-standalone` for starting up the
GridFTP server in a testing mode.
Any additional config files under `/etc/gridftp.d` will be used for both the init.d and standalone GridFTP server.
`/etc/sysconfig/gridftp-hdfs` contains additional site-specific environment variables that are used by the gridftp-hdfs
DSI module in both the init.d and standalone GridFTP server.
Some of the environment variables that can be used in `/etc/sysconfig/gridftp-hdfs` include:

| Option Name                 | Needs Editing? | Suggested value                                                                                                                                                                                                                                                                 |
|-----------------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GRIDFTP_HDFS_REPLICA_MAP`  | No             | File containing a list of paths and replica values for setting the default # of replicas for specific file paths                                                                                                                                                                |
| `GRIDFTP_BUFFER_COUNT`      | No             | The number of 1MB memory buffers used to reorder data streams before writing them to Hadoop                                                                                                                                                                                     |
| `GRIDFTP_FILE_BUFFER_COUNT` | No             | The number of 1MB file-based buffers used to reorder data streams before writing them to Hadoop                                                                                                                                                                                 |
| `GRIDFTP_SYSLOG`            | No             | Set this to 1 in case if you want to send transfer activity data to syslog (only used for the HadoopViz application)                                                                                                                                                            |
| `GRIDFTP_HDFS_CHECKSUMS`    | Maybe          | List of checksum calculations to perform on-the-fly (default: `"MD5,ADLER32,CRC32,CKSUM,CVMFS"`)                                                                                                                                                                                                                                                                                |
| `GRIDFTP_HDFS_MOUNT_POINT`  | Maybe          | The location of the FUSE mount point used during the Hadoop installation. Defaults to /mnt/hadoop. This is needed so that gridftp-hdfs can convert fuse paths on the incoming URL to native Hadoop paths. **Note:** this does not imply you need FUSE mounted on GridFTP nodes! |
| `GRIDFTP_LOAD_LIMIT`        | No             | GridFTP will refuse to start new transfers if the load on the GridFTP host is higher than this number; defaults to 20.                                                                                                                                                          |
| `TMPDIR`                    | Maybe          | The temp directory where the file-based buffers are stored. Defaults to /tmp.                                                                                                                                                                                                   |

`/etc/sysconfig/gridftp-hdfs` is also a good place to increase per-process resource limits. For example, many installations will require more than the default number of open files (`ulimit -n`).

Lastly, you will need to configure an authentication mechanism for GridFTP.

#### Configuring authentication ####

For information on how to configure authentication for your GridFTP installation, please refer to the [configuring authentication section of the GridFTP guide](gridftp#configuring-authentication).

### GridFTP Gratia Transfer Probe Configuration ###

!!! note
    Needed by GridFTP node only.

See the [GridFTP documentation](/data/gridftp#enabling-gridftp-transfer-probe) for configuration details.

### Hadoop Storage Probe Configuration ###

!!! note
    This is only needed by the Hadoop NameNode

Here are the most relevant file and directory locations:

| Purpose             | Needs Editing? | Location                                                |
|---------------------|----------------|---------------------------------------------------------|
| Probe Configuration | Yes            | /etc/gratia/hadoop-storage/ProbeConfig                  |
| Probe Executable    | No             | /usr/share/gratia/hadoop-storage/hadoop\_storage\_probe |
| Log files           | No             | /var/log/gratia                                         |
| Temporary files     | No             | /var/lib/gratia/tmp                                     |

The RPM installs the Gratia probe into the system crontab, but does not configure it. The configuration of the probe is controlled by two files

    /etc/gratia/hadoop-storage/ProbeConfig
    /etc/gratia/hadoop-storage/storage.cfg

#### ProbeConfig ####

This is usually one XML node spread over multiple lines. Note that comments (\#) have no effect on this file. You will need to edit the following:

| Attribute     | Needs Editing | Value                                                                                                                                   |
|---------------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| CollectorHost | Maybe         | Set to the hostname and port of the central collector. By default it sends to the OSG collector. You probably do not want to change it. |
| SiteName      | Yes           | Set to the resource group name of your SE as registered in OIM.                                                                         |
| Grid          | Maybe         | Set to "ITB" if this is a test resource; otherwise, leave as OSG.                                                                       |
| EnableProbe   | Yes           | Set to 1 to enable the probe.                                                                                                           |

#### storage.cfg ####

This file controls which paths in HDFS should be monitored. This is in the Windows INI format.

**Note: for the current version of the storage.cfg, there is an error, and you may need to delete the "probe/" subdirectory for the ProbeConfig location**

``` file
ProbeConfig = /etc/gratia/%RED%probe/%ENDCOLOR%hadoop-storage/ProbeConfig
```

For each logical "area" (arbitrarily defined by you), specify both a given name and a list of paths that belong to that area. Unix globs are accepted.

To configure an area named "CMS /store" that monitors the space usage in the paths /user/cms/store/\*, one would add the following to the storage.cfg file.

``` file
[Area CMS /store]
Name = CMS /store
Path = /user/cms/store/*
Trim = /user/cms
```

For each such area, add a section to your configuration file.

##### Example file #####

Below is a configuration file that includes three distinct areas. Note that you shouldn't have to touch the \[Gratia\] section if you edited the ProbeConfig above:

``` file
[Gratia]
gratia_location = /opt/vdt/gratia
ProbeConfig = %(gratia_location)s/probe/hadoop-storage/ProbeConfig

[Area /store]
Name = CMS /store
Path = /store/*

[Area /store/user]
Name = CMS /store/user
Path = /store/user/*

[Area /user]
Name = Hadoop /user
Path = /user/*
```

\***NOTE These lines in the \[gratia\] section are wrong and need to be changed to the following by hand for now until the rpm is updated:**

``` file
gratia_location = /etc/gratia
ProbeConfig = %(gratia_location)s/hadoop-storage/ProbeConfig
```

Running Services
----------------


Start the services in the order listed and stop them in reverse order. As a reminder, here are common service commands (all run as `root`):

| To...                                   | On EL6, run the command...                  | On EL7, run the command...                      |
| :-------------------------------------- | :----------------------------------------   | :--------------------------------------------   |
| Start a service                         | `service <SERVICE-NAME> start` | `systemctl start <SERVICE-NAME>`   |
| Stop a  service                         | `service <SERVICE-NAME> stop`  | `systemctl stop <SERVICE-NAME>`    |
| Enable a service to start on boot       | `chkconfig <SERVICE-NAME> on`  | `systemctl enable <SERVICE-NAME>`  |
| Disable a service from starting on boot | `chkconfig <SERVICE-NAME> off` | `systemctl disable <SERVICE-NAME>` |


The relevant service for each node is as follows:

| Node               | Service                       |
| :----------------- | :---------------------------- |
| Primary NameNode   | hadoop-hdfs-namenode          |
| Secondary NameNode | hadoop-hdfs-secondarynamenode |
| DataNode           | hadoop-hdfs-datanode          |
| GridFTP            | globus-gridftp-server         |



Validation
----------

The first thing you may want to do after installing and starting your primary NameNode is to verify that the web interface works. In your web browser go to:

``` file
http://%RED%namenode.hostname%ENDCOLOR%:50070/dfshealth.jsp
```

Get familiar with Hadoop commands. Run hadoop with no arguments to see the list of commands.

<details>
  <summary>Show detailed ouput</summary>
   <p>
``` console
user$ hadoop
Usage: hadoop [--config confdir] COMMAND
where COMMAND is one of:
  namenode -format     format the DFS filesystem
  secondarynamenode    run the DFS secondary namenode
  namenode             run the DFS namenode
  datanode             run a DFS datanode
  dfsadmin             run a DFS admin client
  mradmin              run a Map-Reduce admin client
  fsck                 run a DFS filesystem checking utility
  fs                   run a generic filesystem user client
  balancer             run a cluster balancing utility
  fetchdt              fetch a delegation token from the NameNode
  jobtracker           run the MapReduce job Tracker node
  pipes                run a Pipes job
  tasktracker          run a MapReduce task Tracker node
  job                  manipulate MapReduce jobs
  queue                get information regarding JobQueues
  version              print the version
  jar <jar>            run a jar file
  distcp <srcurl> <desturl> copy file or directories recursively
  archive -archiveName NAME -p <parent path> <src>* <dest> create a hadoop archive
  oiv                  apply the offline fsimage viewer to an fsimage
  classpath            prints the class path needed to get the
                       Hadoop jar and the required libraries
  daemonlog            get/set the log level for each daemon
 or
  CLASSNAME            run the class named CLASSNAME
Most commands print help when invoked w/o parameters.
```
</p>
</details>

For a list of supported filesystem commands:

<details>
  <summary>Show 'hadoop fs' detailed ouput</summary>
   <p>
``` console
user$ hadoop fs
Usage: java FsShell
           [-ls <path>]
           [-lsr <path>]
           [-df [<path>]]
           [-du <path>]
           [-dus <path>]
           [-count[-q] <path>]
           [-mv <src> <dst>]
           [-cp <src> <dst>]
           [-rm [-skipTrash] <path>]
           [-rmr [-skipTrash] <path>]
           [-expunge]
           [-put <localsrc> ... <dst>]
           [-copyFromLocal <localsrc> ... <dst>]
           [-moveFromLocal <localsrc> ... <dst>]
           [-get [-ignoreCrc] [-crc] <src> <localdst>]
           [-getmerge <src> <localdst> [addnl]]
           [-cat <src>]
           [-text <src>]
           [-copyToLocal [-ignoreCrc] [-crc] <src> <localdst>]
           [-moveToLocal [-crc] <src> <localdst>]
           [-mkdir <path>]
           [-setrep [-R] [-w] <rep> <path/file>]
           [-touchz <path>]
           [-test -[ezd] <path>]
           [-stat [format] <path>]
           [-tail [-f] <file>]
           [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
           [-chown [-R] [OWNER][:[GROUP]] PATH...]
           [-chgrp [-R] GROUP PATH...]
           [-help [cmd]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|jobtracker:port>    specify a job tracker
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
```
</p>
</details>

An online guide is also available at [Apache Hadoop commands manual](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/CommandsManual.html). You can use Hadoop commands to perform filesystem operations with more consistency.

Example, to look into the internal hadoop namespace:

``` console
user$ hadoop fs -ls /
Found 1 items
drwxrwxr-x   - engage engage          0 2011-07-25 06:32 /engage
```

Example, to adjust ownership of filesystem areas (there is usually no need to specify the mount itself `/mnt/hadoop` in Hadoop commands):

``` console
root@host # hadoop fs -chown -R engage:engage /engage
```

Example, compare `hadoop fs` command vs. using FUSE mount:

``` console
user$ hadoop fs -ls /engage
Found 3 items
-rw-rw-r--   2 engage engage  733669376 2011-06-15 16:55 /engage/CentOS-5.6-x86_64-LiveCD.iso
-rw-rw-r--   2 engage engage  215387183 2011-06-15 16:28 /engage/condor-7.6.1-x86_rhap_5-stripped.tar.gz
-rw-rw-r--   2 engage engage    9259360 2011-06-15 16:32 /engage/glideinWMS_v2_5_1.tgz

user$ ls -l /mnt/hadoop/engage
total 935855
-rw-rw-r-- 1 engage engage 733669376 Jun 15 16:55 CentOS-5.6-x86_64-LiveCD.iso
-rw-rw-r-- 1 engage engage 215387183 Jun 15 16:28 condor-7.6.1-x86_rhap_5-stripped.tar.gz
-rw-rw-r-- 1 engage engage   9259360 Jun 15 16:32 glideinWMS_v2_5_1.tgz
```

### GridFTP Validation ###

!!! note
    The commands used to verify GridFTP below assume you have access to a node where you can first generate a valid proxy using `voms-proxy-init` or `grid-proxy-init`. Obtaining grid credentials is beyond the scope of this document.

``` console
user$ globus-url-copy file:///home/users/jdost/test.txt gsiftp://devg-7.t2.ucsd.edu:2811/mnt/hadoop/engage/test.txt
```

If you are having troubles running GridFTP refer to [Starting GridFTP in Standalone Mode](#starting-gridftp-in-standalone-mode) in the Troubleshooting section.

Troubleshooting
---------------

### Hadoop ###

To view all of the currently configured settings of Hadoop from the web interface, enter the following url in your browser:

``` file
http://%RED%namenode.hostname%ENDCOLOR%:50070/conf
```

You will see the entire configuration in XML format, for example:

<details>
  <summary>Expand XML configuration</summary>
    <p>

``` file
<?xml version="1.0" encoding="UTF-8" standalone="no"?><configuration>
<property><!--Loaded from core-default.xml--><name>fs.s3n.impl</name><value>org.apache.hadoop.fs.s3native.NativeS3FileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.cache.levels</name><value>2</value></property>
<property><!--Loaded from mapred-default.xml--><name>map.sort.class</name><value>org.apache.hadoop.util.QuickSort</value></property>
<property><!--Loaded from core-site.xml--><name>hadoop.tmp.dir</name><value>/data1/hadoop//scratch</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.native.lib</name><value>true</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.namenode.decommission.nodes.per.interval</name><value>5</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.https.need.client.auth</name><value>false</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.client.idlethreshold</name><value>4000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.system.dir</name><value>${hadoop.tmp.dir}/mapred/system</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.data.dir.perm</name><value>755</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.persist.jobstatus.hours</name><value>0</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.namenode.logging.level</name><value>all</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.address</name><value>0.0.0.0:50010</value></property>
<property><!--Loaded from core-default.xml--><name>io.skip.checksum.errors</name><value>false</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.block.access.token.enable</name><value>false</value></property>
<property><!--Loaded from Unknown--><name>fs.default.name</name><value>hdfs://nagios.t2.ucsd.edu:9000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.child.tmp</name><value>./tmp</value></property>
<property><!--Loaded from core-default.xml--><name>fs.har.impl.disable.cache</name><value>true</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.skip.reduce.max.skip.groups</name><value>0</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.safemode.threshold.pct</name><value>0.999f</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.heartbeats.in.second</name><value>100</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.namenode.handler.count</name><value>40</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.blockreport.initialDelay</name><value>0</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.instrumentation</name><value>org.apache.hadoop.mapred.JobTrackerMetricsInst</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.dns.nameserver</name><value>default</value></property>
<property><!--Loaded from mapred-default.xml--><name>io.sort.factor</name><value>10</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.timeout</name><value>600000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.max.tracker.failures</name><value>4</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.rpc.socket.factory.class.default</name><value>org.apache.hadoop.net.StandardSocketFactory</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.jobhistory.lru.cache.size</name><value>5</value></property>
<property><!--Loaded from core-default.xml--><name>fs.hdfs.impl</name><value>org.apache.hadoop.hdfs.DistributedFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.skip.map.auto.incr.proc.count</name><value>true</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.block.access.key.update.interval</name><value>600</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.job.complete.cancel.delegation.tokens</name><value>true</value></property>
<property><!--Loaded from core-default.xml--><name>io.mapfile.bloom.size</name><value>1048576</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.reduce.shuffle.connect.timeout</name><value>180000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.safemode.extension</name><value>30000</value></property>
<property><!--Loaded from mapred-site.xml--><name>tasktracker.http.threads</name><value>50</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.shuffle.merge.percent</name><value>0.66</value></property>
<property><!--Loaded from core-default.xml--><name>fs.ftp.impl</name><value>org.apache.hadoop.fs.ftp.FTPFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.output.compress</name><value>false</value></property>
<property><!--Loaded from core-site.xml--><name>io.bytes.per.checksum</name><value>4096</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.healthChecker.script.timeout</name><value>600000</value></property>
<property><!--Loaded from core-default.xml--><name>topology.node.switch.mapping.impl</name><value>org.apache.hadoop.net.ScriptBasedMapping</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.https.server.keystore.resource</name><value>ssl-server.xml</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.reduce.slowstart.completed.maps</name><value>0.05</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.reduce.max.attempts</name><value>4</value></property>
<property><!--Loaded from core-default.xml--><name>fs.ramfs.impl</name><value>org.apache.hadoop.fs.InMemoryFileSystem</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.block.access.token.lifetime</name><value>600</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.skip.map.max.skip.records</name><value>0</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.name.edits.dir</name><value>${dfs.name.dir}</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.security.group.mapping</name><value>org.apache.hadoop.security.ShellBasedUnixGroupsMapping</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.persist.jobstatus.dir</name><value>/jobtracker/jobsInfo</value></property>
<property><!--Loaded from core-site.xml--><name>hadoop.log.dir</name><value>/var/log/hadoop</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3.buffer.dir</name><value>${hadoop.tmp.dir}/s3</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.block.size</name><value>134217728</value></property>
<property><!--Loaded from mapred-default.xml--><name>job.end.retry.attempts</name><value>0</value></property>
<property><!--Loaded from core-default.xml--><name>fs.file.impl</name><value>org.apache.hadoop.fs.LocalFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.output.compression.type</name><value>RECORD</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.local.dir.minspacestart</name><value>0</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.ipc.address</name><value>0.0.0.0:50020</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.permissions</name><value>true</value></property>
<property><!--Loaded from core-default.xml--><name>topology.script.number.args</name><value>100</value></property>
<property><!--Loaded from core-default.xml--><name>io.mapfile.bloom.error.rate</name><value>0.005</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.max.tracker.blacklists</name><value>4</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.profile.maps</name><value>0-2</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.https.address</name><value>0.0.0.0:50475</value></property>
<property><!--Loaded from core-site.xml--><name>dfs.umaskmode</name><value>002</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.userlog.retain.hours</name><value>24</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.secondary.http.address</name><value>gratia-1:50090</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.replication.max</name><value>32</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.persist.jobstatus.active</name><value>false</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.security.authorization</name><value>false</value></property>
<property><!--Loaded from core-default.xml--><name>local.cache.size</name><value>10737418240</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.min.split.size</name><value>0</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.namenode.delegation.token.renew-interval</name><value>86400000</value></property>
<property><!--Loaded from mapred-site.xml--><name>mapred.map.tasks</name><value>7919</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.child.java.opts</name><value>-Xmx200m</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.https.client.keystore.resource</name><value>ssl-client.xml</value></property>
<property><!--Loaded from Unknown--><name>dfs.namenode.startup</name><value>REGULAR</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.queue.name</name><value>default</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.retiredjobs.cache.size</name><value>1000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.https.address</name><value>0.0.0.0:50470</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.balance.bandwidthPerSec</name><value>2000000000</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.server.listen.queue.size</name><value>128</value></property>
<property><!--Loaded from mapred-default.xml--><name>job.end.retry.interval</name><value>30000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.inmem.merge.threshold</name><value>1000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.skip.attempts.to.start.skipping</name><value>2</value></property>
<property><!--Loaded from hdfs-site.xml--><name>fs.checkpoint.dir</name><value>/var/hadoop/checkpoint-a</value></property>
<property><!--Loaded from mapred-site.xml--><name>mapred.reduce.tasks</name><value>1543</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.merge.recordsBeforeProgress</name><value>10000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.userlog.limit.kb</name><value>0</value></property>
<property><!--Loaded from core-default.xml--><name>webinterface.private.actions</name><value>false</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.max.objects</name><value>0</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.shuffle.input.buffer.percent</name><value>0.70</value></property>
<property><!--Loaded from mapred-default.xml--><name>io.sort.spill.percent</name><value>0.80</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.map.tasks.speculative.execution</name><value>true</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.util.hash.type</name><value>murmur</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.dns.nameserver</name><value>default</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.blockreport.intervalMsec</name><value>3600000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.map.max.attempts</name><value>4</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.job.acl-view-job</name><value> </value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.handler.count</name><value>10</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.client.block.write.retries</name><value>3</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.max.reduces.per.node</name><value>-1</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.reduce.shuffle.read.timeout</name><value>180000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.expiry.interval</name><value>600000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.https.enable</name><value>false</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.maxtasks.per.job</name><value>-1</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.job.history.block.size</name><value>3145728</value></property>
<property><!--Loaded from mapred-default.xml--><name>keep.failed.task.files</name><value>false</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.failed.volumes.tolerated</name><value>0</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.profile.reduces</name><value>0-2</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.client.tcpnodelay</name><value>false</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.output.compression.codec</name><value>org.apache.hadoop.io.compress.DefaultCodec</value></property>
<property><!--Loaded from mapred-default.xml--><name>io.map.index.skip</name><value>0</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.server.tcpnodelay</name><value>false</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.namenode.delegation.key.update-interval</name><value>86400000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.running.map.limit</name><value>-1</value></property>
<property><!--Loaded from mapred-default.xml--><name>jobclient.progress.monitor.poll.interval</name><value>1000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.default.chunk.view.size</name><value>32768</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.logfile.size</name><value>10000000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.reduce.tasks.speculative.execution</name><value>true</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.tasktracker.outofband.heartbeat</name><value>false</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3n.block.size</name><value>67108864</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.datanode.du.reserved</name><value>10000000000</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.security.authentication</name><value>simple</value></property>
<property><!--Loaded from hdfs-site.xml--><name>fs.checkpoint.period</name><value>3600</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.running.reduce.limit</name><value>-1</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.reuse.jvm.num.tasks</name><value>1</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.web.ugi</name><value>webuser,webgroup</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.completeuserjobs.maximum</name><value>100</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.df.interval</name><value>60000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.tracker.task-controller</name><value>org.apache.hadoop.mapred.DefaultTaskController</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.data.dir</name><value>/data1/hadoop//data</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3.maxRetries</name><value>4</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.dns.interface</name><value>default</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.support.append</name><value>true</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.job.acl-modify-job</name><value> </value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.local.dir</name><value>${hadoop.tmp.dir}/mapred/local</value></property>
<property><!--Loaded from core-default.xml--><name>fs.hftp.impl</name><value>org.apache.hadoop.hdfs.HftpFileSystem</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.permissions.supergroup</name><value>root</value></property>
<property><!--Loaded from core-default.xml--><name>fs.trash.interval</name><value>0</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3.sleepTimeSeconds</name><value>10</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.submit.replication</name><value>10</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.replication.min</name><value>1</value></property>
<property><!--Loaded from core-default.xml--><name>fs.har.impl</name><value>org.apache.hadoop.fs.HarFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.map.output.compression.codec</name><value>org.apache.hadoop.io.compress.DefaultCodec</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.dns.interface</name><value>default</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.namenode.decommission.interval</name><value>30</value></property>
<property><!--Loaded from Unknown--><name>dfs.http.address</name><value>nagios:50070</value></property>
<property><!--Loaded from mapred-site.xml--><name>mapred.job.tracker</name><value>nagios:9000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.heartbeat.interval</name><value>3</value></property>
<property><!--Loaded from core-default.xml--><name>io.seqfile.sorter.recordlimit</name><value>1000000</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.name.dir</name><value>${hadoop.tmp.dir}/dfs/name</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.line.input.format.linespermap</name><value>1</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.taskScheduler</name><value>org.apache.hadoop.mapred.JobQueueTaskScheduler</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.instrumentation</name><value>org.apache.hadoop.mapred.TaskTrackerMetricsInst</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.datanode.http.address</name><value>0.0.0.0:50075</value></property>
<property><!--Loaded from mapred-default.xml--><name>jobclient.completion.poll.interval</name><value>5000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.max.maps.per.node</name><value>-1</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.local.dir.minspacekill</name><value>0</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.replication.interval</name><value>3</value></property>
<property><!--Loaded from mapred-default.xml--><name>io.sort.record.percent</name><value>0.05</value></property>
<property><!--Loaded from core-default.xml--><name>fs.kfs.impl</name><value>org.apache.hadoop.fs.kfs.KosmosFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.temp.dir</name><value>${hadoop.tmp.dir}/mapred/temp</value></property>
<property><!--Loaded from mapred-site.xml--><name>mapred.tasktracker.reduce.tasks.maximum</name><value>4</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.replication</name><value>2</value></property>
<property><!--Loaded from core-default.xml--><name>fs.checkpoint.edits.dir</name><value>${fs.checkpoint.dir}</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.tasks.sleeptime-before-sigkill</name><value>5000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.reduce.input.buffer.percent</name><value>0.0</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.indexcache.mb</name><value>10</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.job.split.metainfo.maxsize</name><value>10000000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.skip.reduce.auto.incr.proc.count</name><value>true</value></property>
<property><!--Loaded from core-default.xml--><name>hadoop.logfile.count</name><value>10</value></property>
<property><!--Loaded from core-default.xml--><name>fs.automatic.close</name><value>true</value></property>
<property><!--Loaded from core-default.xml--><name>io.seqfile.compress.blocksize</name><value>1000000</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.hosts.exclude</name><value>/etc/hadoop-0.20/conf/hosts_exclude</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3.block.size</name><value>67108864</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.tasktracker.taskmemorymanager.monitoring-interval</name><value>5000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.acls.enabled</name><value>false</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapreduce.jobtracker.staging.root.dir</name><value>${hadoop.tmp.dir}/mapred/staging</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.queue.names</name><value>default</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.access.time.precision</name><value>3600000</value></property>
<property><!--Loaded from core-default.xml--><name>fs.hsftp.impl</name><value>org.apache.hadoop.hdfs.HsftpFileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.tracker.http.address</name><value>0.0.0.0:50060</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.reduce.parallel.copies</name><value>5</value></property>
<property><!--Loaded from core-default.xml--><name>io.seqfile.lazydecompress</name><value>true</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.safemode.min.datanodes</name><value>0</value></property>
<property><!--Loaded from mapred-default.xml--><name>io.sort.mb</name><value>100</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.client.connection.maxidletime</name><value>10000</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.compress.map.output</name><value>false</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.tracker.report.address</name><value>127.0.0.1:0</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.healthChecker.interval</name><value>60000</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.client.kill.max</name><value>10</value></property>
<property><!--Loaded from core-default.xml--><name>ipc.client.connect.max.retries</name><value>10</value></property>
<property><!--Loaded from core-default.xml--><name>fs.s3.impl</name><value>org.apache.hadoop.fs.s3.S3FileSystem</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.job.tracker.http.address</name><value>0.0.0.0:50030</value></property>
<property><!--Loaded from core-default.xml--><name>io.file.buffer.size</name><value>4096</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.jobtracker.restart.recover</name><value>false</value></property>
<property><!--Loaded from core-default.xml--><name>io.serializations</name><value>org.apache.hadoop.io.serializer.WritableSerialization</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.task.profile</name><value>false</value></property>
<property><!--Loaded from hdfs-site.xml--><name>dfs.datanode.handler.count</name><value>10</value></property>
<property><!--Loaded from mapred-default.xml--><name>mapred.reduce.copy.backoff</name><value>300</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.replication.considerLoad</name><value>true</value></property>
<property><!--Loaded from mapred-default.xml--><name>jobclient.output.filter</name><value>FAILED</value></property>
<property><!--Loaded from hdfs-default.xml--><name>dfs.namenode.delegation.token.max-lifetime</name><value>604800000</value></property>
<property><!--Loaded from mapred-site.xml--><name>mapred.tasktracker.map.tasks.maximum</name><value>4</value></property>
<property><!--Loaded from core-default.xml--><name>io.compression.codecs</name><value>org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.BZip2Codec</value></property>
<property><!--Loaded from core-default.xml--><name>fs.checkpoint.size</name><value>67108864</value></property>
</configuration>
```
</p>
</details>

Please refer to the [Apache Hadoop FAQ webpage](http://wiki.apache.org/hadoop/FAQ) for answers to common questions/concerns

### FUSE ###

#### Notes on Building a FUSE Module ####

If you are running a custom kernel, then be sure to enable the `fuse` module with `CONFIG_FUSE_FS=m` in your kernel config. Building and installing a `fuse` kernel module for your custom kernel is beyond the scope of this document.

#### Running FUSE in Debug Mode ####

To start the FUSE mount in debug mode, you can run the FUSE mount command by hand:

``` console
root@host #  /usr/bin/hadoop-fuse-dfs  /mnt/hadoop -o rw,server=%RED%namenode.host%ENDCOLOR%,port=9000,rdbuffer=131072,allow_other -d
```

Debug output will be printed to stderr, which you will probably want to redirect to a file. Most FUSE-related problems can be tackled by reading through the stderr and looking for error messages.

#### GridFTP ####

#### Starting GridFTP in Standalone Mode ####

If you would like to test the gridftp-hdfs server in a debug standalone mode, you can run the command:

``` console
root@host # gridftp-hdfs-standalone
```

The standalone server runs on port 5002, handles a single GridFTP request, and will log output to stdout/stderr.

### File Locations ###


| Component | File Type                 | Location                                                              | Needs editing?                    |
|-----------|---------------------------|-----------------------------------------------------------------------|-----------------------------------|
| Hadoop    | Log files                 | `/var/log/hadoop/*`                                                   | No                                |
|           | PID files                 | `/var/run/hadoop/*.pid`                                               | No                                |
|           | init scripts              | `/etc/init.d/hadoop`                                                  | No                                |
|           | init script config file   | `/etc/sysconfig/hadoop`                                               | Yes                               |
|           | runtime config files      | `/etc/hadoop/conf/*`                                                  | Maybe                             |
|           | System binaries           | `/usr/bin/hadoop`                                                     | No                                |
|           | JARs                      | `/usr/lib/hadoop/*`                                                   | No                                |
|           | runtime config files      | `/etc/hosts_exclude`                                                  | Yes, must be present on NameNodes |
|           | Log files                 | `/var/log/gridftp-auth.log`, `/var/log/gridftp.log`                   | No                                |
| GridFTP   | Transfer log              | `/var/log/gridftp.log`                                                | No                                |
|           | Authentication log        | `/var/log/gridftp-auth.log`                                           | No                                |
|           | LCMAPS auth error log     | `/var/log/messages`                                                   | No                                |
|           | init.d script             | `/etc/init.d/globus-gridftp-server`                                   | No                                |
|           | runtime config files      | `/etc/gridftp-hdfs/*`, `/etc/sysconfig/gridftp-hdfs`                  | Maybe                             |
|           | System binaries           | `/usr/bin/gridftp-hdfs-standalone`, `/usr/sbin/globus-gridftp-server` | No                                |
|           | System libraries          | `/usr/lib64/libglobus_gridftp_server_hdfs.so*`                        | No                                |
|           | LCMAPS VOMS configuration | `/etc/lcmaps.db`                                                      | Yes                               |
|           | CA certificates           | `/etc/grid-security/certificates/*`                                   | No                                |

### Known Issues ###

#### Replicas ####

You may need to change the following line in `/usr/share/gridftp-hdfs/gridftp-hdfs-environment`:

``` file
export GRIDFTP_HDFS_REPLICAS=2
```

#### copyFromLocal java IOException ####

When trying to copy a local file into Hadoop you may come across the following java exception:

<details>
  <summary>Show detailed java exception</summary>
    <p>
``` console
11/06/24 11:10:50 WARN hdfs.DFSClient: Error Recovery for block null bad datanode[0]
nodes == null
11/06/24 11:10:50 WARN hdfs.DFSClient: Could not get block locations. Source file
"/osg/ddd" - Aborting...
copyFromLocal: java.io.IOException: File /osg/ddd could only be replicated to 0
nodes, instead of 1
11/06/24 11:10:50 ERROR hdfs.DFSClient: Exception closing file /osg/ddd :
org.apache.hadoop.ipc.RemoteException: java.io.IOException: File /osg/ddd could only
be replicated to 0 nodes, instead of 1
        at
org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:1415)
        at org.apache.hadoop.hdfs.server.namenode.NameNode.addBlock(NameNode.java:588)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
        at
sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
        at java.lang.reflect.Method.invoke(Method.java:597)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:528)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1319)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1315)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:396)
        at
org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1063)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:1313)
```
</p>
</details>

This can occur if you try to install a DataNode on a machine with less than 10GB of disk space available. This can be changed by lowering the value of the following property in `/usr/lib/hadoop-0.20/conf/hdfs-site.xml`:

``` file
<property>
  <name>dfs.datanode.du.reserved</name>
  <value>10000000000</value>
</property>
```

Hadoop always requires this amount of disk space to be available for non-hdfs usage on the machine.

Getting Help
------------

To get assistance, please use the [this page](/common/help).

References
----------

-   [Using Hadoop as a Grid Storage Element](http://www.iop.org/EJ/article/1742-6596/180/1/012047/jpconf9_180_012047.pdf), *Journal of Physics Conference Series, 2009*.
-   [Hadoop Distributed File System for the Grid](http://osg-docdb.opensciencegrid.org/0009/000911/001/Hadoop.pdf), *IEEE Nuclear Science Symposium, 2009*.

### Users ###

This installation will create following users unless they are already created.

| User        | Comment                                           |
|:------------|:--------------------------------------------------|
| `hadoop`    | Runs the NameNode services                        |
| `hdfs`      | Used by Hadoop to store data blocks and meta-data |
| `mapred`    |                                                   |
| `zookeeper` |                                                   |

For this package to function correctly, you will have to create the users needed for grid operation. Any user that can be authenticated should be created.

