# Ddos detector for CheckPoint
### Инструкция по конфигурации модуля: ###
</br>

**Поддерживаемые версии Gaia**: R81 и новее (Gaia Embedded (SMB) шлюзы пока не поддерживаются).</br>

1. Подключитесь к шлюзу по SSH и войдите в режим Expert.</br>

2. Далее создайте папку */bin/ddos_detector* с помощью команды:</br></br>
**`mkdir /bin/ddos_detector`**</br>

4. Переместите архив с модулем в папку (скачать его можно [по этой ссылке](https://github.com/sagittarius-H/DdosDetectorForCheckPoint/releases/download/v1.0.1/package_ddos_detector.tar)), подключившись к шлюзу по SFTP, и распакуйте архив:</br></br>
**`tar -xvf package_ddos_detector.tar`**</br>

5. Добавьте разрешение на выполнение исполняемых файлов командой:</br></br>
**`chmod +x ddos*`**</br>

6. Создайте нового пользователя Gaia, который сможет выполнять cron задания, но не будет иметь пароля и возможности подключиться по Web или SSH. Для этого выполните команды на шлюзе из режима Clish:</br></br>
**`add user jobuser uid 0 homedir /home/jobuser`**</br>
**`save config`**</br>

7. Подключитесь к Web интерфейсу шлюза и перейдите на вкладку User management -> Users. 
Задайте для созданного пользователя Shell = /bin/bash и Role = AdminRole.</br>

8. Теперь снова вернитесь к SSH подключению, в режим Expert и отредактируйте cron файл для пользователя jobuser командой:</br></br>
**`crontab -u jobuser -e`**</br></br>
В расписание необходимо вставить следующее:</br></br>
**`5 0 * * * /bin/ddos_detector/ddos_stats.run --regular`**</br>
**`*/5 * * * * /bin/ddos_detector/ddos_probe.run --regular 25 600`**</br></br>
Модуль автоматизации делится на три взаимосвязанные части (блока).</br></br>
Блок модуля автоматизации ddos_stats.run должен запускаться раз в сутки и собирать статистику за прошлые 10 дней, когда шлюз был в uptime и не было зафиксировано аномалий, указывающих на возможную атаку.</br></br>
Блок модуля автоматизации ddos_probe.run должен запускаться каждые N минут в течение всего дня и проверять метрики интерфейсов (TX, RX, Throughput TX/RX) и метрику Concurrent Connections на аномалию, указывающую на возможную атаку. Аномалией будет считаться превышение метрикой значений этой же метрики за все прошлые дни на заданный процент в течение 5 секунд. В примере выше задан порог = 25%, т.е. аномалией будет считаться значение 125% от всех прошлых метрик, Вы можете задать свой порог исходя из особенностей инфраструктуры и трафика.</br></br>
Отслеживание и интерпретация результата каждой метрики занимает не меньше 5 секунд. И хоть в программе оно выполняется параллельно для каждого интерфейса, нужно учитывать это время, поэтому запуск данного модуля чаще чем раз в 5 минут не рекомендуется.</br></br>
Для оптимизации Вы можете добавить некоторые интерфейсы в исключения проверки ddos_probe.run, указав их имена по одному на каждой строке в файле */bin/ddos_detector/config/probe_exclusions*. Проверить имена интерфейсов можно с помощью команды Clish: **`clish -c "show interfaces"`**</br></br>
Параметр 600 в примере выше обозначает минимальное количество секунд, которое должно пройти с прошлого детектирования, прежде чем программа отправит уведомление в Telegram при повторном детекте.</br></br>
Блок модуля автоматизации ddos_alert.run вызывается автоматически при необходимости, добавлять его в cron не нужно.</br>
9. Теперь необходимо настроить Telegram бот. Откройте диалог с ботом @BotFather и отправьте ему команду:</br></br>
**`/newbot`**</br>
10. Введите название бота. Например, Notifications from CP. После этого @BotFather предложит ввести уникальный username бота, по которому его можно будет найти в Telegram. Оно должно оканчиваться словом "bot". Например, cpwarningsbot.</br>
11. Сохраните значение токена из ответа. Его необходимо вставить в файл на шлюзе */bin/ddos_detector/config/token_tg*</br>
12. На последнем шаге настройки уведомлений нужно получить уникальные для каждого пользователя бота идентификаторы чатов. Затем вставить их по одному на каждой строчке в файл на шлюзе */bin/ddos_detector/config/chat_id_tg*</br></br>
Получить идентификаторы чатов можно если администратор бота, который знает токен, выполнит в браузере переход по следующей ссылке:
*[https://api.telegram.org/bot<Токен Вашего бота>/getUpdates]()*


