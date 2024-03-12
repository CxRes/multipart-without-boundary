---
title: Multipart without Boundaries
category: std

docname: draft-gupta-mediaman-multipart-without-boundaries-latest
updates: 2046
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Media Type Maintenance"
keyword:
 - multipart
 - boundary
 - boundaries
 - no boundary
 - no boundaries
 - media-type
 - media type
 - content-type
 - content-type
venue:
  group: "Media Type Maintenance"
  type: "Working Group"
  mail: "media-types@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/media-types/"
  github: "CxRes/multipart-without-boundaries"
  latest: "https://CxRes.github.io/multipart-without-boundaries/draft-gupta-mediaman-multipart-without-boundaries.html"

author:
 -
    fullname: Rahul Gupta
    email: cxres+ietf@protonmail.com

autolink-iref-cleanup: true

normative:
  HTTP: RFC9110
  RFC2046:
informative:


--- abstract

This document extends the syntax of the `multipart` media-type, such that the encapsulated messages are not separated by a boundary delimiter. Not only is this syntax simpler to parse, it is safer to use when the encapsulated messages are not known in advance.


--- middle

# Introduction

The `multipart` media-type ({{RFC2046, Section 5.1}}) is the canonical format for message encapsulation for e-mail and HTTP ({{HTTP, Section 8.3.3}}).

The `multipart` media-type uses strings as boundary delimiters as a simple way to separate encapsulated messages. However, the use of boundary strings as separators in `multipart` media-type has two significant shortcomings:

1. A middleware/transformer that extracts parts must necessarily convert the entire convert the multipart message to a string and parse the contents in their entirety to discover the boundary separators. If the part body is not meant to be consumed as a string, the same needs to then be re-encoded. This burden only increases when multipart messages are nested, with the recipient having to check for boundary strings corresponding to each nested multipart message.

2. It is unsuitable for encapsulating messages where the contents are not known in advance. There is simply no way to ensure that a message generated in the future will not contain the chosen boundary string.

For this reason, we propose to extend the `multipart` media-type, replacing the use of boundary string as a part separator with the `Content-Length` header field to determine the length of each part.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Syntax

In lieu of the `boundary` parameter ({{RFC2046, Section 5.1.1}}), the `Content-Type` header field for multipart entities MAY instead contain a `no-boundary` parameter. The `no-boundary` parameter is a boolean which is true when defined, false otherwise.

If the `Content-Type` header specifies a `boundary` parameter, the `no-boundary` parameter MUST be ignored and the message shall be parsed as specified in {{Section 5.1.1 of RFC2046}}.

When the `no-boundary` parameter is defined (and the `boundary` parameter is not defined):

+ Each part (other than the exceptions specified below) MUST define a `Content-Length` header field. Other than being defined for each part of a multipart message, the `Content-Length` header field must be interpreted as defined in {{Section 8.3 of HTTP}}.
+ Each part of the multipart message, except the first, MUST be preceded by at least two line breaks (CRLF). The line breaks preceding a part MUST be ignored when calculating the content length.
+ The last message MUST include a `Content-Part` header field with its value set to `-1`.

The two exceptions where a part of the multipart message need not specify the `Content-Length` header field are:

1. In case the part is an empty last message that only includes the `Content-Part` header field set to `-1`.
1. In case the content type of the part is `multipart/*`. In other words, the part body is itself a nested multipart message.

{:aside}
> **Implementation Guidance:**
>
> A sender may terminate a multipart message by sending a part that contains only a `Content-Part` header field set to `-1` (followed by two line breaks) and no part body. In this case, the `Content-Part` header, acts just like the last boundary delimiter.

## Example

Consider the following example of a multipart message (adapted from {{RFC2046, Section 5.1.1}}) constructed using the original syntax:

~~~ http
From: Nathaniel Borenstein <nsb@bellcore.com>
To: Ned Freed <ned@innosoft.com>
Date: Sun, 21 Mar 1993 23:56:48 -0800 (PST)
Subject: Sample message
Content-type: multipart/mixed; boundary="simple boundary"

--simple boundary

This is implicitly typed plain US-ASCII text.
It does NOT end with a linebreak.
--simple boundary
Content-type: text/plain; charset=us-ascii

This is explicitly typed plain US-ASCII text.
It DOES end with a linebreak.

--simple boundary--
~~~

Here is the equivalent multipart message using the new syntax:

~~~ http
From: Nathaniel Borenstein <nsb@bellcore.com>
To: Ned Freed <ned@innosoft.com>
Date: Sun, 21 Mar 1993 23:56:48 -0800 (PST)
Subject: Sample message
Content-Type: multipart/mixed; no-boundary

Content-Length: 79

This is implicitly typed plain US-ASCII text.
It does NOT end with a linebreak.

Content-Length: 76
Content-Part: -1
Content-Type: text/plain; charset=us-ascii

This is explicitly typed plain US-ASCII text.
It DOES end with a linebreak.

~~~

## Error Handling

When the `Content-Type` header specifies a `no-boundary` parameter (and does not specify a `boundary` parameter), the recipient MUST fail parsing upon encountering a part that does not conform to the specified syntax and close the stream.

If a part received is incomplete as determined by the `Content-Length` at the moment when the response stream is closed, the aforementioned part MUST be ignored.

## Preamble and Epilogue

The original `multipart` media-type syntax allows for messages to contain information before the first boundary delimiter and after the final boundary delimiter, which is meant to be discarded by the recipient ({{RFC2046, Section 5.1.1}}). The syntax specified here does not allow for this possibility.

# Security Considerations

The security considerations that apply to the use of `multipart` media-type, are applicable here as well.

# IANA Considerations

This document has no IANA actions.


--- back
