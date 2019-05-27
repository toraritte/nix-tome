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
> (TODO Add a note or a link here to sections describing what a component is. In the PhD thesis, the section is "3.1 What is a component?". Figure out where to put it.)
> Add [155] Clemens Szyperski. Component technology—what, where, and how? In Proceedings of the 25th International Conference on Software Engineering (ICSE 2003), pages 684–693, May 2003.
> ... and section 2.1 The Nix store
>
> https://dl.acm.org/citation.cfm?id=515228
> Clemens Szyperski: Component Software: Beyond Object-Oriented Programming
> 
> “A software component is a unit of composition with contractually
> specified interfaces and explicit context dependencies only. A
> software component can be deployed independently and is subject
> to composition by third parties.”


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

> [26] Anders Christensen and Tor Egge. Store — a system for handling third-party applications in a heterogeneous computer environment. In Selected papers from the ICSE SCM-4 and SCM-5 Workshops on Software Configuration Management, number 1005 in Lecture Notes in Computer Science. Springer-Verlag, 1995.

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

  + The cryptographic hashing scheme used by the Nix component store prevents undeclared dependencies, giving us **complete deployment** (i.e., there are no missing dependencies during any deployment task). Furthermore it provides support for side-by-side existence of component versions and variants.

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

## 2. An Overview of Nix

> TODO: This section in the PhD thesis informally introduces some of the terminology, shows an example derivation, and some of the main features, then later sections jump in the deep end, referencing back. This approac has been borrowed by other Nix tutorials (or theFreeswitch 1.8 book that goes into the dialplan topic 3 times, scattering the resources)
> Maybe this is the right approach. Experiment.

> TODO: How much did the Nix internals change?

### what is a component

Nix is a system for software deployment. The term component will be used to denote the basic units of deployment. These are simply sets of files that implement some arbitrary functionality through an interface. In fact, Nix does not really care what a component actually is. As far as Nix is concerned a component is just a set of files in a file system. That is, we have a very weak notion of component, weaker even than the commonly used definition in [155]. (What we call a component typically corresponds to the ambiguous notion of a package in package management systems. Nix’s notion of components is discussed further in Section 3.1.)

