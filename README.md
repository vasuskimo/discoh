# discoh
discoh stands for **dis**tributed **coh**erence. 
## What is Distributed Coherence?
A distributed system is said to be coherent if and only if, 
  1. all components of the system
  2. have a consistent view of the last written value 
  3. of every key
  4. at any given time.
## How does discoh help me?
If your distributed system goal is close to strong consistency without sacrificing availability, discoh helps you achieve close to strong consistency with your existing infrastructure. discoh can be quickly integrated with your existing infrastructure containing any cache cluster such as memcached or Redis or using discoh itself.
## How do I get started?
1. Change your client code to include the following rpc call: v' = discoh_read(k) in parallel along with a v = get(k) to your cache server, whenever you do a read. 
- If discoh_read returns a value, your client takes that value, otherwise the client will take the value v returned by the cache. 
- Don't worry the discoh cache would be invalidated every 60 seconds. (You can change this setting if you want to).
2. Change your client code to include the following rpc call: discoh_pull(k), whenever your client does a write to the database.
- discoh_pull(k) would do a fetch the value from the DB.
- You need to configure the database connection parameters.
3. Deploy a cluster of discoh servers, to create shards of discoh servers using consistent hashing to load balance your cache servers.
4. Deploy one cluster in one region (e.g., East Region) and another in the second region (e.g., West Region).
## Do you have a reference architecture?
Sure. Here is the Facebook Memcached Architecture per the published paper.
![facebook memcached architecture](Facebook_Memcached_Architecture.png)

Here is how reads happen in the client per the original FB architecture:

```
v = get(k);
if v is nil // on a cache miss
   v = fetch_from_db(k)
   set(k,v)  // set the cache with the key,value pair
```

Here is the modified read in the client with discoh in the Enhanced FB architecture:

```
v = discoh_read(k);
v' = get(k);
if v is nil 
   if v' is nil // on a cache miss
     v = fetch_from_db(k)
     set(k,v)  // set the cache with the key,value pair
   else
      v = v'
```
![read and write flow](Memcache_read_write.png)

Here is how writes happen in the client per the original FB architecture:
```
upsert_in_db(k,v) // update or insert the key/value pair in the db
delete(k) // delete the key from the cache
```

Here is the modified write in the client with discoh in the Enhanced FB architecture:

```
upsert_in_db(k,v) // update or insert the key/value pair in the db
delete(k) // delete the key from the cache
discoh_pull(k)
```

For more detailed information refer to the original paper <http://www.cs.utah.edu/~stutsman/cs6963/public/papers/memcached.pdf>

## What are the configurations needed to get discoh up and running?
You can run discooh as either as a default transient cache for writes (--type default) or a cache server (--type cache)
You need to add a config.txt before running it with the following key and value pairs.
1. ttl=30 You could change the default cache ttl from 60 secs.
2. You need to add connection parameters to the database. (mysql, oracle, sqlserever and postgres).
3. You need to add the SQL query used by clients to fetch the key and value. 
4. Here is a sample config.txt
- db=customer
- username=john
- password=jdoe123
- type=mysql
- host=https://acmedddd.com:3306/
- command=select username as key, city as value from users
- peer_ip=discoh223.acmedddd.com
- host_port=7654
- peer_port=7654
5. Run from the terminal the following: discoh --config config.txt

## Tokens
1. In order to prevent stale content, a discoh instance gives a lease to a client to set data back into the cache when that client experiences a cache miss. 
2. The lease is a 64-bit token bound to the specific key the client originally requested. 
3. The client provides the lease token when setting the value in the cache. 
4. With the lease token, discoh can verify and determine whether the data should be stored and
thus arbitrate concurrent writes. 
5. Verification can fail if discoh has invalidated the lease token due to receiving a delete request for that item. 
6. A slight modification to leases also mitigates thundering herds. 
7. Each discoh server regulates the rate at which it returns tokens defaulting to 10 seconds per key.
8. Requests for a key’s value within 10 seconds of a token being issued results in a special notification informing the client to wait a short amount of time. 
9. Typically, the client with the lease will have successfully set the data within a few milliseconds. Thus, when waiting clients
retry the request, the data is often present in cache.
