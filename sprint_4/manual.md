# Спринт 4. Настройка мониторинга

## Подготовка
1. Установим нужный пакет с БД
```commandline
sudo apt update

sudo apt install mysql-server

sudo systemctl start mysql
```

2. Установим репозиторий Zabbix
```commandline
sudo wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

3. Устанавливаем необходимые пакеты

```commandline
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

4. Создаём базу для нашей системы мониторинга с помощью уже установленного пакета mysql
```commandline
sudo mysql -uroot -p
722121
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by '722121';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```
5. Извлекаем схему базы данных Zabbix из архива server.sql.gz, распаковываем её и передаём результат в mysql, которая импортирует схему с указанной кодировкой и учётными данными пользователя.

```commandline
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

6. Изменим значение переменной конфигурации, чтобы ограничить доступ к функциям создания

```commandline
sudo mysql -uroot -p
722121
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

7. Внесем изменения в файл конфигурации /etc/zabbix/zabbix_server.conf. Укажем пароль

```commandline
 sudo nano /etc/zabbix/zabbix_server.conf
 ...
 DBPassword=722121
```

8. В файле конфигурации веб-сервера /etc/zabbix/nginx.conf раскомментируем следующие строки:

```commandline
sudo nano /etc/zabbix/nginx.conf
...
# listen 8080;
# server_name 10.10.1.93; 
```
9. Перезапустим службы Zabbix сервера и агента:

```commandline
sudo systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
```

10. Добавим в автозапуск служб Zabbix сервер и агента:

```commandline
sudo systemctl enable zabbix-server zabbix-agent php8.1-fpm
```
11. Открываем файл настройки по пути /etc/nginx/sites-enabled/default — с помощью редактора nano или другого — и изменяем строку root /var/www/html на root /usr/share/zabbix. После этого перезапускаем Nginx:

```commandline
sudo nano /etc/nginx/sites-enabled/default
 ...
root /usr/share/zabbix;
```

Перезапускаем nginx
```commandline
sudo systemctl restart nginx
```
11. Прежде чем открыть Zabbix в браузере, важно добавить правила для файрвола UFW. Так мы разрешим доступ к портам, с которыми работает Zabbix и Nginx, — 10050/tcp, 10051/tcp и 80/tcp. Перезагружаем наши правила:

```commandline
sudo ufw allow 10050/tcp
sudo ufw allow 10051/tcp
sudo ufw allow 80/tcp
sudo ufw reload
```
Теперь zabbix можно открыть в браузере по адресу http://10.10.1.93:8080

12. Выполним русификацию 

```commandline
sudo sed -i  "s/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/g" /etc/locale.gen
sudo locale-gen
sudo systemctl restart zabbix-server nginx php8.1-fpm 
```

