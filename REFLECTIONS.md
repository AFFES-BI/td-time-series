# Réflexions — TD4 Séries temporelles

## Stage 0 — Cadrage

**Réflexion 1.** Pour vérifier si des données historiques réelles de CO2 existaient,
j'aurais pu lire la documentation de l'API Open-Meteo ou faire un appel test avec
`past_days=90` et regarder ce qui revenait. Cela prend une minute. L'ancienne version
du TD avait supposé qu'il n'y avait pas d'historique et avait donc utilisé des données
fausses, alors que les vraies étaient disponibles. J'en retiens qu'une hypothèse sur
les données disponibles est elle-même une affirmation à tester avant de s'en servir
pour concevoir quoi que ce soit.

## Stage 1 — Collecte

**Réflexion 2 (prédiction, écrite avant de regarder les données).** Je m'attends à
ce que le CO2 au Mans soit en *hausse* sur les 90 derniers jours. Sur cette période,
la saison de croissance se termine : les feuilles tombent et les plantes absorbent
moins de CO2 par photosynthèse, donc le « puits » naturel de l'été s'affaiblit. En
parallèle, les émissions humaines augmentent après la pause estivale (la rentrée
scolaire et la reprise du travail font monter le trafic) et le chauffage démarre
quand les températures baissent. Ces deux effets poussent la concentration vers le
haut.

*Vérification après l'exécution :* la pente mesurée est de +0,160 ppm/jour, donc le
CO2 est bien en hausse sur la fenêtre du 11/08/2026 au 08/11/2026. Ma prédiction est
confirmée dans sa direction ; je n'ai pas isolé la part de chaque cause.

## Stage 2 — Nettoyage

**Réflexion 3.** Non, on ne peut pas supprimer le code de détection des trous. Zéro
trou signifie seulement que la source a été propre cette fois-ci, pas qu'elle le sera
toujours. Si un jour manque et que ce code n'existe plus, la moyenne glissante
mélangerait en silence des jours non consécutifs et personne ne le saurait. Ce code
coûte presque rien et protège le jour où la source changera ou cassera.

## Stage 3 — Modélisation

**Réflexion 4.** Dans mes données, la valeur brute monte brusquement à 470,6 ppm. Le
lissage exponentiel monte à 457,4 ce jour-là, contre 453,5 pour la moyenne glissante :
il réagit plus vite. Je le préférerais pour détecter tôt un changement brutal, par
exemple une alerte de pollution. Je préférerais la moyenne glissante pour suivre une
tendance lente et stable, car chaque jour de la fenêtre pèse pareil et un pic isolé
influence moins le résultat dans la durée.

## Stage 4 — Stockage

**Réflexion 5.** Si l'appel à l'API échoue les 4 fois, le job s'arrête à l'étape
`fetch_series.py` et les étapes suivantes (analyse et commit) ne s'exécutent pas.
`series.csv` et `series_analysis.json` gardent donc les valeurs de la semaine
précédente. Le dashboard ne casse pas, il devient périmé. Seule la date « Données
analysées le… » trahit ce retard, et rien ne l'affiche comme une alerte.

## Stage 5 — Visualisation

**Réflexion 6.** Je ferais confiance à cette projection sur 7 jours au maximum. Ma
pente est de +0,16 ppm/jour : sur une semaine la droite ne monte que d'environ 1 ppm.
Pourtant mes valeurs quotidiennes varient d'environ 3,7 ppm d'un jour à l'autre et
s'étalent de 429 à 471 ppm. Le bruit est donc plus grand que le mouvement de la
tendance, et la droite ne donne qu'un ordre de grandeur (autour de 450 ppm), pas une
valeur précise. Plus loin, le cycle saisonnier du CO2 peut changer la pente, et une
droite ne le voit pas. Contrairement à une appli météo, qui s'appuie sur des modèles
physiques corrigés en continu, ce dashboard ne fait que prolonger une droite. Je
n'irais pas au-delà de 7 jours, et même à 7 jours je la lirais comme une zone, pas
comme un point.

## Stage 6 — En autonomie

**Réflexion 7.** Je n'ai changé que les coordonnées (`LAT, LON`) pour passer du Mans
à Paris, et tout le pipeline a continué à fonctionner : nettoyage, lissage, projection
et dashboard. C'est possible parce que les fonctions d'`analyze_series.py`
(`clean_series`, `rolling_mean`, `exponential_smoothing`, `linear_trend_projection`)
ne mentionnent ni le CO2, ni un lieu, ni une unité. Elles ne manipulent que des dates
et des nombres : une liste de paires jour/valeur. Elles ne savent donc pas ce que ces
nombres représentent. Le lieu n'existe que dans `fetch_series.py`, et c'est là que je
l'ai changé. Il y a quand même une limite : si j'avais changé de variable (par exemple
`pm2_5`), il aurait fallu renommer la colonne `co2_ppm` lue par `load_series` et
l'unité « ppm », et adapter le texte de la note d'honnêteté. Ces détails sont
l'étiquette des données, pas leur traitement, et c'est pour cela qu'on peut changer de
lieu ou de variable sans toucher à la logique.

*Résultat de la comparaison (11/08 → 08/11/2026) :* Le Mans, pente +0,160 ppm/jour et
moyenne 442,5 ppm ; Paris, pente +0,379 ppm/jour et moyenne 453,3 ppm.
