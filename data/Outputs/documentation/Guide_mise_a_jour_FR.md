# Mise à jour de l’étude ETA

## Résultat principal
Le meilleur résultat descriptif est **Random Forest / Distribution train-selected : 79.14% dans ±30 secondes, MAE 24.51 s**. L’objectif actuel de 80% est **non atteint**. Le résultat contient 1635 prédictions dans la tolérance sur 2 066; il en faut 1653 pour atteindre 80%. Il manque 18 prédictions correctes. Le score n’est pas arrondi à 80%.

L’objectif initial était 85%; l’objectif actuel est 80%. La tolérance reste trente secondes et les observations évaluées restent identiques. R² n’est pas le pourcentage de prédictions correctes.

| Modèle | Observations | Dans ±30 s | MAE (s) | RMSE (s) | R² |
|---|---:|---:|---:|---:|---:|
| LightGBM | 2066 | 72.46% | 24.76 | 36.05 | 0.787 |
| Linear Regression | 2066 | 69.26% | 27.87 | 42.06 | 0.711 |
| Random Forest | 2066 | 79.14% | 24.51 | 35.48 | 0.794 |
| XGBoost | 2066 | 70.52% | 25.88 | 37.34 | 0.772 |

## Données ajoutées
Les exports du 27 octobre 2026 contiennent 14 500 lignes GPS pour 29 véhicules, avec exactement 500 lignes par véhicule. Le journal contient 106 arrivées pour l’unité 16 seulement. Le GPS U16 couvre 16:57:45–19:35:54. Onze arrivées ont un passage GPS apparié et donnent 162 observations ETA utilisables. Les autres bus n’ont pas de journal d’arrivée fourni.

Le jeu actualisé contient 2 066 observations et 88 arrivées : les 1 904 observations / 77 arrivées de juin, plus les 162 observations / 11 arrivées d’octobre. Les neuf groupes comprennent les sept tournées initiales et deux parties de tournées d’octobre. Ce ne sont pas neuf tournées complètes. Les identifiants 107/108 désignent les parties observées des tournées 7/8.

Les coordonnées du nouveau CSV sont correctement nommées. Les anciens en-têtes Excel latitude/longitude étaient inversés et sont traités séparément. Les originaux restent inchangés. Les copies CSV aux en-têtes anglais conservent chaque valeur brute.

## Ce qui a changé dans la méthode
Les variables de base restent disponibles et les 87 colonnes de juin ont été vérifiées sans changement. Des variables supplémentaires décrivent le dernier point de contrôle quitté, le temps écoulé et la distance parcourue depuis. Elles utilisent le GPS présent/passé sur un historique continu par véhicule. Changer le GPS futur ne modifie pas ces variables.

Les quatre modèles de départ sont conservés. Les réglages de la forêt et sa règle de prédiction sont choisis dans trois plis internes d’entraînement. Le choix vise le pourcentage des prédictions internes dans trente secondes, puis le MAE. Les observations de la tournée test ne servent pas à ces choix.

La forêt estime les temps plausibles à partir des feuilles des arbres et choisit un intervalle de soixante secondes. Plusieurs règles sont comparées sur entraînement : observations de la feuille, échantillons utilisés par l’arbre, milieu d’un intervalle également probable et léger lissage. Le lissage est un réglage du modèle, pas un changement de la tolérance d’évaluation.

Des corrections de temps, un mélange forêt/boosting, Extra Trees et une forêt de 1 200 arbres ont aussi été testés. Extra Trees est une cinquième méthode complémentaire; elle ne remplace aucun des quatre modèles demandés. Les résultats moins bons restent dans les tableaux. Les poids du mélange et toutes les corrections proviennent uniquement de prédictions internes des groupes d’entraînement.

## Effet des nouvelles données à test identique
Sur les mêmes 1 904 observations de juin et avec des réglages fixes, la forêt passe de **76.58%** à **77.36%** en ajoutant octobre à l’entraînement. Cette comparaison isole l’effet de l’ajout de données. Le résultat principal utilise aussi de nouvelles variables et règles de prédiction; il ne faut pas attribuer toute sa différence au seul ajout de lignes.

Le résultat historique 76,52% appartenait à la première version. Les effectifs et procédures diffèrent de ceux du résultat principal actualisé.

## Test sur la date suivante
Le premier test entraîne seulement sur juin et prédit les 162 observations d’octobre avec des réglages fixés à partir de l’étude précédente. La forêt y obtient **72.84%**, MAE 28.67 s. Ce score reste distinct de la validation par groupes sur les deux dates. Les méthodes ont ensuite été affinées sur ces données; une nouvelle journée indépendante est nécessaire pour confirmer les choix.

| Modèle | Observations | Dans ±30 s | MAE (s) | RMSE (s) | R² |
|---|---:|---:|---:|---:|---:|
| LightGBM | 162 | 71.60% | 24.31 | 32.39 | 0.829 |
| Linear Regression | 162 | 65.43% | 51.95 | 221.99 | -7.008 |
| Random Forest | 162 | 72.84% | 28.67 | 41.07 | 0.726 |
| XGBoost | 162 | 71.60% | 24.05 | 31.63 | 0.837 |

