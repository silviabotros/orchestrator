## About

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
At the time of _orchestrator_ creation GTID is available via MySQL 5.6 and MariaDB 10.0. And yet GTID is still not as mature as one would hope for. 
The majority of users still use plain-old binlog file:position based MySQL replication, and apparently this will hold for some time. 

![Orcehstrator screenshot](images/orchestrator-simple.png)

## License
_Orchestrator_ is released as open source under the [Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0)

## Download
_Orchestrator_ is released as open source and is available at [GitHub](https://github.com/outbrain/orchestrator). Find official releases in https://github.com/outbrain/orchestrator/releases
 
## Requirements
_Orchestrator_ is a standalone Go application. It requires a MySQL backend to hold topologies state, maintenance status and audit history. 
It is built and tested on Linux 64bit, and binaries are availably for this OS type alone. The author has not tested any other operating system, 
though any other unix-like OS should do just fine.

## Installation
The following assumes you will be using the same machine for both the _orchestrator_ binary and the MySQL backend. 
If not, replace `127.0.0.1` with appropriate host name. Replace `orch_backend_password` with your own super secret password.

#### Setup backend MySQL server

Setup a MySQL server for backend, and invoke the following:

    CREATE DATABASE IF NOT EXISTS orchestrator;
    GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'127.0.0.1' IDENTIFIED BY 'orch_backend_password';

_Orchestrator_ uses a configuration file, located in either `/etc/orchestrator.conf.json` or relative path to binary `conf/orchestrator.conf.json` or 
`orchestrator.conf.json`. Edit this file to match the above as follows:

    ...
    "MySQLOrchestratorHost": "127.0.0.1",
    "MySQLOrchestratorPort": 3306,
    "MySQLOrchestratorDatabase": "orchestrator",
    "MySQLOrchestratorUser": "orchestrator",
    "MySQLOrchestratorPassword": "orch_backend_password",
    ...

#### Grant access to orchestrator on all your MySQL servers
For _orchestrator_ to detect your replication topologies, it must also have an account on each and every topology. At this stage this has to be the 
same account (same user, same password) for all topologies. On each of your masters, issue the following:

    GRANT SUPER, PROCESS ON *.* TO 'orchestrator'@'orch_host' IDENTIFIED BY 'orch_topology_password';

Replace `orch_host` with hostname or orchestrator machine (or do your wildcards thing). Choose your password wisely. Edit `orchestrator.conf.json` to match:

    "MySQLTopologyUser": "orchestrator",
    "MySQLTopologyPassword": "orch_topology_password",

#### Extract orchestrator binary and files

Extract the archive you've downloaded from https://github.com/outbrain/orchestrator/releases
For example, let's assume you wish to install _orchestrator_ under `/usr/local/orchestrator`:

    sudo mkdir -p /usr/local/orchestrator
    sudo cd /usr/local/orchestrator
    sudo tar xzfv orchestrator-1.0.tar.gz
    
To execute _orchestrator_ in command line mode or in HTTP API only, all you need is the `orchestrator` binary. 
To enjoy the rich web interface, including topology visualizations and drag-and-drop topology changes, you will need 
the `resources` directory and all that is underneath it.

## Execution

#### Executing as web/API service

Assuming you've installed _orchestrator_ under `/usr/local/orchestrator`:

    cd /usr/local/orchestrator && ./orchestrator http

That's it, you're ready to go. You may skip to next sections.
    
If you like your debug messages, issue:

    cd /usr/local/orchestrator && ./orchestrator --debug http

The above looks for configuration in `/etc/orchestrator.conf.json`, `conf/orchestrator.conf.json`, `orchestrator.conf.json`, in that order.
Classic is to put configuration in `/etc/orchestrator.conf.json`. Since it contains credentials to your MySQL servers you may wish to limit access to that file. 
You may choose to use a different location for the configuration file, in which case execute:

    cd /usr/local/orchestrator && ./orchestrator --debug --config=/path/to/config.file http
 
Web/API service will, by default, issue a continuous, infinite polling of all known servers. This keeps _orchestrator_'s data up to date.
You typically want this behavious, but you may disable it, making _orchestrator_ just serve API/Web but never update the instances status:

    cd /usr/local/orchestrator && ./orchestrator --discovery=false http
    
The above is useful for development and testing purposes. You probably wish to keep to the defaults.

#### Executing as command line

Following is a synopsis of command line samples. For simplicitly, we assume `orchestrator` is in your path.
If not, replace `orchestrator` with `/path/to/orchestrator`.

Samples below use a test `mysqlsandbox` topology, where all instances are on same host `127.0.0.1` and on different ports. `22987` is master, 
and `22988`, `22989`, `22990` are slaves. 

Show currently known clusters (replication topologies):

    orchestrator -c clusters cli
    
The above looks for configuration in `/etc/orchestrator.conf.json`, `conf/orchestrator.conf.json`, `orchestrator.conf.json`, in that order.
Classic is to put configuration in `/etc/orchestrator.conf.json`. Since it contains credentials to your MySQL servers you may wish to limit access to that file. 
You may choose to use a different location for the configuration file, in which case execute:

    orchestrator -c clusters --config=/path/to/config.file cli

Discover a new instance ("teach" _orchestrator_ about your topology). _Orchestrator_ will automatically recursively drill up the master chain (if any)
and down the slaves chain (if any) to detect the entire topology:

    orchestrator -c discover -i 127.0.0.1:22987 cli

Do the same, and be more verbose:

    orchestrator -c discover -i 127.0.0.1:22987 --debug cli

`--debug` can be useful in all operations.

Forget an instance (an instance may be manually or automatically re-discovered via `discover` command above):

    orchestrator -c forget -i 127.0.0.1:22987 cli
    
Print an ASCII tree of topology instances. Pass a cluster name via `-i` (see `clusters` command above):

    orchestrator -c topology -i 127.0.0.1:22987 cli
    
Sample output:

    127.0.0.1:22987
    + 127.0.0.1:22989
      + 127.0.0.1:22988
    + 127.0.0.1:22990
        
Move a slave up the topology (make it sbling of its master, or direct slave of its "grandparent"):

    orchestrator -c move-up -i 127.0.0.1:22988 cli

The above command will only succeed if the instance _has_ a grandparent, and does nto have _problems_ such as slave lag etc.

Move a slave below its sibling:

    orchestrator -c move-below -i 127.0.0.1:22988 -s 127.0.0.1:22990 --debug cli

The above command will only succeed if `127.0.0.1:22988` and `127.0.0.1:22990` are siblings (slaves of same master), none of them has _problems_ (e.g. slave lag),
and the sibling _can_ be master of instance (i.e. has binary logs, has `log_slave_updates`, no version collision etc.)

        
Begin maintenance mode on an instance. While in maintenance mode, _orchestrator_ will not allow moving this instance:

    orchestrator -c begin-maintenance -i 127.0.0.1:22988 cli

End maintenance mode on an instance:

    orchestrator -c end-maintenance -i 127.0.0.1:22988 cli

Make an infinite, continuous discovery and investigation of known instances. Typically this is what the web service executes.

    orchestrator -c continuous --debug cli


## Using the Web interface

## Using the web API

If you're a keen web developer, you can see (via Firebug or Developer Tools) how the web interface 
completely relies on JSON API requests. 

The JSON API provides with all the maintenance functionality you can find in the web interface or the 
command line mode.

Most users will not be interested in accessing the API. If you're unsure: you don't need it.
For creators of frameworks and maintenance tools, it may provide with great powers (and great responsibility).

The following is a brief listing of the web API exposed by _orchestrator_:

* `/api/instance/:host/:port`: reads and returns an instance's details (example `/api/instance/mysql10/3306`)
* `/api/discover/:host/:port`: discover given instance and all the topology it is associated with (example `/api/discover/mysql10/3306`)
* `/api/refresh/:host/:port`: synchronously re-read instance status 
* `/api/forget/:host/:port`: remove records of this instance. It may be automatically rediscovered by 
  following up on its master or one of its slaves. 
* `/api/move-up/:host/:port` (attempt to) move this instacne up the topology (make it child of its grandparent)
* `/api/move-below/:host/:port/:siblingHost/:siblingPort` (attempt to) move an instance below its sibling.
  the two provided instances must be siblings: slaves of the same master. (example `/api/move-below/mysql10/3306/mysql24/3306`)
* `/api/begin-maintenance/:host/:port/:owner/:reason`: declares and begins maintenance mode for an instance.
  While in maintenance mode, _orchestrator_ will not allow moving this instance. 
  (example `/api/begin-maintenance/mysql10/3306/gromit/upgrading+mysql+version`)
* `/api/end-maintenance/:host/:port`: end maintenance on instance
* `/api/end-maintenance/:maintenanceKey` end maintenance on instance, based on maintenance key given on `begin-maintenance` 
* `/api/start-slave/:host/:port`: issue a `START SLAVE` on an instance 
* `/api/stop-slave/:host/:port`: issue a `STOP SLAVE` on an instance 
* `/api/maintenance`: list instances in active maintenance mode
* `/api/cluster/:clusterName`: list instances in a topology cluster. Each topology is automatically given a unique 
  name. At this point the name is set by the topology's master (and if there's a master-master setup, then one of the masters).
  For example, a topology's name might be `mysql10:3306`, based on the understanding the server `mysql10` on port `3306`
  is the master of the topology.  
