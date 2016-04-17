# POSTFIX

### Wstęp
Postfix to serwer, który umożliwia klientom wysyłanie oraz odbiera wiadomości z innych serwerów. Komunikacja z innymi serwerami odbywa się najczęsciej na porcie `25`.
Kiedy klient łączy się do postfixa, aby wysłać wiadomość robi to najczęściej na porcie `587`. W ten sposób skonfigurowanego postfixa, będę właśnie pokazywał w tym rozdziale.


### Instalacja
```
sudo apt update && sudo apt install postfix && sudo service postfix stop
```

### Struktura plików konfiguracyjnych

Plik `master.cf` zawiera serwisy, które będą uruchamiane przy starcie postfixa. Istnieje w nim możliwość nadpisywania parametrów konfiguracji pod konkretne serwisy.

Plik `main.cf` zawiera domyślne reguły konfiguracyjne serwera.


### Konfiguracja master.cf

Należy w nim odkomentować linię:
```
smtp      inet  n       -       -       -       -       smtpd
```
Aby nasz postfix słuchał na 25 porcie.


Następnie należy włączyć serwis `submissions`, aby umożliwić wysyłanie przez port 587. Należy do niego nadpisać pewne parametry.

Finalnie tak powinna wyglądać jego deklaracja:
```
submission inet n       -       -       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
```

Należy uważać przy konfiguracji `master.cf` ponieważ jest on czuły na wcięcia. Programiści PHP znają to pewnie z plików yamla. Należy kontrolować każdą pojedyńczą spacje,
ponieważ może ona zepsuć poprawne odczytanie konfiguracji.


### Konfiguracja main.cf

Po pierwsze podajemy ścieżki do naszych certyfikatów SSL.
```
smtpd_tls_cert_file=/etc/letsencrypt/live/example.com/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/example.com/privkey.pem
```

Wyłączamy przestarzałe metody szyfrowania:
```
smtpd_tls_exclude_ciphers=aNULL, eNULL, EXPORT, DES, RC4, MD5, PSK, aECDH, EDH-DSS-DES-CBC3-SHA, EDH-RSA-DES-CDB3-SHA, KRB5-DES, CBC3-SHA
smtpd_tls_protocols = !SSLv2
```

Ustawiamy LMTP jako domyślny sposób przekazywania wiadomości do skrzynek IMAP.
```
virtual_transport = lmtp:unix:private/dovecot-lmtp
```

Upewniamy się, że parametry odpowiedzialne za autoryzacje są prawidłowo ustawione:
```
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes 
smtpd_recipient_restrictions = permit_mynetworks,permit_sasl_authenticated,defer_unauth_destination
smtpd_client_restrictions= permit_mynetworks,permit_sasl_authenticated,defer_unauth_destination
smtpd_relay_restrictions = permit_mynetworks,permit_sasl_authenticated,defer_unauth_destination
```

Jeśli brakuje Ci któregoś z nich lub jest ustawiony na inną wartość dokonaj zmian w konfiguracji.


Teraz dołączamy pliki z konfiguracją zapytań do mySQL:
```
virtual_mailbox_domains = mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
virtual_mailbox_maps = mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
virtual_alias_maps = mysql:/etc/postfix/mysql-virtual-alias-maps.cf
```

Skoro je dołączyliśmy wypadałoby je stworzyć. Oto zawartość każdego z nich:

##### mysql-virtual-alias-maps.cf
```
user = mail
password = tajne_hasło
hosts = localhost
dbname = mail
query = SELECT destination FROM virtual_aliases WHERE source='%s'
```

##### mysql-virtual-mailbox-maps.cf
```
user = mail
password = tajne_hasło
hosts = localhost
dbname = mail
query = SELECT 1 FROM virtual_users WHERE email='%s'
```

##### mysql-virtual-mailbox-domains.cf
```
user = mail
password = tajne_hasło
hosts = localhost
dbname = mail
query = SELECT 1 FROM virtual_domains WHERE name='%s'
```

Przy zabezpieczaniu tych plików należy uwzględnić, ze użytkownik `postfix` powinien móc je czytać.


Błędów w konfiguracji należy szukać w `/var/log/mail.warn` oraz `/var/log/mail.err`. Domyślnie postfix rzuca logi również do sysloga w `/var/log/syslog`.

Nie startujemy postfixa, ponieważ potrzebujemy jeszcze serwera IMAP. Wybierzemy `dovecot`. Zapraszam do następnego rozdziału.

