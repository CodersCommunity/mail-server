# SPAMASSASSIN - Filtr poczty przychodzącej

### Instalacja
```
sudo apt install spamassassin spamc && sudo service spamassassin stop
```

### Konfiguracja

Tworzymy nowego użytkownika:

```
adduser spamd --disabled-login
```

Następnie otwieramy plik `/etc/default/spamassassin`.

Ustawiamy w nim zmienną ENABLED:
```
ENABLED=1
```

Ustawiamy opcje dla spamassassin. Definiujemy m.in. gdzie mają być zapisywane logi oraz na jakim porcie ma słuchać nasz filtr.
```
SPAMD_HOME="/home/spamd"
OPTIONS="-x -q -i 127.0.0.1:783 --max-children 5 --username spamd --helper-home-dir $SPAMD_HOME/ -s $SPAMD_HOME/spamd.log"
```

Ustawiamy ścieżkę do pliku z PID:
```
PIDFILE="/var/run/spamd.pid"
```

Włączamy automatyczne aktualizacje preferencji:
```
CRON=1
```

Zapisujemy plik. Teraz otwieramy główną konfiguracje `/etc/spamassassin/local.cf`.

Ustawiamy w niej następujące parametry:
```
rewrite_header Subject ***** SPAM _SCORE_ *****
report_safe             0
required_score          4.0
use_bayes               1
use_bayes_rules         1
bayes_auto_learn        1
skip_rbl_checks         0
use_razor2              0
use_dcc                 0
use_pyzor               0
```

Za pomocą `rewrite_header` dodajemy informacje o SPAMie do tematu niechcianych wiadomości. Kolejnym ważnym parametrem jest `required_score`.
W obecnej konfiguracji działa to tak: **im większe prawdopodobieństwo SPAMu - tym więcej punktów otrzyma wiadomość.** Jeśli `required_score` jest ustawiony na `4.0`
to wiadomość, która dostanie więcej punktów niż 4 zostanie oznaczona jako SPAM.

Więcej o formacie pliku `/etc/spamassassin/local.cf` znajdziesz [tutaj.](https://spamassassin.apache.org/full/3.4.x/doc/Mail_SpamAssassin_Conf.html)


Pozostało już tylko podpiąć filtr pod postfixa. Otwieramy `/etc/postfix/master.cf`. I nadpisujemy parametr `content_filter` dla serwisu `smtp`:
```
smtp      inet  n       -       -       -       -       smtpd
  -o content_filter=spamassassin
```

Oraz definiujemy nowy serwis:
```
spamassassin unix -     n       n       -       -       pipe
  user=spamd argv=/usr/bin/spamc -u ${user}@${domain} -4 -f -e /usr/sbin/sendmail -oi -f ${sender} ${recipient}
```

Na koniec zostało:
```
sudo service spamassassin start && sudo service postfix restart
```
