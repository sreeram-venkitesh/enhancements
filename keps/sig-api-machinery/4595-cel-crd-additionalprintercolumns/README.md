<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [x] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [x] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [x] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [x] **Fill out as much of the kep.yaml file as you can.**
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
# KEP-4595: CEL for CRD AdditionalPrinterColumns

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
  - [Calculating CEL cost limits](#calculating-cel-cost-limits)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
    - [CEL evaluation resulting in error](#cel-evaluation-resulting-in-error)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
      - [Prerequisite testing updates](#prerequisite-testing-updates)
      - [Unit tests](#unit-tests)
      - [Integration tests](#integration-tests)
      - [e2e tests](#e2e-tests)
  - [Graduation Criteria](#graduation-criteria)
    - [Alpha](#alpha)
    - [Beta](#beta)
    - [GA](#ga)
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
  - [ ] (R) Ensure GA e2e tests meet requirements for [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
  - [ ] (R) Minimum Two Week Window for GA e2e tests to prove flake free
- [ ] (R) Graduation criteria is in place
  - [ ] (R) [all GA Endpoints](https://github.com/kubernetes/community/pull/1806) must be hit by [Conformance Tests](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/conformance-tests.md) 
- [ ] (R) Production readiness review completed
- [ ] (R) Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

<!--
This section is incredibly important for producing high-quality, user-focused
documentation such as release notes or a development roadmap. It should be
possible to collect this information before implementation begins, in order to
avoid requiring implementors to split their attention between writing release
notes and implementing the feature itself. KEP editors and SIG Docs
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.

Both in this section and below, follow the guidelines of the [documentation
style guide]. In particular, wrap lines to a reasonable length, to make it
easier for reviewers to cite specific portions, and to minimize diff churn on
updates.

[documentation style guide]: https://github.com/kubernetes/community/blob/master/contributors/guide/style-guide.md
-->

This enhancement proposes to let users define human readable printer columns for custom resource definitions using CEL expressions as an alternative to using JSON path.

## Motivation

<!--
This section is for explicitly listing the motivation, goals, and non-goals of
this KEP.  Describe why the change is important and the benefits to users. The
motivation section can optionally provide links to [experience reports] to
demonstrate the interest in a KEP within the wider Kubernetes community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

Currently, when creating CustomResourceDefinitions you can define a map of `additionalPrinterColumns` that would be displayed when querying the custom resources with kubectl. This list of `additionalPrinterColumns` are defined using JSON paths. If your CustomResourceDefinition is defined in the following manner, running `kubectl get mycrd myresource` would yield the following response.

```yaml
additionalPrinterColumns:
- name: Desired
  type: integer
  jsonPath: .spec.replicas
- name: Current
  type: integer
  jsonPath: .status.replicas
- name: Age
  type: date
  jsonPath: .metadata.creationTimestamp
```

```
NAME                 DESIRED    CURRENT     AGE
myresource           1          1           7s
```

This approach has a few limitations such as not being able to support arrays, missing support for processing conditionals, not being able to compute column value from multiple fields and difficulty with formatting dates as duration from another timestamp.

With the advent of CEL, we can provide an alternative input for `additionalPrinterColumns` to represent the value in CEL for more complicated table readings. This would be added along with the existing JSON path and users can define `additionalPrinterColumns` for their CRDs in either JSON path or as a CEL expression.

### Goals

<!--
List the specific goals of the KEP. What is it trying to achieve? How will we
know that this has succeeded?
-->

* Enable support for defining `additionalPrinterColumns` using **CEL expressions** in CustomResourceDefinitions (CRDs).
* Allow CEL-based and JSONPath-based `additionalPrinterColumns` to coexist in the same CRD. Although, each `additionalPrinterColumn` must use either `jsonPath` or a CEL `expression` field to define the column, not both. However, you can define multiple columns in the same CRD, with some using `jsonPath` and others using CEL `expression` independently. (TODO: fix it)

---

* CEL expressions are compiled at CRD creation time—first during validation via [`ValidateCustomResourceColumnDefinition(...)`](https://github.com/kubernetes/kubernetes/blob/b35c5c0a301d326fdfa353943fca077778544ac6/staging/src/k8s.io/apiextensions-apiserver/pkg/apis/apiextensions/validation/validation.go#L789-L790), and again when setting up the table convertor via [`tableconvertor.New(...)`](https://github.com/kubernetes/kubernetes/blob/b35c5c0a301d326fdfa353943fca077778544ac6/staging/src/k8s.io/apiextensions-apiserver/pkg/registry/customresource/tableconvertor/tableconvertor.go#L39-L41). They are then evaluated at runtime when [printing columns during resource listing](https://github.com/kubernetes/kubernetes/blob/b35c5c0a301d326fdfa353943fca077778544ac6/staging/src/k8s.io/apiextensions-apiserver/pkg/registry/customresource/tableconvertor/tableconvertor.go#L115-L135). This feature does not support recompiling CEL expressions at runtime—changes require re-applying the CRD.

* Leverage CEL’s capabilities to allow expressive, type-safe logic for formatting and displaying CR instances in `kubectl get` outputs.
* Ensure consistency in validation and cost enforcement by reusing existing CEL cost models already integrated across the Kubernetes project.
* Evaluate CEL expressions during `Table` responses (e.g., `kubectl get`), enabling runtime column rendering based on the resource state.
* Apply `type` and `format` attributes to CEL expressions just as they are for `jsonPath`.
* Maintain backward compatibility; CRDs using `jsonPath` for `additionalPrinterColumns` remain fully supported.
* Ensure safe and predictable behavior when CEL expressions fail at runtime—fallback to empty columns and log errors.

---

### Non-Goals

<!--
What is out of scope for this KEP? Listing non-goals helps to focus discussion
and make progress.
-->

* Modify, replace, or phase out JSONPath-based column definitions.
* Expanding CEL’s access scope beyond the current design constraints (e.g., no access to arbitrary `metadata.*` fields beyond `name` and `generateName`). Refer caveats for information. (TODO:)
* Changes to `kubectl` or other clients are required.


## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. What is the desired outcome and how do we measure success?.
The "Design Details" section below is for the real
nitty-gritty.
-->

This KEP propses a new, mutually exclusive sibling field to `additionalPrinterColumns[].jsonPath` called `additionalPrinterColumns[].expression`. This field allows defining printer column values using CEL (Common Expression Language) expressions that evaluate to strings.

To support this, the Kubernetes API will be extended to accept CEL expressions for printer columns, and the API server will evaluate these expressions dynamically when responding to `Table` requests (e.g., `kubectl get`), producing richer, computed, or combined column outputs.

## Example

Given this CRD snippet:

```yaml
additionalPrinterColumns:
- name: Replicas
  type: string
  expression: "%d/%d".format([self.status.replicas, self.spec.replicas])
- name: Age
  type: date
  jsonPath: .metadata.creationTimestamp
- name: Status
  type: string
  expression: self.status.replicas == self.spec.replicas ? "READY" : "WAITING"
```

The `kubectl get` output might look like:

```
NAME         REPLICAS   AGE    STATUS
myresource   1/1        7s     READY
myresource2  0/1        2s     WAITING
```

This enhancement enables flexible, human-friendly column formatting and logic in `kubectl get` outputs without requiring external tooling or complex `JSONPath` workarounds.

---

### Calculating CEL cost limits

We want to make sure that the time taken for computing the CEL expressions for additionalPrinterColumns would be in the same order of magnitude as that of using JSONPath and set the CEL cost limit accordingly. Let's assume we have a representative CRD with 10 additionalPrinterColumns. The total time taken for executing the all the JSONPaths would be equal to the time taken for executing a single JSONPath expression times the total number of additionalPrinterColumns. As part of this proposal we aim to calculate a reasonable total cost for the CEL expressions by multiplying the total number of seconds it takes the expressions to get evaluated with a CEL cost per second, which would be a constant.

As part of this benchmarking, we are also planning to calculate the percentage of time of a `kubectl get` call on a CRD is taken up to compute the additionalPrinterColumns JSONPath. If it is a low enough value, we can set a higher CEL cost before users would notice a time difference between using CEL or JSONPath.

---

### User Stories (Optional)

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

**As a Kubernetes user,** I want to define `additionalPrinterColumns` that correctly aggregate and display all nested arrays within my CRD, so that `kubectl get` outputs the full list of hosts instead of only showing the first array. Current JSONPath-based columns only print the first matching array, resulting in incomplete data.

Using CEL expressions for `additionalPrinterColumns` allows combining all nested arrays into a single flattened list, providing complete and accurate output in `kubectl get`.

```yaml
additionalPrinterColumns:
- name: hosts
  type: string
  description: "All hosts from all servers"
  expression: "flatten(self.spec.servers.map(s, s.hosts))"
```

```
output
```

In the above example:

* `spec.servers` is mapped to extract each `hosts` array.
* `flatten(...)` merges all those nested arrays into one list.
* The resulting list of all hosts is displayed in the column output.

**References:**

* https://github.com/kubernetes/kubectl/issues/517
* https://github.com/kubernetes/kubernetes/pull/67079
* https://github.com/kubernetes/kubernetes/pull/101205
* https://groups.google.com/g/kubernetes-sig-api-machinery/c/GxXWe6T8DoM

#### Story 2

As a Kubernetes user, I want to display the status of a specific condition (e.g., the "Ready" condition) from a list of status conditions in a human-readable column when using `kubectl get`. Currently, `jsonPath`-based `additionalPrinterColumns` cannot directly extract and display a single condition's status from an array of conditions, which limits usability and clarity.

With CEL-based `additionalPrinterColumns`, I can define a column using an expression that filters and selects the relevant condition, making the output more meaningful.

**Example:**

Using the following CRD snippet, I define a `READY` column that uses a CEL expression to extract the status of the "Ready" condition:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
...
spec:
  ...
  versions:
    ...
    schema:
      openAPIV3Schema:
        type: object
        properties:
          status:
            type: object
            properties:
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
  ...
  additionalPrinterColumns:
    - name: READY
      type: string
      description: 'Status of the Ready condition'
      expression: 'self.status.conditions.exists(c, c.type == "Ready") ? self.status.conditions.filter(c, c.type == "Ready")[0].status : "Unknown"'
```

```
output
```

This expression checks if a condition with `type == "Ready"` exists. If so, it returns its status; otherwise, it returns `"Unknown"`. This approach enables clear, user-friendly status reporting for conditions stored as arrays in the CRD.

**References:**

* https://github.com/kubernetes/kubernetes/issues/67268


#### Story 3

As a Kubernetes user, I want to define an additional printer column that combines multiple fields from a sub-resource into a single human-readable string. The `additionalPrinterColumns` columns defined using `jsonPath` can’t concatenate fields, so the output is either limited or unclear.

With CEL expressions in `additionalPrinterColumns`, it is possible to format and combine multiple fields cleanly for better readability.

For example, in a CRD with `.spec.sub.foo` and `.spec.sub.bar`, this column defined using CEL expression combines the two fields with a slash:

```yaml
additionalPrinterColumns:
- name: "Combined"
  type: string
  description: "Combined Foo and Bar values"
  expression: 'format("%s/%s", self.spec.sub.foo, self.spec.sub.bar)'
```

```
output
```

This shows output like `val1/val2` in `kubectl get` columns, improving clarity.

**References:**

* https://github.com/operator-framework/operator-sdk/issues/3872

#### Story 4

As a Kubernetes user, I want to format dates as relative durations (e.g., "5m ago" instead of absolute timestamps) in printer columns, making it easier to understand resource age or timing at a glance.

**Example:**

```yaml
additionalPrinterColumns:
  - name: Duration
    type: string
    description: Duration between start and completion
    expression: 'timestamp(self.status.completionTimestamp) - timestamp(self.status.startTimestamp)'
```

```
output
```

This would allow `kubectl get` to display the elapsed time between start and completion timestamps as a formatted duration.

**Reference:**

- https://stackoverflow.com/questions/70557581/kubernetes-crd-show-durations-in-additionalprintercolumns



### Notes/Constraints/Caveats (Optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

As of this writing, when defining `additionalPrinterColumns` in a CRD using **CEL expressions**, access to fields under `metadata` is **limited**.

Only `metadata.name` and `metadata.generateName` are supported, as per the current [design decision](https://github.com/kubernetes/kubernetes/blob/55f2bc10435160619d1ece8de49a1c0f8fcdf276/staging/src/k8s.io/apiextensions-apiserver/pkg/apiserver/schema/cel/model/schemas.go#L39-L73).

This makes CEL-based columns less flexible than those defined using `jsonPath`, which can access additional `metadata` fields like `creationTimestamp`, `labels`, and `ownerReferences`, etc.

For example, the following `jsonPath`-based configuration is valid and used in the [Cluster API project](https://github.com/kubernetes-sigs/cluster-api/blob/ef10e5aea3d3c9525dd83fa8a15005fc0b97d1b9/test/infrastructure/docker/config/crd/bases/infrastructure.cluster.x-k8s.io_devmachines.yaml#L19-L39):

```yaml
additionalPrinterColumns:
- name: Age
  type: date
  description: Time since creation
  jsonPath: .metadata.creationTimestamp
- name: Cluster
  type: string
  description: Associated Cluster
  jsonPath: .metadata.labels['cluster\.x-k8s\.io/cluster-name']
- name: Machine
  type: string
  description: Owning Machine
  jsonPath: .metadata.ownerReferences[?(@.kind=="Machine")].name
```

Attempting to define the same columns using CEL expressions fails because any field under `metadata` (except `metadata.name` and `metadata.generateName`) is dropped during the [conversion of the CRD structural schema to a CEL declaration](https://github.com/kubernetes/kubernetes/blob/71f0fc6e72d53d5caf50b1314ca4d754463117f0/staging/src/k8s.io/apiextensions-apiserver/pkg/apiserver/schema/cel/model/schemas.go#L26-L75):

```yaml
additionalPrinterColumns:
- name: Age
  type: date
  description: Time since creation
  expression: self.metadata.creationTimestamp  
```

**Error:** (TODO: Psaggu to ask Sreeram to provide the exact error, rework the examples.)

```
compilation failed: ERROR:: undefined field 'creationTimestamp'
```


### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->

#### Complex CEL expressions may impact compilation performance

With CEL-based `additionalPrinterColumns`, users may write highly complex expressions to fulfill specific use cases. These expressions can lead to longer compilation times or excessive compute cost during CRD creation.

**Mitigation:**

A finite CEL cost model is enforced, as is standard with other CEL-enabled features in Kubernetes. This model limits the computational cost during expression compilation. If a CEL expression exceeds the allowed cost, the compilation will timeout and fail gracefully.

For expressions that are within the cost limits but still slow due to complexity, the responsibility lies with the CRD author to keep or drop them.


#### Runtime evaluation errors despite successful compilation

CEL expressions are compiled during CRD creation but evaluated later during API usage, such as `kubectl get <resource>`. As a result, runtime data inconsistencies can cause evaluation errors even if compilation was successful.

For example, if a CEL expression references fields not present in a given Custom Resource instance—due to missing data, schema changes, or optional fields—the evaluation may fail.

**Mitigation:**

This behavior is aligned with how `jsonPath`-based `additionalPrinterColumns` currently function. If a `jsonPath` evaluation fails, an empty value is printed in the column.

The same strategy will be applied for CEL: evaluation failures will result in an empty column, and the underlying error will be logged. This ensures user experience remains consistent and resilient to partial data issues.


## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

We extend the `CustomResourceColumnDefinition` type by adding an `expression` field which takes CEL expressions as a string.

```go
type CustomResourceColumnDefinition struct {
	// ...
	expression string
}
```

```go
type CustomResourceColumnDefinition struct {
	// ...
	expression string `json:"expression,omitempty" protobuf:"bytes,7,opt,name=expression"`
}
```

We expect the additionalPrinterColumns of a CustomResourceDefinition to either have a `jsonPath` or an `expression` field. The validation would look like this:

```go
func ValidateCustomResourceColumnDefinition(col *apiextensions.CustomResourceColumnDefinition, fldPath *field.Path) field.ErrorList {
	// ...
	if len(col.JSONPath) == 0 && len(col.expression) == 0 {
		allErrs = append(allErrs, field.Required(fldPath.Child("JSONPath or expression"), "either JSONPath or CEL expression must be specified"))
	}

	if len(col.JSONPath) != 0 {
		if errs := validateSimpleJSONPath(col.JSONPath, fldPath.Child("jsonPath")); len(errs) > 0 {
			allErrs = append(allErrs, errs...)
		}
	}

	if len(col.expression) != 0 {
		// Placeholder method for the actual CEL validation that would happen at this stage.
		if errs := validateCELExpression(col.expression, fldPath.Child("expression")); len(errs) > 0 {
			allErrs = append(allErrs, errs...)
		}
	}

	return allErrs
}
```

For executing the CEL expression and printing the result would be done from `staging/src/k8s.io/apiextensions-apiserver/pkg/registry/customresource/tableconvertor/tableconvertor.go` like so. We'd also define our own `findResults` and `printResults` methods for computing the CEL expressions.

```go
func New(crdColumns []apiextensionsv1.CustomResourceColumnDefinition) (rest.TableConvertor, error) {
	// ...

	fetch baseEnv
	extend env

	for _, col := range crdColumns {
		ast := env.Compile(col.expression)

		if len(col.JSONPath == 0){
			// existing JSONPath code
		} else {
			celProgram, err := cel.NewProgram(ast)
			if err != nil {
				return c, fmt.Errorf("unrecognised column definition %q", col.expression)
			}
			c.additionalColumns = append(c.additionalColumns, celProgram)
		}
}
```

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*
The goal is to ensure that we don't accept enhancements with inadequate testing.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

[x] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

##### Prerequisite testing updates

<!--
Based on reviewers feedback describe what additional tests need to be added prior
implementing this enhancement to ensure the enhancements have also solid foundations.
-->

##### Unit tests

<!--
In principle every added code should have complete unit test coverage, so providing
the exact set of tests will not bring additional value.
However, if complete unit test coverage is not possible, explain the reason of it
together with explanation why this is acceptable.
-->

<!--
Additionally, for Alpha try to enumerate the core package you will be touching
to implement this enhancement and provide the current unit coverage for those
in the form of:
- <package>: <date> - <current test coverage>
The data can be easily read from:
https://testgrid.k8s.io/sig-testing-canaries#ci-kubernetes-coverage-unit

This can inform certain test coverage improvements that we want to do before
extending the production code to implement this enhancement.
-->

- `k8s.io/apiextensions-apiserver/pkg/apiserver/schema`: `<date>` - `<test coverage>`

##### Integration tests

<!--
Integration tests are contained in k8s.io/kubernetes/test/integration.
Integration tests allow control of the configuration parameters used to start the binaries under test.
This is different from e2e tests which do not allow configuration of parameters.
Doing this allows testing non-default options and multiple different and potentially conflicting command line options.
-->

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html
-->

- <test>: <link to test coverage>

##### e2e tests

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html

We expect no non-infra related flakes in the last month as a GA graduation criteria.
-->

- <test>: <link to test coverage>

### Graduation Criteria

<!--
**Note:** *Not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, [feature gate] graduations, or as
something else. The KEP should keep this high-level with a focus on what
signals will be looked at to determine graduation.

Consider the following in developing the graduation criteria for this enhancement:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Feature gate][feature gate] lifecycle
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc
definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)
or by redefining what graduation means.

In general we try to use the same stages (alpha, beta, GA), regardless of how the
functionality is accessed.

[feature gate]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Below are some examples to consider, in addition to the aforementioned [maturity levels][maturity-levels].

#### Alpha

- Feature implemented behind a feature flag
- Initial e2e tests completed and enabled

#### Beta

- Gather feedback from developers and surveys
- Complete features A, B, C
- Additional tests are in Testgrid and linked in KEP

#### GA

- N examples of real-world usage
- N installs
- More rigorous forms of testing—e.g., downgrade tests and scalability tests
- Allowing time for feedback

**Note:** Generally we also wait at least two releases between beta and
GA/stable, because there's no opportunity for user feedback, or even bug reports,
in back-to-back releases.

**For non-optional features moving to GA, the graduation criteria must include
[conformance tests].**

[conformance tests]: https://git.k8s.io/community/contributors/devel/sig-architecture/conformance-tests.md

#### Deprecation

- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag
-->

#### Alpha

- Feature implemented behind a feature flag
- Unit tests and integration tests completed and enabled
- Implement the new expression field for additionalPrinterColumns with an approximate CEL cost

#### Beta

- Gather feedback from developers and surveys
- Additional e2e tests added
- Benchmarking JSONPath execution and modify CEL cost if needed

#### GA

- Upgrade/downgrade e2e tests
- Scalability tests
- Allowing time for feedback

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

No change in how users upgrade/downgrade their clusters. This feature may remove
complexity by removing risk that a tightened validation on Kubernetes' part
does not break workflow.

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and nodes?
- How does an n-3 kubelet or kube-proxy without this feature available behave when this feature is used?
- How does an n-1 kube-controller-manager or kube-scheduler without this feature available behave when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

N/A

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/1194-prod-readiness.

The production readiness review questionnaire must be completed and approved
for the KEP to move to `implementable` status and be included in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.
-->

### Feature Enablement and Rollback

<!--
This section must be completed when targeting alpha to a release.
-->

###### How can this feature be enabled / disabled in a live cluster?

<!--
Pick one of these and delete the rest.

Documentation is available on [feature gate lifecycle] and expectations, as
well as the [existing list] of feature gates.

[feature gate lifecycle]: https://git.k8s.io/community/contributors/devel/sig-architecture/feature-gates.md
[existing list]: https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
-->

- [x] Feature gate (also fill in values in `kep.yaml`)
  - Feature gate name: `CRDAdditionalPrinterColumnCEL`
  - Components depending on the feature gate: `apiextensions-apiserver`, `kube-apiserver`
- [ ] Other
  - Describe the mechanism:
  - Will enabling / disabling the feature require downtime of the control
    plane?
  - Will enabling / disabling the feature require downtime or reprovisioning
    of a node?

###### Does enabling the feature change any default behavior?

<!--
Any change of default behavior may be surprising to users or break existing
automations, so be extremely careful here.
-->

No default behaviour will be changed since we still support additionalPrinterColumns with JSONPath.

###### Can the feature be disabled once it has been enabled (i.e. can we roll back the enablement)?

<!--
Describe the consequences on existing workloads (e.g., if this is a runtime
feature, can it break the existing applications?).

Feature gates are typically disabled by setting the flag to `false` and
restarting the component. No other changes should be necessary to disable the
feature.

NOTE: Also set `disable-supported` to `true` or `false` in `kep.yaml`.
-->

Yes, the feauter can be disabled once it has been enabled. Enabling this feature would let users define additionalPrinterColumns in their custom resource definitions with CEL expressions instead of JSONPath. Existing JSONPath support is untouched. If the feature is disabled, the existing additionalPrinterColumns with JSONPaths would work as expected. Resources with CEL expressions in their additionalPrinterColumn definition would be invalid if we disable the feature however.

###### What happens if we reenable the feature if it was previously rolled back?

Nothing. CRDs which had failed validation previously might now succeed if the CEL expression is valid.

###### Are there any tests for feature enablement/disablement?

<!--
The e2e framework does not currently support enabling or disabling feature
gates. However, unit tests in each component dealing with managing data, created
with and without the feature, are necessary. At the very least, think about
conversion tests if API types are being modified.

Additionally, for features that are introducing a new API field, unit tests that
are exercising the `switch` of feature gate itself (what happens if I disable a
feature gate after having objects written with the new field) are also critical.
You can take a look at one potential example of such test in:
https://github.com/kubernetes/kubernetes/pull/97058/files#diff-7826f7adbc1996a05ab52e3f5f02429e94b68ce6bce0dc534d1be636154fded3R246-R282
-->

We will add an integration test to ensure that the feature is disabled when the feature gate is off.

### Rollout, Upgrade and Rollback Planning

<!--
This section must be completed when targeting beta to a release.
-->

###### How can a rollout or rollback fail? Can it impact already running workloads?

<!--
Try to be as paranoid as possible - e.g., what if some components will restart
mid-rollout?

Be sure to consider highly-available clusters, where, for example,
feature flags will be enabled on some API servers and not others during the
rollout. Similarly, consider large clusters and how enablement/disablement
will rollout across nodes.
-->

This feature will not impact rollouts or already-running workloads.

###### What specific metrics should inform a rollback?

<!--
What signals should users be paying attention to when the feature is young
that might indicate a serious problem?
-->

###### Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?

<!--
Describe manual testing that was done and the outcomes.
Longer term, we may want to require automated upgrade/rollback tests, but we
are missing a bunch of machinery and tooling and can't do that now.
-->

No, a Kubernetes upgrade/downgrade operation is not expected to affect this feature.

###### Is the rollout accompanied by any deprecations and/or removals of features, APIs, fields of API types, flags, etc.?

<!--
Even if applying deprecation policies, they may still surprise some users.
-->

No.

### Monitoring Requirements

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### How can an operator determine if the feature is in use by workloads?

<!--
Ideally, this should be a metric. Operations against the Kubernetes API (e.g.,
checking if there are objects with field X set) may be a last resort. Avoid
logs or events for this purpose.
-->

###### How can someone using this feature know that it is working for their instance?

<!--
For instance, if this is a pod-related feature, it should be possible to determine if the feature is functioning properly
for each individual pod.
Pick one more of these and delete the rest.
Please describe all items visible to end users below with sufficient detail so that they can verify correct enablement
and operation of this feature.
Recall that end users cannot usually observe component logs or access metrics.
-->

- [ ] Events
  - Event Reason: 
- [ ] API .status
  - Condition name: 
  - Other field: 
- [x] Other (treat as last resort)
  - Details: Users will be able to define additionalPrinterColumns for their custom resources with `expression` instead of `jsonPath`.

###### What are the reasonable SLOs (Service Level Objectives) for the enhancement?

<!--
This is your opportunity to define what "normal" quality of service looks like
for a feature.

It's impossible to provide comprehensive guidance, but at the very
high level (needs more precise definitions) those may be things like:
  - per-day percentage of API calls finishing with 5XX errors <= 1%
  - 99% percentile over day of absolute value from (job creation time minus expected
    job creation time) for cron job <= 10%
  - 99.9% of /health requests per day finish with 200 code

These goals will help you determine what you need to measure (SLIs) in the next
question.
-->

###### What are the SLIs (Service Level Indicators) an operator can use to determine the health of the service?

<!--
Pick one more of these and delete the rest.
-->

- [ ] Metrics
  - Metric name:
  - [Optional] Aggregation method:
  - Components exposing the metric:
- [ ] Other (treat as last resort)
  - Details:

###### Are there any missing metrics that would be useful to have to improve observability of this feature?

<!--
Describe the metrics themselves and the reasons why they weren't added (e.g., cost,
implementation difficulties, etc.).
-->

No.

### Dependencies

<!--
This section must be completed when targeting beta to a release.
-->

###### Does this feature depend on any specific services running in the cluster?

<!--
Think about both cluster-level services (e.g. metrics-server) as well
as node-level agents (e.g. specific version of CRI). Focus on external or
optional services that are needed. For example, if this feature depends on
a cloud provider API, or upon an external software-defined storage or network
control plane.

For each of these, fill in the following—thinking about running existing user workloads
and creating new ones, as well as about cluster-level services (e.g. DNS):
  - [Dependency name]
    - Usage description:
      - Impact of its outage on the feature:
      - Impact of its degraded performance or high-error rates on the feature:
-->

No.

### Scalability

<!--
For alpha, this section is encouraged: reviewers should consider these questions
and attempt to answer them.

For beta, this section is required: reviewers must answer these questions.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### Will enabling / using this feature result in any new API calls?

<!--
Describe them, providing:
  - API call type (e.g. PATCH pods)
  - estimated throughput
  - originating component(s) (e.g. Kubelet, Feature-X-controller)
Focusing mostly on:
  - components listing and/or watching resources they didn't before
  - API calls that may be triggered by changes of some Kubernetes resources
    (e.g. update of object X triggers new updates of object Y)
  - periodic API calls to reconcile state (e.g. periodic fetching state,
    heartbeats, leader election, etc.)
-->

No.

###### Will enabling / using this feature result in introducing new API types?

<!--
Describe them, providing:
  - API type
  - Supported number of objects per cluster
  - Supported number of objects per namespace (for namespace-scoped objects)
-->

No.

###### Will enabling / using this feature result in any new calls to the cloud provider?

<!--
Describe them, providing:
  - Which API(s):
  - Estimated increase:
-->

No.

###### Will enabling / using this feature result in increasing size or count of the existing API objects?

<!--
Describe them, providing:
  - API type(s):
  - Estimated increase in size: (e.g., new annotation of size 32B)
  - Estimated amount of new objects: (e.g., new Object X for every existing Pod)
-->

No.

###### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?

<!--
Look at the [existing SLIs/SLOs].

Think about adding additional work or introducing new steps in between
(e.g. need to do X to start a container), etc. Please describe the details.

[existing SLIs/SLOs]: https://git.k8s.io/community/sig-scalability/slos/slos.md#kubernetes-slisslos
-->

Performance of CRD reads might be impacted. Benchmarking needs to be done to know the exact difference between using JSONPath and CEL.

###### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?

<!--
Things to keep in mind include: additional in-memory state, additional
non-trivial computations, excessive access to disks (including increased log
volume), significant amount of data sent and/or received over network, etc.
This through this both in small and large cases, again with respect to the
[supported limits].

[supported limits]: https://git.k8s.io/community//sig-scalability/configs-and-limits/thresholds.md
-->

Planning to benchmark this before beta.

###### Can enabling / using this feature result in resource exhaustion of some node resources (PIDs, sockets, inodes, etc.)?

<!--
Focus not just on happy cases, but primarily on more pathological cases
(e.g. probes taking a minute instead of milliseconds, failed pods consuming resources, etc.).
If any of the resources can be exhausted, how this is mitigated with the existing limits
(e.g. pods per node) or new limits added by this KEP?

Are there any tests that were run/should be run to understand performance characteristics better
and validate the declared limits?
-->

No.

### Troubleshooting

<!--
This section must be completed when targeting beta to a release.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.

The Troubleshooting section currently serves the `Playbook` role. We may consider
splitting it into a dedicated `Playbook` document (potentially with some monitoring
details). For now, we leave it here.
-->

###### How does this feature react if the API server and/or etcd is unavailable?

The same way any write to apiserver would.

###### What are other known failure modes?

<!--
For each of them, fill in the following information by copying the below template:
  - [Failure mode brief description]
    - Detection: How can it be detected via metrics? Stated another way:
      how can an operator troubleshoot without logging into a master or worker node?
    - Mitigations: What can be done to stop the bleeding, especially for already
      running user workloads?
    - Diagnostics: What are the useful log messages and their required logging
      levels that could help debug the issue?
      Not required until feature graduated to beta.
    - Testing: Are there any tests for failure mode? If not, describe why.
-->

None.

###### What steps should be taken if SLOs are not being met to determine the problem?

Disable the feature.

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->

## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

An alternative to the CEL approach proposed by this KEP would be to extend JSONPath to support arrays and other complex queries. There have been a couple of attempts to implement this previously.

- [Adding support for complex json paths in AdditionalPrinterColumns #101205](https://github.com/kubernetes/kubernetes/pull/101205)
- [apiextensions: allow complex json paths for additionalPrinterColumns #67079](https://github.com/kubernetes/kubernetes/pull/67079)

These attempts were not successful because of breaking changes to JSONPath. Now that we have CEL as an option, we can move away from trying to extend JSONPath and embrace CEL, since it covers a much larger ground than what we could achieve with extending JSONPath.

## Infrastructure Needed (Optional)

<!--
Use this section if you need things from the project/SIG. Examples include a
new subproject, repos requested, or GitHub details. Listing these here allows a
SIG to get the process for these resources started right away.
-->
