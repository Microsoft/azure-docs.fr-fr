---
title: 'Tutoriel : Nouvelle affectation de stratégie avec le portail Azure'
description: Dans ce tutoriel, vous allez utiliser le portail Azure pour créer une attribution Azure Policy afin d’identifier les ressources non conformes.
ms.topic: tutorial
ms.date: 10/07/2020
ms.openlocfilehash: 9a07e490525ce532f8f843b30b3b83715e65ce3c
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91826588"
---
# <a name="tutorial-create-a-policy-assignment-to-identify-non-compliant-resources"></a>Tutoriel : Créer une affectation de stratégie pour identifier les ressources non conformes

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources. Azure Policy prend en charge l’audit de l’état de votre serveur activé pour Arc avec des stratégies de configuration d’invité. Les stratégies de configuration d’invité ne s’appliquent pas aux configurations : elles auditent seulement les paramètres à l’intérieur de la machine. Ce tutoriel vous guide tout au long du processus de création et d’affectation d’une stratégie, en identifiant les serveurs activés pour Arc où l’agent Log Analytics n’est pas installé.

À la fin de ce processus, vous identifierez correctement les machines sur lesquelles l’agent Log Analytics pour Windows ou Linux n’est pas installé. Elles sont _non conformes_ à l’attribution de stratégie.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

Dans ce tutoriel, vous créez une affectation de stratégie et vous affectez la définition de stratégie _\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux_.

1. Lancez le service Azure Policy dans le portail Azure en cliquant sur **Tous les services**, puis en recherchant et en cliquant sur **Stratégie**.

   :::image type="content" source="./media/tutorial-assign-policy-portal/search-policy.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

1. Sélectionnez **Affectations** du côté gauche de la page Azure Policy. Une affectation est une stratégie qui a été affectée pour être appliquée dans une étendue spécifique.

   :::image type="content" source="./media/tutorial-assign-policy-portal/select-assignment.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

1. Sélectionnez **Assigner une stratégie** en haut de la page**Stratégie - Affectations**.

   :::image type="content" source="./media/tutorial-assign-policy-portal/select-assign-policy.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

1. Dans la page **Assigner une stratégie**, sélectionnez **l’étendue** en cliquant sur les points de suspension et en sélectionnant un groupe d’administration ou un abonnement. Sélectionnez éventuellement un groupe de ressources. Une étendue détermine les ressources ou le regroupement de ressources sur lequel la stratégie est appliquée. Cliquez ensuite sur **Sélectionner** dans le bas de la page **Étendue**.

   Cet exemple utilise l’abonnement **Parnell Aerospace**. Votre abonnement sera différent.

1. Vous pouvez exclure des ressources en fonction de **l’étendue**. Les **exclusions** commencent à un niveau inférieur à celui de **l’étendue**. Les **exclusions** étant facultatives, laissez ce champ vide pour l’instant.

1. Sélectionnez les points de suspension de **Définition de stratégie** pour ouvrir la liste des définitions disponibles. Azure Policy est fourni avec des définitions de stratégie intégrées que vous pouvez utiliser. De nombreuses définitions de stratégie sont disponibles, par exemple :

   - Enforce tag and its value
   - Apply tag and its value
   - Hériter d’une étiquette du groupe de ressources en cas d’absence

   Pour obtenir une liste partielle des stratégies intégrées disponibles, consultez [Exemples Azure Policy](../../../governance/policy/samples/index.md).

1. Dans la liste des définitions de stratégie, recherchez la définition _\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Windows_ si vous avez activé l’agent Serveurs avec Arc sur une machine Windows. Pour un ordinateur Linux, recherchez la définition de stratégie correspondante _\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux_. Cliquez sur cette stratégie, puis sur **Sélectionner**.

   :::image type="content" source="./media/tutorial-assign-policy-portal/select-available-definition.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

