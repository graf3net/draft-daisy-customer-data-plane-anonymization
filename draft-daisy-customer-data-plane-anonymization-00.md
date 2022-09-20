## Abstract

This document addresses potential privacy concerns for confidential classified (Swisscom, secret) customer data-plane metrics which became recently more important at Swisscom due the public cloud migration defined in [draft-daisy-imply-cloud-interim]. It focuses on data anonymization and pseudonymized techniques to obscure or make it impossible to perform data correlation between traffic and IP/MAC address accounting. Thus, discourage a potential attacker to establish the correlation between accounted traffic and personal customer data. Reduces further risk of reputational damage when data breach would occur.


## Scope

This document describes two data anonymization techniques, aggregation and pseudonymized and follows where possible existing IETF documents and proposes new IETF documents where needed and proposes their application within the data collection, message broker, post processing and data storage.


## Status of this memo

Request feedback from HCS DSY Ahmed Elhassany, Marco Tollini, Uwe Storbeck, Confluent Senad Jukic, Christoph Schubert, DNA ART Architect Philippe Göllnitz.


## Network Introduction

To make a customer network device reachable for IP traffic, a 64-bit MAC address is pre-provisioned on an ethernet interface. Through different kinds of protocols such as DHCP or IPCP, an IPv4/6 address is being allocated to that MAC address. Which IPv4/6 address is being assigned to which MAC address at which point in time is call IP address accounting throughout this document. This accounting information is key to establish the correlation between the service provider customer and traffic being accounted to that customer. That clarifies that although traffic accounting contains IPv4/6 address dimensions it does not correlate directly to the customer inventory data.

[RFC6235] describes in section 4.1 and 4.2 how to obscure the correlation between traffic and IP accounting by anonymizing IPv4/6 and MAC addresses in IPFIX. Even though this document only describes its IPFIX application, it hints well in section 6 how the [RFC6021] YANG data modelling would need to be extended to support data pseudonymization as well and how in [AVRO] data pseudonymization could be applied.

While IPFIX and YANG are being used to exchange data between network nodes and network data collection, YANG can be preserved in a Data Mesh environment as described in [draft-daisy-kafka-yang-integration]. Today, IPFIX forwarding-plane and BMP control-plane data models are being transformed to AVRO at the data collection. In the future AVRO will be replaced by YANG due to shortcomings in data-type support and to align with YANG data modelling.

[RFC7015] describes data aggregation and in section 5.1.7 of [RFC5102] the IE148 flowId entity to uniquely identify and [IE3 deltaFlowCount] to sum the amount of flows without exposing customer IPv4/6 and MAC address dimensions.

[RFC5476] describes the packet sampling process. Both data aggregation and sampling, particularly in combination, reduces amount of collected data, cardinality and therefore obscures the data transport visibility which in turn addresses potential privacy concerns.


## Data Classification

The [GDPR] Article 4 defines two relevant subjects.

~~~~
‘personal data’ means any information relating to an identified or identifiable natural person (‘data subject’); an identifiable natural person is one who can be identified, directly or indirectly, in particular by reference to an identifier such as a name, an identification number, location data, an online identifier or to one or more factors specific to the physical, physiological, genetic, mental, economic, cultural or social identity of that natural person;
~~~~

~~~~
pseudonymisation’ means the processing of personal data in such a manner that the personal data can no longer be attributed to a specific data subject without the use of additional information, provided that such additional information is kept separately and is subject to technical and organisational measures to ensure that the personal data are not attributed to an identified or identifiable natural person;
~~~~

[Wikipedia GDPR] describes Pseudonymisation and Tokenisation describes as two means to comply data privacy requirements. As of May 25, 2018 it explicitly describes in Article 6, Section 4 that data processing is allowed when pseudonymized. In Article 25 section 1, pseudonymisation and data minimization are described as two means to ensure data-protection.

As per [FMG] and GDPR law, customer data-plane metrics are classified confidential. Swisscom choses to increase classification from C3 (confidential) to C4 (secret). FMG allows to store customer data-plane metrics outside Switzerland if metrics are no longer classified personal related data which can be achieved with pseudonymisation and data minimization.


## Flow Aggregation without customer data-plane

Below [IPFIX IANA] and in AVRO schemata shows the forwarding-plane without customer data-plane IPv4/6/MAC address and BGP control-plane prefix mask dimensions data set.

