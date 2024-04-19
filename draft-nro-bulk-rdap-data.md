%%%
Title = "Bulk RDAP Data"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "bulk-rdap-data"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-nro-bulk-rdap-data-00"
stream = "IETF"
status = "standard"
date = 2024-04-19T00:00:00Z

[[author]]
initials="J."
surname="Singh"
fullname="Jasdip Singh"
organization="ARIN"
[author.address]
email = "jasdips@arin.net"

%%%

.# Abstract

TODO

{mainmatter}

Emulating Bulk Whois, JSON bulk data files for IP Network, Autonomous System Number, Domain, and Entity objects

In each bulk data file:
* Include an "rdapConformance" member at the top, and if needed within an object, to highlight which specifications that data adheres to
* Include a serial number at the top
* Include "self" links for each primary object and the secondary objects it contains in order to have parity with the "ref" element in Bulk Whois
* For compactness, do not include details for a secondary object beyond its "self" link, "handle" when defined, and relationship to the primary object

Use JSON Web Signature (JWS) / JSON Web Key (JWK) to sign and validate bulk data files

{backmatter}
