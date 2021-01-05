---
title: Configurer le niveau PremiumV3
description: Découvrez comment améliorer les performances de vos applications web, mobiles et API dans Azure App Service en les faisant évoluer vers le nouveau niveau tarifaire PremiumV3.
keywords: app service, azure app service, mise à l'échelle, évolutif, plan app service, coût d'app service
ms.assetid: ff00902b-9858-4bee-ab95-d3406018c688
ms.topic: article
ms.date: 10/01/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 7ade24b6478f78a51e0be68ae69ae0b076ecff1f
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607872"
---
# <a name="configure-premiumv3-tier-for-azure-app-service"></a>Configurer le niveau PremiumV3 pour Azure App Service

Le nouveau niveau tarifaire **PremiumV3** offre des processeurs plus rapides, un stockage SSD et le quadruple de ratio mémoire-cœur des niveaux tarifaires précédents (double le niveau **PremiumV2**). Avec cet avantage de performances, vous pouvez économiser de l’argent en exécutant vos applications sur moins d’instances. Dans cet article, vous allez apprendre à créer une application dans le niveau **PremiumV3** ou à faire évoluer une application vers le niveau **PremiumV3**.

## <a name="prerequisites"></a>Prérequis

Pour faire évoluer une application vers le niveau **PremiumV3**, vous devez disposer d’une application Azure App Service qui s’exécute dans un niveau tarifaire inférieur à **PremiumV3** et dans un déploiement App Service prenant en charge PremiumV3.

<a name="availability"></a>

## <a name="premiumv3-availability"></a>Disponibilité de PremiumV3

Le niveau **PremiumV3** est disponible tant pour les applications natives que conteneur, y compris les conteneurs Windows et les conteneurs Linux.

> [!NOTE]
> Tous les conteneurs Windows exécutés dans le niveau **Conteneur Premium** pendant la période de préversion continuent à fonctionner en l’état, mais le niveau **Conteneur Premium** continue de rester en préversion. Le niveau **PremiumV3** remplace officiellement le niveau **Conteneur Premium**. 

**PremiumV3** est disponible dans certaines régions Azure et la disponibilité dans d’autres régions est continuellement ajoutée. Pour savoir si ce niveau est disponible dans votre région, exécutez la commande Azure CLI suivante dans [Azure Cloud Shell](../cloud-shell/overview.md) :

```azurecli-interactive
az appservice list-locations --sku P1V3
```

<a name="create"></a>

## <a name="create-an-app-in-premiumv3-tier"></a>Créer une application dans le niveau PremiumV3

Le niveau tarifaire d’une application App Service est défini dans le [plan App Service](overview-hosting-plans.md) dans lequel elle s’exécute. Vous pouvez créer un plan App Service de façon séparée ou dans le cadre de la création d’une application.

Quand vous configurez le plan App Service dans le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, sélectionnez **Niveau tarifaire**. 

Sélectionnez **Production**, puis **P1V3**, **P2V3** ou **P3V3**, et cliquez ensuite sur **Appliquer**.