Nix stores components in a component store, also called the Nix store. The store is simply a designated directory in the file system, usually /nix/store . The entries in that directory are components (and some other auxiliary files discussed later). Each component is stored in isolation; no two components have the same file name in the store.  A subset of a typical Nix store is shown in Figure 2.1. It contains a number of applications—GNU Hello 2.1.1 (a toy program that prints “Hello World”, Subversion 1.1.4 (a version management system), and Subversion 1.2.0—along with some of their dependencies.

These components are not single files, but directory trees. For instance, Subversion consists of a directory called bin containing a program called svn , and a directory lib containing many more files belonging to the component. (For simplicity, many files and dependencies have been omitted from the example.) The arrows denote runtime dependencies between components, which will be described shortly.

The notable feature in Figure 2.1 is the names of the components in the Nix store,
such as /nix/store/bwacc7a5c5n3...-hello-2.1.1 . The initial part of the file names, e.g.,
bwacc7a5c5n3... , is what gives Nix the ability to prevent undeclared dependencies and
component interference. It is a representation of the cryptographic hash of all inputs in-
volved in building the component.
Cryptographic hash functions are hash functions that map an arbitrary-length input onto
a fixed-length bit string. They have the property that they are collision-resistant: it is
computationally infeasible to find two inputs that hash to the same value. Cryptographic
hashes are discussed in more detail in Section 5.1. Nix uses 160-bit hashes represented in
a base-32 notation, so each hash is 32 characters long. In this thesis, hashes are abridged
to ellipses ( ... ) most of the time for reasons of legibility. The full path of a directory entry
in the Nix store is referred to as its store path. An example of a full store path is:
/nix/store/bwacc7a5c5n3qx37nz5drwcgd2lv89w6-hello-2.1.1
So the file bin/hello in that component has the full path
/nix/store/bwacc7a5c5n3qx37nz5drwcgd2lv89w6-hello-2.1.1/bin/hello
The hash is computed over all inputs, including the following:
• The sources of the components.
• The script that performed the build.
• Any arguments or environment variables [152] passed to the build script.
• All build time dependencies, typically including the compiler, linker, any libraries
used at build time, standard Unix tools such as cp and tar , the shell, and so on.
Cryptographic hashes in store paths serve two main goals. They prevent interference
between components, and they allow complete identification of dependencies. The lack
of these two properties is the cause for the vast majority of correctness and flexibility
problems faced by conventional deployment systems, as we saw in Section 1.2.
**Preventing interference** Since the hash is computed over all inputs to the build process
of the component, any change, no matter how slight, will be reflected in the hash. This
includes changes to the dependencies; the hash is computed recursively. Thus, the hash
essentially provides a unique identifier for a configuration. If two component compositions
differ in any way, they will occupy different paths in the store (except for dependencies that
they have in common). Installation or uninstallation of a configuration therefore will not
interfere with any other configuration.
For instance, in the Nix store in Figure 2.1 there are two versions of Subversion. They
were built from different sources, and so their hashes differ. Additionally, Subversion
1.2.0 uses a newer version of the OpenSSL cryptographic library. This newer version of
OpenSSL likewise exists in a path different from the old OpenSSL. Thus, even though
installing a new Subversion entails installing a new OpenSSL, the old Subversion instance
is not affected, since it continues to use the old OpenSSL.
Recursive hash computation causes changes to a component to “propagate” through the
dependency graph. This is illustrated in Figure 2.3, which shows the hash components of
the store paths computed for the Mozilla Firefox component (a web browser) and some of
its build time dependencies, both before and after a change is made to one of those depen-
dencies. (An edge from a node A to a node B denotes that the build result of A is an input to
the build process of B.) Specifically, the GTK GUI library dependency is updated from ver-
sion 2.2.4 to 2.4.13. That is, its source bundle ( gtk+-2.2.4.tar.bz2 and gtk+-2.4.13.tar.bz2 ,
respectively) changes. This change propagates through the dependency graph, causing dif-
ferent store paths to be computed for the GTK component and the Firefox component.
However, components that do not depend on GTK, such as Glibc (the GNU C Library),
are unaffected.
An important point here is that upgrading only happens by rebuilding the component in
question and all components that depend on it. We never perform a destructive upgrade.
Components never change after they have been built—they are marked as read-only in the
file system. Assuming that the build process for a component is deterministic, this means
that the hash identifies the contents of the components at all times, not only just after it has
been built. Conversely, the build-time inputs determine the contents of the component.
Therefore we call this a purely functional model. In purely functional programming
languages such as Haskell [135], as in mathematics, the result of a function call depends
exclusively on the definition of the function and on the arguments. In Nix, the contents of
a component depend exclusively on the build inputs. The advantage of a purely functional
model is that we obtain strong guarantees about components, such as non-interference.
Preventing interference
**Identifying dependencies** The use of cryptographic hashes in component file names
also prevents the use of undeclared dependencies, which (as we saw in Section 1.2) is the
major cause of incomplete deployment. The reason that incomplete dependency informa-
tion occurs is that there is generally nothing that prevents a component from accessing
another component, either at build time or at runtime. For instance, a line in a build script
or Makefile such as
Figure 2.3.: Propagation of dependency changes through the store paths of the build-time
component dependency graph
gcc -o program main.c ui.c -lssl
which links a program consisting of two C modules against a library named ssl , causes
the linker to search in a set of standard locations for a library called ssl 1 . These standard
locations typically include “global” system directories such as /usr/lib on Unix systems. If
the library happens to exist in one of those directories, we have incurred a dependency.
However, there is nothing that forces us to include this dependency in the dependency
specifications of the deployment system (e.g., in the RPM spec file of Figure 1.2).
At runtime we have the same problem. Since components can perform arbitrary I/O
actions, they can load and use other components. For instance, if the library ssl used
above is a dynamic library, then program will contain an entry in its dynamic linkage
meta-information that causes the dynamic linker to search for the library when program is
started. The dynamic linker similarly searches in global locations such as /lib and /usr/lib .
Of course, there are countless mechanisms other than static or dynamic linking that es-
tablish a component composition. Some examples are including C header files, importing
Java classes at compile time, calling external programs found through the PATH environ-
ment variable, and loading help files at runtime.
The hashing scheme neatly prevents these problems by not storing components in global
locations, but in isolation from each other. For instance, assuming that all components in
the system are stored in the Nix store, the linker line
gcc -o program main.c ui.c -lssl
will simply fail to find libssl . Unless the path to an OpenSSL instance (e.g., /nix/store/-
5jq6jgkamxjj...-openssl-0.9.7d ) was explicitly fed into the build process and added to the
linker’s search path, no such instance will be found by the linker.
Also, we go to some lengths to ensure that component builders are pure, that is, not
influenced by outside factors. For example, the builder is called with an empty set of
environment variables (such as the PATH environment variable, which is used by Unix
shells to locate programs) to prevent user settings such as search paths from reaching tools
invoked by the builder. Similarly, at runtime on Linux systems, we use a patched dynamic
linker that does not search in any default locations—so if a dynamic library is not explicitly
declared with its full path in an executable, the dynamic linker will not find it.
Thus, when the developer or deployer fails to specify a dependency explicitly (in the Nix
expression formalism, discussed below), the component will fail deterministically. That
is, it will not succeed if the dependency already happens to be available in the Nix store,
without having been specified as an input. By contrast, the deployment systems discussed
in Section 1.2 allow components to build or run successfully even if some dependencies
are not declared.

**Retained dependencies** A rather tricky aspect to dependency identification is the oc-
currence of retained dependencies. This is what happens when the build process of a
component stores a path to a dependency in the component. For instance, the linker invo-
cation above will store the path of the OpenSSL library in program , i.e., program will have
a “hard-coded” reference to /nix/store/5jq6jgkamxjj...-openssl-0.9.7d/lib/libssl.so .
Figure 2.4 shows a dump of parts of the Subversion executable stored in the file /nix/-
store/v2cc475f6nv1...-subversion-1.1.4/bin/svn . Offsets are on the left, a hexadecimal rep-
resentation in the middle, and an ASCII representation on the right. The build process for
Subversion has caused a reference to the aforementioned OpenSSL instance to be stored in
the program’s executable image. The path of OpenSSL has been stored in the RPATH field
of the header of the ELF executable, which specifies a list of directories to be searched
by the dynamic linker at runtime [160]. Even though our patched dynamic linker does
not search in /nix/store/5jq6jgkamxjj...-openssl-0.9.7d/lib by default, it will find the library
anyway through the executable’s RPATH .
This might appear to be bad news for our attempts to prevent undeclared dependencies.
Of course, we happen to know the internal structure of Unix executables, so for this specific
file format we can discover retained dependencies. But we do not know the format of every
file type, and we do not wish to commit ourselves to a single component composition
mechanism. E.g., Java and .NET can find retained dependencies by looking at class files
and assemblies, respectively, but only for their specific dynamic linkage mechanisms (and
not for dependencies loaded at runtime).
The hashing scheme comes to the rescue once more. The hash part of component paths
is highly distinctive, e.g., 5jq6jgkamxjj... . Therefore we can discover retained dependen-
cies generically, independent of specific file formats, by scanning for occurrences of hash
parts. For instance, the executable image in Figure 2.4 contains the highlighted string
5jq6jgkamxjj... , which is evidence that an execution of the svn program might need that
particular OpenSSL instance. Likewise, we can see that it has a retained dependency on
some Glibc instance ( /nix/store/72by2iw5wd8i... . Thus, we automatically add these as run-
time dependencies of the Subversion component. Using this scanning approach, we find
the runtime dependencies indicated in Figure 2.1.
This approach might seem a bit dodgy. After all, what happens when a file name is rep-
resented in a non-standard way, e.g., in UTF-16 [34, Section 2.5] or when the executable
is compressed? In Chapter 3 the dependency problem is cast in terms of memory man-
agement in programming languages, which justifies the scanning approach as an analogue
of conservative garbage collection. Whether this is a legitimate approach is an empirical
question, which is addressed in Section 7.1.5.
Note that strictly speaking it is not the use of cryptographic hashes per se, but globally
unique identifiers in file names that make this work. A sufficiently long pseudo-random
number also does the trick. However, the hashes are needed to prevent interference, while
at the same time preventing unnecessary duplication of identical components (which would
happen with random paths).

**Closures** Section 1.2 first stated the goal of complete deployment: safe deployment requires that there are no missing dependencies. This means that we need to deploy closures
of components under the “depends-on” relation. That is, when we deploy (i.e., copy) a
component X to a client machine, and X depends on Y , then we also need to deploy Y to
the client machine.
The hash scanning approach gives us all runtime dependencies of a component, while
hashes themselves prevent undeclared build-time dependencies. Furthermore, these de-
pendencies are exact, not nominal (see page 8). Thus, Nix knows the entire dependency
graph, both at build time and runtime. With full knowledge of the dependency graph, Nix
can compute closures of components. Figure 2.2 shows the closure of the Subversion 1.1.4
instance in the Nix store, found by transitively following all dependency arrows.
In summary, the purely functional model, and its concrete implementation in the form
of the hashing approach used by the Nix store, prevents interference and enables complete
deployment. It makes deployment much more likely to be correct, and is therefore one
of the major results of this thesis. However, the purely functional model does provoke a
number of questions, such as:
Closures
• Hashes do not appear to be very user-friendly. Can we hide them from users in
everyday interaction?
• Can we deploy upgrades efficiently? That is, suppose that we want to upgrade Glibc
(a dependency of all other components in Figure 2.1). Can we prevent a costly
redownload of all dependent components?
As we will see in the remainder of this thesis, the answer to these questions is “yes”.

# Glossary

**complete deployment** No missing dependencies during any deployment task.
