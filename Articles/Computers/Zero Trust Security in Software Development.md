Link: https://cacm.acm.org/blogcacm/zero-trust-security-in-software-development/

1. **Zero trust** = treat every action in the system as malicious unless proven otherwise. Earlier the motto was "trust but verify".
2. The idea is that since no one or nothing can be trusted, apply the following principles to all parts of development and operation. These principles are
	1. Micro-Segmentation : partition an entity such that an infection in one cannot migrate to another. Such as have isolated environments for dev, test and prod.
	2. Least Privileged Access : not only processes, but personnel must also only have as much access as their role needs.
	3. Zero Trust Networking : do not implicitly trust a component even when it's internal such a machine in local network
	4. Continuous Authentication : monitor persona (developer, administrator, or user) in real-time. You can no longer authenticate and chill, every action must be authenticated and monitored for anomalous behaviour
3. Parts of development and operations that must be protected by these principles (well all parts must be protected but the following usually don't appear into our imagination unless called out explicitly)
	1. Source code - everyone should get to see it (the more eyes to analyse, the better), but not everyone should be able to alter it
	2. Third Party Code - we love to use new libraries but we don't know what malaise they carry. Supply chain attacks have rocked many worlds.
	3. Deployment pipelines - one wrong deployment setting can spill water over the state-of-the art design pattern used to secure the system
4. Will people buy a piece of software if they see the phrase "zero trust" on the packaging? I don't think so. It actually depends,
	1. If the customer is not the user, usually large and complex enterprises fit this description, then the purchasing manager may not worry too much about the ease of use for the operator. Additionally, higher stakes or regulations may nudge them towards fancy guarantees.
	2. But if the customer is the user, usually small firms and retail customers, then the phrase might not work on them. They just wish to get by and assume that they're too small to hurt.
5. What would you look like doing if you were adopting zero trust?
	1. Securing your codebase, deployment pipelines and resource as well as third party library stores
	2. Incident response and continuous monitoring - every failure must be analyzed and failures as whole must be monitored to see if something is out of ordinary
	3. Securing (encrypting and integrity checking) data at rest (as it sits in hard disk), data in transit (as it flies through the network) and possibly data in use (as it is used in RAM, cache and processor registers)
	4. Evangelizing - most of us just want to clear our to do lists and we wouldn't mind taking harmless shortcuts to do it but these shortcuts may prove too costly when an unlikely event hits us because we took the proscribed way.
