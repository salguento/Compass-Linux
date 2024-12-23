# Trilha DevSecOps | Nov2024 : Trabalho 1 - Linux

# Introdução

## Descrição da atividade

1. Parte prática:
	Criar um ambiente Linux no Windows:
	Utilizando o WSL do Windows, crie um subsistema do Ubuntu 20.04 ou superior.
2. Atividade no Linux:
	• Subir um servidor Nginx; estar online e rodando;
	• Criar um script que valide se o serviço está online e envie o resultado da validação para um diretório que você definir.
	• O script deve conter - Data HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline;
	• O script deve gerar 2 arquivos de saída: 1 para o serviço online e 1 para o serviço OFFLINE;
	• Preparar a execução automatizada do script a cada 5 minutos.
	• Fazer o versionamento da atividade;
	• Fazer a documentação explicando o processo de instalação do Linux.

Esse trabalho visa documentar o processo de execução de todo o trabalho.

# Tutorial 
## 1. Habilitar WSL

- Abrir o PowerShell e rodar o comando:

```powershell
wsl --install
```

- Reiniciar o computador. Depois de reiniciado a instalação será concluída e dará prompt para configuração de um novo nome de usuario UNIX e senha

- Após a configuração de usuario, atualize o sistema

```sh
sudo apt update && sudo apt upgrade
```

> https://learn.microsoft.com/pt-br/windows/wsl/install
> https://learn.microsoft.com/pt-br/windows/wsl/setup/environment#set-up-your-linux-username-and-password
## 2. Instalar NGINX

> https://ubuntu.com/tutorials/install-and-configure-nginx#1-overview

```sh
sudo apt update && sudo apt install nginx
```

- Após a instalação o Nginx já estava rodando. Confira se ele está rodando com o comando. O resultado esperado é `Active: active (running)`. 

```sh
systemctl status nginx
```

- Caso o serviço não esteja rodando, inicie ele:

```sh
sudo systemctl start nginx
```

- Do mesmo modo, caso precise encerrar o serviço:

```sh
sudo systemctl stop nginx
```

- Para habilitar a inicialização com o boot

```sh
sudo systemctl enable nginx
```

- Para desabilitar a inicialização com boot

```sh
sudo systemctl disable nginx
```

- Para acessar a página servida pelo nginx acesse http://localhost:80. 

## 3. Script

- Crie uma pasta `scripts` dentro da pasta do usuário:

```sh
mkdir scripts
```

- Crie o arquivo `nginx_upcheck.sh` com o editor de texto de sua preferência, aqui o Nano:

```sn
nano nginx_upcheck.sh
```

- Inicie o Script com um bash bang:

```sh
#! /bin/bash
```

- Crie uma variável para o endereço da pasta de logs:

```sh
DIR="/home/<SEU_USUARIO>/logs_nginx"
```

- Crie uma variável para a data e hora:

```sh
TIME=$(date '+%Y-%m-%dT%H-%M-%S')
```

- Crie uma variável para o estado do serviço:

```sh
STATE=$(systemctl is-active nginx)
```

- Crie uma pasta para arquivar os logs. O argumento `-p` cria a pasta apenas quando ela não existir:

```sh
mkdir -p "$DIR"
```

- Faça a comparação do resultado com um operador ternário e execute o código para cada caso. Adicionando a linha de log para seu respectivo arquivo com a marcação `>>`.

```sh
case "$STATE" in
	active) echo "$TIME: NGINX | ONLINE | O serviço está ativo." >> "$DIR/LOG_ONLINE.log";;
	*) echo "$TIME: NGINX | OFFLINE | O serviço está inativo." >> "$DIR/LOG_OFFLINE.log";;
esac
```

- Script finalizado:

```sh
#! /bin/bash
DIR="/home/linux/logs_nginx"
TIME=$(date '+%Y-%m-%dT%H-%M-%S')
STATE=$(systemctl is-active nginx)

mkdir -p "$DIR"

case "$STATE" in
	active) echo "$TIME: NGINX | ONLINE | O serviço está ativo." >> "$DIR/log_online.log";;
	*) echo "$TIME: NGINX | OFFLINE | O serviço está inativo." >> "$DIR/log_offline.log";;
esac
```

- Salve e saia do editor de texto.

## 4. Execução do script

- É necessário tonar o script executável, utilize `chmod` com o argumento `+x` para atribuir essa permissão:

```sh
chmod +x /scripts/nginx_uptime.sh
```

- Verifique as permissões do arquivo:

```sh
ls -l /scripts/nginx_uptime.sh
```

- Execute o script com o comando `bash`:

```sh
bash /scripts/nginx_uptime.sh
```

## 5. Automação

- Para executar o script a cada cinco minutos é necessário agendar essa ação com o `crontab`. Utilize o argumento `e`, de editar, para adicionar uma nova entrada.

```sh
crontab -e
```

- Na primeira execução ele perguntará qual editor você deseja usar.

- Adicione na última linha a notação de tempo em cron, seguida do endereço do arquivo executável. Os parâmetros `2>&1 | logger -t mycmd` possibilitam ver os logs sem instalação de pacotes adicionais. 

```sh
*/5 * * * * (/home/<SEU_USUARIO>/scripts/nginx_upcheck.sh) 2>&1 | logger -t mycmd
```

- Certifique que seu agendamento foi concluído, use o argumento `l` para listar todos os cronjobs agendados.
 
```sh
crontab -l
```

- Confira os logs do Cron para confirmar a execução do script ou consultar erros: 

```sh
grep 'mycmd' /var/log/syslog 
```

