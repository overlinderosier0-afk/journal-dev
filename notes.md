# Notes quotidiennes

## 2026-09-22

Premier jour du journal ! L'idée : une courte note par jour pour garder le rythme — un concept appris, une astuce, un rappel. Petit pas quotidien, grosse discipline sur la durée.

## 2026-09-23

Ne mets jamais un token de session ou un secret dans une URL : il finit dans l'historique du navigateur, les logs du serveur et parfois les en-têtes Referer. Envoie-le plutôt dans le corps de la réponse (JSON) et stocke-le en cookie HttpOnly/Secure — c'est une faute qu'on retrouve encore sur de gros sites.

## 2026-09-24

Avant chaque git add ., prends le réflexe de faire un git status. Ça évite de commiter par accident un .env, un fichier de build ou une clé égarée — deux secondes qui épargnent bien des git reset et des secrets à révoquer.

## 2026-09-25

Si ton API Node lit process.env.DATABASE_URL mais crash sur « undefined », vérifie que dotenv est chargé AVANT tout le reste : import 'dotenv/config' doit être la première ligne du point d'entrée (main.ts) — et aussi des scripts comme seed.ts, sinon ils ignorent le .env. Attention aussi : dotenv n'écrase jamais les variables déjà exportées dans le shell — une vieille valeur exportée par erreur te fait déboguer un fantôme.
