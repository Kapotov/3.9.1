# Домашнее задание к занятию «Элементы безопасности информационных систем»


### Цель задания

В результате выполнения задания вы: 

* настроите парольный менеджер, что позволит не использовать один и тот же пароль на все ресурсы и удобно работать со множеством паролей;
* настроите веб-сервер на работу с HTTPS. Сегодня HTTPS является стандартом в интернете. Понимание сути работы центра сертификации, цепочки сертификатов позволит сконфигурировать SSH-клиент на работу с разными серверами по-разному, что даёт большую гибкость SSH-соединений. Например, к некоторым серверам мы можем обращаться по SSH через приложения, где недоступен ввод пароля;
* поработаете со сбором и анализом трафика, которые необходимы для отладки сетевых проблем.


### Инструкция к заданию

1. Создайте .md-файл для ответов на задания в своём репозитории, после выполнения прикрепите ссылку на него в личном кабинете.
2. Любые вопросы по выполнению заданий задавайте в чате учебной группы или в разделе «Вопросы по заданию» в личном кабинете.

### Дополнительные материалы для выполнения задания

1. [SSL + Apache2](https://digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04).

------

## Задание

1. Установите плагин Bitwarden для браузера. Зарегестрируйтесь и сохраните несколько паролей.

# Ответ:
Я если честно не хочу устанавливать менеджер паролей для браузера. Когда-то пользовался LastPass, на работе активно использую sysPass.

2. Установите Google Authenticator на мобильный телефон. Настройте вход в Bitwarden-акаунт через Google Authenticator OTP.

# Ответ:
Для личных аккаунтов использую Microsoft Authenticator. 

3. Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.

# Ответ:
```
$ sudo apt install apache2
Генерируем самоподписанный сертификат:
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
(везде жал просто Enter, для создания apache-selfsigned.key ввел passphrase)

Пример конфига apache:
vim /etc/apache2/sites-available/default-ssl.conf

<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin superadmin@test-srv.com
                ServerName test-srv.com

                DocumentRoot /var/www/html

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
                SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

        </VirtualHost>
</IfModule>

a2enmod ssl
a2enmod headers
a2ensite default-ssl
a2enconf ssl-params

vim /etc/apache2/sites-enabled/000-default
<VirtualHost *:80>
        . . .

        Redirect permanent "/" "https://test-srv.com/"

        . . .
</VirtualHost>

systemctl restart apache2

в /etc/hosts добавлен test-srv.com

root@vagrant:~# curl -s -I -XGET http://test-srv.com
HTTP/1.1 302 Found
Date: Thu, 03 Mar 2022 12:37:49 GMT
Server: Apache/2.4.41 (Ubuntu)
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Location: https://test-srv.com/
Content-Length: 277
Content-Type: text/html; charset=iso-8859-1
```


4. Проверьте на TLS-уязвимости произвольный сайт в интернете (кроме сайтов МВД, ФСБ, МинОбр, НацБанк, РосКосмос, РосАтом, РосНАНО и любых госкомпаний, объектов КИИ, ВПК и т. п.).
# Ответ:

```
root@vagrant:~/testssl.sh# ./testssl.sh -U --sneaky https://en.wikipedia.org/

###########################################################
    testssl.sh       3.1dev from https://testssl.sh/dev/

      This program is free software. Distribution and
             modification under GPLv2 permitted.
      USAGE w/o ANY WARRANTY. USE IT AT YOUR OWN RISK!

       Please file bugs @ https://testssl.sh/bugs/

###########################################################

 Using "OpenSSL 1.0.2-chacha (1.0.2k-dev)" [~179 ciphers]
 on spb-svc:./testssl.sh/bin/openssl.Linux.x86_64
 (built: "Jan 18 17:12:17 2019", platform: "linux-x86_64")


 Start 2021-12-08 15:13:22        -->> 91.198.174.192:443 (en.wikipedia.org) <<--

 Further IP addresses:   2620:0:862:ed1a::1 
 rDNS (91.198.174.192):  text-lb.esams.wikimedia.org.
 Service detected:       HTTP


 Testing vulnerabilities 

 Heartbleed (CVE-2014-0160)                not vulnerable (OK), no heartbeat extension
 CCS (CVE-2014-0224)                       not vulnerable (OK)
 Ticketbleed (CVE-2016-9244), experiment.  not vulnerable (OK)
 ROBOT                                     Server does not support any cipher suites that use RSA key transport
 Secure Renegotiation (RFC 5746)           supported (OK)
 Secure Client-Initiated Renegotiation     not vulnerable (OK)
 CRIME, TLS (CVE-2012-4929)                not vulnerable (OK)
 BREACH (CVE-2013-3587)                    no gzip/deflate/compress/br HTTP compression (OK)  - only supplied "/" tested
 POODLE, SSL (CVE-2014-3566)               not vulnerable (OK)
 TLS_FALLBACK_SCSV (RFC 7507)              No fallback possible (OK), no protocol below TLS 1.2 offered
 SWEET32 (CVE-2016-2183, CVE-2016-6329)    not vulnerable (OK)
 FREAK (CVE-2015-0204)                     not vulnerable (OK)
 DROWN (CVE-2016-0800, CVE-2016-0703)      not vulnerable on this host and port (OK)
                                           make sure you don't use this certificate elsewhere with SSLv2 enabled services
                                           https://censys.io/ipv4?q=488ABE61E7804EF26974DF1FB95B5ACA7FFECC7B17BD8BD7BA89D4CF99169ECA could help you to find out
 LOGJAM (CVE-2015-4000), experimental      not vulnerable (OK): no DH EXPORT ciphers, no DH key detected with <= TLS 1.2
 BEAST (CVE-2011-3389)                     not vulnerable (OK), no SSL3 or TLS1
 LUCKY13 (CVE-2013-0169), experimental     not vulnerable (OK)
 Winshock (CVE-2014-6321), experimental    not vulnerable (OK)
 RC4 (CVE-2013-2566, CVE-2015-2808)        no RC4 ciphers detected (OK)


 Done 2021-12-08 15:15:55 [ 160s] -->> 91.198.174.192:443 (en.wikipedia.org) <<--
 ```

5. Установите на Ubuntu SSH-сервер, сгенерируйте новый приватный ключ. Скопируйте свой публичный ключ на другой сервер. Подключитесь к серверу по SSH-ключу.

# Ответ:
```
vm1: 
ssh-keygen -t rsa
cd ./ssh
ssh-copy-id -i id_rsa.pub vagrant@10.20.2.5

vm2:
vim /etc/ssh/sshd_config
PermitRootLogin no 
PasswordAuthentication no 

systemctl restart ssh.service
ufw allow ssh
```
 
6. Переименуйте файлы ключей из задания 5. Настройте файл конфигурации SSH-клиента так, чтобы вход на удалённый сервер осуществлялся по имени сервера.
# Ответ:
```
mv ~/.ssh/id_dsa ~/.ssh/id_dsa_bla_bla
vim ~/.ssh/config
Host *
  IdentitiesOnly yes
Host test-vm2
  Hostname 10.20.2.5
  User vagrant
  IdentityFile ~/.ssh/id_dsa_bla_bla
  ```

7. Соберите дамп трафика утилитой tcpdump в формате pcap, 100 пакетов. Откройте файл pcap в Wireshark.
# Ответ:

*В качестве решения приложите: скриншоты, выполняемые команды, комментарии (при необходимости).*

 ---
 
## Задание со звёздочкой* 

Это самостоятельное задание, его выполнение необязательно.

8. Просканируйте хост scanme.nmap.org. Какие сервисы запущены?

9. Установите и настройте фаервол UFW на веб-сервер из задания 3. Откройте доступ снаружи только к портам 22, 80, 443.

----

### Правила приёма домашнего задания

В личном кабинете отправлена ссылка на .md-файл в вашем репозитории.

-----

### Критерии оценки

Зачёт:

* выполнены все задания;
* ответы даны в развёрнутой форме;
* приложены соответствующие скриншоты и файлы проекта;
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку:

* задание выполнено частично или не выполнено вообще;
* в логике выполнения заданий есть противоречия и существенные недостатки.  
 
Обязательными являются задачи без звёздочки. Их выполнение необходимо для получения зачёта и диплома о профессиональной переподготовке.

Задачи со звёздочкой (*) являются дополнительными или задачами повышенной сложности. Они необязательные, но их выполнение поможет лучше разобраться в теме.
