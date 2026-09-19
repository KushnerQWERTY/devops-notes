#!/bin/bash

echo "Имя скрипта: $0"
echo "Первый аргумент: $1"
echo "Второй аргумент: $2"
echo "Всего аргументов: $#"
echo "Все аргументы: $@"

Вывод команды: ./test-args.sh nginx mysql redis

Имя скрипта: ./test-args.sh
Первый аргумент: nginx
Второй аргумент: mysql
Всего аргументов: 3
Все аргументы: nginx mysql redis


Скрипт менеджер для работы со службами

#!/bin/bash

log() { - Функция логирования, а именно вывод даты
        echo "[$(date '+%H:%M:%S')] %1" 
}

check_exists() { - Проверка на существование
        if ! systemctl list-unit-files | grep -q "$1.service"; then
                log "Service 1$ is not found"
                exit 1
        fi
}

if [ $# -ne 2 ]; then - Проверка количество заданных аргументов
        echo "Using: $0 <name> <action>"
        echo "Actions: start, stop, restart, status"
        exit 1
fi

SERVICE=$1 - Переменные
ACTION=$2

check_exists $SERVICE - Проверка на существование службы
log "DO $ACTION for service $SERVICE"

case $ACTION in
        start)
                sudo systemctl start $SERVICE
                echo "service $SERVICE started"
        ;;
        stop)
                sudo systemctl stop $SERVICE
                echo "service $SERVICE stopped"
        ;;
        restart)
                sudo systemctl restart $SERVICE
                echo "service $SERVICE restarted"
        ;;
        status)
                sudo systemctl status $SERVICE --no-pager
        ;;
        *) - остальное
                echo "Unknown action: $ACTION"
                echo "Allowed actions: start, stop, restart, status"
                exit 1
        ;;

esac