~~~~
IE1 	octetDeltaCount
IE2 	packetDeltaCount
IE3 	deltaFlowCount
IE4 	protocolIdentifier
IE10 	ingressInterface
IE14 	egressInterface
IE21 	flowEndSysUpTime
IE22 	flowStartSysUpTime
IE89	forwardingStatus
IE90 	mplsVpnRouteDistinguisher
IE484 	bgpSourceCommunityList
IE485 	bgpDestinationCommunityList
IE487 	bgpSourceExtendedCommunityList
IE488 	bgpDestinationExtendedCommunityList
~~~~

~~~~
{
  "type": "record",
  "name": "acct_data",
  "fields": [
    {
      "name": "label",
      "type": {
        "type": "map",
        "values": "string"
      }
    },
    {
      "name": "comms",
      "type": "string"
    },
    {
      "name": "ecomms",
      "type": "string"
    },
    {
      "name": "peer_ip_src",
      "type": "string"
    },
    {
      "name": "comms_src",
      "type": "string"
    },
    {
      "name": "ecomms_src",
      "type": "string"
    },
    {
      "name": "iface_in",
      "type": "long"
    },
    {
      "name": "iface_out",
      "type": "long"
    },
    {
      "name": "mpls_vpn_rd",
      "type": "string"
    },
    {
      "name": "ip_proto",
      "type": "string"
    },
    {
      "name": "timestamp_min",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "timestamp_max",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "stamp_inserted",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "stamp_updated",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "packets",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "flows",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "bytes",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "writer_id",
      "type": "string"
    }
  ]
}
~~~~

## Flow Anonymization Encryption

[Crypto-PAn] is a pseudostandard for Prefix-Preserving Pseudonymization described in this [Crypto-PAn Thesis] from Georgia Institute of Technology. This would be applied for IPv4/6 addresses as described in section 4.1.4 and for MAC addresses as in section 4.2.4 in [RFC6235].

[cryptopANT] is a C library for Prefix-Preserving Pseudonymization based on the [Crypto-PAn Thesis]. The library supports anonymization and de-anonymization (provided you possess a secret key) of IPv4, IPv6, and MAC addresses. This library is independent (and not binary compatible) of the original [Crypto-PAn] implementation from Georgia Institute of Technology and is maintained by the ANT Lab research group spanning at the University of Southern California (USC). 

For completeness, but not be considered, we are also mentioning [FPE] which takes a 128-bit AES key and uses the [FFSEM] (Feistel Finite Set Encryption Mode) mode. FPE is used widely in e-commerce applications on the Internet and described in the [NIST privacy framework] in two modes [FF1 and FF3]. 

With [LLV-DAT-044], [LLV-DAT-045] and [LLV-DAT-046] we have three Swisscom GSE requirements on Encryption Algorithms, Key lengths and Block ciphers. [Crypto-PAn] fulfills the Swisscom GSE requirements on Encryption Algorithms and Key lengths by using AES encryption and a 256-bit key. The ECB block ciphers used do not meet Swisscom GSE requirements. CTR block cyphers are desired. To our current understanding by reviewing the [cryptopANT] source code, CTR block ciphers could be easily implemented since [cryptopANT] is using [OpenSSL] for encryption.

With [Network Data Anonymization Thesis] a thesis is available which analyzes the differences between [Crypto-PAn], [cryptopANT] and [AES-CTR].


## Key Distribution

The key being used for pseudonymization is stored in two data center redundant key stores on-prem and accessible through a anycast REST API URL from the data collection and the data visualization on-prem. In the key store multiple keys are bing stored. In the data collection a map file describes 

* the URL of the key store
* the encryption key id
* the encryption method

The configuration can be changed during runtime. These 3 dimensions will be augmented to the AVRO schema definition as described below, stored within the time series data base and being used when data being visualized. The key never leaves Swisscom premises and is never be stored with the data together. The key will be rotated in defined intervals.


## Data Visualization Access Rights

In the data visualization, 2 different access rights for searching and displaying of customer data-plane data needs to be distinguished and supported in the Data Visualization

* Users which have access to de-pseudonymized IPv4/6/MAC address dimensions
* Users which have access to pseudonymized IPv4/6/MAC address dimensions (default)

A company guideline needs to be stablished to define criteria's to be fulfilled to grant access to de-pseudonymized IPv4/6/MAC address dimensions.


## Data Visualization Access Impediments

