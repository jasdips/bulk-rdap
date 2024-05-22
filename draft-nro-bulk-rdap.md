%%%
Title = "Bulk RDAP"
abbrev = "bulk-rdap"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-01"
stream = "IETF"
status = "standard"
date = 2024-05-22T00:00:00Z

[[author]]
organization="Number Resource Organization"
[author.address]
email = "secretariat@nro.net"

%%%

.# Abstract

To complement the move from Whois to RDAP for the Regional Internet Registries (RIRs), this document specifies a new
service, named the Bulk RDAP, that an RIR can deploy in lieu of its Bulk Whois service.

{mainmatter}

# Introduction

As the RIRs shift from Whois to RDAP, they also need an RDAP counterpart for their Bulk Whois service. To that end, this
document specifies a new service, named the Bulk RDAP, that an RIR can deploy to replace the Bulk Whois. This service is
intended to be a simple, easy-to-implement replacement.

At a higher level, the Bulk RDAP comprises JSON data for IP Network, Autonomous System Number, Domain, Nameserver, and
Entity object classes ([@!RFC9083, section 5]), plus some JSON metadata. It can be easily extended to include data for
any future RDAP object class. Furthermore, it is an HTTPS-based service that the RIR customers can use to securely get
this bulk data.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119]
[@!RFC8174] when, and only when, they appear in all capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate element relationships, and are not a REQUIRED
feature of this service.

"..." in examples is used as shorthand for elements defined outside of this document.

# Bulk Data Format {#bulk_data_format}