* `/api/clusters`: list names of known topologies. 
* `/api/search/:searchString`: list instances matching search string
* `/api/problems`: list instances who have known problems (e.g. not replicating, lagging etc.) 
* `/api/audit`: show most recent audit entries
* `/api/audit/:page`: show latest audit entries, paginated (example: `/api/audit/3` for 3rd page)  


## Risks

Most of the time _orchestrator_ only reads status from your topologies. Default configuration is to poll each instance once per minute.
This is very relaxed, and you can go way more intensive than that. But do be aware that _orchestrator_ opens connections to all your servers
(and typically reuses them).

Actual risk begins when you modify instances. Namely moving slaves around the topology. _Orchestrator_ does its best to:

1. Make sure you only move an instance to a location where it is valid for it to replicate (e.g. that you don't put a 5.5 server below a 5.6 server)
2. Make sure you move an instance at the right time (ie the instance and whicever affected servers are not lagging badly, so that operation can compeltely in a timely manner).
3. Do the math correctly: stop the slve at the right time, roll it forward to the right position, `CHANGE MASTER` to the correct location & position.

All the above is tested, and have been put to practice in our production topologies. We have not witnessed a miscalculation or misprotection throughout our production use.

When _orchestrator_ encounters an error throught the moving process, it does its best to rollback. However extreme cases such as a new master crashing in the middle of the move
may leave the topology unstable (though the same instance could crash before the mvoe and leave whatever topology it was in charge of unstable just as well). Point being - weird
and evil stuff can happen, and there is a risk in a slave losing its position vs. its master.

Now that you're a bit more scared, it's time to reflect: how much did your hands tremble when you navigated your slaves _by hand_ up and down through the topology? 
We suspect the automation provided by _orchestrator_ makes for a _safer_ management mechanism than we get with our shaking hands.

Also, read the [LICENSE](https://github.com/outbrain/orchestrator/blob/master/LICENSE).
 
 
 