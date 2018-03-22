
# Jboss Init Scripts

Scripts para manutenção de instancias jboss **"standalone"** e **"clusters (ajp)""**.

### [jbossInitScript (cluster)](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/jbossInitScript.cluster.sh)

Este script é útil para manutenção (stop/start) de instâncias jboss a partir de um unico node.

###### *Obs.: Necessário relação de confiança (baseada em troca de chaves rsa/dsa) entre os nodes.*

#### Operação

Este script suporta operações `stop`, `start`, `status`, `restart` em instancias de appserver locais e remotas (contidas no arquivo "/etc/hosts").

- __[User management](https://nodeca.github.io/pica/demo/) - Gerenciamento de usuarios__
- __[SSH access control](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) - Controle de acesso SSH__
- __[sudoers](https://www.sudo.ws/) - Proteção SUDO__


#### Requerimentos

Para execução do comando nos nodes remotos é necessario garantir relação de confiança (troca de chaves RSA/DSA) entre os hosts. Seguem procedimentos:

* Geração de par de chaves DSA/RSA

```
$ ssh-keygen -t rsa
```
![SSH key generation ](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/ssh-keygen.png "ssh-keygen")


* Autorizacão por intercambio da chave pública (origem -> destino)

```
$ ssh-copy-id remoteuser@remotehost
```
![SSH copy id key.pub ](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/ssh-copy-id.png "ssh-copy-id")

#### Uso

```
$ sudo /etc/init.d/jboss            {stop|start|status|restart} {all|00}
    .                .                           .                 .
    .                .                           .                 .
    .                .                           .                 ...... Node (Ex. 01 ou all)
    .                .                           ........................ Operacao
    .                .................................................... Script
    ..................................................................... Executa como superuser

```

### [jbossInitScript (standalone)](https://github.com/concrete-aecio-barreto-junior/jbossInitScripts/blob/master/jbossInitScript.standalone.sh)

Script para manutenção de instancia jboss standalone.

#### Uso

```
/etc/init.d/gerenciadorJboss.sh {stop|start|status|restart|log}
 .                .                           .
 .                .                           .
 .                .                           .
 .                .                           ........................ Operacao
 .                .................................................... Script
 ..................................................................... Executa como superuser
```


## Notas

Considerando que a aplicação não tenha sido desenvolvida de maneira a garantir o devido tratamento do sinal `SIGTERM, 15`, onde por padrão a aplicação sairia do ar adequadamente, ambos scripts `standalone` e `cluster` suportam o argumento `stop` com parada p do appserver de maneira compreensiva.

+ Shutdown (via controller) - falhou?
  - kill `-15` 5x - (falhou)?
    - kill `-9`


1. **st:** Quando invocado o argumento `stop` incialmente o script tentará parar o appserver através do comando shutdown emitido p/ o controller e aguardará um timeout. Caso não haja eficácia no comando shutdown um segundo artificio (aseguir descrito) será lançado. Segue função comentada:


```bash
## Funcao p/ invocar shutdown pelo controller
_StopDefault(){
   # Imprime na output operacao de stop
   echo Stopping...
   # Invoca o shutdown ao controller
   /usr/bin/sudo -u jboss $JBOSS_HOME/bin/jboss-cli.sh --connect --controller=127.0.0.1:8888 command=:shutdown
}
```

2. **nd:** Será emitido o `SIGTERM/15` 5x e aguardará um timeout (tempo regular para encerramento de trheads/conexões). Caso o appserver não trate o sinal 15 segue terceiro e ultimo artificio. Segue código comentado:


```bash
## Funcao de controle de Stop
### 1st shutdown
### 2st kill -15 x5
### 3st kill -9! ~~morte ao jboss~~
_Stop(){
   # Instancia variavel de controle de RC
   local RC=0
   # Intervalo entre tentativas
   TimeWait=30
   # Invoca stop ao controller (default)
   _StopDefault
   # Aguarda...
   sleep $TimeWait
   # Verifica o processo
   local Processo=$( _Status > /dev/null 2>&1; echo $? )
   # Se o processo persiste...
   if [ $Processo -eq 0 ]; then
      # Primeira tentativa com kill -15
      _Kill -15 || local RC=$?
   fi
   # Verifica se o 'kill -15' obteve sucesso
   if [ $RC -ne 0 ]; then
      # Mais uma vez aguarda...
      sleep $TimeWait
      # Encerra o processo
     _Kill -9
     # Atribui sucesso ao RC uma vez que o '-9' eh eficaz
     local RC=0
   fi
   # Encerra funcao e retorna o RC
   return $RC
}
```

3. **St.:** O Jboss será encerrado com o `kill -9`. Segue código comentado:
> __IMPORTANTE__ Considerar duração do timeout conforme necessidade e riscos de encerrar o Jboss com o `kill -9`


```bash
## Funcao p/ submeter o kill
_Kill(){
   # Signal a ser lançado
   local Signal=$1
   # Var p/ controle de return code
   local RC=1
   # Tentativas
   local Retry=5
   # Contador p/ loop
   local Count=1
   # Checa se o processo existe
   local Processo=$( pgrep java > /dev/null; echo $? )
   # Laco para enquanto o processo existir
   while [ $Processo -eq 0 -a $Count -lt $Retry ]
   do
      # Obtem PID da JVM
      local PID="$( ps aux | grep -v grep | grep java | awk '{ print $2 }'|tr -s '\n' ' ' )"
      # Submete o kill conforme sinal fornecido como argumento
      kill $Signal "$PID"
      # incrementa o Contador
      let Count++
      # timeout
      sleep 10
      # Checa existencia do processo p/ continuidade do laço
      local Processo=$( pgrep java > /dev/null; echo $? )
   done
   # Ultima checagem do processo p/ escalar o RC
   local Processo=$( pgrep java > /dev/null; echo $? )
   # Trata o RC
   [[ $Processo -eq 0 ]] || local RC=0
   # Retorna o RC
   return $RC
}
```

## Links úteis:

| Descrição | Link |
| ------ | ------ |
| GNU Bash | [https://www.gnu.org/software/bash/](https://www.gnu.org/software/bash/) |
| Jboss | [http://www.jboss.org/](http://www.jboss.org/) |
| Markdown Tutorial | [https://www.markdowntutorial.com/](https://www.markdowntutorial.com/) |
| Documentação completa | [https://daringfireball.net/projects/markdown/](https://daringfireball.net/projects/markdown/) |
| Kill command | [http://linuxcommand.org/lc3_man_pages/kill1.html](http://linuxcommand.org/lc3_man_pages/kill1.html) |
