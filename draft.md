# Part I. Introduction

## 1. Introduction

*Software deployment* is about getting computer programs from one machine to another and having them still work when they get there. Though it is a part of the field of Software Configuration Management (SCM), it has not been a subject of academic study until quite recently [169].

> Is this still true? Are there no new studies since 2006?
>
> [169] André van der Hoek, Richard S. Hall, Dennis Heimbigner, and Alexander L. Wolf. Software release management. In M. Jazayeri and H. Schauer, editors, Proceedings of the Sixth European Software Engineering Conference (ESEC/FSE 97), pages 159–175. Springer-Verlag, 1997.

The development of principles and tools to support the deployment process has largely been relegated to industry, system administrators, and Unix hackers. This has resulted in a large number of often ad hoc tools that typically automate manual practices but do not address fundamental issues in a systematic and disciplined way.

This is evidenced by the huge number of mailing list and forum postings about deployment failures, ranging from applications not working due to missing dependencies, to
subtle malfunctions caused by incompatible components. Deployment problems also seem
curiously resistant to automation: the same concrete problems appear time and again. De-
ployment is especially difficult in heavily component-based systems—such as Unix-based
open source software—because the effort of dealing with the dependencies can increase
super-linearly with each additional dependency.

> Admonishment (NOTE):
(Add a note or a link here to sections describing what a component is. In the PhD thesis, the section is "3.1 What is a component?". Figure out where to put it.)

## 1.1 Software deployment

Software deployment is the problem of managing the distribution of software to end-user machines.

### 1.1.2 Correct deployment

That is, a developer has created some piece of software, and this ultimately has to end up on the machines of end-users. After the initial installation of the software, it might need to be upgraded or uninstalled.  Presumably, the developer has tested the software and found it to work sufficiently well, so the challenge is to make sure that the software works just as well, i.e., the same, on the end-user machines.

**Correct deployment**: given identical inputs, the software should behave the same on an end-user machine as on the developer machine.

> Admonishment (NOTE):
Beware of the gross simplifications made above: First, in general there is no single “developer”. Second, there are usually several intermediaries between the developer and the end-user, such as a system administrator. However, for a discussion of the main issues this will suffice.

### 1.1.3 Factors affecting correct deployment

Deployment *should* be a simple problem. For instance, if the software consists of a set of files, then deployment should be a simple matter of *copying* those to the target machines. In practice, this turns out to be much harder, and the causes fall into two broad categories: **environment issues** and **manageability issues**.

#### 1.1.3.1 Environment issues

The first category is essentially about correctness. The software might make all sorts of demands about the *environment* in which it executes: that certain other software components are present in the system, that certain configuration files exist, that certain modifications were made to the Windows registry, and so on. If any of those environmental characteristics does not hold, then there is a possibility that the software does not work the same as it did on the developer machine.

Some concrete issues are the following:

  + **Identifying the dependencies of software components**

    A software component is almost never self-contained; rather, it depends on other components to do some work on its behalf. These are its **runtime dependencies**. For correct deployment, it is necessary that all dependencies are identified. This identification is quite hard, however, as it is often difficult to test whether the dependency specification is complete. After all, if we forget to specify a dependency, we don’t discover that fact if the machine on which we are testing already happens to have the dependency installed.

    In *source-based* deployment scenarios - when a component is deployed in source form- the component has to be built in the first place. This requires certain dependencies (**build-time dependencies**, such as compilers), and these need not be the same as the runtime dependencies, although there may be some overlap.

  + **Compatibility of dependencies**

    Dependencies also need to be *compatible* with what is expected by the referring component. In general, not all versions of a component will work. This is the case even in the presence of type-checked interfaces, since interfaces never give a full specification of the observable behaviour of a component.

    Components also often exhibit build-time *variability*, meaning that they can be built with or without certain optional features, or with other parameters selected at build time. Even worse, the component might be dependent on a specific compiler, or on specific compilation options being used for its dependencies (e.g., for Application Binary Interface (ABI) compatibility).

  + **Location of dependencies**

    Even if all required dependencies are present, our component still has to *find* them, in order to actually establish a concrete composition of components. This is often a rather labour-intensive part of the deployment process. Examples include setting up the dynamic linker search path on Unix systems [160], or the CLASSPATH in the Java environment.

