# INSTALAÇÃO DO CACTI

Para ser instalado em máquinas Linux, o Cacti depende de ferramentas como o PHP, o MySQL e o Apacha (ou Nginx).
Visando automatizar este processo, podemos fazer a seguinte forma:

```bash
php="php php-common php-gd php-gmp php-intl php-ldap php-mbstring php-xml
mysql="php-mysql mysql-client mysql-server"
snmp="php-snmp snmp snmpd"
apache="apache2 libapache2-mod-php"
rrdtool="rrdtool"
extras="wget tar unzip"
```

Essas são as dependências do Cacti, para instalá-las basta fazer:

```bash
sudo apt update && sudo apt install -y $php $mysql $snmp $apache $rrdtool $extras
```

Agora, para baixar o Cacti basta fazer:

```bash
cacti_version="1.2.30" # versão do cacti para ser instalada
cacti_url="https://files.cacti.net/cacti/linux/cacti-${cacti_version}.tar.gz
cacti_tar="cacti-${cacti_version}.tar.gz" # nome do arquivo baixado
cacti_dir="/var/www/html/cacti"  # destino onde o cacti será instalado

# baixa o cacti
wget "$cacti_url"
```

Instalar

```bash
mkdir -p "$cacti_dir" # criando a paste de destino

tar -xzvf "$cacti_tar" --strip-components=1 -C "$cacti_dir" # extraindo o arquivo para a paste de destino

chown -R www-data:www-data "$cacti_dir" # dando permissões para o Apache
```

Configurar MySQL

```bash
mysql_secure_install
```

```sql
CREATE DATABASE Cacti;

CREATE USER 'cactiuser'@'%' IDENTIFIED BY 'cactipassword';

GRANT ALL PRIVILEGES ON Cacti.* TO 'cactiuser'@'%';

FLUSH PRIVILEGES;
```

```bash
mysql -u cactiuser -p -h 127.0.0.1 Cacti < /var/www/html/cacti/cacti.sql
```

```php
$database_type     = 'mysql';
$database_default  = 'Cacti';
$database_hostname = '127.0.0.1';
$database_username = 'cactiuser';
$database_password = 'cactipassword';
$database_port     = '3306';      // padrão para mysql
$database_ssl      = false;
```

````bash
sudo systemctl restart apache2
```

http://<ip>/cacti
