# ietf-pm-streaming

This repository hosts a family of three related Internet-Drafts on
YANG-based performance management (PM) collection and streaming.
The set was originally a single document, `draft-yoon-ccamp-pm-streaming`,
which was split into three documents so that the generic, transport-neutral
collection model could be reused independently of any specific transport
technology.

| Document | Module | Target | Status |
|---|---|---|---|
| [draft-yoon-ippm-collection-measure](draft-yoon-ippm-collection-measure.md) | `ietf-pm-collection` | IPPM WG | Individual, -00 |
| [draft-yoon-ippm-collection-interval-capabilities](draft-yoon-ippm-collection-interval-capabilities.md) | `ietf-pm-interval-capabilities` | IPPM WG | Individual, -00 |
| [draft-yoon-ccamp-pm-streaming](draft-yoon-ccamp-pm-streaming.md) | `ietf-pm-parameters` | CCAMP WG | Individual, submitted through -05 |

`draft-yoon-ccamp-pm-streaming` imports both `ietf-pm-collection` and
`ietf-pm-interval-capabilities`; the three documents and their YANG
modules are kept in one repository because of this direct dependency,
even though they target two different working groups.

## Repository layout

```
.
├── draft-yoon-ccamp-pm-streaming.md
├── draft-yoon-ippm-collection-measure.md
├── draft-yoon-ippm-collection-interval-capabilities.md
└── yang/
    ├── ietf-pm-parameters@2026-06-03.yang
    ├── ietf-pm-collection@2026-05-02.yang
    ├── ietf-pm-interval-capabilities@2026-06-11.yang
    └── deps/            # vendored external standard modules, for local
                          # validation only -- see yang/deps/README.md
```

## Building

The drafts are written in kramdown-rfc format. To build locally:

```
kramdown-rfc draft-yoon-ccamp-pm-streaming.md > draft-yoon-ccamp-pm-streaming.xml
xml2rfc draft-yoon-ccamp-pm-streaming.xml --text --html
```

To validate the YANG modules with `pyang`:

```
pyang -p yang -p yang/deps yang/ietf-pm-collection@2026-05-02.yang
pyang -p yang -p yang/deps yang/ietf-pm-interval-capabilities@2026-06-11.yang
pyang -p yang -p yang/deps yang/ietf-pm-parameters@2026-06-03.yang
```

## Versioning

Source files use the `-latest` convention rather than baking the
Internet-Draft version number into the filename; git history is the
record of change between versions. The official, citable version
history for each document lives on the IETF Datatracker. Specific
submitted versions are marked with git tags matching the datatracker
version (e.g. `draft-yoon-ccamp-pm-streaming-05`).
