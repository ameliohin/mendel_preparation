# mendel_preparation

## Описание подготовки дистрибутива для введения в эксплуатацию на плате Google coral TPU

Для записи ПО на плату необходимо выполнить подготовительные работы, которые включают в себя:
- установку ОС
- редактирование конфигурации дисплея и тачскрина weston
- редактирование конфигурации SSH (после чего можно производить работы удаленно)
- установку и настройку сервиса ssh туннеля для доступа к плате извне
- установку необходимых пакетов
- заливку свежей версии ПО

## Установка ОС Mendel Linux

Дистрибутив поставляется в виде образа, который следует записать на SD-card, подробнее процесс установки описан тут:

https://coral.ai/docs/dev-board/get-started#flash-the-board

Кратко: 
- устанавливаем пины 3,4 (дальние от вентилятора) в положение ON
![image](https://user-images.githubusercontent.com/79811164/168913322-d1c59bb0-f0c7-47f5-9f94-d3793eca2274.png)
- вставляем флешку с записанным образом в слот до щелчка
- включаем плату и наблюдаем на дисплее процесс установки (достаточно долгий)
- после того как плата выключится переставляем пины обратно
![image](https://user-images.githubusercontent.com/79811164/168913566-2791b2ec-a91d-443e-bbba-88794d5b79e0.png)
- включаем плату и даем завершиться процессу послеустановочных процедур


## Display and Touch

Поскольку мы используем дисплей разрешение которого меньше того что на плате по-умолчанию, буквы будут маленькие и плохо разборчивы, необходимо откорректировать настройки дисплея. Для этого надо запустить терминал нажав на единственную иконку что в наличии на панели.

Редактируем файл настроек weston:
nano /etc/xdg/weston/weston.ini

В последней секции [output] меняем значение mode на 1280x720, т.е. должна получиться строка:
mode=1280x720

Выходим из редактора: Ctrl+X

## SSH

Таким же образом надо отредактировать настройки ssh: 

nano /etc/ssh/sshd_config

Ищем строку: EnablePasswordAuthentication no

И меняем её на: EnablePasswordAuthentication yes

Выходим из редактора: Ctrl+X

# SSH Tunnel Service

https://askubuntu.com/questions/1316798/establishing-persistent-autossh-reverse-ssh-tunnel

Create the tunnel private/public key using ssh-keygen on the remote machine. You will be prompted for a passphrase. You can press Enter to ignore the passphrase questions, but this is not recommended. It would mean that anyone on the remote computer could make an SSH connection to your local computer without being challenged for a password (see the "Using SSH With Keys" section).

Install the public key in your remote user@remote.hosts .ssh/authorized_keys file

Test it by manually trying the ssh command and make sure the reverse tunnel is working.

vi /etc/systemd/system/tunnel.service

```
[Unit]
Description=Maintain Tunnel
After=network.target

[Service]
User=localuser
ExecStart=/usr/bin/ssh -i ~localuser/.ssh/tunnel -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -gnNT -R 22222:localhost:22 remoteuser@remotehost vmstat 5
RestartSec=15
Restart=always
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

Then run:

```
sudo systemctl daemon-reload
sudo systemctl enable tunnel
sudo systemctl start tunnel
```

