# Домашняя работа OTUS-Angie-2025-10-10 OTUS_Angie_10_Оптимизация_производительности_веб-сервисов

#### 1. Рассмотрите, какие возможности по клиентской оптимизации вы можете применить для текущей конфигурации.
Будем делать клиентское кеширование, динамическое сжатие zstd, включим HTTP2, для этого так же настроим TLS.

#### 2. Внесите изменения в настройки сайта.

В файле [angie/angie.conf](https://github.com/SergeyBoonin/OTUS-Angie-2025-10-10/blob/main/angie/angie.conf)

- В контексте main
```
load_module modules/ngx_http_zstd_filter_module.so;
load_module modules/ngx_http_zstd_static_module.so;
```
- В контексте http
```
    zstd            on;
    zstd_min_length 256;
    zstd_comp_level 3;
    zstd_types      text/plain text/css text/xml text/javascript 
                    application/json application/javascript 
                    application/x-javascript application/xml 
                    application/xml+rss application/atom+xml
                    image/svg+xml;
```
В файле [/angie/http.d/default.conf](https://github.com/SergeyBoonin/OTUS-Angie-2025-10-10/blob/main/angie/http.d/default.conf)
```
server {
#        listen 80;
#        listen [::]:80;
        listen 443 ssl;
        listen [::]:443 ssl;

        ssl_certificate     /etc/angie/ssl/certs/wordpress.local.crt;
        ssl_certificate_key /etc/angie/ssl/keys/wordpress.local.key;
        ssl_protocols       TLSv1.2 TLSv1.3;

        http2 on;
```
#### 3. Настройте серверное кэширование одной страницы сайта.
В файле [/angie/http.d/default.conf](https://github.com/SergeyBoonin/OTUS-Angie-2025-10-10/blob/main/angie/http.d/default.conf)
```
        location ~* \.(css|gif|ico|jpeg|jpg|js|png|woff2)$ {
                expires max;
                log_not_found off;
        }
```
#### 4. Обоснуйте использование тех или иных решений.
После предыдущей домашней работы некоторая оптимизация (в частности клиентское кеширование для части объектов уже было настроено). Тестирование выявило то, что файлы шрифтов (.woff2) в условие кеширования не попадали. Строка регулярного выражения была дополнена.

Динамический ZSTD был выбран в качестве алгоритма сжатия, поскольку в нстоящий момент мне не известно, легко ли реализовать автоматическое формирование статических .br файлов при внесении изменений в процессе вёрстки сайта в WordPress.

HTTP2 был включен для мультиплексирования потоков.

#### 5. Дополнительно: проведите тестирование сайта до и после оптимизации, сделайте выводы об эффективности шагов.
Значительного улучшения производительности в условиях лабы выявлено не было, однако количество замечаний системы тестирования уменьшилось.

- Результаты теста до оптимизации: [01_Test_Before_optimization.html](https://htmlpreview.github.io/?https://github.com/SergeyBoonin/OTUS-Angie-2025-10-10/blob/main/01_Test_Before_optimization.html)
- Результаты теста после оптимизации: [02_Test_After_optimization.html](https://htmlpreview.github.io/?https://github.com/SergeyBoonin/OTUS-Angie-2025-10-10/blob/main/02_Test_After_optimization.html)
