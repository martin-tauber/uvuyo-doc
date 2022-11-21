---
layout: default
title: Requirements
---
# Hardware and Software Requirements

The uvuyo gateway is build based on java version 12. It runs on all major Linux systems including Ubuntu, Red Hat and Suse which are not out of support by their suppliers. For the Telefonica installation we chose to use RHEL 7.9 for the test and production system. Before upgrading or switching to new OS versions in production, the upgrade should be performed and tested in the test environment. This should not just include the upgrading process of the operating system, but also must include the functional testing of the adapter itself.

Customers must ensure that all security fixes are installed in time on the test and the production environment.

No Database is required to run the uvuyo gateway

## Sizing Kafka instance provided

The following hardware is required to run a node in the Test environment hosting the BEM Event Connector and the Helix ITSM Adapter:

8 Cores, 16GB Memory, 256 GB Storage

The following hardware is required to run a node in the Production environment hosting the BEM Event Connector and the Helix ITSM Adapter:

16 Cores, 16GB Memory, 512 GB Storage

## Sizing Kafka instance not provided

If a kafka server is not provided and needs to be setup on the hosts running the uvuyo gateway additional hardware is required.

The following hardware is required to run a node in the Test environment hosting the BEM Event Connector and the Helix ITSM Adapter:

16 Cores, 32GB Memory, 512 GB Storage

The following hardware is required to run a node in the Production environment hosting the BEM Event Connector and the Helix ITSM Adapter:

32 Cores, 32GB Memory, 1024 GB Storage

