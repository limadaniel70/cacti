# INSTALAÇÃO DO CACTI

Para instalar o Cacti em sistemas Linux, é necessário garantir a presença de algumas dependências, como PHP, MySQL e um servidor web, como Apache ou Nginx.

A seguir, mostramos uma forma automatizada de realizar essa instalação.

## 1. Definindo as dependencias

```bash
php="php php-common php-gd php-gmp php-intl php-ldap php-mbstring php-xml"
mysql="php-mysql mysql-client mysql-server"
snmp="php-snmp snmp snmpd"
apache="apache2 libapache2-mod-php"
rrdtool="rrdtool"
extras="wget tar unzip"
```

## 2. Instalando os pacotes

```bash
sudo apt update && sudo apt install -y $php $mysql $snmp $apache $rrdtool $extras
```

## 3. Baixando e extraindo o Cacti

```bash
cacti_version="1.2.30" # versão do cacti para ser instalada
cacti_url="https://files.cacti.net/cacti/linux/cacti-${cacti_version}.tar.gz"
cacti_tar="cacti-${cacti_version}.tar.gz" # nome do arquivo baixado
cacti_dir="/var/www/html/cacti"  # destino onde o cacti será instalado

# baixa o cacti
wget "$cacti_url"

mkdir -p "$cacti_dir" # criando a paste de destino

tar -xzvf "$cacti_tar" --strip-components=1 -C "$cacti_dir" # extraindo o arquivo para a paste de destino

chown -R www-data:www-data "$cacti_dir" # dando permissões para o Apache
```

## 4. Configurando o MySQL

Execute o assistente de segurança:

```bash
mysql_secure_install
```

Em seguida, crie o banco de dados e o usuário para o Cacti:

```sql
CREATE DATABASE Cacti;

CREATE USER 'cactiuser'@'localhost' IDENTIFIED BY 'cactipassword';

GRANT ALL PRIVILEGES ON Cacti.* TO 'cactiuser'@'localhost';

FLUSH PRIVILEGES;
```

Importe a estrutura do banco de dados:

```bash
mysql -u cactiuser -p Cacti < /var/www/html/cacti/cacti.sql
```

## 5. Configurando o arquivo de conexão do Cacti

Edite o arquivo include/config.php dentro do diretório do Cacti e adicione as seguintes configurações (ajuste conforme necessário):

```php
$database_type     = 'mysql';
$database_default  = 'Cacti';
$database_hostname = '127.0.0.1';
$database_username = 'cactiuser';
$database_password = 'cactipassword';
$database_port     = '3306';      // padrão para mysql
$database_ssl      = false;
```

## 6. Reiniciando serviços e acessando

```bash
sudo systemctl restart apache2
sudo systemctl enable apache2
sudo systemctl restart snmpd
```

Acesse o Cacti pelo navegador usando a seguinte url: `http://<ip_ou_hostname>/cacti`

## 7. Configurando o cron

Essa linha é uma entrada de cron que agenda a execução periódica do script de coleta (poller) do Cacti.

```bash
*/5 * * * * php /var/www/html/cacti/poller.php > /dev/null 2>&1

```

| Campo | Significado |
| ----- | ----------- |
|*/5 |A cada 5 minutos. O asterisco (*) indica “todos os valores” e o /5 é o passo (step) “de 5 em 5”.|
|Primeiro *| Horas: “todos” (ou seja, sempre que o minuto bater).|
Segundo * |Dias do mês: “todos”.|
Terceiro * |Meses: “todos”.|
Quarto * |Dias da semana: “todos”.|
php |Comando/interpretador usado para rodar o script PHP.|
/var/www/html/cacti/poller.php| Caminho completo para o script de polling do Cacti. Esse script coleta dados dos dispositivos configurados via SNMP e armazena no banco de dados.|
| > /dev/null| Redireciona a saída padrão (stdout) do comando para “null”, descartando quaisquer mensagens normais.|
|2>&1 |Redireciona a saída de erro (stderr, descritor 2) para o mesmo destino da saída padrão (descritor 1), ou seja, também para /dev/null.