Wildcard searches on IPv4/6/MAC address dimensions are no longer possible. Can be partially solved by supporting a prefix search in the IPv4/6 data type. In this case all possible IPv4/6 addresses matching the prefix needs to be pseudonymized and then searched. The bigger the mask the bigger the impact on compute.


## Flow Aggregation with L3VPN customer data-plane

Below [IPFIX IANA] and in AVRO schemata shows the forwarding-plane with L3VPN customer data-plane IPv4/6 and BGP control-plane prefix mask dimensions data set.

~~~~
IE1 	octetDeltaCount
IE2 	packetDeltaCount
IE3 	deltaFlowCount
IE4 	protocolIdentifier
IE5	ipClassOfService
IE6	tcpControlBits
IE7	sourceTransportPort
IE8 	sourceIPv4Address (pseudonymized)
IE9	sourceIPv4PrefixLength
IE10 	ingressInterface
IE11	destinationTransportPort
IE12 	destinationIPv4Address (pseudonymized)
IE13	destinationIPv4PrefixLength
IE14 	egressInterface
IE16	bgpSourceAsNumber
IE17	bgpDestinationAsNumber
IE21 	flowEndSysUpTime
IE22 	flowStartSysUpTime
IE27 	sourceIPv6Address (pseudonymized)
IE28 	destinationIPv6Address (pseudonymized)
IE29	sourceIPv6PrefixLength
IE30	destinationIPv6PrefixLength
IE44	sourceIPv4Prefix
IE45	destinationIPv4Prefix
IE70-75	mplsTopLabelStackSection
IE89	forwardingStatus
IE90 	mplsVpnRouteDistinguisher
IE169	destinationIPv6Prefix
IE170	sourceIPv6Prefix
IE225 	postNATSourceIPv4Address (pseudonymized)
IE226 	postNATDestinationIPv4Address (pseudonymized)
IE227 	postNAPTSourceTransportPort
IE228 	postNAPTDestinationTransportPort
IE230 	natEvent
IE484 	bgpSourceCommunityList
IE485 	bgpDestinationCommunityList
IE487 	bgpSourceExtendedCommunityList
IE488 	bgpDestinationExtendedCommunityList
~~~~

being pseudonymized in an AVRO schema, Complex Types are used to define

* IP address data type
* pseudonymized data type defining type of encryption, key id and key url

~~~~
{
  "name": "IPAddress",
  "namespace": "ietf.yang.types.ip",
  "type": "record",
  "fields": [
    {
      "name": "mac",
      "type": "ietf.yang.types.ip.address"
    },
    {
      "name": "encryption",
      "type": [
        "null",
        {
          "type": "record",
          "name": "cryptopANT",
          "fields": [
            {
              "name": "keyId",
              "type": "string"
            },
            {
              "name": "registryURL",
              "type": "string"
            }
          ]
        }
      ]
    }
  ]
}

{
  "type": "record",
  "name": "acct_data",
  "fields": [
    {
      "name": "label",
      "type": {
        "type": "map",
        "values": "string"
      }
    },
    {
      "name": "comms",
      "type": "string"
    },
    {
      "name": "ecomms",
      "type": "string"
    },
    {
      "name": "lcomms",
      "type": "string"
    },
    {
      "name": "as_path",
      "type": "string"
    },
    {
      "name": "peer_ip_src",
      "type": "string"
    },
    {
      "name": "comms_src",
      "type": "string"
    },
    {
      "name": "ecomms_src",
      "type": "string"
    },
    {
      "name": "lcomms_src",
      "type": "string"
    },
    {
      "name": "as_path_src",
      "type": "string"
    },
    {
      "name": "iface_in",
      "type": "long"
    },
    {
      "name": "iface_out",
      "type": "long"
    },
    {
      "name": "mpls_vpn_rd",
      "type": "string"
    },
    {
      "name": "ip_src",
      "type": "ietf.yang.types.ip.IPAddress"
    },
    {
      "name": "net_src",
      "type": "string"
    },
    {
      "name": "ip_dst",
      "type": "ietf.yang.types.ip.IPAddress"
    },
    {
      "name": "net_dst",
      "type": "string"
    },
    {
      "name": "mask_src",
      "type": "long"
    },
    {
      "name": "mask_dst",
      "type": "long"
    },
    {
      "name": "port_src",
      "type": "long"
    },
    {
      "name": "port_dst",
      "type": "long"
    },
    {
      "name": "tcp_flags",
      "type": {
        "type": "array",
        "items": "string"
      }
    },
    {
      "name": "fwd_status",
      "type": "string"
    },
    {
      "name": "mpls_label_stack",
      "type": {
        "type": "array",
        "items": "string"
      }
    },
    {
      "name": "ip_proto",
      "type": "string"
    },
    {
      "name": "tos",
      "type": "long"
    },
    {
      "name": "post_nat_ip_src",
      "type": "ietf.yang.types.ip.IPAddress"
    },
    {
      "name": "post_nat_ip_dst",
      "type": "ietf.yang.types.ip.IPAddress"
    },
    {
      "name": "post_nat_port_src",
      "type": "long"
    },
    {
      "name": "post_nat_port_dst",
      "type": "long"
    },
    {
      "name": "nat_event",
      "type": "long"
    },
    {
      "name": "tunnel_tcp_flags",
      "type": {
        "type": "array",
        "items": "string"
      }
    },
    {
      "name": "timestamp_start",
      "type": "string"
    },
    {
      "name": "timestamp_end",
      "type": "string"
    },
    {
      "name": "timestamp_arrival",
      "type": "string"
    },
    {
      "name": "timestamp_min",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "timestamp_max",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "custom_primitives",
      "type": {
        "type": "map",
        "values": "string"
      }
    },
    {
      "name": "stamp_inserted",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "stamp_updated",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "packets",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "flows",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "bytes",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "writer_id",
      "type": "string"
    }
  ]
}
~~~~


