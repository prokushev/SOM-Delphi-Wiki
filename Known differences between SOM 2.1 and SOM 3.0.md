SOM 2.1 for Windows is compiled with Microsoft C, and SOM 3.0 for Windows is compiled with VisualAge for C++.

Both have samples, and samples contain compiler switches. SOM 2.1 contain compiler switches for both Microsoft C and VisualAge for C++, SOM 3.0 only contains VisualAge for C++ switches.

VisualAge for C++ allocates as little space for enum as possible (controlled by /Su switch), while Microsoft C allocates int (4 bytes). This is notable when opening emitdef.dll (or any other emitter) in IDA. Where SOM 2.1 has "dword ptr [eax+4]", SOM 3.0 has "byte ptr [eax+4]". All Emitter Framework enums are affected! They are hacked to be C enums as opposed to normal SOM enums.

This is a quote from VisualAge for C++ User's Guide:  
Use /Su to control the size of enum variables. If you do not provide a size, all enum variables are made 4 bytes.  
By default, the compiler uses the SAA rules: make all enum variables the size of the smallest integral type that can contain all variables.  
You can specify the following sizes:  
 /Su[+] Make all enum variables 4 bytes.  
 /Su1   Make all enum variables 1 byte.  
 /Su2   Make all enum variables 2 bytes.  
 /Su4   Make all enum variables 4 bytes.

With regards to the switches used to build samples (found in samples\VACMAKE.HD and samples\MSCMAKE.HD):

SOM 3.0:  
CFLAGSCOMMON = /Ti /O- /Os- /W1 /H128           /Q+ /c /Gd+ /Gm+

SOM 2.1:  
CFLAGSCOMMON = /Ti /O- /Os- /W1 /H128 /Gs+ /Sp1 /Q+ /c /Gd+ /Gm+

/Ti   Generate debugger information.  
/O-   Optimize code.  
/Os-  Invoke the instruction scheduler.  
/W1   Set severity level of messages the compiler produces and counts.  
/H128 Set maximum length of external names.  
/Gs+  Remove stack probes.  
/Sp1  Specify alignment or packing of data items within structures and unions.  
/Q+   Suppress the compiler logo when invoking the compiler.  
/Gd+  Dynamically link to the runtime library, instead of linking statically.  
/Gm+  Link with the multithread library, instead of the single-thread library.

You should see the difference here.

/Gs+  Remove stack probes.  
/Sp1  Specify alignment or packing of data items within structures and unions.  

The default is /Sp8.

This is VisualAge for C++ User Guide again:  
Use /Sp to specify alignment or packing of data items within structures and unions.  
By default, structures and unions are aligned along 8-byte boundaries (normal alignment).  
/Sp is equivalent to /Sp1.

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

Also, IBM VisualAge for C++ v3.5 has DirectToSOM sample, and it has independent flags:

DEBUGFLAGS=/Ti /O- /Os-  
BROWSEFLAGS=/Fb  
PERFLAGS=/Gh  
CPPFLAGS=$(DEBUGFLAGS) $(BROWSEFLAGS) $(PERFLAGS)  

Note that "/Sp1 Packing of data items" is not present, and so is "/Su" for enum sizes. IDA shows that SOM_CreateLocalEnvironment() macro in xhmain.cpp expands into SOMCalloc for 16 bytes. So enum size might be 1 here, but aligned pointers after "exception_type" expand this structure into 16-byte one. Unfortunatelly, xhmain.cpp does not check __SOMEnv->_major.

Also, IBM VisualAge for C++ v3.5 has some kind of OR/M in SOM sample (IBMCPPW\SAMPLES\database\stocksom\CLIENT.CPP). It has yet another flags:  
icc.exe /Tdp /Sn /Gh /Ti /Gm /Gd /I. /Fb /Fo"%|dpfF.obj" /C .\prclist.cpp

Interesting to note, there is also no /Sp1 here. And also no "/Su". IDA shows that SOM.DLL clearly expects Environment._major to be 4 byte. Happily for us, this sample has exception handling code, and we can check our expectations:


```
#!assembler

txt0:00445E7C                 xor     eax, eax
txt0:00445E7E                 mov     al, [ecx]
txt0:00445E80                 cmp     eax, 2
txt0:00445E83                 jz      short loc_445E97
txt0:00445E85                 cmp     eax, 1
txt0:00445E88                 jz      short loc_445EC0
txt0:00445E8A                 test    eax, eax
txt0:00445E8C                 jz      loc_445FDA

```

Gotcha! So in SOM 2.1 ABI enums (not only SOM enums, but system ones) must be 4 bytes because Microsoft C compiler was used to compile SOM 2.1, and it is supposed to be frozen afterwards. However, VisualAge C++ samples never use /Su compiler option to make their enums 4 bytes. They have seemingly got strange errors, but tried to cure them with alignment option as opposed to enum options. So far, there can be another strange errors. Wrt environment structure it is being initialized with zeroes, so there should be no problem. No matter if you read bytes or double words, you get the same result. But in some other circumstances (like emitters) one side might write byte and another one might read double word and get garbage.

And the final test is to use TypeCode_size on hand-made types:


```
#!delphi

procedure TestAlignmentAndSize;
var
  MyTC: TypeCode;
begin
  MyTC := TypeCode.Create(tk_struct, ['Test_1',
                            'VByte', TypeCode.Create(tk_octet),
                            'VInteger', TypeCode.Create(tk_long)
                          ]);
  WriteLn('Test_1 size: ', MyTC.Size);
  MyTC.Free;
  MyTC := TypeCode.Create(tk_struct, ['Test_2',
                            'VByte', TypeCode.Create(tk_octet),
                            'VSubrecord', TypeCode.Create(tk_struct, ['Test_1',
                              'VByte', TypeCode.Create(tk_octet),
                              'VInteger', TypeCode.Create(tk_long)
                            ])
                          ]);
  WriteLn('Test_2 size: ', MyTC.Size);
  MyTC.Free;
  MyTC := TypeCode.Create(tk_struct, ['Test_3',
                            'VByte', TypeCode.Create(tk_octet),
                            'VSubrecord', TypeCode.Create(tk_struct, ['Test_4',
                              'VByte', TypeCode.Create(tk_octet),
                              'VByte', TypeCode.Create(tk_octet),
                              'VDouble', TypeCode.Create(tk_double)
                            ])
                          ]);
  WriteLn('Test_3 size: ', MyTC.Size);
  MyTC.Free;
end;

(*
Output on SOM 2.1:
Test_1 size: 5
Test_2 size: 6
Test_3 size: 11

Output on SOM 3.0:
Test_1 size: 8
Test_2 size: 12
Test_3 size: 24
*)

```

So, clearly, that's not just IBM developers mishandled compiler flags. That's indeed different ABI in SOM 2.1 and SOM 3.0. But the issue with mishandled compiler flags is also present.
