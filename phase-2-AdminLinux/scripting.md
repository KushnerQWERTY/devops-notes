#!/bin/bash

# Объявляем переменную с именем службы
SERVICE="my-monitor"

# Проверяем статус службы
# systemctl is-active возвращает 0 (истина), если служба работает
if systemctl is-active --quiet $SERVICE; then
    echo "✅ Отлично: Служба $SERVICE работает нормально."
else
    echo "❌ ВНИМАНИЕ: Служба $SERVICE не работает! Пытаемся перезапустить..."
    
    # Пытаемся перезапустить
    sudo systemctl restart $SERVICE
    
    # Проверяем еще раз после перезапуска
    if systemctl is-active --quiet $SERVICE; then
        echo "✅ Успех: Служба $SERVICE была успешно перезапущена."
    else
        echo "🚨 КРИТИЧЕСКАЯ ОШИБКА: Не удалось запустить $SERVICE. Проверьте логи!"
    fi
fi
