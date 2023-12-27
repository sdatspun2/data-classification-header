---
coding: utf-8
abbrev:
title: The Data-Classification HTTP Header Field
docname: draft-http-data-classification-header-latest
category: std

ipr: trust200902
area: Applications and Real-Time
workgroup: HTTPAPI
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

venue:
  group: "Building Blocks for HTTP APIs"
  type: "Working Group"
  home: "https://ietf-wg-httpapi.github.io/"
  mail: "httpapi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/httpapi/"

author:
  -
    ins: S. Dalal
    name: Sanjay Dalal
    email: sanjay.dalal@cal.berkeley.edu
    uri: https://github.com/sdatspun2
  -
    ins: D. Miller
    name: Darrel Miller
    email: darrel@tavis.ca
    uri: http://dret.net/netdret

normative:
  HTTP: RFC9110
  LINK: RFC8288
  HTTP-SIGN:
    title: Structured Field Values for HTTP
    date: 2023-11-06
    author:
      - ins: A. Backman
      - ins: J. Richer
      - ins: M. Sporny
    target: https://datatracker.ietf.org/doc/draft-ietf-httpbis-message-signatures/
  URI: RFC3986

informative:


--- abstract

The Data-Classification HTTP header field is used to signal to both producer and consumers of a URI-identified resource that HTTP Message contains data that requires handling according to the given data classification scheme. Additionally, the data-classification link relation can be used to link to a resource that provides additional information about the data classification scheme, and possibly describes procedures that producers and consumers of a resource can follow to handle data according to the classification requirements.

--- middle



# Introduction

Data-Classification HTTP header field ({{Section 3.1 of HTTP}}) communicates information about how to handle data according to given data classification scheme.

In addition to the Data-Classification header field, the resource provider can use the [Link] header field with specified relation type to convey additional information related to data classification. This can be information such as where to find documentation related to the data classification scheme, criteria used to classify data elements, procedure(s) to use the protect the data, among other things.

##  Notational Conventions

{::boilerplate bcp14-tagged}

This specification uses the Augmented Backus-Naur Form (ABNF) notation of {{!RFC5234}}.

The term "resource" is to be interpreted as defined in {{Section 3.1 of HTTP}}.

# The Data-Classification HTTP Header Field

The `Data-Classification` HTTP header field allows a client to convey to the server that the URI and/ or request payload contains data that requires handling according to given data classification scheme. Similarly, the `Data-Classification` HTTP header field enables the server to communicate to a client that the response payload contains data that requires handling according to the given data classification scheme.

## Syntax

The ABNF for the field value is:

~~~ abnf2616
  Data-Classification       = #link-value
  link-value = "<" URI-Reference ">" *( OWS ";" OWS link-param )
  link-param = token BWS [ "=" BWS ( token / quoted-string ) ]
~~~

Note that any link-param can be generated with values using either the token or the quoted-string syntax; therefore, recipients MUST be able to parse both forms. In other words, the following parameters are equivalent:

~~~
  x=y
  x="y"
~~~

Individual link-params specify their syntax in terms of the value after any necessary unquoting (as per {{!RFC7230}}, Section 3.2.6).

This specification establishes the link-param "anchor".

## Link Target

Each link-value conveys one target IRI as a URI-Reference (after conversion to one, if necessary; see {{!RFC3987}}, Section 3.1) inside angle brackets ("&lt;&gt;"). If the URI-Reference is relative, parsers MUST resolve it as per {{!RFC3986}}, Section 5. Note that any base IRI appearing in the message's content is not applied.

## Link Context {#header-context}

By default, the context of a link conveyed in the Data-Classification header field is the URL of the representation it is associated with, as defined in {{!RFC7231}}, Section 3.1.4.1, and is serialised as a URI.

When present, the anchor parameter overrides this with another {{!URI}}, such as a fragment of this resource, or a third resource (i.e., when the anchor value is an absolute URI). If the anchor parameter's value is a relative URI, parsers MUST resolve it as per {{!RFC3986}}, Section 5. Note that any base URI from the body's content is not applied.

The ABNF for the `anchor` parameter's value is:

~~~ abnf2616
  URI-Reference ; Section 4.1 of [RFC3986]
~~~

Client or server can choose to ignore links with an anchor parameter. For example, the application in use might not allow the link context to be assigned to a different resource. In such cases, the entire link is to be ignored; link applications MUST NOT process the link without applying the anchor.

