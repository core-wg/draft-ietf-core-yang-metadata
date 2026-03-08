---
v: 3

title: Representing metadata annotations in YANG-CBOR
abbrev: Metadata Annotations in YANG-CBOR
category: std
consensus: true
stream: IETF
updates: 9595

docname: draft-bormann-cbor-yang-metadata-latest
number:
date:
area: ART (Applications and Real-Time)
workgroup: CBOR
keyword:
venue:
  mail: cbor@ietf.org
  github: cabo/yang-metadata

author:
  -
    ins: C. Bormann
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org


normative:
  RFC7952: md
  RFC9254: yang-cbor
  RFC9595: yang-sid
  STD94: cbor
  IANA.cbor-tags: tags
  RFC8610: cddl
  RFC9165: control1
  RFC9741: control2
  RFC9682: cddlupd

informative:
  I-D.ietf-cbor-edn-literals: edn

--- abstract

This specification defines the representation of metadata annotations
(RFC 7952) in YANG-CBOR (RFC 9254).

It updates RFC 9595 to add a separate namespace enabling the
representation of SIDs for metadata annotations in the ".sid" files
defined there.

--- middle

# Introduction

This specification defines the representation of metadata annotations
{{RFC7952}} in YANG-CBOR {{RFC9254}}.

It updates {{RFC9595}} to add a separate namespace enabling the
representation of SIDs for metadata annotations in the ".sid" files
defined there.

## Conventions and Definitions

{::boilerplate bcp14-tagged-bcp}

The term "CDDL" refers to the data definition language defined in
{{-cddl}} and its registered extensions (such as those in {{-control1}}
and {{-control2}}), as
well as {{-cddlupd}}.

Specific examples are notated in CBOR Extended
Diagnostic Notation (EDN), as originally introduced in {{Section 8 of
RFC8949@-cbor}} and extended in {{Appendix G of -cddl}}.
({{-edn}} more rigorously defines and further extends EDN.)



[^cpa]

[^cpa]: RFC-Editor: \\
      This document uses the CPA (code point allocation)
      convention described in [I-D.bormann-cbor-draft-numbers].  For
      each usage of the term "CPA", please remove the prefix "CPA"
      from the indicated value and replace the residue with the value
      assigned by IANA; perform an analogous substitution for all other
      occurrences of the prefix "CPA" in the document.  Finally,
      please remove this note.


The terms of {{-md}} and {{-yang-cbor}} apply.

# Specification

This section defines the metadata encoding for YANG-CBOR {{-yang-cbor}},
analogous to the Subsections for YANG-XML and YANG-JSON of {{Section 5 of -md}}.

{{Section 5.2.1 of -md}} defines a "Metadata Object" for YANG-JSON.
Analogously, the YANG-CBOR encoding of metadata annotations uses a
*Metadata Map*, which is identical in structure to the other CBOR maps
used in {{-yang-cbor}}.

Where YANG SIDs are used as the basis for the map keys for the metadata
map, the map's reference SID is the reference SID of the enclosing data
structure, as defined in {{Section 3.2 of -yang-cbor}}.
Where names ({{Section 3.3 of -yang-cbor}}) are used as the map keys for
the metadata map, they MUST be fully qualified, analogous to {{Section
5.2.1 of -md}}.

Metadata annotations are added to a data node instance by replacing
the representation of the instance ("`Instance-Representation`") with
the structure specified in CDDL in {{fig-metadata-tag}}:

