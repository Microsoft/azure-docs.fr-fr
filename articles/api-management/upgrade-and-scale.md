---
title: Mettre à niveau une instance du service Gestion des API Azure et la mettre à l’échelle | Microsoft Docs
description: Cette rubrique explique comment mettre à niveau et mettre à l’échelle une instance du service Gestion des API Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 466af29a3f073f73d4a0fc62d74709cd59f82425
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183785"
---
# <a name="upgrade-and-scale-an-azure-api-management-instance"></a>Mettre à niveau une instance du service Gestion des API Azure et la mettre à l’échelle  

Les clients peuvent mettre à l’échelle une instance du service Gestion des API Azure en ajoutant ou en supprimant des unités. Une **unité** est constituée de ressources Azure dédiées et inclut une capacité de gestion de charge définie, exprimée sous la forme d’un certain nombre d’appels d’API par mois. Ce nombre ne représente pas une limite d’appel, mais plutôt une valeur de débit maximal pour permettre une planification approximative de la capacité. Le débit et la latence réels dépendent de nombreux facteurs, tels que le nombre et le taux de connexions simultanées, le type et le nombre de stratégies configurées, les tailles des requêtes et des réponses et la latence du backend.

La capacité et le prix de chaque unité dépendent du **niveau** auquel se trouve l’unité. Vous avez le choix entre quatre niveaux : **Développeur**, **De base**, **Standard**, **Premium**. Si vous avez besoin d’augmenter la capacité d’un service au sein d’un niveau, vous devez ajouter une unité. Si le niveau actuellement sélectionné dans votre instance Gestion des API ne permet pas l’ajout d’unités supplémentaires, vous devez effectuer une mise à niveau vers une couche de niveau supérieur.

Le prix de chaque unité et les fonctionnalités disponibles (par exemple, le déploiement dans plusieurs régions) dépendent de la couche que vous avez choisie pour votre instance de la Gestion des API. L’article relatif aux [informations de tarification](https://azure.microsoft.com/pricing/details/api-management/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) explique le prix unitaire et les fonctionnalités que vous obtenez dans chaque couche. 

>[!NOTE]
>L’article relatif aux [informations de tarification](https://azure.microsoft.com/pricing/details/api-management/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) indique la valeur de capacité unitaire approximative dans chaque couche. Pour obtenir des chiffres plus précis, vous devez consulter un scénario réaliste pour vos API. Consultez l’article [Capacité d’une instance du service Gestion des API Azure](api-management-capacity.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans cet article, vous devez :

+ Avoir un abonnement Azure actif.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ Disposer d’une instance de la Gestion des API. Pour en savoir plus, voir [Créer une instance de gestion des API Azure](get-started-create-service-instance.md).

+ Comprendre le concept de [Capacité d’une instance du service Gestion des API Azure](api-management-capacity.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="upgrade-and-scale"></a>Mise à niveau et mise à l’échelle  

Vous avez le choix entre quatre niveaux : **Développeur**, **De base**, **Standard** et **Premium**. Le niveau **Développeur** doit être utilisé pour évaluer le service ; il ne doit pas être utilisé pour un environnement de production. Le niveau **Développeur** n’est associé à aucun SLA, et vous ne pouvez pas le mettre à l’échelle (ajouter/supprimer des unités). 

Les niveaux **De base**, **Standard** et **Premium** sont des niveaux de production, associés à des contrats SLA. Ils peuvent faire l’objet d’une mise à l’échelle. De tous les niveaux assortis d’un contrat SLA, le niveau **De base** est le plus abordable et peut monter jusqu’à deux unités, tandis que le niveau **Standard** peut monter jusqu’à quatre unités. Vous pouvez ajouter n’importe quel nombre d’unités au niveau **Premium**.

Le niveau **Premium** vous permet de distribuer une seule instance du service Gestion des API Azure sur n’importe quel nombre de régions Azure. Quand vous créez un service Gestion des API Azure, l’instance contient une seule unité et se trouve dans une seule région Azure. La région initiale est désignée comme la région **principale**. D’autres régions peuvent être facilement ajoutées. Lorsque vous ajoutez une région, vous spécifiez le nombre d’unités que vous souhaitez allouer. Par exemple, vous pouvez avoir une unité dans la région **principale** et cinq unités, dans une autre région. Vous pouvez adapter le nombre d’unités au trafic dans chaque région. Pour en savoir plus, découvrez [comment déployer une instance de service Gestion des API Azure dans plusieurs régions Azure](api-management-howto-deploy-multi-region.md).

Vous pouvez effectuer la mise à niveau et la rétrogradation vers n’importe quel niveau. Le passage à un niveau supérieur ou inférieur peut entraîner la suppression de certaines fonctionnalités, comme les réseaux virtuels ou les déploiements multirégions, notamment lors du passage du niveau Premium au niveau Standard ou De base.

> [!NOTE]
> Le processus de mise à niveau ou de mise à l’échelle peut durer entre 15 et 45 minutes. Vous recevez une notification lorsque le processus est terminé.

> [!NOTE]
> Le service Gestion des API dans le niveau **Consommation** est automatiquement mis à l’échelle en fonction du trafic.

## <a name="scale-your-api-management-service"></a>Mettre à l’échelle le service Gestion des API

![Mettre à l’échelle le service Gestion des API dans le portail Azure](./media/upgrade-and-scale/portal-scale.png)

1. Accédez au service Gestion des API dans le [portail Azure](https://portal.azure.com/).
2. Dans le menu, sélectionnez **Emplacements**.
3. Cliquez sur la ligne correspondant à l’emplacement que vous souhaitez mettre à l’échelle.
4. Spécifiez le nouveau nombre d’**unités**. Pour cela, utilisez le curseur ou tapez le nombre directement.
5. Cliquez sur **Appliquer**.

## <a name="change-your-api-management-service-tier"></a>Changer le niveau de service de la Gestion des API

1. Accédez au service Gestion des API dans le [portail Azure](https://portal.azure.com/).
2. Dans le menu, cliquez sur **Niveau tarifaire**.
3. Dans la liste déroulante, sélectionnez le niveau de service souhaité. Utilisez le curseur pour spécifier l’échelle du service Gestion des API après la modification.
4. Cliquez sur **Enregistrer**.

## <a name="downtime-during-scaling-up-and-down"></a>Temps d’arrêt pendant la montée et la descente en puissance
Si vous effectuez une mise à l’échelle à partir de ou vers le niveau développeur, il y aura des temps d’arrêt. Dans le cas contraire, il n’y a aucun temps d’arrêt. 

## <a name="compute-isolation"></a>Isolation du calcul
Si vos exigences de sécurité incluent l’[isolation de calcul](../azure-government/azure-secure-isolation-guidance.md#compute-isolation), vous pouvez utiliser le niveau de tarification **Isolé**. Ce niveau garantit que les ressources de calcul d’une instance de service de gestion des API consomment l’intégralité de l’hôte physique et fournissent le niveau d’isolation nécessaire pour prendre en charge les charges de travail de niveau 5 du Ministère de la défense des États-Unis (IL5), par exemple. Pour obtenir l’accès au niveau isolé, [créez un ticket de support](../azure-portal/supportability/how-to-create-azure-support-request.md). 



## <a name="next-steps"></a>Étapes suivantes

- [Comment déployer une instance de service Gestion des API Azure dans plusieurs régions Azure](api-management-howto-deploy-multi-region.md)
- [Mettre à l’échelle automatiquement une instance du service Gestion des API Azure](api-management-howto-autoscale.md)
- [Optimiser et réduire vos coûts de cloud](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)