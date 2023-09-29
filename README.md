---
title: observability-ui-operator
authors:
  - @jgbernalp
reviewers: # Include a comment about what domain expertise a reviewer is expected to bring and what area of the enhancement you expect them to focus on. For example: - "@networkguru, for networking aspects, please look at IP bootstrapping aspect"
  - TBD
approvers: # A single approver is preferred, the role of the approver is to raise important questions, help ensure the enhancement receives reviews from all applicable areas/SMEs, and determine when consensus is achieved such that the EP can move forward to implementation.  Having multiple approvers makes it difficult to determine who is responsible for the actual approval.
  - TBD
api-approvers: # In case of new or modified APIs or API extensions (CRDs, aggregated apiservers, webhooks, finalizers). If there is no API change, use "None"
  - "None"
creation-date: 2023-09-18
last-updated: 2023-09-18
tracking-link:
  - [OU-204](https://issues.redhat.com/browse/OU-204)
see-also:
  - ""
replaces:
  - ""
superseded-by:
  - ""
---

title: observability-ui-operator

# Observability UI Operator

The Observability UI Operator aims to manage dynamic console plugins for observability signals inside the OpenShift console, ensuring a consistent user experience and efficient management of UI plugins.

## Summary

This proposal introduces the Observability UI Operator, a tool designed to manage UI plugins related to observability signals within the OpenShift console. By centralizing the management of such plugins, we can offer a unified observability experience in the console and accommodate new use cases, all while decoupling UI responsibilities from other operators.

## Motivation

### Why:

The current state of observability signals in the OpenShift console has each operator responsible for its own console plugin. This sometimes results in operators deploying plugins outside their primary scope. As the requirements for the console's UI grow, there's a clear need for a centralized system that can manage diverse UI components spanning across various signals to offer a unified observability experience in the console.

### User Stories

- As an OpenShift user, I want an operator from the Red Hat catalog that can deploy various observability UI components so that all signals supported by the cluster are easily accessible and can be used for troubleshooting.

- As an OpenShift administrator, I want a centralized operator for observability UI components so that I can streamline console requirements and integrate diverse signals effectively.

- As an OpenShift user, I want to customize observability dashboards with various signals so that I can quickly identify and resolve issues.

### Goals

Provide a centralized operator to manage dynamic UI plugins for observability signals within the OpenShift console.

Allow OpenShift users and administrators to access various observability signals through integrated console plugins.

Decouple the responsibility of managing observability UI from operators, enabling each operator to focus solely on its primary functionalities.

Enhance the observability experience on the console by providing components like [Perses](https://github.com/perses/perses) for customizable dashboards and [korrel8r](https://github.com/korrel8r/korrel8r) for correlation.

Ensure that the Observability UI operator remains independent of the OpenShift Container Platform (OCP) release cycle, thus allowing more frequent updates with new features and fixes.

### Non-Goals

Deploying third-party Observability UI solutions like Grafana or Kibana.

Supporting UIs coming from other observability projects such as Jaeger UI, Prometheus/Thanos UI, or AlertManager UI.

Replacing or superseding the monitoring plugin deployed by the Cluster Monitoring Operator.

## Proposal

The proposal is to introduce an Observability UI Operator that primarily aims at centralizing the management of dynamic console plugins dedicated to observability signals for the OpenShift console.

### Current State and Challenges:

Currently, individual operators, each responsible for specific observability signals, deploy their own console plugins in OpenShift, allowing users to access related signal data. Some operators end up deploying plugins not related to their primary function. For example, the logging operator might deploy a plugin exclusively using services from the Loki operator.

The current design sees a rapid expansion of the console's UI requirements, which now exceed just interfacing with the default signal sources. Users are demanding components that can work across various signals, encompassing features like customizable dashboards in Perses, and observability UIs that can read signals from multiple operators in a cluster, such as ACM or STF.

A significant drawback of the existing system is that, for advanced features like customizable dashboards, UI components require backend services that do not fit seamlessly within the current operators.

### Design:

The Observability UI Operator will be available in the Red Hat catalog.

It will manage the deployment of several components which will be added incrementally based on priority, as shown in the table below:

| Component                          | Component Type         | Current features                                                                                                                                                                                                                                                   | Planned features                                                                                                                               |
| ---------------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Dashboards console plugin          | Console Dynamic Plugin | Allows the current dashboards to fetch data from other Prometheus services in the cluster, different from the cluster monitoring                                                                                                                                   | Integrate Perses UI for dashboards <br> Allow new dashboards to consume data from several data sources <br> Allow new dashboards customization |
| Logging view plugin                | Console Dynamic Plugin | Displays logs under the observe section for the admin perspective <br> Displays logs in the pod detail view <br> Displays logs as a tab in the observe menu for the dev perspective <br> Merges log-based alerts with monitoring alerts in the observe alerts view | Allow dev console users to see logs from several namespaces at the same time <br> Display log metrics                                          |
| Perses Operator                    | Operator               | N/A – not yet released                                                                                                                                                                                                                                             | Allow dashboards definitions as CRDs <br> Manage dashboards authorization <br> Manage schema migrations from grafana                           |
| Distributed tracing console plugin | Console Dynamic Plugin | N/A – not yet released                                                                                                                                                                                                                                             | Display a scatter plot with a list of traces <br> Display a gantt chart to display the detail of a trace                                       |
| Korrel8r console plugin            | Console Dynamic Plugin | N/A – not yet released                                                                                                                                                                                                                                             | Display a side panel to allow navigate across observability signals present on the cluster                                                     |

To maintain a level of flexibility and ensure compatibility, each plugin within this operator will come with a set of feature toggles. These toggles can be activated or deactivated based on the OCP version and the version of the respective signal operator.

The release cycle for the Observability UI Operator will be distinct from the OpenShift Container Platform's minor version releases. With this approach, we anticipate more frequent updates with newer features and fixes. Tentatively, minor releases are planned every 3 months, subject to change based on the specific features targeted for each release.

Migration plans will be crucial. Current plugins deployed by other signal operators are optional. These can be disabled and replaced by plugins from the Observability UI Operator. Existing operators deploying an observability plugin will need to direct customers towards migration to the Observability UI Operator and strategize for the phasing out of their plugin support.

### Console plugin configuration

The Observability UI Operator will be configured through a series of custom resource (CR) that will allow the user to enable or disable the console plugins and link them with the corresponding backend from signal operators.

### Objective:

The core intention behind this change is twofold:

To streamline the observability experience on the OpenShift console, offering users a unified and adaptable approach.

Decouple the UI responsibilities from specific operators, allowing them to focus exclusively on their main functions.

By centralizing the observability UI components under one operator, we hope to minimize redundancy, improve user experience, and cater more efficiently to the expanding requirements of the console's UI.

### Workflow Description

The workflow for the Observability UI Operator focuses on enhancing the user experience when interfacing with the OpenShift console. This proposal targets a more unified and enhanced observability experience through a new Kubernetes operator.

**OpenShift cluster administrator** is responsible for installing, enabling, configuring, and managing the plugins and operators within the OpenShift environment.
**OpenShift user** is the end-user interfacing with the OpenShift console and making use of the observability signals presented by the dynamic console plugins.

1. The cluster administrator installs the ObservabilityUI operator from the RedHat Catalog.
2. If there is an existing observability UI plugin deployed by another operator, the cluster administrator disables it.
3. The cluster administrator configures the operator adding a custom resources (CR) to deploy the desired plugins and link them with the corresponding signal operators.
4. The operator will reconcile the necessary deployments for each enabled plugin, then it will reconcile the required [CRs for the console operator](https://github.com/openshift/enhancements/blob/master/enhancements/console/dynamic-plugins.md#delivering-plugins) so they become be available in the OpenShift console.
5. The user accesses the OpenShift console and interacts with the observability signals through the plugins deployed by the operator.

### API Extensions

This enhancement introduces a new CRD to represent observability UI console plugins. The `ObservabilityUIConsolePlugin` CR for a plugin will be defined as follows:

```yaml
apiVersion: observability-ui.openshift.io/v1alpha1
kind: ObservabilityUIConsolePlugin
metadata:
  name: logging-view-plugin
  namespace: openshift-observability-ui
spec:
  displayName: "Logging View Plugin"
  deployment:
    containers:
      - name: logging-view-plugin
        image: "quay.io/gbernal/logging-view-plugin:latest"
        ports:
          - containerPort: 9443
            protocol: TCP
  backend:
    type: logs
    sources:
      - alias: logs-backend
        caCertificate: '-----BEGIN CERTIFICATE-----\nMIID....'
        authorize: true
        endpoint:
          type: Service
          service:
            name: lokistack-dev
            namespace: openshift-logging
            port: 8080
  settings:
    timeout: 30s
    logsLimit: 300
```

#### Behavior Modification of Existing Resources:

No existing resources are modified by this operator. However, operators that previously deployed their own UI plugins may need to consider to point their users to the Observability UI Operator docs so they can continue using the plugin in the console.

#### Operational Aspects of API Extensions:

- Labels and Annotations: Owner references will be added to the deployments and console operator CRs so resources are cleaned up when an observability UI component is deleted.

- Compatibility and Feature Flags: The Observability UI operator will add a set of feature toggles to each plugin deployment to ensure that only compatible features are deployed. This compatibility is based on the OCP version and the signal operator version. See the [logging view plugin compatibility matrix](https://github.com/openshift/logging-view-plugin/tree/main#compatibility-matrix)

### Risks and Mitigations

| Risk                 | Description                                                                                                                                                                                                            | Mitigation                                                                                                                                                                                                                                                                |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Compatibility Issues | Given that each plugin will have feature toggles based on the OCP version and signal operator version, there is a potential for compatibility issues to arise, especially as more plugins and features are introduced. | Ensure robust testing mechanisms that simulate different environments and configurations. Maintain a detailed compatibility matrix and ensure that it is updated regularly. Provide a mechanism for users to report compatibility issues, and prioritize addressing them. |
| Migration Challenges | Existing operators deploying observability plugins might face challenges when migrating to the new Observability UI Operator.                                                                                          | Develop a step-by-step migration guide. Offer dedicated support during the initial migration phase to help teams seamlessly transition. Provide tooling, if possible, to automate or simplify parts of the migration process.                                             |

Review Processes:

- Security Review: Security will be reviewed by the ProdSec team. They will evaluate the architecture, perform vulnerability assessments, and validate the security practices adopted in the operator.

- UX Review: The UX team will review the design and user experience of the observability UI plugins. Feedback will be incorporated to ensure a seamless and consistent user experience across the OpenShift console.

- Stakeholder Engagement: Involve teams that work on Loki, Jaeger, Prometheus, and other related projects in the review process. Their feedback will be invaluable in understanding the broader implications and ensuring that the operator integrates well.

### Drawbacks

// TODO

The idea is to find the best form of an argument why this enhancement should
_not_ be implemented.

What trade-offs (technical/efficiency cost, user experience, flexibility,
supportability, etc) must be made in order to implement this? What are the reasons
we might not want to undertake this proposal, and how do we overcome them?

Does this proposal implement a behavior that's new/unique/novel? Is it poorly
aligned with existing user expectations? Will it be a significant maintenance
burden? Is it likely to be superceded by something else in the near future?

## Design Details

// TODO

### Open Questions [optional]

This is where to call out areas of the design that require closure before deciding
to implement the design. For instance,

> 1.  This requires exposing previously private resources which contain sensitive
>     information. Can we do this?

### Test Plan

// TODO

**Note:** _Section not required until targeted at a release._

Consider the following in developing a test plan for this enhancement:

- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?
- What additional testing is necessary to support managed OpenShift service-based offerings?

No need to outline all of the test cases, just the general strategy. Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations).

### Graduation Criteria

// TODO

**Note:** _Section not required until targeted at a release._

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. Initial proposal
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this
enhancement:

- Maturity levels
  - [`alpha`, `beta`, `stable` in upstream Kubernetes][maturity-levels]
  - `Dev Preview`, `Tech Preview`, `GA` in OpenShift
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning),
or by redefining what graduation means.

In general, we try to use the same stages (alpha, beta, GA), regardless how the functionality is accessed.

[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

**If this is a user facing change requiring new or updated documentation in [openshift-docs](https://github.com/openshift/openshift-docs/),
please be sure to include in the graduation criteria.**

**Examples**: These are generalized examples to consider, in addition
to the aforementioned [maturity levels][maturity-levels].

#### Dev Preview -> Tech Preview

// TODO

- Ability to utilize the enhancement end to end
- End user documentation, relative API stability
- Sufficient test coverage
- Gather feedback from users rather than just developers
- Enumerate service level indicators (SLIs), expose SLIs as metrics
- Write symptoms-based alerts for the component(s)

#### Tech Preview -> GA

// TODO

- More testing (upgrade, downgrade, scale)
- Sufficient time for feedback
- Available by default
- Backhaul SLI telemetry
- Document SLOs for the component
- Conduct load testing
- User facing documentation created in [openshift-docs](https://github.com/openshift/openshift-docs/)

**For non-optional features moving to GA, the graduation criteria must include
end to end tests.**

#### Removing a deprecated feature

// TODO

- Announce deprecation and support policy of the existing feature
- Deprecate the feature

### Upgrade / Downgrade Strategy

// TODO

If applicable, how will the component be upgraded and downgraded? Make sure this
is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:

- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to keep previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to make use of the enhancement?

Upgrade expectations:

- Each component should remain available for user requests and
  workloads during upgrades. Ensure the components leverage best practices in handling [voluntary
  disruption](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/). Any exception to
  this should be identified and discussed here.
- Micro version upgrades - users should be able to skip forward versions within a
  minor release stream without being required to pass through intermediate
  versions - i.e. `x.y.N->x.y.N+2` should work without requiring `x.y.N->x.y.N+1`
  as an intermediate step.
- Minor version upgrades - you only need to support `x.N->x.N+1` upgrade
  steps. So, for example, it is acceptable to require a user running 4.3 to
  upgrade to 4.5 with a `4.3->4.4` step followed by a `4.4->4.5` step.
- While an upgrade is in progress, new component versions should
  continue to operate correctly in concert with older component
  versions (aka "version skew"). For example, if a node is down, and
  an operator is rolling out a daemonset, the old and new daemonset
  pods must continue to work correctly even while the cluster remains
  in this partially upgraded state for some time.

Downgrade expectations:

- If an `N->N+1` upgrade fails mid-way through, or if the `N+1` cluster is
  misbehaving, it should be possible for the user to rollback to `N`. It is
  acceptable to require some documented manual steps in order to fully restore
  the downgraded cluster to its previous state. Examples of acceptable steps
  include:
  - Deleting any CVO-managed resources added by the new version. The
    CVO does not currently delete resources that no longer exist in
    the target version.

### Version Skew Strategy

// TODO

How will the component handle version skew with other components?
What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:

- During an upgrade, we will always have skew among components, how will this impact your work?
- Does this enhancement involve coordinating behavior in the control plane and
  in the kubelet? How does an n-2 kubelet without this feature available behave
  when this feature is used?
- Will any other components on the node change? For example, changes to CSI, CRI
  or CNI may require updating that component before the kubelet.

### Operational Aspects of API Extensions

// TODO

Describe the impact of API extensions (mentioned in the proposal section, i.e. CRDs,
admission and conversion webhooks, aggregated API servers, finalizers) here in detail,
especially how they impact the OCP system architecture and operational aspects.

- For conversion/admission webhooks and aggregated apiservers: what are the SLIs (Service Level
  Indicators) an administrator or support can use to determine the health of the API extensions

  Examples (metrics, alerts, operator conditions)

  - authentication-operator condition `APIServerDegraded=False`
  - authentication-operator condition `APIServerAvailable=True`
  - openshift-authentication/oauth-apiserver deployment and pods health

- What impact do these API extensions have on existing SLIs (e.g. scalability, API throughput,
  API availability)

  Examples:

  - Adds 1s to every pod update in the system, slowing down pod scheduling by 5s on average.
  - Fails creation of ConfigMap in the system when the webhook is not available.
  - Adds a dependency on the SDN service network for all resources, risking API availability in case
    of SDN issues.
  - Expected use-cases require less than 1000 instances of the CRD, not impacting
    general API throughput.

- How is the impact on existing SLIs to be measured and when (e.g. every release by QE, or
  automatically in CI) and by whom (e.g. perf team; name the responsible person and let them review
  this enhancement)

#### Failure Modes

// TODO

- Describe the possible failure modes of the API extensions.
- Describe how a failure or behaviour of the extension will impact the overall cluster health
  (e.g. which kube-controller-manager functionality will stop working), especially regarding
  stability, availability, performance and security.
- Describe which OCP teams are likely to be called upon in case of escalation with one of the failure modes
  and add them as reviewers to this enhancement.

#### Support Procedures

// TODO

Describe how to

- detect the failure modes in a support situation, describe possible symptoms (events, metrics,
  alerts, which log output in which component)

  Examples:

  - If the webhook is not running, kube-apiserver logs will show errors like "failed to call admission webhook xyz".
  - Operator X will degrade with message "Failed to launch webhook server" and reason "WehhookServerFailed".
  - The metric `webhook_admission_duration_seconds("openpolicyagent-admission", "mutating", "put", "false")`
    will show >1s latency and alert `WebhookAdmissionLatencyHigh` will fire.

- disable the API extension (e.g. remove MutatingWebhookConfiguration `xyz`, remove APIService `foo`)

  - What consequences does it have on the cluster health?

    Examples:

    - Garbage collection in kube-controller-manager will stop working.
    - Quota will be wrongly computed.
    - Disabling/removing the CRD is not possible without removing the CR instances. Customer will lose data.
      Disabling the conversion webhook will break garbage collection.

  - What consequences does it have on existing, running workloads?

    Examples:

    - New namespaces won't get the finalizer "xyz" and hence might leak resource X
      when deleted.
    - SDN pod-to-pod routing will stop updating, potentially breaking pod-to-pod
      communication after some minutes.

  - What consequences does it have for newly created workloads?

    Examples:

    - New pods in namespace with Istio support will not get sidecars injected, breaking
      their networking.

- Does functionality fail gracefully and will work resume when re-enabled without risking
  consistency?

  Examples:

  - The mutating admission webhook "xyz" has FailPolicy=Ignore and hence
    will not block the creation or updates on objects when it fails. When the
    webhook comes back online, there is a controller reconciling all objects, applying
    labels that were not applied during admission webhook downtime.
  - Namespaces deletion will not delete all objects in etcd, leading to zombie
    objects when another namespace with the same name is created.

## Implementation History

// TODO

Major milestones in the life cycle of a proposal should be tracked in `Implementation
History`.

## Alternatives

// TODO

Similar to the `Drawbacks` section the `Alternatives` section is used to
highlight and record other possible approaches to delivering the value proposed
by an enhancement.
