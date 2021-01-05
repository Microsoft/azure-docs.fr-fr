---
title: Comment ajouter des détails techniques pour votre offre Azure Application
description: Découvrez comment ajouter des détails techniques pour votre offre Azure Application dans l’Espace partenaires.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 11/06/2020
ms.openlocfilehash: a8658242bca48ee490e7fd44582f46e7c9c78e5b
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369737"
---
# <a name="how-to-add-technical-details-for-your-azure-application-offer"></a>Comment ajouter des détails techniques pour votre offre Azure Application

Cet article explique comment entrer des détails techniques qui permettent à la place de marché commerciale de Microsoft de se connecter à votre solution. Cette connexion permet d’approvisionner votre offre pour le client, s’il choisit de l’acquérir et de la gérer.

Suivez cette section uniquement si votre offre comprend une application managée qui émet des événements de mesure utilisant les [API de facturation à la consommation de la Place de marché](partner-center-portal/marketplace-metering-service-apis.md) et dispose d’un service qui s’authentifie avec un jeton de sécurité Azure AD. Pour plus d’informations, consultez les différentes options d’authentification dans [Stratégies d’authentification du service de mesure de la Place de marché](partner-center-portal/marketplace-metering-service-authentication.md).

## <a name="technical-configuration-offer-level"></a>Configuration technique (niveau de l’offre)

L’onglet **Configuration technique** ne vous concerne que si vous créez une application managée qui émet des événements de mesure utilisant les [API de facturation à la consommation de la Place de marché](partner-center-portal/marketplace-metering-service-apis.md). Dans ce cas, suivez les étapes décrites. Dans le cas contraire, passez aux [étapes suivantes](#next-steps). 

Pour plus d’informations sur ces champs, consultez [Planifier une offre Azure Application pour la Place de marché commerciale](plan-azure-application-offer.md#technical-configuration).

1. Sous l’onglet **Configuration technique**, entrez l’**ID de locataire Azure Active Directory** et l’**ID d’application Azure Active Directory** utilisés pour vérifier que la connexion entre nos deux services se trouve derrière une communication authentifiée.

1. Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : Vue d’ensemble du plan.

## <a name="next-steps"></a>Étapes suivantes

- [Comment créer des plans pour votre offre Azure Application](create-new-azure-apps-offer-plans.md)
