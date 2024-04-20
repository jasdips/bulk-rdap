%%%
Title = "Bulk RDAP"
abbrev = "bulk-rdap"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-00"
stream = "IETF"
status = "standard"
date = 2024-04-20T00:00:00Z

[[author]]
organization="Number Resource Organization"
[author.address]
email = "secretariat@nro.net"

%%%

.# Abstract

To complement the move from Whois to RDAP for the RIRs (Regional Internet Registries), this document specifies a new
service, named Bulk RDAP, that an RIR can deploy in lieu of their Bulk Whois service.

{mainmatter}

# Introduction

As the RIRs shift from Whois to RDAP, they also need an RDAP replacement for their Bulk Whois service. To that end, this
document specifies a new service, named Bulk RDAP, that an RIR can deploy in lieu of Bulk Whois. This service is
intended to be a simple, easy-to-implement replacement for Bulk Whois.

At a higher level, Bulk RDAP comprises JSON data by IP Network, Autonomous System Number, Domain, and Entity object
classes ([@!RFC9083, section 5]), plus some JSON metadata. It can be easily extended to include data for any future RDAP
object class. Furthermore, it is an HTTPS-based service that the RIR customers could use to securely download and
consume this data.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119]
[@!RFC8174] when, and only when, they appear in all capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate element relationships, and are not a REQUIRED
feature of this service.

"..." in examples is used as shorthand for elements defined outside of this document.

# Data Format

The Bulk RDAP data is a JSON object comprising JSON members for metadata and JSON data for RDAP object classes.

## Metadata

The following JSON members for metadata MUST be included:

* "rdapConformance" -- An array of strings ([@!RFC9083, section 4.1]) to signal the RDAP extensions the data is based
  on.
* "serial" -- An unsigned 32-bit number for sequencing the data snapshots. It is RECOMMENDED to follow the serial
  number-wrapping arithmetic from [@!RFC1982].

## Data For Object Classes

The JSON data for RDAP object classes is an "objects" member that is a JSON array comprising all the objects for one or
more RDAP object classes. It is RECOMMENDED that an RIR provides bulk data for IP Network, Autonomous System Number,
Domain, and Entity object classes ([@!RFC9083, section 5]).

Furthermore, within the JSON array:

* The "self" links for each primary object and the secondary objects it contains MUST be included to have parity with
  the "ref" element in Bulk Whois.
* For compactness, it is NOT RECOMMENDED to include details for a secondary object beside its "self" link, "handle" when
  defined, and relationship to the primary object (e.g., using the "roles" member in an Entity object to relate to an IP
  Network object).
* An "rdapConformance" member MAY be included at the top of a primary object if the RDAP extensions used to produce it
  are different from those listed in the "rdapConformance" member for metadata.

Here is an elided example of a JSON object containing bulk data for both IP Network and Autonomous System Number object
classes:

```
{
  "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
  "serial": "12345",
  "objects":
  [
    {
      "objectClassName": "ip network",
      "handle": "XXXX-RIR",
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
      ...
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "YYYY-RIR",
          "roles": [ "administrative", ... ]
          "links":
          [
            {
              "value": "https://example.net/ip/2001:db8::/48",
              "rel": "self",
              "href": "https://example.net/entity/YYYY-RIR",
              "type": "application/rdap+json"
            }
          ],
        },
        ...
      ],
    },
    ...
    {
      "objectClassName": "autnum",
      "handle": "ZZZZ-RIR",
      "links":
      [
        {
          "value": "https://example.net/autnum/65537",
          "rel": "self",
          "href": "https://example.net/autnum/65537",
          "type": "application/rdap+json"
        },
        ...
      ],
      ...
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "YYYY-RIR",
          "roles": [ "administrative", ... ]
          "links":
          [
            {
              "value": "https://example.net/autnum/65537",
              "rel": "self",
              "href": "https://example.net/entity/YYYY-RIR",
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

# Extension Identifier

The "nroBulkRdap1" extension identifier (see (#rdap_extensions_registry)) MUST be included in the top-level
"rdapConformance" member of the JSON object for bulk data, to signal adherence to this specification.

It is RECOMMENDED to also include the extension identifier for the NRO RDAP Profile ("nro_rdap_profile_0") in the
top-level "rdapConformance" member. But, when in conflict, the Bulk RDAP extension requirements SHOULD supersede the NRO
RDAP Profile extension requirements, primarily to afford bulk data compactness.

# Bulk Data URL

* Scheme: HTTPS
* Method: GET
* Path Segment: nroBulkRdap1?objectClasses=<comma-separated string of RDAP object class names>
* Content-Type: application/rdap+json

Using comma (',') to delimit multiple object class names for the value of the "objectClasses" query parameter safely
assumes that there will be no commas in future RDAP object class names.

Here is an example URL to get bulk data for IP Network, Autonomous System Number, and Entity object classes:

```
https://example.net/nroBulkRdap1?objectClasses=ip network,autnum,entity
```

# Security Considerations

It is RECOMMENDED to use JSON Web Signature (JWS) [@!RFC7515] / JSON Web Key (JWK) [@!RFC7517] to sign and validate JSON
data. It is further RECOMMENDED that Elliptic Curve Digital Signature Algorithm (ECDSA) ([@!RFC7518, section 3.4]) be
used for JWS.

When JWS and JWK are used to sign JSON data, the JWS string is returned in the HTTP response for the download call. The
client first verifies the JWS string and then decodes the Base64URL-encoded payload for JSON data.

Furthermore, it is RECOMMENDED to follow the guidance from [@!RFC7481, section 3] to secure the bulk data URL for
encryption, authentication, and authorization.

# Operational Considerations

It is NOT RECOMMENDED to make the RDAP bulk data available over FTP ([@RFC959]). Compared to HTTPS, FTP is considered
more complex to operate, less secure, and less firewall-friendly.

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

Though this specification is intended to provide access to bulk data from the RIRs through RDAP, it MAY also be used for
the following potential use cases:

* Escrow: Although there is presently no formal requirement for the RIRs to escrow their registration data, the JSON
  data format described here could be used for that purpose in the future.
* Data ingestion for RDAP server: The RIR customers are typically required to report back on the utilization of their
  registered IP addresses and autonomous system numbers to the RIR. The customer could locally run an RDAP server and
  upload the utilization info for its number resources in the form of RDAP objects. The JSON data format described here
  could be used for ingesting such data onto their local RDAP server.
