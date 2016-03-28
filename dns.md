# Domain Name System
 
Jeśli chcesz mieć dobry serwer pocztowy, potrzebujesz szybkiego i łatwego dostępu do konfiguracji DNS domen.

### Rekord MX
Pierwsza i postawowa sprawa to rekord MX. Należy go ustawić na domenę, która wskazuje na serwer pocztowy. 

Przyjmijmy więc, że ustawiamy rekord MX dla domeny `example.com`.
Natomiast domena `example.com` wskazuje na adres ip serwera, na którym znajduje się zarówno serwer pocztowy jak i webowy.
```
example.com.     IN      A      5.6.7.8
```

Należałoby więc umieścić taki wpis w konfiguracji DNS:
```
example.com.     IN      MX      10 example.com.
```

Cyferka 10 przed adresem to priorytet, który przydaje się kiedy mamy więcej serwerów. W tym przypadku ustawiamy najwyższy.



Jeśli serwer poczty mamy na innej maszynie, z innym adresem ip niż serwer webowy, należy dodać nową subdomenę wskazującą na serwer poczty:
```
mail.example.com.     IN      A      1.2.3.4
```

A następnie ustawić rekord MX, na te subdomenę:
```
example.com.     IN      MX      10 mail.example.com.
```




### SPF - Sender Policy Framework

Jest to specjalny wpis w DNS, który definiuje jakie serwery są uprawnione do wysyłania emaili z tej domeny.
Przyjmując więc, że adres ip naszego serwera poczty to: `5.6.7.8`, nalezy umieścić w DNS taki wpis typu TXT:

```
example.com.     IN      TXT      "v=spf1 ip4:5.6.7.8 -all"
```

Jeśli chcesz dowiedzieć się więcej o składni SPF, [przeczytaj koniecznie to](http://www.openspf.org/SPF_Record_Syntax).




### DKIM

Jest to kolejny wpis typu TXT w DNS. Tym razem dla subdomeny `mail._domainkey.example.com`. Powinien on zawierać specjalny klucz wygenerowany przez `opendkim`.
Opis generowania takich kluczy znajdziesz w rozdziale `Instalacja i konfiguracja opendkim.`. Przykładowy wpis DKIM, wygląda tak:
```
mail._domainkey.example.com.    IN      TXT     "v=DKIM1; k=rsa; p=długi_losowy_ciąg_znaków"
```

