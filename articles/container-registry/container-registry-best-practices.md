---
title: Bonnes pratiques concernant les registres
description: Découvrez comment utiliser votre instance Azure Container Registry de manière efficace en suivant ces meilleures pratiques.
ms.topic: article
ms.date: 09/27/2018
ms.openlocfilehash: fc84fb8cb98f58e28570095370d55a7358ce3a99
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83682689"
---
# <a name="best-practices-for-azure-container-registry"></a>Meilleures pratiques pour Azure Container Registry

En suivant ces meilleures pratiques, vous pouvez optimiser les performances et la rentabilité d’utilisation de votre registre Docker privé dans Azure.

Consultez également les [Suggestions pour la création de balises et de versions pour les images de conteneurs](container-registry-image-tag-version.md) pour des stratégies de création de balises et de versions pour les images dans votre registre. 

## <a name="network-close-deployment"></a>Déploiement proche du réseau

Créez votre registre de conteneurs dans la même région Azure que celle où vous déployez des conteneurs. En plaçant votre registre dans une région dont le réseau est proche des hôtes de vos conteneurs, vous pouvez en réduire la latence et le coût.

Un déploiement proche du réseau est l’une des principales raisons qui justifient l’utilisation d’un registre de conteneurs privé. Les images Docker ont une [construction en couches](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/) efficace qui permet un déploiement incrémentiel. Les nouveaux nœuds doivent cependant extraire toutes les couches requises pour une image donnée. Ce `docker pull` initial peut rapidement atteindre plusieurs gigaoctets. Le fait d’avoir un registre privé proche de votre déploiement réduit la latence du réseau.
En outre, tous les clouds publics, y compris Azure, appliquent des frais de sortie du réseau. L’extraction d’images d’un centre de données à un autre augmente non seulement la latence, mais également les frais de sortie du réseau.

## <a name="geo-replicate-multi-region-deployments"></a>Géo-réplication de déploiements dans plusieurs régions

Utilisez la fonction de [géo-réplication](container-registry-geo-replication.md) d’Azure Container Registry si vous déployez des conteneurs dans plusieurs régions. Si vous intervenez auprès de clients du monde entier à partir de centres de données locaux ou si votre équipe de développement se trouve sur différents sites, vous pouvez géo-répliquer votre registre pour en simplifier la gestion et réduire la latence. La géoréplication est disponible uniquement avec les registres [Premium](container-registry-skus.md).

Pour savoir comment utiliser la géo-réplication, consultez le didacticiel en trois parties relatif à la [géoréplication dans Azure Container Registry](container-registry-tutorial-prepare-registry.md).

## <a name="repository-namespaces"></a>Espaces de noms du référentiel

Grâce aux espaces de noms de référentiel, vous pouvez autoriser le partage d’un même registre entre plusieurs groupes au sein de votre organisation. Les registres peuvent être partagés entre plusieurs déploiements et équipes. Azure Container Registry prend en charge les espaces de noms imbriqués de manière à faciliter l’isolement de groupes.

Prenons par exemple les balises d’image de conteneur suivantes : Les images qui sont utilisées dans l’ensemble de l’entreprise, telles que `aspnetcore`, sont placées dans l’espace de noms racine, tandis que les images de conteneur détenues par les groupes Produits et Marketing utilisent chacune leurs propres espaces de noms.

- *contoso.azurecr.io/aspnetcore:2.0*
- *contoso.azurecr.io/products/widget/web:1*
- *contoso.azurecr.io/products/bettermousetrap/refundapi:12.3*
- *contoso.azurecr.io/marketing/2017-fall/concertpromotions/campaign:218.42*

## <a name="dedicated-resource-group"></a>Groupe de ressources dédié

Étant donné que les registres de conteneurs sont des ressources utilisées sur plusieurs hôtes de conteneur, il est préférable qu'un registre réside dans son propre groupe de ressources.

