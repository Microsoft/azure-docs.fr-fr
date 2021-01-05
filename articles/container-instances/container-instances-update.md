---
title: Mettre à jour un groupe de conteneurs
description: Découvrez comment mettre à jour les conteneurs en cours d’exécution dans vos groupes de conteneurs Azure Container Instances.
ms.topic: article
ms.date: 04/17/2020
ms.openlocfilehash: cfc27de8caae98dd1c3065b5ed06433c4baaa5d2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82928718"
---
# <a name="update-containers-in-azure-container-instances"></a>Mettre à jour les conteneurs dans Azure Container Instances

Lors d’une utilisation normale de vos instances de conteneurs, vous pouvez avoir besoin de mettre à jour les conteneurs en cours d’exécution d’un [groupe de conteneurs](./container-instances-container-groups.md). Par exemple, vous souhaiterez peut-être mettre à jour une propriété telle qu’une version d’image, un nom DNS ou une variable d’environnement, ou actualiser une propriété dans un conteneur dont l’application s’est plantée.

Pour mettre à jour les conteneurs d’un groupe de conteneurs en cours d’exécution, redéployez un groupe existant en ayant modifié au moins une propriété. Lorsque vous mettez à jour un groupe de conteneurs, tous les conteneurs en cours d’exécution sont redémarrés sur place, en général sur le même hôte conteneur sous-jacent.

> [!NOTE]
> Les groupes de conteneurs terminés ou supprimés ne peuvent pas être mis à jour. Une fois qu’un groupe de conteneurs s’est arrêté (dans un état de réussite ou d’échec) ou a été supprimé, le groupe doit être déployé en tant que nouveau groupe. Consultez les autres [limitations](#limitations).

## <a name="update-a-container-group"></a>Mettre à jour un groupe de conteneurs

Pour mettre à jour un groupe de conteneurs existant

* Émettez la commande create (ou utilisez le portail Azure) et spécifiez le nom d’un groupe existant. 
* Modifiez ou ajoutez au moins une propriété du groupe qui prend en charge la mise à jour quand vous redéployez. Certaines propriétés [ne prennent pas en charge les mises à jour](#properties-that-require-container-delete).
* Définissez d’autres propriétés avec les valeurs que vous avez fournies précédemment. Si vous ne définissez pas de valeur pour une propriété, elle revient à sa valeur par défaut.

> [!TIP]
> Un [fichier YAML](./container-instances-container-groups.md#deployment) permet de conserver la configuration de déploiement d’un groupe de conteneurs et fournit un point de départ pour déployer un groupe mis à jour. Si vous avez appliqué une autre méthode pour créer le groupe, vous pouvez exporter la configuration vers YAML à l’aide d’[az container export][az-container-export]. 

### <a name="example"></a> Exemple

L’exemple Azure CLI suivant met à jour un groupe de conteneurs avec une nouvelle étiquette de nom DNS. Étant donné que la propriété d’étiquette de nom DNS du groupe est celui qui peut être mis à jour, le groupe de conteneurs est redéployé et ses conteneurs sont redémarrés.

Déploiement initial avec l’étiquette de nom DNS *myapplication-staging* :

```azurecli-interactive
# Create container group
az container create --resource-group myResourceGroup --name mycontainer \
    --image nginx:alpine --dns-name-label myapplication-staging
```

Mettez à jour le groupe de conteneurs avec une nouvelle étiquette de nom DNS, *myapplication*, et définissez les propriétés restantes avec les valeurs utilisées précédemment :

```azurecli-interactive
# Update DNS name label (restarts container), leave other properties unchanged
az container create --resource-group myResourceGroup --name mycontainer \
    --image nginx:alpine --dns-name-label myapplication
```

## <a name="update-benefits"></a>Avantages de la mise à jour

Lorsque vous mettez à jour un groupe de conteneurs existant, le déploiement est plus rapide. Lorsque vous redéployez un groupe de conteneurs existant, ses calques d’image de conteneur sont tirés (pull) de ceux mis en cache lors du déploiement précédent. Au lieu de tirer tous les calques d’image récents provenant du Registre, comme c’est le cas avec les nouveaux déploiements, seuls les calques modifiés (s’il en existe) sont tirés.

Lorsque vous mettez à jour un groupe existant au lieu d’en déployer un nouveau, les applications basées sur des images conteneur volumineuses (par exemple, Windows Server Core) peuvent être déployées beaucoup plus rapidement.

## <a name="limitations"></a>Limites

* Certaines propriétés de groupe de conteneurs ne peuvent pas être mises à jour. Pour modifier certaines propriétés d’un groupe de conteneurs, vous devez d’abord supprimer, puis redéployer le groupe. Consultez [Propriétés qui nécessitent la suppression du conteneur](#properties-that-require-container-delete).
* Tous les conteneurs d’un groupe de conteneurs sont redémarrés lorsque vous mettez à jour ce groupe. Vous ne pouvez pas effectuer une mise à jour ou un redémarrage sur place pour un seul conteneur du groupe.
* L’adresse IP d’un groupe de conteneurs est généralement conservée entre les mises à jour, mais il n’est pas garanti qu’elle reste la même. Tant que le groupe de conteneurs est déployé sur le même hôte sous-jacent, le groupe de conteneurs conserve son adresse IP. Bien que rares, certains événements internes Azure peuvent entraîner un redéploiement sur un hôte différent. Pour atténuer ce problème, nous vous recommandons d’utiliser une étiquette de nom DNS pour vos instances de conteneur.
* Les groupes de conteneurs terminés ou supprimés ne peuvent pas être mis à jour. Une fois qu’un groupe de conteneurs est arrêté (à l’état *Arrêté*) ou supprimé, le groupe est déployé en tant que nouveau groupe.

## <a name="properties-that-require-container-delete"></a>Propriétés qui nécessitent la suppression du conteneur

Certaines propriétés de groupe de conteneurs ne peuvent pas être mises à jour. Par exemple, pour modifier la stratégie de redémarrage d’un conteneur, vous devez d’abord supprimer le groupe de conteneurs, puis le recréer.

Les modifications apportées à ces propriétés nécessitent la suppression du groupe de conteneurs avant le redéploiement :

* Type de système d’exploitation
* Ressources processeur, mémoire ou GPU
* Stratégie de redémarrage
* Profil réseau

Lorsque vous supprimez, puis recréez un groupe de conteneurs, celui-ci n’est pas redéployé, mais recréé, comme un nouveau groupe. Tous les calques d’image sont fraîchement tirés (pull) du Registre. Il ne s’agit pas des calques mis en cache par un déploiement précédent. L’adresse IP du conteneur peut également changer si celui-ci est déployé sur un autre hôte sous-jacent.

## <a name="next-steps"></a>Étapes suivantes

Le **groupe de conteneurs** est mentionné plusieurs fois dans cet article. Chaque conteneur d’Azure Container Instances est déployé dans un groupe de conteneurs, et chaque groupe de conteneurs peut contenir plusieurs conteneurs.

[Groupes de conteneurs dans Azure Container Instances](./container-instances-container-groups.md)

[Déployer un groupe multiconteneur](container-instances-multi-container-group.md)

[Arrêter ou démarrer manuellement des conteneurs dans Azure Container Instances](container-instances-stop-start.md)

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container?view=azure-cli-latest#az-container-create
[azure-cli-install]: /cli/azure/install-azure-cli
[az-container-export]: /cli/azure/container#az-container-export
