---
title: "A YANG Data Model for Performance Monitoring (PM) Interval Capabilities"
abbrev: "PM Interval Capabilities YANG"
category: std

docname: draft-yoon-ippm-collection-interval-capabilities-latest
submissiontype: IETF
number:
date:
consensus: false
v: 3
workgroup: IPPM Working Group
keyword:
 - performance monitoring
 - interval capabilities
 - capability advertisement
 - YANG

venue:
  github: "binyeongyoon-ietf/ietf-pm-streaming"
  latest: "https://binyeongyoon-ietf.github.io/ietf-pm-streaming/draft-yoon-ippm-collection-interval-capabilities.html"

author:
 -
    name: Bin Yeong Yoon
    org: ETRI
    email: byyun@etri.re.kr
 -
    name: Youngkil You
    org: woori-net
    email: young@woori-net.com

contributor:
  -
    name: Kwangkoog Lee
    org: KT
    email: kwangkoog.lee@kt.com
  -
    name: Jongyoon Shin
    org: SK Telecom
    email: jongyoon.shin@sk.com
  -
    name: Sungyong Nam
    org: LGU+
    email: sy.nam@lguplus.co.kr

normative:
  RFC6020:
  RFC6241:
  RFC7950:
  RFC8340:
  RFC8341:
  RFC8342:
  RFC8525:
  RFC9196:
  I-D.yoon-ippm-collection-measure:

informative:
  G7710:
    title: Common Equipment Management Function Requirements
    author:
      org: ITU-T
    date: 2025-11
    target: https://www.itu.int/rec/T-REC-G.7710
    seriesinfo:
      ITU-T: Recommendation G.7710
  RFC7799:
  RFC6390:
  RFC9195:

--- abstract

This document defines a YANG data model, "ietf-pm-interval-capabilities",
that enables a server to advertise which sampling and collection
intervals, measurement methods, and collection types it supports for
each performance monitoring (PM) parameter.  A client reads this
capability information before configuring PM collection, so that it
selects only values that the server can honour.  The capabilities
are advertised by augmenting the "ietf-system-capabilities" module
defined in {{RFC9196}}, so that a client discovers them at the same
well-known location used for subscription and notification
capabilities.  The model imports the "profile-names" and
"collection-type" types and the "performance-parameter" and
"measurement-method" identities from the companion PM collection
model defined in
{{I-D.yoon-ippm-collection-measure}} and mirrors its
profile-and-parameter structure, ensuring direct alignment between
capability discovery and collection configuration.  The model does
not define collection data structures or any delivery mechanism;
those are defined in the companion document.

--- note_Note_to_the_RFC_Editor

Please replace "RFC XXXX" with the RFC number assigned to this
document, and "RFC YYYY" with the RFC number assigned to
{{I-D.yoon-ippm-collection-measure}}, throughout this document and
in the YANG module, and remove this note.

--- middle

# Introduction

The PM collection data model defined in
{{I-D.yoon-ippm-collection-measure}} allows a client to configure sampling
and collection intervals for performance parameters within named
parameter profiles.  The same parameter may require different intervals
depending on its type and monitoring objective, and different network
elements support different ranges and granularities of intervals.
Without a prior discovery step, a client risks configuring interval
values that the server cannot honour, leading to configuration errors or
suboptimal monitoring.

This document defines a YANG data model {{RFC7950}},
"ietf-pm-interval-capabilities", for advertising the interval
capabilities a server supports.  A client reads
this information from the operational datastore before it configures
PM collection.  The model mirrors the profile-and-parameter structure
of the companion PM collection model and reuses its
"profile-names" type and "performance-parameter" identity, so that
capability discovery aligns directly with collection configuration.

