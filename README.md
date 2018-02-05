
# jboss Init Scripts

## Description

Scripts shell for Jboss (ajp/cluster, standalone) maintenance.

### [jbossInitScript (cluster)](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/jbossInitScript.cluster.sh)

This script is useful for maintenance (stop / start) of jboss instances from a single node
Required trust relationship (rsa/dsa key exchange) between nodes.

#### Operation

this script supports "stop | start | status | restart" operations on nodes contained in the "/etc/hosts" file.
To execute the command on the remote nodes, it is necessary to ensure a trust relationship (RSA/DSA key exchange) between the hosts.

#### Usage

```
$ sudo /etc/init.d/jboss            {stop|start|status|restart} {all|00}
    .                .                           .                 .
    .                .                           .                 .
    .                .                           .                 ...... Node (Ex. 01 ou all)
    .                .                           ........................ Operation
    .                .................................................... Script
    ..................................................................... Run as super user

```

### [jbossInitScript (standalone)](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/jbossInitScript.standalone.sh)

This script is useful for maintenance (stop / start) of standalone instance.

#### Usage

```
/etc/init.d/gerenciadorJboss.sh {stop|start|status|restart|log}
                  .                           .
                  .                           .
                  .                           .
                  .                           ........................ Operation
                  .................................................... Script
```
