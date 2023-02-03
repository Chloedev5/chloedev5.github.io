---
layout: post
title: Controler un chauffe-lait avec Arduino
categories: Agriculture, Arduino, Chauffe-lait
description: Pour que votre lait soit toujours à la bonne température
keywords: Agriculture Arduino Chauffe-lait
---

**Pour que votre lait soit toujours à la bonne température !**

<!-- affichage du sommaire en haut de la page
* TOC
{:toc}
-->

### Oups... 🫢
Je travaille dans une ferme laitière, on élève des vaches et leurs veaux.
Le matin, je commence ma journée en allant voir les vaches qui sont a quelques jours pour voir si elles ont vêlé pendant la nuit.
Ensuite, je vais dans les vaches laitières, je les pousse en salle de traite, je nettoie et paille les logettes (leur lit), et je trais.
Après la traite, vient le moment de s'occuper des veaux. Je mets donc le lait que j'ai gardé pour eux a chauffer, et pendant ce temps je nettoie la salle de traite.
Le problème, c'est que le chauffe lait ne s'arrête pas et parfois, il suffit que je ne fasse pas attention et je retrouve le lait à 70°C.
Dommage quand on doit le donner aux veaux entre 38°C et 40°C. Donc après il faut attendre que le lait refroidisse à une température optimale (45°C) avant de pouvoir commencer a abreuver les veaux.

Perte de temps, les veaux ont faim, ... ça n'arrange personne, il fallait donc trouver une solution pour que je puisse me permettre de faire mon travail sans regarder la température du lait toutes les deux minutes.

### L'astuce
Bricoleuse comme je suis, j'allais bien trouver une solution.
Après réflexion, je me suis souvenue que j'avais un Arduino Uno a trainer chez moi. J'avais acheté ça parce que j'en avais entendu parler et je voulais tester ce qu'il est possible de faire avec.
En combinant une sonde de température étanche qu'on mettrait dans le lait et un relais qui commanderait l'alimentation du chauffe lait, l'Arduino pourrait récuperer la température du lait, couper le chauffe lait quand le lait est trop chaud, et le remettre en route quand le lait est trop froid.

