---
title: Considérations relatives à la conception de Microsoft Azure Virtual Machine Scale Sets
description: En savoir plus sur les considérations relatives à la conception de Microsoft Azure Virtual Machine Scale Sets. Comparez les fonctionnalités des groupes identiques à celles des machines virtuelles.
keywords: machine virtuelle linux, groupes de machines virtuelles identiques
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 06/25/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 99dc7a2350631f662e1c993908f7ef56e4f9a194
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88648562"
---
# <a name="design-considerations-for-scale-sets"></a>Considérations relatives à la conception des groupes de machines virtuelles identiques
Cet article présente les considérations à prendre en compte pour créer des groupes de machines virtuelles identiques. Pour plus d'informations sur les groupes de machines virtuelles identiques, reportez-vous à la rubrique [Présentation des groupes de machines virtuelles identiques](./overview.md).

## <a name="when-to-use-scale-sets-instead-of-virtual-machines"></a>Quand utiliser des groupes identiques plutôt que des machines virtuelles individuelles ?
En règle générale, les groupes identiques sont utiles pour le déploiement d’infrastructures hautement disponibles où un ensemble de machines a une configuration similaire. Toutefois, certaines fonctionnalités sont uniquement disponibles dans les groupes identiques, tandis que d’autres le sont seulement dans les machines virtuelles individuelles. Afin de prendre une décision avisée et déterminer dans quel cas utiliser l’une ou l’autre de ces technologies, vous devez tout d’abord examiner certaines des fonctionnalités couramment utilisées disponibles avec les groupes identiques, mais pas avec les machines virtuelles individuelles :

### <a name="scale-set-specific-features"></a>Fonctionnalités spécifiques des groupes identiques

