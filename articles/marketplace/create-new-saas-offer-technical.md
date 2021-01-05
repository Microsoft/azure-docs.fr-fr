---
title: Comment ajouter des détails techniques pour votre offre SaaS dans l’Espace partenaires Microsoft
description: Découvrez comment fournir des détails techniques pour votre offre SaaS pour la place de marché commerciale de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: 28c0be40387f411286230f94c19fa23a80e650af
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746401"
---
# <a name="how-to-add-technical-details-for-your-saas-offer"></a>Comment ajouter des détails techniques pour votre offre SaaS

Cet article explique comment entrer des détails techniques qui permettent à la place de marché commerciale de Microsoft de se connecter à votre solution. Cette connexion permet d’approvisionner votre offre pour le client, s’il choisit de l’acquérir et de la gérer. Pour plus d’informations sur ces paramètres, consultez [Informations techniques](plan-saas-offer.md#technical-information).

> [!NOTE]
> Si vous décidez de traiter les transactions de façon indépendante, vous ne verrez pas cette option. Au lieu de cela, passez à [Comment vendre votre offre SaaS](create-new-saas-offer-marketing.md).

## <a name="technical-configuration"></a>Configuration technique

Sous l’onglet **Configuration technique**, vous allez définir les détails techniques que la place de marché commerciale utilise pour communiquer avec votre application ou solution SaaS. 

- **URL de la page d’arrivée** (obligatoire) : définissez l’URL du site SaaS (par exemple : `https://contoso.com/signup`) sur laquelle les clients finaux arriveront après avoir acquis votre offre sur la place de marché commerciale et déclenché le processus de configuration à partir de l’abonnement SaaS nouvellement créé.

  > [!IMPORTANT]
  > Votre page d’arrivée doit être opérationnelle en permanence. C’est la seule façon pour vous d’être averti des nouveaux achats de vos offres SaaS effectuées dans la place de marché commerciale, ou des demandes de configuration d’un abonnement actif à une offre.

- **Webhook de connexion** (obligatoire) : pour tous les événements asynchrones que Microsoft doit vous envoyer (par exemple, l’abonnement SaaS a été annulé), nous vous demandons de fournir une URL de Webhook de connexion. Nous appellerons cette URL pour vous informer de l’événement.

  > [!IMPORTANT]
  > Votre webhook doit être opérationnel en permanence, car c’est pour vous la seule façon d’être informé des mises à jour concernant les abonnements SaaS de vos clients achetés via la place de marché commerciale.

- **ID de locataire Azure Active Directory** (obligatoire) : pour trouver l’ID de locataire de votre application Azure Active Directory (AD), accédez au panneau [Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) dans Azure Active Directory. Dans la colonne **Nom d’affichage**, sélectionnez l’application. Recherchez ensuite le numéro d’**ID de répertoire (locataire)** référencé (par exemple, `50c464d3-4930-494c-963c-1e951d15360e`).

- **ID d’application Azure Active Directory** (obligatoire) : pour rechercher votre [ID d’application](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in), accédez au panneau [inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) dans Azure Active Directory. Dans la colonne **Nom d’affichage**, sélectionnez l’application. Recherchez ensuite le numéro d’ID d’application (client) référencé (par exemple, `50c464d3-4930-494c-963c-1e951d15360e`).

Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : Vue d’ensemble du plan.

## <a name="next-steps"></a>Étapes suivantes

- [Comment créer des plans pour votre offre SaaS](create-new-saas-offer-plans.md).