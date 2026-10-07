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