## Flow Aggregation with L2VPN customer data-plane

Below [IPFIX IANA] and in AVRO schemata shows the forwarding-plane with L2VPN customer data-plane IPv4/6/MAC data set.

~~~~
IE1 	octetDeltaCount
IE2 	packetDeltaCount
IE3 	deltaFlowCount
IE4 	protocolIdentifier
IE5	ipClassOfService
IE6	tcpControlBits
IE7	sourceTransportPort
IE8 	sourceIPv4Address (pseudonymized)
IE10 	ingressInterface
IE11	destinationTransportPort
IE12 	destinationIPv4Address (pseudonymized)
IE14 	egressInterface
IE21 	flowEndSysUpTime
IE22 	flowStartSysUpTime
IE27 	sourceIPv6Address (pseudonymized)
IE28 	destinationIPv6Address (pseudonymized)
IE56 	sourceMacAddress (pseudonymized)
IE80 	destinationMacAddress (pseudonymized)
IE351	layer2SegmentId
~~~~

being pseudonymized in an AVRO schema, Complex Types are used to define

* MAC address data type
* pseudonymized data type defining type of encryption, key id and key url

~~~~
{
  "name": "MACAddress",
  "namespace": "ietf.yang.types.mac",
  "type": "record",
  "fields": [
    {
      "name": "mac",
      "type": "ietf.yang.types.mac.address"
    },
    {
      "name": "encryption",
      "type": [
        "null",
        {
          "type": "record",
          "name": "cryptopANT",
          "fields": [
            {
              "name": "keyId",
              "type": "string"
            },
            {
              "name": "registryURL",
              "type": "string"
            }
          ]
        }
      ]
    }
  ]
}

{
  "type": "record",
  "name": "acct_data",
  "fields": [
    {
      "name": "label",
      "type": {
        "type": "map",
        "values": "string"
      }
    },
    {
      "name": "mac_src",
      "type": "ietf.yang.types.mac.address.MACAddress"
    },
    {
      "name": "mac_dst",
      "type": "ietf.yang.types.mac.address.MACAddress"
    },
    {
      "name": "peer_ip_src",
      "type": "string"
    },
    {
      "name": "iface_in",
      "type": "long"
    },
    {
      "name": "iface_out",
      "type": "long"
    },
    {
      "name": "ip_src",
      "type": "string"
    },
    {
      "name": "ip_dst",
      "type": "string"
    },
    {
      "name": "port_src",
      "type": "long"
    },
    {
      "name": "port_dst",
      "type": "long"
    },
    {
      "name": "tcp_flags",
      "type": {
        "type": "array",
        "items": "string"
      }
    },
    {
      "name": "ip_proto",
      "type": "string"
    },
    {
      "name": "tos",
      "type": "long"
    },
    {
      "name": "tunnel_tcp_flags",
      "type": {
        "type": "array",
        "items": "string"
      }
    },
    {
      "name": "vxlan",
      "type": "long"
    },
    {
      "name": "timestamp_start",
      "type": "string"
    },
    {
      "name": "timestamp_end",
      "type": "string"
    },
    {
      "name": "timestamp_arrival",
      "type": "string"
    },
    {
      "name": "timestamp_min",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "timestamp_max",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "custom_primitives",
      "type": {
        "type": "map",
        "values": "string"
      }
    },
    {
      "name": "stamp_inserted",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "stamp_updated",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "name": "packets",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "flows",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "bytes",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "name": "writer_id",
      "type": "string"
    }
  ]
}
~~~~


