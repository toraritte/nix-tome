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

The second category is about our ability to properly manage the deployment process. There are all kinds of operations that we need to be able to perform, such as packaging, transferring, installing, upgrading, uninstalling, and answering various queries; i.e., we have to be able to support the evolution of a software system. All these operations require various bits of information, can be time-consuming, and if not done properly can lead to incorrect deployment. For example:

• When we uninstall a component, we have to know what steps to take to safely undo the installation, e.g., by deleting files and modifying configuration files. At the same time we must also take care never to remove any component still in use by some other part of the system.

• Likewise, when we perform a component upgrade, we should be careful not to over- write any part of any component that might induce a failure in another part of the system. This is the well-known DLL hell, where upgrading or installing one applica- tion can cause a failure in another application due to shared dynamic libraries. It has been observed that software systems often suffer from the seemingly inexplicable phenomenon of “bit rot,” i.e., that applications that worked initially stop working over time due to changes in the environment [26].

• Administrators often want to perform queries such as “to what component does this file belong?”, “how much disk space will it take to install this component?”, “from what sources was this component built?”, and so on.

• Maintenance of a system means keeping the software up to date. There are many different policy choices that can be made. For instance, in a network, system ad- ministrators may want to push updates (such as security fixes) to all client machines periodically. On the other hand, if users are allowed to administer their own ma- chines, it should be possible for them to select components individually.

• When we upgrade components, it is important to be able to undo, or roll back the effects of the upgrade, if the upgrade turns out to break important functionality. This requires both that we remember what the old configuration was, and that we have some way to reproduce the old configuration.

• In heterogeneous networks (i.e., consisting of many different types of machines), or in small environments (e.g., a home computer), it is not easy to stay up to date with software updates. In particular in the case of security fixes this is an important problem. So we need to know what software is in use, whether updates are available, and whether such updates should be performed.

• Components can often be deployed in both source and binary form. Binary pack- ages have to be built for each supported platform, and sometimes in several variants as well. For instance, the Linux kernel has thousands of build-time configuration options. This greatly increases the deployment effort, particularly if packaging and transfer of packages is a manual or semi-automatic process.

• Since components often have a huge amount of variability, we sometimes want to expose that variability to certain users. For instance, Linux distributors or system administrators typically want to make specific feature selections. A deployment system should support this.

