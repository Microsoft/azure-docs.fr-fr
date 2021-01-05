---
title: Évaluer l’impact d’une nouvelle définition Azure Policy
description: Découvrez le processus à suivre quand vous introduisez une nouvelle définition de stratégie dans votre environnement Azure.
ms.date: 10/05/2020
ms.topic: conceptual
ms.openlocfilehash: 9d73d703c38dce1335a471bfad9171d8b30a83c5
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91873865"
---
# <a name="evaluate-the-impact-of-a-new-azure-policy-definition"></a>Évaluer l’impact d’une nouvelle définition Azure Policy

Azure Policy est un outil puissant qui permet de gérer vos ressources Azure en respectant les standards du secteur et en répondant aux exigences de conformité. Lorsque des personnes, des processus ou des pipelines créent ou mettent à jour des ressources, Azure Policy examine la requête impliquée. Lorsque l’effet de la définition de stratégie est [Modify](./effects.md#modify), [Append](./effects.md#deny) ou [DeployIfNotExists](./effects.md#deployifnotexists), Azure Policy modifie la requête ou y ajoute des éléments. Lorsque l’effet de la définition de stratégie est [Audit](./effects.md#audit) ou [AuditIfNotExists](./effects.md#auditifnotexists), Azure Policy provoque la création d’une entrée dans le journal d’activité pour les nouvelles ressources et celles mises à jour. Enfin, lorsque l’effet de la définition de stratégie est [Deny](./effects.md#deny), Azure Policy arrête la création ou la modification de la requête.

Ces résultats sont exactement ceux que vous souhaitez lorsque vous savez que la stratégie est définie correctement. Toutefois, il est important de vérifier qu’une nouvelle stratégie fonctionne comme prévu avant de l’autoriser à modifier ou à bloquer un travail. Cette vérification vise à garantir que seules les ressources prévues sont déterminées comme non conformes et qu’aucune ressource conforme n’a été incluse dans les résultats (c’est ce qu’on appelle un _faux positif_).

L’approche recommandée pour vérifier une nouvelle définition de stratégie est la suivante :

- Définir rigoureusement sa stratégie
- Auditer les ressources existantes
- Auditer les requêtes de ressources nouvelles ou mises à jour
- Déployer la stratégie sur les ressources
- Supervision continue

## <a name="tightly-define-your-policy"></a>Définir rigoureusement sa stratégie

Il est important de comprendre comment la stratégie d’entreprise est implémentée en tant que définition de stratégie, et comment sont liés les ressources Azure et les autres services Azure. Pour cela, vous devez [comprendre les exigences](../tutorials/create-custom-policy-definition.md#identify-requirements) et [déterminer les propriétés des ressources](../tutorials/create-custom-policy-definition.md#determine-resource-properties).
Toutefois, il est également important de voir au-delà de la définition de votre stratégie d’entreprise. Par exemple, votre stratégie indique-t-elle quelque chose comme « Toutes les machines virtuelles doivent... » ? Qu’en est-il des autres services Azure qui utilisent des machines virtuelles, tels que HDInsight ou AKS ? Lorsque vous définissez une stratégie, vous devez réfléchir à la façon dont elle va impacter les ressources qui sont utilisées par d’autres services.

Pour cette raison, vos définitions de stratégie doivent être rigoureusement définies et focalisées sur les ressources et les propriétés dont vous avez besoin pour évaluer la conformité.

## <a name="audit-existing-resources"></a>Auditer les ressources existantes

Avant de chercher à gérer les ressources nouvelles ou mises à jour avec votre nouvelle définition de stratégie, il est préférable de voir comment elle évalue un sous-ensemble limité de ressources existantes, comme un groupe de ressources de test. Utilisez le [mode d’application](./assignment-structure.md#enforcement-mode) _Désactivé_ (DoNotEnforce) dans votre attribution de stratégie afin d’empêcher le déclenchement de l’[effet](./effects.md) ou la création d’entrées dans le journal d’activité.

Cette étape vous permet d’évaluer les résultats de conformité de la nouvelle stratégie sur les ressources existantes, sans impacter le workflow. Vérifiez qu’aucune ressource conforme n’est marquée comme non conforme (_faux positif_) et que toutes les ressources dont il est prévu qu’elles soient non conformes sont marquées correctement.
Une fois que le sous-ensemble de ressources d’origine est validé, étendez progressivement l’évaluation à toutes les ressources existantes.

Le fait de procéder ainsi pour l’évaluation des ressources existantes permet de corriger les ressources non conformes avant l’implémentation complète de la nouvelle stratégie. Ce nettoyage peut être effectué manuellement ou par le biais d’une [tâche de correction](../how-to/remediate-resources.md) si l’effet de la définition de stratégie est _DeployIfNotExists_.

## <a name="audit-new-or-updated-resources"></a>Auditer les ressources nouvelles ou mises à jour

Une fois que vous avez validé votre nouvelle définition de stratégie, vous pouvez vous intéresser à l’impact de la stratégie lors de la création ou de la mise à jour des ressources. Si la définition de stratégie prend en charge le paramétrage des effets, utilisez [Audit](./effects.md#audit). Cette configuration vous permet de superviser la création et la mise à jour des ressources pour voir si la nouvelle définition de stratégie déclenche la création d’une entrée dans le journal d’activité Azure en cas de ressource non conforme, sans impacter les tâches ou les requêtes existantes.

Il est recommandé de mettre à jour et de créer de nouvelles ressources correspondant à votre définition de stratégie pour voir si l’effet _Audit_ se déclenche comme prévu. Surveillez les requêtes de ressources qui ne doivent pas être affectées par la nouvelle définition de stratégie qui déclenche l’effet _Audit_.
Ces ressources impactées constituent un autre exemple de _faux positifs_, et doivent être corrigées dans la définition de la stratégie avant l’implémentation complète.

Si la définition de la stratégie est modifiée à cette phase du test, il est recommandé de commencer le processus de validation par l’audit des ressources existantes. La modification d’une définition de stratégie en raison d’un _faux positif_ concernant des ressources nouvelles ou mises à jour est susceptible d’impacter les ressources existantes.

## <a name="deploy-your-policy-to-resources"></a>Déployer la stratégie sur les ressources

Une fois effectuée la validation de votre nouvelle définition de stratégie avec les ressources existantes et les requêtes de ressources nouvelles ou mises à jour, vous pouvez commencer à implémenter la stratégie. Il est recommandé de créer d’abord l’attribution de stratégie de la nouvelle définition de stratégie pour un sous-ensemble de ressources, comme un groupe de ressources. Après avoir validé le déploiement initial, étendez la stratégie à des niveaux de plus en plus larges, tels que des abonnements ou des groupes d’administration. Pour ce faire, vous devez supprimer l’attribution et en créer une nouvelle pour les étendues cibles jusqu’à ce qu’elle soit attribuée à l’étendue complète des ressources qui sont concernées par votre nouvelle définition de stratégie.

Pendant le déploiement, si des ressources qui doivent être exemptées de votre nouvelle définition de stratégie sont détectées, traitez-les de l’une des manières suivantes :

- Mettez à jour la définition de stratégie pour qu’elle soit plus explicite afin de réduire tout impact non voulu.
- Modifiez l’étendue de l’attribution de stratégie (en supprimant et en créant une nouvelle attribution).
- Ajoutez le groupe de ressources à la liste d’exclusion pour l’attribution de stratégie.

Toute modification apportée à l’étendue (niveau ou exclusions) doit être entièrement validée et communiquée à vos organisations de sécurité et de conformité afin de garantir une couverture complète.

## <a name="monitor-your-policy-and-compliance"></a>Superviser la stratégie et la conformité

L’implémentation et l’attribution de votre définition de stratégie ne constituent pas les dernières étapes. Supervisez continuellement le niveau de [conformité](../how-to/get-compliance-data.md) des ressources par rapport à votre nouvelle définition de stratégie, et configurez des [alertes et des notifications Azure Monitor](../../../azure-monitor/platform/alerts-overview.md) pour les appareils non conformes qui sont détectés. Il est également recommandé de planifier l’évaluation de la définition de stratégie et des attributions associées pour vérifier que la définition de stratégie répond aux exigences de conformité et à la stratégie d’entreprise. Les stratégies doivent être supprimées lorsque vous n’en avez plus besoin. Les stratégies doivent également être mises à jour de temps en temps, à mesure que les ressources Azure sous-jacentes évoluent et que de nouvelles propriétés et fonctionnalités leur sont ajoutées.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [structure des définitions de stratégies](./definition-structure.md)
- En savoir plus sur la [structure des attributions de stratégies](./assignment-structure.md)
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).