In addition to interval ranges, the same discovery step advertises
which measurement methods {{?RFC7799}} the server can apply to each
parameter (active, passive, or hybrid). A parameter may be
measurable by more than one method on a given network element -- for
example, delay measured either by an active probe or by a hybrid
in-band technique -- and a client needs to know the available
methods before it configures a measurement or establishes a
subscription, so that it obtains data with the intended operational
meaning. Advertising the supported methods alongside the supported
intervals lets the client make both choices in a single discovery
step, using the companion configuration model
{{I-D.yoon-ippm-collection-measure}}.

The same discovery step also advertises which collection types
(counts, snapshot, and tidemarks) the server supports for each
parameter, so that a client knows which types it can enable before
it configures a parameter; a profile may also restrict a parameter to a subset of
the types, for example counts only.

Rather than defining a separate top-level container, this module
augments the "system-capabilities" container of the
"ietf-system-capabilities" module defined in {{RFC9196}}.  RFC 9196
provides a placeholder structure that other modules augment to expose
YANG-related system capabilities; the companion module
"ietf-notification-capabilities" (Section 3 of {{RFC9196}}) uses the
same anchor to advertise subscription and notification capabilities.
By augmenting the same anchor, this module lets a NETCONF or RESTCONF
client discover the PM interval capabilities and the
subscription capabilities through a single, standardised query, without
prior knowledge of this module's own layout.

This document does not define collection data structures, collection
types, threshold events, or any delivery mechanism.  All of those are
defined in the companion document {{I-D.yoon-ippm-collection-measure}}.

## Terminology

The terms "client", "server", "datastore", and "operational state" are
used as defined in {{RFC6241}} and {{RFC8342}}.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in
all capitals, as shown here.

## Relationship to Companion Documents {#relationship}

The functional division between this document and
{{I-D.yoon-ippm-collection-measure}} is as follows.

{{I-D.yoon-ippm-collection-measure}} defines "ietf-pm-collection", which
configures PM collection and exposes the collected data.  This
document depends only on the types and identities that it imports
from that module.

This document defines "ietf-pm-interval-capabilities", which covers:

- Read-only advertisement of the sampling and collection interval
  ranges, units, defaults, and granularity that a server supports
- Read-only advertisement of the measurement methods and collection
  types that a server supports for each parameter
- Hierarchical alignment with the companion model's profile-and-
  parameter structure
- Capability discovery prior to collection configuration

The separation of concerns is:

- "What data is collected and how it is structured" is in
  {{I-D.yoon-ippm-collection-measure}}.
- "Which configurations the server can support" is in this
  document.

Specific performance parameters are defined as identities derived
from "performance-parameter" in other documents.

{{fig-document-relationship}} illustrates the relationship between
the two YANG modules.

~~~~ ascii-art
  +------------------------------------------+
  |     ietf-pm-interval-capabilities        |
  |         (this document)                  |
  |                                          |
  |  pm-interval-capabilities                |
  |    parameter-profile [profile-names] -+  |
  |      pm-parameter [identityref]       |  |
  |        supported-collection-types     |  |
  |        interval-relationships         |  |
  |          sampling-interval            |  |
  |            collection-interval        |  |
  +------------------------------|-----------+
                                 | imports profile-names,
                                 | performance-parameter,
                                 | measurement-method,
                                 | collection-type
                                 v
  +------------------------------------------+
  |     ietf-pm-collection                   |
  |     (companion document)                 |
  |     PM collection model                  |
  +------------------------------------------+
