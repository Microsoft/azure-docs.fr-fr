---
title: Mieux gérer votre SOC avec des métriques d’incident dans Azure Sentinel | Microsoft Docs
description: Utilisez les informations de l’écran et du classeur des métriques d’incidents Azure Sentinel pour vous aider à gérer votre centre des opérations de sécurité (SOC).
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/11/2020
ms.author: yelevin
ms.openlocfilehash: 408913fed864ee5f966b96c81afbfee4b2dc8678
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660727"
---
# <a name="manage-your-soc-better-with-incident-metrics"></a>Mieux gérer votre SOC avec des métriques d’incident

> [!IMPORTANT]
> Les fonctionnalités des métriques d’incident sont actuellement disponibles en préversion publique.
> Ces fonctionnalités sont fournies sans contrat de niveau de service et sont déconseillées pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En tant que responsable d’un centre des opérations de sécurité (SOC), vous devez disposer de mesures et d’indicateurs d’efficacité globale à portée de main pour évaluer les performances de votre équipe. Vous voudrez voir les opérations d’incident au fil du temps selon de nombreux critères différents, comme la gravité, les tactiques MITRE, le temps moyen de triage, le temps moyen de résolution, et plus encore. Azure Sentinel met désormais ces données à votre disposition avec le nouveau tableau et schéma **SecurityIncident** dans Log Analytics et le classeur **Efficacité des opérations de sécurité** qui l’accompagne. Vous serez en mesure de visualiser les performances de votre équipe dans le temps et d’utiliser ces informations pour améliorer l’efficacité. Vous pouvez également écrire et utiliser vos propres requêtes KQL sur le tableau des incidents afin de créer des classeurs personnalisés qui répondent à vos besoins spécifiques en matière d’audit et indicateurs de performance clés.

## <a name="use-the-security-incidents-table"></a>Utiliser le tableau des incidents de sécurité

Le tableau **SecurityIncident** est intégré à Azure Sentinel. Vous le trouverez avec les autres tableaux de la collection **SecurityInsights** sous la rubrique **Journaux**. Vous pouvez l’interroger comme n’importe quel autre tableau dans Log Analytics.

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incident-table.png" alt-text="Tableau des incidents de sécurité":::

Chaque fois que vous créez ou mettez à jour un incident, une nouvelle entrée de journal est ajoutée au tableau. Cela vous permet de suivre les modifications apportées aux incidents et d’obtenir des métriques SOC encore plus puissantes, mais vous devez en tenir compte lorsque vous créez des requêtes pour ce tableau, car vous devrez peut-être supprimer les entrées en double pour un incident (selon la requête exacte que vous exécutez). 

Par exemple, si vous souhaitez renvoyer une liste de tous les incidents triés par leur numéro d’incident mais que vous souhaitez uniquement renvoyer le journal le plus récent par incident, vous pouvez le faire en utilisant l’[opérateur KQL summarize](/azure/data-explorer/kusto/query/summarizeoperator) avec la [fonction d’agrégation](/azure/data-explorer/kusto/query/arg-max-aggfunction) `arg_max()` :


```Kusto
SecurityIncident
| summarize arg_max(LastModifiedTime, *) by IncidentNumber
```
### <a name="more-sample-queries"></a>Plus d’exemples de requêtes

Temps moyen de clôture
```Kusto
SecurityIncident
| summarize arg_max(TimeGenerated,*) by IncidentNumber 
| extend TimeToClosure =  (ClosedTime - CreatedTime)/1h
| summarize 5th_Percentile=percentile(TimeToClosure, 5),50th_Percentile=percentile(TimeToClosure, 50), 
  90th_Percentile=percentile(TimeToClosure, 90),99th_Percentile=percentile(TimeToClosure, 99)
```

Temps moyen de triage
```Kusto
SecurityIncident
| summarize arg_max(TimeGenerated,*) by IncidentNumber 
| extend TimeToTriage =  (FirstModifiedTime - CreatedTime)/1h
| summarize 5th_Percentile=max_of(percentile(TimeToTriage, 5),0),50th_Percentile=percentile(TimeToTriage, 50), 
  90th_Percentile=percentile(TimeToTriage, 90),99th_Percentile=percentile(TimeToTriage, 99) 
```

## <a name="security-operations-efficiency-workbook"></a>Classeur Efficacité des opérations de sécurité

Pour compléter le tableau **SecurityIncidents**, nous vous avons fourni un modèle de classeur **Efficacité des opérations de sécurité** prêt à l’emploi que vous pouvez utiliser pour surveiller vos opérations SOC. Le classeur contient les métriques suivantes : 
- Incident créé au fil du temps 
- Incidents créés par classification de clôture, gravité, propriétaire et état 
- Temps moyen de triage 
- Temps moyen de clôture 
- Incidents créés par gravité, propriétaire, état, produit et tactiques au fil du temps 
- Pourcentages du temps de triage 
- Pourcentages du temps avant clôture 
- Temps moyen de triage par propriétaire 
- Activités récentes 
- Classifications de clôture récentes  

Vous pouvez trouver ce nouveau modèle de classeur en choisissant **Classeurs** dans le menu de navigation d’Azure Sentinel et en sélectionnant l’onglet **Modèles**. Choisissez **Efficacité des opérations de sécurité** à partir de la galerie, puis cliquez sur l’un des boutons **Afficher le classeur enregistré** et **Afficher le modèle**.

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-incidents-workbooks-gallery.png" alt-text="Galerie de classeurs d’incidents de sécurité":::

:::image type="content" source="./media/manage-soc-with-incident-metrics/security-operations-workbook-1.png" alt-text="Classeur d’incidents de sécurité terminé":::

Vous pouvez utiliser le modèle pour créer vos propres classeurs personnalisés, adaptés à vos besoins spécifiques.

## <a name="securityincidents-schema"></a>Schéma SecurityIncidents

[!INCLUDE [SecurityIncidents schema](../../includes/sentinel-schema-security-incident.md)]

## <a name="next-steps"></a>Étapes suivantes

- Pour utiliser Azure Sentinel, vous devez disposer d’un abonnement à Microsoft Azure. Si vous n’avez pas d’abonnement, vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/free/).
- Découvrez comment [intégrer vos données à Azure Sentinel](quickstart-onboard.md) et [obtenir une visibilité de vos données et des menaces potentielles](quickstart-get-visibility.md).