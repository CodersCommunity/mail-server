# Certyfikat SSL

Nie będę się tutaj rozwodził dlaczego warto szyfrować. Po prostu warto. 
Należy jednak pamiętać o poprawnym ustawieniu serwerów, aby nasze szyfrowanie nie było daremne.


Aby używać szyfrowania potrzebujesz certyfikatu SSL. Większość certyfikatów jest płatna, ze względu na długi termin ważności.


Jednak jeśli nie chcesz wydać ani złotówki na nie, polecam Ci zobaczyć to: [letsencrypt](https://letsencrypt.org/getting-started/)

Są to darmowe certyfikaty, które trzeba odnawiać co 3 miesiące. Ale kto sprytny ten sobie poradzi. ;)
Po wygenerowaniu certyfikatów dla domeny `example.com` znajdują się one w `/etc/letsencrypt/live/example.com`.
Możesz używać ich również z apache, nginx czy innym serwerm http, aby umożliwić połączenia https.


Jeśli posiadasz już pliki z certyfikatem i kluczem, nazwijmy je sobie na potrzeby tego kursu: `cert.pem` i `privkey.pem`. 
Pamiętaj o połączeniu certyfikatu z tzw. chainem, jeśli Twój dostawca tego wymaga. 
Jeśli korzystasz z letsencrypt połączony plik nazywa się `fullchain.pem` i to jego powinienieś traktować jako plik z certyfikatem.
