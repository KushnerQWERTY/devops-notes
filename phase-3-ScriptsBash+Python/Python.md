запуск или создание файла python
mkdir test.py - создание скрипта 
python3 test.py - запуск

#!/usr/bin/env python3

import subprocess - Позволяет запускать команды Linux в  Python
from datetime import datetime

services = ["nginx", "mysql", "my-monitor", "ssh"] - создание массива 

def log(message):
        timestamp = datetime.now().strftime('%H:%M:%S')
        print(f"[{timestamp}] {message}")

def check_service(name):
        result = subprocess.run( - Запуск Bash
                ['systemctl', 'is-active', name], Ввод команд, name - переменная в данном случае
                capture_output=True, - Команда для захвата вывода из команды в stdout - без него вывод пойдет в терминал
                text=True - Вывод в формате строки, а не в байтах
        )
        return result.stdout.strip() - strip убирает пробелы и переносы строк

log("=== Start monitoring service ===")

for service in services:
        status = check_service(service)
        if status == "active":
                print(f"{service}: correct")
        else:
                print(f"{service}: incorrect (status is {status})")
log("=== Monitoring is done ===")
