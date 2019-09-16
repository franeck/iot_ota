# Mises à jour OTA avec ESP8266

La mise à jour "Over The Air" (OTA) permet aux objets connectés de mettre à jour leur software automatiquement à travers une connexion sans fil. Les améliorations et les corrections peuvent être appliquées à de multiples équipements sans effort.

Dans ce document vous allez voir comment mettre en place les mises à jour OTA dans le cas du module ESP8266.


## Prérequis

- IDE Arduino configuré pour l'ESP8266
- Un réseau wifi

## Création d'un programme

Commencez par créer un programme simple qui se connecte à un réseau WiFi:

``` C
#include <ESP8266WiFi.h>

const char* ssid = "";
const char* password = "";

void setup() {
    WiFi.begin(ssid, password);
    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
      delay(3000);
      ESP.restart();
    }
}

void loop() {
}
```

## Implémentation de l'OTA

Afin d'implementer l'OTA dans votre programme ajoutez les éléments suivants:

Bibliothèques :

```C
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
```

Une configuration dans le bloc `setup()` :

``` C
ArduinoOTA.setHostname("myEspModule");
ArduinoOTA.begin();
```

Une commande pour chercher et appliquer les modifications dans le bloc `loop()` :

``` C
ArduinoOTA.handle(); 
```

Le programme devrait ressembler à ceci :

``` C
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>

const char* ssid = "";
const char* password = "";

void setup() {
    WiFi.begin(ssid, password);
    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
      delay(3000);
      ESP.restart();
    }
    
    ArduinoOTA.setHostname("myEspModule");
    ArduinoOTA.begin();
}

void loop() {
    ArduinoOTA.handle(); 
}
```

## Premier téléversement

Avant de téléverser ce programme via la liaison série, vérifier dans le menu "Outils" que la "Flash size" sélectionnée correspond bien à votre module ESP8266. 

Pour un ESP8266 ESP-12, il s'agit de **4M (1M SPIFFS)**. 

Vous pouvez maintenant téléverser le programme puis débrancher l'ESP de votre PC.

## Première mise à jour OTA

Branchez l'ESP à une source d'alimentation externe.

Dans "Outils > Port" vous devriez voir une nouvelle option apparaître : 

![](https://i.imgur.com/h6Bayjy.png)

Si ce n'est pas le cas attendez une minute et reverifiez. Un redemarrage de l'IDE peut aider aussi.

#### Remarque
> Si malgré l'attente le port n'apparait pas, assurez vous que vous n'utilisiez pas une connexion WiFi dont la configuration pourrait bloquer des connexions à l'ESP.

Selectionnez le port réseau et téléversez votre programme. 
Bravo! Votre ESP supporte désormais les mises à jour OTA.

#### Remarque
> Si vous supprimez par inadvertance ArduinoOTA.begin() ou ArduinoOTA.handle(), vous serez obligé de reconnecter votre ESP8266 à l'ordinateur via la connexion série pour y envoyer un nouveau programme.


## Sécurisez votre connexion

En l'état, toute personne connectée à votre réseau WiFi pourrait aisément modifier le code exécuté sur votre ESP8266. Si vous le souhaitez, vous pouvez exiger qu'un mot de passe soit demandé lors des mises à jour.

Ajoutez l'instruction suivante dans le bloc `setup()` avant `ArduinoOTA.begin()` : 

```C
ArduinoOTA.setPassword("my_password");
```

Ainsi, la prochaine fois l'IDE Arduino vous demandera de saisir un mot de passe pour téléverser le programme.


## Debug


En étant deconnecté de la machine de développement vous perdez la possibilité d'observer le moniteur série. Ainsi les messages publiés par l'intérmediaire du `Serial.print();` ( bien utiles pour le debug ) ne sont plus visibles. Bien évidemment, une solution existe.

### Solution

Vous pouvez remplacer le moniteur série par défaut avec la bibliothèque RemoteDebug. Celle-ci communique avec le l'ESP à travers le protocole Telnet. 

#### Ajout de la bibliothèque

Téléchargez la dernière release en format zip : 

https://github.com/JoaoLopesF/RemoteDebug/releases

Ajoutez-la à votre projet :

`Croquis > Inclure une bibliothèque > Ajouter la bibliothèque .ZIP ...`

#### Modification du programme

Ajoutez la ligne suivante :
``` C
#include <RemoteDebug.h>
```

Definissez la variable suivante **avant** le bloc `setup()`  :
``` C
RemoteDebug Debug;
```

Initialisez la bibliothèque **dans** le bloc `setup()` :
``` C
Debug.begin("myEspModule");
```


Ajoutez la commande d'envoi dans le bloc `loop()` :
``` C
Debug.handle();
```

Votre code devrait désormais ressembler à ceci :
``` C
#include <ESP8266WiFi.h>
#include <WiFiUdp.h>
#include <ArduinoOTA.h>
#include <RemoteDebug.h>
    
const char* ssid = "";
const char* password = "";
    
RemoteDebug Debug;
    
void setup() {
    WiFi.begin(ssid, password);    
    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
        delay(3000);
        ESP.restart();
    }

    ArduinoOTA.setHostname("myEspModule"); 
    ArduinoOTA.setPassword("my_password");
    ArduinoOTA.begin();
    
    Debug.begin("myEspModule");
}

void loop() {    
    Debug.println("Hello World!");
    delay(1000);
    
    ArduinoOTA.handle();   
    Debug.handle();
}
```

Sauvegardez et téléversez votre programme.

#### Connexion

Vérifiez que votre ESP communique bien. Depuis un terminal d'une machine connectée au même réseau, connectez-vous à l'ESP : 

``` Bash
$ telnet your_ip
```

Cette commande devrait normalement afficher :
``` Bash
Trying your_ip...
Connected to esp0abe36.home.
Escape character is '^]'.
*** Remote debug - over telnet - for ESP8266 (NodeMCU)
* Host name: myEspModule (your_ip)
* Free Heap RAM: 40696
******************************************************
* Commands:
    ? or help -> display these help of commands
    q -> quit (close this connection)
    m -> display memory available
    v -> set debug level to verbose
    d -> set debug level to debug
    i -> set debug level to info
    w -> set debug level to warning
    e -> set debug level to errors
    l -> show debug level
    t -> show time (millis)
    p -> profiler - show time between actual and last message (in millis)

* Please type the command and press enter to execute.(? or h for this help)
***
```

Si tout va bien, les messages de l'ESP apparaîtront dans votre terminal.

## Conclusion

Dans ce document vous avez appris comment configurer un ESP8266 pour reçcevoir des mises à jour OTA, ainsi que reçevoir les messages du debug à travers le Telnet.
