
# jboss Init Scripts
---
## Description

Scripts para manutenção de instancias jboss standalone e clusters (ajp).

### jbossInitScript (cluster)

Este script é útil para manutenção (stop/start) de instâncias jboss a partir de um unico node.
Necessário relação de confiança (baseada em troca de chaves rsa/dsa) entre os nodes.

#### Operation

Este script suporta operações `stop`, `start`, `status`, `restart` em instancias de appserver locais e remotas (contidas no arquivo "/etc/hosts").

- __[User management](https://nodeca.github.io/pica/demo/) - Gerenciamento de usuarios__
- __[SSH access control](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) - Controle de acesso SSH__
- __[sudoers](https://www.sudo.ws/) - Proteção SUDO__


#### Requerimentos

Para execução do comando nos nodes remotos é necessario garantir relação de confiança (troca de chaves RSA/DSA) entre os hosts.

* Geração de par de chaves DSA/RSA

```
$ ssh-keygen -t rsa
```
[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

* Autorizacão por intercambio da chave pública (origem -> destino)

```
$ ssh-copy-id remoteuser@remotehost
```
[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

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


## Notes


Considerando que a aplicação não tenha sido desenvolvida a garantir devido tratamento do `SIGTERM`[ˆfirst] 15 (software termination signal) ambos scripts `standalone` e `cluster` suportam o argumento `stop` com parada p do appserver de maneira compreensiva.

+ Shutdown (via controller) - falhou?
  - kill[ˆsecond] `-15` 5x - (falhou)?
    - kill[ˆsecond] `-9` 8-)

*[controller]: Hyper Text Markup Language


1. St: Quando invocado o argumento `stop` incialmente o script tentará parar o appserver através do comando shutdown emitido p/ o controller e aguardará um timeout. Caso não haja eficácia no comando shutdown um segundo artificio (aseguir descrito) será lançado.

2. St: Será emitido o `SIGTERM/15` 5x e aguardará um timeout (tempo regular para encerramento de trheads/conexões). Caso o appserver não trate o sinal 15 segue terceiro e ultimo artificio

3. St. O Jboss será encerrado com o KILL -9
> __IMPORTANTE__ ==CONSIDERAR TIMEOUTS NECESSARIO e POSSIBILIDDE DE KILL -9==


```bash
## Funcao p/ invocar shutdown pelo controller
_StopDefault(){
   echo Stopping..
   /usr/bin/sudo -u jboss $JBOSS_HOME/bin/jboss-cli.sh --connect --controller=127.0.0.1:8888 command=:shutdown
}

## Funcao de controle de Stop
### 1st shutdown
### 2st kill -15 x5
### 3st kill -9!
_Stop(){
   local RC=0
   TimeWait=30
   _StopDefault
   sleep $TimeWait
   local Processo=$( _Status > /dev/null 2>&1; echo $? )
   if [ $Processo -eq 0 ]; then
      _Kill -15 || local RC=$?
   fi
   if [ $RC -ne 0 ]; then
      sleep $TimeWait
     _Kill -9
     local RC=0
   fi
   return $RC
}

## Funcao p/ submeter o kill
_Kill(){
   local Signal=$1
   local RC=1
   local Retry=5
   local Count=1
   local Processo=$( pgrep java > /dev/null; echo $? )
   while [ $Processo -eq 0 -a $Count -lt $Retry ]
   do
      local PID="$( ps aux | grep -v grep | grep java | awk '{ print $2 }'|tr -s '\n' ' ' )"
      kill $Signal "$PID"
      let Count++
      sleep 10
      local Processo=$( pgrep java > /dev/null; echo $? )
   done
   local Processo=$( pgrep java > /dev/null; echo $? )
   [[ $Processo -eq 0 ]] || local RC=0
   return $RC
}
```

## Links úteis:

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| Github | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |
