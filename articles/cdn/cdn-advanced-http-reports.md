---
title: Analyser les statistiques d’utilisation avec les rapports HTTP avancés dans Microsoft Azure CDN | Microsoft Docs
description: Découvrez comment créer des rapports HTTP avancés dans Microsoft Azure CDN. Ces rapports fournissent des informations détaillées sur l’activité CDN.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: ef90adc1-580e-4955-8ff1-bde3f3cafc5d
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: e2ef1367cbbb1fb71159c4520c3d1481904c1769
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779320"
---
# <a name="analyze-usage-statistics-with-azure-cdn-advanced-http-reports"></a>Analyser les statistiques d’utilisation avec les rapports HTTP avancés dans Microsoft Azure CDN
## <a name="overview"></a>Vue d’ensemble
Ce document présente les rapports HTTP avancés disponibles dans Microsoft Azure CDN. Ces rapports fournissent des informations détaillées sur l’activité CDN.

[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="accessing-advanced-http-reports"></a>Accès aux rapports HTTP avancés
1. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton de gestion du panneau de profil CDN](./media/cdn-advanced-http-reports/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
2. Pointez sur l’onglet **Analytics** puis sur le menu volant **Rapports HTTP avancés** .  Cliquez sur **HTTP Large Platform** .
   
    ![Portail de gestion CDN - Menu Rapports avancés](./media/cdn-advanced-http-reports/cdn-advanced-reports.png)
   
    Les options de rapport sont affichées.

## <a name="geography-reports-map-based"></a>Rapports géographiques (à partir d’une carte)
Cinq rapports utilisent une carte pour indiquer les régions à partir desquelles sont recensées les demandes de contenu. Il s’agit des rapports World Map, United States Map, Canada Map, Europe Map et Asia Pacific Map.

Chaque rapport dérivé d’une carte classe les entités géographiques par ordre de priorité (par exemple, pays/régions). Une carte est fournie pour vous aider à visualiser les emplacements à partir desquels votre contenu est demandé. Cette visualisation est simplifiée grâce à l’utilisation d’un code couleur sur chaque région en fonction de la quantité de la demande recensée dans la région en question. Les régions ombrées plus claires indiquent une demande de contenu plus faible, tandis que les régions plus foncées illustrent des niveaux plus soutenus.

Des informations détaillées sur le trafic et la bande passante sont fournies pour chaque région directement au-dessous de la carte. Vous pouvez ainsi afficher le nombre total d’accès, le pourcentage d’accès, la quantité totale de données transférées (en Go) et le pourcentage de données transférées pour chaque région. Vous pouvez afficher la description de chacune de ces mesures. Enfin, lorsque vous pointez sur une région (par exemple, pays/région, État ou province), le nom et le pourcentage d’accès recensé dans la région s’afficheront sous la forme d’une info-bulle.

Une brève description est fournie ci-dessous pour chaque type de rapport géographique dérivé d’une carte.

| Nom du rapport | Description |
| --- | --- |
| World Map |Ce rapport vous permet de visualiser la demande mondiale de votre contenu CDN. Chaque pays/région est coloré sur la carte du monde pour indiquer le pourcentage d’accès associé à cette région. |
| United States Map |Ce rapport vous permet de visualiser la demande de votre contenu CDN recensée aux États-Unis. Chaque État est coloré sur la carte pour indiquer le pourcentage d’accès associé à cette région. |
| Canada Map |Ce rapport vous permet de visualiser la demande de votre contenu CDN recensée au Canada. Chaque province est colorée sur la carte pour indiquer le pourcentage d’accès associé à cette région. |
| Europe Map |Ce rapport vous permet de visualiser la demande de votre contenu CDN recensée en Europe. Chaque pays/région est coloré sur cette carte pour indiquer le pourcentage d’accès associé à cette région. |
| Asia Pacific Map |Ce rapport vous permet de visualiser la demande de votre contenu CDN recensée en Asie. Chaque pays/région est coloré sur cette carte pour indiquer le pourcentage d’accès associé à cette région. |

## <a name="geography-reports-bar-charts"></a>Rapports géographiques (graphiques à barres)
Deux rapports supplémentaires fournissent des informations statistiques en fonction de la zone géographique : Top Cities et Top Countries. Ces rapports classent respectivement les villes et les pays/régions selon le nombre d’accès recensés dans ces pays/régions. Lors de la génération de ce type de rapport, un graphique à barres indique les 10 villes principales ou les 10 pays/régions principaux ayant demandé du contenu sur une plateforme spécifique. Ce graphique à barres vous permet d’identifier rapidement les régions qui génèrent le plus grand nombre de demandes autour de votre contenu.

La partie gauche du graphique (axe y) indique le nombre d’accès recensés dans la région spécifiée. Chacun des 10 principales régions est annotée directement sous le graphique (axe x).

### <a name="using-the-bar-charts"></a>Utilisation des graphiques à barres
* Si vous pointez sur une barre, le nom et le nombre total d’accès recensés dans la région s’afficheront sous la forme d’une info-bulle.
* L’info-bulle correspondant au rapport Top Cities identifie une ville par son nom, par l’État/la province dont elle dépend ainsi que par l’abréviation du pays/de la région.
* Si la ville ou la région (par exemple, État/province) dans laquelle a été initiée une demande n’a pas pu être identifiée, l’info-bulle précise qu’elles sont inconnues. Si le pays/la région est inconnu, deux points d’interrogation (??) s’affichent.
* Un rapport peut contenir des mesures pour les zones « Europe » ou « Asie/Pacifique ». Ces éléments ne sont pas destinés à fournir des informations statistiques sur l’ensemble des adresses IP de ces régions. Ils s’appliquent uniquement aux demandes issues d’adresses IP réparties sur toute la zone Europe ou Asie/Pacifique, sans s’attacher à une ville ou un pays/une région en particulier.

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous obtenez alors le nombre total d’accès, le pourcentage d’accès, la quantité de données transférées (en Go) et le pourcentage de données transférées pour les 250 principales régions. Vous pouvez afficher la description de chacune de ces mesures.

Une brève description est fournie pour les deux types de rapports ci-dessous.

| Nom du rapport | Description |
| --- | --- |
| Top Cities |Ce rapport évalue les villes en fonction du nombre d’accès recensés dans cette région. |
| Top Countries |Ce rapport évalue les pays/régions en fonction du nombre d’accès recensés dans ce pays/cette région. |

## <a name="daily-summary"></a>Daily Summary
Le rapport Daily Summary vous permet d’afficher le nombre total d’accès et de données transférées chaque jour sur une plateforme spécifique. Ces informations peuvent être utilisées pour identifier rapidement les modèles d’activités CDN. Par exemple, ce rapport peut vous aider à détecter les jours où a été recensé un trafic supérieur ou inférieur au niveau attendu.

Lors de la génération de ce type de rapport, un graphique à barres fournit une indication visuelle quant à la quantité de demande recensée chaque jour sur une plateforme donnée pendant la période couverte par le rapport. Le graphique affiche une barre correspondant à chaque jour du rapport. Par exemple, si vous sélectionnez la période appelée « Last Week », vous obtiendrez un graphique contenant sept barres. Chaque barre indique le nombre total d’accès recensés ce jour-là.

La partie gauche du graphique (axe y) indique le nombre d’accès recensés à la date spécifiée. Chaque jour couvert par le rapport est annoté sous forme de date (au format AAAA-MM-JJ) directement sous le graphique (axe x).

> [!TIP]
> Si vous pointez sur une barre, le nombre total d’accès recensés à cette date s’affichera sous la forme d’une info-bulle.
> 
> 

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous y trouverez le nombre total d’accès et la quantité de données transférées (en Go) pour chaque jour couvert par le rapport.

## <a name="by-hour"></a>By Hour
Le rapport By Hour vous permet d’afficher le nombre total d’accès et de données transférées chaque heure sur une plateforme spécifique. Ces informations peuvent être utilisées pour identifier rapidement les modèles d’activités CDN. Par exemple, ce rapport peut vous aider à détecter les moments de la journée pendant lesquels a été recensé un trafic supérieur ou inférieur au niveau attendu.

Lors de la génération de ce type de rapport, un graphique à barres fournit une indication visuelle quant à la quantité de demande recensée chaque heure sur une plateforme donnée pendant la période couverte par le rapport. Le graphique affiche une barre correspondant à chaque heure couverte par le rapport. Par exemple, si vous sélectionnez une période de 24 heures, vous obtiendrez un graphique contenant vingt-quatre barres. Chaque barre indique le nombre total d’accès recensés cette heure-là.

La partie gauche du graphique (axe y) indique le nombre d’accès recensés pendant l’heure spécifiée. Chaque heure couverte par le rapport est annotée sous forme de date/heure (au format AAAA-MM-JJ hh:mm) directement sous le graphique (axe x). L’heure est affichée au format 24 heures et repose sur le fuseau horaire UTC/GMT.

> [!TIP]
> Si vous pointez sur une barre, le nombre total d’accès recensés pendant l’heure en question s’affichera sous la forme d’une info-bulle.
> 
> 

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous y trouverez le nombre total d’accès et la quantité de données transférées (en Go) pour chaque heure couverte par le rapport.

## <a name="by-file"></a>By File
Le rapport By File vous permet d’afficher la quantité de demande et de trafic générés sur une plateforme spécifique pour les ressources les plus demandées. Lors de la génération de ce type de rapport, un graphique à barres est généré pour les 10 ressources les plus demandées au cours de la période spécifiée.

> [!NOTE]
> Dans le cadre de ce rapport, les URL edge CNAME sont converties en URL CDN équivalentes. Cela permet un décompte précis du nombre total d’accès associés à une ressource indépendamment de l’URL CDN ou edge CNAME utilisée pour la demande.
> 
> 

La partie gauche du graphique (axe y) indique le nombre de demandes recensées pour chaque ressource pendant la période spécifiée. Le nom de fichier correspondant à chacune des 10 ressources les plus demandées est annoté directement sous le graphique (axe x).

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous obtenez alors les informations suivantes pour chacune des 250 ressources les plus demandées : chemin d’accès relatif, nombre total d’accès, pourcentage d’accès, quantité de données transférées (en Go) et pourcentage de données transférées.

## <a name="by-file-detail"></a>By File Detail
Le rapport By File Detail vous permet d’afficher la quantité de demande et de trafic générés sur une plateforme spécifique pour une ressource donnée. Tout en haut de ce rapport figure l’option File Details For. Cette option dresse la liste des ressources les plus demandées sur la plateforme sélectionnée. Pour générer un rapport By File Detail, vous devez sélectionner la ressource souhaitée dans l’option File Details For. Un graphique à barres indique ensuite la quantité de demande journalière générée au cours de la période spécifiée.

La partie gauche du graphique (axe y) indique le nombre total de demandes recensées pour une ressource un jour donné. La date (au format AAAA-MM-JJ) à laquelle a été recensée la demande CDN pour la ressource en question est annotée directement sous le graphique (axe x).

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous y trouverez le nombre total d’accès et la quantité de données transférées (en Go) pour chaque jour couvert par le rapport.

## <a name="by-file-type"></a>By File Type
Le rapport By File Type vous permet d’afficher la quantité de demande et de trafic recensés par type de fichier. Lors de la génération de ce type de rapport, un graphique en anneau indique le pourcentage d’accès généré par les 10 principaux types de fichiers.

> [!TIP]
> Si vous pointez sur une tranche du graphique, le type de contenu Internet correspondant à ce type de fichier s’affichera sous la forme d’une info-bulle.
> 
> 

Vous pouvez afficher sous le graphique en anneau les données qui ont été utilisées pour générer ce graphique. Vous obtenez alors l’extension du nom de fichier/le type de contenu Internet, le nombre total d’accès, le pourcentage d’accès, la quantité de données transférées (en Go) et le pourcentage de données transférées pour chacun des 250 principaux types de fichiers.

## <a name="by-directory"></a>By Directory
Le rapport By Directory vous permet d’afficher la quantité de demande et de trafic générés sur une plateforme spécifique pour du contenu issu d’un répertoire spécifique. Lors de la génération de ce type de rapport, un graphique à barres indique le nombre total d’accès générés par contenu dans les 10 principaux répertoires.

### <a name="using-the-bar-chart"></a>Utilisation du graphique à barres
* Placez le curseur sur une barre pour afficher le chemin d’accès relatif au répertoire correspondant.
* Le contenu stocké dans un sous-dossier d’un répertoire n’est pas pris en compte dans le calcul de la demande par répertoire. Ce calcul s’appuie uniquement sur le nombre de demandes générées pour le contenu stocké dans le répertoire réel.
* Dans le cadre de ce rapport, les URL edge CNAME sont converties en URL CDN équivalentes. Cela permet un décompte précis de toutes les statistiques associées à une ressource indépendamment de l’URL CDN ou edge CNAME utilisée pour la demande.

La partie gauche du graphique (axe y) indique le nombre total de demandes recensées pour le contenu stocké dans vos 10 principaux répertoires. Chaque barre du graphique représente un répertoire. Utilisez le schéma de codage couleur pour associer une barre à un répertoire figurant dans la section Top 250 Full Directories.

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous obtenez alors les informations suivantes pour chacun des 250 principaux répertoires : chemin d’accès relatif, nombre total d’accès, pourcentage d’accès, quantité de données transférées (en Go) et pourcentage de données transférées.

## <a name="by-browser"></a>By Browser
Le rapport By Browser vous permet d’afficher les navigateurs utilisés pour les demandes de contenu. Lors de la génération de ce type de rapport, un graphique à secteurs indique le pourcentage de demandes traitées par les 10 principaux navigateurs.

### <a name="using-the-pie-chart"></a>Utilisation du graphique à secteurs
* Pointez sur une tranche du graphique à secteurs pour afficher le nom et la version d’un navigateur.
* Dans le cadre de ce rapport, chaque combinaison navigateur/version unique est considérée comme un navigateur à part entière.
* La section « Others » indique le pourcentage de demandes traitées par tous les autres navigateurs et versions.

Vous pouvez afficher sous le graphique à secteurs les données qui ont été utilisées pour générer ce graphique. Vous y trouverez le type/numéro de version du navigateur, le nombre total d’accès et le pourcentage d’accès pour chacun des 250 principaux navigateurs.

## <a name="by-referrer"></a>By Referrer
Le rapport By Referrer vous permet d’afficher les principaux référents au contenu sur la plateforme sélectionnée. Un référent indique le nom d’hôte à partir duquel une demande a été générée. Lors de la génération de ce type de rapport, un graphique à barres indique la quantité de demande (c’est-à-dire d’accès) générée par les 10 principaux référents.

La partie gauche du graphique (axe y) indique le nombre total de demandes recensées pour une ressource et pour chaque référent. Chaque barre du graphique représente un référent. Utilisez le schéma de codage couleur pour associer une barre à un référent figurant dans la section Top 250 Referrer.

Vous pouvez afficher sous le graphique à barres les données qui ont été utilisées pour générer ce graphique. Vous y trouverez l’URL, le nombre total d’accès et le pourcentage d’accès générés pour chacun des 250 principaux référents.

## <a name="by-download"></a>By Download
Le rapport By Download vous permet d’analyser les modèles de téléchargement pour vos contenus les plus demandés. La partie supérieure du rapport contient un graphique à barres qui compare les tentatives de téléchargement aux téléchargements terminés pour les 10 ressources les plus demandées. Chaque barre est colorée selon qu’il s’agit d’une tentative de téléchargement (bleu) ou d’un téléchargement terminé (vert).

> [!NOTE]
> Dans le cadre de ce rapport, les URL edge CNAME sont converties en URL CDN équivalentes. Cela permet un décompte précis de toutes les statistiques associées à une ressource indépendamment de l’URL CDN ou edge CNAME utilisée pour la demande.
> 
> 

La partie gauche du graphique (axe y) indique le nom de fichier pour chacune des 10 ressources les plus demandées. Le nombre total de tentatives de téléchargement/téléchargements terminés est annoté directement sous le graphique (axe x).

Les informations suivantes figurent sous le graphique pour les 250 ressources les plus demandées : chemin d’accès relatif (y compris nom de fichier), nombre de tentatives avant téléchargement complet, nombre de demandes recensées et pourcentage de demandes ayant abouti à un téléchargement complet.

> [!TIP]
> Notre CDN n’est pas informé par un client HTTP (par exemple, un navigateur) lorsqu’une ressource a été entièrement téléchargée. Par conséquent, nous devons déterminer si une ressource a été entièrement téléchargée en fonction des codes d’état et des demandes de plage d’octets. Pour ce calcul, la première chose à faire est de déterminer si les résultats de la demande génèrent un code d’état de 200 (succès de la requête). Dans ce cas, nous examinons les demandes de plage d’octets pour vérifier qu’elles couvrent bien l’ensemble de la ressource. Enfin, nous comparons la quantité de données transférées à la taille de la ressource demandée. Si les données transférées sont supérieures ou égales à la taille du fichier et si les demandes de plage d’octets sont appropriées pour cette ressource, l’accès sera alors considéré comme un téléchargement complet.
> 
> Puisque ce rapport repose sur des interprétations, rappelez-vous que la cohérence et la précision de ce rapport peut être affectée pour les raisons suivantes :
> 
> * Les modèles de trafic ne peuvent pas être prédits avec précision lorsque les agents utilisateurs se comportent différemment. Cela peut porter le taux de téléchargement terminé au-delà de 100 %.
> * Les ressources qui utilisent un téléchargement progressif HTTP ne peuvent pas être représentés avec précision dans ce rapport. Cela est dû aux utilisateurs qui recherchent des positions différentes dans une vidéo.
> 
> 

## <a name="by-404-errors"></a>By 404 Errors
Le rapport By 404 Errors vous permet d’identifier le type de contenu qui génère le plus grand nombre de codes d’état « 404 Not Found ». La partie supérieure du rapport contient un graphique à barres qui regroupe les 10 principales ressources ayant donné lieu au renvoi d’un code d’état « 404 Not Found ». Ce graphique à barres compare le nombre total de demandes aux demandes ayant généré un code d’état « 404 Not Found » pour les ressources concernées. Chaque barre est codée par couleur. Une barre jaune indique que la demande a généré un code d’état « 404 Not Found ». Une barre rouge indique le nombre total de demandes pour la ressource concernée.

> [!NOTE]
> Dans le cadre de ce rapport, notez les points suivants :
> 
> * Un accès représente une demande de ressource, quel que soit le code d’état.
> * Les URL edge CNAME sont converties en URL CDN équivalentes. Cela permet un décompte précis de toutes les statistiques associées à une ressource indépendamment de l’URL CDN ou edge CNAME utilisée pour la demande.
> 
> 

La partie gauche du graphique (axe y) indique le nom de fichier pour chacune des 10 ressources les plus demandées ayant généré un code d’état « 404 Not Found ». Le nombre total de demandes et le nombre de demandes ayant généré un code d’état « 404 Not Found » sont annotés directement sous le graphique (axe x).

Les informations suivantes figurent sous le graphique pour les 250 ressources les plus demandées : chemin d’accès relatif (y compris nom de fichier), nombre de demandes ayant généré un code d’état « 404 Not Found », nombre de demandes recensées pour la ressource et pourcentage de demandes ayant abouti à un code d’état « 404 Not Found ».

## <a name="see-also"></a>Voir aussi
* [Vue d'ensemble d'Azure CDN](cdn-overview.md)
* [Statistiques en temps dans Microsoft Azure CDN](cdn-real-time-stats.md)
* [Remplacement du comportement HTTP par défaut à l’aide du moteur de règles](./cdn-verizon-premium-rules-engine.md)
* [Analyser les performances de serveurs Edge](cdn-edge-performance.md)