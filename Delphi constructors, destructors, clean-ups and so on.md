```
#!

Outer-Create
Outer-Create => virtual NewInstance
Outer-Create => virtual NewInstance => _GetMem
Outer-Create => virtual NewInstance
Outer-Create => virtual NewInstance => non-virtual InitInstance
Outer-Create => virtual NewInstance => non-virtual InitInstance => FillChar(0)
Outer-Create => virtual NewInstance => non-virtual InitInstance
Outer-Create => virtual NewInstance
Outer-Create
Outer-Create => Create
Outer-Create
Outer-Create => virtual AfterConstruction
Outer-Create

non-virtual Free
non-virtual Free => Outer-Destroy
non-virtual Free => Outer-Destroy => virtual BeforeDestruction
non-virtual Free => Outer-Destroy
non-virtual Free => Outer-Destroy => Destroy
non-virtual Free => Outer-Destroy
non-virtual Free => Outer-Destroy => virtual FreeInstance
non-virtual Free => Outer-Destroy => virtual FreeInstance => non-virtual CleanupInstance
non-virtual Free => Outer-Destroy => virtual FreeInstance => non-virtual CleanupInstance => _FinalizeRecord
non-virtual Free => Outer-Destroy => virtual FreeInstance => non-virtual CleanupInstance
non-virtual Free => Outer-Destroy => virtual FreeInstance
non-virtual Free => Outer-Destroy => virtual FreeInstance => _FreeMem
non-virtual Free => Outer-Destroy => virtual FreeInstance
non-virtual Free => Outer-Destroy
non-virtual Free

```
