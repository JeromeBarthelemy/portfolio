# Portfolio — Jérôme Barthélemy

Quatre projets, quatre problèmes différents. Le code des rendus École 42 reste privé
(règle de l'école) ; ce dépôt explique ce qui a été construit, pourquoi, et ce que
j'en ai tiré. Accès au code sur demande pendant un entretien.

---

## TAP — serveur de jeu multijoueur TCP
**Go · en binôme · juin 2026**

**Le problème.** Faire tenir plusieurs joueurs dans un même monde partagé, via un
protocole assez précisément spécifié pour que le client d'une autre équipe puisse se
connecter à notre serveur sans une ligne de code en commun.

**Ce que nous avons construit.** Un protocole texte ligne à ligne, rédigé en RFC avant
la première ligne de code, puis un serveur TCP et deux clients. Le serveur possède
l'état du monde ; les clients ne font que l'observer. Les événements — un joueur entre
dans la salle, un combat commence — sont poussés de façon asynchrone.

**Ce qui a compté.**
- Écrire la spécification avant le code : chaque désaccord d'implémentation se réglait en relisant la RFC, pas en débattant.
- `go test -race` sur toute la suite : la concurrence ne pardonne pas les tests optimistes.
- Journalisation JSON structurée (`log/slog`) et détection d'abus : un serveur qu'on ne peut pas observer est un serveur qu'on ne peut pas exploiter.

---

## Agent Smith — harnais d'agent LLM
**Python · à trois · août 2026**

**Le problème.** Faire résoudre à un modèle de langage de vraies tâches de programmation
— écrire une fonction (MBPP), corriger un bug dans un vrai dépôt (SWE-bench) — sans
qu'un plantage ni un dépassement de budget ne réduise le résultat à zéro.

**Ce que nous avons construit.** Une boucle Pensée → Code → Observation. Le modèle
n'émet pas d'appel d'outil JSON : il écrit un bloc de code Python, exécuté dans un
interpréteur restreint où les outils sont de simples fonctions. Il lit ensuite ce qui
s'est réellement passé.

**Ce qui a compté.**
- **Un plantage vaut zéro**, donc rien ne remonte d'exception : chaque chemin d'échec renvoie un résultat valide avec son erreur renseignée.
- **Chaque plafond est dur** : itérations, tokens en entrée et en sortie, temps réel. La boucle réserve de quoi tenter une dernière soumission plutôt que de mourir en pleine réflexion.
- Agnostique du fournisseur : un endpoint OpenAI-compatible absent de la config ne demande aucune modification de code, seulement une URL.

---

## RAG against the machine — recherche augmentée
**Python · en solo · juin 2026**

**Le problème.** Répondre à des questions sur la base de code vLLM 0.10.1 en citant
ses sources, avec un modèle assez petit pour tourner sur CPU.

**Ce que j'ai construit.** Un pipeline en quatre étapes — ingestion, recherche,
réponse, évaluation — derrière une seule CLI. Le corpus est découpé, indexé, puis
interrogé ; le modèle Qwen3-0.6B rédige la réponse à partir des seuls extraits retrouvés.

**Ce qui a compté.**
- La qualité de la recherche est une métrique, pas une impression : *recall@k* contre un jeu de référence, mesuré à chaque changement de stratégie de découpage.
- Séparer récupération et génération : quand la réponse est fausse, on sait laquelle des deux moitiés a échoué.
- 43 tests unitaires et mypy en mode strict sur un projet solo — c'est là que ça se relâche d'habitude.

---

## Inception — infrastructure conteneurisée
**Docker · en solo · juillet 2026**

**Le problème.** Monter une infrastructure web complète sans tirer une seule image
toute faite du Docker Hub : chaque conteneur construit à la main, un service par conteneur.

**Ce que j'ai construit.** WordPress avec php-fpm et MariaDB derrière un NGINX qui est
le seul point d'entrée, en TLS 1.2 et 1.3 uniquement. Puis cinq services en bonus :
cache objet Redis, client de base Adminer, monitoring Netdata, serveur FTP, site
statique de documentation. Huit images, toutes depuis `alpine`, jamais le tag `latest`.

**Ce qui a compté.**
- Les données survivent au conteneur : volumes nommés pour la base et les fichiers du site.
- Moindre privilège appliqué là où c'est tentant de tricher : le monitoring n'a ni privilèges, ni montage de l'hôte, ni accès au socket Docker — il observe la machine, il ne peut pas la piloter.
- Un seul port ouvert vers l'extérieur en dehors du FTP, qui n'est pas de l'HTTP et ne peut donc pas passer par le proxy.

---

📫 **jerbarth@gmail.com**
