# nextcloud
Установка облака NextCloud в docker контейнере + mariadb + redis

Для установки используем docker-compose.yml, пароли меняем на свои.

Доверенные домены используются Nextcloud для предотвращения отравления заголовка хоста. Вам нужно указать каждый домен, по которому можно получить доступ к Nextcloud.
Это означает, что если у вас есть Nextcloud, установленный на "192.168.0.29", а также хотите, чтобы он был доступен на "cloud.example.com” вам нужно будет изменить trusted_domains запись в вашем config.php файле.

Для редактирования открываем файл: `sudo nano /var/www/nextcloud/config/config.php` (поменять путь к файлу на свой)
```
  'trusted_domains' =>
  array (
    0 => '192.168.0.29',
  ),
```
Чтобы добавить новый домен, просто добавьте новые записи, добавив новый элемент в массив PHP:

 ```
 'trusted_domains' =>
  array (
    0 => '192.168.0.29',
    1 => 'cloud.example.com',
  ),
```
Nextcloud оптимизирован для использования только с одним доменом.

Тюнинг:
Оригинал статьи с базой данных postgres (https://antoshabrain.blogspot.com/2021/12/docker-nextcloud-postgres-redis-amd64.html).

Останавливаем контейнер.

в docker-compose.yml раскоментируем строку **#- ./nextcloud.ini:/usr/local/etc/php/conf.d/nextcloud.ini** (убираем знак '#')
В папке приложени, рядом с docker-compose.yml создадим фаил nextcloud.ini (nano nextcloud.ini)

Содержание подправьте под ресурсы вашего сервера

```
upload_max_filesize=16G
post_max_size=16G
memory_limit=2G
max_input_time 7200
max_execution_time 7200
```

Чиним cron:

Nextcloud использует специальный фаил cron.php для того чтобы инициировать такие процессы как сканирование фаилов, музыки, очистку мусора и другие. Желательно выполнять этот фаил каждые 5-10 минут.

Используя NextCloud в докере из коробки к сожалению эта фича не работает и приходится использовать режим AJAX, который выполняет фаил cron.php при загрузке каждой страницы. Это создаёт дополнительную нагрузку на сервер.

Мы делегируем выполнение этого функционала, штатному cron. Для этого сначала установим его

`sudo apt install cron`

Проверим что он запущен и работает

`sudo systemctl status cron`

запускать наше задание от root поэтому пишем

`sudo crontab -e`

В самую нижнюю строчку добавляем наше задание

***/5 * * * * sudo /usr/bin/docker exec -u www-data *nextcloud-app-1* php -f /var/www/html/cron.php** 
(где ***nextcloud-app-1*** - имя контейнера, поменяйте на свой, если отличается)