The data returned for a Bulk RDAP request ((#bulk_rdap_url)) is a sequence of JSON objects, each followed by a newline
('\n') character. Such a sequence is commonly known as [JSON Lines](https://jsonlines.org/), or newline-delimited JSON,
and makes bulk data streaming computationally efficient for the clients.
The first object in the returned sequence provides the metadata information for the bulk data, and the following objects
are data objects for one or more RDAP object classes ([@!RFC9083, section 5]).

## Metadata

The metadata JSON object MUST include the following members:

* "extensionId" -- the "nroBulkRdap1" string (see (#rdap_extensions_registry))
* "versionId" -- a version 4 Universally Unique IDentifier (UUID, [@!RFC9562, section 5.4]) string that MUST be the same
  for all the bulk data generated for various RDAP object classes at a particular interval (say, on a particular day)
* "producer" -- a string identifying the registry that produced the bulk data, with possible values of "AFRINIC",
  "APNIC", "ARIN", "LACNIC", "RIPE NCC", or an agreed-upon string literal for a registry at the national or local level
* "productionDate" -- a string containing the date and time with the UTC offset ([@!RFC3339]) of the producer,
  indicating when the bulk data being returned was produced
* "objectCount" -- a positive number (greater than 0) representing the count of data objects following the metadata
  object, to help clients detect a partial response for a bulk data request

## Data

The metadata object is followed by data objects for one or more RDAP object classes. Beside adhering to the object class
definition per [@!RFC9083, section 5] or the specification of a future RDAP object class, each returned object has the
following requirements:

* An "rdapConformance" member ([@!RFC9083, section 4.1]) MUST be included to indicate the RDAP extensions used for
  constructing the object.
* For data compactness, the object MUST include only its first-level nested objects and no more. For example, an
  entity object (representing an organization) can contain IP network and autonomous system number objects but not the
  entity objects for those IP network and autonomous system number objects.
* The "self" links ([@!RFC9083, section 4.2]) for the object and its first-level nested objects MUST be included. For
  example, include the "self" links for an entity object (representing an organization) and its IP network and
  autonomous system number objects.
* For further data compactness, a first-level nested object MUST NOT include any other data beside its "self" link,
  "handle" (representing a registry-unique identifier for the object) if defined, and relationship to the containing
  object if defined (for example, using the "roles" member in a nested entity object to relate to its containing IP
  network object).

## Sample Bulk Data

The following is an elided example of the bulk data generated on a particular day for the IP Network object class:

```
{
  "extensionId": "nroBulkRdap1",
  "versionId": "3f8183db-1de6-4304-a0b3-e8df6c7ff1f2",
  "producer": "ARIN",
  "productionDate": "2024-05-14T04:00:00-04:00"
  "objectCount": 500000
}
{
  "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
  "objectClassName": "ip network",
  "handle": "XXXX-ARIN",
  "startAddress": "2001:db8::",
  "endAddress": "2001:db8:0:ffff:ffff:ffff:ffff:ffff",
  "ipVersion": "v6",
  ...
  "links":
  [
    {
      "value": "https://example.net/ip/2001:db8::/48",
      "rel": "self",
      "href": "https://example.net/ip/2001:db8::/48",
      "type": "application/rdap+json"
    },
    ...
  ],
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "AAAA-ARIN",
      "roles": [ "administrative", ... ]
      "links":
      [
        {
          "value": "https://example.net/ip/2001:db8::/48",
          "rel": "self",
          "href": "https://example.net/entity/AAAA-ARIN",
          "type": "application/rdap+json"
        }
      ],
    },
    ...
  ]
}
{
  "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
  "objectClassName": "ip network",
  "handle": "YYYY-ARIN",
  "startAddress": "2001:db8:1::",
  "endAddress": "2001:db8:1:ffff:ffff:ffff:ffff:ffff",
  "ipVersion": "v6",
  ...
  "links":
  [
    {
      "value": "https://example.net/ip/2001:db8:1::/48",
      "rel": "self",
      "href": "https://example.net/ip/2001:db8:1::/48",
      "type": "application/rdap+json"
    },
    ...
  ],
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "BBBB-ARIN",
      "roles": [ "technical", ... ]
      "links":
      [
        {
          "value": "https://example.net/ip/2001:db8:1::/48",
          "rel": "self",
          "href": "https://example.net/entity/BBBB-ARIN",
          "type": "application/rdap+json"
        }
      ],
    },
    ...
  ]
}
...
```

The following is an elided example of the bulk data generated on the same day for all the RDAP object classes the RIR
supports:

```
{
  "extensionId": "nroBulkRdap1",
  "versionId": "3f8183db-1de6-4304-a0b3-e8df6c7ff1f2",
  "producer": "ARIN",
  "productionDate": "2024-05-14T04:30:00-04:00"
  "objectCount": 750000
}
{
  "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
  "objectClassName": "ip network",
  "handle": "XXXX-ARIN",
  "startAddress": "2001:db8::",
  "endAddress": "2001:db8:0:ffff:ffff:ffff:ffff:ffff",
  "ipVersion": "v6",
  ...
  "links":
  [
    {
      "value": "https://example.net/ip/2001:db8::/48",
      "rel": "self",
      "href": "https://example.net/ip/2001:db8::/48",
      "type": "application/rdap+json"
    },
    ...
  ],
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "AAAA-ARIN",
      "roles": [ "administrative", ... ]
      "links":
      [
        {
          "value": "https://example.net/ip/2001:db8::/48",
          "rel": "self",
          "href": "https://example.net/entity/AAAA-ARIN",
          "type": "application/rdap+json"
        }
      ],
    },
    ...
  ]
}
...
{
  "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
  "objectClassName": "autnum",
  "handle": "ZZZZ-ARIN",
  "startAutnum": 65536,
  "endAutnum": 65541,
  ...
  "links":
  [
    {
      "value": "https://example.net/autnum/65536",
      "rel": "self",
      "href": "https://example.net/autnum/65536",
      "type": "application/rdap+json"
    },
    ...
  ],
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "BBBB-ARIN",
      "roles": [ "technical", ... ]
      "links":
      [
        {
          "value": "https://example.net/autnum/65536",
          "rel": "self",
          "href": "https://example.net/entity/BBBB-ARIN",
          "type": "application/rdap+json"
        }
      ],
    },
    ...
  ]
}
...
```

# Extension Identifier

The "nroBulkRdap1" extension identifier (see (#rdap_extensions_registry)) MUST be included as the value for the
"extensionId" member in the metadata object of a bulk data response.

The "rdapConformance" array for each returned data object MUST include both the "nroBulkRdap1" extension identifier and
the extension identifier for
the [NRO RDAP Profile](https://bitbucket.org/nroecg/nro-rdap-profile/raw/v1/nro-rdap-profile.txt) ("nro_rdap_profile_0"
as of this writing). In the data objects, the Bulk RDAP extension requirements MUST supersede the NRO RDAP Profile
extension requirements, primarily to afford bulk data compactness.

# Bulk RDAP URL {#bulk_rdap_url}

* Scheme: HTTPS
* Method: GET
* Path Segment: nroBulkRdap1?objectClass=<RDAP object class name>
* Returns: See (#content_types)

The HTTPS scheme MUST be used for the Bulk RDAP URL.

When the "objectClass" query parameter is present in the request URL, the bulk data response MUST be limited to all the
objects for the RDAP object class listed in that query parameter.

When the "objectClass" query parameter is absent in the request URL, the bulk data response MUST return all the objects
for all the RDAP object classes the server supports.

The following is an example URL to get bulk data for the IP Network object class:

```
https://example.net/nroBulkRdap1?objectClass=ip%20network
```

If the RDAP object class listed in the "objectClass" query parameter is valid ([@!RFC9083, section 5] or a future RDAP
object class) but the bulk data functionality has not been implemented for it, the server SHOULD return a 501 Not
Implemented response ([@!RFC9110, section 15.6.2]).

If the RDAP object class listed in the "objectClass" query parameter is invalid, the server SHOULD return a 400 Bad
Request response ([@!RFC9110, section 15.5.1]).

The following is an example URL to get bulk data for all the RDAP object classes an RIR supports:

```
https://example.net/nroBulkRdap1
```

## Content Types {#content_types}

The content type for a bulk data response can be:

* "application/json" -- for a newline-delimited sequence of metadata JSON object and RDAP objects, or for JWS JSON
  serialization ([@!RFC7515, section 3.2]) of the metadata and data objects (see (#security_considerations))
* "application/jose" -- for JWS compact serialization ([@!RFC7515, section 3.1]) of the metadata and data objects (see
  (#security_considerations))
* "application/gzip" -- for gzip ([@RFC1952]) of either the metadata and data objects or their JWS

It is RECOMMENDED that a bulk data response be returned as gzip with the "application/gzip" content type.

# Security Considerations {#security_considerations}

It is RECOMMENDED that JSON Web Signature (JWS) [@!RFC7515] and JSON Web Key (JWK) [@!RFC7517] be used to sign and
validate the JSON data returned for a Bulk RDAP response. It is further RECOMMENDED that Elliptic Curve Digital
Signature Algorithm (ECDSA) ([@!RFC7518, section 3.4]) be used for JWS.

Furthermore, the guidance from [@!RFC7481, section 3] MUST be followed to secure the Bulk RDAP URL for authentication,
authorization, and encryption.

# Operational Considerations

If a server uses the JWS to secure bulk data, the related JWK is assumed to be distributed out-of-band to the clients.

Since bulk data generation and optionally signing it are considered computationally expensive, it is RECOMMENDED that
these operations be performed off-line and once a day.

# IANA Considerations

## RDAP Extensions Registry {#rdap_extensions_registry}

IANA is requested to register the following value in the RDAP Extensions Registry at
https://www.iana.org/assignments/rdap-extensions/:

* Extension identifier: nroBulkRdap1
* Registry operator: Regional Internet Registries (RIRs), including at national and local levels.
* Published specification: This document.
* Contact: Number Resource Organization (NRO) <secretariat@nro.net>
* Intended usage: This NRO-level extension describes version 1 of a method to access registration data in bulk from the
  RIRs through RDAP.

# Acknowledgements

The JSON Web Signature idea is borrowed from the RDAP Mirroring Protocol proposal
([@?I-D.harrison-regext-rdap-mirroring]). In the metadata object, using a UUID for the "versionId" field is influenced
by the RPKI Repository Delta Protocol (RRDP, [@RFC8182]), and the "producer" and "productionDate" fields by the NRO
Transfer Log Format.

{backmatter}

# Other Potential Use Cases

## Escrow

Although there is presently no requirement for the RIRs to escrow their registration data, the JSON data format
described here could be used for that purpose in the future.

# Change History

## Changes from 00 to 01

* A bulk data response is now a newline-delimited sequence of metadata JSON object and RDAP objects.
* Updated the metadata object definition to include the "extensionId", "versionId" (in lieu of "serial"), "producer",
  "productionDate", and "objectCount" fields.
* Each RDAP data object now has its own "rdapConformance" member to indicate the RDAP extensions used for its
  construction.
* Clarified requirements for the data objects to help clients consume bulk data consistently across all the RIRs.
* No more mention of FTP since the HTTPS scheme is now a must for the Bulk RDAP.
* Can now get bulk data for all the RDAP object classes an RIR supports.
* Clarified available content types for the bulk data and identified the preferred one.
* Updated the Operational Considerations section for out-of-band JWK distribution and off-line, daily generation of bulk
  data.
