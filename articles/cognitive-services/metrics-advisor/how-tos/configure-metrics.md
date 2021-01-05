---
title: Configuration de votre instance Metrics Advisor à l’aide du portail Web
titleSuffix: Azure Cognitive Services
description: Comment configurer votre instance Metrics Advisor et affiner les résultats de la détection d’anomalie.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: 7923dad3d47122c0ceb04d1240736e2b66a0dd64
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92048251"
---
# <a name="how-to-configure-metrics-and-fine-tune-detecting-configuration"></a>Procédure : Configurer des métriques et affiner la configuration de la détection

Utilisez cet article pour commencer à configurer votre instance Metrics Advisor à l’aide du portail Web. Pour parcourir les métriques d’un flux de données spécifique, accédez à la page **Data feeds** (Flux de données) et sélectionnez un flux. Vous obtenez une liste de métriques associées à ce flux.

:::image type="content" source="../media/metrics/select-metric.png" alt-text="Sélectionner une métrique" lightbox="../media/metrics/select-metric.png":::

Cliquez sur le nom de l’une des métriques pour en afficher les détails. Dans cette vue détaillée, vous pouvez basculer vers une autre métrique du même flux de données à l’aide de la liste déroulante située en haut à droite de l’écran.

Lorsque vous affichez les détails d’une métrique pour la première fois, vous pouvez charger une série chronologique en laissant à Metrics Advisor le soin de la choisir pour vous. Mais vous pouvez également spécifier des valeurs à inclure pour chaque dimension. 

Vous pouvez même sélectionner des intervalles de temps et modifier le layout de la page.

> [!NOTE]
> - L’heure de début est inclusive.
> - L’heure de fin est exclusive. 

Vous pouvez cliquer sur l’onglet **Incidents** pour afficher les anomalies et obtenir un lien vers le [hub d’incidents](diagnose-incident.md).

## <a name="tune-the-detecting-configuration"></a>Paramétrer la configuration de détection

Une métrique peut appliquer une ou plusieurs configurations de détection. Il existe une configuration par défaut pour chaque métrique, que vous pouvez modifier ou ajouter en fonction de vos besoins d’analyse.

### <a name="tune-the-configuration-for-all-series-in-current-metric"></a>Paramétrer la configuration de toutes les séries dans la métrique actuelle

Cette configuration sera appliquée à toutes les séries de cette métrique, à l’exception de celles qui présentent une configuration distincte. Une configuration de niveau métrique est appliquée par défaut lors de l’intégration des données. Elle est affichée dans le panneau de gauche. Les utilisateurs peuvent modifier directement la configuration du niveau de métrique sur la page de la métrique. 

Pour affiner encore la configuration, vous pouvez utiliser d’autres paramètres, tels que **Direction** ou **Valid anomaly** (Anomalie valide). Vous pouvez également combiner différentes méthodes de détection. 

:::image type="content" source="../media/configuration-combination.png" alt-text="Sélectionner une métrique" lightbox="../media/configuration-combination.png":::

### <a name="tune-the-configuration-for-a-specific-series-or-group"></a>Paramétrer la configuration d’une série ou d’un groupe spécifique

Cliquez sur **Advanced Configuration** (Configuration avancée) sous les options de configuration de niveau métrique pour afficher la configuration au niveau du groupe. Vous pouvez ajouter une configuration pour une série spécifique ou pour un groupe de séries en cliquant sur l’icône **+** dans cette fenêtre. Les paramètres sont similaires aux paramètres de configuration de niveau métrique, mais vous devrez peut-être spécifier au moins une valeur de dimension pour une configuration de niveau groupe afin d’identifier un groupe de séries. Pour identifier une série spécifique, vous devez également spécifier toutes les valeurs de dimension pour la configuration au niveau de la série. 

Cette configuration sera appliquée au groupe de séries ou à une série spécifique au lieu de la configuration de niveau métrique. Une fois que vous avez défini les conditions pour ce groupe, enregistrez-le.

:::image type="content" source="../media/advanced-configuration.png" alt-text="Sélectionner une métrique" lightbox="../media/advanced-configuration.png":::

### <a name="anomaly-detection-methods"></a>Méthodes de détection d’anomalies

Metrics Advisor propose plusieurs méthodes de détection d’anomalie. Vous pouvez en utiliser une ou combiner plusieurs d’entre elles à l’aide d’opérateurs logiques en cliquant sur le bouton **+** . 

**Détection intelligente** 

La détection intelligente est optimisée par un moteur d’apprentissage automatique qui assimile des modèles à partir de données historiques et les utilise pour les prochaines détections. La **sensibilité** est le paramètre le plus important de cette méthode car elle permet d’affiner les résultats de la détection. Vous pouvez en augmenter ou en diminuer la valeur à l’aide d’un simple curseur pour modifier la visualisation à droite de la page. Choisissez la valeur qui convient pour vos données et enregistrez-la. 


