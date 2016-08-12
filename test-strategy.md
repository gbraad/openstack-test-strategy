Testing strategy
================


* Ceph storage testing
    * hardware validation tests
    * software validation tests
        * **Precondition**: stable software release available
* OpenStack software tests
    * single-node tests
        * **Preconditon**: stable release available
        * PackStack deployment, scenario-001
    * multi-node tests
        * **Preconditon**: single-node tests passed
        * deployment using scripts of control, compute, network; basic tempest test
    * single-node tests + ceph
        * **Preconditon**: after validation passed on Ceph and single-node tests
        * deployment of all-in-one targeting ceph storage backend
    * mult-node tests + ceph
        * **Preconditon**: after validation and multi-node passed


**Note**: _Testing is not a separate phase, but a continuous process. Some of the
things detailed here can be performed as an initial phase, but tests will have to
be performed automated and continually_


Testing phase detailed description
----------------------------------


### Ceph storage testing
Test ceph deployment using the latest packages in a NON-OpenStack environment and test
if storage works as expected. This means running performance tests, reliability tests,
etc.

This would have to be divided in several distinct tests groups
   
* hardware
* software
* integration


#### Hardware validation tests
The initial test for hardware is performed as a pre-flight (validation) check. Check
if the hardware is capable of running Ceph for storage purposes.


#### Software validation tests
**Precondition**  
These should only be performed on stable releases from the Ceph project. Therefore,
software releases during testing are not considered for this 'Testing phase'. The
stable releases of Ceph have passed the tests that are defined by the Ceph project.

The initial test for software can be perfomed in an isolated environment, such as VM
instances, in which a storage cluster is defined as minimal setup. Basic tests can be
tested against it.

Since tests at the Ceph project have been performed before, it can be considered a
test that also targets our deployment tools. However, this is not the aim of the
storage testing phase.

**Outcome**  
After these tests have passed, the release can be considered for the OpenStack
integration tests where Ceph is used as the storage backend.


#### Integration tests
**Precondition**   
If the software passes these tests, it can be deployed on physical hardware that has
passed the validation check. Again, the basic set of tests need to be run here. Only
if this task passes, more advanced tests can be considered.

In this phase deployment tools are used to prepare the physical hardware to run the
Ceph storage backend.

**Outcome**  
Only when all these tests have passed, Ceph can be deployed in a datacenter


### Single-node OpenStack testing
**Precondition**  
A stable release of OpenStack needs to be available.

Deployment of a single node test, where controller, network and compute are all hosted
on the same machine. This can be done using eg. PackStack, targeting the internal
package repository and running Tempest tests against this node.
   
WeIRDO can be used to run against a target/localhost: it will run `packstack` with a
Tempest test set.

The reason why we test a stable marked releases is that in previous releases of RDO,
services and packages were delivered in isolation and weren't tested as integration.
This also allows us to accomodate our release to have customized packages, such as
patches or repcached.

**Outcome**  
Usable release of OpenStack packages.


### Minimal Multi-node OpenStack testing
**Precondition**  
OpenStack packages have passed the single-node testing.


Deployment of a minimal environment using Controller, Network node and Compute, using the
latest packages to test general functionality; can a VM instance be created? If network
working as expected? Most of these tests can be handled by running Tempest against this
setup

**Outcome**  
Usable release that allows communication between nodes, for control and network.


### Minimal OpenStack + storage testing
**Precondition**

* Multi-node tests have passed
* Ceph storage tests have passed

Deployment of the minimal OpenStack environment, where the Ceph storage backend is used.
This means that six nodes are at least needed to perform these tests.

* Tempest needs to be run against this environment. Followed by possible performance tests.
* Explorative Testing and a full test scenario;
    * can a multi-tier application be deployed?  
      https://github.com/gbraad/openstack-handsonlabs/blob/master/neutron/building-multitier-application.md  
      This can help with assessing if security groups and networking works as expected.

**Outcome**  
Usable release of OpenStack that can be used for deployment. It allows proper communication between nodes and
storage, and is therefore usable for scale-out.


### OpenStack HA tests
Deployment of an OpenStack environment with the focus on High Availability of the Control
node. This is a minimal setup in which the control node needs to be deployed with HA
in mind. This would mean that a minimum of three control nodes is needed. Compute and
network can be offered in this situation as a single node solution. A set of tests
is needed to assess if this environment works as expected; what if node 1 goes down? What
if 2 nodes are down? etc.


### OpenStack HA + storage testing
As above, but including Ceph as the storage backend to make sure integration of these parts
work as expected. This can also include performance tests.


### Full HA tests
Dependent of the needs, a HA environment needs to be tests in which also the compute and
network nodes offer HA. This is very dependent of the needs of the customer, as live
migration/spinup of nodes is only possible if Storage is also considered.
