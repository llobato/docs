OSG Site Administrator Documentation
====================================

Welcome to the home of the Open Science Grid (OSG) Site Administrator documentation!  This documentation aims to
provide OSG site admins with the necessary information to install, configure, and operate site services.

If you are not a site adminstrator:

- If you are a **researcher** interested in using OSG resources, you may want to view our
  [user documentation](https://support.opensciencegrid.org/support/home).
- If you'd like to learn more about the OSG and our mission, visit the OSG consortium's [homepage](https://www.opensciencegrid.org/).

This document outlines the overall installation process for an OSG site and provides many links into detailed
installation, configuration, troubleshooting, and similar pages. If you do not see software-related technical
documentation listed here, try the search bar at the top or contacting us at
[help@opensciencegrid.org](mailto:help@opensciencegrid.org).

Plan the Site
-------------

If you have not done so already, [plan the overall architecture of your OSG site](site-planning). It is recommended that your plan be sufficiently detailed to include the OSG hosts that are needed and the main software components for each host. Be sure to consider [the operating systems that OSG supports](release/supported_platforms). For example, a basic site might include:

| Purpose              | Host                                | Major Software                                           |
|:---------------------|:------------------------------------|:---------------------------------------------------------|
| Compute Element (CE) | `osg-ce.example.edu`                | OSG CE, HTCondor Central Manager, etc. (`osg-ce-condor`) |
| Worker Nodes         | `wNNN.cluster.example.edu`          | OSG worker node client (`osg-wn-client`)                 |

Prepare the Batch System
------------------------

The assumption is that you have an existing batch system at your site. Currently, we support [HTCondor](http://research.cs.wisc.edu/htcondor/), [LSF](https://www.ibm.com/us-en/marketplace/hpc-workload-management), [PBS](http://www.pbsworks.com) and [TORQUE](http://www.adaptivecomputing.com/products/open-source/torque/), [SGE](http://en.wikipedia.org/wiki/Oracle_Grid_Engine), and [Slurm](http://slurm.schedmd.com) batch systems.

For smaller sites (less than 50 worker nodes), the most common way to add a site to OSG is to install the OSG Compute Element (CE) on the central host of your batch system.  At such a site - especially if you have minimal time to maintain a CE - you may want to contact <mailto:help@opensciencegrid.org> to ask about using an OSG-hosted CE instead of running your own.  Before proceeding with an install, be sure that you can submit and successfully run a job from your OSG CE host into your batch system.

Add OSG Software
----------------

If necessary, provision all OSG hosts that are in your site plan that do not exist yet.  The general steps to installing an OSG site are:

1. Install [OSG Yum Repos](release/yum-basics) and the [Compute Element software](#installing-and-configuring-the-compute-element) on your CE host
1. Install the [Worker Node client](#adding-osg-software-to-worker-nodes) on your worker nodes.
1. Install [optional software](#installing-and-configuring-other-services) to increase the capabilities of your site.
1. [Test the OSG installation](#test-osg-software).
1. Make your [site available to researchers](#start-glideinwms-pilot-submissions).
1. Verify your [site's accounting](#verify-reporting-and-monitoring).

!!! note
    For sites with more than a handful of worker nodes, it is recommended to use some sort of configuration management tool to install, configure, and maintain your site. While beyond the scope of OSG’s documentation to explain how to select and use such a system, some popular configuration management tools are [Puppet](http://puppetlabs.com), [Chef](https://www.chef.io), [Ansible](https://www.ansible.com), and [CFEngine](http://cfengine.com).

### General Installation Instructions ###

-   [Security information for OSG signed RPMs](release/signing)
-   [Using Yum and RPM](release/yum-basics)
-   [Install the OSG repositories](common/yum)
-   [OSG Software release series](release/release_series) - look here to upgrade between releases 3.1, 3.2, 3.3, and 3.4.

### Installing and Managing Certificates for Site Security ###

-   [Installing the grid certificate authorities (CAs)](common/ca)
-   [How do I get PKI host and service X.509 certificates?](security/host-certs)
-   [Automatically updating the grid certificate authorities (CAs)](security/certificate-management)
-   [OSG PKI command line client reference](security/certificate-management)

### Installing and Configuring the Compute Element ###

-   Install the compute element (HTCondor-CE and other software):
    -   [Overview and architecture](compute-element/htcondor-ce-overview)
    -   [Install HTCondor-CE](compute-element/install-htcondor-ce)
    -   [Configure the HTCondor-CE job router](compute-element/job-router-recipes), including common recipes
    -   [Troubleshooting HTCondor-CE installations](compute-element/troubleshoot-htcondor-ce)
    -   [Submitting jobs to HTCondor-CE](compute-element/submit-htcondor-ce)
-   [`osg-configure` Reference](other/configuration-with-osg-configure)

### Adding OSG Software to Worker Nodes ###

-   [Worker Node (WN) Client Overview](worker-node/using-wn)
-   Install the WN client software on every worker node – pick a method:
    -   [Using RPMs](worker-node/install-wn) – useful when managing your worker nodes with a tool (e.g., Puppet, Chef)
    -   [Using a tarball](worker-node/install-wn-tarball) – useful for installation onto a shared filesystem (does not require root access)
    -   [Using OASIS](worker-node/install-wn-oasis) – useful when [CVMFS](worker-node/install-cvmfs) is already mounted on your worker nodes
-   (optional) [Install the CernVM-FS client](worker-node/install-cvmfs) to make it easy for user jobs to use needed software from OSG's OASIS repositories
-   (optional) [Install singularity on the OSG worker node](worker-node/install-singularity), to allow pilot jobs to isolate user jobs.


### Installing and Configuring Other Services ###

All of these node types and their services are optional, although OSG requires an HTTP caching service if you have
installed [CVMFS](worker-node/install-cvmfs) on your worker nodes.

-   [Install Frontier Squid](data/frontier-squid), an HTTP caching proxy service.
-   Storage element:
    -   Existing POSIX-based systems (such as NFS, Lustre, or GPFS):
        -   [Install standalone OSG GridFTP](data/gridftp): GridFTP server
        -   (optional) [Install load-balanced OSG GridFTP](data/load-balanced-gridftp): when a single GridFTP server isn't enough
    -   Hadoop Distributed File System (HDFS):
        -   [Hadoop Overview](data/hadoop-overview): HDFS information, planning, and guides
    -   XRootD:
        -   [XRootd Overview](data/xrootd-overview): XRootD information, planning, and guides
        -   [Install XRootD Server](data/install-xrootd): XRootD redirector installation
-   RSV monitoring to monitor and report to OSG on the health of your site
    -   [Install RSV](monitoring/install-rsv)
-   [Install the GlideinWMS VO Frontend](other/install-gwms-frontend) if your want your users’ jobs to run on the OSG
    -   [Install the RSV GlideinWMS Tester](monitoring/install-rsv-gwms-tester) if you want to test your front-end's
        ability to submit jobs to sites in the OSG

Test OSG Software
-----------------

It is useful to test *manual* submission of jobs from inside and outside of your site through your CE to your batch system. If this process does not work manually, it will probably not work for the glideinWMS pilot factory either.

-   [Test job submission into an HTCondor-CE](compute-element/submit-htcondor-ce)

Start GlideinWMS Pilot Submissions
----------------------------------

To begin running [GlideinWMS](http://glideinwms.fnal.gov/) pilot jobs at your site, e-mail <osg-gfactory-support@physics.ucsd.edu> and tell them that you want to start accepting Glideins. Please provide them with the following information:

-   The fully qualified hostname of the CE
-   Resource/WLCG name
-   Supported OS version of your worker nodes (e.g., EL6, EL7, or both)
-   Support for multicore jobs
-   Maximum job walltime
-   Maximum job memory usage

Once the factory team has enough information, they will start submitting pilots from the test factory to your CE. Initially, this will be one pilot at a time but once the factory verifies that pilot jobs are running successfully, that number will be ramped up to 10, then 100.

Verify Reporting and Monitoring
-------------------------------

To verify that your site is correctly reporting to the OSG, check [OSG's Accounting Portal](https://gracc.opensciencegrid.org/dashboard/db/site-summary) for records of your site reports (select your site from the drop-down box). If you have enabled the OSG VO, you can also check <http://osg-flock.grid.iu.edu/monitoring/condor/sites/all_1day.html>.

Scale Up Site to Full Production
--------------------------------

After successfully running all the pilot jobs that are submitted by the test factory and verifying your site reports, your site will be deemed production ready. No action is required on your end, factory operations will start submitting pilot jobs from the production factory.
