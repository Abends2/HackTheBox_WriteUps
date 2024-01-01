# HackTheBox: Codify

---

Проводим сканирование портов при помощи Nmap:

![ScreenShot](screenshots/1.png)

Найденный службы:
- 22 port - OpenSSH 8.9p1
- 80 port - Apache httpd 2.4.52
- 3000 port - Node.js Express framework

Далее в файл `/etc/hosts` добавляем IP и доменное имя жертвы:

![ScreenShot](screenshots/2.png)

Переходим на сайт:

![ScreenShot](screenshots/3.png)

Самое интересное, что данный ресурс предназнаен для исполнения кода (node.js). Это позволяет нам реализовать RCE

![ScreenShot](screenshots/4.png)

![ScreenShot](screenshots/5.png)

Сделаем `reverse shell`. Сначала включаем `nc` для прослушивания порта 4242:

![ScreenShot](screenshots/6.png)

Пытаемся через `bash` создать соединение, но получаем error:

![ScreenShot](screenshots/7.png)

Снова проверяем работоспособность, а также определяем наличие `python`:

![ScreenShot](screenshots/8.png)

![ScreenShot](screenshots/9.png)

Тут был немного ступор, поэтому пришлось смотреть решения других. И одно из них мне крайне понравилось.

Создаем одну пару ключей SSH:

![ScreenShot](screenshots/10.png)

Далее загружаем публичный ключ на хост:

```sh
mkdir /home/svc/.ssh/ && cd .ssh && echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB<LONG_LONG_KEY>= abends@parrot" > authorized_keys
```

![ScreenShot](screenshots/11.png)

Проверяем наличие:

![ScreenShot](screenshots/12.png)

![ScreenShot](screenshots/13.png)

Теперь просто подключаемся по SSH:

![ScreenShot](screenshots/14.png)

Находим `tickets.db`, в котором находится хэш пароля пользователя `joshua`

![ScreenShot](screenshots/15.png)

![ScreenShot](screenshots/16.png)

Брутим пароль:

![ScreenShot](screenshots/17.png)

Переключаемся на `joshua`

![ScreenShot](screenshots/18.png)

Посмотрим возможности относительно sudo:

![ScreenShot](screenshots/19.png)

Нам доступен к запуску файл `/opt/scripts/mysql-backup.sh`

![ScreenShot](screenshots/20.png)

Окей, но вернемся назад и вспомним, что мы как минимум не забрали `user.txt`:

![ScreenShot](screenshots/21.png)

Помимо этого здесь же лежать такие файлы как `brute.py`, `script.py` и `wordlist.txt`

![ScreenShot](screenshots/22.png)

![ScreenShot](screenshots/23.png)

Немного изменим скрипт:

![ScreenShot](screenshots/24.png)

Запускаем скрипт и видим, как посимвольно появляется пароль:

![ScreenShot](screenshots/25.png)

Переключаемся на `root` и забираем флаг:

![ScreenShot](screenshots/26.png)

![ScreenShot](screenshots/27.png)

---
