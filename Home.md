# Project status and description

This is a research project that might also produce usable results. This is kind of teaser. SOM contains unique ideas still not outperformed, and at the same time it is outdated: no Unicode, no ARC out of box. So eventually some another model is to be designed, but the current goal is to make something mind-blowing and get better understanding of low-level stuff required to redesign.

## Achievements

Created by hand Delphi bindings (old, procedural) for SOM 3.0 kernel, Emitter Framework and Interface Repository Framework. Created IR importer in Delphi using these bindings. Generated new object-style bindings for SOM 2.1. Ported IR importer from old bindings (and SOM 3.0) to the new bindings (and SOM 2.1). (SOM 2.1 is better, see below.) Now the importer uses bindings produced by itself. That means, these new bindings are usable enough to traverse the whole SOM.IR and regenerate bindings from scratch.

There is only one tool at the moment: SOMIRIMP.exe, the name is chosen to be similar to [TLIBIMP.exe](http://docwiki.embarcadero.com/RADStudio/Berlin/en/TLIBIMP.EXE).

## Current tasks

Better exceptions handling (mapped to one class currently). Implement subclassing (currently client only). Check SOM.DLL for DEP issue and patch it like before.

The IR importer is designed to generate self-sufficient files (this is to prevent RRBC issues on inter-BPL level in particular), so every file produced by IR importer is self-sufficient standalone unit. The output unit contains the dynamic part generated from SOM.IR (more complete compared to manual bindings) and the hardwired part (less complete). These are the procedures and functions not covered by SOM.IR database. Copying everything from SOM 3.0 bindings is not an option since SOM 2.1 is older and contains less procedures, so everything needs checking before copying.

## Near future plans

Progress was slow because I was busy at university. In summer 2016 I have graduated. In inception project was aimed at SOM 3.0 support since it looked like the best thing and it was the only thing for Windows anyway. Since then other things were discovered:

1. Patch 3.5.9 for VisualAge C++ for Windows containing full SOM 2.1
2. The same patch (found on IBM FTP) also contains enough files to run DTC C++
3. OpenDoc for Windows with ComponentGlue works with SOM 2.1, but not SOM 3.0

With SOM 2.1 we get DirectToSOM C++ and OLE Automation (read: JScript, VBScript, Python, PHP, ...) this way. This could be an effective demonstration. SOM is known for its backwards compatibility, but SOM 3.0 Release (December 1996) is out of line. It looks like SOM 3.0 Beta (February 1996) was designed properly and could be used as SOM 2.1 replacement, but it is not available. Something happened in the middle of 1996, near to the project closure, and it impacted SOM 3.0 Release quality. So the preference is for SOM 2.1 currently (and then somFree).

After all I have decided to make another object model like SOM from scratch instead of using somFree. The object model itself rocks, but multiple minor features are crying for update. And since there is little point in being compatible with the 20-years old SOM, where most compiled libraries don't even work on modern OSes, we could restart with something modern.

Get rid of 1-byte strings and start with 4-byte ones from the beginning just like COM started with 2-byte ones. Get ARC just like in modern Objective-C. Use something based on WebSockets for interop as opposed to IIOP over raw sockets. That's most important changes I would like to make to object model.

## What are SOM and somFree?

There were times dating back to DOS era when different teams could write in different programming languages (Pascal, Assembler, Fortran, C) and then link everything together. Choosing programming language was a matter of taste and another considerations. Then object-oriented programming emerged, complexity raised, and programming libraries virtually dictate in what programming language should you use them. You can't easily write PHP module in Delphi or use FireMonkey from Ada. Both COM and SOM (and also XPCOM, UNO, GObject, libobjc) aim to bridge the gap and restore the lost paradise. SOM is not widely known compared to other technologies, but it is superior from several points of view.

SOM was mostly known on OS/2, z/OS, Mac OS Classic, AIX, and is still mostly associated with OS/2. However, IBM also had version for Windows. You can grab it here: https://bitbucket.org/OCTAGRAM/somobjects/ in Downloads section. It had a serious problem with Data Execution Prevention enabled on some version of Windows (e. g. Win2003), so I recommend using som301nt.7z, in which this problem is fixed. SOM-Delphi is currently being tested against this patched SOMObjects DTK.

IBM SOM is closed-source, so it was cloned at least 2 times: NOM and somFree. NOM is not binary compatible with SOM, which is not a big problem, but it also forces using Boehm Garbage Collector which I dislike very much. Before Mac OS X 10.4 there was no garbage collection in Objective-C, and in Mac OS X 10.5 Objective-C 2.0 was introduced, where libraries could be optionally GC-enabled. Any single GC-enabled libraries forces using GC over all process. Hopefully GC-kiddies did no do big damage, and developers' community generally preferred not to use GC. And Apple also did a wise thing by introducing ARC, automatic reference counting, similar to interface reference handling in Delphi, that made programming with RC more comfortable. I like that RC won in Apple ecosystem. Despite all the GC-kiddies who got COM wrong or who thinks that closures are impossible without GC, or that XML is impossible to represent without GC, we can see Apple and Delphi 2009+ ecosystems with closures (blocks) and XML and everything else, and no GC. So I don't even want to mess with NOM since it forces GC no matter if it's needed or not.

somFree, on the other hand, aims to be binary compatible. So it's like SOM, but open source and ported to lots of platforms like Linux and Mac OS X.

## SOM, somFree and Delphi

Now you might have a question why do I mess with closed source 16-years-old IBM SOM when there is promising somFree. Initially it was related to the lack of Emitter Framework in somFree. Since then Interface Repository Framework was used instead, and it's not a problem anymore. However, DTS C++ compiler and Novell CompomentGlue all rely on IBM SOM2.1, so we would have much more to show with SOM 2.1. Compatibility with somFree might be tested afterwards.

Delphi is harder to make bindings compared to Ada, that's why Delphi comes first. If we do Delphi, Ada would be easier.

## Full bootstrap

There is a problem with unpopular technologies is that it is hard to learn them. There are lots of documentation about SOM, but I can't say they allow me learn SOM very well. This is the reason why I decided to do a full bootstrap, implement Delphi emitters in Delphi.

Bootstrapping is hard because in order to write Delphi emitters in Delphi I would normally use Delphi emitter, and there is none, I don't even have a full picture of mapping different SOM features to Delphi, so I did it by hand. And while I was doing it, my understanding of SOM internals improved.

## Impedance mismatch

SOMObjects are supposed to bridge different languages, but they are itself a language that can have mismatches with other languages. I think it will be a good idea to define some additional rules that should not be violated in order to not create problems for emitters. However, I see that even kernel stuff IDL violates them, so there should be a fix for kernel objects, and newly created objects should avoid violations.

In particular:

* Delphi, Ada and so on have a strict mapping of packages (units) to filenames. C++ and CORBA (SOM is based on CORBA) have another system. Developer includes files, and every file can define new namespaces and can append anything to any namespace. It would be beautiful and self-obvious to map CORBA namespaces to units or packages, but then it should not be allowed for any IDL file to append anything to foreign namespace. Fallback mapping should be provided on demand, but it won't be so beautiful.
* In CORBA and C++ it is possible to define a class name and say nothing more about it at all. It is possible to have pointers on such class instance, but not possible to invoke methods or query fields. For instance, SOMClassMgrObject (which is one of essential kernel objects!) has a method returning an object of class Repository, but no one knows what is Repository class exactly except that it is probably a descendant of SOMObject. If one just includes somcm.idl and follows every include, one won't find anything about Repository. One needs to know that Repository is defined in another not logically related file. In C++, when one does not need to work with Interface Repository, one can just not call this method. C++ rules allow it to be defined. And if one wants to work with this class, one includes one more file, and mysterious Repository becomes declared and usable. It is not possible to punk with Delphi and Ada compiler this way. SOMClassMgr bindings in these languages must return something known. So emitter have to either emit SOMObject pointer result or be able to locate Repository and every other class. There should be no unknown classes.
* In C and C++ one can do a single include, and it can include something else by chain. In the end one gets lots of names in namespaces. Delphi and Ada does not work this way, but for symmetry it might make sense to contain multiple bindings in a single file. Once again, then we need to know how to group IDLs and make it just work.
* Even if we resolve forward references in a single namespace, we would have trouble resolving them between namespaces because Delphi (but not Ada) requires all circular references to be resolved in a single "type" definition without breaks, and we can't have single "type" definition spanning across units.

These are the problems I've met so far. I want Delphi emitter (and Ada emitter in the future) to do the best what they can, but it is a trial-and-error work. I need to write something to start from. Then I can experiment with IDLs. I can try to create bindings for complete SOM 3.0 DTK. I can try to create bindings for WPS. The fact that WPS is only for OS/2 should not be a problem for testing emitter. There are also OpenDoc IDLs.

Proposed solution: as opposed to file-to-file emitter using Emitter Framework make SOM Interface Repository import. SOM IR is a binary database, somewhat similar to COM TLB. This way we would be able to access all information in one place. Then we produce a single or several big files containing every SOM class and everything related to them. Every component developer choose own unit name for importing SOM stuff. This way we eliminate possible RRBC conflict between different .bpl.

## Far future plans
* Trying to use somFree as opposed to IBM SOM
* Ada emitter in Ada
* Trying to run Windows version of somFree on Wine as opposed to using Linux version of somFree
* Same for Mac OS X
* Creating some kind of standard library that is compiled natively on different platforms, but thanks to somFree can be called from a single executable
* Cut WinAPI out of Wine as much as possible forcing to use somFree for file access and so on.
* This should become "Somatic Runtime", it should be cross-platform like Java, but natively compiled. x86 and x64 PE/COFF executables should run on every x86 and x64 based OS the same way as Java applications do. Having several versions of the same application for the same CPU architecture, but for Windows, Linux, Mac OS X, Solaris, FreeBSD is stupid, and in practice unpopular OSes lack programs, or they cannot be compiled without modifications. Only Java and Python work well everywhere. But we don't want to live in world full of Java and Python, right? So let it be clear: one architecture - one binary. End of story. Linux was hard to emulate on Windows, I don't know good user-level emulator. Cywin requires recompilation, and it's damn fork() does not work well on Windows Vista and higher due to randomization of address space. Java, on the other hand, works well in Windows Vista. You wonder why? Because there is no damn fork() in Java. And if you think that's a problem of Windows, let me give you one more problem. Run Linux program on Mac OS X without full-blown virtual machine. I know there is Linux emulator in FreeBSD, and there are Containers in Solaris allowing to run Linux binaries. While Linux emulation in FreeBSD is excellent example of what can be helpful in creating Somatic Runtime, creating container in Solaris requires work; and user level emulation of Linux in Mac OS X is not done at all. At the same time we have Wine for every of these platforms, so it is the only solution for now. arm-hf and mipsel targets, however, might not be based on Wine.
* Somatic Runtime should be usable from different programming languages, but there should be a programming language best suited for it. For instance, hypothetical Delphi with Somatic Extensions might have some tools to consume and produce SOM classes, but it can't easily adopt several restrictions required in order to maintain release-to-release binary compatibility. If you know Delphi well, you should be aware that .dcu from one version of Delphi is impossible to use from another version of Delphi. And also change in one .dcu requires recompilation of everything that is dependent on it. Ada and C++ have similar problems, named "fragile base class problem". This is what SOM solves. Programming languages can easily adopt extensions, but adopting restrictions is often impossible, there will be lots of code that cannot be compiled. If we don't have a code that can be compiled with these new restrictions, it makes sense to have another language, let it be named Somatic Pascal, designed with SOM in mind from the very beginning.