## Validation, références simples et sensibilité
Chaque groupe observé est laissé entièrement hors entraînement une fois. Toutes les observations d’une arrivée restent ensemble. Les tableaux distinguent les effectifs GPS et le nombre d’arrivées. Les groupes d’octobre sont petits et partiels; les moyennes par groupe et le score sur toutes les observations doivent être lus séparément.

fixed_reference_summary.csv présente les mêmes paramètres Default dans les neuf plis, avec moyenne, écart-type, minimum et maximum du MAE. La comparaison optimisée garde les mêmes variables pour les quatre modèles, mais peut sélectionner des paramètres différents dans les groupes d’entraînement. Les méthodes simples distance/vitesse et moyenne d’entraînement restent dans baseline_results.csv; la méthode à vitesse nulle n’est pas définie et sa couverture est indiquée.

La sensibilité réduit les événements d’entraînement de 100% à 10%, avec trois graines sous 100%, en conservant les tests. Elle compare les quatre modèles de base à réglages fixes. Elle ne mesure ni une baisse de fréquence GPS ni une économie d’énergie. Les modèles complémentaires restent des essais séparés.

## Hypothèses et limites
La correspondance GPS U16 / unité 16 est soutenue par onze passages appariés mais n’est pas confirmée par le fournisseur. La vitesse suit la convention précédente en km/h; l’unité du nouvel export n’est pas confirmée séparément. Aucun décalage d’horloge n’est appliqué. La documentation du système d’enregistrement des nouvelles arrivées et la précision de l’horloge sont inconnues. Des horaires identiques ne prouvent pas l’indépendance des systèmes.

Les limites de tournée et les fenêtres d’horizon réel >30 et ≤300 secondes sont construites après les trajets. La destination est donnée au modèle. L’historique GPS reste continu par véhicule pour éviter des réinitialisations fondées sur des arrivées futures; ce choix adapte la règle initiale de calcul par tournée et doit être expliqué. Il ne s’agit pas d’une application de suivi en direct.

La limite légale de vitesse est inconnue. Les seuils 20/30/40/50 km/h sont des scénarios, pas des infractions établies. Une journée supplémentaire partielle ne permet pas de garantir les résultats sur toute la flotte. La reconstruction de trajectoires manquantes, la satisfaction passager et l’énergie ne sont pas validées par ces données.

Le notebook et les cellules ajoutées ont été exécutés localement. Des calculs inchangés ont été réutilisés après vérification des sources et empreintes des données. L’exécution dans le cloud Colab n’a pas été testée. Le modèle sauvegardé est réentraîné sur toutes les données pour démonstration et n’a pas de nouveau score de test.

## Données à demander pour la suite
Demander le GPS complet de l’unité 16 pour toute la journée du 27 octobre, sans limite de 500 lignes, ainsi que les journaux d’arrivée des autres véhicules présents dans le GPS. Confirmer les unités de vitesse, la correspondance des véhicules, le fuseau horaire et le mode d’enregistrement des arrivées. Les géométries manquantes des autres points de contrôle doivent venir d’une source documentée.

## Fichiers et mise à jour du mémoire
Hanine_ETA_Bus_v2.ipynb contient toute la programmation. input/ contient les cinq fichiers originaux. Le dashboard HTML est autonome : le télécharger puis l’ouvrir dans Chrome ou Edge. Une prévisualisation de stockage en ligne peut ne pas exécuter ses graphiques.

model_summary.csv donne les expériences et scores globaux; model_summary_by_period.csv sépare juin et octobre; all_results_by_trip.csv détaille chaque groupe. primary_model_predictions.csv contient les prédictions du résultat principal. Les autres essais et décisions d’entraînement restent disponibles pour vérification. Le dashboard montre une sélection de comparaisons; il ne supprime aucune observation des méthodes affichées.

Les figures 01–03 distinguent test temporel, validation élargie et effet de données ajoutées. Les figures 04–08 détaillent la référence élargie, ses erreurs et la sensibilité. La figure 09 montre l’ajout d’historique; 10 les corrections; 11–12 le résultat principal. Chaque PNG et SVG représente la même figure.

Mettre à jour ensemble les données, le protocole, les tableaux, figures, discussion et conclusion du mémoire. Ne pas remplacer seulement un pourcentage. Le mémoire intégral n’est pas rédigé ici.

## Référence de méthode
Meinshausen, N. (2006), Quantile Regression Forests, JMLR 7(35), 983–999 : https://jmlr.org/papers/v7/meinshausen06a.html . Cette référence présente l’estimation d’une distribution avec les feuilles d’une forêt. La règle d’intervalle utilisée ici est une adaptation à la tolérance de l’étude, pas une reproduction de son prédicteur de quantiles.
