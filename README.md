# discoh
discoh stands for **dis**tributed **coh**erence. 
## What is distributed Coherence?
A distributed system is said to be coherent if and only if, 
  1. all components of the system
  2. have a consistent view of the last written value 
  3. of every key
  4. at any given time.
## How does discoh help me?
If your distriuted system goal is strong consistency without sacrificing availability, Discoh helps you achieve strong consistency with your existing infrastructure. Discoh can be quickly integrated with your existing infrastructure containing either memcached or Redis cache clusters.
## How do I get started?
1. Change your client code to include the following rpc call: v' = discoh_read(k) in parallel along with a v = get(k) to your cache server, whenever you do a read. 
- If discoh_read returns a value, your client takes that value, otherwise the client will take the value v returned by the cache. 
- Don't worry the discoh cache would be invalidated every 60 seconds. (You can change this setting if you want to).
2. Change your client code to include the following rpc call: discoh_pull(k), whenever your client does a write to the database.
- discoh_pull(k) would do a fetch the value from the DB.
- You need to configure the database connection parameters.
3. Deploy a cluster of discoh servers, to mirror the cluster of cache servers.
- If you used consistent hashing to load balance your cache servers, you are in luck, as discoh uses consistent hashing as well. 
## How does the architecture work?
discoh uses the directory-based coherence protocol which is a proven cache coherence mechanism in parallel processors and multicore processors have been in production with the directory based coherence mechanism since 2005.
