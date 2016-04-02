# Dovecot - serwer IMAP/POP3

Jest to serwer, który umożliwia użytkownikom logowanie, zarządzanie i przeglądanie skrzynek pocztowych. Wykorzystuje do tego głównie jeden z dwóch protokołów: IMAP lub POP3.
W tym artykule pokazuję jak skonfigurować go pod IMAP. Powodem tego jest ciągle spadająca popularność POP3, ze względu na małe możliwości jakie daje w porównaniu do IMAP.


### Instalacja

```
sudo apt-get update && sudo apt-get install dovecot-imapd dovecot-lmtp && sudo service dovecot stop
```

### Konfiguracja

Główny plik konfiguracyjny `/etc/dovecot/dovecot.conf` robi to co większość głównych plików konfiguracyjnych serwerów - dołącza pliki podrzędne z konfiguracją konkretnych funkcjonalności.
Jedyne co musimy w nim teraz zrobić to dodać taką linijkę:
```
protocols = imap lmtp
```

Włącza ona dovecot do działania na protokołach:

 - `imap`: służący do połączenia klienta z serwerem
 - `lmtp`: służący do dostarczania poczty z postfix


Kolejnym folderem, który nas interesuje jest `/etc/dovecot/conf.d`. W nim znajdują się konfiguracje różnych funkcjonalności. W tych plikach będziemy musieli wprowadzić najwięcej zmian.


##### 10-mail.conf
W pliku `/etc/dovecot/conf.d/10-mail.conf` należy dodać takie linie:
```
mail_location = maildir:/var/mail/%d/%n
mail_privileged_group = mail
```

Definiujemy w ten sposób, gdzie będą znajdowały się skyrznki pocztowe na dysku.

Następnie dodamy nowego grupę i użytkownika systemowego, który będzie miał dostęp do skrzynek. W terminalu należy wykonać takie polecenie:
```
sudo groupadd -g 5000 vmail && sudo useradd -g vmail -u 5000 vmail -d /var/mail && sudo chown -R vmail:vmail /var/mail
```

##### 10-auth.conf
Następnie przenosimy się do pliku `/etc/dovecot/conf.d/10-auth.conf`. W nim ustawimy główne parametry logowania oraz dołączymy plik z konfiguracją zapytań mySQL.

Wyłączamy logowanie plaintext:
```
disable_plaintext_auth = yes
```

Ustawiamy wspierane mechanizmy logowania:
```
auth_mechanisms = plain login
```

Wyłączamy systemowych użytkowników biorąc w komentarz te linie:
```
# !include auth-system.conf.ext
```

Włączamy autoryzacje przez mySQL dodając taką linie:
```
!include auth-sql.conf.ext
```

##### auth-sql.conf.ext
W tym pliku dołączamy plik z konfiguracją zapytań mySQL. Należy w nim dodać następujący fragment:
```
passdb {
  driver = sql
  args = /etc/dovecot/dovecot-sql.conf.ext
}
userdb {
  driver = static
  args = uid=vmail gid=vmail home=/var/mail/vhosts/%d/%n
} 
```

Następnie otwieramy plik `/etc/dovecot/dovecot-sql.conf.ext` i ustawiamy w nim credentials i zapytania do bazy danych:
```
driver = mysql
connect = host=127.0.0.1 dbname=email_database user=my_email_user password=secret_password_123
default_pass_scheme = SHA512-CRYPT
password_query = SELECT email as user, password FROM virtual_users WHERE email='%u';
```

Zostało tylko dać uprawnienia użytkownik `vmail` do konfiguracji dovecot. Robimy to takim poleceniem w terminalu:
```
sudo chown -R vmail:dovecot /etc/dovecot && sudo chmod -R o-rwx /etc/dovecot 
```

##### 10-master.conf
W tym pliku czeka nas najwięcej zmian. Pierwszą sprawą jest ustawienie lmtp jako protokołu odbierania poczty z serwera oraz wyłączenie nieszyfrowanego `IMAP`:
```
service imap-login {
  inet_listener imap {
    port = 0
}

service lmtp {
   unix_listener /var/spool/postfix/private/dovecot-lmtp {
	   mode = 0600
	   user = postfix
	   group = postfix
   }
} 
```

Następnie musimy odnaleźć `service auth` i wprowadzić w nim następujące zmiany:
```
service auth {

  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }

  unix_listener auth-userdb {
    mode = 0600
    user = vmail
    #group =
  }

  user = dovecot
}
```

Na koniec w tym pliku zostało już tylko zmienić użytkownika serwisu `auth-worker` na `vmail`, którego niedawno stworzyliśmy:
```
service auth-worker {
  # Auth worker process is run as root by default, so that it can access
  # /etc/shadow. If this isn't necessary, the user should be changed to
  # $default_internal_user.
  user = vmail
}
```

##### 10-ssl.conf

Na deser została nam konfiguracja szyfrowania. Ustawiamy wymaganie połączenia szyfrowanego w pliku `/etc/dovecot/conf.d/10-ssl.conf`:
```
ssl = required
```

A następnie podpinamy nasze certyfikaty SSL:
```
ssl_cert = </etc/letsencrypt/live/example.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/example.com/privkey.pem
```

Oraz wyłączamy przestarzałe metody szyfrowania:
```
ssl_cipher_list = ALL:!LOW:!SSLv2:!SSLv3:!EXP:!aNULL
```

W tym momencie możemy już wystartować nasz serwer pocztowy:
```
sudo service opendkim start && sudo service dovecot start && sudo service postfix start
```

I zweryfikować czy słuchają na portach poleceniem `sudo netstat -plnt`: 

    - postfix: 25 oraz 587
    - dovecot: 993
    - opendkim: 12301
