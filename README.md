# 🛡️ SERVER_PROTECTION

##Первичная настройка

1. sudo adduser USERNAME - Создание нового пользователя
2. sudo usermod -aG USERNAME - Выдача прав суперпользователя новому созданному пользователю
3. su - USERNAME - Переход на пользователя
4. sudo apt update - Обновление версий пакетов
5. sudo apt upgrade - Установка новых версий обновленных пакетов
6. sudo apt install -y fish micro - Установка пакета fish:
        fish - альтернативная командная оболочка (Fish Shell);
7. chsh -s /usr/bin/fish - Замена класической оболочки на fish
8. exit - Отключение от пользователя
9. exit - Выход из системы
10. ssh-keygen -t rsa -b 4096 -C "USERNAME@example.com" - Создание пары приватного и публичного ключей
11. ssh USERNAME@SERVER_IP - Вход на сервер под новым пользователем
12. mkdir -p ~/.ssh - Создание папки .ssh
13. chmod 700 ~/.ssh - Выдача прав для папки
14. nano ~/.ssh/authorized_keys - Открытие файла authorized_keys для записи публичного ключа из 10 пункта
15. chmod 600 ~/.ssh/authorized_keys - Установка прав
16. ssh USERNAME@SERVER_IP -p 22 -i ПУТЬ ДО ПРИВАТНОГО КЛЮЧА - На локальной машине, нужно создать файл с расширением .bat для быстрого входа на сервер
17. ss -tuln | grep ЛЮБОЙ ПОРТ В ДИАПАЗОНЕ 9999-65535 - Проверка доступности порта
18. sudo nano /etc/ssh/sshd_config - Редактирование файла:
        1. #Port 22 - нужно раскомментировать и заменить на выбранный;
        2. PasswordAuthentication no - Отключение входа по паролю;
        3. PermitRootLogin no - Отключение входа для root по паролю
20. sudo systemctl restart sshd - Перезапуск службы демона
21. chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys - Выдача прав на ключи
22. sudo grep -ir passwordauthentication /etc/ssh/ - проверка других файлов которые используют пароли для ssh (Там где "PasswordAuthentication yes" нужно закомментировать)
23. sudo systemctl restart sshd - Перезапуск службы демона после чистки

Настройка fail2ban
1. sudo apt install fail2ban - Установка сервиса fail2ban
2. sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local - Копирование конфига чтобы не работать в основном файле
3. sudo nano /etc/fail2ban/jail.local - Открытие файла jail.local:
        1. Находим строку [sshd] и комментируем ее и все что идет до следующего параметра в квадратных скобках;
        2. Вписываем свои настройки:
                 [sshd]
                 enabled = true #Служба включена
                 port = ПОРТ ИЗ ПУНКТА 17
                 filter = sshd
                 logpath = /var/log/auth.log #Путь к логам
                 maxretry = 3 #Максимальное кол-во запросов за отведенное время findtime
                 bantime = 10800 #Бан на 3 часа (-1 вечная блокировка)
                 findtime = 600
4. sudo nano /etc/fail2ban/filter.d/sshd.conf - Открытие текущего конфига:
        1. Находим строку failregex = ... и комментируем ее;
        2. Вписываем строку failregex = ^%(__prefix_line)sConnection closed by authenticating user .+ <HOST> port \d+ \[preauth\]$
5. sudo systemctl start fail2ban - Запускаем службу fail2ban
6. sudo systemctl enable fail2ban - Активируем службу fail2ban
7. sudo systemctl status fail2ban - Проверка статуса fail2ban если "Active: active (running)" все работает корректно
8. sudo fail2ban-client status sshd - Проверка забаненных пользователей
9. sudo fail2ban-client set sshd unbanip <IP ПОЛЬЗОВАТЕЛЯ> - Убираем пользователя из бана

Настройка nftables
nftables — это современная подсистема фильтрации сетевого трафика в Linux, которая заменяет старые инструменты: iptables, ip6tables, arptables и ebtables.
1. sudo apt install nftables - Установка nftables
2. sudo systemctl stop iptables - Остановка старой службы iptables (Если эта служба есть)
3. sudo systemctl disable iptables - Отключение службы iptables (Если эта служба есть)
4. sudo nano /etc/nftables.conf - Открытие файла конфигурации nftables и удаляем все содержимое и вставляем следующее:
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
	set ssh1 {
		type ipv4_addr
		size 10
		flags dynamic,timeout
		timeout 10s
		comment "Первый стук"
	}

	set ssh2 {
		type ipv4_addr
		size 10
		flags dynamic,timeout
		timeout 10s
		comment "Второй стук"
	}

	set ssh3 {
		type ipv4_addr
		size 10
		flags dynamic,timeout
		timeout 10s
		comment "Третий стук"
	}

	set ssh.accept {
		type ipv4_addr
		size 10
		flags dynamic,timeout
		timeout 10m
		comment "Это уже разрешённые адреса"
	}

	chain input {
		type filter hook input priority filter; policy drop;
		ip saddr @ssh1 tcp flags syn tcp dport { 7000 } delete @ssh1 { ip saddr } counter drop comment "Удаление после первого"
		tcp flags syn tcp dport 7000 add @ssh1 { ip saddr } counter drop comment "Первый стук"
		ip saddr @ssh2 tcp flags syn tcp dport { 8000 } delete @ssh2 { ip saddr } log prefix "ban-ssh " counter drop comment "Удаление после второго"
		ip saddr @ssh1 tcp flags syn tcp dport 8000 add @ssh2 { ip saddr } counter drop comment "Второй стук"
		ip saddr @ssh3 tcp flags syn tcp dport { 9000 } delete @ssh3 { ip saddr } log prefix "ban-ssh " counter drop comment "Удаление после третьго"
		ip saddr @ssh2 tcp flags syn tcp dport 9000 add @ssh3 { ip saddr } counter drop comment "Третий стук"
		ip saddr @ssh1 ip saddr @ssh2 ip saddr @ssh3 tcp flags syn tcp dport != { 22, 80, 443, 8080, 62719, 10000 } delete @ssh3 { ip saddr } log prefix "ban-ssh " counter drop comment "Удаление после четвёртого"
		ip saddr @ssh1 ip saddr @ssh2 ip saddr @ssh3 tcp flags syn tcp dport 10000 add @ssh.accept { ip saddr } counter drop comment "Четвёртый стук"
		ct state vmap { invalid : drop, established : accept, related : accept, untracked : accept } comment "Разрешим established,related и удалим invalid, входящие"
		ip saddr @ssh.accept counter accept comment "Это уже разрешённые адреса все порты"
		meta l4proto { tcp, udp } th dport { 80, 443, 8080 } counter accept comment "Порты которые всегда открыты"
	}
}
В строке { 22, 80, 443, 8080, 8182, 10000 } добавляем еще и наш порт
5. sudo systemctl start nftables - Запускаем службу nftables
6. sudo systemctl enable nftables - Включаем службу nftables
7. sudo systemctl status nftables - Проверка статуса nftables если "Active: active (running)" все работает корректно
