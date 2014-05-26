
# Installation
The following assumes you will be using the same machine for both the orchestrator binary and the MySQL backend. If not, replace `127.0.0.1` with appropriate host name. Replace `orch_backend_password` with your own super secret password.

Setup a MySQL server for backend, and invoke the following:

    CREATE DATABASE IF NOT EXISTS orchestrator;
    GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'127.0.0.1' IDENTIFIED BY 'orch_backend_password';

Orchestrator uses a configuration file, located in either `/etc/orchestrator.conf.json` or relative path to binary `conf/orchestrator.conf.json` or `orchestrator.conf.json`. Edit this file to match the above as follows:

    ...
    "MySQLOrchestratorHost": "127.0.0.1",
    "MySQLOrchestratorPort": 3306,
    "MySQLOrchestratorDatabase": "orchestrator",
    "MySQLOrchestratorUser": "orchestrator",
    "MySQLOrchestratorPassword": "orch_backend_password",
    ...

For orchestrator to detect your replication topologies, it must also have an account on each and every topology. At this stage this has to be the same account (same user, same password) for all topologies. On each of your masters, issue the following:

    GRANT SUPER, PROCESS ON *.* TO 'orchestrator'@'orch_host' IDENTIFIED BY 'orch_topology_password';

Replace `orch_host` with hostname or orchestrator machine (or do your wildcards thing). Choose your password wisely. Edit `orchestrator.conf.json` to match:

    "MySQLTopologyUser": "orchestrator",
    "MySQLTopologyPassword": "orch_topology_password",
