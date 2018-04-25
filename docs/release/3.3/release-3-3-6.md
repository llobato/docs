OSG Software Release 3.3.6
==========================

**Release Date**: 2015-12-08

Summary of changes
------------------

This release contains:

-   Fixed a bug that prevented HTCondor 8.4's CREAM gahp from starting
-   CA certificates based on [IGTF 1.70](https://dist.eugridpma.info/distribution/igtf/current/CHANGES)
    -   Updated CRL URL hosted by KIT for ArmeSFO (AM)
    -   Added NorduGrid 2015 trust anchor (DK,NO,SE,FI,IS)
    -   Discontinued superseded DigiCertGridCA-1G2-Classic (US)
-   Restored the 'Sign AUP on behalf of user' feature in VOMS Admin Server

These [JIRA tickets](https://jira.opensciencegrid.org/issues/?jql=project%20%3D%20SOFTWARE%20AND%20fixVersion%20%3D%203.3.6%20ORDER%20BY%20priority%20DESC) were addressed in this release.

Detailed changes are below. All of the documentation can be found [here](/index.md)

Known Issues
------------

-   The `osg-cert-request` and `osg-gridadmin-cert-request` may return `CILogon 500` errors when retrieving certificates. To fix this, follow the below instructions:
    1.  Save the following text (including the empty line at the end) to `~/osgpkitools.patch`:  
    <pre class="file">Index: OSGPKIUtils.py

**`===============================================================`** --- OSGPKIUtils.py (revision 22192) +++ OSGPKIUtils.py (revision 22193) @@ -362,6 +362,7 @@ \#

self.X509Request.set\_pubkey(pkey=self.PKey) + self.X509Request.set\_version(0) self.X509Request.sign(pkey=self.PKey, md='sha1') return self.X509Request

</pre>

1.  `cd` into the appropriate folder:  
<pre class="rootscreen">\# For EL5 hosts:

root@host # cd /usr/lib/python2.4/site-packages/osgpkitools/ \# For EL6 hosts: root@host # cd /usr/lib/python2.6/site-packages/osgpkitools/</pre>

1.  Apply the patch:  
<pre class="rootscreen">root@host # patch < ~/osgpkitools.patch</pre> \* HTCondor 8.4.0 has changed it's behavior in ways that cause the GlideinWMS frontend configuration to break. In order to correct this, the following setting needs to be added to the configuration file:

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

-   [condor-8.4.2-1.2.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=condor-8.4.2-1.2.osg33.el6)
-   [gip-1.3.11-8.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=gip-1.3.11-8.osg33.el6)
-   [igtf-ca-certs-1.70-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=igtf-ca-certs-1.70-1.osg33.el6)
-   [osg-ca-certs-1.51-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-ca-certs-1.51-1.osg33.el6)
-   [osg-configure-1.2.5-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-configure-1.2.5-1.osg33.el6)
-   [osg-info-services-1.1.0-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-info-services-1.1.0-1.osg33.el6)
-   [osg-test-1.4.32-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-test-1.4.32-1.osg33.el6)
-   [osg-version-3.3.6-1.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-version-3.3.6-1.osg33.el6)
-   [voms-admin-server-2.7.0-1.17.osg33.el6](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=voms-admin-server-2.7.0-1.17.osg33.el6)

#### Enterprise Linux 7

-   [condor-8.4.2-1.2.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=condor-8.4.2-1.2.osg33.el7)
-   [gip-1.3.11-8.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=gip-1.3.11-8.osg33.el7)
-   [igtf-ca-certs-1.70-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=igtf-ca-certs-1.70-1.osg33.el7)
-   [osg-ca-certs-1.51-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-ca-certs-1.51-1.osg33.el7)
-   [osg-configure-1.2.5-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-configure-1.2.5-1.osg33.el7)
-   [osg-info-services-1.1.0-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-info-services-1.1.0-1.osg33.el7)
-   [osg-test-1.4.32-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-test-1.4.32-1.osg33.el7)
-   [osg-version-3.3.6-1.osg33.el7](https://koji-hub.batlab.org/koji/search?match=glob&type=build&terms=osg-version-3.3.6-1.osg33.el7)

### RPMs

If you wish to manually update your system, you can run yum update against the following packages:

    condor condor-all condor-bosco condor-classads condor-classads-devel condor-cream-gahp condor-debuginfo condor-kbdd condor-procd condor-python condor-std-universe condor-test condor-vm-gahp gip igtf-ca-certs osg-ca-certs osg-configure osg-configure-ce osg-configure-cemon osg-configure-condor osg-configure-gateway osg-configure-gip osg-configure-gratia osg-configure-infoservices osg-configure-lsf osg-configure-managedfork osg-configure-misc osg-configure-monalisa osg-configure-network osg-configure-pbs osg-configure-rsv osg-configure-sge osg-configure-slurm osg-configure-squid osg-configure-tests osg-info-services osg-test osg-version voms-admin-server

If you wish to only update the RPMs that changed, the set of RPMs is:

#### Enterprise Linux 6

``` file
condor-8.4.2-1.2.osg33.el6
condor-all-8.4.2-1.2.osg33.el6
condor-bosco-8.4.2-1.2.osg33.el6
condor-classads-8.4.2-1.2.osg33.el6
condor-classads-devel-8.4.2-1.2.osg33.el6
condor-cream-gahp-8.4.2-1.2.osg33.el6
condor-debuginfo-8.4.2-1.2.osg33.el6
condor-kbdd-8.4.2-1.2.osg33.el6
condor-procd-8.4.2-1.2.osg33.el6
condor-python-8.4.2-1.2.osg33.el6
condor-std-universe-8.4.2-1.2.osg33.el6
condor-test-8.4.2-1.2.osg33.el6
condor-vm-gahp-8.4.2-1.2.osg33.el6
gip-1.3.11-8.osg33.el6
igtf-ca-certs-1.70-1.osg33.el6
osg-ca-certs-1.51-1.osg33.el6
osg-configure-1.2.5-1.osg33.el6
osg-configure-ce-1.2.5-1.osg33.el6
osg-configure-cemon-1.2.5-1.osg33.el6
osg-configure-condor-1.2.5-1.osg33.el6
osg-configure-gateway-1.2.5-1.osg33.el6
osg-configure-gip-1.2.5-1.osg33.el6
osg-configure-gratia-1.2.5-1.osg33.el6
osg-configure-infoservices-1.2.5-1.osg33.el6
osg-configure-lsf-1.2.5-1.osg33.el6
osg-configure-managedfork-1.2.5-1.osg33.el6
osg-configure-misc-1.2.5-1.osg33.el6
osg-configure-monalisa-1.2.5-1.osg33.el6
osg-configure-network-1.2.5-1.osg33.el6
osg-configure-pbs-1.2.5-1.osg33.el6
osg-configure-rsv-1.2.5-1.osg33.el6
osg-configure-sge-1.2.5-1.osg33.el6
osg-configure-slurm-1.2.5-1.osg33.el6
osg-configure-squid-1.2.5-1.osg33.el6
osg-configure-tests-1.2.5-1.osg33.el6
osg-info-services-1.1.0-1.osg33.el6
osg-test-1.4.32-1.osg33.el6
osg-version-3.3.6-1.osg33.el6
voms-admin-server-2.7.0-1.17.osg33.el6
```

#### Enterprise Linux 7

``` file
condor-8.4.2-1.2.osg33.el7
condor-all-8.4.2-1.2.osg33.el7
condor-bosco-8.4.2-1.2.osg33.el7
condor-classads-8.4.2-1.2.osg33.el7
condor-classads-devel-8.4.2-1.2.osg33.el7
condor-debuginfo-8.4.2-1.2.osg33.el7
condor-kbdd-8.4.2-1.2.osg33.el7
condor-procd-8.4.2-1.2.osg33.el7
condor-python-8.4.2-1.2.osg33.el7
condor-test-8.4.2-1.2.osg33.el7
condor-vm-gahp-8.4.2-1.2.osg33.el7
gip-1.3.11-8.osg33.el7
igtf-ca-certs-1.70-1.osg33.el7
osg-ca-certs-1.51-1.osg33.el7
osg-configure-1.2.5-1.osg33.el7
osg-configure-ce-1.2.5-1.osg33.el7
osg-configure-cemon-1.2.5-1.osg33.el7
osg-configure-condor-1.2.5-1.osg33.el7
osg-configure-gateway-1.2.5-1.osg33.el7
osg-configure-gip-1.2.5-1.osg33.el7
osg-configure-gratia-1.2.5-1.osg33.el7
osg-configure-infoservices-1.2.5-1.osg33.el7
osg-configure-lsf-1.2.5-1.osg33.el7
osg-configure-managedfork-1.2.5-1.osg33.el7
osg-configure-misc-1.2.5-1.osg33.el7
osg-configure-monalisa-1.2.5-1.osg33.el7
osg-configure-network-1.2.5-1.osg33.el7
osg-configure-pbs-1.2.5-1.osg33.el7
osg-configure-rsv-1.2.5-1.osg33.el7
osg-configure-sge-1.2.5-1.osg33.el7
osg-configure-slurm-1.2.5-1.osg33.el7
osg-configure-squid-1.2.5-1.osg33.el7
osg-configure-tests-1.2.5-1.osg33.el7
osg-info-services-1.1.0-1.osg33.el7
osg-test-1.4.32-1.osg33.el7
osg-version-3.3.6-1.osg33.el7
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

