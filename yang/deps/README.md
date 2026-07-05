# Vendored external YANG modules

The modules in this directory are **not defined by this repository's
drafts**. They are copies of standard, already-published IETF modules,
vendored here only so that `pyang`/`yanglint` can fully resolve imports
when validating `ietf-pm-interval-capabilities` and `ietf-pm-collection`
locally (see the `import` statements at the top of each module in the
parent `yang/` directory).

| File | Defined in |
|---|---|
| `ietf-system-capabilities@2022-02-17.yang` | RFC 9196 |
| `ietf-netconf-acm@2018-02-14.yang` | RFC 8341 |
| `ietf-yang-library@2019-01-04.yang` | RFC 8525 |
| `ietf-yang-types@2013-07-15.yang` | RFC 6991 |
| `ietf-inet-types@2013-07-15.yang` | RFC 6991 |
| `ietf-datastores@2018-02-14.yang` | RFC 8342 |

Source: [YangModels/yang](https://github.com/YangModels/yang), the
community-maintained mirror of standard YANG modules. Each module
carries its own copyright notice and redistribution terms under the
IETF Trust's Simplified BSD License.
