# DKIM -  DomainKeys Identified Mail

Najprościej wytłumaczyć zasadę działania DKIM w taki sposób:

Generujemy klucz dla danej domeny, a następnie umieszczany go w [odpowiednim rekordzie DNS.](https://github.com/CodersCommunity/mail-server/blob/master/dns.md#dkim)
Podczas wysyłania wiadomości z naszego serwera dołączany jest do niej podpis. Serwer, który odbierze naszą wiadomość sprawdzi ten podpis i nasz klucz w DNS.
Jeśli nie będą się zgadzać, będzie mógł rozpoznać ewentualnego podszywacza, który to próbował wysłać wiadomość jako nasz serwer.


### Instalacja
```
sudo apt-get update && sudo apt-get install opendkim opendkim-tools && sudo service opendkim stop
```

### Konfiguracja główna

Otwieramy plik `/etc/opendkim.conf`. Następnie wprowadzamy w nim takie zmiany:
```
AutoRestart             Yes
AutoRestartRate         10/1h
UMask                   002
Syslog                  yes
SyslogSuccess           Yes
LogWhy                  Yes

Canonicalization        relaxed/simple

ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
InternalHosts           refile:/etc/opendkim/TrustedHosts
KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable

Mode                    sv
PidFile                 /var/run/opendkim/opendkim.pid
SignatureAlgorithm      rsa-sha256

UserID                  opendkim:opendkim

Socket                  inet:12301@localhost
```

Najważniejszy parametr to `Socket`, w którym definiujemy na jakim porcie i adresie będzie słuchał serwer opendkim. 
Jest to ważne, ponieważ należy następnie podłączyć postfixa do opendkim na tym samym porcie.

Następnie dodajemy odpowiednią linie do pliku `/etc/default/opendkim`:
```
SOCKET="inet:12301@localhost"
```

Widzimy w konfiguracji `opendkim.conf` również 3 ścieżki do plików z tabelami. Stwórzmy więc te pliki oraz folder na klucze:
```
sudo mkdir /etc/opendkim && \
sudo touch /etc/opendkim/TrustedHosts && \
sudo touch /etc/opendkim/KeyTable && \
sudo touch /etc/opendkim/SigningTable && \
sudo mkdir /etc/opendkim/keys
```

To w nich będziemy przechowywać nasze klucze DKIM.

Jeśli chcesz dowiedzieć się więcej o formacie konfiguracji opendkim [zobacz ten link.](http://www.opendkim.org/opendkim.conf.5.html)



### Podłączenie postfixa

Aby postfix mógł podpisywać wysyłane maile oraz sprawdzać podpisy przychodzących należy podłączyć go pod opendkim.

Dodajemy do pliku `/etc/postfix/main.cf` następujące parametry:
```
milter_protocol = 6
milter_default_action = accept
smtpd_milters = inet:localhost:12301
non_smtpd_milters = inet:localhost:12301
```


### Generowanie kluczy 

Aby sprawnie generować nowe klucze można napisać kawałek skryptu bash. Będzie to zdecydowanie szybsze rozwiązanie niż robienie tego wszystkiego ręcznie.
Moja implementacja wygląda w ten sposób:
```sh
# !/usr/bin/env bash
if [ -z "$1" ] ; then 
    echo "Require domain for dkim."; exit; 
fi  
 
echo "*@$1 mail._domainkey.$1" >> /etc/opendkim/SigningTable; 
echo "Added signing table record!"; 

echo "mail._domainkey.$1 $1:mail:/etc/opendkim/keys/$1/mail.private" >> /etc/opendkim/KeyTable 
echo "Added key to table!"; 

mkdir /etc/opendkim/keys/$1 
opendkim-genkey -s mail -D /etc/opendkim/keys/$1 -d $1 
chown opendkim:opendkim /etc/opendkim/keys/$1/mail.private 
echo "Generated DKIM!"; 

echo; 
echo "Copy this key and add to DNS server."; 
cat /etc/opendkim/keys/$1/mail.txt;
```

Zapisujemy taki plik, np. jako `/usr/local/bin/generate_dkim`, a następnie dajemy prawa do wykonania:
```
sudo chmod +x /usr/local/bin/generate_dkim
```

Teraz wystarczy, że wykonamy ten plik z argumentem, który będzie domeną dla jakiej generujemy klucz:
```
sudo generate_dkim example.com
```
Na wyjściu zobaczymy klucz jaki należy wpisać do [odpowiedniego rekordu DNS.](https://github.com/CodersCommunity/mail-server/blob/master/dns.md#dkim)

Następnie wystarczy zrestartować postfixa i opendkim:
```
sudo service opendkim restart && sudo service postfix restart
```

Jeśli nie skonfigurowałeś jeszcze `dovecot`, o którym będę pisał w następnym rozdziale powstrzymaj się jeszcze ze startowaniem tych serwerów.
