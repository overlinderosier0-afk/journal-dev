# Notes quotidiennes

## 2026-09-22

Premier jour du journal ! L'idée : une courte note par jour pour garder le rythme — un concept appris, une astuce, un rappel. Petit pas quotidien, grosse discipline sur la durée.

## 2026-09-23

Ne mets jamais un token de session ou un secret dans une URL : il finit dans l'historique du navigateur, les logs du serveur et parfois les en-têtes Referer. Envoie-le plutôt dans le corps de la réponse (JSON) et stocke-le en cookie HttpOnly/Secure — c'est une faute qu'on retrouve encore sur de gros sites.

## 2026-09-24

Avant chaque git add ., prends le réflexe de faire un git status. Ça évite de commiter par accident un .env, un fichier de build ou une clé égarée — deux secondes qui épargnent bien des git reset et des secrets à révoquer.

## 2026-09-25

Si ton API Node lit process.env.DATABASE_URL mais crash sur « undefined », vérifie que dotenv est chargé AVANT tout le reste : import 'dotenv/config' doit être la première ligne du point d'entrée (main.ts) — et aussi des scripts comme seed.ts, sinon ils ignorent le .env. Attention aussi : dotenv n'écrase jamais les variables déjà exportées dans le shell — une vieille valeur exportée par erreur te fait déboguer un fantôme.

## 2026-09-26

Sur ESP32, si ton programme redémarre en boucle dès que tu actives le WiFi ou un moteur, c'est presque toujours le brownout : l'USB du PC ne fournit pas assez de courant. Un condensateur de 470 µF soudé au plus près des broches 5V/GND, ou une vraie alim 5V/2A, règle le problème neuf fois sur dix.

## 2026-09-27

Sur ESP32, évite les delay() dans la loop() : chaque pause bloque tout le reste (capteurs, affichage, WiFi). Préfère un timer avec millis() : compare l'heure actuelle au dernier déclenchement et agis quand l'intervalle est écoulé. Le code reste réactif et tu peux gérer plusieurs tâches « en parallèle » sans RTOS.

## 2026-09-29

En production, sauvegarde ta base de données dès le premier jour, pas « quand le site marchera ». Un pg_dump chaque nuit en cron + une copie hors du serveur, c'est dix minutes à mettre en place — et ça sauve le projet le jour où un disque rend l'âme ou une migration tourne mal.

## 2026-09-30

Si ton app utilise Firebase, ne laisse jamais les règles par défaut en prod : « allow read, write: if true; » transforme ta base en base publique, et les bots la scannent en permanence. Écris des règles qui vérifient request.auth (et les champs exacts), puis teste-les avec le simulateur de règles avant de déployer. Une base ouverte, c'est la fuite la plus bête qui existe.

## 2026-10-01

En bug bounty, la recon passe avant l'attaque : cartographier la surface d'attaque (sous-domaines, endpoints, technologies) avec des moyens passifs avant de toucher à quoi que ce soit. La plupart des bons rapports naissent d'une bonne compréhension de l'appli, pas d'un scanner lancé à l'aveugle. Une heure de recon méthodique vaut mieux que trois heures de fuzzing au hasard.

## 2026-10-02

Sur ESP32, passer la puce en deep sleep entre deux mesures au lieu de la laisser tourner en boucle peut diviser la consommation par ~100. Avec esp_sleep_enable_timer_wakeup(), la puce se réveille toute seule à intervalle régulier — parfait pour un capteur sur batterie. Attention : en deep sleep, la RAM est coupée, donc il faut sauvegarder les données importantes en RTC memory ou en flash avant de dormir.

## 2026-10-03

Petit rappel git : préfère des commits petits et atomiques — un changement = un commit, avec un message qui dit ce que le commit change. « Corrige la validation du formulaire » plutôt que « fix stuff ». Un historique propre rend le débogage et le revert d'un bug beaucoup plus faciles.

## 2026-10-04

En bug bounty, note toujours les en-têtes de sécurité HTTP manquants : Content-Security-Policy absent ou trop permissif, pas de X-Frame-Options, HSTS absent. Seuls, ces findings rapportent peu, mais combinés à une XSS ou un clickjacking sur une page sensible, ils transforment un refus en bounty. Vérifie-les pendant la recon passive, ça coûte 30 secondes avec curl -I.

