---
layout: post
title: API Standards
tags: [ api ]
excerpt_separator: <!--more-->
author: primarilysoftware
---
As our team continues to build out a distributed micro-service based architecture, we need to ensure
some level of consistency for the APIs that become part of this ecosystem.  Having consistent API
standards will help to increase developer productivity while also helping to improve quality and
robustness of API implementations.

<!--more-->

Recognizing that our team is not and should not be experts in developing API design guidelines (we 
should focus our efforts on building expertise in the TMS domain), we have chosen to piggy back our
guidelines on the [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/).

Microsoft has devoted [significant time and effort](https://developer.microsoft.com/en-us/office/blogs/rest-api-design-guidelines/)
into the development of these guidelines, and continues to do so.  In addition to the guidelines
themselves, Microsoft provides libraries and tooling for ASP.NET to help implement these guidelines,
making it a natural choice for our team.

What follows is a summary of some of the areas from the Microsoft REST API Guidelines that our team
has chosen to emphasize.  For topics not explicitly clarified or expanded upon in this document, our
APIs should follow the Microsoft REST API Guidelines.

### Consistency Fundamentals

> Humans SHOULD be able to easily read and construct URLs

> Operations MUST use the proper HTTP methods whenever possible

> For non-success conditions, developers SHOULD be able to write one piece of code that handles errors consistently

Specifically, our team has adopted the [HTTP Problem Details](https://tools.ietf.org/html/rfc7807) format.

> All APIs compliant with the Microsoft REST API Guidelines MUST support explicit versioning. It's critical that
> clients can count on services to be stable over time, and it's critical that services can add features and make
> changes.

The guidelines allow for different versioning patterns.  Our team has settled on using an `api-version` parameter
in either the query string or as a header in the http request.

```
https://api.contoso.com/products?api-version=1.0
```

or

```
GET https://api.contoso.com/products
Host: api.contoso.com
api-version: 1.0
```

### Documentation
In addition to following the above guidelines when designing and building APIs, documentation must be provided.