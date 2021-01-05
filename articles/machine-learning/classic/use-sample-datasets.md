---
title: 'ML Studio (classique) : Utiliser les exemples de jeux de données – Azure'
description: Descriptions des jeux de données utilisés dans les exemples de modèles inclus dans Machine Learning Studio (classique). Vous pouvez utiliser ces exemples de jeux de données pour vos expériences.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: sample
author: likebupt
ms.author: keli19
ms.custom: previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 01/19/2018
ms.openlocfilehash: 737c63c02ea852e8cf17e2d520ca91b05a5ba12b
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325618"
---
# <a name="use-the-sample-datasets-in-azure-machine-learning-studio-classic"></a>Utilisation des exemples de jeux de données dans Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


[top]: #machine-learning-sample-datasets

Quand vous créez un espace de travail dans Azure Machine Learning Studio (classique), vous disposez par défaut d’un certain nombre d’exemples de jeux de données et d’expériences. Plusieurs de ces exemples de jeux de données sont utilisés par les exemples de modèles [d’Azure AI Gallery](https://gallery.azure.ai/). D’autres sont inclus comme exemples des différents types de données généralement utilisées dans Machine Learning.

Certains de ces jeux de données sont disponibles dans le Stockage Blob Azure. Le tableau ci-dessous fournit un lien direct vers ces jeux de données. Vous pouvez utiliser ces jeux de données dans vos expériences à l’aide du module [Importer les données][import-data].

Les autres exemples de jeux de données sont disponibles dans votre espace de travail, dans **Jeux de données enregistrés**. Vous le trouverez dans la palette de modules, à gauche du canevas de l’expérience, dans Machine Learning Studio (classique).
Vous pouvez utiliser un de ces jeux de données dans votre propre expérience en le faisant glisser sur le canevas de l'expérience.

## <a name="datasets"></a>Groupes de données

<table>

<tr>
  <th>Nom du jeu de données</th>
  <th>Description du jeu de données</th>
</tr>

<tr>
  <td>Jeu de données Adult Census Income Binary Classification</td>
  <td>
Sous-ensemble de la base de données Census de 1994, qui recense les adultes de plus de 16 ans en activité avec un index des revenus ajustés supérieur à 100.
<p></p>
<b>Utilisation :</b> classifier des personnes en utilisant des données démographiques pour prédire si une personne gagne plus de 50 000 $ par an.
<p></p>
<b>Recherche associée :</b> Kohavi, R., Becker, B., (1996). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Jeu de données des codes de l'aéroport</td>
  <td>
Codes des aéroports des États-Unis.
<p></p>
Ce jeu de données contient une seule ligne par aéroport des États-Unis. Il indique le numéro d'identification de l'aéroport et son nom, ainsi que la ville et l'état dans lesquels il se trouve.
  </td>
</tr>

<tr>
  <td>Données sur le prix des véhicules automobiles (brutes)</td>
  <td>
Informations sur les véhicules automobiles par marque et modèle, incluant le prix, des caractéristiques telles que le nombre de cylindres et de litres au 100, et une note de risque d'assurance.
<p></p>
La note de risque est initialement associée au prix de l’automobile. Elle est ensuite ajustée en fonction du risque réel selon un processus que les actuaires connaissent sous le nom de symbolisation. La valeur +3 indique que le véhicule est à risque et la valeur -3 qu’il est plutôt sûr.
<p></p>
<b>Utilisation :</b> prédire le score de risque en fonction des caractéristiques, en utilisant une classification de régression ou multivariée. 
<p></p>
<b>Recherche associée :</b> Schlimmer, J.C. (1987). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Jeu de données de location de vélos de l'UCI</td>
  <td>
Le jeu de données de location de vélo de l'UCI est basé sur les données réelles de la société Capital Bikeshare qui assure l'entretien du réseau de location de vélos à Washington DC.
<p></p>
Ce jeu de données comporte une seule ligne pour chaque heure de chaque jour en 2011 et 2012, soit un total de 17 379 lignes. Le nombre de locations de vélos par heure varie entre 1 et 977.

  </td>
</tr>

<tr>
  <td>Image RVB de Bill Gates</td>
  <td>
Fichier image accessible au public et converti en données CSV.
<p></p>
Le code utilisé pour convertir l’image se trouve dans la page d’informations sur le modèle de <strong>Quantification des couleurs à l’aide de l’algorithme des k-moyennes</strong>.
  </td>
</tr>

<tr>
  <td>Données sur le don de sang</td>
  <td>
Sous-ensemble de la base de données des donneurs de sang du centre de transfusion sanguine de Hsin-Chu City à Taiwan.
<p></p>
Les données associées aux donneurs incluent le nombre de mois depuis le dernier don et la fréquence, ou le nombre total de dons, la durée écoulée depuis le dernier don et la quantité de sang donnée.
<p></p>
<b>Utilisation :</b> l’objectif est de prédire par la classification si le donneur a donné du sang en mars 2007, 1 indiquant un donneur pendant la période ciblée et 0 une personne n’ayant pas fait de don. 
<p></p>
<b>Recherche associée :</b> Yeh, I.C., (2008). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science
<p></p>
Yeh, I-Cheng, Yang, King-Jang et Ting, Tao-Ming, « Détection des connaissances sur le modèle RFM à l’aide de la séquence Bernoulli, » Expert Systems avec Applications, 2008, <a href="https://dx.doi.org/10.1016/j.eswa.2008.07.018">https://dx.doi.org/10.1016/j.eswa.2008.07.018</a>
  </td>
</tr>

<tr>
  <td>Données sur le cancer du sein</td>
  <td>
Un des trois jeux de données sur le cancer fournis par l’Oncology Institute, fréquemment cité dans la littérature sur l’apprentissage automatique. Combine des informations de diagnostic et des caractéristiques d'analyse de laboratoire sur environ 300 échantillons de tissu.
<p></p>
<b>Utilisation :</b> classifier le type de cancer en se basant sur 9 attributs, certains étant linéaires et d’autres catégoriels. 
<p></p>
<b>Recherche associée :</b> Wohlberg, W.H., Street, W.N., & Mangasarian, O.L. (1995). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Caractéristiques du cancer du sein <td>
Le jeu de données contient des informations sur 102 000 régions suspectes (candidats) d’images radio, chacune décrite par 117 caractéristiques. Ces caractéristiques sont propriétaires et leur signification n’est pas révélée par les créateurs du jeu de données (Siemens Healthcare). 
  </td>
</tr>

<tr>
  <td>Informations sur le cancer du sein</td>
  <td>
Le jeu de données contient des informations complémentaires sur chaque région suspecte dans l’image radio. Chaque exemple donne des informations (par exemple, étiquette, ID du patient, coordonnées du correctif par rapport à la totalité de l’image) sur le numéro de ligne correspondant dans le jeu de données Caractéristiques cancer du sein. Chaque patient présente un certain nombre d’exemples. Pour les patients atteints de cancer, certains exemples sont positifs et d’autres négatifs. Pour les patients non atteints de cancer, tous les exemples sont négatifs. Le jeu de données contient 102 000 exemples. Le jeu de données est biaisé, avec 0,6 % des points positifs, le reste étant négatif. Ce jeu de données a été fourni par Siemens Healthcare.
  </td>
</tr>

<tr>
  <td>Étiquettes de l'appétence CRM partagées</td>
  <td>
Étiquettes provenant du KDD Cup 2009 Customer Relationship Prediction Challenge (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_appetency.labels">orange_small_train_appetency.labels</a>).
  </td>
</tr>

<tr>
  <td>Étiquettes de l'attrition CRM partagées</td>
  <td>
Étiquettes provenant du KDD Cup 2009 Customer Relationship Prediction Challenge (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_churn.labels">orange_small_train_churn.labels</a>).
  </td>
</tr>

<tr>
  <td>Jeu de données CRM partagé</td>
  <td>
Ces données proviennent du KDD Cup 2009 Customer Relationship Prediction Challenge (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train.data.zip">orange_small_train.data.zip</a>).
<p></p>
Le jeu de données contient 50 000 clients de la société de télécoms française Orange. Chaque client possède 230 caractéristiques rendues anonymes, dont 190 sont numériques et 40 sont catégorielles. Elles sont très fragmentées.
  </td>
</tr>

<tr>
  <td>Étiquettes de vente incitative CRM partagées</td>
  <td>
Étiquettes provenant du KDD Cup 2009 Customer Relationship Prediction Challenge (<a href="http://www.sigkdd.org/site/2009/files/orange_large_train_upselling.labels">orange_large_train_upselling.labels</a>).
  </td>
</tr>

<tr>
  <td>Données de régression sur l’efficacité énergétique</td>
  <td>
Collection de profils d'énergie simulés, basée sur 12 formes différentes de bâtiments. Huit caractéristiques différencient les bâtiments, notamment la zone de vitrage, sa distribution et l’orientation.
<p></p>
<b>Utilisation :</b> utiliser la régression ou la classification pour prédire la notation de l’efficacité énergétique sous la forme de l’une des deux réponses en valeur réelle. Pour la classification multiclasse, arrondit la variable de la réponse à l’entier le plus proche. 
<p></p>
<b>Recherche associée :</b> Xifara, A. & Tsanas, A. (2012). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Données relatives aux vols retardés</td>
  <td>
Données de ponctualité des vols passagers provenant de la collection de données TranStats du ministère Ministère des transports des États-Unis (<a href="https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">à l’heure</a>).
<p></p>
Le jeu de données couvre la période d’avril à octobre 2013. Avant son chargement dans Azure Machine Learning Studio (classique), le jeu de données a été traité comme suit :
<ul>
  <li>Le jeu de données a été filtré afin de prendre uniquement en compte les 70 aéroports les plus fréquentés aux États-Unis.</li>
  <li>Les vols annulés ont été considérés comme ayant été retardés de plus de 15 minutes</li>
  <li>Les vols déviés ont été supprimés.</li>
  <li>Les colonnes suivantes ont été sélectionnées : Year, Month, DayofMonth, DayOfWeek, Carrier, OriginAirportID, DestAirportID, CRSDepTime, DepDelay, DepDel15, CRSArrTime, ArrDelay, ArrDel15, Canceled</li>
</ul>
</td>
</tr>

<tr>
  <td>Performance concernant les vols à l'heure (brutes)</td>
  <td>
Enregistrements des arrivées et départs de vols aux États-Unis à compter d’octobre 2011.
<p></p>
<b>Utilisation :</b> prédire les retards des vols. 
<p></p>
<b>Recherche associée :</b> Provenant du département des transports des États-Unis <a href="https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time</a>.
  </td>
</tr>

<tr>
  <td>Données sur les feux de forêt</td>
  <td>
Contient des données météorologiques, par exemple, les indices de température et d’humidité et la vitesse du vent. Les données proviennent d’une région du Nord-Est du Portugal et d’archives d’incendies de forêt.
<p></p>
<b>Utilisation :</b> il s’agit d’une tâche difficile de régression, dont l’objectif est de prédire la zone brûlée par des feux de forêt. 
<p></p>
<b>Recherche associée :</b> Cortez, P., & Morais, A. (2008). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science
<p></p>
[Cortez et Morais, 2007] P. Cortez et A. Morais. A Data Mining Approach to Predict Forest Fires using Meteorological Data. Aux éditions J. Neves, M. F. Santos et J. Machado, New Trends in Artificial Intelligence, Proceedings of the 13th EPIA 2007 - Conférence portugaise sur l’intelligence artificielle, décembre, Guimarães, Portugal, pp. 512-523, 2007. APPIA, ISBN-13 978-989-95618-0-9. Disponibles sur : <a href="http://www.dsi.uminho.pt/~pcortez/fires.pdf">http://www.dsi.uminho.pt/~pcortez/fires.pdf</a>.
  </td>
</tr>

<tr>
  <td>Jeu de données d'UCI pour une carte de crédit allemande</td>
  <td>
Jeu de données UCI Statlog (carte de crédit allemande) (<a href="https://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">Statlog+German+Credit+Data</a>), utilisant le fichier german.data.
<p></p>
Le jeu de données classe les gens, décrits par un ensemble d’attributs, par risque de crédit (faible ou élevé). Chaque exemple représente une personne. Il y a 20 caractéristiques, numériques et catégoriques, ainsi qu’une étiquette binaire (la valeur du risque de crédit). Les entrées avec un risque de crédit élevé portent une étiquette = 2, tandis que les entrées avec un risque de crédit faible portent une étiquette = 1. Le coût d’une erreur de classification d’un risque de crédit faible en risque élevé est de 1 tandis qu’il est de 5 dans le cas inverse (classification d’un risque de crédit élevé en risque faible).
  </td>
</tr>

<tr>
  <td>Titres de films IMDB</td>
  <td>
Ce jeu de données contient des informations sur les films évalués dans des tweets sur Twitter : ID de film IMDB, titre du film, genre et année de production. Ce jeu de données contient 17 000 films. Le jeu de données provient du document « S. Dooms, T. De Pessemier et L. Martens. MovieTweetings: a Movie Rating Dataset Collected From Twitter. Workshop on Crowdsourcing and Human Computation for Recommender Systems, CrowdRec at RecSys 2013. »
  </td>
</tr>

<tr>
  <td>Données sur deux classes d'iris</td>
  <td>
Il s’agit sans doute de l’une des bases de données les plus réputées dans la littérature de la reconnaissance des formes. Le jeu de données est relativement petit, avec 50 exemples de mesures de pétales de trois variétés d’iris.
<p></p>
<b>Utilisation :</b> prédire le type d’iris à partir des mesures.  
<p></p>
<b>Recherche associée :</b> Fisher, R.A. (1988). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Tweets de films</td>
  <td>
Ce jeu de données est une version étendue du jeu de données Movie Tweetings. Il contient 170 000 évaluations de film, tirées de tweets structurés publiés sur Twitter. Chaque instance correspond à un tweet et constitue un tuple : ID utilisateur, ID IMDB, évaluation, horodatage, nombre de favoris pour ce tweet et nombre de retweets pour ce tweet. Ce jeu de données a été fourni par A. Said, S. Dooms, B. Loni et D. Tikk dans le cadre du Recommender Systems Challenge 2014.
  </td>
</tr>

<tr>
  <td>Données sur la quantité de litres au 100 pour différents véhicules automobiles</td>
  <td>
Ce jeu de données diffère légèrement de la version fournie par la bibliothèque StatLib de l’université de Carnegie Mellon. Le jeu de données a été utilisé lors de l'exposition de l'American Statistical Association de 1983.
<p></p>
Les données répertorient la consommation de carburant de différentes automobiles, en miles par gallon. Elles comprennent également des informations telles que le nombre de cylindres, la cylindrée, la puissance, le poids total et l’accélération.
<p></p>
<b>Utilisation :</b> prédire l’économie de carburant à partir de trois attributs discrets multivalués et de cinq attributs continus. 
<p></p>
<b>Recherche associée :</b> StatLib, Carnegie Mellon University, (1993). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Jeu de données sur la classification binaire du diabète chez les indiens Pima</td>
  <td>
Sous-ensemble de la base de données du National Institute of Diabetes and Digestive and Kidney Diseases. Le jeu de données a été filtré pour s’intéresser aux patientes féminines des descendants des indiens Pima. Les données incluent des données médicales comme les taux de glucose et d'insuline, et des facteurs liés au style de vie.
<p></p>
<b>Utilisation :</b> prédire si le sujet a du diabète (classification binaire). 
<p></p>
<b>Recherche associée :</b> Sigillito, V. (1990). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml  »</a>. Irvine, CA: University of California, School of Information and Computer Science </td>
</tr>

<tr>
  <td>Données sur les clients de restaurant</td>
  <td>
Jeu de données sur les clients, comprenant des données démographiques et des préférences.
<p></p>
<b>Utilisation :</b> utiliser ce jeu de données avec les deux autres jeux de données sur les restaurants, pour former et tester un système de recommandation. 
<p></p>
<b>Recherche associée :</b> Bache, K. and Lichman, M. (2013). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>Données sur les caractéristiques de restaurants</td>
  <td>
Jeu de métadonnées sur des restaurants et leurs caractéristiques, comme le type de gastronomie, le style de lieu et l'emplacement.
<p></p>
<b>Utilisation :</b> utiliser ce jeu de données avec les deux autres jeux de données sur les restaurants, pour former et tester un système de recommandation. 
<p></p>
<b>Recherche associée :</b> Bache, K. and Lichman, M. (2013). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>Notations de restaurants</td>
  <td>
Contient les notes attribuées par les consommateurs à des restaurants sur une échelle de 0 à 2.
<p></p>
<b>Utilisation :</b> utiliser ce jeu de données avec les deux autres jeux de données sur les restaurants, pour former et tester un système de recommandation. 
<p></p>
<b>Recherche associée :</b> Bache, K. and Lichman, M. (2013). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science.
  </td>
</tr>

<tr>
  <td>Jeu de données multiclasse de recuit d'acier</td>
  <td>
Ce jeu de données contient une série d’enregistrements d’essais de recuit de l’acier. Il contient les attributs physiques (largeur, épaisseur, type : bobine, feuille, etc.) des types d’acier obtenus.
<p></p>
<b>Utilisation :</b> prédire un des deux attributs de classe numérique : robustesse ou force. Vous pouvez également analyser les corrélations entre attributs.
<p></p>
Les qualités d'acier répondent à un standard défini par la SAE et d'autres organisations. Vous recherchez une qualité d’acier spécifique (variable de classe) et vous cherchez à comprendre les valeurs nécessaires. 
<p></p>
<b>Recherche associée :</b> Sterling, D. & Buntine, W. (NA). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information and Computer Science
<p></p>
Vous trouverez ici un guide utile sur les qualités d’acier : <a href="https://www.steamforum.com/pictures/Outokumpu-steel-grades-properties-global-standards.pdf">https://www.steamforum.com/pictures/Outokumpu-steel-grades-properties-global-standards.pdf</a>
  </td>
</tr>

<tr>
  <td>Données sur les télescopes</td>
  <td>
Enregistrement sur les sursauts de particules gamma de haute énergie et le bruit de fond, à partir de simulations de Monte-Carlo.
<p></p>
L’objectif de la simulation était d’améliorer la précision de télescopes gamma Tcherenkov atmosphériques au sol. Pour cela, des méthodes statistiques ont été utilisées pour faire la différence entre le signal souhaité (douches à effet Tcherenkov) et le bruit de fond (douches hadroniques dues à des rayons cosmiques dans la haute atmosphère).
<p></p>
Les données ont été prétraitées pour créer un cluster allongé avec l'axe long orienté vers le centre de la caméra. Les caractéristiques de cette ellipse (souvent appelées paramètres de Hillas) font partie des paramètres d’image qui peuvent servir pour la discrimination.
<p></p>
<b>Utilisation :</b> prédire si l’image d’une douche représente un signal ou un bruit de fond.
<p></p>
<b>Remarques :</b> La simple précision de la classification n’est pas significative pour ces données, car classifier un événement de bruit de fond comme événement de signal est pire que classifier un événement de signal comme événement de bruit de fond. Pour comparer différents classificateurs, il faut utiliser la courbe ROC. La probabilité d’accepter un événement de bruit de fond comme signal doit être inférieure à un des seuils suivants : 0,01, 0,02, 0,05, 0,1 ou 0,2.
<p></p>
Notez également que le nombre d’événements de bruit de fond (h, pour douches hadroniques) est sous-estimé. Dans les mesures réelles, la classe h ou classe bruitée représente la majorité des événements. 
<p></p>
<b>Recherche associée :</b> Bock, R.K. (1995). Référentiel Machine Learning UCI <a href="https://archive.ics.uci.edu/ml">https://archive.ics.uci.edu/ml</a>. Irvine, CA: University of California, School of Information </td>
</tr>

<tr>
  <td>Jeu de données météorologiques</td>
  <td>
Observations météorologiques terrestres effectuées toutes les heures par la NOAA (<a href="https://az754797.vo.msecnd.net/data/WeatherDataset.csv">données fusionnées, de 201304 à 201310</a>).
<p></p>
Les données météorologiques couvrent les observations effectuées depuis les stations météo des aéroports, entre les mois d’avril et octobre 2013. Avant son chargement dans Azure Machine Learning Studio (classique), le jeu de données a été traité comme suit :
<ul>
  <li>Les identifiants des stations météo ont été alignés sur les identifiants des aéroports correspondants.</li>
  <li>Les stations météo non associées à un des 70 aéroports sélectionnés ont été supprimées.</li>
  <li>La colonne Date a été fractionnée en plusieurs colonnes distinctes indiquant l'année, le mois et le jour.</li>
  <li>Les colonnes suivantes avaient été sélectionnées : AirportID, Year, Month, Day, Time, TimeZone, SkyCondition, Visibility, WeatherType, DryBulbFarenheit, DryBulbCelsius, WetBulbFarenheit, WetBulbCelsius, DewPointFarenheit, DewPointCelsius, RelativeHumidity, WindSpeed, WindDirection, ValueForWindCharacter, StationPressure, PressureTendency, PressureChange, SeaLevelPressure, RecordType, HourlyPrecip, Altimeter</li>
</ul>
  </td>
</tr>

<tr>
  <td>Jeu de données Wikipedia concernant le SP 500</td>
  <td>
Les données sont extraites de Wikipédia (<a href="https://www.wikipedia.org/">https://www.wikipedia.org/</a>), notamment d’articles sur chaque société S&P 500 et sont stockées sous forme de données XML.
<p></p>
Avant son chargement dans Azure Machine Learning Studio (classique), le jeu de données a été traité comme suit :
<ul>
  <li>Extraction du contenu textuel de chaque société particulière</li>
  <li>Suppression de la mise en forme wiki</li>
  <li>Suppression des caractères non alphanumériques</li>
  <li>Conversion de tout le texte en minuscule</li>
  <li>Ajout d'autres catégories de sociétés connues</li>
</ul>
<p></p>
Notez que pour certaines sociétés, aucun article n’a pu être trouvé, donc le nombre d’enregistrements est inférieur à 500.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/direct_marketing.csv">direct_marketing.csv</a></td>
  <td>
Le jeu de données contient les informations client et les indications relatives à leurs réponses à une campagne de publipostage direct. Chaque ligne représente un client. Le jeu de données contient neuf caractéristiques sur les données démographiques et le comportement passé des utilisateurs, ainsi que trois colonnes d’étiquettes (visite, conversion et dépense).  La colonne visite est binaire ; elle indique qu’un client a effectué une visite après la campagne marketing. La conversion indique qu’un client a acheté quelque chose. La dépense est le montant dépensé.  Le jeu de données a été fourni par Kevin Hillstrom dans le cadre du MineThatData E-Mail Analytics And Data Mining Challenge.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_test.csv">lyrl2004_tokens_test.csv</a></td>
  <td>
Caractéristiques des exemples de test du jeu de données de presse RCV1-V2 Reuters. Ce jeu de données contient 781 000 articles de presse, accompagnés de leur ID (première colonne du jeu de données). Chaque article est tokénisé, traité et les mots vides définis. Ce jeu de données est fourni par David. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_train.csv">lyrl2004_tokens_train.csv</a></td>
  <td>
Caractéristiques des exemples d’apprentissage du jeu de données de presse RCV1-V2 Reuters. Ce jeu de données contient 23 000 articles de presse, accompagnés de leur ID (première colonne du jeu de données). Chaque article est tokénisé, traité et les mots vides définis. Ce jeu de données est fourni par David. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a><br></td>
  <td>
Jeu de données provenant du KDD Cup 1999 Knowledge Discovery and Data Mining Tools Competition (<a href="https://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html">kddcup99.html</a>).
<p></p>
Le jeu de données a été téléchargé et stocké dans le stockage d’objets blob Azure (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a>) et il inclut à la fois les jeux de données d’apprentissage et de test. Le jeu de données d'apprentissage comporte environ 126 000 lignes et 43 colonnes, étiquettes comprises. Trois colonnes font partie des informations d’étiquette et 40 colonnes, constituées de caractéristiques numériques et catégorielles/de chaîne, sont disponibles pour s’entraîner avec le modèle. Les données de test contiennent environ 22 500 exemples avec les mêmes 43 colonnes que les données d’apprentissage.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/rcv1-v2.topics.qrels.csv">rcv1-v2.topics.qrels.csv</a></td>
  <td>
Affectation de rubriques aux articles de presse contenus dans le jeu de données de presse Reuters RCV1-V2. Un article de presse peut être affecté à plusieurs rubriques. Le format de chaque ligne est « &lt;nom de rubrique&gt; &lt;ID de document&gt; 1 ». Le jeu de données contient 2,6 millions d’affectations de rubrique. Ce jeu de données est fourni par David. D. Lewis.
  </td>
</tr>

<tr>
  <td><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a></td>
  <td>
Ces données proviennent du KDD Cup 2010 Student Performance Evaluation Challenge (<a href="https://www.kdd.org/kdd-cup/view/kdd-cup-2010-student-performance-evaluation">évaluation des performances des étudiants</a>). Les données utilisées correspondent au jeu d’apprentissage Algebra_2008_2009 (Stamper, J., Niculescu-Mizil, A., Ritter, S., Gordon, G.J., & Koedinger, K.R. (2010). Algebra I 2008-2009. Jeu de données issu du KDD Cup 2010 Educational Data Mining Challenge. Pour le trouver, accédez à <a href="https://pslcdatashop.web.cmu.edu/KDDCup/downloads.jsp">downloads.jsp</a>.
<p></p>
Le jeu de données a été téléchargé et stocké dans le stockage d’objets blob Azure (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a>) et il contient les fichiers journaux d’un système de tutorat des étudiants. Les caractéristiques fournies incluent l’identifiant du problème avec une description sommaire, l’identifiant de l’étudiant, un horodatage et le nombre de tentatives de résolution du problème avant la réussite. Le jeu de données original comporte 8,9 millions d’enregistrements. Ce jeu de données a été échantillonné pour être réduit aux premières 100 000 lignes. Le jeu de données comporte 23 colonnes séparées par des tabulations de divers types : numérique, catégoriel et horodatage.
  </td>
</tr>

</table>

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Démarrez rapidement vos expériences avec des exemples](sample-experiments.md)

<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data