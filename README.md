
# jboss Init Scripts

## Description

Scripts para manutenção de instancias jboss standalone e clusters (ajp).
### jbossInitScript (cluster)

Este script é útil para manutenção (stop/start) de instâncias jboss a partir de um unico node.
Necessário relação de confiança (baseada em troca de chaves rsa/dsa) entre os nodes.

#### Operation

Este script suporta operações "stop|start|status|restart" em nodes contidos no arquivo "/etc/hosts".

Para execução do comando nos nodes remotos é necessario garantir relação de confiança (troca de chaves RSA/DSA) entre os hosts.

#### Usage

```
$ sudo /etc/init.d/jboss            {stop|start|status|restart} {all|00}
    .                .                           .                 .
    .                .                           .                 .
    .                .                           .                 ...... Node (Ex. 01 ou all)
    .                .                           ........................ Operacao
    .                .................................................... Script
    ..................................................................... Executa como superuser

```

### jbossInitScript (standalone)

Script para manutenção de instancia jboss standalone.

#### Usage

```
/etc/init.d/gerenciadorJboss.sh {stop|start|status|restart|log}
 .                .                           .
 .                .                           .
 .                .                           .
 .                .                           ........................ Operacao
 .                .................................................... Script
 ..................................................................... Executa como superuser
```
