Installing the Worker Node Client From RPMs
===========================================

The **OSG Worker Node Client** is a collection of software components that is expected to be added to every worker node
that can run OSG jobs. It provides a common environment and a minimal set of common tools that all OSG jobs can expect
to use. Contents of the worker node client can be found [here](#worker-node-contents).

!!! note
    It is possible to install the Worker Node Client software in a variety of ways, depending on your local site:

    -   Install using RPMs and Yum (this guide) - useful when managing your worker nodes with a tool (e.g., Puppet, Chef) that can automate RPM installs
    -   [Use from OASIS](install-wn-oasis.md) - useful when worker nodes already mount [CVMFS](install-cvmfs)
    -   [Install using a tarball](install-wn-tarball.md) - useful when installing onto a shared filesystem for distribution to worker nodes

This document is intended to guide system administrators through the process of configuring a site to make the Worker
Node Client software available from an RPM.

Before Starting
---------------

As with all OSG software installations, there are some one-time (per host) steps to prepare in advance:

-   Ensure the host has [a supported operating system](/release/supported_platforms.md)
-   Obtain root access to the host
-   Prepare [the required Yum repositories](/common/yum.md)
-   Install [CA certificates](/common/ca.md)

Install the Worker Node Client
------------------------------

Install the Worker Node Client RPM:

    :::console
    root@host # yum install osg-wn-client


Services
--------

Fetch-CRL is the only service required to support the WN Client.


| Software  | Service name                          | Notes                                                                                  |
|:----------|:--------------------------------------|:---------------------------------------------------------------------------------------|
| Fetch CRL | `fetch-crl-boot` and `fetch-crl-cron` | See [CA documentation](/common/ca.md) for more info |

!!! note
    `fetch-crl-boot` will begin fetching CRLS, which can take a few minutes and fail on transient errors. You can add configuration to ignore these transient errors in `/etc/fetch-crl.conf`:

        :::file
        noerrors

As a reminder, here are common service commands (all run as `root`):


| To …                                        | Run the command …                     |
|:--------------------------------------------|:--------------------------------------|
| Start a service                             | `service SERVICE-NAME start` |
| Stop a service                              | `service SERVICE-NAME stop`  |
| Enable a service to start during boot       | `chkconfig SERVICE-NAME on`  |
| Disable a service from starting during boot | `chkconfig SERVICE-NAME off` |

Validating the Worker Node Client
-------------------------------

To verify functionality of the worker node client, you will need to submit a test job against your CE and verify the job's output.

1.  Submit a job that executes the `env` command (e.g. Run [condor\_ce\_trace](/compute-element/troubleshoot-htcondor-ce#condor_ce_trace) with the `-d` flag from your HTCondor CE)
2.  Verify that the value of `OSG_GRID` is set to `/etc/osg/wn-client`

How to get Help?
----------------

To get assistance please use this [Help Procedure](/common/help).


Reference
---------

Please see the documentation on using [Yum and RPM](/release/yum-basics.md), and using the [OSG Yum repositories](/common/yum.md).

### Worker node contents

The worker node may be updated from time to time. As of OSG 3.3.21 in February 2017, the OSG worker node client contains:

-   OSG Certificates
-   curl
-   Fetch CRL
-   FTS client
-   gfal2
-   globus-url-copy (GridFTP client)
-   globus-xio-udt-driver
-   ldapsearch
-   MyProxy
-   osg-system-profiler
-   UberFTP
-   vo-client (includes /etc/vomses file)
-   VOMS client
-   wget
-   xrdcp

To see the currently installed version of the worker node package, run the following command:

``` console
root@host # rpm -q --requires osg-wn-client
```

Click [here](/release/yum-basics.md) for more details on using RPM to see what was installed.
