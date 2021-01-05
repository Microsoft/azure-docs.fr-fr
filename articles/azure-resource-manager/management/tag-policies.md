---
title: Stratégies pour l’étiquetage des ressources
description: Décrit les stratégies Azure que vous pouvez attribuer pour garantir la conformité des étiquettes.
ms.topic: conceptual
ms.date: 03/20/2020
ms.openlocfilehash: c6867bc01306ac3c08a9797ece0567a45e060af2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89293731"
---
# <a name="assign-policies-for-tag-compliance"></a>Attribuer des stratégies pour la conformité des étiquettes

Vous pouvez utiliser [Azure Policy](../../governance/policy/overview.md) pour appliquer des règles et conventions d’étiquetage. En créant une stratégie, vous pouvez éviter de déployer dans votre abonnement des ressources n’ayant pas les étiquettes attendues pour votre organisation. Au lieu d’appliquer des étiquettes ou de rechercher des ressources non conformes manuellement, vous pouvez créer une stratégie qui applique automatiquement les étiquettes nécessaires pendant le déploiement. Les étiquettes peuvent également être appliquées aux ressources existantes avec le nouvel effet [Modifier](../../governance/policy/concepts/effects.md#modify) et une [tâche de correction](../../governance/policy/how-to/remediate-resources.md). La section suivante présente des exemples de stratégies pour les étiquettes.

## <a name="policies"></a>Stratégies

[!INCLUDE [Tag policies](../../../includes/policy/reference/bycat/policies-tags.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la catégorisation des ressources, consultez [Organisation des ressources Azure à l’aide d’étiquettes](tag-resources.md).
* Les types de ressources ne prennent pas tous en charge les étiquettes. Pour déterminer si vous pouvez appliquer une étiquette à un type de ressource, consultez [Prise en charge des étiquettes pour les ressources Azure](tag-support.md).
