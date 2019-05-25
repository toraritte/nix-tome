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

• A software component is almost never self-contained; rather, it depends on other
components to do some work on its behalf. These are its dependencies. For correct
deployment, it is necessary that all dependencies are identified. This identification
is quite hard, however, as it is often difficult to test whether the dependency specifi-
cation is complete. After all, if we forget to specify a dependency, we don’t discover
that fact if the machine on which we are testing already happens to have the depen-
dency installed.
• Dependencies are not just a runtime issue. To build a component in the first place we
need certain dependencies (such as compilers), and these need not be the same as the
runtime dependencies, although there may be some overlap. In general, deployment
of the build-time dependencies is not an end-user issue, but it might be in source-
based deployment scenarios; that is, when a component is deployed in source form.
This is common in the open source world.
• Dependencies also need to be compatible with what is expected by the referring
component. In general, not all versions of a component will work. This is the case
even in the presence of type-checked interfaces, since interfaces never give a full
specification of the observable behaviour of a component. Also, components often
exhibit build-time variability, meaning that they can be built with or without certain
optional features, or with other parameters selected at build time. Even worse, the
component might be dependent on a specific compiler, or on specific compilation
options being used for its dependencies (e.g., for Application Binary Interface (ABI)
compatibility).
• Even if all required dependencies are present, our component still has to find them,
in order to actually establish a concrete composition of components. This is often
a rather labour-intensive part of the deployment process. Examples include setting
up the dynamic linker search path on Unix systems [160], or the CLASSPATH in the
Java environment.
• Components can depend on non-software artifacts, such as configuration files, user
accounts, and so on. For instance, a component might keep state in a database that
has to be initialised prior to its first use.
