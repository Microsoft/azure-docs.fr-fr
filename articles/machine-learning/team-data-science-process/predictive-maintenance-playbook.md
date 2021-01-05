---
title: Guide Azure AI pour les solutions de maintenance prédictive - Team Data Science Process
description: Une description complète de la science des données qui alimente les solutions de maintenance prédictive dans plusieurs secteurs d’activité.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: previous-author=fboylu, previous-ms.author=fboylu
ms.openlocfilehash: 1661f0f6cf024fde48d3706a1f8e47bf65f0d46f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321972"
---
# <a name="azure-ai-guide-for-predictive-maintenance-solutions"></a>Guide Azure AI pour les solutions de maintenance prédictive

## <a name="summary"></a>Résumé

La maintenance prédictive ( **PdM** ) est une application d’analyse prédictive populaire qui peut aider les entreprises de nombreux secteurs à utiliser les ressources de manière intensive et à réduire les coûts d’exploitation. Ce guide réunit les recommandations et les meilleures pratiques analytiques et d’entreprise pour développer et déployer des solutions de PdM à l’aide de la technologie de la [plateforme Microsoft Azure AI](https://azure.microsoft.com/overview/ai-platform).

Pour commencer, ce guide présente des scénarios d’entreprise spécifiques au secteur et le processus de qualification de ces derniers pour la PdM. Les données requises et les techniques de modélisation pour créer des solutions de PdM sont également fournies. Le contenu principal du guide concerne le processus de science des données,  y compris les étapes de préparation des données, l’ingénierie de fonctionnalité, la création du modèle et son opérationnalisation. Pour compléter ces concepts clés, ce guide répertorie un ensemble de modèles de solutions vous permettant d’accélérer le développement d’applications de PdM. Le guide renvoie également vers des ressources de formation utiles pour que le praticien en sache plus sur l’IA derrière la science des données. 

### <a name="data-science-guide-overview-and-target-audience"></a>Vue d’ensemble du guide de science des données et public cible
La première moitié de ce guide décrit des problèmes classiques des entreprises, les avantages de la PdM pour résoudre ces problèmes, et répertorie certains cas d’usage courants. Ce contenu est particulièrement intéressant pour les décideurs commerciaux (BDM). La seconde moitié explique la science des données derrière la PdM et fournit une liste de solutions de PdM basées sur les principes énoncés dans ce guide. Il fournit également des chemins d’accès et des pointeurs vers des supports de formation. Les décideurs techniques (TDM) trouveront ce contenu utile.

| Prise en main... | Si vous êtes... |
|:---------------|:---------------|
| [Analyse de rentabilité de la maintenance prédictive](#business-case-for-predictive-maintenance) |un décideur (BDM)  cherche à réduire les temps d’arrêt et les coûts d’exploitation ainsi qu’à améliorer l’utilisation de l’équipement |
| [Science des données pour la maintenance prédictive](#data-science-for-predictive-maintenance) |un décideur technique (TDM) évalue les technologies de PdM pour comprendre les exigences uniques en matière de traitement des données et d’IA pour la maintenance prédictive |
| [Modèles de solutions pour la maintenance prédictive](#solution-templates-for-predictive-maintenance)|un architecte logiciel ou un développeur d’IA  cherchent à mettre rapidement en place une démonstration ou une preuve de concept |
| [Ressources de formation pour la maintenance prédictive](#training-resources-for-predictive-maintenance) | s’adresse à l’ensemble des acteurs ci-dessus qui souhaitent apprendre les notions de base derrière la science des données, les outils et les techniques.

### <a name="prerequisite-knowledge"></a>Connaissances requises
Le contenu BDM ne part pas du principe que le lecteur dispose de connaissances préalables en science des données. Pour le contenu TDM, des connaissances de base en statistique et en science des données sont utiles. Des connaissances sont recommandées dans les domaines suivants : services Azure Data et AI, Python, R, XML et JSON. Les techniques d’IA sont implémentées dans les packages Python et R. Les modèles de solution sont implémentés à l’aide de services Azure, d’outils de développement et de SDK.

## <a name="business-case-for-predictive-maintenance"></a>Analyse de rentabilité de la maintenance prédictive

Les entreprises ont besoin d’équipements critiques pour fonctionner avec un maximum d’efficacité et d’utilisation afin de réaliser leur retour sur les investissements en capitaux. Ces équipements peuvent être des moteurs d’avions, des turbines, des ascenseurs ou des refroidisseurs industriels, qui coûtent des millions, ou des appareils courants tels que les photocopieurs, les machines à café ou les refroidisseurs d’eau.
- Par défaut, certaines entreprises font confiance à la _maintenance corrective_ , où les pièces sont remplacées lorsqu’elles sont défectueuses. La maintenance corrective garantit une utilisation complète des pièces (sans perte de durée de vie du composant), mais génère pour l’entreprise des interruptions du fonctionnement et du travail ainsi que des maintenances nécessaires non programmées (heures d’arrêt ou emplacements inopportuns).
- Au niveau suivant, les entreprises pratiquent la _maintenance préventive_ en déterminant la durée de vie utile d’une pièce et en l’entretenant ou en la remplaçant avant qu’une défaillance ne survienne. La maintenance préventive permet d’éviter des défaillances non programmées et catastrophiques. Mais les coûts élevés des temps d’arrêt planifiée, de la sous-exploitation du composant pendant sa durée de vie utile et de la main-d’œuvre demeurent.
- L’objectif de la _maintenance prédictive_ est d’optimiser l’équilibre entre les maintenances corrective et prédictive en permettant un remplacement _juste à temps_ des composants. Cette approche ne prévoit le remplacement de ces composants que juste avant leur défaillance. En étendant la durée des composants (par rapport à la maintenance préventive) et en réduisant la maintenance non programmée et les coûts du travail (en plus de la maintenance corrective), les entreprises peuvent réaliser des économies et avoir des avantages concurrentiels.

## <a name="business-problems-in-pdm"></a>Problèmes des entreprises liés à la PdM
Les entreprises sont confrontées à un risque opérationnel élevé en raison de défaillances inattendues et ont des connaissances limitées de la cause racine des problèmes dans les systèmes complexes. Voici quelques-unes des principales préoccupations des entreprises :

- Détecter les anomalies de l’équipement ou bien des performances ou de la fonctionnalité du système.
- Prédire si un équipement pourrait présenter une défaillance dans un futur proche.
- Estimer la durée de vie utile résiduelle d’un équipement.
- Identifier les principales causes de défaillance d’un équipement.
- Identifier les actions de maintenance à effectuer sur un équipement et la date d’exécution.

Les objectifs de la PdM sont généralement les suivants :

- Réduire les risques opérationnels des équipements stratégiques.
- Augmenter le taux de rendement des équipements en prévoyant les défaillances avant qu’elles ne surviennent.
- Contrôler le coût de la maintenance en permettant des opérations de maintenance juste à temps.
- Réduire la perte de clientèle et de ventes, améliorer l’image de marque.
- Réduire les coûts d’inventaire en réduisant les niveaux d’inventaire grâce à la prédiction du seuil de réapprovisionnement
- Découvrir les modèles liés à différents problèmes de maintenance
- Fournir des KPI (indicateurs de performance clés) tels que les scores d’intégrité qui indiquent l’état des équipements.
- Estimer la durée de vie résiduelle des équipements.
- Recommander des activités de maintenance en temps voulu.
- Activer l’inventaire juste à temps grâce à l’estimation des dates de commande pour le remplacement de pièces.

Ces objectifs sont les points de départ qui permettent :

- aux _scientifiques des données_ d’analyser et de résoudre les problèmes prédictifs spécifiques ;
- aux _architectes et développeurs de cloud_ d’établir une solution de bout en bout.

## <a name="qualifying-problems-for-predictive-maintenance"></a>Qualification des problèmes pour la maintenance prédictive
Il est important de souligner que ni tous les cas d’usage ni tous les problèmes d’entreprises ne peuvent être résolus efficacement par la PdM. Il existe trois critères de qualification importants à prendre en compte lors de la sélection des problèmes :

- Le problème doit être prédictif par nature ; autrement dit, il doit y avoir une cible ou un résultat à prédire. Le problème doit également avoir un parcours d’action clair afin d’éviter les défaillances lorsqu’elles sont détectées.
- Le problème doit comporter un enregistrement de l’historique opérationnel de l’équipement avec _les bons et les mauvais résultats_. L’ensemble des mesures prises pour atténuer de mauvais résultats doit également être disponible dans le cadre de ces enregistrements. Les rapports d’erreurs, les journaux d’activité de maintenance concernant la dégradation des performances ainsi que les journaux d’activité de réparation et de remplacement sont également importants. En outre, les réparations effectuées pour les améliorer et les enregistrements des remplacements sont également utiles.
- L’historique enregistré doit se refléter dans les données _pertinentes_ dont la qualité est _suffisante_ pour prendre en charge le cas d’usage. Pour plus d’informations sur la pertinence et la quantité nécessaire des données, consultez [Données requises pour la maintenance prédictive](#data-requirements-for-predictive-maintenance).
- Enfin, l’entreprise doit avoir des experts du domaine qui ont une vision claire du problème. Ils doivent être au courant des processus et des pratiques internes pour pouvoir aider l’analyste à comprendre et à interpréter les données. Ils doivent également pouvoir apporter les modifications nécessaires à des processus métier existants pour aider à collecter les données adaptées aux problèmes si nécessaire.

## <a name="sample-pdm-use-cases"></a>Exemples de cas d’usage de la PdM
Cette section est axée sur une collection de cas d’usage de PdM dans divers secteurs tels que l’aéronautique, la fourniture de services et le transport. Chaque section commence par un problème qui se pose à l’entreprise et décrit les avantages de la PdM, les données pertinentes cernant le problème de l’entreprise et, enfin, les avantages d’une solution de PdM.

| Problème d'entreprise | Avantages de la PdM |
|:-----------------|-------------------|
|**Aéronautique**      |                   |
|_Retards et annulations de vols_ en raison de problèmes mécaniques. Les défaillances qui ne peuvent pas être réparées à temps sont susceptibles d’entraîner des annulations de vols et de perturber leur programmation et leur exploitation. |Les solutions de PdM peuvent prédire la probabilité de retard ou d’annulation d’un avion en raison de défaillances mécaniques.|
|_Défaillance de pièces de moteurs d’avions_  : les remplacements de pièces de moteurs d’avions sont parmi les tâches de maintenance les plus courantes dans le transport aérien. Les solutions de maintenance nécessitent une gestion rigoureuse de la disponibilité en stock, de la livraison et de la planification des pièces.|La possibilité de recueillir des informations sur la fiabilité des pièces permet de réduire sensiblement les coûts d’investissement.|
|**Finances** |                         |
|La _défaillance des distributeurs automatiques de billets_ (DAB) est un problème courant dans le secteur bancaire. Ici, le problème consiste à indiquer la probabilité qu’une transaction de retrait de billets à un distributeur soit interrompue en raison d’un bourrage papier ou de la défaillance d’une pièce du distributeur. Sur la base des prédictions de défaillances au cours des transactions, les DAB peuvent faire l’objet d’opérations d’entretien proactives afin de prévenir les défaillances.| Plutôt que de risquer que la machine ne tombe en panne pendant une transaction, il est préférable de la programmer de manière à ce qu’elle se mette hors service en fonction de la prédiction|
|**Énergie** |                          |
|_Défaillance d’éoliennes_  : les éoliennes sont la principale source d’énergie dans les pays/régions soucieux de l’environnement, mais elles impliquent des coûts d’investissement élevés. Un composant clé des éoliennes est la génératrice moteur dont toute défaillance rend la turbine inefficace. De plus, sa réparation coûte très cher.|Les KPI de prédiction tels que MTTF (temps moyen de fonctionnement avant panne) peuvent aider les fournisseurs d’énergie à empêcher des pannes de turbine et à garantir des temps d’arrêt minimaux. Les probabilités de défaillance informeront les techniciens qu’il faut surveiller les turbines susceptibles de tomber en panne sous peu et permettront de programmer des régimes de maintenance dans le temps. Des modèles prédictifs fournissent des informations sur différents facteurs qui contribuent à la défaillance, ce qui permet aux techniciens de mieux comprendre les causes racines des problèmes.|
|_Défaillances de disjoncteurs_  : l’alimentation en électricité d’habitations et d’entreprises nécessite des lignes électriques constamment opérationnelles afin de garantir la livraison d’énergie. Les disjoncteurs aident à limiter ou à éviter des dommages des lignes électriques dus à des surcharges ou à de mauvaises conditions météorologiques. Ici, le problème de l’entreprise est de prédire les défaillances des disjoncteurs.| Les solutions de PdM aident à réduire les coûts de réparation et à augmenter la durée de vie d’équipements tels que les disjoncteurs. Elles permettent d’améliorer la qualité du réseau électrique en réduisant les défaillances inattendues et les interruptions de service.|
|**Transport et logistique** |    |
|_Défaillances de portes d’ascenseurs_  : les grandes sociétés fournissant des ascenseurs proposent une gamme complète de services pour des millions d’ascenseurs en fonctionnement dans le monde. La sécurité des ascenseurs, leur fiabilité et leur temps de fonctionnement sont les principales préoccupations de leurs clients. Ces sociétés surveillent ces attributs, ainsi que différents autres, pour faciliter la gestion de la maintenance préventive et corrective. Dans un ascenseur, le problème le plus souvent rencontré par les clients est le dysfonctionnement des portes. Le problème de l’entreprise est de fournir une application prédictive de base de connaissances qui prédit les causes potentielles des défaillances des portes.| Les ascenseurs représentent des investissements en capital pour une durée potentielle de 20 à 30 ans: Par conséquent, chaque vente potentielle peut être très concurrentielle. Les attentes en matière de service et de support sont donc à l’avenant. La maintenance prédictive peut procurer à ces sociétés un avantage sur leurs concurrents, au-delà des produits et des services proposés.|
|_Défaillance des roues_  : les défaillances des roues sont responsables de la moitié des déraillements de trains et coûtent des milliards au secteur des transports ferroviaires. Les roues défaillantes provoquent également la détérioration des rails, provoquant parfois leur rupture prématurée. Les ruptures de rails sont la cause de catastrophes, par exemple, des déraillements. Pour éviter de tels événements, les sociétés ferroviaires surveillent les performances des roues et les remplacent de manière préventive. Le problème de l’entreprise est de prédire les défaillances des roues.| La maintenance prédictive des roues permet d’effectuer le remplacement des roues juste à temps. |
|_Défaillances des portes des rames de métro_  : les défaillances des portes de rames de métro sont une des principales causes des retards. Ici, le problème de l’entreprise est de prédire les défaillances des portes des trains.|La prévision de la défaillance d’une porte ou du nombre de jours restants jusqu’à cette défaillance aidera l’entreprise à optimiser les plans de maintenance des portes des trains.|

La section suivante détaille comment profiter des avantages susmentionnés de la PdM.

## <a name="data-science-for-predictive-maintenance"></a>Science des données pour la maintenance prédictive

Cette section fournit des recommandations générales relatives aux principes de la science des données et aux pratiques de PdM. Elle doit aider un TDM, un architecte de solutions ou un développeur à comprendre les conditions préalables et le processus de création d’applications d’IA de bout en bout pour la PdM. Vous pouvez lire cette section, ainsi que des analyses des démonstrations et des modèles de preuves de concept répertoriées dans[Modèles de solutions pour la maintenance prédictive](#solution-templates-for-predictive-maintenance). Vous pourrez ensuite appliquer ces principes et ces meilleures pratiques pour implémenter votre solution de PdM dans Azure.

> [!NOTE]
> Ce guide n’est PAS destiné à enseigner la science des données au lecteur. Plusieurs sources utiles sont fournies pour plus d’informations dans la section [Ressources de formation pour la maintenance prédictive](#training-resources-for-predictive-maintenance). Les [modèles de solution](#solution-templates-for-predictive-maintenance) répertoriés dans le guide illustrent certaines de ces techniques d’IA dans le contexte de problèmes de PdM spécifiques.

## <a name="data-requirements-for-predictive-maintenance"></a>Exigences en matière de données pour la maintenance prédictive

La réussite d’un apprentissage dépend (a) de la qualité de ce qui est enseigné et (b) des capacités de l’apprenant. Les modèles prédictifs tirent leurs connaissances des données historiques et prédisent les résultats futurs avec une certaine probabilité en fonction des schémas observés. La précision prédictive d’un modèle dépend de la pertinence, de la quantité et de la qualité des données d’apprentissage et de test. Les nouvelles données obtenues à l’aide de ce modèle doivent avoir les mêmes fonctionnalités et suivre le même schéma que les données de formation/de test. Les caractéristiques de fonctionnalité (type, densité, distribution, etc.) de nouvelles données doivent correspondre à celles des jeux de données d’apprentissage et de test. Cette section se concentre sur ces exigences auxquelles doivent satisfaire les données.

### <a name="relevant-data"></a>Données pertinentes

Tout d’abord, les données doivent _correspondre au problème_. Prenons le cas d’usage de la _défaillance de roues_ exposé plus haut : les données de formation doivent contenir des fonctionnalités liées au fonctionnement des roues. Si le problème était de prévoir la défaillance du _système de traction_ , les données de formation doivent englober tous les différents composants de ce système. Le premier cas cible un composant spécifique, tandis que le second cas cible la défaillance d’un sous-système plus étendu. D’une manière générale, il est recommandé de concevoir des systèmes de prédiction pour des composants spécifiques plutôt que pour des sous-systèmes plus étendus, étant donné que ces derniers auront des données plus dispersées. L’expert dans ce domaine (consultez [Qualification des problèmes pour la maintenance prédictive](#qualifying-problems-for-predictive-maintenance)) doit aider à sélectionner les sous-ensembles de données les plus pertinents pour l’analyse. Les sources de données pertinentes sont mentionnées plus en détail dans [Préparation des données pour la maintenance prédictive](#data-preparation-for-predictive-maintenance).

### <a name="sufficient-data"></a>Données suffisantes
Deux questions sont fréquemment posées sur les données des historiques des défaillances : (1) « Combien d’événements de défaillance sont nécessaires pour entraîner un modèle ? » (2) « Combien d’enregistrements sont considérés comme "suffisants" ? » Il n’y a pas de réponses définitives, mais uniquement des règles empiriques. Concernant (1), plus il y d’événements de défaillances, plus le modèle est meilleur. Concernant (2), le nombre exact des événements de défaillance varie selon les données et le contexte du problème à résoudre. En revanche, si une machine tombe trop souvent en panne, l’entreprise la remplacera, ce qui réduira les instances de défaillance. Ici encore, les conseils donnés par l’expert du domaine concerné sont importants. Cependant, il existe des méthodes permettant de faire face au problème des _événements rares_. Elles sont présentées dans la section [Gestion de données déséquilibrées](#handling-imbalanced-data).

### <a name="quality-data"></a>Données de qualité
La qualité des données est cruciale : chaque valeur d’attribut de prédiction doit être _précise_ , de même que la valeur de la variable cible. La qualité des données est un domaine bien étudié dans les statistiques et la gestion des données et n’est donc pas traitée dans ce guide.

> [!NOTE]
> Il existe plusieurs ressources et produits d’entreprise pour fournir des données de qualité. Vous trouverez ci-dessous un exemple de références :
> - Dasu, T, Johnson, Exploratory Data Mining and Data Cleaning, Wiley, 2003.
> - [Exploratory Data Analysis, Wikipedia](https://en.wikipedia.org/wiki/Exploratory_data_analysis)
> - [Hellerstein, J, Quantitative Data Cleaning for Large Databases](http://db.cs.berkeley.edu/jmh/papers/cleaning-unece.pdf)
> - [de Jonge, E, van der loo, M, Introduction to Data Cleaning with R](https://cran.r-project.org/doc/contrib/de_Jonge+van_der_Loo-Introduction_to_data_cleaning_with_R.pdf)

## <a name="data-preparation-for-predictive-maintenance"></a>Préparation des données pour la maintenance prédictive

### <a name="data-sources"></a>Sources de données

Les sources de données pertinentes pour la maintenance prédictive incluent, sans s’y limiter :
- L’historique des défaillances
- Historique des maintenances/des réparations
- Les conditions de fonctionnement de la machine
- Les métadonnées de l’équipement

#### <a name="failure-history"></a>L’historique des défaillances
Les événements de défaillance sont rares dans les applications de PdM. Toutefois, lors de la création de modèles de prédiction, l’algorithme doit en savoir plus sur le modèle de fonctionnement normal d’un composant, ainsi que sur ses modèles de défaillance. Par conséquent, les données d’apprentissage doivent contenir un nombre suffisant d’exemples de ces deux catégories. Les dossiers de maintenance et l’historique du remplacement des pièces sont de bonnes sources pour rechercher des événements de défaillance. Grâce à des connaissances du domaine respectivement traité, des anomalies des données d’apprentissage peuvent également être définies comme des défaillances.

#### <a name="maintenancerepair-history"></a>Historique des maintenances/des réparations
L’historique de maintenance d’un élément contient des détails sur les composants remplacés, les réparations effectuées, etc. Ces événements enregistrent les modèles de dégradation. L’absence de ces informations cruciales dans les données d’apprentissage peut fausser les résultats des modèles. L’historique des défaillances peut également se trouver dans l’historique de maintenance sous la forme de codes d’erreur spéciaux ou de dates de commandes de pièces. Des sources de données supplémentaires qui influencent les modèles de défaillance doivent être étudiées et fournies par des experts du domaine.

#### <a name="machine-operating-conditions"></a>Les conditions de fonctionnement de la machine
Les données de diffusion en continu issues de capteurs (ou d’autres dispositifs) de l’équipement en fonctionnement sont une source de données importantes. Une hypothèse clé dans la PdM est que l’état d’intégrité d’une machine se dégrade au fil du temps pendant son fonctionnement de routine. Nous prévoyons que les données contiendront des caractéristiques variant avec le temps qui capturent ce modèle de vieillissement et toute anomalie entraînant une dégradation. L’aspect temporel des données est requis pour que l’algorithme apprenne les modèles de défaillance et de non-défaillance au fil du temps. En fonction de ces points de données, l’algorithme apprend à prédire pendant combien d’unités de temps en plus une machine peut continuer à fonctionner avant de présenter une défaillance.

#### <a name="static-feature-data"></a>Données de fonctionnalité statique
Les fonctionnalités statiques sont des métadonnées sur l’équipement. Il peut, par exemple, s’agir de la marque, du modèle, de la date de fabrication, de la date de mise en service, de l’emplacement du système et d’autres spécifications techniques.

Des exemples de données pertinentes pour les [exemples de cas d’usage de la PdM](#sample-pdm-use-cases) sont mentionnés ci-dessous sous forme de tableau :

| Cas d’usage | Exemples de données pertinentes |
|:---------|---------------------------|
|_Retards et annulations de vols_ | Informations sur les itinéraires de vol sous la forme d’étapes de vol et de journaux d’activité. Les données des étapes de vol contiennent des détails tels que la date de départ/d’arrivée, l’heure, l’aéroport, les escales, etc. Les journaux incluent une série de codes d’erreur et de maintenance enregistrés par le personnel de maintenance au sol.|
|_Défaillance de pièces de moteurs d’avions_ | Données collectées à partir de capteurs de l’appareil qui fournissent des informations sur l’état des différentes pièces. Des enregistrements de maintenance aident à identifier quand des défaillances des composants se sont produites et quand ces composants ont été remplacés.|
|_Défaillance de distributeurs automatiques de billets_ | Lectures des capteurs lors de chaque transaction (versements en espèces/chèques) et retrait d’espèces. Informations sur l’écart mesuré entre les billets, l’épaisseur des billets, l’intervalle d’arrivée des billets, les attributs des chèques, etc. Dossiers de maintenance qui fournissent des codes d’erreur, des informations sur les réparations, l’heure de la dernière recharge du distributeur de billets.|
|_Défaillance d’éoliennes_ | Des capteurs surveillent les conditions de fonctionnement de la turbine, par exemple, la température, la direction du vent, l’électricité produite, la vitesse du générateur, etc. Les données de plusieurs éoliennes de parcs situés dans différentes régions sont recueillies. En règle générale, plusieurs lectures des capteurs avec transmission des valeurs mesurées auront lieu à intervalles fixes.|
|_Défaillances de disjoncteurs_ | Journaux d’activité de maintenance qui incluent des actions correctives, prévention et systématiques. Données opérationnelles qui incluent des commandes automatiques et manuelles envoyées à des disjoncteurs, par exemple, pour l’ouverture et la fermeture. Métadonnées du dispositif, telles que la date de fabrication, l’emplacement, le modèle, etc. Spécifications des disjoncteurs, telles que les niveaux de tension, la géolocalisation, les conditions ambiantes.|
|_Défaillances de portes d’ascenseurs_| Métadonnées de l’ascenseur, telles que le type d’ascenseur, la date de fabrication, les intervalles de maintenance, le type de construction, etc. Informations opérationnelles, telles que le nombre de cycles de la porte et la durée moyenne de fermeture de la porte. Historique des défaillances avec les causes.|
|_Défaillance des roues_ | Données de capteur pour mesurer l’accélération des roues, les instances de freinage, la distance parcourue, la vitesse, etc. Informations statiques sur les roues, telles que le fabricant et la date de fabrication. Données de défaillance provenant de la base de données de commande de pièces, qui assurent le suivi des dates de commande et des quantités commandées.|
|_Défaillances des portes des rames de métro_ | Heures d’ouverture et de fermeture des portes, autres données opérationnelles, telles que l’état actuel des portes des trains. Les données statiques incluraient identificateur de ressource, l’heure et les colonnes de valeurs de condition.|

### <a name="data-types"></a>Types de données
Au vu des sources de données ci-dessus, les deux principaux types de données suivants sont observés dans le domaine de la PdM :

- _Jointure temporelle_  : télémétrie opérationnelle, état de la machine, type d’ordre de travail, codes de priorité ayant des horodatages au moment de l’enregistrement. L’historique des défaillances, des maintenance/réparations et de l’utilisation aura également des horodatages associés à chaque événement.
- _Données statiques_  : les caractéristiques des machines et des opérateurs sont généralement statiques, dans la mesure où elles décrivent les spécifications techniques des machines ou les attributs des opérateurs. Si ces fonctionnalités changent au fil du temps, des horodatages doivent également leur être associés.

Les variables de prédiction et cible doivent être prétraitées/transformées en [types de données numériques, par catégorie et autres](https://www.statsdirect.com/help/basics/measurement_scales.htm) selon l’algorithme utilisé.

### <a name="data-preprocessing"></a>Prétraitement des données
Une condition préalable à _l’ingénierie des caractéristiques_ est la préparation des données à partir de différents flux, pour composer un schéma à partir duquel on pourra facilement créer des fonctionnalités. Commencez par visualiser les données sous la forme d’un tableau d’enregistrements. Chaque ligne du tableau représente une instance d’apprentissage et les colonnes représentent des fonctionnalités _de prédiction_ (également appelées attributs indépendants ou variables). Organisez les données de telle manière que la ou les dernières colonnes soient la _cible_ (variable dépendante). Pour chaque instance d’apprentissage, assignez une _étiquette_ en tant que valeur de cette colonne.

Pour obtenir des données temporelles, divisez la durée des données du capteur en unités de temps. Chaque enregistrement doit appartenir à une unité de temps d’un équipement _et contenir des informations distinctes_. Les unités de temps sont définies sur la base des besoins métier en multiples de secondes, de minutes, d’heures, de jours, de mois, et ainsi de suite. L’unité de temps _ne doit pas nécessairement être la même que la fréquence de collecte des données_. Si la fréquence est élevée, les données peuvent ne pas présenter de différences significatives d’une unité à l’autre. Par exemple, supposons que la température ambiante a été collectée toutes les 10 secondes. Utiliser ce même intervalle pour les données d’apprentissage augmente uniquement le nombre d’exemples sans fournir d’informations supplémentaires. Dans ce cas, l’utilisation d’une moyenne des données sur 10 minutes ou sur une heure, en fonction de la justification économique, serait une meilleure stratégie.

Pour les données statiques
- _Enregistrements de maintenance_  : les données brutes de maintenance ont un identificateur d’équipement et un horodatage avec des informations sur les activités de maintenance qui ont été effectuées à un moment donné. Transformez les activités de maintenance en colonnes _de catégories_ , où chaque descripteur de catégorie correspond uniquement à une action de maintenance spécifique. Le schéma des enregistrements de maintenance inclut un identificateur d’équipement, une heure et une action de maintenance.

- _Enregistrements des défaillances_  : les défaillances ou leurs causes peuvent être enregistrées sous forme de codes d’erreurs ou d’événements de défaillance spécifiques, selon des conditions spécifiques à l’entreprise. Dans les cas où l’équipement a plusieurs codes d’erreurs, l’expert du domaine doit aider à identifier celles qui sont pertinentes par rapport à la variable cible. Utilisez les codes d’erreurs restants ou les conditions pour construire des fonctionnalités _de prédiction_ mises en corrélation avec ces défaillances. Le schéma des enregistrements des défaillances comprendra l’identificateur d’équipement, la date/heure, ou la raison de la défaillance si elle est disponible.

- _Métadonnées des machines et des opérateurs_  : fusionnez les données des machines et des opérateurs dans un seul schéma, de façon à associer un équipement à son opérateur, avec leurs attributs respectifs. Le schéma de l’état de la machine comprendra un l’identificateur d’équipement, les caractéristiques de l’équipement, l’identificateur de l’opérateur et les caractéristiques de l’opérateur.

D’autres étapes de prétraitement des données incluent _la gestion des valeurs manquantes_ et la _normalisation_ de valeurs d’attributs. Une discussion détaillée dépasserait le cadre de ce guide. Vous trouverez des références utiles dans la section suivante.

Une fois les sources de données prétraitées ci-dessus en place, la transformation finale avant l’ingénierie des caractéristiques consiste à joindre les tables ci-dessus en fonction de l’identificateur d’équipement et de l’horodatage. Le tableau résultant a des valeurs Null dans la colonne des défaillances si la machine fonctionne normalement. Ces valeurs Null peuvent être imputées par un indicateur pour un fonctionnement normal. Utilisez cette colonne de défaillances pour créer des _étiquettes pour le modèle prédictif_. Pour plus d’informations, consultez la section sur les [techniques de modélisation pour la maintenance prédictive](#modeling-techniques-for-predictive-maintenance).

## <a name="feature-engineering"></a>Ingénierie des caractéristiques
L’ingénierie des caractéristiques est la première étape avant la modélisation des données. Son rôle dans le processus de science des données [est décrit ici](./create-features.md). Une _fonctionnalité_ est un attribut prédictif du modèle, tel que la température, la pression, la vibration, etc. Pour la PdM, l’ingénierie des caractéristiques implique d’extraire l’intégrité d’une machine de données historiques collectées pendant une durée conséquente. Dans ce sens, elle est différente de ses pairs, tels que l’analyse à distance, la détection d’anomalies et la détection de défaillances. 

### <a name="time-windows"></a>Fenêtres de temps
La surveillance à distance implique des rapports sur les événements qui se produisent à partir de _moments donnés_. Les modèles de détection des anomalies évaluent les flux entrants de données pour indiquer des anomalies à partir de moments donnés. La détection des défaillances classe les défaillances en types spécifiques en fonction du moment où elles surviennent. En revanche, la PdM implique la prédiction de défaillances pour une _période future_ , selon des fonctionnalités qui reflètent le comportement de la machine sur une _période de temps historique_. Pour la PdM, les données de fonctionnalité de moments individuels sont trop bruyantes pour être prédictives. Par conséquent, les données de chaque fonctionnalité doivent être _lissées_ en agrégeant les points de données sur des fenêtres de temps.

### <a name="lag-features"></a>Caractéristiques antérieures correspondantes
Les exigences métier définissent jusqu’où le modèle doit prédire dans le futur. Cette durée permet, quant à elle, de définir « jusqu’où le modèle doit remonter » pour faire ces prédictions. Cette période de « retour » est appelée _décalage_ , et les fonctionnalités conçues sur cette période de décalage sont appelées _caractéristiques antérieures correspondantes_. Cette section décrit les caractéristiques antérieures correspondantes qui peuvent être construites à partir de sources de données avec horodatages et présente une création à partir de sources de données statiques. Les caractéristiques antérieures correspondantes sont généralement de nature _numérique_.

> [!IMPORTANT]
> La taille de la fenêtre est déterminée par expérimentation et doit être finalisée à l’aide d’un expert du domaine. La même restriction est valable pour la sélection et la définition de caractéristiques antérieures correspondantes, de leurs agrégations et du type de fenêtres.

#### <a name="rolling-aggregates"></a>Agrégats cumulatifs
Pour chaque enregistrement d’un équipement, une fenêtre propagée de taille « W » est choisie comme nombre d’unités de temps pour calculer les agrégats. Nous calculons ensuite les caractéristiques antérieures correspondantes en utilisant les périodes W _antérieures à la date_ de cet enregistrement. Dans la Figure 1, les lignes bleues indiquent les valeurs de capteur enregistrées sur un équipement pour chaque unité de temps. Elles indiquent une moyenne propagée des valeurs de fonctionnalité sur une fenêtre de taille W = 3. La moyenne propagée est calculée sur tous les enregistrements dont les horodatages se situent dans la plage de t<sub>1</sub> (en orange) à t<sub>2</sub> (en vert). La valeur de W est généralement exprimée en minutes ou en heures, en fonction de la nature des données. Mais, pour certains problèmes, la sélection d’une grande fenêtre de taille W (par exemple, 12 mois) peut fournir tout l’historique d’un équipement jusqu’au moment de l’enregistrement.

![Figure 1. Caractéristiques des agrégats cumulatifs](./media/predictive-maintenance-playbook/rolling-aggregate-features.png)

Figure 1. Caractéristiques des agrégats cumulatifs

Des exemples d’agrégats cumulatifs sur une fenêtre de temps sont le nombre, la moyenne, les mesures CUMESUM (somme cumulée) et les valeurs mini/maxi. La variance, l’écart standard et le nombre de valeurs aberrantes au-delà des écarts standard N sont également souvent utilisés. Les exemples d’agrégats qui peuvent être appliqués aux [cas d’usage](#sample-pdm-use-cases) de ce guide sont répertoriés ci-dessous. 
- _Retard de vol_  : nombre de codes d’erreurs le jour précédent/la semaine précédente.
- _Défaillances des pièces de moteurs d’avions_  : moyennes propagées, écart standard et somme pour le jour précédent, la semaine précédente, etc. Cette métrique doit être déterminée avec l’expert métier du domaine.
- _Défaillances des distributeurs automatiques de billets_  : moyennes propagées, valeurs médianes, plage, écarts standard, nombre de valeurs aberrantes au-delà de trois déviations standard, mesures CUMESUM supérieures et inférieures.
- _Défaillances des portes des rames de métro_  : nombre d’événements le jour précédent, la semaine précédente, les deux semaines précédentes, etc.
- _Défaillances de disjoncteurs_  : nombre de défaillances la semaine précédente, l’année précédente, les trois années précédentes, etc.

Une autre technique utile dans la PdM consiste à capturer les changements de tendance, les pics et les changements de niveau en utilisant des algorithmes qui détectent des anomalies dans les données.

#### <a name="tumbling-aggregates"></a>Agrégats bascules
Pour chaque intitulé enregistrement d’un équipement, une fenêtre de taille _W - <sub>k</sub>_ est définie, où _k_ est le nombre de fenêtres de taille _W_. Les agrégats sont ensuite créés sur _k_ _fenêtres bascules_ _W-k, W- <sub>(k-1)</sub>, ..., W- <sub>2</sub>, W- <sub>1</sub>_ pour les périodes précédant l’horodatage d’un enregistrement. _k_ peut être un petit nombre pour capturer des effets à court terme, ou un grand nombre pour capturer des modèles de dégradation à long terme. (voir Figure 2).

![Figure 2 : Caractéristiques des agrégats bascules](./media/predictive-maintenance-playbook/tumbling-aggregate-features.png)

Figure 2 : Caractéristiques des agrégats bascules

Par exemple, les caractéristiques antérieures correspondantes pour le cas d’usage des éoliennes peuvent être créées avec W = 1 et k = 3. Elles impliquent le décalage pour chacun des trois derniers mois à l’aide de valeurs aberrantes supérieures et inférieures.

### <a name="static-features"></a>Caractéristiques statiques

Les spécifications techniques de l’équipement, comme la date de fabrication, le numéro de modèle ou l’emplacement sont des exemples de caractéristiques statiques. Elles sont traitées comme des _variables_ de catégorie pour la modélisation. Quelques exemples pour le cas d’usage des disjoncteurs sont la tension, le courant, la capacité énergétique, le type de transformateur et la source d’alimentation. Concernant les défaillances des roues, le type de roue (alliage vs acier) est un exemple.

Les efforts de préparation des données présentées jusqu’ici doivent aboutir à une organisation des données comme indiqué ci-dessous. Les données d’apprentissage, de test et de validation doivent avoir ce schéma logique (cet exemple illustre le temps en unités de jours).

| ID d’équipement | Temps | \<Feature Columns> | Étiquette |
| ---- | ---- | --- | --- |
| A123 |Jour 1 | . . . | . |
| A123 |Jour 2 | . . . | . |
| ...  |...   | . . . | . |
| B234 |Jour 1 | . . . | . |
| B234 |Jour 2 | . . . | . |
| ...  |...   | . . . | . |

La dernière étape de l’ingénierie des caractéristiques est l’ **étiquetage** de la variable cible. Ce processus dépend de la technique de modélisation. Quant à la technique de modélisation, elle dépend du problème métier et de la nature des données disponibles. L’étiquetage est décrit dans la section suivante.

> [!IMPORTANT]
> La préparation des données et l’ingénierie des caractéristiques sont aussi importantes que les techniques de modélisation pour arriver à des solutions de PdM réussies. L’expert du domaine et le praticien doivent investir beaucoup de temps afin d’obtenir les bonnes caractéristiques et données pour le modèle. Un petit échantillon de nombreux livres sur l’ingénierie des caractéristiques est répertorié ci-dessous :
> - Pyle, D. Data Preparation for Data Mining (The Morgan Kaufmann Series in Data Management Systems), 1999
> - Zheng, A., Casari, A. Feature Engineering for Machine Learning: Principles and Techniques for Data Scientists, O’Reilly, 2018.
> - Dong, G. Liu, H. (Editors), Feature Engineering for Machine Learning and Data Analytics (Chapman & Hall/CRC Data Mining and Knowledge Discovery Series), CRC Press, 2018.

## <a name="modeling-techniques-for-predictive-maintenance"></a>Techniques de modélisation pour la maintenance prédictive

Cette section présente les principales techniques de modélisation pour les problèmes de PdM, ainsi que leurs méthodes de construction d’étiquettes spécifiques. Notez qu’une technique de modélisation unique peut être utilisée dans différents secteurs d’activité. La technique de modélisation est plutôt liée au problème de la science des données qu’au contexte des données disponibles.

> [!IMPORTANT]
> Le choix des étiquettes pour les cas de défaillances et la stratégie d’étiquetage  
> doit être déterminé en accord avec l’expert du domaine.

### <a name="binary-classification"></a>Classification binaire
La classification binaire est utilisée pour _prédire la probabilité de défaillance d’un équipement dans une période de temps future_ appelée _période d’horizon futur_. X est déterminé par le problème de l’entreprise et les données disponibles en accord avec l’expert du domaine. Voici quelques exemples :
- _délai minimal_ requis pour remplacer des composants, déployer des ressources de maintenance, effectuer une maintenance pour éviter un problème qui peut se produire pendant cette période.
- _nombre minimal d’événements_ qui peuvent se produire avant qu’un problème ne survienne.

Dans cette technique, deux types d’exemples d’apprentissage sont identifiés. Un exemple positif, _qui indique une défaillance_ , avec l’étiquette = 1. Un exemple négatif, qui indique une défaillance, avec l’étiquette = 0. La variable cible, donc les valeurs d’étiquettes, sont _catégorielles_. Le modèle doit identifier chaque nouvel exemple comme étant susceptible de faire l’objet d’une défaillance ou de fonctionner normalement pendant les X prochaines unités de temps.

#### <a name="label-construction-for-binary-classification"></a>Construction des étiquettes pour la classification binaire
La question ici est : « Quelle est la probabilité de défaillance de l’équipement pendant les X prochaines unités de temps ? » Pour répondre à cette question, étiquetez X enregistrements avant la défaillance d’un équipement comme « sur le point de tomber en panne » (étiquette = 1) et étiquetez tous les autres enregistrements comme « normaux » (label = 0). (voir Figure 3).

![Figure 3. Étiquetage pour la classification binaire](./media/predictive-maintenance-playbook/labelling-for-binary-classification.png)

Figure 3. Étiquetage pour la classification binaire

Des exemples de stratégies d’étiquetage pour certains des cas d’usage sont répertoriés ci-dessous.
- _Retards des vols_  : X peut être choisi comme représentant 1 jour pour prédire des retards dans les prochaines 24 heures. Tous les vols programmés dans les 24 heures avant les défaillances sont étiquetés 1.
- _Défaillances des distributeurs automatiques de billets_  : un objectif peut consister à déterminer la probabilité de défaillance d’une transaction dans l’heure à venir. Dans ce cas, toutes les transactions effectuées dans l’heure passée de la défaillance sont étiquetées 1. Afin de prédire la probabilité de défaillance pour les N billets suivants distribués, tous les billets distribués avec les derniers billets N d’une défaillance sont étiquetés 1.
- _Défaillances de disjoncteurs_  : l’objectif peut être de prédire la prochaine défaillance de la commande du disjoncteur. Dans ce cas, X est choisi pour être une commande ultérieure.
- _Défaillances des portes des rames_  : X peut être choisi comme représentant deux jours.
- _Défaillance d’éoliennes_  : X peut être choisi comme représentant deux mois.

### <a name="regression-for-predictive-maintenance"></a>Régression pour la maintenance prédictive
Des modèles de régression sont utilisés pour _calculer la durée de vie utile (RUL) résiduelle d’un équipement_. La durée de vie utile est définie comme le temps de fonctionnement d’un équipement jusqu’à la prochaine défaillance. Chaque exemple d’apprentissage est un enregistrement qui correspond à une unité de temps _nY_ d’un équipement, où _n_ est le multiple. Le modèle doit calculer la durée de vie utile de chaque nouvel exemple sous forme de _nombre continu_. Ce nombre indique le temps résiduel avant la défaillance.

#### <a name="label-construction-for-regression"></a>Construction des étiquettes pour la régression
La question ici est : « Quelle est la durée de vie restante de l’équipement ? » Pour chaque enregistrement précédant la défaillance, calculez l’étiquette afin qu’elle correspond au nombre d’unités de temps résiduelles avant la défaillance suivante. Dans cette méthode, les étiquettes sont des variables continues. (Figure 4)

![Figure 4. Étiquetage pour la régression](./media/predictive-maintenance-playbook/labelling-for-regression.png)

Figure 4. Étiquetage pour la régression

Pour la régression, l’étiquetage est effectué avec une référence à un point de défaillance. Son calcul n’est pas possible sans connaître la durée pendant laquelle l’équipement a survécu avant une défaillance. Par opposition à la classification binaire, les équipements dont les données ne contiennent pas de défaillances ne peuvent pas être utilisés pour la modélisation. Ce problème est mieux résolu par une autre technique statistique appelée [Analyse de survie](https://en.wikipedia.org/wiki/Survival_analysis). Cependant, des complications peuvent survenir lors de l’application de cette technique à des cas d’usage de la PdM impliquant des données qui varient dans le temps à des intervalles fréquents. Pour plus d’informations sur l’analyse de survie, consultez [ce document d’une page](https://www.cscu.cornell.edu/news/news.php/stnews78.pdf).

### <a name="multi-class-classification-for-predictive-maintenance"></a>Classification multiclasse pour la maintenance prédictive
Les techniques de classification multiclasse peuvent être utilisées dans des solutions de PdM pour deux scénarios :
- Prédire _deux résultats futurs_  : le premier résultat est _une plage de temps jusqu’à la défaillance_ d’un équipement. L’équipement est affecté à une de plusieurs périodes possibles. Le second est la probabilité d’une défaillance dans une période future en raison d’ _une des différentes causes racines_. Cette prédiction permet à l’équipe de maintenance de surveiller les signes annonciateurs et de programmer la maintenance.
- Prédire _la cause racine la plus probable_ d’une défaillance donnée. Pour ce résultat, il faut disposer du bon ensemble d’actions de maintenance afin de remédier à une défaillance. Une liste des causes racines et des réparations recommandées classée par ordre de priorité peut aider les techniciens à hiérarchiser leurs actions de réparation après une défaillance.

#### <a name="label-construction-for-multi-class-classification"></a>Construction des étiquettes pour la classification multiclasse
La question ici est : « Quelle est la probabilité de défaillance d’un équipement pendant les _nZ_ prochaines unités de temps, où _n_ représente le nombre de périodes ? » Pour répondre à cette question, étiquetez nZ enregistrements avant la défaillance d’un équipement à l’aide de compartiments de temps (3Z, 2Z, Z). Étiquetez tous les autres enregistrements comme « normaux » (étiquette = 0). Dans cette méthode, la variable cible contient des valeurs _de catégorie_. (voir Figure 5).

![Figure 5. Étiquettes de prédiction du moment de l’échec pour la classification multiclasse](./media/predictive-maintenance-playbook/labelling-for-multiclass-classification-for-failure-time-prediction.png)

Figure 5. Étiquetage pour la classification multiclasse en vue de la prédiction de l’heure de la défaillance

La question ici est : « Quelle est la probabilité de défaillance de l’équipement pendant les X prochaines unités de temps en raison de la cause racine/du problème _P <sub>i</sub>_  ? » où _i_ est le nombre de causes racines possibles. Pour répondre à cette question, étiquetez X enregistrements avant la défaillance d’un équipement comme « sur le point de tomber en panne » en raison de la cause racine _P <sub>i</sub>_ (étiquette = _P <sub>i</sub>_ ). Étiquetez tous les autres enregistrements comme étant « normaux » (étiquette = 0). Dans cette méthode également, les étiquettes sont catégorielles (voir Figure 6).

![Figure 6. Étiquettes de prédiction de la cause racine pour la classification multiclasse](./media/predictive-maintenance-playbook/labelling-for-multiclass-classification-for-root-cause-prediction.png)

Figure 6. Étiquetage pour la classification multiclasse, pour la prédiction de la cause racine

Le modèle affecte une probabilité de défaillance en raison de chaque _P <sub>i</sub>_ , ainsi que la probabilité qu’il n’y ait pas de défaillance. Ces probabilités peuvent être classées par ordre de grandeur pour permettre la prédiction des problèmes les plus susceptibles de se produire dans le futur.

La question ici est : « Quelles actions de maintenance recommandez-vous après une défaillance ? » Pour répondre à cette question, l’étiquetage _ne nécessite pas de choisir un horizon futur_ , car le modèle ne prédit pas de défaillance dans le futur. Il prédit simplement la cause racine la plus vraisemblable _une fois que la défaillance est déjà survenue_.

## <a name="training-validation-and-testing-methods-for-predictive-maintenance"></a>Méthodes d’apprentissage, de validation et de test pour la maintenance prédictive
Le [processus d’équipe de la science des données](./overview.md) fournit une prise en charge complète du cycle apprentissage/validation/test. Cette section traite des aspects propres à la PdM.

### <a name="cross-validation"></a>Validation croisée
L’objectif de la [validation croisée](https://en.wikipedia.org/wiki/Cross-validation_(statistics)) est de définir un jeu de données pour « tester » le modèle dans la phase d’apprentissage. Ce jeu de données est appelé _jeu de validation_. Cette technique aide à limiter des problèmes tels que le _surajustement_ et donne un aperçu de la manière dont le modèle sera généralisé pour devenir un jeu de données indépendant. Autrement dit, un jeu de données inconnu qui pourrait être tiré d’un problème réel. La routine d’apprentissage et de test pour la PdM doit prendre en compte les aspects variables dans le temps pour mieux se généraliser sur des données futures non consultées.

De nombreux algorithmes d’apprentissage automatique dépendent d’un certain nombre d’hyperparamètres qui peuvent changer considérablement les performances du modèle. Les valeurs optimales de ces hyperparamètres ne sont pas calculées automatiquement lors de l’apprentissage du modèle. Ils doivent être spécifiés par le scientifique des données. Il existe plusieurs façons de trouver les bonnes valeurs des hyperparamètres.

La plus courante est la _validation croisée de k échantillons_ , qui fractionne les exemples de façon aléatoire en _k_ échantillons. Pour chaque ensemble de valeurs d’hyperparamètres, exécutez l’algorithme d’apprentissage _k_ fois. À chaque itération, utilisez les exemples de l’échantillon actuel comme ensemble de validation et le reste des exemples comme ensemble d’apprentissage. Effectuez l’apprentissage de l’algorithme sur des exemples d’apprentissage, et calculez les mesures de performance avec des exemples de validation. À la fin de cette boucle, calculez la moyenne de _k_ métriques de performances. Dans chaque ensemble de valeurs d’hyperparamètres, choisissez celles qui ont la meilleure performance moyenne. La tâche du choix d’hyperparamètres est, par nature, souvent expérimentale.

Dans les problèmes de PdM, les données sont enregistrées sous la forme d’une série chronologique d’événements provenant de plusieurs sources de données. Ces enregistrements peuvent être classés en fonction du moment de l’étiquetage. Par conséquent, si le jeu de données est fractionné _de manière aléatoire_ en un jeu d’apprentissage et un jeu de validation, _certains exemples d’apprentissage peuvent être postérieurs à certains exemples de validation_. Les performances futures de valeurs d’hyperparamètres seront estimées sur la base de données correspondant à des moments _antérieurs_ à l’apprentissage du modèle. Ces estimations peuvent être trop optimistes, surtout si les séries chronologiques ne sont pas fixes et évoluent au fil du temps. Par conséquent, les valeurs choisies pour les hyperparamètres peuvent ne pas être optimales.

Il est recommandé de fractionner les exemples en un jeu d’apprentissage et un jeu de validation d’une façon _temporelle_ , où tous les exemples de la validation sont postérieurs à tous les exemples de l’apprentissage. Pour chaque ensemble de valeurs d’hyperparamètres, assurez l’apprentissage de l’algorithme sur le jeu de données d’apprentissage. Mesurez les performances du modèle sur le même jeu de validation. Choisissez les valeurs d’hyperparamètres qui affichent les meilleures performances. Les valeurs des hyperparamètres choisies selon un fractionnement d’apprentissage/de validation conduisent à de meilleures performances futures du modèle que celles choisies de façon aléatoire par validation croisée.

Le modèle final peut être généré en formant un algorithme d’apprentissage sur des données d’apprentissage complètes à l’aide des meilleures valeurs d’hyperparamètres.

### <a name="testing-for-model-performance"></a>Test des performances du modèle
Une fois qu’un modèle est créé, une estimation de ses performances futures sur de nouvelles données est requise. Une bonne estimation est une mesure de la performance des valeurs d’hyperparamètres calculées sur le jeu de validation ou une mesure de la performance moyenne calculée à partir de la validation croisée. Ces estimations sont souvent trop optimistes. L’entreprise peut souvent bénéficier de conseils supplémentaires sur la façon de tester le modèle.

La méthode recommandée pour la PdM consiste à fractionner les exemples en jeux de données d’apprentissage, de validation et de test _temporels_. Tous les exemples de test doivent être postérieurs à tous les exemples d’apprentissage et de validation. Après le fractionnement, générez le modèle et mesurez ses performances, comme décrit précédemment.

Quand les séries chronologiques sont fixes et faciles à prédire, les deux approches (aléatoire et temporelle) génèrent des estimations similaires des performances futures. Cependant, quand les séries chronologiques ne sont pas fixes et/ou sont difficiles à prédire, l’approche temporelle va générer une estimation plus réaliste des performances futures.

### <a name="time-dependent-split"></a>Fractionnement temporel
Cette section décrit les meilleures pratiques pour implémenter le fractionnement temporel. Un fractionnement bidirectionnel temporel entre des jeux d’apprentissage et des jeux de test est décrit ci-dessous.

Supposons que nous avons un flux d'événements horodatés tels que des mesures de différents capteurs. Définissez les caractéristiques et les étiquettes des exemples d’apprentissage et de test sur des périodes contenant plusieurs événements. Par exemple, pour la classification binaire, créez des caractéristiques basées sur des événements passés et des étiquettes en fonction d’événements futurs dans « X » unités de temps (voir les sections sur l’[ingénierie des caractéristiques](#feature-engineering) et les techniques de modélisation). Par conséquent, la période d’étiquetage d'un exemple intervient après la période de ses caractéristiques.

Pour le fractionnement temporel, choisissez une _limite d’apprentissage T <sub>c</sub>_ pour développer un modèle, avec des hyperparamètres réglés à l’aide des données historiques jusqu’à T <sub>c</sub>. Pour empêcher la fuite des étiquettes futures qui dépassent T<sub>c</sub> dans les données d'apprentissage, nous choisissons X unités avant T<sub>c</sub> comme dernière période pour étiqueter des exemples d’apprentissage. Dans l’exemple illustré dans la Figure 7, chaque carré représente un enregistrement dans le jeu de données, où les fonctionnalités et les étiquettes sont calculées comme décrit ci-dessus. La figure illustre les enregistrements qui doivent être placés dans des jeux d’apprentissage et de test pour X = 2 et W = 3 :

![Figure 7. Fractionnement temporel pour la classification binaire](./media/predictive-maintenance-playbook/time-dependent-split-for-binary-classification.png)

Figure 7. Fractionnement temporel pour la classification binaire

Les carrés verts représentent des enregistrements appartenant aux unités de temps qui peuvent être utilisées pour l'apprentissage. Chaque exemple d’apprentissage est généré en prenant en compte les trois dernières périodes de génération de caractéristiques et les deux périodes futures pour l’étiquetage avant<sub>c</sub>. Lorsqu’une partie des deux périodes futures va au-delà de T<sub>c</sub>, excluez cet exemple du jeu de données d’apprentissage, car aucune visibilité n’est supposée au-delà de T<sub>c</sub>.

Les carrés noirs représentent les enregistrements du jeu de données étiqueté final qui ne doivent pas être utilisés dans le jeu de données d'apprentissage au vu de la contrainte susmentionnée. Ces enregistrements ne seront pas non plus utilisés dans des données de test, puisqu’ils sont antérieurs à T<sub>c</sub>. En outre, le délai d’exécution de leur étiquetage dépend en partie du délai d’exécution de l’apprentissage, qui n’est pas idéal. Les données d’apprentissage et de test doivent avoir des délais d’exécution d’étiquetage distincts afin d’éviter les fuites d’informations d’étiquettes.

La technique présentée jusqu'à présent permet un chevauchement entre des exemples d’apprentissage et de test qui contiennent des horodatages proches de T<sub>c</sub>. Une solution pour obtenir une meilleure séparation consiste à exclure des exemples qui se situent dans des unités de temps W de T<sub>c</sub> du jeu de test. Mais ce fractionnement agressif dépend de la bonne disponibilité des données.

Les modèles de régression utilisés pour prédire la durée de vie utile sont plus gravement affectés par le problème de fuite. L’utilisation de la méthode de fractionnement aléatoire entraîne un surajustement extrême. Pour les problèmes de régression, le fractionnement doit être tel que les enregistrements appartenant à des équipements ayant présenté des défaillances avant T<sub>c</sub> fassent partie du jeu d’apprentissage. Les enregistrements d’équipements ayant présenté des défaillances après la limite font partie du jeu de test.

Une autre meilleure pratique pour fractionner des données d’apprentissage et de test consiste à utiliser un fractionnement par ID d’équipement. Le fractionnement doit être tel qu’aucun des équipements utilisés dans le jeu d’apprentissage ne le soit pour tester les performances du modèle. Grâce à cette approche, un modèle est plus susceptible de fournir des résultats plus réalistes avec de nouveaux équipements.

### <a name="handling-imbalanced-data"></a>Gestion de données déséquilibrées
Dans les problèmes de classification, s'il y a plus d'exemples pour une classe que pour les autres, on parle d’un jeu de données _déséquilibré_. Dans l'idéal, on préfère avoir suffisamment de représentants de chaque classe dans les données d'apprentissage pour pouvoir distinguer les différentes classes. Si une classe représente moins de 10 % des données, les données sont considérées comme déséquilibrées. La classe sous-représentée est appelée _classe minoritaire_.

De nombreux problèmes de PdM sont confrontés à de tels jeux de données déséquilibrés, où une classe est fortement sous-représentée par rapport à l'autre classe, ou aux classes. Dans certains cas, la classe minoritaire peut ne représenter que 0,001 % du total des points de données. Le déséquilibre des classes ne concerne pas uniquement la PdM. On rencontre des problèmes similaires dans d’autres domaines où les défaillances et les anomalies sont rares, par exemple, la détection des fraudes et l’intrusion dans des réseaux. Ces défaillances sont à l’origine des exemples de classes minoritaires.

Le déséquilibre des classes dans les données compromet les performances de la plupart des algorithmes d’apprentissage standard, car ils tendent à minimiser le taux d’erreur global. Pour un jeu de données avec 99 % d’exemples négatifs et 1 % d’exemples positifs, un modèle peut avoir une précision de 99 % en étiquetant toutes les instances comme négatives. Mais le modèle classifiera mal tous les exemples positifs. Ainsi, malgré une très grande précision, l’algorithme ne sera pas utile. Par conséquent, les mesures d'évaluation classiques, telles que la _précision globale sur le taux d'erreur_ , sont insuffisantes pour un apprentissage déséquilibré. Face à des jeux de données déséquilibrés, d’autres métriques sont utilisés pour l’évaluation du modèle :
- Precision
- Rappel
- Score F1
- ROC (caractéristiques d’opération du destinataire) ajustées aux coûts

Pour plus d’informations sur ces métriques, consultez l’[évaluation des modèles](#model-evaluation).

Toutefois, il existe des méthodes qui permettent de résoudre le problème de déséquilibre des classes. Les deux plus importantes sont les _techniques d’échantillonnage_ et l’ _apprentissage sensible au coût_.

#### <a name="sampling-methods"></a>Méthodes d’échantillonnage
L’apprentissage déséquilibré implique l’utilisation de méthodes d’échantillonnage pour transformer le jeu de données d’apprentissage en jeu de données équilibrée. Les méthodes d’échantillonnage ne doivent pas être appliquées au jeu de test. Bien qu'il existe de nombreuses méthodes d'échantillonnage différentes, les plus efficaces sont le _suréchantillonnage_ et le _sous-échantillonnage aléatoires_.

Le _suréchantillonnage aléatoire_ implique la sélection d’un échantillon aléatoire dans la classe minoritaire, la réplication de ces exemples et leur ajout au jeu de données d’apprentissage. Par conséquent, le nombre d'exemples dans la classe minoritaire augmente et finit par équilibrer le nombre d'exemples dans les différentes classes. Un inconvénient du suréchantillonnage est que plusieurs instances de certains exemples entraînent une trop grande spécificité du classifieur, conduisant à un surajustement. Le modèle peut indiquer une grande précision d’apprentissage, mais ses performances sur des données de test non consultées peuvent ne pas être optimales.

À l'inverse, le _sous-échantillonnage aléatoire_ consiste à sélectionner un échantillon aléatoire dans la classe majoritaire et à supprimer ces exemples du jeu de données d’apprentissage. Toutefois, la suppression d’exemples dans la classe majoritaire peut aboutir à ce que le classifieur manque d’importants concepts appartenant à la classe majoritaire. _L'échantillonnage hybride_ , selon lequel la classe minoritaire est suréchantillonnée et la classe majoritaire sous-échantillonnée en même temps, est une autre approche viable.

Il existe de nombreuses techniques d’échantillonnage sophistiquées. La technique choisie varie selon les propriétés des données et les résultats des expériences itératives obtenus par le scientifique des données.

#### <a name="cost-sensitive-learning"></a>Apprentissage sensible au coût
Dans la PdM, les défaillances qui constituent la classe minoritaire sont plus intéressants que des exemples normaux. Par conséquent, on se concentre sur les performances de l’algorithme en cas de défaillances Prédire de manière incorrecte qu’une classe positive sera une classe négative peut coûter plus que l’inverse. Cette situation est généralement appelée perte inégale ou coût asymétrique d’éléments mal classés dans différentes classes. Le classifieur idéal doit offrir une grande précision de prédiction sur la classe minoritaire, sans compromettre la précision de la classe majoritaire.

Il existe plusieurs manières d’atteindre cet équilibre. Pour atténuer le problème des pertes inégales, attribuez un coût élevé à la classification incorrecte de la classe minoritaire et essayez de minimiser le coût total. Les algorithmes comme _SVM (machine à vecteur de support)_ adoptent cette méthode par nature, en autorisant que le coût des exemples positifs et négatifs soit spécifié pendant l’apprentissage. De même, des méthodes d’amélioration telles que des _arbres de décision optimisés_ affichent généralement de bonnes performances en cas de données déséquilibrées.

## <a name="model-evaluation"></a>Évaluation de modèle
L’erreur de classification est un problème sérieux pour les scénarios de PdM où le coût des fausses alertes est élevé pour l’entreprise. Par exemple, la décision de ne pas faire décoller un avion basée sur une prédiction incorrecte d’une défaillance de moteur peut perturber les horaires et les plans de voyage. Déconnecter une machine sur une ligne d’assemblage peut entraîner une perte de revenus. L’évaluation du modèle avec les bonnes métriques de performances par rapport aux nouvelles données de test est donc essentielle.

Des métriques de performances standard utilisées pour évaluer des modèles de PdM sont décrites ci-dessous :

- La [justesse](https://en.wikipedia.org/wiki/Accuracy_and_precision) est la métrique la plus utilisée pour décrire les performances d’un classifieur. Mais la précision est sensible aux distributions de données et est une mesure inefficace pour les scénarios avec des jeux de données déséquilibrés. D’autres métriques sont utilisées à la place. Des outils tels que la [matrice de confusion](https://en.wikipedia.org/wiki/Confusion_matrix) sont utilisés pour calculer et analyser la précision du modèle.
- La [précision](https://en.wikipedia.org/wiki/Precision_and_recall) de modèles de PdM se rapporte à la fréquence de fausses alertes. Une précision inférieure du modèle correspond généralement à un taux plus élevé de fausses alertes.
- Les taux de [rappel](https://en.wikipedia.org/wiki/Precision_and_recall) indiquent le nombre de défaillances dans le jeu de test qui ont été correctement identifiées par le modèle. Des taux de rappel plus élevés signifient que le modèle parvient à identifier les défaillances réelles.
- Le [score F1](https://en.wikipedia.org/wiki/F1_score) est la moyenne harmonique de la précision et du rappel, dont la valeur est comprise entre 0 (la pire) et 1 (la meilleure).

Pour la classification binaire
- [Les courbes ROC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) sont également une métrique populaire. Dans les courbes ROC, les performances du modèle sont interprétées sur la base d’un point de fonctionnement fixe sur la courbe.
- Mais, pour les problèmes de PdM, _les tables déciles_ et les _graphiques de courbes d’élévation_ sont plus explicites. Ils se concentrent uniquement sur la classe positive (défaillances) et fournissent une image plus complexe des performances de l’algorithme que des courbes ROC.
  - Les _tables de déciles_ sont créées à l’aide d’exemples de tests dans l’ordre décroissant des probabilités de défaillance. Les échantillons commandés sont ensuite regroupés en déciles (10 % des échantillons avec la plus grande probabilité, puis 20 %, 30 %, etc.). Le ratio (taux positif vrai)/(ligne de base aléatoire) de chaque décile aide à évaluer les performances de l’algorithme pour chaque décile. La ligne de base aléatoire prend les valeurs 0,1, 0,2, etc.
  - Le tracé [Graphiques de courbes d’élévation](http://www2.cs.uregina.ca/~dbd/cs831/notes/lift_chart/lift_chart.html) trace le taux positif vrai des déciles par rapport au taux positif vrai aléatoire de tous les déciles. Les premiers déciles sont généralement les résultats les plus intéressants, car ils affichent les gains les plus élevés. Les premiers déciles peuvent également représenter les éléments « à risque » quand ils sont utilisés pour la PdM.

## <a name="model-operationalization-for-predictive-maintenance"></a>Opérationnalisation du modèle pour la maintenance prédictive

La  science des données ne présente d’avantage que si le modèle d’apprentissage est rendu opérationnel. Autrement dit, le modèle doit être déployé dans des systèmes d’entreprise pour élaborer des prédictions basées sur des données nouvelles n’ayant pas encore été consultées.  Les nouvelles données doivent être exactement conformes à la _signature du modèle_ ayant effectué l’apprentissage en deux points :
- toutes les fonctionnalités doivent être présentes dans chaque instance logique (par exemple, une ligne dans une table) des nouvelles données.
- les nouvelles données doivent être prétraitées, et chacune des caractéristiques doit être conçue exactement de la même façon que les données d’apprentissage.

Le processus ci-dessus est établi de nombreuses manières dans les publications universitaires et industrielles. Toutefois, toutes les instructions suivantes ont la même signification :
- _Évaluer de nouvelles données_ à l’aide du modèle
- _Appliquer le modèle_ aux nouvelles données
- _Opérationnaliser_ le modèle
- _Déployer_ le modèle
- _Exécuter le modèle_ avec de nouvelles données

Comme indiqué précédemment, l’opérationnalisation du modèle pour la PdM diffère de celle de ses pairs. Les scénarios qui impliquent la détection d’anomalies et la détection de défaillances implémentent généralement le _scoring en ligne_ (également appelé _scoring en temps réel_ ). Ici, le modèle _évalue_ chaque enregistrement entrant et renvoie une prédiction. Pour la détection d’anomalies, la prédiction est une indication du fait qu’une anomalie s’est produite (exemple : SVM à une classe). Pour la détection de défaillances, ce serait le type ou la classe de défaillance.

En revanche, la PdM implique _scoring par lot_. Pour se conformer à la signature du modèle, les caractéristiques dans les nouvelles données doivent être conçues de la même manière que les données d’apprentissage. Pour les jeux de données volumineux qui sont généralement utilisés pour les nouvelles données, les fonctionnalités sont agrégées sur les fenêtres de temps et notées dans le lot. Le scoring par lot est généralement effectué dans des systèmes distribués comme [Spark](https://spark.apache.org/) ou [Azure Batch](../../batch/batch-service-workflow-features.md). Il existe deux alternatives dont aucune n’est optimale :
- Les moteurs de diffusion en continu de données prennent en charge l’agrégation sur des fenêtres dans la mémoire. On peut donc considérer qu’ils prennent en charge le scoring en ligne. Mais ces systèmes sont adaptés à des données denses dans des fenêtres de temps étroites, ou à des éléments épars sur des fenêtres plus larges. Il est possible qu’ils ne se mettent pas correctement à l’échelle sur des fenêtres de temps plus larges, comme on l’a vu dans des scénarios de PdM.
- Si le scoring par lot n’est pas disponible, la solution consiste à adapter un scoring en ligne pour gérer les nouvelles données en une seule fois dans les lots de petite taille.

## <a name="solution-templates-for-predictive-maintenance"></a>Modèles de solutions pour la maintenance prédictive

La dernière section de ce guide fournit une liste de modèles de solutions de PdM, des tutoriels et des expériences implémentées dans Azure. Ces applications de PdM peuvent être déployées dans un abonnement Azure en quelques minutes dans certains cas. Elles peuvent être utilisées comme démonstrations de preuves de concept, comme bacs à sables pour faire des essais avec des alternatives, ou comme accélérateurs pour des implémentations de production réelles. Ces modèles sont situés dans la [galerie Azure AI](https://gallery.azure.ai) ou dans [Azure GitHub](https://github.com/Azure). Ces différents échantillons seront déployés dans ce modèle de solution au fil du temps.

| # | Intitulé | Description |
|--:|:------|-------------|
| 2 | [Échantillon de solution de maintenance prédictive Azure](https://github.com/Azure/AI-PredictiveMaintenance) | Modèle de solution open source illustrant la modélisation Azure Machine Learning et une infrastructure Azure complète pouvant prendre en charge des scénarios de maintenance prédictive dans le contexte de la supervision à distance de l’IoT. |
| 3 | [Apprentissage profond pour la maintenance prédictive](https://github.com/Azure/MachineLearningSamples-DeepLearningforPredictiveMaintenance) | Azure Notebook avec une solution de démonstration de l’utilisation des réseaux LSTM (Long Short-Term Memory) (une classe de réseaux de neurones récurrents) pour une maintenance prédictive, avec un [billet de blog sur cet échantillon](https://azure.microsoft.com/blog/deep-learning-for-predictive-maintenance).|
| 4 | [Maintenance prédictive Azure pour l’aérospatiale](https://gallery.azure.ai/Solution/Predictive-Maintenance-for-Aerospace-1) | Un des premiers modèles de solution de PdM basés sur Azure ML v1.0 pour la maintenance des avions. Ce guide résulte de ce projet. |
| 5 | [Kit d’outils d’IA Azure pour IoT Edge](https://github.com/Azure/ai-toolkit-iot-edge) | IA dans IoT Edge avec TensorFlow ; le kit d’outils réunit des modèles Deep Learning dans des conteneurs Docker compatibles Azure IoT Edge et les présente sous forme d’API REST.
| 6 | [Microsoft Azure IoT – Maintenance prédictive](https://github.com/Azure/azure-iot-predictive-maintenance) | Azure IoT Suite PCS – Solution préconfigurée Modèle de PdM d’avions avec IoT Suite. [Un autre document](../../iot-accelerators/quickstart-predictive-maintenance-deploy.md) et [une procédure pas à pas](../../iot-accelerators/iot-accelerators-predictive-walkthrough.md) associés au même projet. |
| 7 | [Modèle de maintenance prédictive avec les services SQL Server R](https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Template-with-SQL-Server-R-Services-1) | Démonstration du scénario de durée de vie utile résiduelle basée sur les services de R. |
| 8 | [Guide de modélisation de maintenance prédictive](https://gallery.azure.ai/Collection/Predictive-Maintenance-Modelling-Guide-1) | Caractéristique de jeu de données de maintenance d’avions conçue en utilisant R avec des [expériences](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Modelling-Guide-Experiment-1), des [jeux de données](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Modelling-Guide-Data-Sets-1) et des notebooks et [expériences](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Step-1-of-3-data-preparation-and-feature-engineering-2) Azure dans AzureML v1.0|

## <a name="training-resources-for-predictive-maintenance"></a>Ressources de formation pour la maintenance prédictive

En plus du contenu et de formations sur les concepts généraux et la mise en pratique de l’intelligence artificielle, Microsoft Azure offre des parcours d’apprentissage pour les concepts fondamentaux de techniques de maintenance prédictive.

| Ressources de formation  | Disponibilité |
|:-------------------|--------------|
| [Parcours d’apprentissage pour la maintenance prédictive avec des arborescences et des forêts aléatoires](https://aischool.microsoft.com/learning-paths/1H5vH5wAYcAy88CoQWQcA8) | Public | 
| [Parcours d’apprentissage pour la maintenance prédictive avec l’apprentissage profond (deep learning)](https://aischool.microsoft.com/learning-paths/FSIXxYkOGcauo0eUO8qAS) | Public |
| [Développeur d’IA sur Azure](https://azure.microsoft.com/training/learning-paths/azure-ai-developer) | Public |
| [Microsoft AI School](https://aischool.microsoft.com/learning-paths) | Public |
| [Apprentissage AI Azure à partir de GitHub](https://github.com/Azure/connectthedots/blob/master/readme.md) | Public |
| [LinkedIn Learning](https://www.linkedin.com/learning) | Public |
| [Webinaires Microsoft AI sur YouTube](https://www.youtube.com/watch?v=NvrH7_KKzoM&t=4s) | Public |
| [Présentation de Microsoft AI](https://channel9.msdn.com/Shows/AI-Show) | Public |
| [LearnAI@MS](https://learnanalytics.microsoft.com) | Partenaires |
| [Microsoft Partner Network](https://partner.microsoft.com/training/training-center) | Partenaires |

De plus, des MOOCS gratuits (cours en ligne ouverts à tous) sur l’IA sont proposés par des universités comme Stanford et MIT, ainsi que par d’autres établissements d’enseignement.