---
Title: Install and upgrade modules
linkTitle: Install and upgrade modules
description:
weight: 4
alwaysopen: false
categories: ["Modules"]
---

Several modules come packaged with [Redis Enterprise]({{<relref "/rs">}}). However, if you want to use additional modules or upgrade a module to a more recent version, you need to:

1. [Install a module package]({{<relref "/modules/install/add-module-to-cluster">}}) on the cluster.
1. [Enable a module]({{<relref "/modules/install/add-module-to-database">}}) for a new database or [upgrade a module]({{<relref "/modules/install/upgrade-module">}}) in an existing database.

For custom modules not packaged and certified by Redis, [package them with the RAMP utility]({{<relref "/modules/install/packaging-modules">}}) before installation on a Redis Enterprise cluster.