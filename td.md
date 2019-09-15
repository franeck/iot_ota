# TD - Mises à jour OTA

Ce TD vous permettra de vous excercer dans la mise à jour à distance de vos programmes embarqués.

## Exercice 1

> Lien vers le cours

1.1 Créez un programme qui imprimera un "Hello World" dans le moniteur série

1.2 Implémentez les mises à jour OTA et faites toruner le programme sans être connecté au PC. Que constatez vous concernant les messages?

1.3 Pour restaurer la capacité de reçevoir les messages implémentez dans votre programme le RemoteDebug. Tout cela sans rebrancher la carte à l'ordinateur.

## Exercice 2

> https://github.com/ThingPulse/esp8266-weather-station/tree/master/examples/WeatherStationDemo

2.1 Reprenez le code de la station météo et implementez-y les mises à jour OTA 

2.2 Téléversez le sur la carte à l'aide de la connexion série

2.3 Modifiez le programme afin de ne laisser que l'heure dans l'affichage

2.4 Téléversez le sur la carte à l'aide de l'OTA

## Exercice 3 - pour les courageux

Téléversez sur la carte un programme basique avec les mises à jour OTA. Puis, dans la suite de l'exercice n'utilisez plus le câble pour se connecter au PC.

Endpoint API
> Pour le jour en cours
> http://vps729659.ovh.net/api/v1/events
> Pour un jour précis
> http://vps729659.ovh.net/api/v1/events?start=2019-09-16
> Pour une période
> http://vps729659.ovh.net/api/v1/events?start=2019-09-16&end=2019-09-17 

3.1 Grâce à l'API mise à votre disposition, recuperez les données de l'emploi du temps dans l'ESP.

3.2 Affichez les cours de la journée sur l'écran OLED.

3.3 Ajouter un defilement afin de visualiser les 3 prochain jours. Vous pouvez vous inspirer du programme de la station météo pour l'affichage.
