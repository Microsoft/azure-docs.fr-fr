---
title: Déployer un groupe de conteneurs sur un réseau virtuel Azure
description: Découvrez comment déployer un groupe de conteneurs sur un réseau virtuel Azure nouveau ou existant en utilisant l’interface de ligne de commande Azure.
ms.topic: article
ms.date: 07/02/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 02cf514e6c19387e3a9e2f1c78b65f346fff764e
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746894"
---
# <a name="deploy-container-instances-into-an-azure-virtual-network"></a>Déployer des instance de conteneur dans un réseau virtuel Azure

Un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) fournit un accès réseau privé et sécurisé à vos ressources Azure et locales. En déployant des groupes de conteneurs dans un réseau virtuel Azure, vos conteneurs peuvent communiquer en toute sécurité avec d’autres ressources dans le réseau virtuel.

Cet article explique comment utiliser la commande [az container create][az-container-create] dans Azure CLI pour déployer des groupes de conteneurs sur un réseau virtuel nouveau ou existant. 

Pour les scénarios et les limitations de mise en réseau, consultez [Ressources et scénarios relatifs aux réseaux virtuels pour Azure Container Instances](container-instances-virtual-network-concepts.md).

> [!IMPORTANT]
> Le déploiement d’un groupe de conteneurs sur un réseau virtuel est généralement disponible pour les conteneurs Linux, dans la plupart des régions où Azure Container Instances est disponible. Pour plus d’informations, consultez [Régions et disponibilité des ressources](container-instances-virtual-network-concepts.md#where-to-deploy). 

Les exemples présentés dans cet article sont mis en forme pour l’interpréteur de commandes Bash. Si vous préférez un autre interpréteur de commandes comme PowerShell ou l’invite de commande, modifiez les caractères de continuation de ligne en conséquence.


## <a name="deploy-to-new-virtual-network"></a>Déployer dans un nouveau réseau virtuel

Pour déployer dans un nouveau réseau virtuel et laisser Azure créer automatiquement les ressources réseau, spécifiez les éléments suivants lorsque vous exécutez [as container create][az-container-create] :

* Nom du réseau virtuel
* Préfixe d’adresse de réseau virtuel au format CIDR
* Nom du sous-réseau
* Préfixe d’adresse de sous-réseau au format CIDR

Le réseau virtuel et les préfixes d’adresse de sous-réseau spécifient les espaces d’adressage du réseau virtuel et du sous-réseau, respectivement. Ces valeurs sont représentées en notation CIDR (Classless Inter-domain Routing), par exemple `10.0.0.0/16`. Pour plus d’informations sur l’utilisation des sous-réseaux, consultez [Ajouter, modifier ou supprimer un sous-réseau de réseau virtuel](../virtual-network/virtual-network-manage-subnet.md).

Une fois que vous avez déployé votre premier groupe de conteneurs avec cette méthode, vous pouvez déployer dans le même sous-réseau en spécifiant les noms du réseau virtuel et du sous-réseau, ou le profil réseau créé automatiquement par Azure pour vous. Comme Azure délègue le sous-réseau à Azure Container Instances, vous ne pouvez déployer *que* des groupes de conteneurs dans le sous-réseau.

### <a name="example"></a> Exemple

La commande [az container create][az-container-create] suivante spécifie les paramètres pour un nouveau réseau virtuel et un nouveau sous-réseau. Fournissez le nom d’un groupe de ressources créé dans une région où des déploiements de groupe de conteneurs dans un réseau virtuel sont [disponibles](container-instances-region-availability.md). Cette commande déploie le conteneur Microsoft [aci-helloworld][aci-helloworld] public qui exécute un petit serveur web Node.js qui gère une page web statique. Dans la section suivante, vous allez déployer un deuxième groupe de conteneurs dans le même sous-réseau et tester la communication entre les deux instances de conteneur.

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

Lorsque vous déployez dans un nouveau réseau virtuel avec cette méthode, l’opération peut prendre quelques minutes, le temps de créer les ressources réseau. Après le déploiement initial, les déploiements de groupe de conteneurs sur le même sous-réseau sont accomplis plus rapidement.

## <a name="deploy-to-existing-virtual-network"></a>Déployer dans un réseau virtuel existant

Pour déployer un groupe de conteneurs dans un réseau virtuel existant :

1. Créez un sous-réseau au sein de votre réseau virtuel existant, utilisez un sous-réseau existant dans lequel un groupe de conteneurs est déjà déployé, ou utilisez un sous-réseau existant vide de *toute* autre ressource.
1. Déployez un groupe de conteneurs avec [az container create][az-container-create] et spécifiez un des éléments suivants :
   * Nom du réseau virtuel et nom du sous-réseau
   * ID de ressource de réseau virtuel et ID de ressource de sous-réseau, ce qui permet d’utiliser un réseau virtuel d’un groupe de ressources différent
   * Nom ou ID du profil réseau que vous pouvez obtenir en utilisant [az network profile list][az-network-profile-list]

### <a name="example"></a> Exemple

L’exemple suivant déploie un deuxième groupe de conteneurs sur le sous-réseau créé précédemment et vérifie la communication entre les deux instances de conteneur.

Tout d’abord, obtenez l’adresse IP du premier groupe de conteneurs déployé, *appcontainer* :

```azurecli
az container show --resource-group myResourceGroup \
  --name appcontainer \
  --query ipAddress.ip --output tsv
```

La sortie affiche l’adresse IP du groupe de conteneurs dans le sous-réseau privé. Par exemple :

```console
10.0.0.4
```

Maintenant, attribuez à `CONTAINER_GROUP_IP` l’adresse IP que vous avez récupérée avec la commande `az container show` et exécutez la commande suivante `az container create`. Ce deuxième conteneur, *commchecker* , utilise une image Linux Alpine et exécute `wget` sur l’adresse IP du sous-réseau privé du premier groupe de conteneurs.

```azurecli
CONTAINER_GROUP_IP=<container-group-IP-address>

az container create \
  --resource-group myResourceGroup \
  --name commchecker \
  --image alpine:3.5 \
  --command-line "wget $CONTAINER_GROUP_IP" \
  --restart-policy never \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Après le déploiement de ce deuxième conteneur, consultez dans ses journaux d’activité la sortie de la commande `wget` qu’il a exécutée :

```azurecli
az container logs --resource-group myResourceGroup --name commchecker
```

Si le deuxième conteneur a correctement communiqué avec le premier, la sortie doit être similaire à ceci :

```console
Connecting to 10.0.0.4 (10.0.0.4:80)
index.html           100% |*******************************|  1663   0:00:00 ETA
```

La sortie du journal doit montrer que `wget` a réussie à se connecter le fichier d’index et à le télécharger à partir du premier conteneur à l’aide de son adresse IP privée dans le sous-réseau local. Le trafic réseau entre les deux groupes de conteneurs est resté dans le réseau virtuel.

### <a name="example---yaml"></a>Exemple – YAML

Vous pouvez également déployer un groupe de conteneurs sur un réseau virtuel existant en utilisant un fichier YAML, un [modèle Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
) ou une autre méthode de programmation, telle que le Kit de développement logiciel (SDK) Python. 

Par exemple, lors de l’utilisation d’un fichier YAML, vous pouvez effectuer un déploiement sur un réseau virtuel avec un sous-réseau délégué à Azure Container Instances. Spécifiez les propriétés suivantes :

* `ipAddress`: paramètres d’adresse IP privée pour le groupe de conteneurs.
  * `ports`: ports à ouvrir, le cas échéant.
  * `protocol`: protocole (TCP ou UDP) du port ouvert.
* `networkProfile`: paramètres réseau pour le réseau virtuel et le sous-réseau.
  * `id`: ID complet de ressource Resource Manager du `networkProfile`.

Pour obtenir l’ID du profil réseau, exécutez la commande [az network profile list][az-network-profile-list], en spécifiant le nom du groupe de ressources contenant votre réseau virtuel et le sous-réseau délégué.

``` azurecli
az network profile list --resource-group myResourceGroup \
  --query [0].id --output tsv
```

Exemple de sortie :

```console
/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-aci-subnet
```

Une fois que vous avez l’ID du profil réseau, copiez le fichier YAML suivant dans un nouveau fichier nommé *vnet-deploy-aci.yaml* . Sous `networkProfile`, remplacez la valeur `id` par l’ID que vous venez de récupérer, puis enregistrez le fichier. Ce fichier YAML crée un groupe de conteneurs nommé *appcontaineryaml* dans votre réseau virtuel.

```YAML
apiVersion: '2019-12-01'
location: westus
name: appcontaineryaml
properties:
  containers:
  - name: appcontaineryaml
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  ipAddress:
    type: Private
    ports:
    - protocol: tcp
      port: '80'
  networkProfile:
    id: /subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-subnet
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Déployez le groupe de conteneurs avec la commande [az container create][az-container-create], en spécifiant le nom du fichier YAML dans le paramètre `--file` :

```azurecli
az container create --resource-group myResourceGroup \
  --file vnet-deploy-aci.yaml
```

Une fois le déploiement effectué, exécutez la commande [az container show][az-container-show] pour afficher son état. Exemple de sortie :

```console
Name              ResourceGroup    Status    Image                                       IP:ports     Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  ------------------------------------------  -----------  ---------  ---------------  --------  ----------
appcontaineryaml  myResourceGroup  Running   mcr.microsoft.com/azuredocs/aci-helloworld  10.0.0.5:80  Private    1.0 core/1.5 gb  Linux     westus
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

### <a name="delete-container-instances"></a>Supprimer des instances de conteneur

Lorsque vous avez fini d’utiliser les instances de conteneur que vous avez créées, supprimez-les avec les commandes suivantes :

```azurecli
az container delete --resource-group myResourceGroup --name appcontainer -y
az container delete --resource-group myResourceGroup --name commchecker -y
az container delete --resource-group myResourceGroup --name appcontaineryaml -y
```

### <a name="delete-network-resources"></a>Supprimer des ressources réseau

Cette fonctionnalité nécessite pour le moment plusieurs commandes supplémentaires pour supprimer les ressources réseau que vous avez créées précédemment. Si vous avez utilisé les exemples de commande dans les sections précédentes de cet article pour créer votre réseau virtuel et votre sous-réseau, vous pouvez utiliser le script suivant pour supprimer ces ressources réseau. Le script suppose que votre groupe de ressources contient un seul réseau virtuel avec un seul profil réseau.

Avant d’exécuter le script, attribuez à la variable `RES_GROUP` le nom du groupe de ressources contenant le réseau virtuel et le sous-réseau à supprimer. Mettez à jour le nom du réseau virtuel si vous n’avez pas utilisé le nom `aci-vnet` suggéré précédemment. Le script est mis en forme pour l’interpréteur de commandes Bash. Si vous préférez un autre interpréteur de commandes, PowerShell ou l’invite de commande, vous devrez ajuster les variables et les accesseurs en conséquence.

> [!WARNING]
> Ce script supprime les ressources ! Le réseau virtuel et tous les sous-réseaux qu’il contient sont supprimés. Assurez-vous de n’avoir plus besoin *des* ressources du réseau virtuel, y compris des sous-réseaux qu’il contient, avant d’exécuter ce script. Une fois supprimées, **ces ressources sont irrécupérables** .

```azurecli
# Replace <my-resource-group> with the name of your resource group
# Assumes one virtual network in resource group
RES_GROUP=<my-resource-group>

# Get network profile ID
# Assumes one profile in virtual network
NETWORK_PROFILE_ID=$(az network profile list --resource-group $RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Delete virtual network
az network vnet delete --resource-group $RES_GROUP --name aci-vnet
```

## <a name="next-steps"></a>Étapes suivantes

Pour déployer un réseau virtuel, un sous-réseau, un profil réseau et un groupe de conteneurs nouveaux à l’aide d’un modèle Resource Manager, consultez [Créer un groupe de conteneurs Azure avec un réseau virtuel](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
).

<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-vnet/aci-vnet-01.png

<!-- LINKS - External -->
[aci-helloworld]: https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-profile-list]: /cli/azure/network/profile#az-network-profile-list