## 2026-10-05

En React, pense toujours à nettoyer ton useEffect : le return du hook sert à couper les intervalles, les abonnements et les requêtes en cours. Sans ça, un composant démonté continue de tourner en tâche de fond et peut appeler l'API en boucle. Un intervalle oublié dans un useEffect, c'est une fuite mémoire garantie. Réflexe simple : à chaque effet qui « ouvre » quelque chose, écris tout de suite le code qui le « ferme ».

## 2026-10-06

Une IDOR, c'est quand une API renvoie les données d'un autre utilisateur juste parce qu'on a changé un ID dans l'URL. En bug bounty, le réflexe à tester en premier : créer deux comptes, puis rejouer chaque requête sensible avec l'ID du second. Une requête authentifiée n'est pas une requête autorisée — ne jamais supposer que le backend vérifie.

## 2026-10-07

Watchdog timer sur ESP32 : si ton code bloque (boucle infinie, Wi-Fi qui ne se connecte jamais), le Task Watchdog redémarre la puce automatiquement. Pense à nourrir le watchdog avec esp_task_wdt_reset() dans les boucles longues, et à configurer son timeout (CONFIG_ESP_TASK_WDT_TIMEOUT_S) selon ton pire cas d'exécution — un plantage silencieux au déploiement coûte bien plus cher qu'un redémarrage propre.

## 2026-10-08

git bisect, c'est le détective de git : tu lui dis qu'un commit est bon et un autre mauvais, et il te fait tester les commits intermédiaires un par un jusqu'à trouver exactement celui qui a introduit le bug. La prochaine fois qu'une régression arrive dans TiketHaiti, essaie « git bisect start » + « git bisect bad HEAD » + « git bisect good <commit> » — ça bat de loin la recherche manuelle.

## 2026-10-09

Sur Arduino, une interruption (attachInterrupt) réagit à un événement matériel sans passer par loop() : parfait pour compter les impulsions d'un capteur ou réveiller le microcontrôleur d'un bouton. Garde le handler court — mets juste à jour une variable volatile, et fais le vrai traitement dans loop(). Un compteur volatil + une interruption propre vaut mieux qu'un polling rapide qui rate des événements.

## 2026-10-10

En Firebase, les règles de sécurité Firestore décident qui lit et écrit quoi — et en mode test ouvert, tout le monde peut tout lire. Pour un projet comme Universite Pro, une règle allow read, write: if request.auth != null && request.auth.uid == userId garantit que chaque étudiant ne touche qu'à ses propres données. Réflexe simple : n'ouvre que le strict nécessaire, et teste tes règles dans la console Firebase avant de déployer.

## 2026-10-11

Un endpoint qui ne répond jamais n'est pas forcément mort : un WAF peut *tarpiter* (ralentir volontairement) les requêtes suspectes pour décourager les scanners. Avant de classer une cible « down », reteste avec des requêtes propres, espacées, et depuis ton navigateur normal. Ça évite de rater une surface qui se défend juste bien.

## 2026-10-12

Les clés d'API n'ont rien à faire dans le dépôt git. Une clé Resend, une clé privée Firebase ou un token GitHub doit vivre dans des variables d'environnement (un .env local, jamais commité) ou dans les secrets du service qui l'utilise.
Avant chaque commit, un réflexe simple : git diff, puis un grep sur les préfixes sensibles (sk_, xoxp-, ghp_) pour repérer ce qui a fuité. Si un secret a déjà fuité, le révoquer et le remplacer, pas juste le retirer du fichier.

## 2026-10-13

Les promesses JavaScript non gérées : un await sans try/catch, ou une promesse sans .catch(), et Node lève un unhandledRejection qui peut planter tout le process.
En dev ça passe inaperçu, en prod ça réveille à 3h du matin.
L'habitude à prendre : toujours try/catch autour des await, et un process.on('unhandledRejection') en filet de sécurité pour logger l'erreur proprement.

## 2026-10-14