> [160] TIS Committee. Tool Interface Specification (TIS) Executable and Linking Format (ELF) Specification, Version 1.2. http://www.x86.org/ftp/manuals/tools/elf.pdf, May 1995.

  + **Non-software dependencies**

    Components can depend on non-software artifacts, such as configuration files, user accounts, and so on. For instance, a component might keep state in a database that has to be initialised prior to its first use.

  + **Hardware requirements**

    Components can require certain hardware characteristics, such as a specific proces- sor type or a video card. These are somewhat outside the scope of software deployment, since we can at most *check* for such properties, not *realise* them if they are missing.

  + **Distributed dependencies**

    Finally, deployment can be a distributed problem. A component can depend on other components running on remote machines or as separate processes on the same machine. For instance, a typical multi-tier web service consists of an HTTP server, a server implementing the business logic, and a database server, possibly all running on different machines.

So we have two problems in deployment:

  1. we must *identify* what our component’s requirements on the environment are, and

  2. we must somehow *realise* those requirements in the target environment.

     Realisation might consist of installing dependencies, creating or modifying configuration files, starting remote processes, and so on.

#### 1.1.3.2 Manageability issues

The second category is about our ability to properly manage the deployment process. There are all kinds of operations that we need to be able to perform, such as packaging, transferring, installing, upgrading, uninstalling, and answering various queries; i.e., we have to be able to support the *evolution* of a software system. All these operations require various bits of information, can be time-consuming, and if not done properly can lead to incorrect deployment. For example:

  + **Uninstalling**

    When we uninstall a component, we have to know what steps to take to safely undo the installation, e.g., by deleting files and modifying configuration files. At the same time we must also take care never to remove any component still in use by some other part of the system.

  + **Upgrading**

    Likewise, when we perform a component upgrade, we should be careful not to overwrite any part of any component that might induce a failure in another part of the system. This is the well-known *DLL hell*, where upgrading or installing one application can cause a failure in another application due to shared dynamic libraries. It has been observed that software systems often suffer from the seemingly inexplicable phenomenon of “bit rot,” i.e., that applications that worked initially stop working over time due to changes in the environment [26].

> [26] Anders Christensen and Tor Egge. Store — a system for handling third-party applications in a heterogeneous computer environment. In Selected papers from the ICSE SCM-4 and SCM-5 Workshops on Software Configuration Management, number 1005 in Lecture Notes in Computer Science. Springer-Verlag,
1995.

  + **Reversibility**

    When we upgrade components, it is important to be able to *undo*, or *roll back* the effects of the upgrade, if the upgrade turns out to break important functionality. This requires both that we remember what the old configuration was, and that we have some way to reproduce the old configuration.

  + **Querying**

    Administrators often want to perform queries such as “to what component does this file belong?”, “how much disk space will it take to install this component?”, “from what sources was this component built?”, and so on.

  + **Maintenance**

    That is, keeping software on a system  up to date. There are many different policy choices that can be made. For instance, in a network, system administrators may want to push updates (such as security fixes) to all client machines periodically. On the other hand, if users are allowed to administer their own machines, it should be possible for them to select components individually.

    In heterogeneous networks (i.e., consisting of many different types of machines), or in small environments (e.g., a home computer), it is not easy to stay up to date with software updates. In particular in the case of security fixes this is an important problem. So we need to know what software is in use, whether updates are available, and whether such updates should be performed.

  + **Deployment**

    Components can often be deployed in both source and binary form. Binary pack- ages have to be built for each supported platform, and sometimes in several variants as well. For instance, the Linux kernel has thousands of build-time configuration options. This greatly increases the deployment effort, particularly if packaging and transfer of packages is a manual or semi-automatic process.

  + **Variability**

    Since components often have a huge amount of variability, we sometimes want to expose that variability to certain users. For instance, Linux distributors or system administrators typically want to make specific feature selections. A deployment system should support this.

