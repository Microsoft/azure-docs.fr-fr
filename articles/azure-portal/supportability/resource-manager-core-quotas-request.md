---
title: Demandes d’augmentation des quotas de processeurs virtuels pour Azure Resource Manager
description: Demandes d’augmentation des quotas de processeurs virtuels pour Azure Resource Manager
author: sowmyavenkat86
ms.author: svenkat
ms.date: 01/27/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: 2580da2a4ac7b943dee3e5e6ff8bdbd49664505b
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745245"
---
# <a name="quota-increase-requests"></a>Demandes d’augmentation de quota

Les quotas de processeurs virtuels de Resource Manager pour les machines virtuelles et les groupes de machines virtuelles identiques sont appliqués sur deux niveaux pour chaque abonnement, dans chaque région.

Le premier niveau correspond à la limite du nombre total de processeurs virtuels régionaux dans toutes les séries de machines virtuelles. Le deuxième niveau correspond à la limite du nombre de processeurs virtuels pour chaque série de machines virtuelles, par exemple les processeurs virtuels de série D. Chaque fois qu’une nouvelle machine virtuelle doit être déployée, la somme de l’utilisation des processeurs virtuels nouveaux et existants pour la série de machines virtuelles concernée ne doit pas dépasser le quota de processeurs virtuels approuvé pour cette série de machines virtuelles. De plus, le nombre total de processeurs virtuels nouveaux et existants déployés sur toutes les séries de machines virtuelles ne doit pas dépasser le quota de processeurs virtuels régionaux total approuvé pour l’abonnement. Si l’un de ces quotas est dépassé, le déploiement des machines virtuelles n’est pas autorisé.
Vous pouvez demander une augmentation de la limite de quota de processeurs virtuels pour une série de machines virtuelles à partir du portail Azure. Une augmentation du quota d’une série de machines virtuelles augmente automatiquement la limite totale de processeurs virtuels régionaux du même montant.

Quand un abonnement est créé, la valeur par défaut du total de processeurs virtuels régionaux peut ne pas être égale à la somme des quotas de processeurs virtuels par défaut pour toutes les séries de machines virtuelles individuelles. De ce fait, l’abonnement peut avoir un quota suffisant pour chacune des séries de machines virtuelles à déployer. Son quota peut être insuffisant pour le total des processeurs virtuels régionaux de tous les déploiements. Dans ce cas, vous devez envoyer une demande d’augmentation explicite de la limite totale des processeurs virtuels régionaux. La limite du nombre de processeurs virtuels régionaux ne peut pas dépasser la somme des quotas approuvés sur toutes les séries de machines virtuelles pour la région.

> [!NOTE]
> Si vous souhaitez élever la limite ou le quota au-dessus de la limite par défaut, [ouvrez une demande de service clientèle en ligne gratuitement](../../azure-resource-manager/templates/error-resource-quota.md#solution).

Pour en savoir plus sur les quotas, consultez [Quotas de processeurs virtuels pour les machines virtuelles](../../virtual-machines/windows/quotas.md) et [Abonnement Azure et limites, quotas et contraintes de service](../../azure-resource-manager/management/azure-subscription-service-limits.md).