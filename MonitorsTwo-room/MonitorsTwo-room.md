# HackTheBox: MonitorsTwo

---

Проводим сканирование портов при помощи Nmap:

![ScreenShot](screenshots/1.png)

Найденный службы:
- 22 port - OpenSSH 8.2p1
- 80 port - nginx 1.18.0

Переходим на сайт:

![ScreenShot](screenshots/2.png)

Перед нами система `Cacti версии 1.2.22`. На всякий случай для начала попробуем найти какие-либо директории:

![ScreenShot](screenshots/3.png)

Директорий достаточно, но мы так и не вошли. Поэтому посмотрим в сторону эксплойтов:

![ScreenShot](screenshots/4.png)

И да, есть `CVE-2022-46169` для `Cacti версии 1.2.22`

Находим эксплойт и сразу смотрим параметры:

![ScreenShot](screenshots/5.png)

![ScreenShot](screenshots/6.png)

Видим, что скрипт принимает параметры `LHOST` и `LPORT` - здесь надо указать IP и Port, где мы будем ожидать подключения. Открываем порт на прослушивание и запускаем эксплойт:

![ScreenShot](screenshots/7.png)

![ScreenShot](screenshots/8.png)

И вот мы получаем шелл:

![ScreenShot](screenshots/9.png)

Осматриваемся и понимаем, что мы внутри `docker-контейнера`

![ScreenShot](screenshots/10.png)

![ScreenShot](screenshots/11.png)

![ScreenShot](screenshots/12.png)

По-хорошему, нам бы воспользоваться `linPEAS`, поэтому для начала проверим наличие `curl`:

![ScreenShot](screenshots/13.png)

Отлично, `curl` установлен, поэтому скачиваем себе `linPEAS`

![ScreenShot](screenshots/14.png)

![ScreenShot](screenshots/15.png)

Инструмент у нас на месте, теперь перед нами стоит задача переброса его в контейнер. Дляэтого воспользуемся HTTP-сервером на основе `python`

![ScreenShot](screenshots/16.png)

Скачивание:

![ScreenShot](screenshots/17.png)

Активация:

![ScreenShot](screenshots/18.png)

Инструмент нашел некоторые пробелы в SUID. Интересно, что нам доступен `capsh`. Смотрим, как нам повысить привилегии относительно данной команды на `GTFORBins`:

![ScreenShot](screenshots/19.png)

Применяем команду:

![ScreenShot](screenshots/20.png)

Еще из интересного - `linPEAS` обнаружил переменные PHP, содержащие в себе логин и пароль в открытом виде для БД:

![ScreenShot](screenshots/21.png)

Из БД обнаруживаем MySQL:

![ScreenShot](screenshots/22.png)

Изучение процесса конфигурирования `Cacti` по мануалу:

![ScreenShot](screenshots/23.png)

Просматриваем таблицы:

![ScreenShot](screenshots/24.png)

Находим хэши пользователей:

![ScreenShot](screenshots/25.png)

![ScreenShot](screenshots/26.png)

Сохраняем в файл для дальнейшего брута:

![ScreenShot](screenshots/27.png)

Находим пароль:

![ScreenShot](screenshots/28.png)

Подключаемся уже напрямую к хосту:

![ScreenShot](screenshots/29.png)

Первый флаг:

![ScreenShot](screenshots/30.png)

Проверяем `sudo` права пользователя `marcus`:

![ScreenShot](screenshots/31.png)

Вот это да, мы без sudo, тогдп просматриваем SUID-файлы:

![ScreenShot](screenshots/32.png)

И тут тоже особо ничего интересного нет. Продолжаем искать и находим кое-что очень интересное:

![ScreenShot](screenshots/33.png)

Письмо с указанием на конкретные CVE. Последняя из трех уязвимостей:

```sh
CVE-2021-41091 - это ошибка в Moby (движке Docker), которая позволяет непривилегированным пользователям Linux просматривать и выполнять программы в каталоге данных (обычно расположенном по адресу /var/lib/docker) из-за неправильно ограниченных разрешений. Эта уязвимость присутствует, когда контейнеры содержат исполняемые программы с расширенными разрешениями, такими как setuid. Непривилегированные пользователи Linux могут затем обнаруживать и запускать эти программы, а также изменять файлы, если UID пользователя на хосте совпадает с владельцем файла или группой внутри контейнера.
```

![ScreenShot](screenshots/34.png)

Делаем себе права на `/bin/bash`

![ScreenShot](screenshots/35.png)

Воспользуемся командой `findmnt`. Эта команда позволяет задействовать одноименную утилиту, предназначенную для поиска точек монтирования файловых систем. Тут мы и находим контейнеры:

![ScreenShot](screenshots/36.png)

Активируем bash относительно одного из контейнеров (где ранее мы выдавали себе права на bash):

![ScreenShot](screenshots/37.png)

И вот мы можем получить второй флаг:

![ScreenShot](screenshots/38.png)

![ScreenShot](screenshots/39.png)
