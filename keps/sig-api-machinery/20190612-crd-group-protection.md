---
title: k8s.io Group Protection
authors:
  - "@deads2k"
owning-sig: sig-api-machinery
participating-sigs:
  - sig-api-machinery
reviewers:
  - "@sttts"
  - "@jpbetz"
  - "@liggitt"
approvers:
  - "@liggitt"
  - "@sttts"
editor:
  name: "@deads2k"
creation-date: 2019-06-12
last-updated: 2019-06-12
status: implementable
---

# k8s.io Group Protection

## Table of Contents

## Summary

API groups are organized by namespace, similar to java packages.  `authorization.k8s.io` is one example.  When users create
CRDs, they get to specify an API group and their type will be injected into that group by the kube-apiserver.

The *.k8s.io or *.kubernetes.io groups are owned by the Kubernetes community and protected by API review (see [What APIs need to be reviewed](https://github.com/kubernetes/community/blob/master/sig-architecture/api-review-process.md#what-apis-need-to-be-reviewed),
to ensure consistency and quality.  To avoid confusion in our API groups and prevent accidentally claiming a
space inside of the kubernetes API groups, the kube-apiserver needs to be updated to protect these reserved API groups.

This KEP proposes adding an `api-approved.kubernetes.io` annotation to CustomResourceDefinition.  This is only needed if
the CRD group is `k8s.io`, `kubernetes.io`, or ends with `.k8s.io`, `.kubernetes.io`.  The value should be a link to the
pull request where the API has been approved.
 
```yaml
metadata:
  annotations:
    "api-approved.kubernetes.io": "https://github.com/kubernetes/kubernetes/pull/78458"
```

## Motivation

* Prevent accidentally including an unapproved API in community owned API groups
 
### Goals

* Ensure API consistency. 
* Prevent accidentally claiming reserved named.

### Non-Goals

* Prevent malicious users from claiming reserved names.

## Proposal

This KEP proposes adding an `api-approved.kubernetes.io` annotation to CustomResourceDefinition.  This is only needed if
the CRD group is `k8s.io`, `kubernetes.io`, or ends with `.k8s.io`, `.kubernetes.io`.  The value should be a link to the
pull request where the API has been approved.
 
```yaml
metadata:
  annotations:
    "api-approved.kubernetes.io": "https://github.com/kubernetes/kubernetes/pull/78458"
```

This field is used by new kube-apiservers to set the `KubeAPIApproved` condition.  
 1. If a new server sees a CRD for a resource in a kube group and sees the annotation, it will set the `KubeAPIApproved` condition to true.
 2. If a new server sees a CRD for a resource in a kube group and does not see the annotation, it will set the `KubeAPIApproved` condition to false.
 3. If a new server sees a CRD for a resource outside a kube group, it does not set the `KubeAPIApproved` condition at all.

In v1, this annotation will be required in order to create a CRD for a resource in one of the kube API groups.

### Behavior of new clusters
1. Current CRD for a resource in the kube group already in API is missing valid `api-approved.kubernetes.io` - `KubeAPIApproved` condition will be false.
2. CRD for a resource in the kube group creating via CRD.v1beta1 is missing valid `api-approved.kubernetes.io` - create as normal.  This ensures compatibility.  `KubeAPIApproved` condition will be false.
3. CRD for a resource in the kube group creating via CRD.v1 is missing valid `api-approved.kubernetes.io` - fail validation and do not store in etcd.
4. CRD for a resource outside the kube group creating via CRD.v1 is contains the `api-approved.kubernetes.io` - fail validation and do not store in etcd.
5. In CRD.v1, remove a required `api-approved.kubernetes.io` - fail validation.
5. In all versions, any update that does not change the `api-approved.kubernetes.io` will go through our current validation rules.
 
  
### Behavior of old clusters
1.  Nothing changes.  The old clusters will persist and keep the annotation

This doesn't actively prevent bad actors from simply setting the annotation, but it does prevent accidentally claiming
an inappropriate name. 

## References

* [Accidental name in Kanary](https://libraries.io/github/AmadeusITGroup/kanary))

### Test Plan

**blockers for GA:**

* Document in the release notes.  The impact is very low

### Graduation Criteria

* the test plan is fully implemented for the respective quality level

### Upgrade / Downgrade Strategy

* annotations and conditions are always persisted.  If set, they remain consistent.  If unset, they also remain consistent.

### Version Skew Strategy

* annotations and conditions are always persisted.  If set, they remain consistent.  If unset, they also remain consistent.

## Alternatives considered

## Implementation History
