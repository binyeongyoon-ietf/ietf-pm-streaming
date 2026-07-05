---
title: "A YANG Data Model for Collection Interval Capabilities"
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
 - performance management
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
    title: A YANG Data Model for Collection Measurement
    author:
      name: Bin Yeong Yoon
      org: ETRI
    date: 2026
    seriesinfo:
      Internet-Draft: draft-yoon-ippm-collection-measure-00
  G7710:
    title: Common Equipment Management Function Requirements
    author:
      org: ITU-T
    date: 2025-11
    target: https://www.itu.int/rec/T-REC-G.7710
    seriesinfo:
      ITU-T: Recommendation G.7710

informative:
  RFC6390:
  RFC9195:

--- abstract

This document defines a YANG data model, "ietf-pm-interval-capabilities",
that enables a server to advertise which collection intervals
it can support.  A client reads this capability information before
configuring performance measurements, so that it selects only sampling
and collection intervals that the server can honour.  The capabilities
are advertised by augmenting the "ietf-system-capabilities" module
defined in {{RFC9196}}, so that a client discovers them at the same
well-known location used for subscription and notification
capabilities.  The model imports the "profile-names" type from the
companion collection measurement model
defined in {{I-D.yoon-ippm-collection-measure}} and mirrors its
profile-and-parameter structure, ensuring direct alignment between
capability discovery and measurement configuration.  The model does not
define measurement data structures or any delivery mechanism; those are
defined in the companion document.

--- middle

# Introduction

The collection measurement data model defined in
{{I-D.yoon-ippm-collection-measure}} allows a client to configure sampling
and collection intervals for performance parameters within named
parameter profiles.  The same parameter may require different intervals
depending on its type and monitoring objective, and different network
elements support different ranges and granularities of intervals.
Without a prior discovery step, a client risks configuring interval
values that the server cannot honour, leading to configuration errors or
suboptimal monitoring.

This document defines a YANG data model {{RFC7950}},
"ietf-pm-interval-capabilities", for advertising the collection
collection interval capabilities a server supports.  A client reads
this information from the operational datastore before it configures
measurements.  The model mirrors the profile-and-parameter structure
of the companion collection measurement model and reuses its
"profile-names" type, so that capability discovery aligns directly with
measurement configuration.

Rather than defining a separate top-level container, this module
augments the "system-capabilities" container of the
"ietf-system-capabilities" module defined in {{RFC9196}}.  RFC 9196
provides a placeholder structure that other modules augment to expose
YANG-related system capabilities; the companion module
"ietf-notification-capabilities" (Section 3 of {{RFC9196}}) uses the
same anchor to advertise subscription and notification capabilities.
By augmenting the same anchor, this module lets a NETCONF or RESTCONF
client discover the measurement interval capabilities and the
subscription capabilities through a single, standardised query, without
prior knowledge of this module's own layout.

This document does not define measurement data structures, collection
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

This document and {{I-D.yoon-ippm-collection-measure}} together provide
the functional coverage that was originally combined in a single
document, draft-yoon-ccamp-pm-streaming.  The original document defined
both the PM collection data model and the interval capability model in
one place and focused on push-based streaming as the delivery mechanism.
The two documents have been separated to achieve cleaner separation of
concerns and to make the models delivery-mechanism neutral.

The functional division is as follows.

{{I-D.yoon-ippm-collection-measure}} defines "ietf-pm-collection", which
covers:

- Parameter profiles and PM parameter groupings
- Three collection types: Counts, Snapshot, and Tidemarks (per
  ITU-T G.7710 {{G7710}})
- Configurable sampling and collection intervals
- Threshold event notifications (periodic and non-periodic)
- Data structures accessible via pull-based retrieval (NETCONF, RESTCONF)
  or push-based subscription (YANG-Push)

This document defines "ietf-pm-interval-capabilities", which covers:

- Read-only advertisement of the sampling and collection interval
  ranges, units, defaults, and granularity that a server supports
- Hierarchical alignment with the companion model's profile-and-
  parameter structure
- Capability discovery prior to measurement configuration

The separation of concerns is:

