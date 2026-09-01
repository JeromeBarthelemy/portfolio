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
**Python · à trois · août 2026 · [code public](https://github.com/Enelsep/agent-smith)** — 224 des 282 commits sont les miens

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
**Docker · en solo · juillet 2026 · [code public](https://github.com/JeromeBarthelemy/Inception)**

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

## Baguettechs — code du robot FTC, saison DECODE
**Java · équipe FIRST Tech Challenge 20989 · octobre 2025 – juin 2026 · [dépôt public](https://gitlab.com/ftc-civ/baguettechs/ftc-decode-2026)**

**Le problème.** Un robot de compétition FTC dispose de deux minutes trente par match,
dont trente secondes entièrement autonomes. Il doit se déplacer avec précision sur un
terrain qu'il ne voit qu'à travers ses capteurs, et lancer des projectiles vers une cible.
Aucune reprise possible : le code tourne le jour du match, devant un jury, ou pas du tout.

**Ce que j'ai construit.** Le pilotage en mecanum, en modes robot-centric et field-centric,
l'orientation par centrale inertielle, la correction de cap, le réglage PID moteur par
moteur, le contrôle du lanceur, et l'asservissement de sa vitesse de tir par la caméra
Limelight. 68 des 108 commits du dépôt sont les miens ; le reste est l'œuvre des élèves
de l'équipe, que j'encadre.

**Ce qui a compté.**
- **Le matériel ne se comporte jamais comme le modèle.** Un moteur ne tourne pas à la vitesse demandée, une centrale inertielle dérive, un servo ne revient pas exactement en position. La moitié du travail consiste à laisser des constantes réglables plutôt qu'à écrire du code plus élégant.
- **Le PID individuel par moteur a fini désactivé** : les quatre boucles se battaient entre elles et dégradaient la trajectoire. Un asservissement global de cap, plus simple, s'est révélé plus stable — la solution correcte n'était pas la plus sophistiquée.
- **Le code est lu et modifié par des lycéens.** Il doit rester compréhensible par quelqu'un qui apprend Java depuis six mois : c'est la contrainte de lisibilité la plus exigeante que j'aie rencontrée.

### Palmarès du club — Robotique CIV

Plus de 70 lycéens, quatre équipes, engagées au Royaume-Uni, en France, aux Pays-Bas et
aux États-Unis depuis 2019. Résultats vérifiables sur les pages officielles *FIRST*.

| Équipe | Résultats principaux |
|---|---|
| [BaguetTechs — FTC 20989](https://ftc-events.firstinspires.org/team/20989) | **Champion de France 2025** (capitaine de l'alliance gagnante, invaincue en 9 matchs) · Vainqueur du Défi Robotique de Valence 2025 · Finaliste France 2026 + Think Award · Inspire Award Londres 2022 · European Premier Event, Eindhoven |
| [FRITES — FTC 20991](https://ftc-events.firstinspires.org/team/20991) | **Champion de France 2026** (alliance gagnante) + Control Award · Finaliste Valence 2026 + Innovate Award · Vainqueur Londres 2022 |
| [TheFrenchineers — FTC 20990](https://ftc-events.firstinspires.org/team/20990) | **Inspire Award, championnat de France 2026** — la plus haute distinction du FIRST Tech Challenge · Alliance gagnante France 2025 |
| [Geekos — FRC 9220](https://frc-events.firstinspires.org/team/9220) | Finaliste régional 2026 + Gracious Professionalism Award · Rising All-Star Award, **New York City Regional 2025** · Imagery Award, **Long Island Regional 2024** |

Le championnat de France FTC n'existe que depuis 2025 : le club a remporté ses deux
éditions, avec deux équipes différentes.


---

📫 **jerbarth@gmail.com**

📫 **jerbarth@gmail.com** · [LinkedIn](https://www.linkedin.com/in/jerome-barthelemy/)
