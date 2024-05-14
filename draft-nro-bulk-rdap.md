%%%
Title = "Bulk RDAP"
abbrev = "bulk-rdap"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-01"
stream = "IETF"
status = "standard"
date = 2024-05-13T00:00:00Z

[[author]]
organization="Number Resource Organization"
[author.address]
email = "secretariat@nro.net"

%%%

.# Abstract

To complement the move from Whois to RDAP for the RIRs (Regional Internet Registries), this document specifies a new
service named the Bulk RDAP that an RIR can deploy to replace their Bulk Whois service.

{mainmatter}

# TODOs

* Metadata object followed by new-line separated RDAP objects to help with streaming
* Metadata -- a UUID string as versionId (in lieu of serial), productionDate with UTC offset, producer, objectCount, and
  rdapConformance with "nroBulkRdap1" extension id - DONE
* Add URL to get objects of all types
* Each RDAP object has its own rdapConformance member, listing all extensions used in its creation; a MUST - DONE
* Limit nested objected to the first level only - DONE
* Explain all possible content types, with the gzip recommended; create a table for readability
* Get JWK out-of-band
* Operational considerations -- daily, off-line generation with rationale (one-time JWS generation cost)
* No need to mention FTP; make HTTPS a MUST
* Make omission for nested objects a MUST - DONE
* Replace "replacement" with "counterpart"?

# Introduction

As the RIRs shift from Whois to RDAP, they also need an RDAP replacement for their Bulk Whois service. To that end, this
document specifies a new service named the Bulk RDAP that an RIR can deploy in lieu of the Bulk Whois. This service is
intended to be a simple, easy-to-implement replacement for the Bulk Whois.

At a higher level, the Bulk RDAP comprises JSON data for IP Network, Autonomous System Number, Domain, Nameserver, and
Entity object classes ([@!RFC9083, section 5]), plus some JSON metadata. It can be easily extended to include data for
any future RDAP object class. Furthermore, it is an HTTPS-based service that the RIR customers could use to securely get
this data.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119]
[@!RFC8174] when, and only when, they appear in all capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate element relationships, and are not a REQUIRED
feature of this service.

"..." in examples is used as shorthand for elements defined outside of this document.

# Bulk Data Format {#bulk_data_format}