### Le matériel
J'ai donc commandé une sonde de température DS18B20 sur Amazon, ainsi que des relais 230 volts (attention a bien prendre des 230 volts si non ça peut être dangereux) commandables en 5 volts (du coté de l'Arduino) AC KY019 toujours chez Amazon
A cela il faut ajouter deux prises, une étanche pour brancher le chauffe lait, et une (peu encombrante, non étanche si possible pour gagner de la place dans la boite de dérivation) autre pour l'alimentation de l'Arduino ainsi qu'une boite de dérivation étanche pour rendre toute la partie sensible à l'humidité (alimentation de l'Arduino et du chauffe lait, Arduino, relais) étanche vu qu'on lave la laiterie au jet d'eau plusieurs fois par jours. Ça pourrait éviter des problèmes.

#### Attention, danger
Il faut faire attention à la résistance du chauffe lait qui n'est pas prévue pour être coupée puis remise en route sans arrêt, ça pourrait la faire claquer et il faudrait donc changer le chauffe lait (200€ environ)
Pour éviter ça, on ne va pas définir une température cible unique dans l'Arduino (48°C dans mon cas, pour pallier la perte de température liée au passage en exterieur pour emmener le lait de la laiterie à la nurserie et aux seaux froids), mais on va plutôt opter pour une température haute, à laquelle le chauffe lait va se couper, et une température basse, à laquelle le chauffe lait va se remettre en route, ce qui va laisser plus de temps entre une coupure et une mise en route du chauffe lait. Par exemple dans mon cas, j'ai choisi un écart de 5°C, que vous pourrez ajuster en fonction de vos besoins (selon la matière du bac que vous utilisez pour chauffer le lait qui va plus ou moins bien conserver la température du lait par exemple).
Ça va donc donner :
Température haute (coupure): 53°C
Température basse (mise en route): 48°C

Il faut aussi penser a ajouter un délai entre deux mesures pour éviter des erreurs de mesures. Un délai de 30 secondes est suffisant, et ne pénalisera ni la qualité de la mesure ni la véracité de la température du lait au moment ou j'en ai besoin.

#### Petite idée
Je pensais aussi a intégrer trois LED dans la boite de dérivation, qui seraient visibles de l'extérieur de la boite de dérivation et qui indiqueraient visuellement la température du lait : Par exemple une verte qui s'allumerait quand le lait est entre la température haute et la température basse, une rouge quand le lait est trop froid, et une jaune quand le lait est trop chaud. Ça permettrait carrément de ne plus utiliser de thermomètre en plus de la sonde de température, et ça faciliterait la lecture.

#### Les branchements
Avant de mettre tout ça en route, il faut déjà tout brancher comme il faut.
Voici comment j'ai procédé :

##### Sonde de température
La sonde dispose de 3 fils : Un fil rouge (+ 3.3v ou +5v, la sonde prend les deux en charge), un fil noir (masse), et un fil jaune qui va remonter l'info sur la température à l'Arduino.
On va donc brancher le fil rouge sur un pin qui est marqué 3.3v ou 5v, le fil noir sur un pin marqué GND, et le fil jaune sur un pin d'entrée analogique marqué Ax (ou x est un nombre entre 0 et 5). J'ai choisi le pin A1 dans mon cas.

##### Relais
Le relais fonctionne de la même manière que la sonde de température : un pin +5v (attention, le relais ne fonctionne qu'avec du 5v), un pin masse et un pin de contrôle.
Pour le brancher, on va donc procéder de la même manière que la sonde : relier le pin + sur un pin de l'Arduino marqué 5v, le pin masse sur un pin marqué GND, et le pin de contrôle sur un pin d'entrée-sortie numérique (les pin avec seulement un numéro, parfois avec un ~ devant le numéro). J'ai choisi le port 10.

##### Alimentation de l'Arduino
Pour alimenter l'Arduino, on va récupérer une arrivée électrique d'une prise proche, et la faire arriver dans la prise non étanche qui sera dans la boite de dérivation. Sur cette prise, on va brancher l'arduino : une prise 230v - 5v USB (un chargeur de téléphone par exemple), qui sera reliée à l'Arduino par le câble USB-A mâle vers USB-B femelle inclus avec l'Arduino.

##### Alimentation du chauffe-lait
Attention, ça se corse.
On va récupérer l'arrivée électrique utilisée par la prise de l'Arduino. On va relier le neutre directement à la prise du chauffe lait, et la phase sur le port marqué COM du relais. Pour finir, on va relier le port marqué NO (Normally Open) à la phase de la prise du chauffe-lait.

##### C'est branché
Voila, tout est branché. Voici un schéma récapitulatif et une photo de ce que ça donne chez moi.

### La partie logicielle
Voici mon code, que vous pouvez adapter selon vos besoins

```cpp
//Chloedev5
//TempoLait Ver. 0.1
//23-01-2023
//
//Sonde de temperature étanche DS18B20
//relais 5V - 230V AC KY019

//Connexion - cablage
//Relais +5V / port de controle = numérique 10
//Sonde de température = +3.3 - 5V / port information (jaune) ANALOG IN A1

#include "OneWire.h"
#include "DallasTemperature.h"

OneWire oneWire(A1); //définition du port de la sonde
DallasTemperature ds(&oneWire);

int attente = 10; //temps d'attente entre 2 mesures en secondes
int temp_high = 52; //température du lait maxi (en °C)
int temp_low = 48; //temperature du lait mini (en °C)

int delai = attente * 1000; //passage du temps d'attente en millisecondes

void setup() {
  Serial.begin(9600);  // définition de l'ouverture du port série
  ds.begin();          // activation de la sonde
  pinMode(10, OUTPUT); // définition du port de controle du relais (10) en sortie
}

void loop() {
  ds.requestTemperatures(); //récupération de la température
  int t = ds.getTempCByIndex(0); //Sauvegarde de la température dans la variable t

  //affichage de la temperature dans la console
  Serial.print(t);
  Serial.println( "C");

  // réglage du relais en fonction de la température relevée
  if (t > temp_high) {
    digitalWrite(10, HIGH);  //si la température est superieure a temp_high = coupure du relais
  }
  else {
    if (t < temp_low) {
      digitalWrite(10, LOW); //si la température est inferieure a temp_low = remise en route du relais
    }
  }

  delay(delai); //pause de la valeur de la variable delai
}
```