- Une fois la configuration du groupe identique spécifiée, vous pouvez mettre à jour la propriété *capacity* pour déployer davantage de machines virtuelles en parallèle. Ce processus est bien mieux que l’écriture d’un script pour orchestrer le déploiement d’un grand nombre de machines virtuelles individuelles en parallèle.
- Vous pouvez [utiliser la mise à l’échelle automatique d’Azure pour dimensionner automatiquement un groupe identique](./virtual-machine-scale-sets-autoscale-overview.md), mais pas les machines virtuelles individuelles.
- Vous pouvez [réinitialiser les machines virtuelles des groupes identiques](/rest/api/compute/virtualmachinescalesets/reimage), mais [pas les machines virtuelles individuelles](/rest/api/compute/virtualmachines).
- Vous pouvez [sur-approvisionner](#overprovisioning) les machines virtuelles d’un groupe identique pour une fiabilité accrue et un déploiement plus rapide. Vous ne pouvez pas sur-approvisionner des machines virtuelles individuelles à moins d’écrire un code personnalisé pour effectuer cette opération.
- Vous pouvez spécifier une [stratégie de mise à niveau](./virtual-machine-scale-sets-upgrade-scale-set.md) pour faciliter le déploiement des mises à niveau sur les machines virtuelles de votre groupe identique. Avec des machines virtuelles individuelles, vous devez orchestrer les mises à jour vous-même.

### <a name="vm-specific-features"></a>Fonctionnalités spécifiques des machines virtuelles

Certaines fonctionnalités sont actuellement disponibles uniquement dans les machines virtuelles :

- Vous pouvez capturer une image à partir d’une machine virtuelle individuelle, mais cela n’est pas possible dans un groupe identique.
- Vous pouvez migrer une machine virtuelle individuelle à partir de disques natifs vers des disques managés, mais vous ne pouvez pas migrer des instances de machine virtuelle dans un groupe identique.
- Vous pouvez assigner des adresses IP publiques IPv6 à des cartes d’interface réseau de machines virtuelles individuelles, mais cela n’est pas possible pour des instances de machine virtuelle dans un groupe identique. Vous pouvez assigner des adresses IP publiques IPv6 aux équilibreurs de charge des machines virtuelles individuelles ou des machines de groupes identiques.

## <a name="storage"></a>Stockage

### <a name="scale-sets-with-azure-managed-disks"></a>Groupes identiques avec Azure Disques managés
Les groupes identiques peuvent être créés avec [Azure Disques managés](../virtual-machines/managed-disks-overview.md) plutôt qu’à partir des comptes de stockage Azure traditionnels. La fonctionnalité Disques managés permet de bénéficier des avantages suivants :
- Vous n’êtes pas obligé de créer au préalable un ensemble de comptes de stockage Azure pour les machines virtuelles du groupe identique.
- Vous pouvez définir des [disques de données associés](virtual-machine-scale-sets-attached-disks.md) pour les machines virtuelles de votre groupe identique.
- Les groupes identiques peuvent être configurés pour [prendre en charge jusqu’à 1 000 machines virtuelles par groupe](virtual-machine-scale-sets-placement-groups.md). 

Si vous avez déjà un modèle, vous pouvez également [le mettre à jour pour qu’il utilise la fonctionnalité Disques managés](virtual-machine-scale-sets-convert-template-to-md.md).

### <a name="user-managed-storage"></a>Stockage géré par l’utilisateur
Un groupe identique qui n’est pas défini avec Azure Disques managés repose sur les comptes de stockage créés par l’utilisateur pour stocker les disques du système d’exploitation des machines virtuelles dans le groupe. Un rapport de 20 machines virtuelles maximum par compte de stockage est recommandé pour atteindre la valeur d’E/S maximale, mais aussi pour tirer parti du _sur-approvisionnement_ (voir ci-dessous). Il est également recommandé de répartir dans l’alphabet les premiers caractères des noms de comptes de stockage. Cette méthode améliore la répartition sur les différents systèmes internes. 


## <a name="overprovisioning"></a>Sur-approvisionnement
Par défaut, les groupes identiques activent « le sur-approvisionnement » de machines virtuelles. Avec le sur-approvisionnement activé, le groupe identique crée en réalité plus de machines virtuelles par rapport au nombre demandé, puis supprime les machines virtuelles supplémentaires une fois ce nombre de machines virtuelles approvisionné. Le sur-approvisionnement améliore les taux de réussite de l’approvisionnement et réduit la durée du déploiement. Vous n’êtes pas facturé pour les machines virtuelles supplémentaires et elles ne seront pas comptabilisées dans vos limites de quotas.

Bien que le sur-approvisionnement améliore les taux de réussite de l’approvisionnement, il peut provoquer un comportement déroutant si une application n’est pas conçue pour gérer les machines virtuelles qui disparaissent, puis réapparaissent. Pour désactiver le sur-approvisionnement, vérifiez que votre modèle contient la chaîne suivante : `"overprovision": "false"`. Vous trouverez plus de détails dans la [documentation de l’API REST de groupes identiques](/rest/api/virtualmachinescalesets/create-or-update-a-set).

Si votre groupe identique utilise le stockage géré par l’utilisateur et que vous désactivez le sur-approvisionnement, vous pouvez avoir plus de 20 machines virtuelles par compte de stockage, mais il n’est pas recommandé de dépasser 40 pour des raisons de performances d’E/S. 

## <a name="limits"></a>limites
Un groupe identique, basé sur une image de la Place de marché (également appelée image de plateforme) et configuré pour utiliser Azure Disques managés, peut prendre en charge jusqu’à 1 000 machines virtuelles. Si vous configurez votre groupe identique pour prendre en charge plus de 100 machines virtuelles, tous les scénarios ne fonctionnent pas de la même manière (par exemple l’équilibrage de charge). Pour plus d’informations, consultez [Utilisation de grands groupes de machines virtuelles identiques](virtual-machine-scale-sets-placement-groups.md). 

Un groupe identique configuré avec des comptes de stockage gérés par l’utilisateur est actuellement limité à 100 machines virtuelles (et 5 comptes de stockage sont recommandés pour cette échelle).

Un groupe identique basé sur une image personnalisée (créée par vous-même) peut prendre en charge jusqu’à 600 machines virtuelles s’il est configuré avec Azure Disques Managés. Si le groupe identique est configuré avec des comptes de stockage gérés par l’utilisateur, il doit créer tous les disques durs virtuels du disque du système d’exploitation dans un même compte de stockage. Par conséquent, le nombre maximal recommandé de machines virtuelles dans un groupe identique basé sur une image personnalisée et dont le stockage est géré par l’utilisateur est de 20. Si vous désactivez le sur-approvisionnement, vous pouvez aller jusqu’à 40.

Si vous souhaitez avoir plus de machines virtuelles que ne l’autorisent ces limites, vous devez déployer plusieurs groupes identiques, comme indiqué dans [ce modèle](https://github.com/Azure/azure-quickstart-templates/tree/master/301-custom-images-at-scale).