The data returned for a Bulk RDAP request ((#bulk_rdap_url)) is a JSON object providing metadata information, followed
by newline-separated RDAP JSON objects for one or more RDAP object classes.

## Metadata

The following JSON members for metadata MUST be included:

* "extensionId" -- the "nroBulkRdap1" string
* "versionId" -- a version 4 Universally Unique IDentifier (UUID, [@!RFC9562, section 5.4]) string that MUST be the same
  for all the bulk data generated for various RDAP object classes at a particular interval (say, on a daily basis)
* "producer" -- a string identifying the registry that produced the bulk data, with possible values of "AFRINIC",
  "APNIC", "ARIN", "LACNIC", "RIPE NCC", or a string literal for a registry at the national or local level
* "productionDate" -- a string containing the date and time with the UTC offset of the producer when the bulk data being
  returned was produced
* "objectCount" -- a number to indicate the number of newline-separated RDAP objects following the metadata object

## Data

The metadata object is followed by newline-separated RDAP JSON objects for one or more RDAP object classes. Beside
adhering to the object class definition per [@!RFC9083, section 5], or to the specification of a future RDAP object
class, each returned object has the following characteristics:

* An "rdapConformance" member MUST be included to signify the RDAP extensions used to construct the object.
* The "self" links for the object, and for the nested objects it contains at the first level, MUST be included to have
  parity with the "ref" element in the Bulk Whois.
* For compactness, a nested object MUST NOT include any other data beside its "self" link, "handle" when defined, and
  relationship to the object (e.g., using the "roles" member in an Entity object to relate to an IP Network object).

## Example

Here is an elided example of a bulk data response for the IP Network object class:

```
{
  "extensionId": "nroBulkRdap1",
  "versionId": "3f8183db-1de6-4304-a0b3-e8df6c7ff1f2",
  "producer": "ARIN",
  "productionDate": "2024-05-14T04:00:00-04:00"
  "objectCount": 500000
}
{
  "rdapConformance":
  [
    "rdap_level_0",
    "nro_rdap_profile_0",
    "nroBulkRdap1",
    ...
  ],
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
  "rdapConformance":
  [
    "rdap_level_0",
    "nro_rdap_profile_0",
    "nroBulkRdap1",
    ...
  ],
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

_Figure 1_

# Extension Identifier

The "nroBulkRdap1" extension identifier (see (#rdap_extensions_registry)) MUST be included as the value for the
"extensionId" member in the metadata object of a bulk data response.

The "rdapConformance" array for each returned data object MUST include both the "nroBulkRdap1" extension identifier and
the extension identifier for the NRO RDAP Profile ("nro_rdap_profile_0" as of this writing). In the data objects, the
Bulk RDAP extension requirements MUST supersede the NRO RDAP Profile extension requirements, primarily to afford bulk
data compactness.

# Bulk RDAP URL {#bulk_rdap_url}

* Scheme: HTTPS
* Method: GET
* Path Segment: nroBulkRdap1?objectClass=<RDAP object class name>
* Returns: See (#content_types)

The HTTPS scheme MUST be used for the Bulk RDAP URL.

When the "objectClass" query parameter is present in the URL, the bulk data response MUST be limited to all the objects
for the RDAP object class listed in that query parameter.

When the "objectClass" query parameter is absent in the URL, the bulk data response MUST return all the objects for all
the RDAP object classes the server supports.

Here is an example URL to get bulk data for the IP Network object class:

```
https://example.net/nroBulkRdap1?objectClass=ip%20network
```

_Figure 2_

If the RDAP object class listed in the "objectClass" query parameter is valid ([@!RFC9083, section 5], or a future RDAP
object class) but the bulk data functionality has not been implemented for it, the server SHOULD return a 501 Not
Implemented response ([@!RFC9110, section 15.6.2]).

If the RDAP object class listed in the "objectClass" query parameter is invalid, the server SHOULD return a 400 Bad
Request response ([@!RFC9110, section 15.5.1]).

## Content Types {#content_types}

The content type for a bulk data response can be:

* "application/rdap+json" -- for a metadata JSON object followed by newline-separated RDAP objects
* "application/jose" -- for a JWS compact serialization of the metadata and data objects
* "application/json" -- for a JWS JSON serialization of the metadata and data objects
* "application/octet-stream" -- for a gzip of either the metadata and data objects or their JWS

It is RECOMMENDED that a bulk data response be returned as a gzip with "application/octet-stream" content type.

# Security Considerations {#security_considerations}

It is RECOMMENDED that JSON Web Signature (JWS) [@!RFC7515] and JSON Web Key (JWK) [@!RFC7517] be used to sign and
validate JSON data for the Bulk RDAP. It is further RECOMMENDED that Elliptic Curve Digital Signature Algorithm (ECDSA)
([@!RFC7518, section 3.4]) be used for JWS.

When the server uses JWS and JWK to sign JSON data, a JWS string is returned in the HTTP response for a Bulk RDAP
request. The client then verifies the JWS string before decoding the Base64URL-encoded payload for JSON data.

Furthermore, it is RECOMMENDED that the guidance from [@!RFC7481, section 3] be followed to secure the Bulk RDAP URL for
encryption, authentication, and authorization.

# Operational Considerations

It is NOT RECOMMENDED to make the RDAP bulk data available over FTP ([@RFC959]). Compared to HTTPS, FTP is generally
considered more complex to operate, less secure, and less firewall-friendly.

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

This work is influenced by the earlier RDAP Mirroring Protocol proposal ([@?I-D.harrison-regext-rdap-mirroring]),
especially for the serial number and JSON Web Signature ideas.

{backmatter}

# Other Potential Use Cases

## Escrow

Although there is presently no requirement for the RIRs to escrow their registration data, the JSON data format
described here could be used for that purpose in the future.
