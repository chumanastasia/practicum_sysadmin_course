#

# Спринт 3. Настройка файлового и прокси-сервера
## Задание №1. Настройка файлового сервера и монтирование директории. 
1. Создать две виртуальные машины через CloudShell - https://cloudshell.sysadmin.education-services.ru/
2. Настроим файловый сервер на первой VM 10.10.0.229

- Подключаемся к серверу по SSH-протоколу, используя логин s5390848 и IP-адрес 10.10.0.229
> ssh s5390848@10.10.0.229

-  Обновляем список доступных пакетов и их версий в системе Ubuntu/Debian
> sudo apt-get update

-  Устанавливаем пакет Samba 
> sudo apt install samba -y

- Создаем нового пользователя smbuser с домашним каталогом 
> sudo useradd -m smbuser

- Добавляем пользователя smbuser в базу паролей Samba и устанавливаем для него пароль.
> sudo smbpasswd -a smbuser

- Предоставляем полные права доступа (777) всем пользователям к домашнему каталогу /home/smbuser/
> sudo chmod 777 /home/smbuser/

- Создаем копию оригинального файла конфигурации Samba
> sudo cp /etc/samba/smb.conf /etc/samba/smb_backup.conf


-  Открываем файл конфигурации Samba для редактирования и добавляем/изменяем следующие настройки:
> sudo nano /etc/samba/smb.conf

```bash
[global]
   security = user
   # остальные настройки...

[home_smbuser]
   path = /home/smbuser
   guest ok = yes
   read only = yes
   writable = no
   public = yes
   valid users = smbuser
```

- Выполним перезапуск службы Samba
> sudo systemctl restart smbd.service

3. Настроим автомонтирование на второй VM 10.11.1.208

-  Подключаемся к клиентскому серверу по SSH-протоколу
> ssh s5390848@10.11.1.208

- Обновляем список доступных пакетов в системе
> sudo apt-get update

-  Устанавливаем пакет cifs-utils
> sudo apt-get install cifs-utils -y

- Создаем директорию, где будут монтироваться ресурсы
> sudo mkdir /mnt/samba/

- Выполняем тестовое ручное монтирование 
> sudo mount.cifs //10.10.0.229/home_smbuser /mnt/samba

- Отмонтируем 
> sudo umount -a -t cifs -l

- Устанавливаем autofs
> sudo apt-get install autofs

- Создаем папку для хранения кастомных конфигураций
> sudo mkdir /etc/autofs/

- Редактируем основной конфигурационный файл /etc/auto.master (после установки autofs появится атвоматически)
> sudo nano /etc/auto.master

Конфиг будет выглядеть так (добавляется только последняя строка, первые две оставляем без изменений)

```bash
+dir:/etc/auto.master.d
+auto.master
/mnt/samba /etc/autofs/auto.samba --timeout 60 --browse
```



## Задание №2. Настройка прокси-сервера.


1. Создать третью VM через CloudShell - https://cloudshell.sysadmin.education-services.ru/
2. 