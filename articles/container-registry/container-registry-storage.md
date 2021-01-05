---
title: Stockage des images conteneur
description: Plus d’informations sur comment vos images conteneur Docker sont stockées dans Azure Container Registry, y compris la sécurité, de redondance et la capacité.
ms.topic: article
ms.date: 06/18/2020
ms.openlocfilehash: d51014e9e0769091aba42682cce3a6a01cfa19de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85214058"
---
# <a name="container-image-storage-in-azure-container-registry"></a>Stockage des images conteneur dans Azure Container Registry

Chaque registre de conteneurs Azure [de base, standard et Premium](container-registry-skus.md) bénéficie de fonctionnalités de stockage Azure avancées, tels que le chiffrement au repos pour la sécurité des données image et la géo-redondance pour la protection des données image. Les sections suivantes décrivent les fonctionnalités et les limites du stockage des images dans Azure Container Registry (ACR).

## <a name="encryption-at-rest"></a>Chiffrement des données au repos

Toutes les images conteneur dans votre registre sont chiffrées au repos. Azure chiffre automatiquement une image avant de la stocker et la déchiffre à la volée lorsque vous ou vos applications et services extraient l’image. Vous pouvez également appliquer une couche de chiffrement supplémentaire avec une [clé managée par le client](container-registry-customer-managed-keys.md).

## <a name="geo-redundant-storage"></a>Stockage géo-redondant

Azure utilise un schéma de stockage géo-redondant pour vous prémunir contre la perte de vos images conteneur. Azure Container Registry réplique automatiquement vos images conteneur dans plusieurs centres de données géographiquement distants, empêchant leur perte en cas de défaillance du stockage régional.

## <a name="geo-replication"></a>Géoréplication

Pour les scénarios nécessitant une assurance de plus haute disponibilité, envisagez d’utiliser la fonctionnalité [géo-réplication](container-registry-geo-replication.md) des registres Premium. La géo-réplication permet de se prémunir contre la perte d’accès à votre registre en cas d’un échec régional *total* et pas seulement un échec de stockage. La géo-réplication offre également d’autres avantages, comme le stockage des images proches du réseau pour les envois et extraits les plus rapides dans des scénarios de développement ou de déploiement distribués.

## <a name="scalable-storage"></a>Stockage évolutif

Azure Container Registry vous permet de créer autant de référentiels, d’images, de couches ou de balises que nécessaire, jusqu’à la [limite de stockage du Registre](container-registry-skus.md#service-tier-features-and-limits). 

Un très grand nombre de balises et de référentiels peuvent affecter les performances de votre registre. Supprimez régulièrement les référentiels, balises et images non utilisés dans le cadre de la routine de maintenance de votre registre, et définissez éventuellement une [stratégie de rétention](container-registry-retention-policy.md) pour les manifestes sans balise. Les ressources de registre supprimées telles que les référentiels, les images et les balises ne *peuvent pas* être récupérées après suppression. Pour plus d’informations sur la suppression de ressources du Registre, voir [Supprimer des images conteneur dans Azure Container Registry](container-registry-delete.md).

## <a name="storage-cost"></a>Coût de stockage

Pour plus d’informations sur la tarification, consultez [Tarification d’Azure Container Registry][pricing].

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les registres de base, standard et Premium, consultez [Niveaux de service Azure Container Registry](container-registry-skus.md).

<!-- IMAGES -->

<!-- LINKS - External -->
[portal]: https://portal.azure.com
[pricing]: https://aka.ms/acr/pricing

<!-- LINKS - Internal -->
