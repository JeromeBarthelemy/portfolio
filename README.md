# Portfolio — Jérôme Barthélemy

Huit projets, huit problèmes différents. Ce dépôt explique ce qui a été construit, pourquoi,
et ce que j'en ai tiré. Six des huit pointent vers du code public — il ne contient que mon
propre travail : sujets, moulinettes et supports pédagogiques appartiennent à l'École 42 et
n'y figurent pas.

---

## TAP — serveur de jeu multijoueur TCP
**Go · en binôme · juin 2026 · [code public](https://github.com/JeromeBarthelemy/tap-server)** — 118 des 145 commits sont les miens

**Le problème.** Faire tenir plusieurs joueurs dans un même monde partagé, via un
protocole assez précisément spécifié pour que le client d'une autre équipe puisse se
connecter à notre serveur sans une ligne de code en commun.

![Deux clients TAP connectés au même serveur : les événements d'un joueur apparaissent chez l'autre](assets/tap-demo.gif)

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

**Mesuré.** Sur MBPP, **233 tâches réussies sur 257** avec `mistral-medium-latest`, contre
205 sur 257 avec `codestral-2508` — le harnais est le même, seul le modèle change. Sur une
série de dix tâches passée sous les plafonds réels, la boucle consomme **5 itérations sur
les 10 autorisées** et **19,3 secondes sur les 120** : la marge est mesurée, pas espérée.

---

## Call Me Maybe — appel de fonctions par un petit modèle local
**Python · en solo · avril 2026 · [code public](https://github.com/JeromeBarthelemy/CallMeMaybe)**

**Le problème.** Traduire une requête en langage naturel — « quelle est la somme de 2 et
3 ? » — non pas en réponse, mais en **appel de fonction structuré** : le nom de la fonction
à invoquer et ses arguments typés, en JSON valide. Le tout avec Qwen3-0.6B, un modèle de
600 millions de paramètres tournant en local.

**La difficulté.** Un modèle de cette taille ne produit du JSON valide qu'environ une fois
sur trois quand on se contente de le lui demander. Le prompt ne suffit pas : il faut
contraindre la génération.

**Ce que j'ai construit.** Un décodage sous contrainte, qui guide le modèle jeton par jeton
pour garantir à chaque étape une sortie structurellement **et** sémantiquement valide — le
nom de fonction ne peut être qu'un nom existant, un argument numérique ne peut être qu'un
nombre. La fiabilité passe d'environ 30 % à la quasi-perfection.

**Ce qui a compté.**
- **Contraindre plutôt que demander.** Espérer d'un modèle qu'il respecte un format est une stratégie ; l'empêcher de produire autre chose en est une autre. Seule la seconde donne des garanties.
- Un modèle de 0,6 milliard de paramètres tournant sur CPU rend le problème intéressant : sans marge de manœuvre, la structure doit venir du code, pas de la puissance du modèle.
- C'est le même déplacement que dans Agent Smith : ne jamais faire confiance à la sortie du modèle, toujours la contraindre puis la vérifier.

**Mesuré.** **100 % de JSON analysable** — le décodage sous contrainte le garantit par
construction, contre environ 30 % en sollicitant naïvement le modèle. **Plus de 90 % de
sélections de fonction et d'extractions d'arguments correctes.** L'ensemble des prompts est
traité en moins de cinq minutes sur une machine ordinaire, le goulot d'étranglement étant
l'absence de cache KV dans le SDK.

---

## RAG against the machine — recherche augmentée
**Python · en solo · juin 2026 · [code public](https://github.com/JeromeBarthelemy/RAG)**

**Le problème.** Répondre à des questions sur la base de code vLLM 0.10.1 en citant
ses sources, avec un modèle assez petit pour tourner sur CPU.

**Ce que j'ai construit.** Un pipeline en quatre étapes — ingestion, recherche,
réponse, évaluation — derrière une seule CLI. Le corpus est découpé, indexé, puis
interrogé ; le modèle Qwen3-0.6B rédige la réponse à partir des seuls extraits retrouvés.

**Ce qui a compté.**
- La qualité de la recherche est une métrique, pas une impression : *recall@k* contre un jeu de référence, mesuré à chaque changement de stratégie de découpage.
- Séparer récupération et génération : quand la réponse est fausse, on sait laquelle des deux moitiés a échoué.
- 43 tests unitaires et mypy en mode strict sur un projet solo — c'est là que ça se relâche d'habitude.

**Mesuré.** Recherche lexicale BM25 sur environ 14 000 fragments. Deux décisions, chacune
validée par la mesure : découper les identifiants (`fused_batched_moe` → `fused batched moe`)
fait passer le **Recall@5 du code de 55 % à 65 %** ; router chaque question vers un index
dédié plutôt qu'un index unique fait passer celui de la documentation **de 83 % à 91 %**.
Une troisième piste — fusionner les deux index — a été **rejetée après mesure**, les scores
n'étant pas comparables d'un corpus à l'autre. La couverture du jeu de référence est de
100 %, donc le découpage n'est jamais le facteur limitant : seul le classement compte.

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

## Fly-in — routage de flotte sur graphe contraint
**Python · en binôme · avril 2026**

![25 drones routés sur trois corridors parallèles à travers un graphe de 52 zones](assets/flyin-demo.gif)

**Le problème.** Amener une flotte de drones d'une zone de départ à une zone d'arrivée
**en un minimum de tours**, sur un graphe où chaque zone a une capacité, chaque liaison
un débit, et où certaines zones coûtent deux tours à traverser ou sont interdites.

**Ce que nous avons construit.** Un flot de coût minimal sur un **graphe étendu dans le
temps** : chaque zone est dédoublée en entrée et sortie, à chaque tour, et l'horizon
s'allonge d'une couche tant que la flotte n'est pas écoulée. Les types de zone deviennent
des coûts d'arête, les capacités des bornes de flot. Un visualiseur rejoue la simulation
tour par tour, avec zoom et navigation avant/arrière.

**Ce qui a compté.**
- **L'optimalité se vérifie, elle ne se suppose pas.** J'ai écrit un oracle indépendant — un flot maximal sur graphe temporel, sans partager une ligne avec le solveur — qui calcule le plus petit horizon réalisable. Sur huit cartes allant de 4 à 52 zones, les deux coïncident exactement : 4, 3, 4, 7, 6, 13, 16 et 43 tours.
- **Un oracle qui contredit le code doit d'abord être suspecté.** Le mien annonçait 9 tours là où le solveur en donnait 16 ; après enquête, l'erreur venait de l'oracle, qui ignorait la capacité par défaut de 1 imposée par la spécification. Le désaccord venait du test, pas du testé.
- **Mesurer le parallélisme demande la bonne métrique.** Compter les zones occupées ou les chemins disjoints ne dit rien : seule la reconstruction de la route de chaque drone, puis le comptage des routes distinctes, répond à la question.

*Code non publié : le dépôt contient le sujet du projet, propriété de l'École 42.*

---


## Pac-Man — machine à états et travail en binôme
**Python · Pygame · en binôme · mai 2026**

![Machine à états du jeu : MainMenu, Playing, Paused, GameOver, Victory, HighscoreEntry, LeaderBoard, avec les transitions et leurs conditions](assets/pacman-state-machine.png)

**Le problème.** Un jeu d'arcade n'est pas surtout un problème d'affichage : c'est un
problème d'états. Menu, partie en cours, pause, mort, victoire, saisie du meilleur score,
tableau des scores — chacun avec ses transitions, ses conditions de sortie, et ce qu'il
faut préserver en passant de l'un à l'autre.

**Ce que nous avons construit.** Une machine à états explicite, portée par `App`.
`GameEngine` n'existe que pendant l'état `PLAYING` et émet à chaque image des événements
de transition qu'`App` consomme ; `ScreenManager` et `GameEngine` sont frères, jamais
imbriqués. Le labyrinthe est fourni par `mazegen`, le paquet issu de mon projet A-Maze-ing,
installé comme dépendance locale.

**Ce qui a compté.**
- **Le schéma ci-dessus est généré, pas dessiné.** Il vient de sources Graphviz versionnées à côté du code : quand la machine à états change, le schéma suit. Une documentation qui se régénère est une documentation qui reste vraie.
- **Une branche par auteur, et des pull requests entre nous.** 54 des 91 commits sont les miens ; le reste est celui de mon binôme, et chaque intégration est passée par une revue. Le dépôt conserve les tickets, les revues et le tableau Kanban.
- **Séparer les états du moteur a payé tard.** Ajouter la pause, la saisie de score et le tableau des scores n'a demandé aucune modification du moteur de jeu — seulement de nouveaux états et de nouvelles transitions.

*Code disponible sur demande : le dépôt contient des ressources graphiques et sonores issues
du jeu d'arcade original, qui ne m'appartiennent pas.*

---


## Baguettechs — code du robot FTC, saison DECODE
**Java · équipe FIRST Tech Challenge 20989 · octobre 2025 – juin 2026 · [dépôt public](https://gitlab.com/ftc-civ/baguettechs/ftc-decode-2026)**

[![La finale du championnat de France 2025](https://img.youtube.com/vi/-FN2Mel6wsQ/maxresdefault.jpg)](https://youtu.be/-FN2Mel6wsQ)

*La finale du championnat de France 2025, sur la chaîne officielle Robotique FIRST France.*

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