## 1.2 Problem summary of existing deployment systems

  + Dependency specifications are not validated, leading to incomplete deployment (i.e., missing dependencies cause the software component fail to run).

  + Dependency specifications are inexact (e.g., nominal).

  + It is not possible to deploy multiple versions or variants of a component side-by-side.

  + Components can interfere with each other.

  + It is not possible to roll back to previous configurations.

  + Upgrade actions are not atomic.

  + Applications must be monolithic, i.e., they must statically contain all their dependencies.

  + Deployment actions can only be performed by administrators, not by unprivileged users.

  + There is no link between binaries and the sources and build processes that built them.

  + The system supports either source deployment or binary deployment, but not both; or it supports both but in a non-unified way.

  + It is difficult to adapt components.

  + Component composition is manual.

  + The component framework is narrowly restricted to components written in a specific programming language or framework.

  + The system depends on non-portable techniques.

> Admonishment (NOTE): See appendix (number and link here) for a comparison of other approaches to deployment.
> TODO: Roll section 1.2 and 7.6 into one. How outdated is it? Maybe making it an appendix would be better.

## 1.3 The Nix approach

The main purpose of Nix is to support safe and efficient deployment. The Nix approach is to store software components in isolation from each other in a central component store, under path names that contain cryptographic hashes of all inputs involved in building the component, such as `/nix/store/rwmfbhb2znwp...-firefox-1.0.4`. It guarantees that:

  + The cryptographic hashing scheme used by the Nix component store prevents undeclared dependencies, giving us **complete deployment**. Furthermore it provides support for side-by-side existence of component versions and variants.

  + **Isolation between components** prevents interference.

  + **Users can install software independently from each other**, without requiring mutual trust relations. Components that are common between users are shared, i.e., stored only once.

  + **Atomic upgrades**; there is no time window in which the system is in an inconsistent state.

  + Nix supports **O(1)-time rollbacks** to previous configurations.

  + Nix supports **automatic garbage collection** of unused components.

  + The Nix component language describes not just how to build individual components, but also **compositions**. The language is a lazy, purely functional language. This is a good basis for a component composition language, as it allows dependencies and variability to be expressed in an elegant and flexible way.

  + Nix has a **transparent source/binary deployment model** that marries the best parts of source deployment models such as the FreeBSD Ports Collection, and binary deployment models such as RPM. In essence, binary deployment is an automatic optimisation of source deployment.

  + Nix is **policy-free**; it provides mechanisms to implement various deployment policies, but does not enforce a specific one. Examples of policies are channels (TODO: link to channels), and pure source deployments.

