---
title: Abonnements dans Gestion des API Azure | Microsoft Docs
description: Découvrez le concept des abonnements dans la Gestion des API Azure. Les consommateurs bénéficient d’un accès aux API à l’aide d’abonnements dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/14/2018
ms.author: apimpm
ms.openlocfilehash: cdc7668b06308bd5532f4885d0e1365fc34ae989
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511244"
---
# <a name="subscriptions-in-azure-api-management"></a>Abonnements dans Gestion des API Azure

Les abonnements représentent un concept important dans la Gestion des API Azure. C’est l’approche la plus courante pour permettre aux consommateurs d’API d’obtenir l’accès aux API publiées par l’intermédiaire d’une instance de Gestion des API. Cet article fournit une vue d’ensemble du concept.

## <a name="what-are-subscriptions"></a>Qu’est-ce qu’un abonnement ?

Lors de la publication d’API via la Gestion des API, il est facile et courant de sécuriser l’accès à ces API au moyen de clés d’abonnement. Les développeurs qui utilisent les API publiées doivent inclure une clé d’abonnement valide dans les requêtes HTTP lorsqu’ils effectuent des appels vers ces API. Sinon, les appels sont immédiatement rejetés par la passerelle de Gestion des API. Ils ne sont pas transférés vers les services backend.

Pour obtenir une clé d’abonnement permettant d’accéder aux API, un abonnement est nécessaire. Un abonnement est avant tout un conteneur nommé pour une paire de clés d’abonnement. Les développeurs devant utiliser les API publiées peuvent obtenir des abonnements. Et ils n’ont pas besoin de l’approbation des éditeurs d’API. Les éditeurs d’API peuvent également créer des abonnements directement pour les consommateurs d’API.

> [!TIP]
> La Gestion des API prend également en charge d’autres mécanismes de protection de l’accès aux API, entre autres :
> - [OAuth2.0](api-management-howto-protect-backend-with-aad.md)
> - [Certificats clients](api-management-howto-mutual-certificates-for-clients.md)
> - [Restrict caller IPs](./api-management-access-restriction-policies.md#RestrictCallerIPs)

## <a name="scope-of-subscriptions"></a>Étendue des abonnements

Les abonnements peuvent être associés à différentes étendues : un produit, toutes les API ou une API individuelle.

### <a name="subscriptions-for-a-product"></a>Abonnements pour un produit

Habituellement, les abonnements dans la Gestion des API sont toujours associés à une seule étendue de [produit d’API](api-management-terminology.md). Les développeurs trouvent la liste des produits sur le portail des développeurs. Puis, ils soumettent des requêtes d’abonnement pour les produits qu’ils souhaitent utiliser. Dès qu’une requête d’abonnement est approuvée, automatiquement ou par les éditeurs d’API, le développeur peut utiliser les clés contenues pour accéder à toutes les API du produit. À l’heure actuelle, le portail des développeurs affiche uniquement les abonnements de portée produit dans la section du profil utilisateur. 

![Abonnements de produit](./media/api-management-subscriptions/product-subscription.png)

> [!TIP]
> Dans certains scénarios, les éditeurs d’API peuvent parfois souhaiter publier un produit d’API à destination du public, sans passer par les abonnements. Ils peuvent pour cela décocher l’option **Exiger un abonnement** dans la page **Paramètres** du produit sur le portail Azure. De cette façon, toutes les API du produit sont accessibles sans clé API.

### <a name="subscriptions-for-all-apis-or-an-individual-api"></a>Abonnements pour toutes les API ou une API individuelle

Quand nous avons introduit le niveau [Consommation](https://aka.ms/apimconsumptionblog) de la Gestion des API, nous avons apporté quelques changements afin de simplifier la gestion des clés :
- Tout d’abord, nous avons ajouté deux étendues d’abonnement : toutes les API et une seule API. L’étendue des abonnements n’est plus limitée à un produit d’API. Vous pouvez maintenant créer des clés accordant l’accès à une API, ou à toutes les API au sein d’une instance de Gestion des API, sans préalablement devoir créer un produit et lui ajouter les API. De plus, chaque instance de Gestion des API intègre désormais un abonnement immuable à toutes les API. Cet abonnement simplifie et facilite les tests et le débogage des API dans la console de test.

- Par ailleurs, la Gestion des API autorise désormais les abonnements **autonomes**. Les abonnements ne doivent plus forcément être associés à un compte de développeur. Cette fonctionnalité est utile dans les scénarios où plusieurs développeurs ou équipes partagent un abonnement.

- Enfin, les éditeurs d’API peuvent à présent [créer des abonnements](api-management-howto-create-subscriptions.md) directement dans le portail Azure :

    ![Abonnements flexibles](./media/api-management-subscriptions/flexible-subscription.png)

## <a name="next-steps"></a>Étapes suivantes
Sachez-en plus sur la Gestion des API :

+ Découvrez les autres [concepts](api-management-terminology.md) de la Gestion des API.
+ Suivez nos [tutoriels](import-and-publish.md) sur la Gestion des API.
+ Consultez notre [page de FAQ](api-management-faq.md) pour prendre connaissance des questions courantes.
