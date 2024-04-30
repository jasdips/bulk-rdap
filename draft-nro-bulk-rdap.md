%%%
Title = "Bulk RDAP"
abbrev = "bulk-rdap"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-00"
stream = "IETF"
status = "standard"
date = 2024-04-30T00:00:00Z

[[author]]
organization="Number Resource Organization"
[author.address]
email = "secretariat@nro.net"

%%%

.# Abstract

To complement the move from Whois to RDAP for the RIRs (Regional Internet Registries), this document specifies a new
service named the Bulk RDAP that an RIR can deploy to replace their Bulk Whois service.

{mainmatter}

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

# Data Format {#data_format}

The data returned for a Bulk RDAP request ((#bulk_rdap_url)) is a JSON object comprising JSON members for metadata and
JSON data for a single RDAP object class.

## Metadata

The following JSON members for metadata MUST be included:

* "rdapConformance" -- An array of strings ([@!RFC9083, section 4.1]) to signal the RDAP extensions the data is based
  on.
* "serial" -- An unsigned 32-bit number for sequencing the data snapshots. It is RECOMMENDED to follow the serial
  number-wrapping arithmetic from [@!RFC1982].

## Data For Object Classes

The JSON data for an RDAP object class is an "objects" member -- a JSON array comprising all the objects for that class.
It is RECOMMENDED that an RIR provide bulk data for IP Network, Autonomous System Number, Domain, Nameserver, and Entity
object classes ([@!RFC9083, section 5]).

Furthermore, within the JSON array:

* The "self" links for each primary object and the secondary objects it contains MUST be included to have parity with
  the "ref" element in the Bulk Whois.
* For compactness, it is NOT RECOMMENDED to include details for a secondary object beside its "self" link, "handle" when
  defined, and relationship to the primary object (e.g., using the "roles" member in an Entity object to relate to an IP
  Network object).
* An "rdapConformance" member MAY be included at the top of a primary object if the RDAP extensions used to produce it
  are different from those listed in the "rdapConformance" member for metadata.

## Example

Here is an elided example of a JSON object containing bulk data for the IP Network object class:

```
{
  "rdapConformance":
  [
    "rdap_level_0",
    "nro_rdap_profile_0",
    "nroBulkRdap1",
    ...
  ],
  "serial": "12345",
  "objects":
  [
    {
      "objectClassName": "ip network",
      "handle": "XXXX-RIR",
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
          "handle": "AAAA-RIR",
          "roles": [ "administrative", ... ]
          "links":
          [
            {
              "value": "https://example.net/ip/2001:db8::/48",
              "rel": "self",
              "href": "https://example.net/entity/AAAA-RIR",
              "type": "application/rdap+json"
            }
          ],
        },
        ...
      ],
    },
    {
      "objectClassName": "ip network",
      "handle": "YYYY-RIR",
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
          "handle": "BBBB-RIR",
          "roles": [ "technical", ... ]
          "links":
          [
            {
              "value": "https://example.net/ip/2001:db8:1::/48",
              "rel": "self",
              "href": "https://example.net/entity/BBBB-RIR",
              "type": "application/rdap+json"
            }
          ],
        },
        ...
      ],
    },
    ...
  ]
}
```
_Figure 1_

# Extension Identifier

The "nroBulkRdap1" extension identifier (see (#rdap_extensions_registry)) MUST be included in the top-level
"rdapConformance" member of the JSON object for bulk data, to signal adherence to this specification.

It is RECOMMENDED that the extension identifier for the NRO RDAP Profile ("nro_rdap_profile_0") also be included in the
top-level "rdapConformance" member. But, when in conflict, the Bulk RDAP extension requirements SHOULD supersede the NRO
RDAP Profile extension requirements, primarily to afford bulk data compactness.

# Bulk RDAP URL {#bulk_rdap_url}

* Scheme: HTTPS
* Method: GET
* Path Segment: nroBulkRdap1?objectClass=<RDAP object class name>
* Returns: Bulk data as either JSON data ((#data_format)) or JSON Web Signature (JWS) string
  ((#security_considerations))
* Content-Type: application/rdap+json

Here is an example URL to get bulk data for the IP Network object class:

```
https://example.net/nroBulkRdap1?objectClass=ip%20network
```
_Figure 2_

For a 200 OK response ([@!RFC9110, section 15.3.1]), the server MUST return a JSON object with its "objects" member
filled with all the objects for the RDAP object class listed in the "objectClass" query parameter.

If the RDAP object class listed in the "objectClass" query parameter is valid ([@!RFC9083, section 5], or a future RDAP
object class) but the bulk data functionality has not been implemented for it, the server SHOULD return a 501 Not
Implemented response ([@!RFC9110, section 15.6.2]).

If the RDAP object class listed in the "objectClass" query parameter is invalid, the server SHOULD return a 400 Bad
Request response ([@!RFC9110, section 15.5.1]).

# Security Considerations {#security_considerations}

It is RECOMMENDED that JSON Web Signature (JWS) [@!RFC7515] and JSON Web Key (JWK) [@!RFC7517] be used to sign and
validate JSON data for the Bulk RDAP. It is further RECOMMENDED that Elliptic Curve Digital Signature Algorithm (ECDSA)
([@!RFC7518, section 3.4]) be used for JWS.

When JWS and JWK are used to sign JSON data, the JWS string is returned in the HTTP response for the Bulk RDAP request.
The client first verifies the JWS string and then decodes the Base64URL-encoded payload for JSON data.

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

Though this specification is intended to provide access to bulk data from the RIRs through RDAP, it may also be used for
the following potential use cases:

* Escrow: Although there is presently no requirement for the RIRs to escrow their registration data, the JSON data
  format described here could be used for that purpose in the future.
* Resource utilization data upload to an RDAP server: The RIR customers are typically required to report back on the
  utilization of their allocated IP addresses and autonomous system numbers to the RIR. A customer could locally run an
  RDAP server and upload the utilization data for its number resources in the form of RDAP objects. The JSON data format
  described here could be used for periodically uploading such data to a local RDAP server.
