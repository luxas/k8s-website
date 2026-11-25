---
layout: blog
title: 'Kubernetes v1.38: Introducing Conditional Authorization (Alpha)'
draft: true
slug: kubernetes-v1-38-conditional-authorization-alpha
author: >
   [Lucas Käldström](https://github.com/luxas) (AWS)
---

Kubernetes users have for a long time expressed interest in extending Kubernetes RBAC with label and field selectors, in order to not just say `lucas can create Pods` but `lucas can create Pods, but only with non-root containers`. For various reasons, we in SIG Auth did not know, until recently, whether would be technically or practically feasible to implement such object-level granularity in Kubernetes authorizers. 

However, we're glad to announce that we've made progress in this area with the Conditional Authorization feature shipped in Kubernetes v1.38, at the alpha maturity level.

This blog post will walk through what a solution to this problem looks like, why it was complex to solve, how you can take the feature into use today, where SIG Auth is planning to go next with this feature, and how you can contribute to the effort.

## Background: Why not Kubernetes RBAC + ValidatingAdmissionPolicy?

Those of you that are familiar with Kubernetes RBAC and Validating Admission Policies might be thinking: "I can express that with a combination of Kubernetes RBAC and Validating Admission Policy rules, how is this different?"

This is partially true, but its a workaround that we've all had to live with, not a solution to the fundamental problem.

Let's recall the Kubernetes request chain. A Kubernetes API write request goes through the following phases (on a high level):

1. **Authentication**: This phase resolves who the user _is_, in terms of a username, a set of groups, and extra user metadata.
1. **Authorization**: This phase checks "can this user perform this type of API request?", based on the user information and request metadata (API group, resource, namespace, name) alone.
  1. Kubernetes RBAC operates in this phase.
1. Decoding of the request payload (new object), if any.
1. **Mutating Admission**: Admission plugins can change fields on the request object according to various policies.
1. **Validating Admission**: Admission plugins ensure the new (request payload) object and the old (stored) API objects are compliant to the rules of the cluster administrator.
  1. Validating Admission Policy (VAP) or validating admission webhooks operates in this phase. 
1. **Storage**: If the authorizer and all admission plugins accepted the request, the write request is applied and written to etcd.

TODO: Insert picture.

While it is possible to declare a rule of "this user can perform write requests like this, but only on objects matching this criteria" by first creating an "over-granting" RBAC allow policy (e.g. `Role` and `RoleBinding`) and then creating a deny policy using `ValidatingAdmissionPolicy` and `ValidatingAdmissionPolicyBinding` to correct the over-grant, this approach has certain limitations:

1. **Administrator complexity**: The administrator needs to create and understand four separate API objects, how they relate to each other, and two different paradigms (RBAC first, then CEL).
  1. Ideally, the adminstrator could define their intended access control policy once, and then let the system figure it out under the hood.
1. **Non-atomicity**: Because four API objects are needed, the desired policy cannot be atomically applied. Thus, the administrator needs to first apply the VAP deny policy, and when successfully applied, apply the RBAC allow rule, and perform removals in the opposite order. This in order to not have face a situation when only the overly broad RBAC allow rule would be applied alone.
  1. Ideally, creates, updates and deletes of the access control policy itself is atomic, either applies or not as a unit.
1. **User experience**: If a user tries to understand their permissions, `kubectl auth can-i` will answer `yes`, even though the user actually is restricted to only write certain objects.
  1. Ideally, the user can introspect the conditions that restrict their abilities to perform requests of a given type.
1. **No read support**: `ValidatingAdmissionPolicy` (and admission plugins in general) only apply to write requests. Webhook authorizers in Kubernetes can restrict read requests since [KEP-4601], but not through a built-in API, and not in a way that resembles VAP.
  1. Ideally, access control policies for writes and reads should be uniform, that is, as close to each other as possible.

Now we have gathered a set of requirements for a solution. Next, enter Conditional Authorization.

## Solution: Conditional Authorization



## Try it out!

## Next stop: RBAC++

## Join the effort and contribute!

Ideally, the administrator could just express the desired policy once, and the system figures it out.

Expressiveness vs usability

there have been technical hurdles 


[KEP-4601]: https://github.com/kubernetes/enhancements/tree/master/keps/sig-auth/4601-authorize-with-selectors


Scratch:
1. **ValidatingAdmissionPolicy writers should be few**:

Placeholder for the
[KEP-5681](https://github.com/kubernetes/enhancements/issues/5681) conditional
authorization feature blog.

Outline:
- Problem statement and proposed solution
- History/Context
  - Link to my and Micah's talk
  - Link to KEP
- How to use this
- What this is and isn't
  - Not RBAC++, that's next
  - Not reads, only writes
- What's next

Thank the reviewers, etc.

