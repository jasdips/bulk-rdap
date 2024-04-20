%%%
Title = "Bulk RDAP"
abbrev = "bulk-rdap"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-00"
stream = "IETF"
status = "standard"
date = 2024-04-19T00:00:00Z

[[author]]
organization="Number Resource Organization"
[author.address]
email = "secretariat@nro.net"

%%%

.# Abstract

To complement the move from Whois to RDAP, this document specifies a new service called Bulk RDAP that an RIR can deploy
in lieu of their Bulk Whois service.

{mainmatter}

# Introduction

As the RIRs shift from Whois to RDAP, they would also need an RDAP replacement for their Bulk Whois service. To that
end, this document specifies a new service named Bulk RDAP that an RIR can deploy in lieu of Bulk Whois. This service is
intended as a simple, easy-to-implement replacement for Bulk Whois, and leverages its files-based architecture.

Emulating Bulk Whois, Bulk RDAP comprises separate JSON data files by IP Network, Autonomous System Number, Domain, and
Entity object classes ([@!RFC9083, section 5]), or a single JSON file containing data for all these object classes, or
data for some but not all of these object classes (say, just IP Network and Entity objects) in a JSON file. An RIR would
publish such files for its customers to securely download and consume.

# JSON File For Single Object Class

In this case, a JSON file contains data for only one object class. It will be a JSON object containing a JSON array for
all the objects for that class. Additionally:

* Include an "rdapConformance" member at the top, and if needed within an object, to highlight which specifications that
  data adheres to.
* Include a "serial" string at the top to help sequence various files for similar data.
* Include "self" links for each primary object and the secondary objects it contains in order to have parity with the
  "ref" element in Bulk Whois.
* For compactness, do not include details for a secondary object beyond its "self" link, "handle" when defined, and
  relationship to the primary object (e.g., using the "roles" member in an Entity object to related to an IP Network
  object).

Here is an elided example of a JSON file containing only IP Network objects:

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
          }
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
    ]
  }
```

# JSON File for Multiple Object Classes

Such a JSON file would essentially be a JSON object contain JSON objects by each included object class.

Here is an elided example of a JSON file containing both IP Network and Autonomous System NUmber objects:

```
  {
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
            }
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
      ]
    },
    {
      "rdapConformance": [ "rdap_level_0", "nro_rdap_profile_0", "nroBulkRdap1", ... ],
      "serial": "12345",
      "objects":
      [
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
            }
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
  }
```

Note that there is some repetition of metadata for each object class' JSON data, but it is a reasonable trade-off for
implementation simplicity.

# Extension Identifier

Include the "nroBulkRdap1" extension identifier in the top-level "rdapConformance" member to signal adherence to this
specification.

Note how the extension identifier for the NRO RDAP Profile ("nro_rdap_profile_0" as of this writing) is included as
well. The Bulk RDAP extension supersedes the NRO RDAP Profile extension to help accommodate data compactness in JSON
files. For example, to selectively include data for secondary objects.

# Security Considerations

Use JSON Web Signature (JWS) [@!RFC7515] / JSON Web Key (JWK) [@RFC7517] to sign and validate JSON files. It is
RECOMMENDED that Elliptic Curve Digital Signature Algorithm (ECDSA) ([@RFC7518, section 3.4]) be used for signing and
validating such files.

{backmatter}
