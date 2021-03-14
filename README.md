# Discoh
A distributed system is said to be coherent, if and only if, all components of the system, at any time, has a consistent view of the last written value for each key.
Discoh enables software engineers to build real-time distributed system that provide the benefits of availability and consistent data and can be quickly integrated with both memcached and Redis.
Simply change your client code to use syncPut(), choose your consistency protocol in your Discoh Cluster Configuration, bring up one or more instances of Discoh in between your memcached or Redis Cluster and your load balancer and you are good to go. 