~~~ cddl
annotated-data-node<Instance-Representation> = #6.109([ ; CPA109
  metadata-map,
  Instance-Representation
])
~~~
{: #fig-metadata-tag title="Metadata-Annotated Data Node"}

In essence, the `annotated-data-node` *stands in* for the
`Instance-Representation`; a consuming implementation that wants to
ignore all metadata received can simply replace each
`annotated-data-node` by the `Instance-Representation` embedded in it.

[^question]: (Editor's note:) QUESTION:

[^question] Do we need to represent metadata maps without the actual
instance representation present?  If yes, we could simply make the
second element of the array in {{fig-metadata-tag}} optional.

[^question] This representation assumes that it is good that metadata
always come before the actual data node, as would also be the case
with XML attributes.
{{Sections 5.2.3 and 5.2.4 of -md}} show examples with metadata last,
though.
Can we simply focus on one of these orders (always first, or always last), or do
we really need to support both (avoid!)?

# Examples

This section provides a number of examples, based on the examples in
{{Section 5.2 of -md}}; please see the descriptions of these examples
there.
Note that the examples here always show an enclosing map if needed; this
is generally elided in {{Section 5.2 of -md}} (which shows only map key and
map value separated by colon).

All but one example below use YANG SIDs ({{Section 3.2 of RFC9254}}).
For this, the examples assume the example SID assignments in
{{example-sids}}, the relevant ones of which are also repeated at the
start of each subsection:

| name                                |   SID |
| cask                                | 61600 |
| seq                                 | 61601 |
| name                                | 61602 |
| stuff                               | 61603 |
| example-last-modified:last-modified | 61610 |
| foo:flag                            | 61620 |
| bibliomod:folio                     | 61630 |
{: #example-sids title="Example SID values"}

For computing the outermost SID deltas, the examples assume the
reference SID is 61000.

TODO: Add explanatory text; add more examples.

## Examples from {{Section 5.2.2 of -md}} {#s522}

The examples here show that the map representing the instance
representation is not extended by a new member as in {{Section 5.2.2 of
-md}}, but is enclosed in an `annotated-data-node` structure like in
the other examples.

| name                                |   SID |
|-------------------------------------|-------|
| cask                                | 61600 |
| seq                                 | 61601 |
| name                                | 61602 |
| example-last-modified:last-modified | 61610 |
{: #s522-sids title="Example SID values for this section"}

"cask" is a container or anydata node:

~~~ cbor-diag
{
   600: /CPA/ 109([                      # SID: 61600
     {
       10: "2015-09-16T10:27:35+02:00"   # SID: 61610
     },
     ... # instance representation in its own map
   ])
}
~~~
{: #fig-cask title="Cask example"}

The same "cask" example with name-based CBOR maps ({{Section 3.3 of RFC9254}}):

~~~ cbor-diag
{
   "cask": /CPA/ 109([
     {
       "example-last-modified:last-modified":
         "2015-09-16T10:27:35+02:00"
     },
     ... # instance representation in its own map
   ])
}
~~~
{: #fig-cask-name title="Cask example with a name-based item identifier"}

"seq" is a list whose key is "name"; annotation "last-modified" is
added only to the first entry:

~~~ cbor-diag
{
   601: [                                # SID: 61601
     /CPA/ 109([
       {
          9: "2015-09-16T10:27:35+02:00" # SID: 61610
       },
       {
          1: "one",                      # SID: 61602
          ...: ...
       }
     ]),
     {                       # no metadata annotation
       1: "two",                         # SID: 61602
       ...: ...
     }
   ]
}
~~~
{: #fig-seq title="Seq example"}

## Examples from {{Section 5.2.3 of -md}}

| name                                |   SID |
| stuff                               | 61603 |
| example-last-modified:last-modified | 61610 |
| foo:flag                            | 61620 |
{: #s523-sids title="Example SID values for this section"}

"flag" is a leaf node of the "boolean" type defined in module "foo".
The SID 61620 for "foo:flag" expresses both the name "flag" and the
namespace name "foo" in its CBOR encoding:

~~~ cbor-diag
{
  620: 109([                             # SID: 61620
    {
      -10: "2015-09-16T10:27:35+02:00"   # SID: 61610
    },
    true
  ])
}
~~~
{: #fig-flag title="Foo:flag example"}


"stuff" is an anyxml node:

~~~ cbor-diag
{
  603: 109([                             # SID: 61603
    {
      7: "2015-09-16T10:27:35+02:00"     # SID: 61610
    },
    [1, null, "three"]
  ])
}
~~~
{: #fig-stuff title="Stuff example"}


## Examples from {{Section 5.2.4 of -md}}

| name                                |   SID |
| example-last-modified:last-modified | 61610 |
| bibliomod:folio                     | 61630 |

~~~ cbor-diag
{
  630: [                                  # SID: 61630
    6,                 # SIDs below: -20 -> SID: 61610
    /CPA/ 109([{-20: "2015-06-18T17:01:14+02:00"}, 3]),
    /CPA/ 109([{-20: "2015-09-16T10:27:35+02:00"}, 7]),
    8,
  ]
}
~~~
{: #fig-folio title="Bibliomod:folio example"}

## (Add more examples)

# SIDs for Metadata Annotations {#sec-sids}

The SID file format defined in {{-yang-sid}} is extended by adding
another alternative in the enumeration used in the leaf `namespace`:

~~~
leaf namespace {
  type enumeration {
    [...]
    enum annotation {
      value 4;
      description
        "The namespace for all names of metadata attributes, as
         defined in YANG-metadata [RFC7952].";
    }
  }
}
~~~

# Security Considerations

The security considerations of {{RFC7952}} and {{RFC9254}} apply.



# IANA Considerations

## CBOR Tags Registry

In the registry "CBOR Tags" {{-tags}}, IANA is requested to allocate one tag:

* Tag: CPA109
* Data item: Array `[metadata, ?data]`
* Semantics: "YANG data node with metadata annotations"
* Reference: This document



--- back

# Acknowledgments
{:unnumbered}

Andy Bierman brought up the need for this document.




