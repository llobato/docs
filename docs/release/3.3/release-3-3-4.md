OSG Software Release 3.3.4
==========================

**Release Date**: 2015-11-10

Summary of changes
------------------

This release builds on the special OSG CA certificate release of November 3rd by making it easier to update from separate installations of the main CA certificate bundle and the new CILogon OSG CA. For hosts with only the main CA certificate bundle, there is no change.

This release contains:

-   Updated CVMFS configuration to support approved repositories from any domain
-   Fixed osg-ca-certs-updater to work when no "compat" packages are present
-   Added a new LCMAPS plug-in for process tracking
-   Added a new RSV PerfSONAR probe
-   Added a new RSV probe to check StashCache cache collector ads at the GOC
-   EL 7: Fixed Frontier Squid to start up properly after a reboot
-   EL 7: Fixed the MyProxy service to start up properly

These [JIRA tickets](https://jira.opensciencegrid.org/issues/?jql=project%20%3D%20SOFTWARE%20AND%20fixVersion%20%3D%203.3.4%20ORDER%20BY%20priority%20DESC) were addressed in this release.

Detailed changes are below. All of the documentation can be found [here](/index.md)

Known Issues
------------

-   CILogon CA certificate files have been reorganized between our various CA certificate packages.  
To avoid file conflicts, you should upgrade your CA certificate RPMs at the same time, such as via the following command:  
<pre class="rootscreen">root@host # yum update '\*-ca-cert\*'</pre>
-   When using osg-configure to configure a CE host, it will fail because it tries to contact a ReSS server that has been shut down permanently. The ReSS service has been deprecated since early 2014, and support for it will be removed from osg-configure in an upcoming version. To work around this osg-configure failure now, edit /etc/osg/config.d/30-infoservices.ini and set the option:

        :::file
        ress_servers = https://localhost[RAW]

-   There is a known memory leak in lcmaps-plugins-scas-client that HTCondor-CE triggers repeatedly. A special OSG release, coming soon, will fix the underlying memory leak, but in the meantime it is possible to make two small configuration changes to slow the rate at which memory leaks.
    1.  In /etc/condor-ce/config.d/01-ce-auth.conf, make sure the following attribute is defined as follows (note the $(USERS) at the end):

            :::file
            COLLECTOR.ALLOW_ADVERTISE_STARTD =  $(UNMAPPED_USERS), $(USERS)

1.  Also in /etc/condor-ce/config.d/01-ce-auth.conf, add the following line to the end of the file:

        ::: file
        GSS_ASSIST_GRIDMAP_CACHE_EXPIRATION = 30*$(MINUTE)

1.  Verify that the attributes are set properly by running the following command:

        ::: file
        $ condor_ce_config_val COLLECTOR.ALLOW_ADVERTISE_STARTD GSS_ASSIST_GRIDMAP_CACHE_EXPIRATION
        *@unmapped.opensciencegrid.org, *@users.opensciencegrid.org
        30*60

-   HTCondor 8.4.0 has changed it's behavior in ways that cause the GlideinWMS frontend configuration to break. In order to correct this, the following setting needs to be added to the configuration file:

        :::file
        COLLECTOR_USES_SHARED_PORT = False

-   StashCache packages need to be manually configured
    -   Manual configuration for origin server
        -   Assuming that the origin server connects only to a redirector (not directly to cache server), minimal xrootd configuration is required. The configuration file, /etc/xrootd/xrootd-stashcache-origin-server.cfg, in this release is overkill. Here are recommended settings to use:

                :::file
                xrd.port 1094
                all.role server
                all.manager stash-redirector.example.com 1213
                all.export / nostage
                xrootd.trace emsg login stall redirect
                ofs.trace none
                xrd.trace conn
                cms.trace all
                sec.protocol  host
                sec.protbind  * none
                all.adminpath /var/run/xrootd
                all.pidpath /var/run/xrootd

-   Manual configuration for cache server
    -   In contrast to the origin server configuration, one needs to declare `pss.origin <stash-redirector.example.com>` instead of configuring the cmsd or manager (only the xrootd daemon is required on the cache server). More detailed configuration of cache server for StashCache is [here](http://opensciencegrid.github.io/StashCache/admin/configure-cache/).
-   In both cases, administrator needs to set the path of custom configuration file for its xrootd/cmds instance in /etc/sysconfig/xrootd, For example, change the cmds default from:

        :::file
        CMSD_DEFAULT_OPTIONS="-l /var/log/xrootd/cmsd.log -c /etc/xrootd/xrootd-clustered.cfg -k fifo"
<br/>

    to

        :::file
        CMSD_DEFAULT_OPTIONS="-l /var/log/xrootd/cmsd.log -c /etc/xrootd/xrootd-stashcache-origin-server.marian -k fifo" 

Updating to the new release
---------------------------

### Update Repositories

To update to this series, you need to [install the current OSG repositories](/common/yum#install-osg-repositories).

### Update Software

Once the new repositories are installed, you can update to this new release with:

``` console
root@host # yum update
```

!!! note "Notes"
    -   Please be aware that running `yum update` may also update other RPMs. You can exclude packages from being updated using the `--exclude=[package-name or glob]` option for the `yum` command.
    -   Watch the yum update carefully for any messages about a `.rpmnew` file being created. That means that a configuration file had been edited, and a new default version was to be installed. In that case, RPM does not overwrite the edited configuration file but instead installs the new version with a `.rpmnew` extension. You will need to merge any edits that have made into the `.rpmnew` file and then move the merged version into place (that is, without the `.rpmnew` extension). Watch especially for `/etc/lcmaps.db`, which every site is expected to edit.

Need help?
----------

Do you need help with this release? [Contact us for help](/common/help).

Detailed changes in this release
--------------------------------

### Packages

We added or updated the following packages to the production OSG yum repository. Note that in some cases, there are multiple RPMs for each package. You can click on any given package to see the set of RPMs or see the complete list below.

#### Enterprise Linux 6

-   [cilogon-openid-ca-cert-1.1-3.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cilogon-openid-ca-cert-1.1-3.osg33.el6)
-   [cilogon-osg-ca-cert-1.0-2.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cilogon-osg-ca-cert-1.0-2.osg33.el6)
-   [cvmfs-config-osg-1.1-8.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cvmfs-config-osg-1.1-8.osg33.el6)
-   [frontier-squid-2.7.STABLE9-24.2.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=frontier-squid-2.7.STABLE9-24.2.osg33.el6)
-   [jglobus-2.1.0-6.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=jglobus-2.1.0-6.osg33.el6)
-   [lcmaps-plugins-process-tracking-0.3-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=lcmaps-plugins-process-tracking-0.3-1.osg33.el6)
-   [myproxy-6.1.12-1.1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=myproxy-6.1.12-1.1.osg33.el6)
-   [osg-ca-certs-updater-1.3-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-ca-certs-updater-1.3-1.osg33.el6)
-   [osg-oasis-5-3.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-oasis-5-3.osg33.el6)
-   [osg-test-1.4.31-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-test-1.4.31-1.osg33.el6)
-   [osg-version-3.3.4-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-version-3.3.4-1.osg33.el6)
-   [rsv-3.12.0-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=rsv-3.12.0-1.osg33.el6)
-   [rsv-perfsonar-1.1.1-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=rsv-perfsonar-1.1.1-1.osg33.el6)

#### Enterprise Linux 7

-   [cilogon-openid-ca-cert-1.1-3.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cilogon-openid-ca-cert-1.1-3.osg33.el7)
-   [cilogon-osg-ca-cert-1.0-2.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cilogon-osg-ca-cert-1.0-2.osg33.el7)
-   [cvmfs-config-osg-1.1-8.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=cvmfs-config-osg-1.1-8.osg33.el7)
-   [frontier-squid-2.7.STABLE9-24.2.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=frontier-squid-2.7.STABLE9-24.2.osg33.el7)
-   [jglobus-2.1.0-6.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=jglobus-2.1.0-6.osg33.el7)
-   [lcmaps-plugins-process-tracking-0.3-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=lcmaps-plugins-process-tracking-0.3-1.osg33.el7)
-   [myproxy-6.1.12-1.1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=myproxy-6.1.12-1.1.osg33.el7)
-   [osg-ca-certs-updater-1.3-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-ca-certs-updater-1.3-1.osg33.el7)
-   [osg-oasis-5-3.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-oasis-5-3.osg33.el7)
-   [osg-test-1.4.31-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-test-1.4.31-1.osg33.el7)
-   [osg-version-3.3.4-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-version-3.3.4-1.osg33.el7)
-   [rsv-3.12.0-1\_clipped.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=rsv-3.12.0-1_clipped.osg33.el7)
-   [rsv-perfsonar-1.1.1-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=rsv-perfsonar-1.1.1-1.osg33.el7)

### RPMs

If you wish to manually update your system, you can run yum update against the following packages:

    cilogon-openid-ca-cert cilogon-osg-ca-cert cvmfs-config-osg frontier-squid frontier-squid-debuginfo jglobus lcmaps-plugins-process-tracking lcmaps-plugins-process-tracking-debuginfo myproxy myproxy-admin myproxy-debuginfo myproxy-devel myproxy-doc myproxy-libs myproxy-server myproxy-voms osg-ca-certs-updater osg-oasis osg-test osg-version rsv rsv-consumers rsv-core rsv-metrics rsv-perfsonar

If you wish to only update the RPMs that changed, the set of RPMs is:

#### Enterprise Linux 6

``` file
cilogon-openid-ca-cert-1.1-3.osg33.el6
cilogon-osg-ca-cert-1.0-2.osg33.el6
cvmfs-config-osg-1.1-8.osg33.el6
frontier-squid-2.7.STABLE9-24.2.osg33.el6
frontier-squid-debuginfo-2.7.STABLE9-24.2.osg33.el6
jglobus-2.1.0-6.osg33.el6
lcmaps-plugins-process-tracking-0.3-1.osg33.el6
lcmaps-plugins-process-tracking-debuginfo-0.3-1.osg33.el6
myproxy-6.1.12-1.1.osg33.el6
myproxy-admin-6.1.12-1.1.osg33.el6
myproxy-debuginfo-6.1.12-1.1.osg33.el6
myproxy-devel-6.1.12-1.1.osg33.el6
myproxy-doc-6.1.12-1.1.osg33.el6
myproxy-libs-6.1.12-1.1.osg33.el6
myproxy-server-6.1.12-1.1.osg33.el6
myproxy-voms-6.1.12-1.1.osg33.el6
osg-ca-certs-updater-1.3-1.osg33.el6
osg-oasis-5-3.osg33.el6
osg-test-1.4.31-1.osg33.el6
osg-version-3.3.4-1.osg33.el6
rsv-3.12.0-1.osg33.el6
rsv-consumers-3.12.0-1.osg33.el6
rsv-core-3.12.0-1.osg33.el6
rsv-metrics-3.12.0-1.osg33.el6
rsv-perfsonar-1.1.1-1.osg33.el6
```

#### Enterprise Linux 7

``` file
cilogon-openid-ca-cert-1.1-3.osg33.el7
cilogon-osg-ca-cert-1.0-2.osg33.el7
cvmfs-config-osg-1.1-8.osg33.el7
frontier-squid-2.7.STABLE9-24.2.osg33.el7
frontier-squid-debuginfo-2.7.STABLE9-24.2.osg33.el7
jglobus-2.1.0-6.osg33.el7
lcmaps-plugins-process-tracking-0.3-1.osg33.el7
lcmaps-plugins-process-tracking-debuginfo-0.3-1.osg33.el7
myproxy-6.1.12-1.1.osg33.el7
myproxy-admin-6.1.12-1.1.osg33.el7
myproxy-debuginfo-6.1.12-1.1.osg33.el7
myproxy-devel-6.1.12-1.1.osg33.el7
myproxy-doc-6.1.12-1.1.osg33.el7
myproxy-libs-6.1.12-1.1.osg33.el7
myproxy-server-6.1.12-1.1.osg33.el7
myproxy-voms-6.1.12-1.1.osg33.el7
osg-ca-certs-updater-1.3-1.osg33.el7
osg-oasis-5-3.osg33.el7
osg-test-1.4.31-1.osg33.el7
osg-version-3.3.4-1.osg33.el7
rsv-3.12.0-1_clipped.osg33.el7
rsv-consumers-3.12.0-1_clipped.osg33.el7
rsv-core-3.12.0-1_clipped.osg33.el7
rsv-metrics-3.12.0-1_clipped.osg33.el7
rsv-perfsonar-1.1.1-1.osg33.el7
```

### Upcoming Packages

We added or updated the following packages to the **upcoming** OSG yum repository. Note that in some cases, there are multiple RPMs for each package. You can click on any given package to see the set of RPMs or see the complete list below.

#### Enterprise Linux 6

#### Enterprise Linux 7

### Upcoming RPMs

If you wish to manually update your system, you can run yum update against the following packages:

If you wish to only update the RPMs that changed, the set of RPMs is:

#### Enterprise Linux 6

``` file
```

#### Enterprise Linux 7

``` file
```

