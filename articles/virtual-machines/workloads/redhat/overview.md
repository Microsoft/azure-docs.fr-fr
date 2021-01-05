---
title: Vue d’ensemble des charges de travail Red Hat sur Azure | Microsoft Docs
description: Découvrez les offres de produits Red Hat disponibles sur Azure.
author: asinn826
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: overview
ms.date: 02/10/2020
ms.author: alsin
ms.reviewer: cynthn
ms.openlocfilehash: 09612ff3d86c669801f9e824d367a2b9a5bb10ac
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97830647"
---
# <a name="red-hat-workloads-on-azure"></a>Charges de travail Red Hat sur Azure

Les charges de travail Red Hat sont prises en charge par le biais de diverses offres sur Azure. Les images RHEL (Red Hat Enterprise Linux) sont au cœur des charges de travail RHEL, tout comme l’infrastructure RHUI (Red Hat Update Infrastructure).

## <a name="red-hat-enterprise-linux-images"></a>Images Red Hat Enterprise Linux

Azure propose une offre étendue d’images RHEL sur Azure. Ces images sont mises à disposition par le biais de deux modèles de licence différents : paiement à l’utilisation et Bring Your Own Subscription (BYOS). De nouvelles images RHEL sur Azure sont publiées quand de nouvelles versions RHEL sont commercialisées et mises à jour tout au long de leur cycle de vie, si nécessaire.

### <a name="pay-as-you-go-images"></a>Images avec paiement à l’utilisation

Azure propose différentes images RHEL avec paiement à l’utilisation. Ces images sont correctement autorisées pour RHEL et sont associées à une source de mises à jour (Red Hat Update infrastructure). Des frais Premium sont facturés pour ces images pour les droits d’utilisation de RHEL et les mises à jour. Les variantes d’images RHEL avec paiement à l’utilisation sont notamment :

* RHEL standard
* RHEL for SAP
* RHEL for SAP avec des services de haute disponibilité et de mise à jour.

Vous pouvez utiliser les images avec paiement à l’utilisation si vous ne souhaitez pas vous soucier de payer séparément le nombre approprié d’abonnements.

### <a name="red-hat-gold-images"></a>Images Red Hat Gold

Azure propose également des images Red Hat Gold (`rhel-byos`). Ces images peuvent être utiles aux clients qui ont des abonnements Red Hat existants et souhaitent les utiliser dans Azure. Vous devez activer vos abonnements Red Hat existants pour Red Hat Cloud Access avant de pouvoir les utiliser dans Azure. L’accès à ces images vous est accordé automatiquement quand vos abonnements Red Hat sont activés pour Cloud Access et qu’ils répondent aux conditions d’éligibilité. L’utilisation de ces images permet à un client d’éviter une double facturation qui peut être encourue suite à l’utilisation des images avec paiement à l’utilisation.
* Découvrez comment [activer vos abonnements Red Hat pour Cloud Access avec Azure](https://access.redhat.com/documentation/en-us/red_hat_subscription_management/1/html/red_hat_cloud_access_reference_guide/enabling-and-maintaining-subs_cloud-access).
* Découvrez comment [localiser les images Red Hat Gold à l’aide du portail Azure, de l’interface Azure CLI ou d’une applet de commande PowerShell](./byos.md).

> [!NOTE]
> Une double facturation est encourue quand un utilisateur paye deux fois les abonnements RHEL. Ce scénario se produit généralement quand un client utilise un gestionnaire d’abonnements Red Hat pour joindre des droits d’utilisation à une machine virtuelle RHEL avec paiement à l’utilisation. Par exemple, les clients qui utilisent le gestionnaire d’abonnements pour joindre des droits d’utilisation pour des packages SAP sur une image RHEL avec paiement à l’utilisation font indirectement l’objet d’une double facturation, car ils paient deux fois pour RHEL. Ils paient non seulement dans le cadre des frais Premium de paiement à l’utilisation, mais également dans le cadre de leur abonnement SAP. Les utilisateurs d’images BYOS sont à l’abri de ce scénario.

### <a name="generation-2-images"></a>Images de 2e génération

Les machines virtuelles de 2e génération fournissent quelques nouvelles fonctionnalités par rapport aux machines virtuelles de 1re génération. Pour plus d’informations, consultez la [documentation sur la 2e génération](../../generation-2.md). En ce qui concerne les images RHEL, la principale différence est la suivante : les machines virtuelles de 2e génération utilisent une interface UEFI et non une interface de microprogramme BIOS. Par ailleurs, au moment du démarrage, elles utilisent une table de partition GUID et non un enregistrement de démarrage principal. L’utilisation d’une table de partition GUID permet, entre autres, des tailles de disque de système d’exploitation supérieures à 2 To. De plus, les [machines virtuelles de la série Mv2](../../mv2-series.md) s’exécutent uniquement sur les images de 2e génération.

Les images RHEL de 2e génération sont disponibles sur la Place de Marché Azure. Recherchez « gen2 » dans la référence SKU d’image dans la liste complète des images qui s’affiche quand vous utilisez l’interface Azure CLI. Accédez à l’onglet **Avancé** durant le processus de déploiement de la machine virtuelle pour déployer une machine virtuelle de 2e génération.

## <a name="red-hat-update-infrastructure"></a>Infrastructure de mise à jour Red Hat

Azure fournit une infrastructure de mise à jour Red Hat (RHUI, Red Hat Update Infrastructure) uniquement pour les machines virtuelles RHEL avec paiement à l’utilisation. RHUI est en fait un miroir des réseaux de distribution de contenu Red Hat, mais n’est accessible qu’aux machines virtuelles RHEL avec paiement à l’utilisation Azure. Vous avez accès aux packages appropriés en fonction de l’image RHEL que vous avez déployée. Par exemple, une image RHEL for SAP a accès aux packages SAP en plus des packages RHEL de base.

### <a name="rhui-update-behavior"></a>Comportement de mise à jour RHUI

Les images RHEL connectées à RHUI effectuent, par défaut, une mise à jour vers la dernière version mineure de RHEL quand une opération `yum update` est exécutée. Ce comportement signifie qu’une machine virtuelle RHEL 7.4 peut être mise à niveau vers RHEL 7.7 si une opération `yum update` est exécutée sur celle-ci. Ce comportement est normal pour l’infrastructure RHUI. Pour atténuer ce comportement de mise à niveau, basculez des dépôts RHEL standard vers les [dépôts EUS (Extended Update support)](./redhat-rhui.md#rhel-eus-and-version-locking-rhel-vms).

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur les [images RHEL sur Azure](./redhat-images.md).
* Apprenez-en davantage sur l’[infrastructure RHUI (Red Hat Update Infrastructure)](./redhat-rhui.md).
* Apprenez-en davantage sur l’[offre d’images Red Hat Gold (`rhel-byos`)](./byos.md).