# SERVER_PROTECTION

Первичная настройка
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
                 port = 
