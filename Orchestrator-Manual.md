# About

_Orchestrator_ is a MySQL replication topology management and visualization tool, allowing for:

* Detection and interrogation of replication clusters
* Safe topology refactoring: moving slaves around the topology
* Sleek topology visualization
* Replication problems visualization
* Topology changes via intuitive drag & drop
* Maintenance mode declaration and enforcement
* Auditing of operations
* More...

Refactoring the topology is a matter of a simple drag & drop. _Orchestrator_ will keep you safe by disallowing invalid replication topologies 
(e.g. replicating from ROW based to STATEMENT based, from 5.5 to 5.1 etc.)

_Orchestrator_ is developed at [Outbrain](http://www.outbrain.com/) to answer for the difficulty in managing replication topologies. 
At the time of orchestrator creation GTID is available via MySQL 5.6 and MariaDB 10.0. And yet GTID is still not as mature as one would hope for. 
The majority of users still use plain-old binlog file:position based MySQL replication, and apparently this will hold for some time. 

# License
_Orchestrator_ is released as open source under the [Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0)

# Download
_Orchestrator_ is released as open source and is available at [GitHub](https://github.com/outbrain/orchestrator). Find official releases in https://github.com/outbrain/orchestrator/releases
 
# Requirements
_Orchestrator_ is a standalone Go application. It requires a MySQL backend to hold topologies state, maintenance status and audit history. 
It is built and tested on Linux 64bit, and binaries are availably for this OS type alone. The author has not tested any other operating system, 
though any other unix-like OS should do just fine.

# Installation
The following assumes you will be using the same machine for both the orchestrator binary and the MySQL backend. 
If not, replace `127.0.0.1` with appropriate host name. Replace `orch_backend_password` with your own super secret password.

Setup a MySQL server for backend, and invoke the following:

    CREATE DATABASE IF NOT EXISTS orchestrator;
    GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'127.0.0.1' IDENTIFIED BY 'orch_backend_password';

Orchestrator uses a configuration file, located in either `/etc/orchestrator.conf.json` or relative path to binary `conf/orchestrator.conf.json` or 
`orchestrator.conf.json`. Edit this file to match the above as follows:

    ...
    "MySQLOrchestratorHost": "127.0.0.1",
    "MySQLOrchestratorPort": 3306,
    "MySQLOrchestratorDatabase": "orchestrator",
    "MySQLOrchestratorUser": "orchestrator",
    "MySQLOrchestratorPassword": "orch_backend_password",
    ...

For orchestrator to detect your replication topologies, it must also have an account on each and every topology. At this stage this has to be the 
same account (same user, same password) for all topologies. On each of your masters, issue the following:

    GRANT SUPER, PROCESS ON *.* TO 'orchestrator'@'orch_host' IDENTIFIED BY 'orch_topology_password';

Replace `orch_host` with hostname or orchestrator machine (or do your wildcards thing). Choose your password wisely. Edit `orchestrator.conf.json` to match:

    "MySQLTopologyUser": "orchestrator",
    "MySQLTopologyPassword": "orch_topology_password",