En mode de détection intelligente, les paramètres de sensibilité et de version limite permettent d’affiner le résultat de la détection d’anomalie.

La sensibilité peut affecter l’étendue de la plage de valeurs attendue pour chaque point. Lorsqu’elle augmente, la plage de valeurs attendue se resserre, ce qui permet de signaler un plus grand nombre d’anomalies :

:::image type="content" source="../media/metrics/smart-detection-high-sensitivity.png" alt-text="Sélectionner une métrique":::

Lorsque vous diminuez la sensibilité, la plage de valeurs attendue s’élargit et le nombre d’anomalies signalées diminue :

:::image type="content" source="../media/metrics/smart-detection-low-sensitivity.png" alt-text="Sélectionner une métrique":::

**Seuil de modification** 

Le seuil de modification est normalement utilisé lorsque les données de métriques se situent généralement autour d’une certaine plage. Le seuil est défini en fonction du **pourcentage de modification**. Le mode **Seuil de modification** permet de détecter des anomalies dans les scénarios :

* En règle générale, vos données sont stables et fluides. Vous souhaitez être averti de toute fluctuation.
* En règle générale, vos données sont relativement instables et ont tendance à beaucoup fluctuer. Vous souhaitez être averti chaque fois qu’elles deviennent trop stables ou statiques.

Pour utiliser ce mode, procédez comme suit :

1. Sélectionnez l’option **Change threshold** comme méthode de détection d’anomalie lorsque vous définissez les configurations de détection d’anomalie pour vos métriques ou séries chronologiques.
    
    :::image type="content" source="../media/metrics/change-threshold.png" alt-text="Sélectionner une métrique":::

2. Sélectionnez le paramètre **out of the range** (hors plage) ou **in the range** (dans la plage), en fonction de votre scénario.

    Si vous souhaitez détecter les fluctuations, sélectionnez **out of the range** (hors plage). Par exemple, avec les paramètres ci-dessous, tous les points de données qui varient de plus de 10 % par rapport au point précédent seront détectés comme des valeurs hors norme.
    :::image type="content" source="../media/metrics/out-of-the-range.png" alt-text="Sélectionner une métrique":::

    Si vous souhaitez détecter des lignes droites dans vos données, sélectionnez **in the range** (dans la plage). Par exemple, avec les paramètres ci-dessous, tous les points de données qui varient de l’ordre de 0,01 % par rapport au point précédent seront détectés comme des valeurs hors norme. Étant donné que le seuil est extrêmement faible (0,01 %), les lignes droites détectées dans les données sont considérées comme des valeurs hors norme.

    :::image type="content" source="../media/metrics/in-the-range.png" alt-text="Sélectionner une métrique":::

3. Définissez le pourcentage de modification qui sera considéré comme une anomalie et sélectionnez les points de données précédemment capturés qui seront utilisés à des fins de comparaison. Cette comparaison se fait toujours entre le point de données actuel et un point de données unique qui le précède de N points.
    
    Le paramètre **Direction** n’est valide que si vous utilisez le mode **out of the range** (hors plage) :
    
    * L’option **Up** permet de configurer la détection pour détecter uniquement les anomalies quand (point de données actuel) - (point de données de comparaison) > **+** pourcentage seuil.
    * L’option **Down** permet de configurer la détection pour détecter uniquement les anomalies quand (point de données actuel) - (point de données de comparaison) < **-** pourcentage seuil.
 
**Seuil dur**

 Le seuil dur est une méthode de base pour la détection d’anomalie. Vous pouvez définir une limite supérieure et/ou inférieure pour déterminer la plage de valeurs attendue. Tout point se trouvant en dehors de la limite est identifié comme une anomalie. 


## <a name="preset-events"></a>Événements prédéfinis

Il peut arriver que certains événements et occurrences attendus (tels que les congés) génèrent des données anormales. Les événements prédéfinis vous permettent d’ajouter des indicateurs à la sortie de la détection d’anomalie pendant les heures spécifiées. Cette fonctionnalité doit être configurée après l’intégration de votre flux de données. Chaque métrique ne peut avoir qu’une seule configuration d’événement de présélection.

> [!Note]
> La configuration des événements prédéfinis prend en compte les congés pendant la détection d’anomalie et peut modifier vos résultats. Elle est appliquée aux points de données ingérés après l’enregistrement de la configuration. 

Cliquez sur le bouton **Configure Preset Event** (Configurer un événement prédéfini) en regard de la liste déroulante des métriques sur chaque page de détails des métriques.
 
:::image type="content" source="../media/metrics/preset-event-button.png" alt-text="Sélectionner une métrique":::

Dans la fenêtre qui s’affiche, configurez les options en fonction de votre scénario d’utilisation. Assurez-vous que l’option **Enable holiday event** (Activer les événements congés) est sélectionnée pour utiliser la configuration. 

