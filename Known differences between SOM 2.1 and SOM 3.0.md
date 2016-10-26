SOM 2.1 for Windows is compiled with Microsoft C, and SOM 3.0 for Windows is compiled with VisualAge for C++.

Both have samples, and samples contain compiler switches. SOM 2.1 contain compiler switches for both Microsoft C and VisualAge for C++, SOM 3.0 only contains VisualAge for C++ switches.

VisualAge for C++ allocates as little space for enum as possible (controlled by /Su switch), while Microsoft C allocates int (4 bytes). This is notable when opening emitdef.dll (or any other emitter) in IDA. Where SOM 2.1 has "dword ptr [eax+4]", SOM 3.0 has "byte ptr [eax+4]". All Emitter Framework enums are affected! They are hacked to be C enums as opposed to normal SOM enums.

With regards to the switches used to build samples (found in samples\VACMAKE.HD and samples\MSCMAKE.HD):

SOM 3.0:
CFLAGSCOMMON = /Ti /O- /Os- /W1 /H128           /Q+ /c /Gd+ /Gm+

SOM 2.1:
CFLAGSCOMMON = /Ti /O- /Os- /W1 /H128 /Gs+ /Sp1 /Q+ /c /Gd+ /Gm+

You should see the difference here.

/Gs+ Remove stack probes
/Sp1 Packing of data items

SOM 2.1 (MSVC):
CFLAGSCOMMON = /MT /G4 /Gs /Zp /Od /H128 /Zi /c /D_WIN32

/MT
/G4   G4  386 instructions, optimize for 486
/Gs   Controls stack probes.
/Zp   Packs structure members.
/Od   Disables optimization.
/H128 Deprecated. Restricts the length of external (public) names.
/Zi   Generates complete debugging information.
/c
/D_WIN32

So it looks like SOM 2.1 packs structures, there are switches for both Microsoft C and VAC; and SOM 3.0 does not. At the moment no any known record can exhibit the difference, but this should be further investigated, and Delphi records should probably be made packed ones.

Also note the difference in headers:
2.1:
typedef enum completion_status {YES, NO, MAYBE} completion_status;
3.0:
typedef enum completion_status {YES, NO, MAYBE,
    completion_status_MAX = 2147483647    /* ensure mapped as 4 bytes */
} completion_status;

As soon as MSVC compiler is used for SOM 2.1 samples, the enum is 4 bytes and consistent with SOM DLL ABI, but VAC compiler is supposedly broken. /Sp1 should make the structure Environment packed, change size and offsets in Environment. That makes it harder to verify assumptions about ABI. The first option is IDA, it unveils how do original DLLs work with data. The second option is to construct custom tk_struct TypeCodes and check tcSize on them.