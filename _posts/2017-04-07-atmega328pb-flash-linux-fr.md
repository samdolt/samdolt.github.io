---
layout: post
title:  "Programmation d'un microcontrôleur ATmega328PB"
date:   2017-04-07 08:55:00 +0200
categories: avr
lang: fr
---

Cet article explique comment configurer `avrdude` pour programmer un ATmega328PB avec un fichier `hex` pré-existant.

Pour la compilation d'un programme `C` en fichier `hex`, voir 
[cet article]({% post_url 2017-04-07-atmega328pb-build-linux-fr %})

## Configuration d'avrdude

De base, `avrdude` ne supporte pas l'ATmega328PB (`avrdude v.0.6.3`).

Pour ajouter son support, il suffit de créer un fichier `avrdude-m328pb.conf` avec le contenu suivant:

```
part parent "m328"
    id = "m328pb";
    desc = "ATmega328PB";
    signature = 0x1e 0x95 0x16;

    ocdrev = 1;

    memory "efuse"
        size = 1;
        min_write_delay = 4500;
        max_write_delay = 4500;
        read = "0 1 0 1 0 0 0 0 0 0 0 0 1 0 0 0",
            "x x x x x x x x o o o o o o o o";

        write = "1 0 1 0 1 1 0 0 1 0 1 0 0 1 0 0",
            "x x x x x x x x x x x x i i i i";
    ;

;
```

## Programmation du microcontrôleur

On peut maintenant utiliser avrdude pour programmer notre microcontrôleur.
Pour les tests, j'utilise la carte `ATmega328PB XPlained mini` qui vient de base avec un programmateur "built-in". Pour l'utilisation d'un autre programmateur, il faut adapter `-c xplainedmini`.

```sh
$ avrdude -c xplainedmini -p m328pb -U flash:w:main.hex -C +avrdude-m328pb.conf
```

Ne pas oublier le `+` avant le nom de fichier.
