---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Concise Software Identifier triple for Concise Module Identifiers"
abbrev: "CoSWID triple for CoMID"
category: std

docname: draft-ietf-rats-corim-swid-triple
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
ipr: trust200902
workgroup: "Remote ATtestation ProcedureS"
keyword:
 - RIM
 - RATS
 - Attestation
 - verifier
 - supply chain

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  email: henk.birkholz@ietf.contact
- ins: T. Fossati
  name: Thomas Fossati
  organization: Linaro
  email: Thomas.Fossati@linaro.org
- ins: Y. Deshpande
  name: Yogesh Deshpande
  organization: arm
  email: yogesh.deshpande@arm.com
- ins: N. Smith
  name: Ned Smith
  org: Intel
  email: ned.smith@intel.com
- ins: D. Glaze
  name: Dionna Glaze
  org: Apple, Inc.
  email: dionnaglaze@apple.com


normative:
  RFC9393: coswid
  IANA.coswid: coswid-reg
  I-D.ietf-rats-corim: corim


informative:

...

--- abstract

TODO Abstract


--- middle

# Introduction

The Concise Reference Integrity Manifest (CoRIM) specification permits including Concise Software Identification tags {{-coswid}}.
A CoSWID tag describes one or more software artifacts in terms of both metadata and content digests.
The Concise Module Identifier (CoMID) tag contains information about hardware, firmware, or module composition.
The CoMID tag's `triples-map` encodes assertions from the CoRIM author about Attesting or Target Enviroments such as security features and measurements.
The CoMID-CoSWID linking triple is the topic of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# CoSWID tag

A CoRIM may contain many CoSWID tags in its tags list.
Each CoSWID tag from its containing CoRIM is stored in the Appraisal Staging Area with its signing authorities.

~~~ cddl
{::include cddl/intrep-coswid-stage.cddl}
~~~

This CoSWID staging area will serve as the reference values for comparison with evidence.

# CoMID-CoSWID Linking Triple {#sec-comid-triple-coswid}

A CoSWID triple relates reference measurements contained in one or more CoSWIDs to a Target Environment.
The subject identifies a Target Environment.
The object is one or more unique tag identifiers of existing CoSWIDs.
The predicate asserts that the named CoSWID tags are expected for a given Target Environment.

~~~ cddl
{::include cddl/coswid-triple-record.cddl}
~~~

Thus for CoMID's CDDL,

~~~ cddl
$$triples-map-extension //= (
  ? &(coswid-triples: 6) => [ + coswid-triple-record ]
)
~~~

A CoMID that includes a CoMID-CoSWID triple is a statement that during Appraisal, the named Environment is expected to have evidence to confirm the asserted relationship.

## Internal representation for Appraisal Process

Each `coswid-triple-record` in an active CoMID is translated to an internal representation for a reference value.

ECT Type           | ECT Field    |  Requirement
---
CoSWID-reference   | environment  | Mandatory
                   | element-list | n/a
                   | authority    | Mandatory
                   | cmtype       | Mandatory
                   | profile      | Optional
                   | members      | n/a
                   | coswid-tags  | Mandatory

The `ECT` representation is extended with an extra field, `? &(coswid-tags: 6) => [ + ir-concise-swid-tag ]`.
The `cmtype` is `reference-values`.
If any of the referenced CoSWID ids are not known to the Verifier, the triple MUST be discarded.

The `profile` field of the ECT is set only if the CoRIM's profile has non-standard semantics to match CoSWID evidence against CoSWID reference.

# CoSWID evidence

Software Identification tags are meant to be flexible and are able to express a broad set of metadata about a software component.
Because of this, the complete details of the tag MAY not be measured by the Attesting Environment.
Instead, it is possible only the `evidence-record` from the CoSWID is collected for a given `environment-map`.
Any extra information in a CoSWID tag from a CoRIM is understood as endorsed properties of the evidence.

An `evidence-record` MAY not include enough information to have high confidence of file integrity, such as missing hash, location, or size.
The Appraisal Process section {{sec-comid-coswid-appraisal}} includes the strictest matching requirements, but Verifier Policy or CoRIM profile MAY reduce these requirements.

Evidence MAY come in different formats, such as Concise Evidence `application/ce` with a direct use of an `&(ev.coswid-triples: 6)` entry, or an IMA event log with an indirect representation of many `coswid-triple-record`s.

Suppose for IMA that every entry has either an `n` or `n-ng` field, and either an `d-ng` or `d-ngv2` field.
The IMA event log `environment-map` definition is out of scope for this document, but it should be defined in order to use `coswid-triples` for reference IMA values.

## Internal representation

For the Appraisal Process, both the CoSWID Evidence conceptual message and the CoSWID tags from CoRIM inputs MUST be translated to an internal representation to compare with the CoMID-CoSWID triples.

Concise Evidence that includes `&(ce.coswid-triples: 4) => [ + ev-coswid-triple-record ]` gets each `ev-coswid-triple-record` translated to

ECT Type        | ECT Field    | Requirement
---
CoSWID-addition | environment  | Mandatory
                | element-list | n/a
                | authority    | Mandatory
                | cmtype       | Mandatory
                | profile      | Optional
                | members      | n/a
                | coswid-tags  | Mandatory

The `cmtype` is `evidence`.
The `coswid-tags` field is populated as follows.

[DICE transformation Issue #17](https://github.com/ietf-rats-wg/draft-ietf-rats-evidence-trans/issues/17).

For each `ev-map` in `[ + ev-coswid-evidence-map ]`,

* Allocate a `coswid-tags` list entry, `e`.
* If `ev-map`.`ce.coswid-tag-id` is present, set `e.tag-id` to `ev-map`.`ce-coswid-tag-id`.
* If `ev-map`.`ce.authorized-by` is present, ignore it ([Issue #1](https://github.com/ietf-rats-wg/draft-ietf-rats-corim-swid-triple/issues/1)).
* Copy `ev-map`.`ce.coswid-evidence` to `e`.`evidence`.

# Appraisal Process {#sec-comid-coswid-appraisal}

The Appraisal Process is when the Verifier determines if the Evidence for some software artifacts within a given Target Environment correspond to the expected linked CoSWID tags.

The CoMID-CoSWID linking triple means that if there is either

-  evidence missing for the presence of software described by a named CoSWID tag,
-  evidence for extra software that is present without reference values from a named CoSWID tag,

then the linking triple is not satisfied.

[Issue #2](https://github.com/ietf-rats-wg/draft-ietf-rats-corim-swid-triple/issues/2)
This is the "complete inventory" interpretation of the linking triple.
An alternative interpretation is that these CoSWIDs are required of the Target Environment, whereas a corresponding CoMID-CoSWID triple in an xcorim would be interpreted as the CoSWIDs to explicitly deny.

Each CoSWID-addition ECT is expected to have a matching CoSWID-reference ECT.
CoSWID tag ids in evidence are used as a hint to the Verifer to quickly reject CoSWID-reference ECTs that don't contain all of the evidence's named CoSWID tag ids.
For the remaining candidate ECTs, **TODO**.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.

--- back

# Complete CDDL for CoMID-CoSWID triple

~~~ cddl
{::include cddl/dependencies.cddl}
{::include cddl/coswid-triple-record.cddl}
~~~

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
