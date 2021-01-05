---
title: Supervision des flux de données de mappage
description: Guide pratique pour superviser visuellement les flux de données de mappage dans Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/22/2020
ms.openlocfilehash: 9ca5ea5cdebe297af5081ae6e219935c56ba942e
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96004865"
---
# <a name="monitor-data-flows"></a>Superviser les flux de données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Après avoir généré et débogué votre flux de données, vous pouvez le planifier de façon à l’exécuter selon une planification dans le contexte d’un pipeline. Vous pouvez planifier le pipeline à partir d’Azure Data Factory à l’aide de déclencheurs. Vous pouvez également utiliser l’option Déclencher maintenant à partir du Générateur de pipeline d’Azure Data Factory pour déclencher une exécution unique afin de tester votre flux de données dans le contexte du pipeline.

Lors de l’exécution de votre pipeline, vous pouvez superviser celui-ci ainsi que toutes les activités qu’il contient, dont celle du flux de données. Cliquez sur l’icône Superviser dans le panneau de gauche de l’interface utilisateur d’Azure Data Factory. Un écran similaire à celui ci-dessous s’affiche. Les icônes en surbrillance vous permettent d’explorer les activités du pipeline, dont celle du flux de données.

![Capture d’écran montrant les icônes à sélectionner pour les pipelines pour plus d’informations.](media/data-flow/mon001.png "Supervision du flux de données")

À ce niveau, vous voyez également des statistiques, notamment l’état et le temps d’exécution. L’ID d’exécution au niveau d’une activité diffère de celui au niveau du pipeline. L’ID d’exécution au niveau précédent a trait au pipeline. En cliquant sur les lunettes, vous pouvez obtenir des informations détaillées sur l’exécution de votre flux de données.

![Capture d’écran montrant l’icône en forme de lunettes pour afficher les détails relatifs à l’exécution du flux de données.](media/data-flow/monitoring-details.png "Supervision du flux de données")

Dans l’affichage de surveillance des nœuds sous forme graphique, vous pouvez voir une version simplifiée en lecture seule de votre graphique de flux de données.

![Capture d’écran montrant la version en mode affichage uniquement du graphique.](media/data-flow/mon003.png "Supervision du flux de données")

Voici une vidéo de présentation de la surveillance des performances de vos flux de données à partir de l’écran d’analyse ADF :

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4u4mH]

## <a name="view-data-flow-execution-plans"></a>Afficher les plans d’exécution de flux de données

Quand votre flux de données est exécuté dans Spark, Azure Data Factory détermine les chemins de code optimaux en fonction de l’intégralité de votre flux de données. De plus, les chemins d’exécution peuvent se trouver sur différents nœuds de scale-out et partitions de données. Par conséquent, le graphique de supervision reflète la conception de votre flux, en tenant compte du chemin d’exécution de vos transformations. En sélectionnant des nœuds individuels, vous pouvez voir des « regroupements » représentant du code exécuté simultanément sur le cluster. Les minutages et les chiffres que vous voyez représentent ces groupes par opposition aux étapes individuelles de votre conception.

![Capture d’écran montrant la page d’un flux de données.](media/data-flow/mon004.png "Supervision du flux de données")

* Lorsque vous sélectionnez l’espace disponible dans la fenêtre de supervision, les statistiques dans le volet inférieur affichent le minutage et le nombre de lignes pour chaque récepteur, ainsi que les transformations ayant conduit aux données du récepteur pour la traçabilité des transformations.

* Quand vous sélectionnez des transformations individuelles, des commentaires supplémentaires apparaissent dans le panneau de droite, présentant les statistiques de partition, les nombres de colonnes, l’asymétrie (degré d’uniformité de la distribution des données entre les partitions) et le kurtosis (intensité des pics de données).

