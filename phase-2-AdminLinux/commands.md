systemd - Система инициализации и управления службами в современном Linux.
/usr/local/bin - стандартная папка для пользовательских скриптов и программ
sudo nano /usr/local/bin my-script.sh - Пример создания скрипта
Содержание скрипта:
#!/bin/bash

while true
do
  DATE_TIME=$(date '+%Y-%m-%d %H:%M:%S')
  echo "#DATE_TIME - System is correctly" >> /home/user/my_script.log
  sleep 10
done

Разбор по строкам:
- #!/bin/bash - SheBang - говорит системе, что этот скрипт нужно запускать через /bin/bash - !Без этой строки systemd не знает, какой интерпретатор использовать.
- while true - бесконечный цикл
- do - начало блока цикла
- DATE_TIME=$(date '+%Y-%m-%d %H:%M:%S') - Сохраняем текущую дату/время в переменную
- echo "$DATE_TIME - System is correctly" >> /home/user/my_script.log - Записываем строку в лог-файл
- sleep 10 - ожидание 10 секунд
- done - конец цикла

Создание unit-файла (конфигурация службы)

sudo nano /etc/systemd/system/my_script.service
/etc/systemd/system/ - папка, где systemd ищет пользовательские службы
.service - расширение файла службы

Содержание файла:
[Unit]
Description= My simple monitoring system's
After=network.target

[Service]
ExecStart=/usr/local/bin/my_script.sh
Restart=always
RestartSec=3
User=root

[Install]
WantedBy=multi-user.target

Разбор секций:
[Unit] - общая информация
Description - Человеко-понятное описание (видно в systemctl status)
After=network.target - Запускать только после того, как поднимется сеть (Если бы наш скрипт отправлял данные по сети, он бы упал, если сеть еще не готова)
[Service] - Как запускать службу
ExecStart - Команда для запуска (полный путь обязателен)
Restart=always - Перезапускать всегда, даже если скрипт упал с ошибкой (no - не перезапускать; on-succces - если код 0(успех); on-failure - ошибка)
RestartSec=3 - Ждать 3 секунды перед перезапуском
User=root - От какого пользователя запускать
[Install] - Автозагрузка
WantedBy=multi-user.target - Добавлять автозагрузку для многопользовательского режима

Активация и запуска:

sudo systemctl daemon-reload - сообщает systemd, что появился новый файл службы
Почему это нужно? - Systemd кэширует список служб при загрузке. Если появился новый файл, systemd о нем не узнает, пока не сработает команда.
sudo systemctl start my_script - запускает службу прямо сейчас
Что происходит?
- Systemd читает файл /etc/systemd/system/my_script.service
- Проверяет зависимость (After=network.target)
- Запускает команду ExecStart=/usr/local/bin/my_script.sh
- Создает процесс и назначает ему PID
- Начинает следить за процессом

sudo systemctl enable my_script - добавляет службу в автозагрузку
sudo systemctl status my_script - Вывод:
  my_script.service - My simple monitoring system's
   Loaded: loaded (/etc/systemd/system/my_script.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2026-08-22 17:03:45 UTC; 25s ago
 Main PID: 1771 (my_script.sh)
    Tasks: 2 (limit: 2216)
   Memory: 572.0K
      CPU: 9ms
   CGroup: /system.slice/my_script.service
           ├─1771 /bin/bash /usr/local/bin/my_script.sh
           └─1816 sleep 10
Разбор каждой строки:
- my_script.service - My simple monitoring system's - Название службы (зеленая точка - активна)
- Loaded: loaded - файл загружен и валиден
- enabled - служба добавлена в автозагрузку
- Active: active (running) - Служба работает прямо сейчас - может быть inactive(dead) - остановлен; failed - ошибка; activating - запускается; deactivating - останавливается 
- since Sat 2026-08-22 17.03.45 UTC; 25s ago - Запущена 25 секунд назад
- Main PID: 1771 - Главный процесс имеет PID 1771
- Tasks: 2 - Служба использует 2 процесса (bash + sleep)
- Memory: 572.0К - Потребляет 572КБ памяти
- CPU: 9ms - использовала 9 миллисекунд процессорного времени
- CGroup: /system.slice/my_script.service - Группа контроля для изоляции ресурсов
- |-1771 /bin/bash /usr/local/bin/my_script.sh - Главный процесс - наш скрипт
- -1816 sleep 10 - Дочерний процесс - команда sleep

sudo journal -u my_script --no-pager -n 10 - Показывает логи systemd для конкретной службы -- -u my_script - только для службы my_script (unit); --no-pager - не использовать less (вывести все сразу)
-n 10 - показать последние 10 строк

Полезные комбинации:
journalctl -u my-monitor -f - Следить за логами в реальном времени (как tail -f)
journalctl -u my-monitor --since today - Логи за сегодня
journalctl -u my-monitor --since "1 hour ago" - Логи за последний час

df -h - показывает общую информацию свободного места на дисках; -h - показывает информацию в КБ,МБ,ГБ, а не в байтах
Разбор вывода:
- Filesystem - имя диска или раздела
- Size - общий размер
- Used - сколько занято
- Avail - сколько доступно
- Usr% - сколько процентов использовано ( если здесь > 85-90% - тревога )
- Mounted on - точка монтирования (куда "прикреплен" диск, например / - корень системы.)

du - помогает найти виновника занятого места.
sudo du -sh /var/log* | sort -hr | head -n 10
Разбор:
- -s - Показывает общий размер папки, а не каждого файла внутри
- -h - читаемый формат
- /var/log/* - проверяем все папки внутри /var/log
- sort -hr - сортировка по убыванию размера
- head -n 10 - показываем только топ 10 самых тяжелых папок

logrotate - Раз в день (или неделю\месяц) проверяет конфиги в /etc/logrotate.d/; Если лог стал слишком большим или старым, он переименовывает его (например my-script.log > my-script.log.1
- Создает новый пустой my-script.log
- Сжимает старые логи архиватором gzip
- Удаляет самые старые архивы (старше месяца)

Вывод команды cat /etc/logrotate.d/rsyslog:
- weekly - ротация раз в неделю
- rotate 4 - хранить 4 старых архива
- compress - сжимает старые логи
- delaycompress - сжимает не сразу, а со следующей ротацией
- missingok - не ругаться, если файла нет
- notifyempty - не ротировать, если файл пустой

sudo nano /etc/logrotate.d/my-monitor

