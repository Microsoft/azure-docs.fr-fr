---
title: Contrôle d’accès aux classeurs Azure Monitor
description: Créer des rapports complexes en toute simplicité grâce à des classeurs paramétrables prédéfinis et personnalisés à l’aide du contrôle d’accès en fonction du rôle
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 7d3bc13dc373cda510153099859cf4cd61b3dd69
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95534532"
---
# <a name="access-control"></a>Contrôle d’accès

Dans les classeurs, le contrôle d’accès fait référence à deux éléments :

* Accès requis pour lire les données dans un classeur. Cet accès est contrôlé par les rôles [Azure standard](../../role-based-access-control/overview.md) sur les ressources utilisées dans le classeur. Les classeurs ne spécifient pas ou ne configurent pas l’accès à ces ressources. Les utilisateurs obtiennent généralement cet accès à ces ressources en utilisant le rôle [Lecteur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-reader) sur ces ressources.

* Accès requis pour enregistrer des classeurs

    - L’enregistrement de classeurs `("My")` privés ne nécessite pas de privilèges supplémentaires. Tous les utilisateurs peuvent enregistrer des classeurs privés, auquel cas ils sont seuls à pouvoir voir ces classeurs.
    - L’enregistrement des classeurs partagés requiert des privilèges d’écriture dans un groupe de ressources pour enregistrer le classeur. Ces privilèges sont généralement spécifiés par le rôle [Contributeur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-contributor), mais peuvent également être définis via le rôle *Contributeur de classeurs*.
    
## <a name="standard-roles-with-workbook-related-privileges"></a>Rôles standard avec privilèges associés à un classeur

[Lecteur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-reader) inclut les privilèges standard /read qui seraient utilisés par les outils de supervision (y compris les classeurs) pour lire les données des ressources.

[Contributeur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-contributor) inclut des privilèges `/write` généraux utilisés par divers outils de surveillance pour l’enregistrement d’éléments (y compris le privilège `workbooks/write` pour l’enregistrement des classeurs partagés).
« Collaborateur Workbooks » ajoute des privilèges « workbooks/écriture » à un objet pour enregistrer les classeurs partagés.
Aucun privilège spécial n’est nécessaire pour permettre aux utilisateurs d’enregistrer les classeurs privés qu’ils peuvent voir.

Pour les rôles personnalisés :

Ajoutez `microsoft.insights/workbooks/write` pour enregistrer les workbooks partagés. Pour plus d’informations, consultez le rôle [Collaborateur du workbook](../../role-based-access-control/built-in-roles.md#monitoring-contributor).

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.