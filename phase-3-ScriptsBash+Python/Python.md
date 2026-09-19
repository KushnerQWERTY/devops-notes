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


#!/usr/bin/env python3
import json
from datetime import datetime

LOG_FILE = '/home/python/app.log'
REPORT_FILE = '/home/python/report.json'

def parse_log():
        errors = []
        try:
                with open(LOG_FILE, 'r') as f: - контекстный менеджер with, открывается файл для действий, r - read, w - write, a - append. as f - присваивает файловый объект переменной и автоматически закрывает файл.
                        lines = f.readlines() - f.readlines() - читает все строки файла и возвращает список, каждая строка файла является новым элементом списка
                        for line in lines:
                                if "ERROR" in line or "FAILED" in line:
                                        errors.append(line.strip()) - добавление строк с подходящим условием, при этом удаляются пробелы и переносы строк.
                print (f"Found errors: {len(errors)}")
        except FileNotFoundError: - если файла не существует. 
                print (f"File {LOG_FILE} not found")
                return
        except Exception as e: - перехват любой ошибки
                print (f"Error while reading: {e}") - вывод ошибки
        report = { - создание словаря
                "generated_at": datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
                "log_file": LOG_FILE,
                "total_errors": len(errors),
                "errors": errors
        }
        try:
                with open(REPORT_FILE, 'a') as f: - Запись в файл, если файл не существует, то он его создает, если существует, то перезаписывает! 
                        json.dump(report, f, indent=4) - dump - запись в формате json, indent - пробелы для визуальной составляющий, чтобы запись выглядела читаемо
                print (f"report saved to {REPORT_FILE}")
        except Exception as e:
                print (f"Errors while saving report: {e}")

if __name__ == "__main__":
        parse_log()

Магическая конструкция Python — проверяет, запущен ли скрипт напрямую
__name__ — специальная переменная:
Если скрипт запущен как python3 script.py → __name__ = "__main__"
Если скрипт импортирован как модуль → __name__ = "script"
Чтобы код не выполнялся при импорте в другой скрипт
