Overview:

Added a new command "stats cachedumps" to extract some useful information from the items stored by memcached. 
The new command provides information on access frequency and last access time for all slabs.
The previous command "stats cachedump x y" gives the lru list of max y items for the given slab with id x. 

The new command "stats cachedumps y" gives the lru list of items for all the slabs, slab by slab, where y is the max items. The details for each
item also contain the least recent access time and the number of times the item was referenced. 

Loading the keys:
We store the keys on to memcached by executing the "keys" file (./keys). It adds items of different sizes, so that they fall into 
more than one slab.
To get an item from the key value, we use `'get key25\r' | nc localhost 11211`. This increases the reference count of key25.


To build and test:

Download the memcached source code and update with the two included files.

```
configure & make
```

To telnet: `telnet localhost 11211`.

So, after we telnet, "stats cachedumps <max number of items you want to see in each slab> 
Example : `stats cachedumps 5`. would give the items list in each slab. 

The result of `stats cachedumps 5` would be something like :

```
ITEM key25 [3 b; 1463765727 s; 174 s; 4 ]
ITEM key52 [5 b; 1463765727 s; 64 s; 1 ]
ITEM key51 [3 b; 1463765727 s; 64 s; 1 ]
ITEM key50 [3 b; 1463765727 s; 64 s; 1 ]
ITEM key49 [3 b; 1463765727 s; 64 s; 1 ]
END

END

END

ITEM key53 [120 b; 1463765727 s; 64 s; 1 ]
END
```
For each item, the third column time indicates the last access time, the forth column gives the refcount. 
Here please note that while we set the item, the refcount is being set to 1. So, 
after the first get, refcount would be 2 and so on. The End here indicates the end of items for each slab.

These details of items can be used to further determine which items should be ignored while considering 
a global lru list over all slabs.