- "What data is collected and how it is structured" is in
  {{I-D.yoon-ippm-collection-measure}}.
- "Which interval configurations the server can support" is in this
  document.

{{fig-document-relationship}} illustrates the relationship between
the two YANG modules.

~~~~ ascii-art
  +------------------------------------------+
  |     ietf-pm-interval-capabilities        |
  |         (this document)                  |
  |                                          |
  |  pm-interval-capabilities                |
  |    parameter-profile [profile-names] -+  |
  |      pm-parameter                     |  |
  |        interval-relationships         |  |
  |          sampling-interval            |  |
  |            collection-interval       |  |
  +------------------------------|----------+
                                 | imports profile-names
                                 v
  +------------------------------------------+
  |     ietf-pm-collection                   |
  |  (draft-yoon-ippm-collection-measure-00)    |
  |                                          |
  |  pm-periodic-collection                 |
  |    parameter-profile [profile-names]     |
  |      pm-parameter                        |
  |        sampling-interval                 |
  |          collection-interval            |
  |            collection-types              |
  |              counts / snapshot /         |
  |              tidemarks                   |
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
{{I-D.yoon-ippm-collection-measure}}, Errored Seconds (ES) sampled every
second may be aggregated over a 1-minute interval for rapid fault
detection in a Network Operations Center (NOC), over a 15-minute
interval for routine maintenance monitoring aligned with ITU-T G.7710
{{G7710}}, and over a 24-hour interval for daily QoS reporting -- all
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
a 100-millisecond sampling interval for ES, with collection intervals
as short as 1 minute and a 5-second granularity step.

Lower-specification network elements, constrained by processing
resources or hardware architecture, may support only longer minimum
intervals and coarser granularity steps.  The same ES parameter on
such a device might have a minimum sampling interval of 1 second and
a minimum collection interval of 15 minutes, adequate for traditional
operations and maintenance monitoring but insufficient for sub-minute
analytics.

{{tab-equipment-classes}} shows an example comparison for the ES
parameter.

