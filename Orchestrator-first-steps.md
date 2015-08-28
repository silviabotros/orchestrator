## Orchestrator first steps

You have _Orchestrator_ installed and deployed. What can you do with it?

A walk through common commands, mostly on the CLI side

#### Must

##### Discover

You need to discover your MySQL hosts. Either browse to your `http://orchestrator:3000/web/discover` page and submti an instance for discovery, or:

	orchestrator -c discover some.mysql.instance.com:3306
	
The `:3306` is not required, since the `DefaultInstancePort` configuration is `3306`. You may also

	orchestrator -c discover some.mysql.instance.com

This discovers a single instance. But: do you also have an _orchestrator_ service running? It will pick up from there and 
will interrogate this instance for its master and slaves, recursively moving on until the entire topology is revealed.

> By the way, you can also run a service-like _orchestrator_ from the command line:
>
>	orchestrator -c continuous
>
