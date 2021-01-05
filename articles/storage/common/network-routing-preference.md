---
title: Configurer une préférence de routage réseau (préversion)
titleSuffix: Azure Storage
description: Configurez une préférence de routage réseau (préversion) pour votre compte de stockage Azure afin de spécifier comment le trafic réseau est acheminé vers votre compte à partir de clients sur Internet.
services: storage
author: santoshc
ms.service: storage
ms.topic: conceptual
ms.date: 05/12/2020
ms.author: santoshc
ms.reviewer: tamram
ms.subservice: common
ms.openlocfilehash: 5b4a1b1f27dff059090d78e24a6a0eca0bbbf01f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85514203"
---
# <a name="configure-network-routing-preference-for-azure-storage-preview"></a>Configurer une préférence de routage réseau pour le service Stockage Azure (préversion)

Vous pouvez configurer une [préférence de routage](../../virtual-network/routing-preference-overview.md) réseau (préversion) pour votre compte de stockage Azure afin de spécifier comment le trafic réseau est acheminé vers votre compte à partir de clients sur Internet. Par défaut, le trafic provenant d’Internet est routé vers le point de terminaison public de votre compte de stockage sur le [réseau Microsoft mondial](../../networking/microsoft-global-network.md). Le stockage Azure fournit des options supplémentaires pour configurer la façon dont le trafic est acheminé vers votre compte de stockage.

La configuration d’une préférence de routage vous offre la possibilité d’optimiser votre trafic pour obtenir des performances réseau d’exception ou un coût plus avantageux. Lorsque vous configurez une préférence de routage, vous spécifiez la manière dont le trafic est acheminé vers le point de terminaison public de votre compte de stockage par défaut. Vous pouvez également publier des points de terminaison spécifiques d’un itinéraire pour votre compte de stockage.

## <a name="microsoft-global-network-versus-internet-routing"></a>Réseau Microsoft mondial et routage Internet

Par défaut, les clients extérieurs à l’environnement Azure accèdent à votre compte de stockage via le réseau Microsoft mondial. Le réseau Microsoft mondial est optimisé pour sélectionner des itinéraires à faible latence afin d’offrir des performances réseau d’exception et une haute fiabilité. Le trafic, tant entrant que sortant, est routé via le point de présence le plus proche du client. Cette configuration de routage par défaut garantit que le trafic entrant et sortant de votre compte de stockage transite essentiellement via le réseau Microsoft mondial, ce qui optimise les performances réseau.

Vous pouvez modifier la configuration du routage pour votre compte de stockage afin que le trafic en provenance et à destination de clients situés en dehors de l’environnement Azure soit acheminé via le point de présence le plus proche du compte de stockage. Cet itinéraire réduit le parcours de votre trafic sur le réseau Microsoft mondial en le remettant au fournisseur de services Internet de transit dès que possible. Cette configuration de routage permet de réduire les coûts de mise en réseau.

Le diagramme suivant montre comment le trafic circule entre le client et le compte de stockage pour chaque préférence de routage :

![Vue d’ensemble des options de routage pour le service Stockage Azure](media/network-routing-preference/routing-options-diagram.png)

Pour plus d’informations sur la préférences de routage dans Azure, consultez [Qu’est-ce qu’une préférence de routage (préversion) ?](../../virtual-network/routing-preference-overview.md).

## <a name="routing-configuration"></a>Configuration de routage

Vous pouvez choisir entre le routage réseau Microsoft mondial et le routage Internet comme préférence de routage par défaut pour le point de terminaison public de votre compte de stockage. La préférence de routage par défaut s’applique à tout le trafic provenant de clients extérieurs à Azure, et affecte les points de terminaison pour Azure Data Lake Storage Gen2, Stockage Blob, Azure Files et les sites web statiques. La configuration de la préférence de routage n’est pas prise en charge pour les files d’attente ou les tables Azure.

Vous pouvez également publier des points de terminaison spécifiques d’un itinéraire pour votre compte de stockage. Lorsque vous publiez des points de terminaison spécifiques d’un itinéraire, le service Stockage Azure crée des points de terminaison publics pour votre compte de stockage, qui acheminent le trafic via l’itinéraire souhaité. Cette flexibilité vous permet de diriger le trafic vers votre compte de stockage sur un itinéraire spécifique sans modifier vos préférences de routage par défaut.

Par exemple, la publication d’un point de terminaison spécifique de l’itinéraire Internet pour « StorageAccountA » a pour effet de publier les points de terminaison suivants pour votre compte de stockage :

| Service de stockage        | Point de terminaison spécifique de l’itinéraire                                  |
| :--------------------- | :------------------------------------------------------- |
| Service d'objets blob           | `StorageAccountA-internetrouting.blob.core.windows.net`  |
| Data Lake Storage Gen2 | `StorageAccountA-internetrouting.dfs.core.windows.net`   |
| Service Fichier           | `StorageAccountA-internetrouting.file.core.windows.net`  |
| Sites web statiques        | `StorageAccountA-internetrouting.web.core.windows.net`   |

Si vous disposez d’un compte de stockage géoredondant avec accès en lecture (RA-GRS) ou d’un compte de stockage géoredondant interzone avec accès en lecture (RA-GZRS) (RA-GZRS), la publication de points de terminaison spécifiques de l’itinéraire a également pour effet de créer automatiquement les points de terminaison correspondants dans la région secondaire pour l’accès en lecture.

| Service de stockage        | Point de terminaison secondaire en lecture seule spécifique de l’itinéraire                        |
| :--------------------- | :----------------------------------------------------------------- |
| Service d'objets blob           | `StorageAccountA-internetrouting-secondary.blob.core.windows.net`  |
| Data Lake Storage Gen2 | `StorageAccountA-internetrouting-secondary.dfs.core.windows.net`   |
| Service Fichier           | `StorageAccountA-internetrouting-secondary.file.core.windows.net`  |
| Sites web statiques        | `StorageAccountA-internetrouting-secondary.web.core.windows.net`   |

Vous pouvez copier les chaînes de connexion pour les points de terminaison spécifiques de l’itinéraire publiés via le [portail Azure](https://portal.azure.com). Ces chaînes de connexion sont utilisables pour l’autorisation par clé partagée avec tous les kits de développement logiciel (SDK) et API de Azure Storage existants.

## <a name="about-the-preview"></a>À propos de la préversion

La préférence de routage pour le service Stockage Azure est disponible dans les régions suivantes :

- France Sud
- Centre-Nord des États-Unis
- Centre-USA Ouest

Les problèmes connus suivants affectent la préversion de la préférence de routage pour le service Stockage Azure :

- Les demandes d’accès au point de terminaison spécifique de l’itinéraire pour le réseau Microsoft mondial échouent avec l’erreur HTTP 404 ou une erreur équivalente. Le routage sur le réseau Microsoft mondial fonctionne comme prévu quand il est défini comme préférence de routage par défaut pour le point de terminaison public.

## <a name="pricing-and-billing"></a>Tarification et facturation

Pour plus de détails sur la tarification et la facturation, consultez la section **Tarification** dans [Qu’est-ce qu’une préférence de routage (préversion) ?](../../virtual-network/routing-preference-overview.md#pricing).

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’une préférence de routage (préversion) ?](../../virtual-network/routing-preference-overview.md)
- [Configurer des pare-feu et des réseaux virtuels dans Stockage Azure](storage-network-security.md)
- [Recommandations de sécurité pour Stockage Blob](../blobs/security-recommendations.md)
