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