Clients or Servers MUST NOT include more than one `Data-Classification` header field in the same request or response.

The following example indicates that given HTTP request or response contains data that is to have the most limited access and requires a high degree of integrity. This is typically data that will do the most damage to the organization should it be disclosed.:

    Data-Classification: https://example.org/privacy/sensitive

# The Data-Classification Link Relation Type

A server can also use links with the "data-classification" link relation type to communicate to the client where to find more information about data classification of the context. This approach can be used to make a data classification policy and data safeguarding instructions discoverable.

This specification places no restrictions on the representation of the linked data classification policy or safeguarding instructions. In particular, these may be available as human-readable documentation or as machine-readable description.

    Link: <https://example.org/privacy/safeguarding>;
          rel="data-classification"; type="text/html"

In this example the linked content provides additional information about data classification related to the resource context. Here, the resource
exposes a link where information is available regarding how data classification is managed for the resource context. This may be documentation explaining the use of the Data-Classification header field, and also explaining under which circumstances and with which policies classified data should be handled.


# IANA Considerations

## The Data-Classification HTTP Header Field

The `Data-Classification` header field should be added to the permanent registry of message header fields (see {{!RFC3864}}).

    Header Field Name: Data-Classification

    Applicable Protocol: Hypertext Transfer Protocol (HTTP)

    Status: Standard

    Author: Sanjay Dalal <sanjay.dalal@cal.berkeley.edu>,
            Darrel Miller <darrel@tavis.ca>

    Change controller: IETF

    Specification document: this specification, Section 2 "The Data-Classification HTTP Header Field"

## The Data-Classification Link Relation Type

The `data-classification` link relation type should be added to the permanent registry of link relation types ({{Section 4.2 of LINK}}).

    Relation Type: data-classification

    Applicable Protocol: Hypertext Transfer Protocol (HTTP)

    Status: Standard

    Author: Sanjay Dalal <sanjay.dalal@cal.berkeley.edu>,
            Darrel Miller <darrel@tavis.ca>

    Change controller: IETF

    Specification document: this specification, Section 3 "The Data-Classification Link Relation Type"

# Security Considerations

The content of the Data-Classification header field is not secure, private, or integrity-guaranteed. Use of Transport Layer Security (TLS) with HTTP {{!RFC2818}} is currently the only end-to-end way to provide these properties.

Applications ought to consider the attack vectors opened by automatically following, trusting, or otherwise using links gathered from HTTP header fields.

For example, Data-Classification header fields that use the “anchor” parameter to associate another resource cannot be trusted since they are effectively assertions by a third party that could be incorrect or malicious. Applications can mitigate this risk by specifying that such links should be discarded unless some relationship between the resources is established (e.g., they share the same authority).

Dereferencing links has a number of risks, depending on the application in use. For example, the Referer header {{!RFC7231}} can expose information about the application’s state (including private information) in its value. Likewise, cookies {{!RFC6265}} are another mechanism that, if used, can become an attack vector. Applications can mitigate these risks by carefully specifying how such mechanisms should operate.

The Data-Classification header field makes extensive use of IRIs and URIs. See {{!RFC3987}}, Section 8 for security considerations relating to IRIs. See {{!RFC3986}}, Section 7 for security considerations relating to URIs. See {{!RFC7230}}, Section 9 for security considerations relating to HTTP header fields.

HTTP permits and sometimes requires intermediaries to transform messages in a variety of ways. This can result in a recipient receiving a message that is not bitwise-equivalent to the message that was originally sent. Therefore, HTTP Messages carrying classified data MAY require message integrity and authenticity verification depending on the data classification scheme used. In such cases, relevant Message Component(s) including the Data-Classification header field SHOULD be protected using digital signature or similar mechanism using approaches described in [HTTP-SIGN] or similar.


# Examples

The following examples do not show complete HTTP interactions. They only show those HTTP header fields in a response that are relevant for data classification.

The first example shows a data classification header field classification for confidential data:

    Data-Classification: https://example.org/privacy/confidential

The second example shows a Link header field with a link describing safeguarding instructions for classified data.

    Link: <https://example.org/privacy/safeguarding>;
          rel="data-classification"; type="text/html"



--- back


# Acknowledgments


The authors would like to thank ... for their contributions.

The authors take all responsibility for errors and omissions.
