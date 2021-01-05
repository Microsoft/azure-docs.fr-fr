---
title: API de traitement SaaS dans la Place de marché commerciale Microsoft
description: Présentation des API de traitement qui vous permettent d’intégrer vos offres SaaS dans Microsoft AppSource et Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/18/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 2e6381afb19018822f6f37171a5ca4b3d929b42e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88037522"
---
# <a name="saas-fulfillment-apis-in-the-microsoft-commercial-marketplace"></a>API de traitement SaaS dans la Place de marché commerciale Microsoft

Les API de traitement SaaS permettent aux éditeurs de logiciels indépendants (ISV) de publier et vendre leurs applications SaaS dans Microsoft AppSource et Place de marché Azure. Ces API permettent aux applications d’éditeurs de logiciels indépendants d’utiliser tous les canaux commerciaux : directs, gérés par des partenaires (revendeurs) et gérés par des acteurs de terrain.  L’intégration avec ces API est une condition requise pour la création et la publication d’une offre SaaS dans l’espace partenaires.

Les éditeurs de logiciels indépendants doivent implémenter les flux d’API suivants en les ajoutant à leur code de service SaaS pour conserver le même état d’abonnement pour les éditeurs de logiciels indépendants (ISV) et Microsoft :

* Flux de la page d’accueil :  Microsoft informe l’éditeur que l’offre SaaS de l’éditeur a été achetée par un client sur la place de marché.
* Flux d’activation :  L’éditeur avertit Microsoft qu’un compte SaaS nouvellement acheté a été configuré côté éditeur.
* Flux de mise à jour : Modification du plan acheté et/ou du nombre de postes achetés.
* Flux de suspension et de réactivation : Interruption de l’offre SaaS achetée si le mode de paiement du client n’est plus valide. L’offre interrompue peut être rétablie lorsque le problème de mode de paiement est résolu.
* Flux de webhook : Microsoft informe l’éditeur des modifications d’abonnement SaaS et de l’annulation déclenchée par le client du côté Microsoft.

Pour l’annulation de l’abonnement SaaS acheté, l’intégration est facultative, car elle peut être effectuée par le client du côté Microsoft.

Une intégration correcte avec les API de réalisation SaaS est essentielle pour s’assurer que

* les clients finaux qui ont acheté l’offre SaaS de l’éditeur sont facturés correctement par Microsoft.
* les clients finaux bénéficient de la bonne expérience utilisateur pour l’achat, la configuration, l’utilisation et la gestion des abonnements SaaS achetés sur la place de marché.

Ces API permettent aux offres de l’éditeur de participer à tous les canaux de commerce activés :

* directes
* partenaire (revendeur, CSP)
* sur le terrain

Dans le scénario revendeur (CSP), un CSP achète l’offre SaaS au nom du client final. Un client est censé utiliser l’offre SaaS, mais le CSP est l’entité qui effectue les opérations suivantes :

* facturation du client
* modification des plans d’abonnement/de la quantité de postes achetés
* annulation des abonnements

L’éditeur n’est pas tenu d’implémenter un flux d’appels d’API différent pour ce scénario.

Pour plus d'informations sur les CSP, consultez https://partner.microsoft.com/licensing.

>[!Warning]
>La version actuelle de cette API (version 2) doit être utilisée pour toutes les nouvelles offres SaaS. La version 1 de l’API est déconseillée. Elle est conservée pour prendre en charge des offres existantes.

>[!Note]
>Les API de traitement SaaS sont uniquement destinées à être appelées à partir d’un service principal de l’éditeur. L’intégration avec les API directement à partir de la page web de l’éditeur n’est pas prise en charge. Seul le workflow d’authentification de service à service doit être utilisé.

## <a name="next-steps"></a>Étapes suivantes

Si ce n’est déjà fait, inscrivez votre application SaaS via le [portail Azure](https://ms.portal.azure.com), comme expliqué dans [Inscrire une application Azure AD](./pc-saas-registration.md).  Ensuite, utilisez la version la plus récente de cette interface pour le développement : [API de traitement SaaS version 2](./pc-saas-fulfillment-api-v2.md).
