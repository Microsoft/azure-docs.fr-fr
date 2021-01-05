---
title: 'Démarrage rapide : Configurer la haute disponibilité avec Azure Front Door - Azure CLI'
description: Ce guide de démarrage rapide vous montre comment utiliser Azure Front Door pour créer une application web mondiale hautement disponible et très performante à l’aide d’Azure CLI.
services: front-door
author: duongau
manager: KumudD
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/21/2020
ms.author: duau
ms.openlocfilehash: e01b56f88270348167ef1eef9d913e65074d9247
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327251"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-cli"></a>Démarrage rapide : Créer une porte d’entrée pour une application web mondiale hautement disponible à l’aide d’Azure CLI

Démarrez avec Azure Front Door en utilisant Azure CLI pour créer une application web globale hautement disponible et très performante.

La porte d’entrée dirige le trafic web vers des ressources spécifiques dans un pool de back-ends. Vous définissez le domaine front-end, ajoutez des ressources à un pool de back-ends et créer une règle de routage. Cet article utilise une configuration simple d’un pool de back-ends avec deux ressources d’application web et une règle de routage unique utilisant le chemin par défaut correspondant à « /* ».

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure CLI installé localement ou Azure Cloud Shell
- Assurez-vous que l’extension Front Door est ajoutée à votre interface de ligne de commande Azure

```azurecli-interactive 
az extension add --name front-door
```

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser l’interface CLI en local, ce guide de démarrage nécessite Azure CLI version 2.0.28 ou ultérieure. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Pour ce guide de démarrage rapide, vous avez besoin de deux groupes de ressources. L’un se trouve dans *USA Centre*, l’autre dans *USA Centre Sud*.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create&preserve-view=true) :

```azurecli-interactive
az group create \
    --name myRGFDCentral \
    --location centralus

az group create \
    --name myRGFDSouthCentral \
    --location southcentralus
```

## <a name="create-two-instances-of-a-web-app"></a>Créer deux instances d’une application web

Deux instances d’une application web qui s’exécutent dans différentes régions Azure sont nécessaires pour ce guide de démarrage rapide. Les deux instances de l’application web s’exécutent en mode Actif/Actif, donc l’une ou l’autre peut traiter le trafic.

Si vous n’avez pas encore d’application web, utilisez le script suivant pour configurer deux exemples d’applications web.

### <a name="create-app-service-plans"></a>Créer des plans App Service

Avant de pouvoir créer des applications web, vous avez besoin de deux plans App Service, l’un dans *USA Centre* et l’autre dans *USA Centre Sud*.

Créez les plans App Service avec la commande [az appservice plan create](/cli/azure/appservice/plan?view=azure-cli-latest#az_appservice_plan_create&preserve-view=true) :

```azurecli-interactive
az appservice plan create \
--name myAppServicePlanCentralUS \
--resource-group myRGFDCentral

az appservice plan create \
--name myAppServicePlanSouthCentralUS \
--resource-group myRGFDSouthCentral
```

### <a name="create-web-apps"></a>Créer des applications web

L’exécution des commandes suivantes crée une application web dans chacun des plans App Service à l’étape précédente. Les noms des applications web doivent être globalement uniques.

Créez une application web avec la commande [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az_webapp_create&preserve-view=true) :

```azurecli-interactive
az webapp create \
--name WebAppContoso1 \
--resource-group myRGFDCentral \
--plan myAppServicePlanCentralUS 

az webapp create \
--name WebAppContoso2 \
--resource-group myRGFDSouthCentral \
--plan myAppServicePlanSouthCentralUS
```

Prenez note du nom d’hôte par défaut de chaque application web afin de pouvoir définir les adresses de back-end au moment de déployer la porte d’entrée à l’étape suivante.

## <a name="create-the-front-door"></a>Créer la porte d’entrée

Créez une porte d’entrée de base avec les paramètres d’équilibrage de charge, la sonde d’intégrité et les règles de routage par défaut comme suit :

Créez une porte d’entrée avec la commande [az network front-door create](/cli/azure/ext/front-door/network/front-door?view=azure-cli-latest#ext_front_door_az_network_front_door_create&preserve-view=true) :

```azurecli-interactive
az network front-door create \
--resource-group myRGFDCentral \
--name contoso-frontend \
--accepted-protocols http https \
--backend-address webappcontoso1.azurewebsites.net webappcontoso2.azurewebsites.net 
```

**--resource-group :** spécifiez un groupe de ressources dans lequel vous souhaitez déployer la porte d’entrée.

**--name :** spécifiez un nom global unique pour votre porte d’entrée Azure. 

**--accepted-protocols :** les valeurs acceptées sont **http** et **https**. Si vous souhaitez utiliser les deux, spécifiez-les en les séparant par un espace.

**--backend-address :** définissez les nom d’hôte des applications web en les séparant par un espace.

Une fois le déploiement terminé, prenez note du nom d’hôte dans la section *frontEndpoints*.

## <a name="test-the-front-door"></a>Tester la porte d’entrée

Ouvrez un navigateur web et entrez le nom d’hôte obtenu à partir des commandes. La porte d’entrée dirige votre demande vers l’une des ressources back-end.

:::image type="content" source="./media/quickstart-create-front-door-cli/front-door-testing-page.png" alt-text="Page de test de la porte d’entrée":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la porte d’entrée, supprimez les deux groupes de ressources. Quand vous supprimez le groupe de ressources, vous supprimez aussi la porte d’entrée et toutes ses ressources associées. 

Pour supprimer le groupe de ressources, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest#az_group_delete&preserve-view=true) :

```azurecli-interactive
az group delete \
--name myRGFDCentral 

az group delete \
--name myRGFDSouthCentral
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :
* Front Door
* Deux applications web

Pour savoir comment ajouter un domaine personnalisé à votre porte d’entrée, passez aux tutoriels Front Door.

> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](front-door-custom-domain.md)
