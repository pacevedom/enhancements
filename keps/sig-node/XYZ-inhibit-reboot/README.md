<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-XYZ: Inhibit reboots under labeled leases

<!--
This is the title of your KEP. Keep it short, simple, and descriptive. A good
title can help communicate what the KEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
      - [Prerequisite testing updates](#prerequisite-testing-updates)
      - [Unit tests](#unit-tests)
      - [Integration tests](#integration-tests)
      - [e2e tests](#e2e-tests)
  - [Graduation Criteria](#graduation-criteria)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (Optional)](#infrastructure-needed-optional)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [ ] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [ ] (R) KEP approvers have approved the KEP status as `implementable`
- [ ] (R) Design details are appropriately documented
- [ ] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input (including test refactors)
  - [ ] e2e Tests for all Beta API Operations (endpoints)
  - [ ] (R) Ensure GA e2e tests for meet requirements for [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
  - [ ] (R) Minimum Two Week Window for GA e2e tests to prove flake free
- [ ] (R) Graduation criteria is in place
  - [ ] (R) [all GA Endpoints](https://github.com/kubernetes/community/pull/1806) must be hit by [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
- [ ] (R) Production readiness review completed
- [ ] (R) Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary
Kubelet should be able to block shutdown under special conditions.

## Motivation
Kubernetes 1.21 introduced graceful node shutdown (#2000), by which kubelet is now able to detect shutdown commands by means of dbus and delay them using [systemd inhibitors](https://www.freedesktop.org/wiki/Software/systemd/inhibit/). Once a shutdown operation has been issued, it will happen sooner or later, it can not be stopped.

Some workloads, however, may not know in advance how much time it will take to fulfill their operations before requiring a reboot/shutdown. For example, picture a workload flashing a hardware device. This process must not be interrupted, and we don't know in advance how much time it will take to complete. Setting a large timeout for delaying shutdown does not help, since we still have the graceful termination period.</br>
If more than one workload needs to perform uninterruptible tasks this is also an issue, as one of them might finish before the others have done so.</br>
Setting PDBs might be troublesome, as some of these workloads might be DaemonSets.</br>
Other tasks, such as performing maintenance operations on a certain node might need to block a shutdown until they are done, and may not even be workloads.

We need a way to block shutdown entirely during a certain time window, which can also be seen as a non-interruptible operation. Kubelet is already using systemd inhibitors for delaying shutdown, but those also provide a block lock instead of just delaying. We present here a way to temporarily block shutdown in a node by using already existing resources with minimal impact in the kubelet.

### Goals
 * Allow shutdown block temporarily in order to avoid unintended disruptions.
 * Handle block locks with as few permissions as possible.
 * Allow several applications to place inhibitors on the same node.

### Non-Goals
 * Completely disable the possibility of shutting down a node.
 * Provide guarantee over shutdown lock, non-graceful, undetected shutdowns or overrides can't be inhibited.
 * Support non systemd inits.

## Proposal
For additional background on systemd-inhibitors please check this [link](https://www.freedesktop.org/wiki/Software/systemd/inhibit/) and also the [graceful node shutdown KEP](https://github.com/kubernetes/enhancements/issues/2000).

This KEP relies on work done in existing features, such as [work in dbus from KEP 2000](https://github.com/kubernetes/enhancements/issues/2000) and [leases in coordination API](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/lease-v1/).</br>
When enabling graceful node shutdown kubelet traces and monitors dbus to get shutdown events, plus creating systemd inhibitors delaying shutdown. Upon reboot, the inhibitors are disabled as they are volatile.</br>
[Inhibitor locks](https://www.freedesktop.org/wiki/Software/systemd/inhibit/) have two different modes: _delay_ and _block_. [Graceful node shutdown]((https://github.com/kubernetes/enhancements/issues/2000)) relies on _delay_, while this KEP relies on _block_. _block_ inhibits operations entirely until the lock is released. If such a lock is taken the operation will fail, while still being overrideable by privileged users or specific ignore options when issuing the command.

Since access support to dbus and systemd inhibitors is guaranteed since [KEP 2000](https://github.com/kubernetes/enhancements/issues/2000), there is a need for workloads (or users) to signal the intention to block shutdowns to perform uninterruptible operations.</br>

By using `leases` from `coordination.k8s.io` API, a workload/user should be able to express an intention to place a temporary lock that kubelet can read and act upon. When kubelet finds a lease in any namespace (except for `kube-node-lease`) with the node's name in it, it will create a _block_ inhibitor.</br>
When the `lease` does not have a `holderIdentity` or is deleted, kubelet can remove the _block_ inhibitor.

#### Using leases
In order to create _block_ inhibitors `lease` objects will be used. A naming strategy must be followed:
* `lease` object needs to share the same name of the node in which a _block_ inhibitor should be created.
* `lease` object may be created in any namespace.
* Namespace `kube-node-lease` is restricted, as it is used for efficient heartbeating. These follow the same strategy with the names.

Kubelet will then list and watch all `lease` objects in any namespace but `kube-node-lease` to react upon any changes. This requires extra permissions for kubelet:

```yaml
  - apiGroups:
    - coordination.k8s.io
    resources:
    - leases
    verbs:
    - create
    - delete
    - get
    - patch
    - update
    - list
    - watch
```

Once kubelet is able to list and watch `lease` objects, it must subscribe to them and wait for changes. In order to have a homogeneous treatment of them, the following basic rules are established:
* `metadata.name` must match the node's name in which kubelet is running. This means there is only one active `lease` per node.
* `metadata.namespace` must not be `kube-node-lease`.
* An empty `spec.holderIdentity` means the `lease` is not owned and should therefore remove any _block_ inhibitors.
* An empty `spec.acquireTime` should be regarded as incomplete, as there is no other way to determine when the `lease` was acquired.
* The rest of the fields are ignored by kubelet, as we will see later.

Kubelet only cares about `lease` objects being created (or acquired) and deleted, there can be no assumptions made on renewal times and/or failure to do so.

On the client side we can leverage on the [leader election](https://pkg.go.dev/k8s.io/client-go/tools/leaderelection) mechanism to handle the `lease` objects without direct interaction. Everytime we need to perform a critical task we may just create a leader election object and run it until completion.

Since there can only be one active `lease` per node, this means that once the `lease` is either deleted or loses the `spec.holderIdentity` value, the _block_ shutdown inhibitor must be released.

#### Node conditions and events
`lease` objects only have a _spec_ section, which means they need to be mutated in order to have a two way communication between who creates the `lease` and who acts on it. An application/user placing a `lease` does ont know whether the shutdown has been blocked or not. Any way of trying to communicate this fact through the `lease` itself is an abuse, therefore a new node condition is proposed:
```json
{
  "lastHeartbeatTime": "2022-07-19T08:24:22Z",
  "lastTransitionTime": "2022-07-18T11:17:59Z",
  "message": "kubelet allows shutdown node shutdown",
  "reason": "KubeletShutdownAllowed",
  "status": "False",
  "type": "ShutdownInhibited"
}
```

Condition is of type `ShutdownInhibited`, and shows whether a node can reboot or not. By doing this, an application knows if it is safe to perform a critical operation by watching node status. This condition is updated based on the number of active `lease` objects on a specific node.

When shutdown is inhibited we will see the node condition change:
```json
{
  "lastHeartbeatTime": "2022-07-20T06:31:44Z",
  "lastTransitionTime": "2022-07-20T06:31:44Z",
  "message": "kubelet inhibited node shutdown",
  "reason": "default/reboot-example",
  "status": "True",
  "type": "ShutdownInhibited"
}
```
The reason shows the `namespace/holderIdentity` for the lease owner, in case it is active. This is the way for workloads to know who is holding the current inhibitor to safely perform any operation.

#### Stacking leases
There can only be one `lease` object per node, as it uses the same name as the node. However, `lease` objects are namespaced and we can take advantage of that. A single namespace can hold one `lease` per node in the cluster, and we can have several `lease` objects with the same name accross different namespaces.</br>
When this happens, kubelet will store this information and keep the inhibitor until all the `leases` have either lost the holder or been deleted accross all namespaces. 

By using local `leases` (to the namespace) any application can handle their own objects with little additional permissions to their own namespace. Different applications may not tamper with other `lease` objects that dont belong to them.

#### PDB considerations
Pod disruption budgets are used to configure the limits of voluntary disruptions(such as rebooting) to applications. When draining a node, if any change in the status would violate a PDB the operation is blocked until conditions can be met.

We could configure a PDB for a workload in order to block a drain, and therefore a controlled shutdown. This is not enough to cover all use cases, such as daemon sets or trying to block shutdown in a node where we want to perform maintenance operations. The same would happen to single node clusters, where there is no drain operation, the reboot is issued without notice.

Draining a node might also wipe out dependencies for a workload, such as pods providing key functions for our application to run. If we don't have PDBs for those too our workload might stop working properly, rendering the PDB useless.

Relying entirely on PDBs might get the cluster in an infinite block from rebooting, since any workload would be able to configure their own PDBs without further control/inspection of the current cluster status.

#### Ignoring lease renewal
The nature of the applications/tasks that need to inhibit reboots is a bit special. To illustrate this, we could see an example. Imagine we have a cluster where there is a policy of rebooting nodes every 24h. If a workload inhibits reboot for a long period of time (over 24h), we may have two possible situations:
* The workload has not finished yet for whatever reason. We can not remove the inhibitor, as the workload is not yet finished and it could have serious consequences.
* The workload has finished but is stuck and did not command removing the inhibitor. The platform should remove it, as it has been held artificially and stopping the node from rebooting.

We can see these two situations are antagonist and indistinguishable from the platform's point of view. There is only one outcome for this, which is to alert the user to take action.

Due to the description above, the renewal of a `lease` is also pointless. Inhibitors only care about creation and removal, and we see that removal must be explicitly commanded. An update to the `renewTime` of a `lease` would mean the workload is still working, but since there is no automatic removal of the inhibitor we can not take any action on a lack of renewals.

To determine when to alert users we need a timeout for a `lease` being held too long. A new configuration parameter in the kubelet stating how long it should wait before signaling an alert will be created, `shutdownInhibitorAlertTimeout`.

In order to control when a `lease` has been acquired the kubelet will use the `spec.acquireTime` field. Everytime an application acquires a `lease`, this field shall be updated. Kubelet will start a timer using this field, expiring `shutdownInhibitorAlertTimeout` into the future. When the timer expires, an event will be generated showing the `lease` has been held for too long and further investigation may be conducted.

#### API server connectivity issues
In the event of having connectivity issues towards API server from a node, kubelet will be unable to get any updates for `lease` resources and workloads are unable to make any changes. The strategy from the kubelet's point of view should be simple: keep the last known status until API server is available again. Let's inspect the possibilities.

In all of the situations that follow we will assume the `lease` operation from the client succeeds, but the update does not reach the kubelet.

A `lease` is created. Kubelet did not get the update, therefore a _block_ inhibitor is not placed and the node conditions are not updated. Applications should not start any critical tasks until they check the `lease` has been acquired and the node conditions show the inhibitor is in place, therefore nothing happens.

A `lease` is removed. Kubelet will not remove the inhibitor, but the workload already released the object, allowing others to grab it. This could end up in a race condition once the API server is back where another workload on the same node would acquire the `lease` and checks for the node condition to be true. This condition could be true from the previous `lease` that was not updated yet, rendering a possible removal of the _block_ inhibitor and then creating it again. In that timespan a shutdown would be allowed. To mitigate this we can include the `spec.holderIdentity` in the node conditions `reason` as a way for workloads to check not only if there is a _block_ inhibitor, but if it belongs to them.

A `lease` is updated. Since kubelet only cares about `spec.holderIdentity` and `spec.acquireTime` this is analogous to the removal.

#### Interaction with graceful node shutdown
If graceful node shutdown is active and a shutdown is initiated, the first thing that will happen is a _SIGTERM_ being sent to all workloads in the node. This is not exclusive of a shutdown though, as pod eviction is also sending the same signal and it does not mean a shutdown will happen.</br>
After _SIGTERM_ is received all workloads must prepare for termination and not perform any interactions with any part of the system besides cleanup. If this pattern is not followed then a `lease` might be created. Since kubelet already knows a shutdown is in progress, it will ignore any `lease` create/delete operations on the node, as these are not actionable until after the reboot. When the node is back up again, normal operation will be resumed: create _blocks_ if needed (inhibitors information is volatile and removed on reboot).

A shutdown in progress, however, should rarely take more than a few seconds and workloads creating leases in that time span would incur into a design flaw. Once a _SIGTERM_ is received no further activity aside from preparing for termination should be done.

Even if we configured a high graceful termination period for our critical workloads plus a high delay for shutdown, we could enter situations in which we would have issues. If other pods in the node providing functions like special networking, for example, were to terminate before our critical workload, the uninterruptible operation may not finish properly.

### User Stories (Optional)
#### Story 1
As a developer I want to ensure there are no reboots on the node where my workload is running while performing critical tasks.

#### Story 2
As an admin I want to ensure there are no reboots on the node where I need to perform operations.

### Notes/Constraints/Caveats (Optional)
N/A

### Risks and Mitigations
* Kubelet is unable to configure inhibitor lock.
  * Mitigation: Kubelet will not be able to place a block on shutdown operations. Any command will fall back into either nothing or graceful node shutdown, if configured. In other words, this is current behavior.
* Shutdown event ignores systemd inhibitors.
  * Mitigation: Some shutdown commands may be set up to ignore systemd inhibitors, meaning they will skip dbus entirely and proceed as if no inhibitors were configured. These can not be controlled and it is also a means of debugging without the system trying to avoid it. Like before, this is current behavior.
* OS does not use systemd or systemd version is < 183
  * Mitigation: Kubelet will not be able to detect shutdown requests nor configure locks. This is the same as current behavior, as shutdowns can not be stopped.

## Design Details
Leveraging on the [graceful node shutdown](https://github.com/kubernetes/enhancements/issues/2000) changes, there is little to be added for this feature to work. We need a new configuration parameter, `shutdownInhibitorAlertTimeout` used to specify how much time we should wait for a `lease` to alert the user since it was acquired, described in above sections.
```go
type KubeletConfiguration struct {
  ...
  ShutdownInhibitorAlertTimeout metav1.Duration
}
```

Kubelet will now have a `lease` informer to react to changes in these objects, which means it should have RBAC permissions to do so in any namespace.</br>
It also relies in the `github.com/godbus/v5` package that [graceful node shutdown](https://github.com/kubernetes/enhancements/issues/2000) is using.

### Test Plan
* Unit tests for kubelet of blocking shutdown upon lease creation.
* Unit tests for kubelet of unblocking shutdown upon lease deletion.
* Unit tests for kubelet of alerting lease expiry/inactivity.
* New E2E tests to validate blocking shutdown.

[X] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

##### Prerequisite testing updates
N/A

##### Unit tests
N/A

##### Integration tests
N/A

##### e2e tests
N/A

### Graduation Criteria
#### Alpha
- Feature implemented on top of systemd dependence for linux.
- Feature flag.
- Unit tests implemented mocking system components.

#### Beta
- Gather and address feedback from developers and alpha testers
- Additional tests are in Testgrid and linked in KEP

#### GA
- Gather and address feedback from beta users.
- No more changes to kubelet are needed.
- No issues open.

### Upgrade / Downgrade Strategy
N/A

### Version Skew Strategy
N/A

## Production Readiness Review Questionnaire

### Feature Enablement and Rollback
###### How can this feature be enabled / disabled in a live cluster?
- [X] Feature gate (also fill in values in `kep.yaml`)
  - Feature gate name: `BlockShutdown`
  - Components depending on the feature gate:
    - `kubelet`
- [ ] Other
  - Describe the mechanism:
  - Will enabling / disabling the feature require downtime of the control
    plane?
    - no
  - Will enabling / disabling the feature require downtime or reprovisioning
    of a node? (Do not assume `Dynamic Kubelet Config` feature is enabled).
    - yes (requires kubelet restart)

###### Does enabling the feature change any default behavior?
Enabling the feature in presence of labeled leases will change behavior to completely block shutdown commands unless:
* Lease is deleted, or label is removed from the lease.
* Lease expires.
* Reboot command includes options to ignore inhibitors.

###### Can the feature be disabled once it has been enabled (i.e. can we roll back the enablement)?
There are several ways to disable it:
* Disabling the feature gate.
* Setting a 0s value for `kubeletConfig.ShutdownBlockTimeout`

###### What happens if we reenable the feature if it was previously rolled back?
If there are any present leases specially labeled, shutdown commands will be blocked for at most `kubeConfig.ShutdownBlockTimeout`.

###### Are there any tests for feature enablement/disablement?
N/A

### Rollout, Upgrade and Rollback Planning
###### How can a rollout or rollback fail? Can it impact already running workloads?
No impact to rollouts.

###### What specific metrics should inform a rollback?
N/A

###### Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?
The feature is part of kubelet config so updating it should enable/disable the feature; upgrade/downgrade is N/A.

###### Is the rollout accompanied by any deprecations and/or removals of features, APIs, fields of API types, flags, etc.?
No

### Monitoring Requirements
###### How can an operator determine if the feature is in use by workloads?
Check if the feature gate and kubelet config settings are enabled on a node.

###### How can someone using this feature know that it is working for their instance?
- [X] Events
  - Event Reason: `Shutdown`
- [ ] API .status
  - Condition name: 
  - Other field: 
- [ ] Other (treat as last resort)
  - Details:

###### What are the reasonable SLOs (Service Level Objectives) for the enhancement?
N/A

###### What are the SLIs (Service Level Indicators) an operator can use to determine the health of the service?
N/A

###### Are there any missing metrics that would be useful to have to improve observability of this feature?
N/A

### Dependencies
###### Does this feature depend on any specific services running in the cluster?
This KEP depends only on systemd running on each node in the cluster.

### Scalability
###### Will enabling / using this feature result in any new API calls?
Kubelet configures an informer for `lease` objects, meaning we need to grant cluster wide `list` and `watch` permissions. Any operation on a `lease` object will trigger an update, but only those with the special label will trigger the new code paths.

###### Will enabling / using this feature result in introducing new API types?
No

###### Will enabling / using this feature result in any new calls to the cloud provider?
No

###### Will enabling / using this feature result in increasing size or count of the existing API objects?
Enabling the feature does not incur into additional resources. When using the feature, it requires at least one lease object to block shutdown.

###### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?
No


###### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?
No

### Troubleshooting
###### How does this feature react if the API server and/or etcd is unavailable?
The kubelet now configures a lease informer, which in case of API server/etcd downtime would stop receiving events and, potentially, skip shutdown locks.
If any locks were already configured then they will be released at the end of `kubeletConfig.ShutdownBlockTimeout`.

###### What are other known failure modes?
N/A

###### What steps should be taken if SLOs are not being met to determine the problem?
N/A

## Implementation History
N/A

## Drawbacks
N/A

## Alternatives
* Synchronize workloads.
  * Have all shutdown/reboot participating workloads establish a way to communicate to each other and coordinate on reboot commands.
  * High coupling between potentially non-related components.
* Use PDBs.
  * Own section above.
* Use CRDs and dedicated controllers.
  * By using a CRD and a controller for it we can centralize control on a single entity, taking care of coordination and blocking. Status updates to each CR to show current progress and info on blocking.
  * Needs to run an instance on all nodes.
  * Introducing another component post-install. Need to care about application lifecycle, and inhibitors are now controlled by kubelet and this component too.
* Use a centralized namespace.
  * Create one lease per node in a centralized namespace. Workloads would need to share the leases, if someone else owns it then wait to grab it.
  * Sharing a unique lease per node comes with sync issues that might slip a shutdown command when it shouldnt.
  * Control plane should create the namespace.
  * Need to have node names as lease names.
* Don't handle shutdown blocks, let admins do manual operations around critical tasks.
  * Workloads are unaware of other workloads needs and intentions, potentially disrupting them.
  * Avoid any automatic rebooting/shutdown and do it manually to ensure no critical operations are interrupted.

## Infrastructure Needed (Optional)
N/A
