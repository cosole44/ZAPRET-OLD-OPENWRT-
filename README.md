Установка tpws на OpenWrt 15.05 (TP-Link TL-WR841N v8.4)

Инструкция по настройке обхода DPI с помощью tpws на старой версии OpenWrt (Chaos Calmer 15.05) с iptables.

⸻

📋 Требования
	•	OpenWrt 15.05 (Chaos Calmer)
	•	Роутер с архитектурой mips (например, TP-Link TL-WR841N v8.4)
	•	Локальный доступ к файловой системе и SSH
	•	tpws-бинарник, собранный под mips (big endian) (не mipsel)

⸻

📦 Шаг 1: Копирование файлов на роутер

Используйте scp с поддержкой старых алгоритмов:

scp -O -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa -r ./init.d/openwrt-minimal/tpws root@192.168.10.1:/
scp -O -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa ./tpws root@192.168.10.1:/usr/bin/tpws


⸻

🔐 Шаг 2: Подключение по SSH

ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa root@192.168.10.1


⸻

🚚 Шаг 3: Перемещение файлов в нужные директории

mv /tpws/etc/init.d/tpws /etc/init.d/tpws
mv /tpws/etc/config/tpws /etc/config/tpws
chmod 755 /etc/init.d/tpws /usr/bin/tpws


⸻

⚙️ Шаг 4: Конфигурация /etc/config/tpws

Пример базовой конфигурации:

config global defaults
    option user daemon
    option tpws /usr/bin/tpws

config tpws
    option port 900
    option opt '--split-pos=2 --oob'
    option enabled 1


⸻

🔥 Шаг 5: Настройка firewall.user

Добавим редирект HTTPS-трафика на локальный порт 900:

export DISABLE_IPV6=1
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 900

Файл /etc/firewall.user должен содержать эти строки.

⸻

🚀 Шаг 6: Запуск

/etc/init.d/tpws enable
/etc/init.d/tpws start
fw3 restart  # либо /etc/init.d/firewall restart


⸻

✅ Шаг 7: Проверка

ps | grep tpws                # проверка процесса
netstat -tuln | grep 900      # прослушивает ли порт
logread | grep tpws           # есть ли лог


⸻

🧾 Примечания
	•	Пакет iptables-mod-extra недоступен для OpenWrt 15.05 — но он и не требуется, если используются базовые правила iptables (REDIRECT).
	•	Конфигурация tpws может быть расширена для обхода блокировок YouTube, Discord, Cloudflare и т.д.
	•	Используемые параметры в option opt нужно адаптировать под поддерживаемые tpws.

⸻

Готово! Вы успешно настроили tpws на старом роутере для базового DPI-обхода. 🎉