La section **Holiday event** (Événement congés) vous aide à supprimer les anomalies inutiles détectées au cours des congés. L’option **Strategy** (Stratégie) contient deux sous-options :

* **Suppress holiday** (Supprimer pendant les congés) : Permet de supprimer toutes les anomalies et alertes dans les résultats de détection d’anomalie au cours de la période de congés.
* **Holiday as weekend** (Congé le week-end) : Calcule les valeurs moyennes attendues de plusieurs week-ends correspondants avant les congés et définit l’état d’anomalie sur la base de ces valeurs.

Vous pouvez configurer plusieurs autres valeurs.

|Option  |Description  |
|---------|---------|
|**Choose one dimension as country** (Choisir une dimension comme pays)     | Choisissez une dimension qui contient des informations relatives au pays. Par exemple, un code de pays.         |
|**Country code mapping** (Mappage des codes de pays)     | Mappage entre un [code de pays](https://wikipedia.org/wiki/ISO_3166-1_alpha-2) standard et les données du pays de la dimension choisie.        |
|**Holiday options** (Options de congés)    | Indique s’il faut prendre en compte tous les congés, uniquement les congés payés ou uniquement les congés sans solde.         |
|**Days to expand** (Jours à étendre)    |  Jours affectés avant et après un congé.        |


La section **Cycle event** (Événement cyclique) peut être utilisée dans certains scénarios afin de réduire les alertes inutiles en utilisant des modèles cycliques dans les données. Exemple : 

- Métriques ayant plusieurs modèles ou cycles, par exemple un modèle hebdomadaire et mensuel. 
- Métriques qui ne sont pas associées à un modèle clair, mais dont les données sont comparables d’une année à l’autre (YoY), d’un mois à l’autre (MoM), d’une semaine à l’autre (WoW) ou d’un jour à l’autre (DoD).
 
Toutes les options ne peuvent pas être sélectionnées pour chaque granularité. Les options disponibles par granularité sont les suivantes :

| Granularité | YoY | MoM | WoW | DoD |
|:-|:-|:-|:-|:-|
| Annuelle | X | X | X | X |
| Mensuelle | X | X | X | X |
| Hebdomadaire | ✔ | X | X | X |
| Quotidien | ✔ | ✔ | ✔ | X |
| Toutes les heures | ✔ | ✔ | ✔ | ✔ |
| Toutes les minutes | X | X | X | X |
| Toutes les secondes | X | X | X | X |
| Personnalisé* | ✔ | ✔ | ✔ | ✔ |

X - Indisponible.  
✔ - Disponible.
  
\* Lors de l’utilisation d’une granularité personnalisée en secondes, disponible uniquement si la métrique est supérieure à une heure et inférieure à un jour.

L’événement cyclique est utilisé pour réduire les anomalies si elles suivent un modèle cyclique, mais signale une anomalie si plusieurs points de données ne suivent pas le modèle. L’option **Strict mode** (Mode strict) est utilisée pour activer la création de rapports d’anomalies si un seul point de données ne suit pas le modèle. 

:::image type="content" source="../media/metrics/preset-events.png" alt-text="Sélectionner une métrique":::

## <a name="view-recent-incidents"></a>Afficher les incidents récents

Metrics Advisor détecte les anomalies sur toutes les données de votre série chronologique à mesure qu’elles sont ingérées. Il n’est cependant nécessaire de faire remonter toutes les anomalies, car leur impact peut être mineur. Une agrégation sera effectuée sur les anomalies afin de regrouper en incidents les anomalies connexes. Vous pouvez afficher ces incidents à partir de l’onglet **Incident** accessible depuis la page de détails des métriques. 

Cliquez sur un incident pour accéder à la page **Incidents analysis** (Analyse des incidents) et obtenir plus de détails. Cliquez sur **Manage incidents in new Incident hub** (Gérer les incidents dans un nouveau hub d’incidents) pour accéder à la page [Incident hub](diagnose-incident.md) (Hub d’incidents) qui vous permet de rechercher tous les incidents associés à la métrique spécifique. 

## <a name="subscribe-anomalies-for-notification"></a>S’abonner aux notifications d’anomalies

Si vous souhaitez être averti chaque fois qu’une anomalie est détectée, vous pouvez vous abonner aux alertes de la métrique, à l’aide d’un hook. Reportez-vous à la section [Configurer des alertes et obtenir des notifications à l’aide d’un hook](alerts.md) pour plus d’informations.


## <a name="next-steps"></a>Étapes suivantes 
- [Configurer des alertes et obtenir des notifications à l’aide d’un hook](alerts.md)
- [Ajuster la détection des anomalies à l’aide de commentaires](anomaly-feedback.md)
- [Diagnostiquer un incident](diagnose-incident.md)