### Normative References

* [RFC6235] IP Flow Anonymization Support
  https://datatracker.ietf.org/doc/html/rfc6235

* [draft-daisy-kafka-yang-integration] Daisy Kafka YANG Integration Architecture
  https://git.swisscom.com/projects/DAISY/repos/documents/browse/draft-daisy-kafka-yang-integration-01.md

* [IE3 deltaFlowCount] IP Flow Information Export (IPFIX) Entities
  https://www.iana.org/assignments/ipfix/ipfix.xhtml

* [RFC7015] Flow Aggregation for the IP Flow Information Export (IPFIX) Protocol
  https://datatracker.ietf.org/doc/html/rfc7015

* [RFC5476] Packet Sampling (PSAMP) Protocol Specifications
  https://datatracker.ietf.org/doc/html/rfc5476

* [RFC6021] Common YANG Data Types
  https://datatracker.ietf.org/doc/html/rfc6021

* [AVRO] Apache Avro 1.11.1 Data Type Documentation
  https://avro.apache.org/docs/1.11.1/specification/_print/

* [IPFIX IANA] IP Flow Information Export Entities
  https://www.iana.org/assignments/ipfix/ipfix.xhtml

* [Crypto-PAn] Cryptography-based Prefix-preserving Anonymization
  https://en.wikipedia.org/wiki/Crypto-PAn

* [Crypto-PAn Thesis] On the Design and Performance of Prefix-Preserving IP Traffic Trace Anonymization
  http://conferences.sigcomm.org/imc/2001/imw2001-papers/69.pdf

* [cryptopANT]
  https://ant.isi.edu/software/cryptopANT/index.html

* [LLV-DAT-044] Swisscom GSE Requirements on Encryption Algorithms
  https://wiki.swisscom.com/display/public/SECFX/LLV-DAT-044

* [LLV-DAT-045] Swisscom GSE Requirements on Key lengths
  https://wiki.swisscom.com/display/public/SECFX/LLV-DAT-045

* [LLV-DAT-046] Swisscom GSE Requirements on Block ciphers
  https://wiki.swisscom.com/display/public/SECFX/LLV-DAT-046

* [AES-CTR] AES Block cipher mode of operation
  https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_(CTR)


### Informative References

* [GDPR] General Data Protection Regulation
  https://gdpr-info.eu/

* [FMG] Swisscom Telecommunication Act
  https://www.fedlex.admin.ch/eli/cc/1997/2187_2187_2187/en

* [Wikipedia GDPR] Pseudonymisation and Tokenisation
  https://en.wikipedia.org/wiki/General_Data_Protection_Regulation#Pseudonymisation
  https://en.wikipedia.org/wiki/Pseudonymization#New_Definition_for_Pseudonymization_Under_GDPR

* [FPE] Format Preserved Encryption
  https://en.wikipedia.org/wiki/Format-preserving_encryption

* [FFSEM] Feistel Finite Set Encryption Mode
  https://csrc.nist.gov/csrc/media/projects/block-cipher-techniques/documents/bcm/proposed-modes/ffsem/ffsem-spec.pdf

* [NIST privacy framework] The NIST Privacy Framework
  https://www.nist.gov/video/nist-privacy-framework-1

* [FF1 and FF3] NIST Recommendation for Block Cipher Modes of Operation
  https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-38G.pdf

* [draft-daisy-imply-cloud-interim] Daisy Imply Hybrid Cloud Interim Guideline
  https://git.swisscom.com/projects/DAISY/repos/documents/browse/draft-daisy-imply-cloud-interim-03.md

* [RFC5102] Information Model for IP Flow Information Export
  https://datatracker.ietf.org/doc/html/rfc5102

* [Network Data Anonymization Thesis] Anonymization of Event Logs for Network Security Monitoring
  https://spectrum.library.concordia.ca/id/eprint/986484/7/Rasic_MSc_S2020.pdf

* [OpenSSL] OpenSSL
  https://www.openssl.org/