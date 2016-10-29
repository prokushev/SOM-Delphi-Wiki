http://octagram.name/pub/somobjects/3.0beta/pdf/som/sdclient.pdf

In contrast, the default policy in DSOM 2.x was a combination of caller-owned for in and inout parameters, dual-owned for out parameters and return results, and suppress_inout_free for inout parameters.

To the caller, dual-owned looks no different than the default behavior. On the server side, dual-owned looks like object-owned; DSOM allocates and initializes the parameters, but frees nothing after method completion, except certain introduced pointers.

in      => caller-owned  
inout   => caller-owned, suppress_inout_free  
out     => dual-owned (object_owns on local and server)  
result  => dual-owned (object_owns on local and server)