* Lorsque vous sélectionnez le récepteur dans la vue du nœud, vous voyez la traçabilité des données de la colonne. Avant d’arriver dans le récepteur, les données sont cumulées tout au long de votre flux de données selon trois méthodes différentes. Il s'agit de :

  * Calculée : Vous utilisez la colonne pour le traitement conditionnel ou dans une expression de votre flux de données, mais les données n’arrivent pas dans le récepteur.
  * Dérivée : La colonne est une nouvelle colonne que vous avez générée dans votre flux, c’est-à-dire qu’elle n’était pas présente dans la source.
  * Mappée : La colonne provient de la source et vous la mappez à un champ du récepteur.
  * État du flux de données : État actuel de l’exécution
  * Heure de démarrage du cluster : Durée d’acquisition de l’environnement de calcul JIT Spark pour l’exécution de votre flux de données
  * Nombre de transformations : Nombre d’étapes de transformation exécutées dans votre flux
  
![Capture d’écran montrant l’option Actualiser.](media/data-flow/monitornew.png "Supervision du flux de données - Nouveau")

## <a name="total-sink-processing-time-vs-transformation-processing-time"></a>Différence entre le temps de traitement du récepteur et le temps de traitement de la transformation

Chaque étape de transformation indique une durée totale d’exécution, dans laquelle sont totalisés les temps d’exécution de chaque partition. Lorsque vous cliquez sur le récepteur, vous devez voir « Temps de traitement du récepteur ». Cette durée comprend la durée totale de la transformation *plus* le temps d’E/S nécessaire à l’écriture de vos données dans le magasin de destination. C’est le temps d’E/S nécessaire à l’écriture de vos données qui fait la différence entre le temps de traitement du récepteur et la durée totale de la transformation.

Vous pouvez également voir le minutage détaillé de chaque étape de transformation de partition en ouvrant la sortie JSON de votre activité de flux de données dans la vue de supervision du pipeline ADF. La sortie JSON contient la durée en millisecondes de chaque partition, tandis que la vue de supervision de l’expérience utilisateur présente le minutage agrégé des partitions :

```
 {
     "stage": 4,
     "partitionTimes": [
          14353,
          14914,
          14246,
          14912,
          ...
         ]
}
```

### <a name="post-processing-time"></a>Temps de post-traitement

Lorsque vous sélectionnez une icône de transformation de récepteur dans votre mappage, le panneau déroulant à droite affiche un point de données supplémentaire appelé « temps de post-traitement » en bas. Il s’agit du temps consacré à l’exécution de votre travail sur le cluster Spark *après* que vos données ont été chargées, transformées et écrites. Ce temps peut comprendre la fermeture de pools de connexions, l’arrêt de pilotes, la suppression de fichiers, la fusion de fichiers, etc. Lorsque vous effectuez des actions dans votre flux, telles que le « déplacement de fichiers » et la « sortie vers un seul fichier », vous voyez probablement une augmentation de la valeur du temps de post-traitement.
  
## <a name="error-rows"></a>Lignes d’erreur

L’activation de la gestion des lignes d’erreur dans votre récepteur de flux de données se reflète dans la sortie de la surveillance. Lorsque vous définissez le récepteur sur « Réussite signalée malgré l’erreur », la sortie de la surveillance indique le nombre de lignes ayant abouti et échoué lorsque vous cliquez sur le nœud surveillance du récepteur.

![Capture d’écran montrant des lignes d’erreur.](media/data-flow/error-row-2.png "Réussite de la supervision des lignes d’erreur")

Lorsque vous sélectionnez « Echec signalé malgré l’erreur », la même sortie s’affiche uniquement dans le texte de sortie de la surveillance de l’activité. Cela est dû au fait que l’activité de flux de données renvoie une erreur d’exécution et que la vue de la surveillance détaillée n’est pas disponible.

![Capture d’écran montrant des lignes d’erreur dans l’activité.](media/data-flow/error-rows-4.png "Échec de la supervision des lignes d’erreur")

## <a name="monitor-icons"></a>Icônes Superviser

Cette icône signifie que les données de la transformation étaient déjà en cache sur le cluster. Les minutages et le chemin d’exécution en ont donc tenu compte :

![Capture d’écran montrant l’icône de disque.](media/data-flow/mon005.png "Supervision du flux de données")

Vous pouvez également voir des icônes de cercle vert dans la transformation. Elles représentent un décompte du nombre de récepteurs dans lesquels les données s’écoulent.