Bien qu’il soit possible de tester un type d’hôte spécifique, par exemple Azure Container Instances, vous souhaiterez probablement supprimer l’instance de conteneur une fois l’opération terminée. Mais vous trouverez peut-être également judicieux de conserver la collection d’images que vous avez transmise à Azure Container Registry. En plaçant votre registre dans son propre groupe de ressources, vous réduisez le risque de supprimer accidentellement la collection d’images dans le registre lorsque vous supprimez le groupe de ressources de l’instance de conteneur.

## <a name="authentication"></a>Authentification

Lorsque vous vous authentifiez avec Azure Container Registry, vous pouvez vous trouver dans deux cas de figure : une authentification individuelle et une authentification de service (ou « sans affichage »). Le tableau suivant fournit une brève vue d’ensemble de ces scénarios et décrit la méthode d’authentification recommandée pour chacun.

| Type | Exemple de scénario | Méthode recommandée |
|---|---|---|
| Identité individuelle | Un développeur qui extrait des images ou transmet des images à partir de son ordinateur de développement. | [az acr login](/cli/azure/acr?view=azure-cli-latest#az-acr-login) |
| Identité de service / sans affichage | Pipelines de génération et de déploiement dans lequel l’utilisateur n’intervient pas directement. | [Principal du service](container-registry-authentication.md#service-principal) |

Pour obtenir des informations détaillées sur l’authentification Azure Container Registry, consultez [S’authentifier avec un registre de conteneurs Azure](container-registry-authentication.md).

## <a name="manage-registry-size"></a>Gérer la taille du registre

Les contraintes de stockage de chaque [niveau de service du registre de conteneurs][container-registry-skus] s’appliquent à un scénario spécifique : **De base** pour la mise en route, **Standard** pour la majorité des applications de production et **Premium** pour les performances à très grande échelle et la [géoréplication][container-registry-geo-replication]. Pendant toute la durée de vie de votre registre, vous devez gérer sa taille en supprimant régulièrement le contenu inutilisé.

Utilisez la commande Azure CLI [az acr show-usage][az-acr-show-usage] pour afficher la taille actuelle de votre registre :

```azurecli
az acr show-usage --resource-group myResourceGroup --name myregistry --output table
```

```output
NAME      LIMIT         CURRENT VALUE    UNIT
--------  ------------  ---------------  ------
Size      536870912000  185444288        Bytes
Webhooks  100                            Count
```

Vous pouvez également retrouver le stockage actuel utilisé dans la **Vue d’ensemble** de votre registre dans le Portail Azure :

![Informations sur l’utilisation du registre dans le portail Azure][registry-overview-quotas]

### <a name="delete-image-data"></a>Supprimer les données d’image

Azure Container Registry prend en charge plusieurs méthodes permettant de supprimer des données image de votre registre de conteneurs. Vous pouvez supprimer des images par balise ou synthèse de manifeste, ou encore supprimer un référentiel dans son intégralité.

Pour en savoir plus sur la suppression des données image de votre registre, notamment des images sans balises (parfois appelées « non résolues » ou « orphelines ») des images, consultez la section [Supprimer des images conteneur dans Azure Container Registry](container-registry-delete.md).

## <a name="next-steps"></a>Étapes suivantes

Azure Container Registry est disponible en plusieurs niveaux, (également appelés références SKU), qui fournissent chacun des fonctionnalités différentes. Pour plus d’informations sur les niveaux de service disponibles, consultez [Niveaux de service Azure Container Registry](container-registry-skus.md).

<!-- IMAGES -->
[delete-repository-portal]: ./media/container-registry-best-practices/delete-repository-portal.png
[registry-overview-quotas]: ./media/container-registry-best-practices/registry-overview-quotas.png

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-acr-show-usage]: /cli/azure/acr#az-acr-show-usage
[azure-cli]: /cli/azure
[azure-portal]: https://portal.azure.com
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-skus]: container-registry-skus.md
