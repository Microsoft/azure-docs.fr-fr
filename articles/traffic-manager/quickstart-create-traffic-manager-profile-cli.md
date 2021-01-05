---
title: 'Démarrage rapide : Créer un profil pour la haute disponibilité des applications – Azure CLI – Azure Traffic Manager'
description: Cet article de démarrage rapide décrit comment créer un profil Traffic Manager pour créer des applications web hautement disponibles à l’aide d’Azure CLI.
services: traffic-manager
author: duongau
mnager: kumud
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/09/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7dabf94c711972f9fe543edac0d7b95469fc2d35
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94661101"
---
# <a name="quickstart-create-a-traffic-manager-profile-for-a-highly-available-web-application-using-azure-cli"></a>Démarrage rapide : Créer un profil Traffic Manager pour assurer une haute disponibilité à vos applications web avec Azure CLI

Ce démarrage rapide explique comment créer un profil Traffic Manager qui assure une haute disponibilité pour votre application web.

Dans ce démarrage rapide, vous allez créer deux instances d’une application web. Chacune d’elles s’exécute dans une région Azure distincte. Vous allez créer un profil Traffic Manager en fonction de la [priorité du point de terminaison](traffic-manager-routing-methods.md#priority-traffic-routing-method). Le profil dirige le trafic utilisateur vers le site principal exécutant l’application web. Traffic Manager supervise en permanence l’application web. Si le site principal est indisponible, il assure un basculement automatique vers le site de secours.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créez un groupe de ressources avec la commande [az group create](https://docs.microsoft.com/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive

  az group create \
    --name myResourceGroup \
    --location eastus

```

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

Créez un profil Traffic Manager en utilisant la commande [az network traffic-manager profile create](https://docs.microsoft.com/cli/azure/network/traffic-manager/profile?view=azure-cli-latest#az-network-traffic-manager-profile-create) qui dirige le trafic utilisateur en fonction de la priorité du point de terminaison.

Dans l’exemple suivant, remplacez **<profile_name>** par un nom de profil Traffic Manager unique.

```azurecli-interactive

az network traffic-manager profile create \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --routing-method Priority \
    --path "/" \
    --protocol HTTP \
    --unique-dns-name <profile_name> \
    --ttl 30 \
    --port 80

```

## <a name="create-web-apps"></a>Créer des applications web

Pour ce guide de démarrage rapide, vous aurez besoin de deux instances d’une application web déployée dans deux régions Azure différentes (*USA Est* et *Europe Ouest*). Chacune servira de point de terminaison principal et de point de terminaison de basculement à Traffic Manager.

### <a name="create-web-app-service-plans"></a>Créer des plans App Service web
Créez des plans App Service web en utilisant la commande [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) pour les deux instances de l’application web que vous allez déployer dans deux régions Azure distinctes.

Dans l’exemple suivant, remplacez **<appspname_eastus>** et **<appspname_westeurope>** par un nom de plan App Service unique.

```azurecli-interactive

az appservice plan create \
    --name <appspname_eastus> \
    --resource-group myResourceGroup \
    --location eastus \
    --sku S1

az appservice plan create \
    --name <appspname_westeurope> \
    --resource-group myResourceGroup \
    --location westeurope \
    --sku S1

```

### <a name="create-a-web-app-in-the-app-service-plan"></a>Créer une application web dans le plan App Service
Créez deux instances de l’application web en utilisant la commande [az webapp create](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) dans les plans App Service dans les régions Azure *USA Est* et *Europe Ouest*.

Dans l’exemple suivant, remplacez **<app1name_eastus>** et **<app2name_westeurope>** par un nom d’application unique, puis remplacez **<appspname_eastus>** et **<appspname_westeurope>** par le nom utilisé pour créer les plans App Service dans la section précédente.

```azurecli-interactive

az webapp create \
    --name <app1name_eastus> \
    --plan <appspname_eastus> \
    --resource-group myResourceGroup

az webapp create \
    --name <app2name_westeurope> \
    --plan <appspname_westeurope> \
    --resource-group myResourceGroup

```

## <a name="add-traffic-manager-endpoints"></a>Ajouter des points de terminaison Traffic Manager
Ajoutez les deux applications web en tant que points de terminaison Traffic Manager en utilisant la commande [az network traffic-manager endpoint create](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-create) dans le profil Traffic Manager comme suit :

- Déterminez l’ID d’application web et ajoutez l’application web située dans la région Azure *USA Est* comme point de terminaison principal pour le routage de tout le trafic utilisateur. 
- Déterminez l’ID d’application web et ajoutez l’application web située dans la région Azure *Europe Ouest* comme point de terminaison de basculement. 

Quand le point de terminaison principal n’est pas disponible, le trafic est automatiquement routé vers le point de terminaison de basculement.

Dans l’exemple suivant, remplacez **<app1name_eastus>** et **<app2name_westeurope>** par les noms d’application créés pour chaque région dans la section précédente. Remplacez ensuite **<profile_name>** par le nom du profil utilisé dans la section précédente. 

**Point de terminaison USA Est**

```azurecli-interactive

az webapp show \
    --name <app1name_eastus> \
    --resource-group myResourceGroup \
    --query id

```

Prenez note de l’ID affiché dans la sortie et utilisez la commande suivante pour ajouter le point de terminaison :

```azurecli-interactive

az network traffic-manager endpoint create \
    --name <app1name_eastus> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 1 \
    --endpoint-status Enabled
```

**Point de terminaison Europe Ouest**

```azurecli-interactive

az webapp show \
    --name <app2name_westeurope> \
    --resource-group myResourceGroup \
    --query id

```

Prenez note de l’ID affiché dans la sortie et utilisez la commande suivante pour ajouter le point de terminaison :

```azurecli-interactive

az network traffic-manager endpoint create \
    --name <app1name_westeurope> \
    --resource-group myResourceGroup \
    --profile-name <profile_name> \
    --type azureEndpoints \
    --target-resource-id <ID from az webapp show> \
    --priority 2 \
    --endpoint-status Enabled

```

## <a name="test-your-traffic-manager-profile"></a>Tester votez profil Traffic Manager

Dans cette section, vous allez vérifier le nom de domaine de votre profil Traffic Manager. Vous allez aussi configurer le point de terminaison principal pour le rendre indisponible. Enfin, vous allez pouvoir constater que l’application est toujours disponible. Cela est dû au fait que Traffic Manager envoie le trafic au point de terminaison de basculement.

Dans l’exemple suivant, remplacez **<app1name_eastus>** et **<app2name_westeurope>** par les noms d’application créés pour chaque région dans la section précédente. Remplacez ensuite **<profile_name>** par le nom du profil utilisé dans la section précédente.

### <a name="determine-the-dns-name"></a>Déterminer le nom DNS

Déterminez le nom DNS du profil Traffic Manager en utilisant la commande [az network traffic-manager profile show](https://docs.microsoft.com/cli/azure/network/traffic-manager/profile?view=azure-cli-latest#az-network-traffic-manager-profile-show).

```azurecli-interactive

az network traffic-manager profile show \
    --name <profile_name> \
    --resource-group myResourceGroup \
    --query dnsConfig.fqdn

```

Copiez la valeur de **RelativeDnsName**. Le nom DNS de votre profil Traffic Manager est *http://<* nomdnsrelatif *>.trafficmanager.net*. 

### <a name="view-traffic-manager-in-action"></a>Afficher Traffic Manager en action
1. Dans un navigateur web, entrez le nom DNS de votre profil Traffic Manager (*http://<* nomdnsrelatif *>.trafficmanager.net*) pour afficher le site web par défaut de votre application web.

    > [!NOTE]
    > Dans ce scénario de démarrage rapide, toutes les demandes sont routées vers le point de terminaison principal. Il est défini sur **Priorité 1**.
2. Pour voir le basculement de Traffic Manager en action, désactivez votre site principal en utilisant la commande [az network traffic-manager endpoint update](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update).

   ```azurecli-interactive

    az network traffic-manager endpoint update \
        --name <app1name_eastus> \
        --resource-group myResourceGroup \
        --profile-name <profile_name> \
        --type azureEndpoints \
        --endpoint-status Disabled
    
   ```

3. Copiez le nom DNS de votre profil Traffic Manager (*http://<* nomdnsrelatif *>.trafficmanager.net*) pour afficher le site web dans une nouvelle session de navigateur web.
4. Vérifiez que l’application web est toujours disponible.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé, supprimez les groupes de ressources, les applications web et toutes les ressources associées en utilisant la commande [az group delete](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-delete).

```azurecli-interactive

az group delete \
    --resource-group myResourceGroup

```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé un profil Traffic Manager qui assure une haute disponibilité pour votre application web. Pour plus d’informations sur le routage du trafic, passez aux tutoriels Traffic Manager.

> [!div class="nextstepaction"]
> [Didacticiels Traffic Manager](tutorial-traffic-manager-improve-website-response.md)
