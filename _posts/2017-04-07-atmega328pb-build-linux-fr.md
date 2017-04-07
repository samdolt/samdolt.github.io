---
layout: post
title:  "Compilation d'un programme C/C++ pour ATmega328PB"
date:   2017-04-07 08:05:00 +0200
categories: avr
lang: fr
---

## Configuration d'avr-libc

De base avr-libc ne supporte pas encore ce microcontrôleur (`avr-libc 2.0.0`). Les fichiers nécessaires à son support peuvent être télécharger à l'adresse suivante: [http://packs.download.atmel.com/Atmel.ATmega_DFP.1.2.132.atpack](http://packs.download.atmel.com/Atmel.ATmega_DFP.1.2.132.atpack)

Une fois l'archive décompressée, on a besoin des fichiers:

- `gcc/dev/atmega328pb/avr5/crtatmega328pb.o` - C Runtime for ATmega328PB
- `gcc/dev/atmega328pb/avr5/libatmega328pb.a`
- `include/avr/iom328pb.h` - Définition des registres

Comme l'archive contient des fichiers binaires précompilés (`.a` et `.o`), il est préférable d'utiliser la version du compilateur `avr-gcc` fournie par Atmel ([http://www.atmel.com/tools/atmelavrtoolchainforlinux.aspx](http://www.atmel.com/tools/atmelavrtoolchainforlinux.aspx)).

## Installation

### Sous Archlinux

Sous Archlinux, avr-gcc et avr-libc en version Atmel peuvent être installés via AUR:

```sh
$ pacaur -S avr-gcc-atmel avr-libc-atmel
```

J'ai créé un packet AUR pour l'installation des fichiers pour le support du Atmega328PB:

```sh
$ pacaur -S avr-libc-atmel-atmega328pb
```

### Avec une autre distribution

Il suffit de copier les trois fichiers dans les dossiers d'avr-gcc.

Par exemple, sous Archlinux:

```sh
$ sudo install -Dm644 gcc/dev/atmega328pb/avr5/crtatmega328pb.o "/usr/avr/lib/avr5/crtatmega328pb.o"
$ sudo install -Dm644 gcc/dev/atmega328pb/avr5/libatmega328pb.a "/usr/avr/lib/avr5/libatmega328pb.a"
$ sudo install -Dm644 include/avr/iom328pb.h "/usr/avr/include/avr/iom328pb.h"
```

## Compilation d'un programme C

Créer un fichier `main.c` avec le contenu suivant:

```c
#include <avr/io.h>

// Allume un led sur la pin PB5

#define LED_DDR  DDRB
#define LED_PORT PORTB
#define LED_PIN  PB5

void main(void){
  LED_DDR |= (1<<LED_PIN);
  LED_PORT ^= ~(1<<LED_PIN);

  for(;;);
}
```

On peut compiler le fichier avec la commande suivante:

```
$ avr-gcc main.c -D__AVR_DEV_LIB_NAME__=m328pb -mmcu=atmega328pb -o hello.elf
$ avr-objcopy -j .text -j .data -O ihex hello.elf hello.hex
```

## Aller plus loins

La programmation d'un ATmega328PB avec `avrdude` est expliquée dans l'article [Programmation d'un microcontrôleur ATmega328PB]({% post_url 2017-04-07-atmega328pb-flash-linux-fr %}).
