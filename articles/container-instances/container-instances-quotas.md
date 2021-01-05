---
title: Quotas de service et disponibilité des régions
description: Quotas, limites et disponibilité des régions du service Azure Container Instances.
ms.topic: article
ms.date: 07/22/2020
ms.openlocfilehash: eaaa8e0b2d72aaea546a1bc351da40932c1deb14
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87384824"
---
# <a name="quotas-and-limits-for-azure-container-instances"></a>Quotas et limites pour Azure Container Instances

Tous les services Azure incluent certains quotas et limites par défaut pour les ressources et fonctionnalités. Cet article décrit en détail les quotas et limites par défaut pour Azure Container Instances.

La disponibilité des ressources de calcul, de mémoire et de stockage pour Azure Container Instances varie selon la région et le système d’exploitation. Pour plus d’informations, consultez [Disponibilité des ressources pour Azure Container Instances](container-instances-region-availability.md).

Utilisez l’API [Lister l’utilisation](/rest/api/container-instances/location/listusage) pour vérifier l’utilisation actuelle des quotas dans une région pour un abonnement.

## <a name="service-quotas-and-limits"></a>Quotas et limites du service

[!INCLUDE [container-instances-limits](../../includes/container-instances-limits.md)]

## <a name="next-steps"></a>Étapes suivantes

Certains quotas et limites par défaut peuvent être augmentés. Pour demander une augmentation d’une ou plusieurs ressources qui peuvent prendre en charge cette augmentation, veuillez envoyer une [demande de support Azure][azure-support] (sélectionnez « Quota » comme **Type de problème**).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
