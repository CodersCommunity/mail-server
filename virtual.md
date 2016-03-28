# Konta wirtualne

Aby łatwo zarządzać kontami użytkowników (np. poprzez własny panel w PHP) stworzymy sobie bazę danych w mysql.
Warto stworzyć również osobnego użytkownika, który będzie miał do niej pełne prawa.

Tworzymy tabelę z domenami:
```
CREATE TABLE IF NOT EXISTS `virtual_domains` (
    `id`  INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(50) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Tworzymy tabelę dla użytkowników z relacją do domen:
```
CREATE TABLE IF NOT EXISTS `virtual_users` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `domain_id` INT NOT NULL,
    `password` VARCHAR(106) NOT NULL,
    `email` VARCHAR(120) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `email` (`email`),
    FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


Tworzymy tabelę aliasów z relacją do domen:
```
CREATE TABLE IF NOT EXISTS `virtual_aliases` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `domain_id` INT NOT NULL,
    `source` varchar(100) NOT NULL,
    `destination` varchar(100) NOT NULL,
    PRIMARY KEY (`id`),
    FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Tworzymy tabelę personalnych preferencji użytkownika w filtrowaniu SPAMu:
```
CREATE TABLE IF NOT EXISTS `userpref` (
        `username` varchar(100) NOT NULL,
        `preference` varchar(30) NOT NULL,
        `value` text NOT NULL,
        `prefid` int(11) NOT NULL AUTO_INCREMENT,
         PRIMARY KEY (`prefid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


Tak na prawdę struktura tych tabel jest dowolna. Odpowiednie zapytania SQL i tak trzeba będzie napisać w konfiguracji serwerów.
Można więc bez obaw dodawać kolejne kolumny, jeśli jest taka potrzeba.
