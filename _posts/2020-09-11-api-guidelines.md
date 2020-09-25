---
layout: post
title: API Standards
tags: [ api ]
excerpt_separator: <!--more-->
author: primarilysoftware
---
As our team continues to build out a distributed micro-service based architecture, we need to ensure
some level of consistency for the APIs that become part of this ecosystem.  Having consistent API
standards will help to improve quality and robustness of API implementations, while also helping to
improve developer productivity.

<!--more-->

Recognizing that our team is not and should not be experts in developing API design guidelines (we 
should focus our efforts on building expertise in the TMS domain), we have chosen to piggy back our
guidelines on the [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/).

Microsoft has devoted [significant time and effort](https://developer.microsoft.com/en-us/office/blogs/rest-api-design-guidelines/)
into the development of these guidelines, and continues to do so.  In addition to the guidelines
themselves, Microsoft provides libraries and tooling for ASP.NET Core to help implement these guidelines,
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
in either the query string or as a header in the http request:

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
While the above consistency fundamentals establish a solid foundation to build upon, our team recognizes
the role good documentation plays in successful API integrations.  What follows are guidelines specific
to documenting APIs that go beyond what is covered in the Microsoft REST API Guidelines.

> All APIs should expose a [Swagger/Open API](https://swagger.io/specification/) endpoint

Over recent years, swagger has emerged as the de facto standard for API documentation.  There is wide support
for swagger across languages and toolsets that our team can leverage to quickly generate swagger files
from source code.

> All APIs should expose an interactive UI to browse and test API endpoints such as
> [Swagger UI](https://swagger.io/tools/swagger-ui/)

As our application becomes more distributed, we are able to decouple API implementations from client applications.
This decoupling gives our team flexibility from a development and deployment perspective.  However, deploying API
implementations without an associated client application to test with can allow bugs to creep into our deployed
environments.  Having a tool such as Swagger UI at our disposal will ensure
that we always have a quick and easy way to independently test API implementations, helping to catch issues sooner
in the development process.

> Documentation for an API endpoint should include realistic sample requests and responses

Many endpoints, in particular endpoints for POSTing new records, can require relatively large JSON objects.
Having to manually craft valid JSON data for large objects can be tedious, and serve as a barrier to
testing.  Our API documentation should provide good sample requests with valid JSON data that can be used
to successfully invoke the associated API with little to no modifications.

> Documentation for an API endpoint should provide a good description

In addition to using URLs that are easy to read and construct, API documentation should
include a brief description of what each endpoint does.  While most developers would be able to
infer the intent of an endpoint from an HTTP Verb + URL (like `POST /api/shipments`), having plain english
description of the API endpoint can go a long way to making an API's capabilities discoverable to more
than just developers (prefer something like `POST /api/shipments - Create a new shipment`.