| Equipment class | Min. sampling | Min. measurement | Granularity |
|-----------------|---------------|------------------|-------------|
| High-performance | 100 ms | 1 min | 5 s |
| Standard | 1 s | 1 min | 1 min |
| Low-specification | 1 s | 15 min | 15 min |
{:#tab-equipment-classes title="Example interval capabilities by equipment class (ES parameter)"}

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

# Structure and the Sampling/Measurement Relationship

The module follows a hierarchical structure that mirrors the measurement
configuration model defined in {{I-D.yoon-ippm-collection-measure}},
ensuring consistency between capability discovery and actual
configuration.  It has three levels: parameter profiles (collections of
related parameters such as "itu-transport-maintenance-15min"), PM
parameters (individual parameters such as ES, SES, BBE), and interval
capabilities (supported sampling and collection intervals for each
parameter).

The model defines a key relationship: a collection interval MUST be a
multiple of its corresponding sampling interval.  This ensures that
aggregation periods align with the data collection frequency.  For
example, if a server supports a 5-second sampling interval, valid
collection intervals are 5s, 10s, 15s, 30s, 60s, and so on.  The
relationship is expressed structurally by nesting collection intervals
within their corresponding sampling interval entry.

For each interval, the model advertises:

- min-value: minimum supported numeric value
- max-value: maximum supported numeric value
- units: list of supported time units (e.g., second, minute)
- default-value: recommended default numeric value
- default-unit: recommended default time unit
- granularity: step size; valid values must be multiples of this

Once a client knows the supported intervals, it can pair a single
supported sampling interval with several supported collection intervals
to serve different operational purposes for the same parameter.  The
"Use Cases" section of {{I-D.yoon-ippm-collection-measure}} gives concrete
examples of such combinations, which motivate the ranges and
granularities a server advertises here.

# Capability Discovery and Configuration Workflow

A client performs the following three steps to configure and access
collection measurements correctly.  Steps 2 and 3 use the companion
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
        |  Step 2: Configure measurement     |
        |  [ietf-pm-collection]              |
        |---NETCONF edit-config-----------> |
        |<--OK------------------------------ |
        |                                    |
        |  Step 3: Access data               |
        |  [ietf-pm-collection]              |
        |                                    |
        |  Option A: Pull (polling)          |
        |---NETCONF get-data--------------> |
        |<--collection-value--------------- |
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
maximum values, allowed units, and granularity.  This runtime exposure
follows the model described in {{RFC9196}} for advertising capabilities
in operational state.  The same information MAY also be published as
static instance data using the format defined in {{RFC9195}}.  The
interval ranges advertised here correspond to the Measurement Timing
aspect of the Performance Metric Specification described in
Section 5.4.2 of {{?RFC6390}}; this document provides the runtime
discovery of those timing constraints, which the companion model
{{I-D.yoon-ippm-collection-measure}} then applies when measurements are
configured.

**Step 2 -- Configure measurements (companion document).**
The client configures the "pm-periodic-collection" container defined
in "ietf-pm-collection" {{I-D.yoon-ippm-collection-measure}}, selecting
only sampling and collection interval values that Step 1 confirmed the
server supports.  This avoids configuration errors such as an
unsupported interval being rejected during collection setup or
subscription establishment.

**Step 3 -- Access data (companion document).**
The client accesses the collected PM data using either pull-based
retrieval or push-based subscription, both supported by "ietf-pm-
collection".  For pull-based access, the client issues NETCONF
"get-data" or RESTCONF GET operations against the operational
datastore to read current measurement values.  For push-based access,
the client establishes a YANG-Push subscription per {{!RFC8639}},
{{!RFC8641}}, {{!RFC8640}} to receive periodic or threshold-triggered
notifications.

# Interval Capabilities Example

{{fig-cap-example}} shows a NETCONF `<get>` request that retrieves
the interval capabilities for the ES parameter in the
"itu-transport-maintenance-15min" profile.

~~~~ xml
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0"
     xmlns:sysc=
       "urn:ietf:params:xml:ns:yang:ietf-system-capabilities"
     xmlns:ipc=
       "urn:ietf:params:xml:ns:yang:ietf-pm-interval-capabilities"
     message-id="301">
  <get>
    <filter>
      <sysc:system-capabilities>
        <ipc:pm-interval-capabilities>
          <parameter-profile>
            <name>itu-transport-maintenance-15min</name>
            <pm-parameter>
              <name>es</name>
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
                  </collection-interval>
                </sampling-interval>
              </interval-relationships>
            </pm-parameter>
          </parameter-profile>
        </ipc:pm-interval-capabilities>
      </sysc:system-capabilities>
    </filter>
  </get>
</rpc>
~~~~
{:#fig-cap-example title="Interval Capabilities Discovery Request Example"}

The response indicates that the server supports 1-second sampling and
a collection interval range of 5 to 1440 minutes with 5-minute
granularity, with a default of 15 minutes.  Using this information,
the client can configure the ES parameter in "ietf-pm-collection" with
any collection interval that is a multiple of 5 within the range
5 to 1440 minutes, knowing the server will accept the configuration.

# Tree Diagram

The following tree diagram, using the notation defined in {{RFC8340}},
shows the structure of the "ietf-pm-interval-capabilities" module.

~~~~ ascii-art
module: ietf-pm-interval-capabilities

  augment /sysc:system-capabilities:
    +--ro pm-interval-capabilities
       +--ro parameter-profile* [name]
          +--ro name            pm-coll:profile-names
          +--ro pm-parameter* [name]
             +--ro name                      string
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
~~~~
{:#fig-tree title="Tree Diagram of ietf-pm-interval-capabilities"}

# YANG Module {#yang-module}

~~~~ yang
{::include yang/ietf-pm-interval-capabilities@2026-06-11.yang}
~~~~
{: sourcecode-markers="true"
   sourcecode-name="ietf-pm-interval-capabilities@2026-06-11.yang"}

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
   Prefix:    ipc
   Reference: RFC XXXX

--- back

# Acknowledgments
{:numbered="false"}

The interval capability model in this document is derived from the
PM interval capabilities work originally defined in
draft-yoon-ccamp-pm-streaming, and has been separated into this
standalone document to complement the collection measurement model
in draft-yoon-ippm-collection-measure.
