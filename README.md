             Introduction
Ce projet a pour objectif d’illustrer, dans un cadre légal et éthique (tests sur vos propres applications, appareils personnels, audits de sécurité autorisés), les techniques d’inspection du trafic HTTPS d’une application Android. Le but est de comprendre comment les développeurs protègent leurs communications avec l’SSL Pinning et comment un testeur peut le contourner pour analyser le trafic chiffré (ex. : recherche de vulnérabilités, validation de conformité).

            Principales étapes du projet :

Mise en place d’un proxy TLS (Burp Suite ou mitmproxy) sur le PC.

Installation d’un certificat CA sur l’appareil Android.

Configuration du proxy manuel sur l’Android.

Neutralisation de l’SSL Pinning via Frida (hooks Java : TrustManager, Conscrypt, OkHttp, WebView).

Si nécessaire, extension du bypass pour du pinning natif (BoringSSL, OpenSSL).

Validation par capture et inspection du trafic HTTPS déchiffré.

Prérequis :

PC Windows (PowerShell), macOS ou Linux.

Python 3.8+ et pip.

ADB (Android Platform Tools).

Frida côté PC et frida-server sur l’appareil (même version).

Android 8+ avec débogage USB activé.

Proxy TLS (Burp Suite ou mitmproxy).


              1. Capture : Résultat de recherche Google "test"
Cette capture montre une page de résultats Google pour le mot-clé test. Elle n’est pas directement liée au code ou à la manipulation technique, mais elle illustre typiquement le type de trafic qu’un testeur pourrait inspecter après avoir contourné l’SSL Pinning.

Éléments notables :

Résultats contenant des services de test en ligne : testweb.bsl.nl (plateforme de tests psychologiques/logopédiques), WebPageTest (outil de performance web), Typetuin (test de vitesse de frappe).

Un résultat web.prod.testwe.eu/login (interface d’authentification).

Utilité dans le projet :
Dans un scénario réel, l’application cible pourrait générer des requêtes vers ces domaines. Une fois le proxy actif et l’SSL Pinning contourné, le testeur peut voir en clair les requêtes HTTP/HTTPS échangées entre l’appareil et ces serveurs – y compris les paramètres, cookies, tokens, etc. Cette capture rappelle simplement la diversité des endpoints potentiels.

       2. Capture :Configuration du proxy manuel sur Android
Contexte :
Écran Android Wifi Manual (Paramètres → Réseau & Internet → Wi‑Fi → Modifier le réseau → Options avancées → Proxy). Cette étape est cruciale pour rediriger le trafic de l’appareil vers le proxy d’interception (Burp Suite, mitmproxy) tournant sur le PC.

Détails de l’interface :

Proxy hostname : 192.168.100.57 (l’adresse IP du PC sur le même réseau local que l’Android).

Proxy port : 8080 (port par défaut de Burp Suite / mitmproxy).

Bypass proxy for : example.com,mycomp.test.com,loc – permet d’exclure certains domaines du proxy (utile pour éviter des boucles ou pour tester sélectivement).

Boutons Cancel / Save : valider les réglages.

Remarque importante :
Le texte en haut précise : “The HTTP proxy is used by the browser but may not be used by the other apps.”
Cela signifie que le proxy manuel ne s’applique qu’aux applications respectant le proxy système (navigateur, certaines librairies réseau). De nombreuses apps (ex. : utilisant OkHttp avec configuration personnalisée) ignorent ce proxy. C’est pourquoi le contournement de l’SSL Pinning par Frida reste indispensable – même avec un proxy, le trafic reste chiffré si l’app ne se fie pas au certificat CA installé.

           3. Capture : Tentative d’injection Frida :
Contexte :
Commande exécutée dans un terminal Windows PowerShell :

powershell
PS C:\Users\setup game\Desktop> frida -U -f com.android.settings -l hello.js
Que signifie cette commande ?

frida -U : se connecte à l’appareil USB (ou émulateur) via Frida.

-f com.android.settings : lance (spawn) l’application com.android.settings (les paramètres système Android).

-l hello.js : charge un script JavaScript nommé hello.js (non montré, probablement un script simple de test).

Message d’erreur :

text
Failed to load script: the connection is closed
Thank you for using Frida!
Causes possibles :

frida-server n’est pas en cours d’exécution sur l’appareil Android.

Le serveur Frida n’est pas de la même version que le client Frida sur le PC.

Le débogage USB n’est pas correctement activé, ou l’appareil n’est pas reconnu par adb devices.

L’application com.android.settings ne permet pas le débogage ou Frida n’a pas les permissions suffisantes.





Ne déployez jamais ces méthodes en environnement réel sans supervision et sans destruction des certificats après test.

Après les tests : désinstallez le certificat CA de l’appareil, désactivez le proxy, et arrêtez frida-server.
