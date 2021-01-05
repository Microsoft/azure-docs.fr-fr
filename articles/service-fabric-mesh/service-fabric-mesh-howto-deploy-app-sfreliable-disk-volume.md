---
title: Volume de disque fiable Service Fabric avec Service Fabric Mesh
description: Découvrez comment stocker un état dans une application Azure Service Fabric Mesh en montant un volume basé sur Service Fabric Reliable Disk dans le conteneur avec Azure CLI.
author: ashishnegi
ms.topic: conceptual
ms.date: 12/03/2018
ms.author: asnegi
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: 86822c5a9cef84ff4b51bc94b6b2dd3dbdee91bf
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97702007"
---
# <a name="mount-highly-available-service-fabric-reliable-disk-based-volume-in-a-service-fabric-mesh-application"></a>Monter un volume basé sur Service Fabric Reliable Disk dans une application Azure Service Fabric Mesh 
La méthode courante pour rendre persistant l’état avec des applications de conteneur consiste à utiliser un stockage à distance, tel que le Stockage Fichier Azure ou une base de données du type Azure Cosmos DB. Cela entraîne une importante latence de lecture et d’écriture sur le magasin distant.

Cet article montre comment stocker un état dans un volume Service Fabric Reliable Disk hautement disponible en montant un volume à l’intérieur du conteneur d’une application Service Fabric Mesh.
Service Fabric Reliable Disk fournit des volumes pour les lectures locales en répliquant les écritures dans le cluster Service Fabric pour garantir une haute disponibilité. Cela supprime les appels réseau pour les lectures et réduit la latence du réseau pour les écritures. Si le conteneur redémarre ou se déplace vers un autre nœud, la nouvelle instance de conteneur verra le même volume que l’ancien. Cela le rend donc efficace et hautement disponible.

Dans cet exemple, l’application de compteur dispose d’un service ASP.NET Core, avec une page web qui affiche la valeur du compteur dans un navigateur.

`counterService` lit périodiquement une valeur de compteur dans un fichier, l’incrémente, puis la réécrit dans le fichier. Le fichier est stocké dans un dossier monté sur le volume soutenu par Service Fabric Reliable Disk.

## <a name="prerequisites"></a>Prérequis

Pour accomplir cette tâche, vous pouvez utiliser le service Azure Cloud Shell ou une installation locale d’Azure CLI. Pour utiliser Azure CLI avec cet article, vérifiez que `az --version` retourne au moins `azure-cli (2.0.43)`.  Installez (ou mettez à jour) le module d’extension Service Fabric Mesh CLI en suivant les [instructions](service-fabric-mesh-howto-setup-cli.md) ci-après.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure et définissez votre abonnement.

```azurecli-interactive
az login
az account set --subscription "<subscriptionID>"
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources dans lequel déployer l’application. La commande suivante crée un groupe de ressources nommé `myResourceGroup` dans un emplacement situé dans l’Est des États-Unis. Si vous modifiez le nom du groupe de ressources dans la commande ci-dessous, pensez à le modifier dans toutes les commandes qui suivent.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="deploy-the-template"></a>Déployer le modèle

>[!NOTE]
> Depuis le 2 novembre 2020, des [limites de taux de téléchargement s’appliquent](https://docs.docker.com/docker-hub/download-rate-limit/) aux requêtes anonymes et authentifiées qui sont envoyées à Docker Hub à partir de comptes de plan Docker Gratuit. Ces limites sont appliquées par adresse IP. 
> 
> Ce modèle utilise des images publiques issues de Docker Hub. Notez que votre taux de téléchargement peut être limité. Pour plus d’informations, consultez [S’authentifier auprès de Docker Hub](../container-registry/buffer-gate-public-content.md#authenticate-with-docker-hub).

La commande suivante déploie une application Linux à l’aide du [modèle counter.sfreliablevolume.linux.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/counter/counter.sfreliablevolume.linux.json). Pour déployer une application Windows, utilisez le [modèle counter.sfreliablevolume.windows.json](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/counter/counter.sfreliablevolume.windows.json). N’oubliez pas que le déploiement d’images de conteneur plus grandes peut prendre plus de temps.

```azurecli-interactive
az mesh deployment create --resource-group myResourceGroup --template-uri https://raw.githubusercontent.com/Azure-Samples/service-fabric-mesh/master/templates/counter/counter.sfreliablevolume.linux.json
```

Vous pouvez également voir l’état du déploiement avec la commande suivante

```azurecli-interactive
az deployment group show --name counter.sfreliablevolume.linux --resource-group myResourceGroup
```

Notez le nom de la ressource de passerelle dont le type de ressources est `Microsoft.ServiceFabricMesh/gateways`. Il sera utilisé pour obtenir l’adresse IP publique de l’application.

## <a name="open-the-application"></a>Ouvrir l’application

Une fois l’application déployée, récupérez l’adresse IP de la ressource de passerelle correspondant à l’application. Utilisez le nom de passerelle que vous avez noté dans la section ci-dessus.
```azurecli-interactive
az mesh gateway show --resource-group myResourceGroup --name counterGateway
```

La sortie doit contenir une propriété `ipAddress`, qui correspond à l’adresse IP publique du point de terminaison du service. Ouvrez-la à partir d’un navigateur. Une page web doit s’afficher avec la valeur du compteur mise à jour toutes les secondes.

## <a name="verify-that-the-application-is-able-to-use-the-volume"></a>Vérifier que l’application est capable d’utiliser le volume

L’application crée un fichier nommé `counter.txt` dans le volume à l’intérieur du dossier `counter/counterService`. Le contenu de ce fichier est la valeur du compteur affiché sur la page web.

## <a name="delete-the-resources"></a>Supprimer les ressources

Supprimez régulièrement les ressources que vous n’utilisez plus dans Azure. Pour supprimer les ressources liées à cet exemple, supprimez le groupe de ressources dans lequel elles ont été déployées (ce qui supprime tous les éléments associés au groupe de ressources) avec la commande suivante :

```azurecli-interactive
az group delete --resource-group myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez l’exemple d’application Service Fabric Reliable Volume Disk sur [GitHub](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter).
- Pour en savoir plus sur le modèle de ressource Service Fabric, voir [Modèle de ressource Service Fabric mesh](service-fabric-mesh-service-fabric-resources.md).
- Pour en savoir plus sur Service Fabric mesh, voir la [vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md).