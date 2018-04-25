OSG Software Release 3.3.32
===========================

**Release Date**: 2018-02-08

Summary of changes
------------------

This release contains:

-   Critical GlideinWMS bug fix for heavily loaded frontends (<https://cdcvs.fnal.gov/redmine/issues/18748>)

These [JIRA tickets](https://jira.opensciencegrid.org/issues/?jql=project%20%3D%20SOFTWARE%20AND%20fixVersion%20%3D%203.3.32%20ORDER%20BY%20priority%20DESC%2C%20key%20DESC) were addressed in this release.

Detailed changes are below. All of the documentation can be found [here](/index.md).

Updating to the new release
---------------------------

### Update Repositories

To update to this series, you need to [install the current OSG repositories](/common/yum#install-osg-repositories#updating-from-osg-31-32-33-to-33-or-34).

### Update Software

Once the repositories are installed, you can update to this new release with:

``` console
root@host # yum update
```

!!! note "Notes"
    -   Please be aware that running `yum update` may also update other RPMs. You can exclude packages from being updated using the `--exclude=[package-name or glob]` option for the `yum` command.
    -   Watch the yum update carefully for any messages about a `.rpmnew` file being created. That means that a configuration file had been edited, and a new default version was to be installed. In that case, RPM does not overwrite the edited configuration file but instead installs the new version with a `.rpmnew` extension. You will need to merge any edits that have made into the `.rpmnew` file and then move the merged version into place (that is, without the `.rpmnew` extension).

Need help?
----------

Do you need help with this release? [Contact us for help](/common/help).

Detailed changes in this release
--------------------------------

### Packages

We added or updated the following packages to the production OSG yum repository. Note that in some cases, there are multiple RPMs for each package. You can click on any given package to see the set of RPMs or see the complete list below.

#### Enterprise Linux 6

-   [glideinwms-3.2.20-2.osg33.el6](https://koji.chtc.wisc.edu/koji/search?match=glob&type=build&terms=glideinwms-3.2.20-2.osg33.el6)
-   [osg-version-3.3.32-1.osg33.el6](https://koji.chtc.wisc.edu/koji/search?match=glob&type=build&terms=osg-version-3.3.32-1.osg33.el6)

#### Enterprise Linux 7

-   [glideinwms-3.2.20-2.osg33.el7](https://koji.chtc.wisc.edu/koji/search?match=glob&type=build&terms=glideinwms-3.2.20-2.osg33.el7)
-   [osg-version-3.3.32-1.osg33.el7](https://koji.chtc.wisc.edu/koji/search?match=glob&type=build&terms=osg-version-3.3.32-1.osg33.el7)

### RPMs

If you wish to manually update your system, you can run yum update against the following packages:

    glideinwms-common-tools glideinwms-condor-common-config glideinwms-factory glideinwms-factory-condor glideinwms-glidecondor-tools glideinwms-libs glideinwms-minimal-condor glideinwms-usercollector glideinwms-userschedd glideinwms-vofrontend glideinwms-vofrontend-standalone igtf-ca-certs osg-ca-certs osg-version

If you wish to only update the RPMs that changed, the set of RPMs is:

#### Enterprise Linux 6

``` file
glideinwms-3.2.20-2.osg33.el6
glideinwms-common-tools-3.2.20-2.osg33.el6
glideinwms-condor-common-config-3.2.20-2.osg33.el6
glideinwms-factory-3.2.20-2.osg33.el6
glideinwms-factory-condor-3.2.20-2.osg33.el6
glideinwms-glidecondor-tools-3.2.20-2.osg33.el6
glideinwms-libs-3.2.20-2.osg33.el6
glideinwms-minimal-condor-3.2.20-2.osg33.el6
glideinwms-usercollector-3.2.20-2.osg33.el6
glideinwms-userschedd-3.2.20-2.osg33.el6
glideinwms-vofrontend-3.2.20-2.osg33.el6
glideinwms-vofrontend-standalone-3.2.20-2.osg33.el6
osg-version-3.3.32-1.osg33.el6
```

#### Enterprise Linux 7

``` file
glideinwms-3.2.20-2.osg33.el7
glideinwms-common-tools-3.2.20-2.osg33.el7
glideinwms-condor-common-config-3.2.20-2.osg33.el7
glideinwms-factory-3.2.20-2.osg33.el7
glideinwms-factory-condor-3.2.20-2.osg33.el7
glideinwms-glidecondor-tools-3.2.20-2.osg33.el7
glideinwms-libs-3.2.20-2.osg33.el7
glideinwms-minimal-condor-3.2.20-2.osg33.el7
glideinwms-usercollector-3.2.20-2.osg33.el7
glideinwms-userschedd-3.2.20-2.osg33.el7
glideinwms-vofrontend-3.2.20-2.osg33.el7
glideinwms-vofrontend-standalone-3.2.20-2.osg33.el7
osg-version-3.3.32-1.osg33.el7
```