1. Le **Nom de l’attribution** est automatiquement rempli avec le nom de stratégie que vous avez sélectionné, mais vous pouvez le modifier. Pour cet exemple, laissez _\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Windows_ ou _\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux_ en fonction de celui que vous avez sélectionné. Vous pouvez également ajouter une **Description** (facultatif). La description fournit des détails sur cette affectation de stratégie.
   Le champ **Affectée par** est automatiquement renseigné en fonction de l’utilisateur connecté. Ce champ étant facultatif, vous pouvez entrer des valeurs personnalisées.

1. Laissez la case **Créer une identité managée** non cochée. Vous _devez_ la cocher si la stratégie ou l’initiative inclut une stratégie avec l’effet [deployIfNotExists](../../../governance/policy/concepts/effects.md#deployifnotexists). La stratégie utilisée dans ce guide de démarrage rapide n'étant pas concernée, ne cochez pas la case. Pour plus d’informations, consultez [Identités managées](../../../active-directory/managed-identities-azure-resources/overview.md) et [Fonctionnement de la sécurité par correction](../../../governance/policy/how-to/remediate-resources.md#how-remediation-security-works).

1. Cliquez sur **Affecter**.

Vous êtes maintenant prêt à identifier les ressources non conformes pour comprendre l’état de conformité de votre environnement.

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Sélectionnez **Conformité** dans la partie gauche de la page. Recherchez ensuite l’affectation de stratégie **\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Windows** ou **\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux** que vous avez créée.

:::image type="content" source="./media/tutorial-assign-policy-portal/policy-compliance.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

Si des ressources existantes ne sont pas conformes à cette nouvelle affectation, elles apparaissent sous **Ressources non conformes**.

Si une condition est évaluée par rapport à vos ressources existantes et génère la valeur true, ces ressources sont marquées comme non conformes à la stratégie. Le tableau suivant montre comment les différents effets des stratégies fonctionnent avec l’évaluation des conditions pour l’état de conformité résultant. Même si vous ne voyez pas la logique d’évaluation dans le portail Azure, les résultats de l’état de conformité sont affichés. Le résultat d’état de conformité est soit conforme, soit non conforme.

| **État de la ressource** | **Effet** | **Évaluation de la stratégie** | **État de conformité** |
| --- | --- | --- | --- |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | True | Non conforme |
| Exists | Deny, Audit, Append\*, DeployIfNotExist\*, AuditIfNotExist\* | False | Conforme |
| Nouveau | Audit, AuditIfNotExist\* | True | Non conforme |
| Nouveau | Audit, AuditIfNotExist\* | False | Conforme |

\* Les effets Append, DeployIfNotExist et AuditIfNotExist nécessitent que l’instruction IF ait la valeur TRUE.
Les effets nécessitent également que la condition d’existence ait la valeur FALSE pour être non conformes. Lorsque la valeur est TRUE, la condition IF déclenche l’évaluation de la condition d’existence pour les ressources associées.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer l’affectation créée, procédez comme suit :

1. Sélectionnez **Conformité** (ou **Affectations**) sur le côté gauche de la page Azure Policy et recherchez l’affectation de stratégie **\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Windows** ou **\[Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux** que vous avez créée.

1. Cliquez avec le bouton droit sur l’affectation de stratégie, puis sélectionnez **Supprimer l’attribution**.

   :::image type="content" source="./media/tutorial-assign-policy-portal/delete-assignment.png" alt-text="Rechercher Stratégie dans Tous les services" border="false":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez affecté une définition de stratégie à une étendue et vous avez évalué son rapport de conformité. La définition de stratégie permet de vérifier que toutes les ressources dans l’étendue sont conformes, ainsi que d’identifier celles qui ne le sont pas. Vous êtes maintenant prêt à superviser votre machine Serveurs avec Azure Arc avec Azure Monitor pour machines virtuelles.

Pour savoir comment superviser et afficher les performances, les processus en cours d’exécution et leurs dépendances à partir de votre machine, passez au tutoriel :

> [!div class="nextstepaction"]
> [Activer Azure Monitor pour machines virtuelles](tutorial-enable-vm-insights.md)