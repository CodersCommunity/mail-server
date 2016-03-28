# Własny serwer pocztowy

Jak postawić własny serwer email, który nie wpada do SPAMu i sam SPAM filtruje?

## Spis treści

- Certyfikat SSL.
- Konfiguracja DNS.
- Instalacja i konfigurcja postfix.
- Instalacja i konfiguracja opendkim.
- Instalacja i konfiguracja dovecot.
- Instalacja i konfiguracja spamassassin.
- Testowanie poprawności konfiguracji.

## Wstęp

Serwery pocztowe w dzisiejszych czasach urosły do wąskiej specjalizacji, którą zajmują się ukierunkowane na nią firmy. Co jednak jeśli nie podobają Ci się limity jakie nakładają? Albo po prostu chciałbyś mieć własny serwer poczty, nad którym masz pełną kontrole?

Po przeczytaniu rozdziałów zawartych w powyższym spisie treści, będziesz umiał to zrobić, jednak musisz coś wiedzieć. Serwery pocztowe (tak na prawdę każde, ale pocztowe zwłaszcza) to ciągła odpowiedzialność. **Nie da się postawić poczty, żeby działała dobrze, bez żadnych zmian przez 5lat.** Czytanie logów i kontrola zabezpieczeń to podstawowe rzeczy jakie należą do obowiązków admina. Im częsciej tym lepiej. Bywa, że konfiguracja musi być zmieniania nawet kilka razy w tygodniu i z tym trzeba się liczyć.
