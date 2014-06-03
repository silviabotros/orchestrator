### Who should use orchestrator?

DBAs and ops who have more than a mere single-master-single-slave replication topology.

### What can orchestrator do for me?

Orchestrator analyzes your replication topologies. It can visualize those topologies, and it allows you to 
move slaves around the topology easily and safely. It provides full audit to operations making for a 
topology changelog. It can serve as a command line tool or it can provide with JSON API for all operations.

### Is this yet another monitoring tool?

No. Orchestrator is strictly _not_ a monitoring tool. There is no intention to make it so; no alerts or emails. It does provide with online visualization of your topology status though, and requires some thresholds of its own in order to manage the topology.

### What kind of replication does orchestrator support?

Orchestrator supports "plain-old-MySQL-replication", the one that uses binary log files and positions. 
If you don't know what you're using, this is probably the one. It is the only type of replication up to and including MySQL 5.5.

### Does orchestrator support Row Based Replication?

Yes. Statement Based Replication and Row Based Replication are both supported (and are in fact irrelevant)

### Does orchestrator support Master-Master (ring) Replication?

Yes, for a ring of two masters (active-active, active-passive). Do note that the tree visualization cannot present the circular replication, 
and will pick an arbitrary master as the root of the tree.

Master-Master-Master[-Master...] topologies, where the ring is composed of 3 or more masters are not supported and not tested. 
And are discouraged. And are an abomination.

### Does orchestrator support Galera Replication?

Yes and no. Orchestrator is unaware of Galera replication. If you have three Galera masters and different slave topologies under each master, 
then orchestrator sees these as three different topologies.

### Does orchestrator support GTID Replication?

Not at this stage. This is mainly because the developers of orchestrator feel GTID is not yet complete, 
and are anyhow not using a MySQL version which supports GTID. It is likely that GTID will be supported in the future.

### Does orchestrator support Parallel Replication?

No. This is because `START SLAVE UNTIL` is not supported in parallel replication, and output of `SHOW SLAVE STATUS` is incomplete. 
There is no expected work on this.

### Does orchestrator support Multi-Master Replication?

No. Multi Master Replication (e.g. as in MariaDB 10.0) is not supported.

### Does orchestrator support Tungsten Replication?

No.

### Does orchestrator support Yet Another Type of Replication?

No.

### Does orchestrator support...

No.

### Is orchestrator open source?

Yes. Orchestrator is released as open source under the Apache 2.0 license and is available at: https://github.com/outbrain/orchestrator

### Who develops orchestrator and why?

Orchestrator is developed by Shlomi Noach at Outbrain to assist in managing multiple large replication topologies; time and human errors saved so far are almost priceless.