![Capture d’écran montrant les niveaux tarifaires recommandés pour votre application.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

> [!IMPORTANT] 
> Si les options **P1V3**, **P2V3** et **P3V3** n’apparaissent pas ou sont grisées, il est probable que **PremiumV3** ne soit pas disponible dans le déploiement App Service sous-jacent qui contient le plan App Service. Pour plus d’informations, voir [Monter en puissance à partir d’une combinaison groupe de ressources/région non prise en charge](#unsupported).

## <a name="scale-up-an-existing-app-to-premiumv3-tier"></a>Faire évoluer une application existante vers le niveau PremiumV3

Avant de faire évoluer une application existante vers le niveau **PremiumV3**, vérifiez la disponibilité de **PremiumV3**. Pour plus d’informations, consultez [Disponibilité de PremiumV3](#availability). S’il n’est pas disponible, voir [Monter en puissance à partir d’une combinaison groupe de ressources/région non prise en charge](#unsupported).

Selon votre environnement d’hébergement, la montée en puissance peut nécessiter des étapes supplémentaires. 

Dans le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, ouvrez la page de votre application App Service.

Dans le volet de navigation de gauche dans la page de votre application App Service, sélectionnez **Monter en puissance (plan App Service)** .

![Capture d’écran montrant comment effectuer un scale-up de votre plan App Service.](media/app-service-configure-premium-tier/scale-up-tier-portal.png)

Sélectionnez **Production**, puis **P1V3**, **P2V3** ou **P3V3**, et cliquez ensuite sur **Appliquer**.

![Capture d’écran montrant les niveaux tarifaires recommandés pour votre application.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

Si l’opération se termine sans erreur, la page de présentation de votre application montre que l’application s’exécute maintenant dans le niveau **PremiumV3**.

![Capture d’écran montrant le niveau tarifaire PremiumV3 sur la page de présentation de votre application.](media/app-service-configure-premium-tier/finished.png)

### <a name="if-you-get-an-error"></a>Si vous obtenez une erreur

Certains plans App Service ne peuvent pas évoluer vers le niveau PremiumV3 si le déploiement App Service sous-jacent ne le prend pas en charge. Pour plus d’informations, voir [Monter en puissance à partir d’une combinaison groupe de ressources/région non prise en charge](#unsupported).

<a name="unsupported"></a>

## <a name="scale-up-from-an-unsupported-resource-group-and-region-combination"></a>Monter en puissance à partir d’une combinaison groupe de ressources/région non prise en charge

Si votre application s’exécute dans un déploiement App Service où **PremiumV3** n’est pas disponible, ou dans une région qui ne prend pas encore en charge **PremiumV3**, vous devez la redéployer pour tirer parti de **PremiumV3**.  Deux options s'offrent à vous :

- Créer une application dans un nouveau groupe de ressources et avec un nouveau plan App Service. Lors de la création du plan App Service, sélectionnez un niveau **PremiumV3**. Cette étape permet de s’assurer que le plan App Service est déployé dans une unité de déploiement qui prend en charge **PremiumV3**. Redéployez ensuite le code de votre application dans l’application qui vient d’être créée. Même si vous faites passer le plan App Service à un niveau inférieur pour réduire les coûts, vous pourrez toujours repasser au niveau **PremiumV3**, car l’unité de déploiement le prend en charge.
- Si votre application s’exécute déjà dans un niveau **Premium** existant, vous pouvez la cloner avec tous ses paramètres, ses chaînes de connexion et sa configuration de déploiement dans un nouveau groupe de ressources sur un nouveau plan App Service qui utilise **PremiumV3**.

    ![Capture d’écran montrant comment cloner votre application.](media/app-service-configure-premium-tier/clone-app.png)

    Sur la page **Cloner l’application**, vous pouvez créer un plan App Service avec **PremiumV3** dans la région de votre choix, puis spécifier les paramètres d’application et la configuration à cloner.

## <a name="moving-from-premium-container-to-premium-v3-sku"></a>Passage d’un niveau tarifaire Conteneur Premium à Premium V3

Si vous avez une application qui utilise le niveau tarifaire Premium Container en préversion et que vous souhaitez passer au nouveau niveau tarifaire Premium V3, vous devez la redéployer pour tirer parti de **PremiumV3**. Pour ce faire, consultez la première option dans la section [Effectuer un scale-up à partir d’une combinaison groupe de ressources/région non prise en charge](#scale-up-from-an-unsupported-resource-group-and-region-combination)

## <a name="automate-with-scripts"></a>Automatiser des tâches à l’aide de scripts

Vous pouvez automatiser la création d’applications dans le niveau **PremiumV3** à l’aide de scripts, en utilisant [Azure CLI](/cli/azure/install-azure-cli) ou [Azure PowerShell](/powershell/azure/).

### <a name="azure-cli"></a>Azure CLI

La commande suivante crée un plan App Service dans _P1V3_. Vous pouvez l’exécuter dans Cloud Shell. Les options possibles pour `--sku` sont P1V3, _P2V3_ et _P3V3_.

```azurecli-interactive
az appservice plan create \
    --resource-group <resource_group_name> \
    --name <app_service_plan_name> \
    --sku P1V3
```

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

La commande suivante crée un plan App Service dans _P1V3_. Les options possibles pour `-WorkerSize` sont _Small_, _Medium_ et _Large_.

```powershell
New-AzAppServicePlan -ResourceGroupName <resource_group_name> `
    -Name <app_service_plan_name> `
    -Location <region_name> `
    -Tier "PremiumV3" `
    -WorkerSize "Small"
```

## <a name="more-resources"></a>Plus de ressources

[Faire évoluer une application dans Azure](manage-scale-up.md)
[Ajustement manuel ou automatique du nombre d’instances](../azure-monitor/platform/autoscale-get-started.md)
