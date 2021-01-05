---
title: Guide de publication des offres de conteneur dans la Place de marché Azure
description: Cet article décrit les conditions requises pour publier des conteneur sur la Place de marché Azure.
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 11/30/2020
ms.openlocfilehash: 83c575aa40b80d9a8e39263e89a5e7860c8f8774
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95741659"
---
# <a name="publishing-guide-for-azure-container-offers"></a>Guide de publication des offres de conteneur Azure

Les offres de conteneur Azure vous aident à publier votre image conteneur sur la Place de marché Azure. Utilisez ce guide pour comprendre les exigences propres à ce type d’offre.

Les offres de conteneur Azure sont des offres de transaction qui sont déployées et facturées par le biais de la Place de marché Azure. L’option de référencement qu’un utilisateur voit est « Obtenir maintenant ».

Utilisez le type d’offre Conteneur Azure quand votre solution est une image conteneur Docker qui est configurée en tant qu’instance de conteneur Azure basée sur Kubernetes.

> [!NOTE]
> Une instance de conteneur Azure est une instance docker d’exécution qui offre la façon la plus simple et la plus rapide d’exécuter un conteneur dans Azure, sans avoir à gérer des machines virtuelles et sans avoir à adopter un service de niveau supérieur. Les instances de conteneur peuvent être déployées directement sur Azure ou orchestrées par Azure Kubernetes Services ou le moteur Azure Kubernetes Services.  

Microsoft prend actuellement en charge les modèles de licence BYOL (apportez votre propre licence) et gratuits.

## <a name="container-offer-requirements"></a>Exigences de l’offre conteneur

| Condition requise | Détails |  
|:--- |:--- |  
| Facturation et mesure | Prise en charge du modèle de facturation gratuit ou BYOL.<br><br> |  
| Image créée à partir d’un Dockerfile | Les images conteneur doivent être basées sur la spécification d’image Docker et générées à partir d’un Dockerfile.<br> <br>Pour plus d’informations sur la création d’images Docker, consultez la section « Utilisation » de la [documentation de référence de Dockerfile](https://docs.docker.com/engine/reference/builder/#usage).<br><br> |  
| Hébergement dans un référentiel Azure Container Registry | Les images conteneur doivent être hébergées dans un référentiel Azure Container Registry.<br> <br>Pour plus d’informations sur l’utilisation d’Azure Container Registry, consultez [Guide de démarrage rapide : Créer un registre de conteneurs privé à l’aide du Portail Azure](../container-registry/container-registry-get-started-portal.md).<br><br> |  
| Balisage d’images | Les images conteneur doivent contenir au moins une balise (nombre maximal de balises : 16).<br><br>Pour plus d’informations sur le balisage d’une image, consultez la page `docker tag` sur le site [Documentation Docker](https://docs.docker.com/engine/reference/commandline/tag).<br><br> |  

## <a name="next-steps"></a>Étapes suivantes

- Afin de préparer des ressources techniques pour une offre de conteneur, consultez [Créer des ressources techniques de conteneur Azure](create-azure-container-technical-assets.md).

- Pour créer une offre de conteneur Azure, consultez [Créer une offre de conteneur Azure dans la Place de marché Azure](create-azure-container-offer.md) afin d’obtenir plus d’informations.
