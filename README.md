# Inotify

Permite que um programa de monitoramento abra um único descritor de arquivo e procure em um ou mais arquivos ou diretórios por um conjunto de eventos específico, como abrir, fechar, mover/renomear, deletar, criar ou alterar atributos.  

## No Docker (Alpine Linux)
[Docker_Inotify](https://github.com/paulo-correia/Docker_Inotify)

## Instalação

Toda a instalação é feita em **root**

### Debian

Instale o pacote Inotify com o comando:

`apt-get install inotify-tools`

### CentOS

Instale o repositório EPEL com o comando:

` yum -y install epel-release`

Instale o pacote Inotify com o comando:

 `yum install -y inotify-tools
`

## Testes

Crie uma pasta a ser monitorada ex: /inotify com o comando:

`mkdir /inotify`

Chame o inotify com o comando:

 `inotifywait -e create,delete,modify,move -mrq /inotify & `

Crie um arquivo com o comando:

 `touch /inotify/a.txt`

O Inotify vai retornar:

`/inotify/ CREATE a.txt `

Mova o arquivo com o comando:

 `mv a.txt b.txt`

O Inotify vai retornar:

 ```
/inotify/ MOVED_FROM a.txt
/inotify/ MOVED_TO b.txt
```

Remova o arquivo com o comando:

 `rm -f b.txt`

O Inotify vai retornar:

 `/inotify/ DELETE b.txt`

## Script

Edite o arquivo de configuração:

 `/etc/inotifywait.conf`

Coloque o seguinte conteúdo:

 ```
# Especifique o arquivo de log
LOGFILE=/var/log/inotify.log

#  Especifique a pasta a ser monitorada
MONITOR=/inotify

# Especifique os eventos a serem monitorados (separados por vírgula)
# Leia o "man inotifywait" para saber os eventos
EVENT=create,delete,modify,move
```

Salve e saia

### Instalação do serviço

Edite o arquivo de inicialização:

`/etc/rc.d/init.d/inotifywait`

Coloque o seguinte conteúdo:

 ```
# create init script

#!/bin/bash

# inotifywait: Start/Stop inotifywait
#
# chkconfig: - 80 20
# description: inotifywait waits for changes to files using inotify.
#
# processname: inotifywait

. /etc/rc.d/init.d/functions
. /etc/sysconfig/network
. /etc/inotifywait.conf

LOCK=/var/lock/subsys/inotifywait

RETVAL=0
start() {
  echo -n $"Starting inotifywait: "
  /usr/bin/inotifywait \
  --format '%w%f %e %T' \
  --timefmt '%Y/%m/%d-%H:%M:%S' \
  --exclude '.*\.sw[pox].*' \
  -e $EVENT \
  -o $LOGFILE \
  -dmrq $MONITOR

  RETVAL=$?
  echo
  [ $RETVAL -eq 0 ] && touch $LOCK
  return $RETVAL
}
stop() {
  echo -n $"Stopping inotifywait: "
  killproc inotifywait
  RETVAL=$?
  echo
  [ $RETVAL -eq 0 ] && rm -f $LOCK
  return $RETVAL
}
case "$1" in
  start)
     start
     ;;
  stop)
     stop
     ;;
  status)
     status inotifywait
     ;;
  restart)
     stop
     start
     ;;
  *)
     echo $"Usage: $0 {start|stop|status|restart}"
     exit 1
esac
exit $?
```

De as permissões com o comando:

`chmod 755 /etc/rc.d/init.d/inotifywait`

Inicie o serviço com o comando:

 `/etc/rc.d/init.d/inotifywait start`

Adicione o serviço a lista de serviços com o comando:

 `chkconfig --add inotifywait`

Coloque o serviço para carregar no boot com o comando:

`chkconfig inotifywait on`

Tudo o que fizer na pasta /inotify estará no arquivo /var/log/inotify.log

### Exemplo de uso

Sem o serviço carregado.

verify="n"

while \[ "$verify" == n \] do EVENT=$(inotifywait --format '%e' /pasta) \[ $? != 0 \] && exit

echo "evento="$EVENT

1. se entrar na pasta ele atribui -> OPEN,ISDIR
2. se remover -> DELETE
3. se criar (touch abc) -> CREATE
4. se der um more abc -> OPEN
5. se modificar com vi -> MODIFY
6. sai salvando do vi -> CREATE

if \[ "$EVENT" = "CREATE" \]
