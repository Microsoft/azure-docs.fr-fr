---
title: Guide de publication par type d’offre - Place de marché commerciale Microsoft
description: Cet article décrit les types d’offres disponibles dans la Place de marché commerciale Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 10/06/2020
ms.openlocfilehash: 2191a4df5b319ec16a4a6116aa99cfac50c87d9b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462909"
---
# <a name="publishing-guide-by-offer-type"></a>Guide de publication par Type d’offre

Cet article décrit les types d’offres disponibles dans la Place de marché commerciale. Le *type d’offre* définit la structure de l’offre, qui inclut les métadonnées, les artefacts et autre contenu présentés dans la place de marché commerciale.

Après avoir [choisi une option de publication](determine-your-listing-type.md), vous devez choisir un type d’offre avant de commencer à créer votre offre dans l’espace partenaires. Le type d’offre correspond au type de l’offre de solution, d’application ou de service que vous souhaitez publier, ainsi qu’à son alignement sur les produits et services Microsoft.

Vous pouvez configurer un type d’offre unique de différentes manières pour proposer différents appels à l’action, options de publication, option de référencement, configurations ou tarifs. L’option de publication et la configuration du type d’offre s’alignent également sur les exigences techniques et les conditions d’éligibilité de l’offre.

Avant de créer votre offre, prenez connaissance des conditions d’éligibilité pour le magasin en ligne et le type d’offre ainsi que les exigences techniques de publication.

## <a name="list-of-offer-types"></a>Liste des types d’offre

Le tableau suivant montre les types d’offre de place de marché commerciale proposés dans l’espace partenaire.

| **Type d’offre**    | **Description**  |
| :------------------- | :-------------------|
| [**Azure Application**](plan-azure-application-offer.md) | Il existe deux types de plans d’application Azure : les _modèles de solution_ et les _applications managées_. Les deux types de plan prennent en charge l’automatisation du déploiement et de la configuration d’une solution au-delà d’une seule machine virtuelle. Vous pouvez automatiser la fourniture de plusieurs ressources, notamment les machines virtuelles, les ressources réseau et les ressources de stockage nécessaires pour fournir des solutions complexes, telles que des solutions IaaS. Les deux types de plans peuvent utiliser de nombreux types de ressources Azure, notamment, mais sans s’y limiter, les machines virtuelles.<ul><li>Les plans de **modèles de solution** constituent l’un des principaux moyens de publier une solution sur la place de marché commerciale. Les plans de modèle de solution ne peuvent pas faire l’objet d’une transaction sur la place de marché commerciale, mais peuvent être utilisés pour déployer des offres de machines virtuelles payantes facturées via la place de marché commerciale. Utilisez le type de plan du modèle de solution lorsque le client gère la solution et que les transactions sont facturées par le biais d’un autre plan.</li><br><li>Les plans d’**applications managées** vous permettent de créer et fournir facilement des applications clés en main et entièrement managées à vos clients. Ils ont les mêmes fonctionnalités que les plans de modèles de solution, malgré quelques différences essentielles :</li><ul><li> Les ressources sont déployées vers un groupe de ressources et sont gérées par l’éditeur de l’application. Le groupe de ressources est présent dans l’abonnement du consommateur, mais une identité du locataire de l’éditeur a accès au groupe de ressources.</li><li>En tant qu’éditeur, vous spécifiez le coût de la prise en charge de la solution en continu, et les transactions sont prises en charge via la place de marché commerciale.</li></ul>Utilisez le type de plan d’application managée lorsque vous ou votre client exigez que la solution soit gérée par un partenaire ou que vous déployiez une solution basée sur un abonnement.</ul> |
| [**Conteneur Azure**](marketplace-containers.md) | Utilisez le type d’offre Conteneur Azure si votre solution est une image de conteneur Docker approvisionnée en tant que service de conteneur Azure Kubernetes. |
| [**Machine virtuelle Azure**](marketplace-virtual-machines.md) | Utilisez le type d’offre Machine virtuelle si vous déployez une appliance virtuelle dans le cadre de l’abonnement associé à votre client. |
| [**Service de conseil**](./plan-consulting-service-offer.md) | Les services de conseil aident les clients à se connecter aux services pour gérer et étendre leur utilisation des services Azure, Dynamics 365 ou Power Suite.|
| [**Dynamics 365**](appsource-offer-publishing-guide.md) | Vous pouvez publier des offres AppSource qui étendent ou s’appuient sur Dynamics 365 Business Central, Dynamics 365 Customer Engagement, Power Apps et des applications de finance et opérations.|
| [**Module IoT Edge**](iot-edge-module.md) | Les modules Azure IoT Edge sont les plus petites unités de calcul gérées par IoT Edge. Ils peuvent contenir des services Microsoft (par exemple, Azure Stream Analytics), des services tiers ou votre propre code de solution. |
| [**Service géré**](partner-center-portal/create-new-managed-service-offer.md) | Vous pouvez créer des offres de service géré et gérer des abonnements ou des groupes de ressources délégués par le client via [Azure Lighthouse](../lighthouse/overview.md).|
| [**Application Power BI**<br/>**Microsoft 365**](appsource-offer-publishing-guide.md) | Vous pouvez publier des offres AppSource qui étendent ou s’appuient sur Power BI et Microsoft 365.|
| [**Software as a Service**](plan-saas-offer.md) | Utilisez le type d’offre SaaS (Logiciel en tant que service) pour permettre à votre client d’acheter votre solution technique SaaS sous la forme d’un abonnement. Pour obtenir des informations sur les exigences relatives à l’authentification unique pour les offres SaaS, consultez [Azure AD et offres SaaS pouvant faire l’objet d’une transaction dans la Place de marché commerciale](azure-ad-saas.md). |


## <a name="next-steps"></a>Étapes suivantes

- Passez en revue les conditions d’éligibilité dans l’article correspondant à votre type d’offre afin de finaliser la sélection et la configuration de votre offre.
- Passez en revue les modèles de publication de chaque magasin en ligne pour obtenir des exemples sur la manière dont votre solution correspond à un type d’offre et à une configuration.