~~~~
{:#fig-document-relationship title="Relationship between ietf-pm-interval-capabilities and ietf-pm-collection"}

# Motivation

ITU-T G.7710 {{G7710}} does not include a clause that mandates a
capability-discovery mechanism for configurable measurement timing
parameters.  However, because sampling and collection intervals are
configurable, a client needs to know in advance which intervals a given
server can support, so that requests can be built without violating
implementation constraints.  This module provides that advertisement.

## Monitoring Objectives Require Different Intervals

The same PM parameter may need to be collected simultaneously at
multiple sampling and collection interval combinations, each serving
a distinct operational objective.  As illustrated by the use cases in
{{I-D.yoon-ippm-collection-measure}}, the number of lost packets counted
every second may be aggregated over a 1-minute interval for rapid fault
detection in a Network Operations Center (NOC), over a 15-minute
interval for routine maintenance monitoring, and over a 24-hour interval for daily QoS reporting -- all
three objectives active concurrently on the same parameter.  Similarly,
latency parameters may be sampled at sub-second granularity with short
collection intervals for digital twin synchronization, or at longer
intervals for AI/ML model training.

A client wishing to configure all of these combinations for all
parameters in a profile must first verify that the target server
supports every required interval.  Without a capability discovery
step, the client has no way to know whether a desired combination is
valid on the specific server, and may submit configurations that the
server silently truncates, rejects, or rounds to the nearest supported
value.

## Equipment Performance Determines Achievable Intervals

Beyond monitoring objective, the processing capacity of the network
element itself directly determines which interval configurations are
feasible for a given parameter.  Network elements vary significantly
in their measurement engines and hardware resources.

High-performance network elements with dedicated measurement processors
can support short minimum sampling intervals and fine-grained
collection interval steps, enabling high-resolution monitoring for
demanding use cases such as real-time fault detection, network digital
twins, and AI-driven analytics.  For example, such a device may support
a 100-millisecond sampling interval for the lost-packets parameter,
with collection intervals
as short as 1 minute and a 5-second granularity step.

Lower-specification network elements, constrained by processing
resources or hardware architecture, may support only longer minimum
intervals and coarser granularity steps.  The same lost-packets parameter on
such a device might have a minimum sampling interval of 1 second and
a minimum collection interval of 15 minutes, adequate for traditional
operations and maintenance monitoring but insufficient for sub-minute
analytics.

{{tab-equipment-classes}} shows an example comparison for the
lost-packets parameter.

| Equipment class | Min. sampling | Min. collection interval | Granularity |
|-----------------|---------------|---------------------------|-------------|
| High-performance | 100 ms | 1 min | 5 s |
| Standard | 1 s | 1 min | 1 min |
| Low-specification | 1 s | 15 min | 15 min |
{:#tab-equipment-classes title="Example interval capabilities by equipment class (lost-packets parameter)"}

A client operating across a heterogeneous network cannot assume a
uniform interval floor and must discover the capabilities of each
network element individually.  The interval capability model enables
a client to select the highest-resolution intervals a given server
supports for each operational objective, rather than applying a
conservative lowest-common-denominator configuration across all
devices.

## Multi-Vendor Interoperability

Equipment from different vendors supports different ranges and
granularities of intervals, and may impose implementation-specific
constraints on the relationship between sampling and collection
intervals.  In multi-vendor networks, clients must adapt to the
capabilities of each element.  Without a discovery mechanism, clients
risk configuration failures, suboptimal monitoring, or system
instability.  A standardised capability model addresses these
challenges and enables interoperability across diverse network
environments.

# Structure and the Sampling/Collection Relationship

The module follows a hierarchical structure that mirrors the
configuration structure of the PM collection model defined in
{{I-D.yoon-ippm-collection-measure}},
ensuring consistency between capability discovery and actual
configuration.  It has three levels: parameter profiles (collections of
related parameters such as "example-ip-qos-15min"),
PM parameters (individual parameters identified by identities
derived from "performance-parameter"), and capabilities
(supported sampling and collection intervals, measurement methods,
and collection types for each parameter).

The relationship between sampling and collection intervals is
expressed structurally by nesting collection intervals within their
corresponding sampling interval entry.

For each interval, the model advertises:

- min-value: minimum supported numeric value
- max-value: maximum supported numeric value
- units: list of supported time units (e.g., second, minute)
- default-value: recommended default numeric value
- default-unit: recommended default time unit
- granularity: step size; valid values must be multiples of this

For a collection interval, the model additionally advertises
`retained-interval-count`: the number of completed intervals the
server retains after they end.  This value bounds both how many
completed intervals a client may read back and, for push delivery,
how long a reporting period may be, since a reporting period may
span at most `retained-interval-count` collection intervals.  A
client uses it to choose a reporting period the server can honour.

For each parameter, the model additionally advertises
`supported-collection-types`: the collection types (counts,
snapshot, tidemarks) that the server can collect for the parameter.
A client enables only the advertised collection types.  An
absent list means the server does not advertise collection-type
information for the parameter.

Once a client knows the supported intervals, it can pair a single
supported sampling interval with several supported collection intervals
to serve different operational purposes for the same parameter.  The
"Use Cases" section of {{I-D.yoon-ippm-collection-measure}} gives concrete
examples of such combinations, which motivate the ranges and
granularities a server advertises here.

# Capability Discovery and Configuration Workflow

A client performs the following three steps to configure PM
collection and access the collected data correctly.  Steps 2 and 3 use the companion
model defined in {{I-D.yoon-ippm-collection-measure}}; Step 1 uses the
model defined in this document.

~~~~ ascii-art
  +-----------+                        +-----------+
  |  Client   |                        |  Server   |
  | (NMS/Ctrl)|                        |   (NE)    |
  +-----+-----+                        +-----+-----+
        |                                    |
        |  Step 1: Discover intervals        |
        |  [ietf-pm-interval-capabilities]   |
        |---NETCONF get-data--------------> |
        |<--pm-interval-capabilities-------- |
        |                                    |
        |  Step 2: Configure collection      |
        |  [ietf-pm-collection]              |
        |---NETCONF edit-config-----------> |
        |<--OK------------------------------ |
        |                                    |
        |  Step 3: Access data               |
        |  [ietf-pm-collection]              |
        |                                    |
        |  Option A: Pull (polling)          |
        |---NETCONF get-data--------------> |
        |<--collected data----------------- |
        |                                    |
        |  Option B: Push (YANG-Push)        |
        |---establish-subscription--------> |
        |<--periodic-notifications---------- |
        |                                    |
~~~~
{:#fig-workflow title="Combined Capability Discovery and Data Access Workflow"}

**Step 1 -- Discover interval capabilities (this document).**
On session establishment, the server's YANG Library {{RFC8525}} and
capability information indicate whether "ietf-pm-collection" and
"ietf-pm-interval-capabilities" are both supported.  The client
queries the "pm-interval-capabilities" container, which this module
augments into the "system-capabilities" container of
"ietf-system-capabilities" {{RFC9196}}.  Because that container is
"config false", the augmented nodes inherit "config false" and reside
in the operational datastore per the Network Management Datastore
Architecture (NMDA) {{RFC8342}}.  Querying the same anchor also returns
any subscription and notification capabilities advertised there by
"ietf-notification-capabilities" (Section 3 of {{RFC9196}}).
From it, the client retrieves the supported sampling and collection
intervals for each parameter and profile, including the minimum and
maximum values, allowed units, and granularity, together with the
supported measurement methods and collection types.  This runtime exposure
follows the model described in {{RFC9196}} for advertising capabilities
in operational state.  The same information MAY also be published as
static instance data using the format defined in {{RFC9195}}.  The
interval ranges advertised here correspond to the Measurement Timing
aspect of the Performance Metric Specification described in
Section 5.4.2 of {{?RFC6390}}; this document provides the runtime
discovery of those timing constraints, which the companion model
{{I-D.yoon-ippm-collection-measure}} then applies when collection is
configured.

**Step 2 -- Configure collection (companion document).**
The client configures PM collection using "ietf-pm-collection"
{{I-D.yoon-ippm-collection-measure}}, selecting
only sampling and collection interval values, measurement methods,
and collection types that Step 1 confirmed the server supports.  This
avoids configuration errors such as an unsupported interval or
collection type being rejected during collection setup or
subscription establishment.

**Step 3 -- Access data (companion document).**
The client accesses the collected PM data, by pull-based retrieval
or push-based subscription, as specified in
{{I-D.yoon-ippm-collection-measure}}.

# Interval Capabilities Example

{{fig-cap-example}} shows a NETCONF `<get-data>` request {{?RFC8526}}
that retrieves, from the operational datastore, the capabilities
advertised for the "ex:lost-packets" parameter in the
"example-ip-qos-15min" profile.  The subtree filter
selects the parameter by its list keys only, so the whole entry is
returned.  The parameter is identified by "ex:lost-packets", an
example identity derived from "performance-parameter".

~~~~ xml
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"
     message-id="301">
  <get-data
      xmlns="urn:ietf:params:xml:ns:yang:ietf-netconf-nmda"
      xmlns:ds="urn:ietf:params:xml:ns:yang:ietf-datastores">
    <datastore>ds:operational</datastore>
    <subtree-filter>
      <system-capabilities
          xmlns=
            "urn:ietf:params:xml:ns:yang:ietf-system-capabilities">
        <pm-interval-capabilities
            xmlns=
          "urn:ietf:params:xml:ns:yang:ietf-pm-interval-capabilities"
            xmlns:ex="urn:example:pm-parameters">
          <parameter-profile>
            <name>example-ip-qos-15min</name>
            <pm-parameter>
              <name>ex:lost-packets</name>
            </pm-parameter>
          </parameter-profile>
        </pm-interval-capabilities>
      </system-capabilities>
    </subtree-filter>
  </get-data>
</rpc>
~~~~
{:#fig-cap-example title="Interval Capabilities Discovery Request Example"}

{{fig-cap-reply}} shows the "pm-interval-capabilities" data returned
within "system-capabilities" in the reply.

~~~~ xml
<pm-interval-capabilities
    xmlns=
  "urn:ietf:params:xml:ns:yang:ietf-pm-interval-capabilities"
    xmlns:ex="urn:example:pm-parameters">
  <parameter-profile>
    <name>example-ip-qos-15min</name>
    <pm-parameter>
      <name>ex:lost-packets</name>
      <supported-collection-types>counts</supported-collection-types>
      <interval-relationships>
        <sampling-interval>
          <id>1s</id>
          <min-value>1</min-value>
          <max-value>1</max-value>
          <units>second</units>
          <default-value>1</default-value>
          <default-unit>second</default-unit>
          <granularity>1</granularity>
          <collection-interval>
            <id>collection-range</id>
            <min-value>5</min-value>
            <max-value>1440</max-value>
            <units>minute</units>
            <default-value>15</default-value>
            <default-unit>minute</default-unit>
            <granularity>5</granularity>
            <retained-interval-count>1</retained-interval-count>
          </collection-interval>
        </sampling-interval>
      </interval-relationships>
    </pm-parameter>
  </parameter-profile>
</pm-interval-capabilities>
~~~~
{:#fig-cap-reply title="Interval Capabilities Discovery Reply Data Example"}

The reply indicates that the server supports 1-second sampling and
a collection interval range of 5 to 1440 minutes with 5-minute
granularity, with a default of 15 minutes; that it supports the
counts collection type for the parameter;
and that it retains the most recently completed collection
interval.  Using this information, the client can configure the
"ex:lost-packets" parameter in "ietf-pm-collection" with any collection interval
that is a multiple of 5 within the range 5 to 1440 minutes, and
with the counts collection type, knowing the server will
accept the configuration.

When the server also advertises supported measurement methods for
the parameter, the reply additionally lists one or more
"supported-measurement-method" values (and optionally a
"default-measurement-method"), from which the client selects the
method to configure in "ietf-pm-collection".

# Tree Diagram

The following tree diagram, using the notation defined in {{RFC8340}},
shows the structure of the "ietf-pm-interval-capabilities" module.

~~~~ ascii-art
module: ietf-pm-interval-capabilities

  augment /sysc:system-capabilities:
    +--ro pm-interval-capabilities
       +--ro parameter-profile* [name]
          +--ro name            pm-co:profile-names
          +--ro pm-parameter* [name]
             +--ro name                            identityref
             +--ro supported-measurement-method*   identityref
             +--ro default-measurement-method?     identityref
             +--ro supported-collection-types*
             |       pm-co:collection-type
             +--ro interval-relationships
                +--ro sampling-interval* [id]
                   +--ro id                      string
                   +--ro min-value?              uint32
                   +--ro max-value?              uint32
                   +--ro units*                  interval-unit
                   +--ro default-value?          uint32
                   +--ro default-unit?           interval-unit
                   +--ro granularity?            uint32
                   +--ro collection-interval* [id]
                      +--ro id               string
                      +--ro min-value?       uint32
                      +--ro max-value?       uint32
                      +--ro units*           interval-unit
                      +--ro default-value?   uint32
                      +--ro default-unit?    interval-unit
                      +--ro granularity?     uint32
                      +--ro retained-interval-count?   uint32
~~~~
{:#fig-tree title="Tree Diagram of ietf-pm-interval-capabilities"}

# YANG Module {#yang-module}

~~~~ yang
{::include yang/ietf-pm-interval-capabilities.yang}
~~~~
{: sourcecode-markers="true"
   sourcecode-name="ietf-pm-interval-capabilities.yang"}

# Manageability Considerations

A server SHOULD ensure that the capabilities advertised in
"pm-interval-capabilities" remain consistent with its actual ability
to honour interval configurations in "ietf-pm-collection".  If a
server's resources change dynamically (e.g., due to load), it SHOULD
update the operational datastore accordingly, and clients that have
already configured measurements using previously advertised values
SHOULD be notified through standard NETCONF or YANG-Push mechanisms.

Operators SHOULD verify that the interval values configured in
"ietf-pm-collection" fall within the ranges advertised in this module
before deploying measurement configurations in production.

# Security Considerations

The YANG module defined in this document defines data nodes that are
designed to be accessed via network management protocols such as
NETCONF {{RFC6241}} or RESTCONF.  The module augments the
"system-capabilities" container of "ietf-system-capabilities"
{{RFC9196}}, which is "config false"; all nodes added by this module
therefore inherit "config false" and reside in the operational
datastore.  Following the guidance in {{RFC9196}}, this section
documents the security considerations of the augmented nodes.

The lowest NETCONF layer is the secure transport layer, and the
mandatory-to-implement secure transport is Secure Shell (SSH).  The
Network Configuration Access Control Model (NACM) {{RFC8341}} provides
the means to restrict access for particular NETCONF or RESTCONF users
to a preconfigured subset of all available NETCONF or RESTCONF
protocol operations and content.

Although all nodes are read-only, the capability information they
expose can reveal vendor-specific implementation details about the
equipment, such as supported interval ranges and granularities.
Operators SHOULD restrict read access to authorised management clients
to prevent unintended disclosure of device implementation
characteristics.

# IANA Considerations

This document requests IANA to register the following URI in the
"ns" subregistry within the "IETF XML Registry" {{?RFC3688}}:

   URI: urn:ietf:params:xml:ns:yang:ietf-pm-interval-capabilities
   Registrant Contact: The IESG.
   XML: N/A; the requested URI is an XML namespace.

This document requests IANA to register the following YANG module
in the "YANG Module Names" registry {{RFC6020}}:

   Name:      ietf-pm-interval-capabilities
   Namespace:
     urn:ietf:params:xml:ns:yang:ietf-pm-interval-capabilities
   Prefix:    pm-ic
   Reference: RFC XXXX

--- back
