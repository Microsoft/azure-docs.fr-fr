---
title: Terminologie relative à Gestion des API Azure | Microsoft Docs
description: Cet article fournit les définitions des termes spécifiques à Gestion des API.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 10/11/2017
ms.author: apimpm
ms.openlocfilehash: 5bc76d2526c5585071a240af36b8a31e3de5708f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87024926"
---
# <a name="azure-api-management-terminology"></a>Terminologie relative à Gestion des API Azure

Cet article fournit les définitions des termes spécifiques à Gestion des API (APIM).

## <a name="term-definitions"></a>Définitions des termes

* **API de serveur principal** : un service HTTP qui implémente votre API et ses opérations. 
* **API de serveur frontal**/**API APIM** : une API APIM n’héberge pas d’API, elle crée des façades pour vos API afin de personnaliser la façade en fonction de vos besoins, sans toucher l’API de serveur principal. Pour plus d’informations, consultez [Import and publish your first API](import-and-publish.md) (Importer et publier votre première API).
* **Produit APIM** : un produit qui contient une ou plusieurs API, ainsi qu’un quota et des conditions d’utilisation. Vous pouvez inclure un certain nombre d’API et les proposer aux développeurs par le biais du portail des développeurs. Pour plus d’informations, consultez [Création et publication d’un produit dans Gestion des API Azure](api-management-howto-add-products.md).
* **Opération d’API APIM** : chaque API APIM représente un ensemble d’opérations disponibles pour les développeurs. Chaque API d’APIM contient une référence au service de serveur principal qui implémente l’API, et ses opérations effectuent un mappage vers les opérations implémentées par le service de serveur principal. Pour plus d’informations, consultez [Mock API responses](mock-api-responses.md) (Simuler des réponses d’API).
* **Version** : parfois, vous souhaitez publier de nouvelles fonctionnalités d’API ou des fonctionnalités d’API différentes pour certains utilisateurs, tandis que d’autres veulent rester sur l’API qui fonctionne actuellement pour eux. Pour plus d’informations, consultez [Publier plusieurs versions d’une API de manière prévisible](api-management-get-started-publish-versions.md).
* **Révision** : quand votre API est prête et commence à être utilisée par les développeurs, vous devez généralement veiller à ne pas interrompre les appelants de votre API quand vous apportez des modifications à cette dernière. Il est également utile d’informer les développeurs des modifications apportées. Pour plus d’informations, consultez l’article [Apporter des modifications sans rupture en toute sécurité à l’aide de révisions](api-management-get-started-revise-api.md).
* **Portail des développeurs** : vos clients (les développeurs) doivent utiliser le portail des développeurs pour accéder à vos API. Le portail des développeurs peut être personnalisé. Pour plus d’informations, consultez [Personnaliser le style du portail des développeurs dans Gestion des API Azure](api-management-customize-styles.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)