Sur ESP32, toutes les GPIO ne sont pas équivalentes au démarrage : les « strapping pins » (GPIO0, 2, 5, 12, 15...) sont lues par la puce au boot pour choisir son mode. GPIO12 par exemple doit rester au niveau bas : si un capteur la tire au haut pendant le reset, la puce peut refuser de démarrer. Réflexe avant de câbler un capteur : vérifier le rôle de la broche au boot, et privilégier des GPIO « sûres » comme 13, 14, 25, 26 ou 27.

## 2026-10-15

En bug bounty, quand un endpoint répond 429 (Too Many Requests), c'est un signal : la cible a un rate limit en place. Le bon réflexe : espacer les requêtes, respecter le header Retry-After, et ne jamais contourner en forçant le rythme. Forcer, c'est risquer un blocage de compte — ou pire, un mini-DoS, ce qui est hors-scope de quasiment tous les programmes.

## 2026-10-16

git reflog, c'est le filet de sécurité de git : même un commit « perdu » après un reset --hard ou un rebase reste visible dans le journal des refs. Un git reflog, puis git checkout ou git reset sur le bon SHA, et le travail « effacé » revient. Réflexe simple : avant de paniquer après une fausse manip, regarde le reflog — git n'oublie presque jamais rien.

## 2026-10-17

Sur l'ESP32, un servo qui tourne et la carte qui redémarre en boucle : c'est souvent le brownout. Le servo tire trop de courant en pointe et la tension s'effondre. La bonne pratique : alimenter les servos avec une alim 5 V séparée, relier les masses (GND) en commun avec l'ESP32, et ajouter un gros condensateur (470–1000 µF) aux bornes de l'alimentation pour lisser les pics. Un seul test à la fois, et les hypothèses restent des hypothèses.

## 2026-10-18

En audit de code, chaque bug que tu trouves mérite sa règle Semgrep maison : le pattern devient un test de régression automatique que tu relances sur tout le codebase et sur tes futurs audits. Commence par les règles simples (mode 'search' sur une API dangereuse), puis passe au taint mode pour suivre le flux des données. Une règle qui détecte le bug hier te fera gagner des heures demain.

## 2026-10-19

Avec Next.js, tout ce qui commence par NEXT_PUBLIC_ finit dans le bundle JavaScript envoyé au navigateur. Ne jamais y mettre une clé Resend, une clé Firebase Admin ou un token privé : tout le monde peut les lire avec les devtools. Les secrets vivent dans des variables serveur (sans préfixe), lues côté API routes ou server components uniquement. Un grep rapide sur « NEXT_PUBLIC_ » avant chaque mise en prod vaut son pesant d'or.

## 2026-10-20

Quand un écran OLED reste noir sur l'ESP32, premier réflexe : un scan I2C. Balayer les adresses de 0x01 à 0x7F et afficher celles qui répondent sur le port série prend une minute, et tranche net entre « mauvais câblage SDA/SCL » et « écran mort ». Astuce : note l'adresse détectée (souvent 0x3C ou 0x3D) au lieu de la deviner dans le code.

## 2026-10-21

Une des premières vérifs en recon bug bounty : les sous-domaines qui pointent vers des services externes abandonnés (S3, Heroku, GitHub Pages...). Si le CNAME DNS existe encore mais que le service est supprimé, n'importe qui peut réclamer le nom et prendre le contrôle du sous-domaine. dnsx + une liste de signatures connues pour scanner, et toujours vérifier à la main avant de rapporter — les faux positifs sont fréquents.

## 2026-10-22

Une entrée GPIO qui n'est branchée à rien ne vaut ni 0 ni 1 : elle flotte et capte les parasites autour d'elle.
Sur ESP32 comme sur Arduino, les boutons-poussoirs se déclarent en INPUT_PULLUP (pas INPUT), et on lit 0 quand le bouton est pressé, 1 quand il est relâché.
Ça évite les lectures fantômes qui rendent un joystick ou un bouton complètement imprévisible au débogage.

## 2026-10-23

Note : piloter un servo (MG90S) depuis un ESP32, le bon câblage. Ne jamais alimenter le servo sur le 3,3 V de la carte : en pointe il tire plusieurs centaines de mA et la carte reboot (brownout). Aliment externe 5 V dédiée, masse commune avec l'ESP32, et signal PWM 50 Hz sur la broche : impulsion 1 ms = 0°, 1,5 ms = 90°, 2 ms = 180°. Avec ledcWrite, la même constante de fréquence partout.