> Admonishment (NOTE):
> One-click installations are also possible (see [TODO: phd bib ref to thesis], page 43), and were in fact implemented in the past, but have been removed since. See [NixOS/nix issue #1783](https://github.com/NixOS/nix/issues/1783).

  + The purely functional model supports **efficient component upgrades** despite the fact that a change to a fundamental component can propagate massively through the dependency graph.

  + Nix supports **distributed multi-platform builds** in a transparent manner, i.e., a remote build is indistinguishable from a local build from the perspective of the user.

  + Nix provides a good basis for the implementation of a build farm, which supports continuous integration, portability testing, improved manageability, and release management (i.e., it builds concrete installable components that can be deployed directly through Nix)

  + The use of Nix for software deployment extends naturally to the field of **service deployment**. Services are running processes that provide some useful facility to users, such as web servers. They consist of software components, static configuration and data, and mutable state. The first two aspects can be managed using Nix, and its advantages—such as rollbacks and side-by-side deployment—apply here as well.
> TODO: microservices

  + Though Nix is typically used to build *large-grained components* (i.e., traditional packages), it can also be used to **build** *small-grained components* such as **individual source files**. When used in this way it is a superior alternative to build managers such as Make [56], ensuring complete dependency specifications and enabling more sharing between builds.

> TODO: Incorporate.
> Parts of this thesis are adapted from earlier publications. Chapter 2, “An Over-
view of Nix” is very loosely based on the LISA ’04 paper “Nix: A Safe and Policy-
Free System for Software Deployment”, co-authored with Merijn de Jonge and Eelco
Visser [50]. Chapter 3, “Deployment as Memory Management” is based on Sections 3–6 of
the ICSE 2004 paper “Imposing a Memory Management Discipline on Software Deploy-
ment”, written with Eelco Visser and Merijn de Jonge [52]. Chapter 6, “The Intensional
Model” is based on the ASE 2005 paper “Secure Sharing Between Untrusted Users in a
Transparent Source/Binary Deployment Model” [48]. Section 7.5 on patch deployment
appeared as the CBSE 2005 paper “Efficient Upgrading in a Purely Functional Compo-
nent Deployment Model” [47]. Chapter 9, “Service Deployment” is based on the SCM-12
paper “Service Configuration Management”, written with Martin Bravenboer and Eelco
Visser [49]. Snippets of Section 10.2 were taken from the SCM-11 paper “Integrating
Software Construction and Software Deployment” [46].

> TODO: Where to put "1.7. Notational conventions"? Appendix? Let's see when getting into the more in-depth sections.
>
> Part II of this thesis contains a number of algorithms in a pseudo-code notation in an im-
> perative style with some functional elements. Keywords in algorithms are in boldface,
> variables are in italic, and function names are in sans serif . Callouts like this 1 are fre-
> quently used to refer to points of interest in algorithms and listings from the text. Variable
> assignment is written x ← value.
> Sets are written in curly braces, e.g., {1, 2, 3}. Since many algorithms frequently extend
> ∪
> variables of set type with new elements, there is a special notation x ← set which is syn-
> ∪
> tactic sugar for x ← x ∪ set. For instance, x ← {2, 3} extends the set stored in x with the
> elements 2 and 3.
> 1Set comprehensions are a concise notation for set construction: {expr | conditions} pro-
> duces a set by applying the expression expr to all values from a (usually implicit) universe
> that meet the predicate conditions. For example, {x 2 | x ∈ {2, 3, 4} ∧ x 6 = 3} produces the
> set {4, 16}.
> Ordered lists are written between square brackets, e.g., [1, 2, 1, 3]. Lists can contain ele-
> ments multiple times. Lists are sometimes manipulated in a functional (Haskell-like [135])
> style using the function map and λ -abstraction: x ← map (λ v . f (v), y) constructs a list x by
> applying a function f (v) to each element in the list x. For instance, map (λ x . x 2 , [4, 2, 6])
> produces the list [16, 4, 36]. Tuples or pairs are enclosed in parentheses, e.g., (a, b, c).
> Throughout this thesis, storage sizes are given in IEC units [33]: 1 KiB = 1024 bytes,
> 1 MiB = 1024 2 bytes, and 1 GiB = 1024 3 bytes.
> Data types are defined in a Haskell-like notation. For instance, the definition
> data Foo = Foo {
> x : String ,
> ys : [ String ],
> zs : { String }
> }
> defines a data type Foo with three fields: a string x , a list of strings ys , and a set of strings
> zs . An example of the construction of a value of this type is as follows:
> Foo {x = "test" , ys = [ "hello" , "world" ], zs = 0}
> /
> In addition, types can have unnamed fields and multiple constructors. For example, the
> definition
> data Tree = Leaf String | Branch Tree Tree
> defines a binary tree type with strings stored in the leaves. An example value is7

