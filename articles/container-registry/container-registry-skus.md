---
title: Niveaux de service et fonctionnalités du registre
description: Découvrez les fonctionnalités et les limites des niveaux de service (SKU) De base, Standard et Premium d’Azure Container Registry.
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: e2a5ad52775e9000aa0beb0a926d809da1c5a0e0
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92048472"
---
# <a name="azure-container-registry-service-tiers"></a>Niveaux de service Azure Container Registry

Azure Container Registry est disponible dans plusieurs niveaux de service (également appelés références SKU). Ces niveaux offrent une tarification prévisible et plusieurs options pour l’alignement en fonction de la capacité et des modèles d’utilisation de votre registre Docker privé dans Azure.

| Niveau | Description |
| --- | ----------- |
| **De base** | Point d’entrée au coût optimisé pour les développeurs apprenant Azure Container Registry. Les registres De base ont les mêmes fonctionnalités de programmation que les registres Standard et Premium (telle que l’[intégration de l’authentification](container-registry-authentication.md#individual-login-with-azure-ad) Azure Active Directory, la [suppression d’image][container-registry-delete] et les [webhooks][container-registry-webhook]). Toutefois, le stockage inclus et le débit d’image sont parfaitement appropriés pour des scénarios d’utilisation inférieure. |
| **Standard** | Les registres Standard offrent les mêmes fonctionnalités que la version De base, avec un stockage inclus et un débit d’image accrus. Les registres Standard devraient satisfaire les besoins de la plupart des scénarios de production. |
| **Premium** | Les registres Premium fournissent la quantité la plus élevée de stockage inclus et d’opérations simultanées, permettant des scénarios impliquant des volumes importants. En plus d’un débit d’image plus élevé, le niveau Premium ajoute des fonctionnalités comme la [géoréplication][container-registry-geo-replication] pour gérer un seul registre dans plusieurs régions, [l’approbation de contenu](container-registry-content-trust.md) pour la signature d’étiquettes d’image et la [liaison privée avec points de terminaison privés](container-registry-private-link.md) pour restreindre l’accès au registre. |

Les niveaux De base, Standard et Premium fournissent toutes les mêmes fonctionnalités de programmation. Elles bénéficient également d’un [stockage des images][container-registry-storage] complètement managé par Azure. Choisir un niveau supérieur permet d’obtenir de meilleures performances et une meilleure mise à l’échelle. Avec plusieurs niveaux de service, vous pouvez commencer avec un niveau De base, puis passer aux niveaux Standard et Premium au fur et à mesure que votre utilisation du registre augmente.

## <a name="service-tier-features-and-limits"></a>Fonctionnalités et limites de niveau de service

Le tableau suivant détaille les fonctionnalités et les limites de registre des niveaux de service De base, Standard et Premium.

[!INCLUDE [container-instances-limits](../../includes/container-registry-limits.md)]

## <a name="changing-tiers"></a>Changer de niveau

Vous pouvez changer le niveau de service d’un registre via l’interface de ligne de commande Azure ou dans le Portail Azure. Vous pouvez changer librement de niveau, tant que le nouveau niveau possède la capacité de stockage maximale requise. 

### <a name="azure-cli"></a>Azure CLI

Pour changer de niveau de service dans Azure CLI, utilisez la commande [az acr update][az-acr-update]. Par exemple, pour basculer vers Premium :

```azurecli
az acr update --name myregistry --sku Premium
```

### <a name="azure-portal"></a>Portail Azure

Dans la **vue d’ensemble** du registre de conteneur dans le portail Azure, sélectionnez **Mettre à jour**, puis choisissez une nouvelle **référence SKU** dans la liste déroulante.

![Mettre à jour la référence de registre de conteneurs dans le portail Azure][update-registry-sku]

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur la tarification de chacun des niveaux de service Azure Container Registry, voir [Tarification Container Registry][container-registry-pricing].

Pour plus d’informations sur la tarification des transferts de données, consultez [Détails sur la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/). 

## <a name="next-steps"></a>Étapes suivantes

**Feuille de route Azure Container Registry**

Consultez la [Feuille de route ACR][acr-roadmap] sur GitHub pour obtenir des informations sur les fonctionnalités à venir dans le service.

**Azure Container Registry UserVoice**

Proposez et votez pour des suggestions de fonctionnalités dans [ACR UserVoice][container-registry-uservoice].

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/forums/903958-azure-container-registry

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md