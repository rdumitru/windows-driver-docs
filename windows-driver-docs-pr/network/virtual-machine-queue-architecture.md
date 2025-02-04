---
title: Virtual Machine Queue (VMQ) Architecture
description: The NDIS virtual machine queue (VMQ) architecture provides several advantages for virtualization.
ms.date: 11/30/2021
ms.custom: contperf-fy22q2
---

# Virtual Machine Queue (VMQ) Architecture

The NDIS virtual machine queue (VMQ) architecture provides advantages for virtualization such as:

-   Virtualization impacts performance and VMQ helps overcome those effects.

-   VMQ supports live migration.

-   VMQ coexists with NDIS task offloads and other optimizations.

This section provides high-level information about the NDIS VMQ interface. You should read this section before writing an NDIS driver that supports VMQ. For information about writing VMQ drivers, see [Writing VMQ Drivers](writing-vmq-drivers.md).

This section includes the following topics:

[Introduction to NDIS Virtual Machine Queue (VMQ)](introduction-to-ndis-virtual-machine-queue--vmq-.md)

[VMQ Components](vmq-components.md)

[VMQ Receive Queues](vmq-receive-queues.md)

[VMQ Receive Filters](vmq-receive-filters.md)

[Security Issues with NDIS Virtual Machine (VM) Shared Memory](security-issues-with-ndis-virtual-machine--vm--shared-memory.md)

[NDIS VMQ Live Migration Support](ndis-vmq-live-migration-support.md)

[NDIS VM Queue States](ndis-virtual-machine-queue-states.md)

 

 





