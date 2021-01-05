---
title: Stratégies recommandées pour les services Azure
description: Décrit comment rechercher et appliquer les stratégies recommandées pour les services Azure, tels que Machines virtuelles Azure.
ms.date: 09/02/2020
ms.topic: conceptual
ms.customer: generated
ms.openlocfilehash: 019541f034ce8f0c3728c38d2ae4425308b4e2a1
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350277"
---
# <a name="recommended-policies-for-azure-services"></a>Stratégies recommandées pour les services Azure

Les clients qui débutent avec Azure Policy recherchent souvent des définitions de stratégies communes pour gérer et régir leurs ressources. Les **stratégies recommandées** d’Azure Policy fournissent une liste ciblée des définitions de stratégies courantes avec lesquelles commencer. L’expérience des **stratégies recommandées** pour une ressource prise en charge est incorporée dans l’expérience du portail pour cette ressource.

Pour obtenir des éléments intégrés Azure Policy supplémentaires, consultez [Définitions intégrées d’Azure Policy](../samples/built-in-policies.md).

## <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Les **stratégies recommandées** pour [machines virtuelles Azure](../../../virtual-machines/index.yml) figurent dans la page **Vue d’ensemble** des machines virtuelles et sous l’onglet **Fonctionnalités**. Dans la carte _Azure Policy_, sélectionnez le texte « Non configuré » ou « N° assigné » pour ouvrir un volet latéral avec les stratégies recommandées. Toute définition de stratégie déjà attribuée à une étendue dont la machine virtuelle est membre est grisée. Sélectionnez les stratégies recommandées à appliquer à cette machine virtuelle et sélectionnez **Attribuer des stratégies** pour créer une attribution pour chacune d’elles.

Lorsqu’une organisation atteint sa maturité en [organisant ses ressources et la hiérarchie des ressources](/azure/cloud-adoption-framework/ready/azure-best-practices/organize-subscriptions), il est recommandé de faire passer ces attributions de stratégie d’un niveau par ressource au niveau de l’abonnement ou du [groupe d’administration](../../management-groups/index.yml).

### <a name="azure-virtual-machines-recommended-policies"></a>Stratégies recommandées des machines virtuelles Azure

|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Auditer des machines virtuelles pour lesquelles la reprise d’activité après sinistre n’est pas configurée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Auditez les machines virtuelles configurées sans reprise d’activité. Pour en savoir plus sur la reprise d’activité, consultez [https://aka.ms/asr-doc](../../../site-recovery/index.yml). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |
|[Faire l’audit des machines virtuelles n’utilisant aucun disque managé](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Cette stratégie fait l’audit des machines virtuelles n’utilisant pas de disque managé |audit |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |
|[La sauvegarde Azure doit être activée pour les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Cette stratégie permet de vérifier si le service Sauvegarde Azure est activé pour toutes les machines virtuelles. Sauvegarde Azure est une solution de sauvegarde économique en un clic, qui simplifie la récupération des données et peut être activée plus facilement que d’autres services de sauvegarde cloud. |AuditIfNotExists, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Compréhension des effets de Policy](./effects.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).