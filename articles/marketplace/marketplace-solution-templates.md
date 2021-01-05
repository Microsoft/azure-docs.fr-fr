---
title: Guide de publication d’offres de modèle de solution d’applications Azure | Place de marché Azure
description: Cet article décrit les conditions requises pour la publication d’un modèle de solution sur la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: msjogarrig
ms.author: jogarrig
ms.date: 04/22/2020
ms.openlocfilehash: 475a8615e9233def091edc03fe91c82ea63cdc27
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433524"
---
# <a name="publishing-guide-for-azure-applications-solution-template-offers"></a>Guide de publication d’offres de modèle de solution d’applications Azure

Cet article explique les conditions requises pour la publication des offres de modèle de solution, qui est une façon de publier une offre d’application Azure dans la Place de marché Azure. Le type d’offre de modèle de solution nécessite un [modèle Azure Resource Manager (modèle ARM)](../azure-resource-manager/templates/overview.md) pour déployer automatiquement votre infrastructure de solution.

Utilisez le type d’offre de *modèle de solution* d’application Azure dans les conditions suivantes :

- Votre solution nécessite une automatisation supplémentaire de la configuration et du déploiement au-delà d’une seule machine virtuelle, par exemple une combinaison de machines virtuelles, de mise en réseau et de ressources de stockage.
- Vos clients vont gérer la solution eux-mêmes.

L’option de référencement que voit le client pour ce type d’offre est *Obtenir maintenant*.

## <a name="requirements-for-solution-template-offers"></a>Conditions requises pour les offres de modèles de solution

| **Configuration requise** | **Détails**  |
| ---------------  | -----------  |
|Facturation et mesure    |  Les offres de modèle de solution ne sont pas des offres de transaction, mais peuvent être utilisés pour déployer des offres de machines virtuelles payantes facturées via la place de marché commerciale de Microsoft. Les ressources que le modèle ARM de la solution déploie sont configurées dans l’abonnement Azure du client. Les machines virtuelles avec paiement à l’utilisation font l’objet de transactions avec le client par le biais de Microsoft et sont facturées dans le cadre de l’abonnement Azure du client.<br/> Pour la facturation BYOL (apportez votre propre licence), bien que Microsoft facture les frais d’infrastructure engagés dans l’abonnement client, vous effectuez la transaction de vos frais de licence logicielle directement avec le client.   |
|Disque dur virtuel compatible avec Azure  |   Les machines virtuelles doivent être basées sur Windows ou Linux. Pour plus d'informations, consultez les pages suivantes : <ul> <li>[Créer une offre d’application Azure](./create-new-azure-apps-offer.md) (pour les disques durs virtuels Windows).</li><li>[Distributions Linux approuvées sur Azure](../virtual-machines/linux/endorsed-distros.md) (pour les disques durs virtuel Linux).</li></ul> |
| Attribution de l’utilisation client | L’activation de l’attribution de l’utilisation de client est obligatoire pour tous les modèles de solution publiés sur la Place de marché Azure. Pour plus d’informations sur l’attribution de l’utilisation de client et sur son activation, consultez [Attribution de l’utilisation de client partenaire Azure](./azure-partner-customer-usage-attribution.md).  |
| Utiliser des disques managés | Les [disques managés](../virtual-machines/managed-disks-overview.md) représentent l’option par défaut pour les disques persistants des machines virtuelles IaaS (infrastructure as a service) dans Azure. Vous devez utiliser des disques managés dans les modèles de solution. <ul><li>Pour mettre à jour vos modèles de solution, suivez les instructions dans [Utiliser les disques managés dans les modèles Azure Resource Manager](../virtual-machines/using-managed-disks-template-deployments.md) et utilisez les [exemples](https://github.com/Azure/azure-quickstart-templates) fournis.<br><br> </li><li>Pour publier le disque dur virtuel sous forme d’image dans la Place de marché Azure, importez les disques durs virtuels sous-jacents des disques managés dans un compte de stockage à l’aide de l’une des méthodes suivantes :<ul><li>[Azure PowerShell](../virtual-machines/scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd.md) </li> <li> [L’interface de ligne de commande Microsoft Azure](../virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd.md) </li> </ul></ul> |

## <a name="next-steps"></a>Étapes suivantes

Si vous ne l’avez pas déjà fait, découvrez comment [développer votre activité dans le cloud avec la Place de marché Azure](https://azuremarketplace.microsoft.com/sell).

Pour vous inscrire à l’Espace partenaires et commencer à travailler dessus :

- [Connectez-vous à l’Espace partenaires](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) pour créer ou terminer votre offre.
- Pour plus d’informations, consultez [Créer une offre d’application Azure](./create-new-azure-apps-offer.md).