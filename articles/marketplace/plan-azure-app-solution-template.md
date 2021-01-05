---
title: Planifier un modèle de solution pour une offre d’application Azure
description: Découvrez ce qui est nécessaire pour créer un plan de modèle de solution pour une nouvelle offre Azure Application à l’aide du portail de la Place de marché commerciale dans l’Espace partenaires Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 3e4d0513808cdc44fc71e182a07fa6b050d182ee
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452512"
---
# <a name="plan-a-solution-template-for-an-azure-application-offer"></a>Planifier un modèle de solution pour une offre d’application Azure

Cet article décrit les exigences liées à la publication d’un plan de modèle de solution pour une offre Azure Application. Un plan de modèle de solution est l’un des deux types de plans que couvrent les offres Azure Application. Pour plus d’informations sur la différence entre ces deux types de plans, consultez [Types de plans](plan-azure-application-offer.md#plans). Si vous ne l’avez déjà fait, lisez [Planifier une offre d’application Azure](plan-azure-application-offer.md).

Le type de plan de modèle de solution nécessite un [modèle Azure Resource Manager (modèle ARM)](../azure-resource-manager/templates/overview.md) pour déployer automatiquement votre infrastructure de solution.

## <a name="solution-template-requirements"></a>Spécifications du modèle de solution

| Spécifications | Détails |
| ------------ | ------------- |
| Facturation et mesure | Les plans de modèle de solution ne peuvent pas faire l’objet d’une transaction, mais peuvent être utilisés pour déployer des offres de machines virtuelles payantes facturées via la Place de marché commerciale Microsoft. Les ressources que le modèle ARM de la solution déploie sont configurées dans l’abonnement Azure du client. Les machines virtuelles avec paiement à l’utilisation font l’objet de transactions avec le client par le biais de Microsoft et sont facturées dans le cadre de l’abonnement Azure du client. <br><br> Pour la facturation BYOL (apportez votre propre licence), bien que Microsoft facture les frais d’infrastructure engagés dans l’abonnement client, vous effectuez la transaction de vos frais de licence logicielle directement avec le client. |
| Disque dur virtuel compatible avec Azure | Les machines virtuelles doivent être basées sur Windows ou Linux. Pour plus d'informations, consultez les pages suivantes :<ul><li>[Créer une ressource technique de machine virtuelle Azure](./azure-vm-create-certification-faq.md#address-a-vulnerability-or-an-exploit-in-a-vm-offer) (pour les disques durs virtuels Windows)</li><li>[Distributions Linux approuvées sur Azure](../virtual-machines/linux/endorsed-distros.md) (pour les disques durs virtuel Linux).</li></ul> |
| Attribution de l’utilisation client | L’activation de l’attribution de l’utilisation de client est obligatoire pour tous les modèles de solution publiés sur la Place de marché Azure. Pour plus d’informations sur l’attribution de l’utilisation de client et sur son activation, consultez [Attribution de l’utilisation de client partenaire Azure](azure-partner-customer-usage-attribution.md). |
| Utiliser des disques managés | Les [disques managés](../virtual-machines/managed-disks-overview.md) représentent l’option par défaut pour les disques persistants des machines virtuelles IaaS (infrastructure as a service) dans Azure. Vous devez utiliser des disques managés dans les modèles de solution.<ul><li>Pour mettre à jour vos modèles de solution, suivez les instructions dans [Utiliser les disques managés dans les modèles Azure Resource Manager](../virtual-machines/using-managed-disks-template-deployments.md) et utilisez les [exemples](https://github.com/Azure/azure-quickstart-templates) fournis.</li><li>Pour publier le disque dur virtuel sous forme d’image dans la Place de marché Azure, importez les disques durs virtuels sous-jacents des disques managés dans un compte de stockage à l’aide d’[Azure PowerShell](../virtual-machines/scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd.md) ou de l’[interface de ligne de commande Azure](../virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd.md).</ul> |
| Package de déploiement | Vous aurez besoin d’un package de déploiement qui permettra aux clients de déployer votre plan. Si vous créez plusieurs plans nécessitant la même configuration technique, vous pouvez utiliser le même package de plan. Pour plus d’informations, consultez la section suivante : Package de déploiement. |
|||

## <a name="deployment-package"></a>Package de déploiement

Le package de déploiement regroupe tous les fichiers de modèle requis pour ce plan, ainsi que toutes les ressources supplémentaires, empaquetés dans un fichier .zip.

Toutes les applications Azure doivent inclure les deux fichiers suivants dans le dossier racine d’une archive .zip :

- Un fichier de modèle Resource Manager nommé [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). Ce modèle qui définit les ressources à déployer dans l’abonnement Azure du client. Pour obtenir des exemples de modèles Resource Manager, consultez la [Galerie de modèles de démarrage rapide Microsoft Azure](https://azure.microsoft.com/documentation/templates/) ou le dépôt [GitHub : Modèles de démarrage rapide Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondant.
- Une définition d’interface utilisateur pour l’expérience de création d’applications Azure nommée [createUiDefinition.json](../azure-resource-manager/managed-applications/create-uidefinition-overview.md). Dans l’interface utilisateur, vous spécifiez les éléments qui permettent aux consommateurs de fournir des valeurs de paramètre.

Tailles maximales de fichiers prises en charge :

- jusqu’à 1 Go dans la taille totale de l’archive .zip compressée ;
- jusqu’à 1 Go pour tous les fichiers non compressés individuels au sein de l’archive .zip.

Toutes les nouvelles offres d’applications Azure doivent également inclure un GUID d’[attribution de l’utilisation de client partenaire Azure](azure-partner-customer-usage-attribution.md).

## <a name="azure-regions"></a>Régions Azure

Vous pouvez publier votre plan dans la région publique Azure, la région Azure Government, ou les deux. Avant de le publier dans [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md), testez et validez votre plan dans l’environnement, car certains points de terminaison peuvent être différents. Pour configurer et tester votre plan, demandez un compte d’essai depuis la page [Microsoft Azure Government Trial](https://azure.microsoft.com/global-infrastructure/government/request/).

En tant qu’éditeur, il vous incombe de mettre en place l’ensemble des contrôles de conformité, des mesures de sécurité et des bonnes pratiques nécessaires. Azure Government utilise des réseaux et des centres de données qui sont physiquement isolés (situés aux États-Unis uniquement).

Pour obtenir la liste des pays et régions que dessert la Place de marché commerciale, consultez [Prise en charge de la disponibilité géographique et des devises](marketplace-geo-availability-currencies.md).

Les services Azure Government gèrent des données soumises à certaines réglementations et exigences gouvernementales. Par exemple, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 et CJIS. Afin de renseigner les clients sur vos certifications pour ces programmes, vous pouvez fournir jusqu’à 100 liens décrivant ces certifications. Ces liens peuvent être des liens vers votre liste de plans directement sur le programme, ou des liens vers des descriptions de votre conformité avec ces certifications sur vos propres sites web. Ces liens sont visibles uniquement par les clients Azure Government.

## <a name="choose-who-can-see-your-plan"></a>Choisir qui peut voir votre plan

Vous pouvez configurer chaque plan de sorte qu’il soit visible de tous (public) ou d’un public spécifique (privé). Vous pouvez créer jusqu’à 100 plans dont jusqu’à 45 peuvent être privés. Vous pouvez créer un plan privé pour offrir différentes options de tarification ou configurations techniques à des clients spécifiques.

Vous accordez l’accès à un plan privé à l’aide d’ID d’abonnement Azure, avec la possibilité d’inclure une description de chaque ID d’abonnement que vous attribuez. Vous pouvez ajouter jusqu’à 10 ID d’abonnement manuellement ou jusqu’à 10 000 ID d’abonnement à l’aide d’un fichier .CSV. Les ID d’abonnements Azure sont représentés par des GUID, où les lettres doivent être en minuscules.

> [!NOTE]
> Si vous publiez un plan privé, vous pouvez modifier sa visibilité sur public ultérieurement. Toutefois, une fois que vous avez publié un plan public, vous ne pouvez pas changer sa visibilité sur privé.

Pour les plans de modèle de solution, vous pouvez également choisir de les masquer dans la Place de marché Azure. Vous pouvez le faire si le plan est uniquement déployé indirectement via un autre modèle de solution ou une application managée.

> [!NOTE]
> Les plans privés ne sont pas pris en charge avec les abonnements Azure souscrits via un revendeur participant au programme Fournisseurs de solutions cloud (CSP).

Pour plus d’informations, consultez [Offres privées dans la Place de marché commerciale Microsoft](private-offers.md).

## <a name="next-steps"></a>Étapes suivantes

- [Comment créer une offre d’application Azure dans la Place de marché commerciale](create-new-azure-apps-offer.md)