# Virtualization

Content

- Overview
- Benefits of Virtualization
- Type of Virtual Machine Managers (VMM)
- Virtualization Providers

## Overview

A concept within computer science that allows computer hardware to be abstracted in a way that allows multiple virtual machines (guest operating systems) to be ran on a single host machine (server/computer). A layer that sits above the hardware and creates a virtual system on which operating systems and applications can run.

At the base of virtualization is the **host**, the underlying hardware. The **virtual machine manager** (vmm or hypervisor) creates and runs virtual machines by providing an interface that is identical to the host. Each virtual machine (guest operating system) is provided with a virtual copy of the host.

**Virtual Machines** are just guest operating systems that run on top of a host operating system. The host operating system could be a virtual machine manager or a normal operating system that is using a virtualization application like VMware Workstation or Oracle VirtualBox.

## Benefits of Virtualization

Virtualization provides many benefits over non-virtualized servers. A non-exhaustive list below shows some of the key benefits that make virtualization so alluring.

- Ability to freeze or **suspend** a running virtual machine.
- Allows the copying of virtual machines with **snapshots**.
- Can **clone** virtual machines and run them on another without interrupting the current running task/application.
- Multiple operating systems can run concurrently on a developer's workstation.
- Efficient **resource utilization**, multiple virtual machines can be run on a single server instead of having multiple servers that serve one application at a time.
- **Templating** allows a saved virtual machine image to be used as a source for multiple running virtual machines, which can increase provisioning speed.
- Application installation is simple and redeploying applications to other systems is much easier than the usual steps of uninstalling and reinstalling with individual servers.

These are just some of the benefits that is afforded to us by virtualization.

## Types of VMMs

Major types of virtual machines, their implementation, their functionality.

**Type 0 Hypervisor:** The VMM is encoded in the firmware, each guest operating system is partitioned dedicated hardware. Essentially, each guest operating system in a type 0 hypervisor is a native operating system with a subset of hardware made available to it.

**Type 1 Hypervisor:** Commonly found in company data centers and are labeled "the data-center operating system". Special purpose operating systems that run natively on the hardware. They create, run, and manage guest operating systems. Type 1 hypervisors run in kernel mode, taking advantage of hardware protection. Where the host CPU allows, they use multiple modes to give guest operating systems their own control and improved performance.

**Type 2 Hypervisor:** Application-level virtual machine managers. This type of VMM is just another process ran and managed by the host operating system, and even the host does not know that virtualization is happening within the VMM. There is extra overhead when running this type of hypervisor because while running on a general purpose operating system, it must also run the VMM and guest operating systems. These hypervisors usually have poorer overall performance than type 0 and type 1.

**Paravirtualization:** Presents the guest with a system that is similar (not identical) to the guests preferred system. This requires the guest operating system to be modified to run on the paravirtualized hardware. The guest operating system helps the VMM with optimizing performance.

**Programming-Environment Virtualization:** A programming language is designed to run within a custom-built virtualized environment. A virtual environment, based on APIs, that provides a set of features we want to have available for a particular language and programs written in that language. Java programs run within the Java Virtual Machine (JVM) environment, and the JVM is compiled to be a native program on systems on which it runs.

**Emulation:** Allows for operating systems to work on non-compatible CPUs. By translating all source CPU instructions to the equivalent instructions of the target CPU. Emulation is useful when the host system has one system architecture and the guest system was compiled for a different architecture.

## Virtualization Providers

| Organization | Type 1 Hypervisor |
| ------------ | ----------------- |
| VMware       | ESXi              |
| Oracle       | OVM               |
| Microsoft    | Hyper-V           |
| Citrix       | Xen Server        |
| Redhat       | KVM               |
