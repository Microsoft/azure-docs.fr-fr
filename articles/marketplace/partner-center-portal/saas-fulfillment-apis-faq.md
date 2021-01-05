---
title: Questions fréquentes (FAQ) sur l’API de traitement SaaS – Place de marché commerciale Microsoft
description: Découvrez plusieurs conditions d’intégration de la Place de marché commerciale Microsoft permettant aux clients Azure de s’abonner à des offres SaaS.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 07/11/2019
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 4c5d8b438764fa9aa3838b2225c63d412afc519b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88606808"
---
# <a name="common-questions-about-saas-fulfillment-apis"></a>Questions courantes sur les API de traitement SaaS

Cet article décrit plusieurs conditions d’intégration de la Place de marché commerciale Microsoft permettant aux clients Azure de s’abonner à des offres SaaS.

## <a name="discovery-experience"></a>Expérience de découverte

Une fois l’offre SaaS publiée, les utilisateurs Azure peuvent la découvrir sur la Place de marché Azure. Vos clients peuvent filtrer les offres en fonction du type de produit (SaaS) et découvrir les services SaaS qui les intéressent.

## <a name="purchase-experience"></a>Expérience d’achat

Si un utilisateur est intéressé par un service SaaS spécifique, il peut s'y abonner depuis la Place de marché Azure.

### <a name="what-does-it-mean-for-an-azure-user-to-subscribe-to-a-saas-offer-in-azure-marketplace"></a>Qu’implique le fait pour un utilisateur Azure de souscrire une offre SaaS sur la Place de marché Azure ?

Cet utilisateur peut alors consulter les conditions d'utilisation et la déclaration de confidentialité associées au service SaaS, et accepter de payer selon les conditions de facturation que vous, l'éditeur de l'offre SaaS, avez définies sur la facture Microsoft. Les utilisateurs peuvent utiliser leur profil de paiement existant dans Azure pour payer la consommation du service SaaS.

Cette possibilité s’avère avantageuse à plusieurs égards. Les clients peuvent désormais découvrir et s'abonner en un seul endroit en utilisant Microsoft Cloud Platform comme source de confiance, sans avoir à vérifier tous les logiciels ISV qu'ils souhaitent utiliser. De plus, les clients peuvent utiliser leur profil de paiement existant sans avoir à payer explicitement chaque logiciel ISV séparément.

### <a name="is-the-user-charged-automatically-when-the-offer-is-subscribed"></a>L'utilisateur est-il automatiquement facturé lors de la souscription de l'offre ?

En souscrivant l'offre SaaS, l'utilisateur a accepté de payer pour la consommation du service SaaS via la plate-forme Microsoft. Toutefois, les frais ne commenceront à s'appliquer que lorsque l'offre sera effectivement consommée. L'utilisateur doit se rendre sur votre offre SaaS et confirmer la création de son compte pour commencer à consommer l'offre. Vous demanderez ensuite à Microsoft de commencer à facturer l'abonnement SaaS de ce client.

### <a name="how-are-you-notified-when-a-user-subscribes-to-your-saas-offer"></a>Comment êtes-vous averti lorsqu'un utilisateur souscrit votre offre SaaS ?

Après avoir souscrit une offre, l'utilisateur Azure peut découvrir et gérer toutes ses offres sur Azure. Par défaut, une offre SaaS nouvellement souscrite présente l’état **Provisioning, fulfillment pending** (Provisionnement, en attente de traitement). Dans cet état, l’utilisateur Azure est invité à **Configurer un compte** pour accéder à son expérience de gestion d’abonnement SaaS sur le portail Azure.

Quand l’utilisateur sélectionne **Configurer un compte**, il est redirigé vers le site web du service SaaS. L’éditeur a configuré l’URL au moment de publier l’offre. Cette page est appelée page d'accueil de l'éditeur. Les utilisateurs Azure se connectent à la page d’accueil SaaS en fonction de leurs informations d’identification AAD existantes dans Azure.

Lorsque l'utilisateur Azure est redirigé vers la page d'accueil, un jeton est ajouté à l'URL de la requête. Ce jeton de courte durée est valide pendant 24 heures. Vous pouvez alors détecter la présence de ce jeton et appeler l'API Microsoft pour obtenir plus de contexte sur celui-ci.

![Flux d’abonnement du client](media/saas-metering-service-integration-flow-a.png)

Pour plus d’informations sur le contrat API pour la gestion des scénarios Transaction dans le cycle de vie d’une offre SaaS, consultez [API de traitement SaaS ](pc-saas-fulfillment-api-v2.md).

### <a name="how-do-you-know-the-saas-offer-to-which-the-user-subscribes-in-azure"></a>Comment identifier l'offre SaaS que l'utilisateur souscrit dans Azure ?

La réponse à l'API `Resolve` inclut des informations sur l'offre et le forfait associés à l'abonnement SaaS.

### <a name="how-can-the-azure-user-change-the-plan-associated-with-this-azure-subscription"></a>Comment l’utilisateur Azure peut-il modifier le forfait associé à cet abonnement Azure ?

* L’utilisateur Azure peut modifier le forfait associé à l’abonnement SaaS directement dans l’expérience SaaS, ou via la plate-forme Microsoft.

* Des conversions peuvent être effectuées à n’importe quel stade du cycle de facturation. Vous êtes invité à accuser réception de toute conversion, laquelle entre en vigueur une fois qu’elle est confirmée.

* Les tarifs des forfaits prépayés ( **mensuels** ou  **annuels**) sont calculés au prorata. Tout dépassement constaté jusqu'au moment de la conversion sera reporté sur la facture suivante. Les nouveaux dépassements seront reportés en fonction du nouveau forfait.

>[!Note]
>Vous pouvez bloquer le passage à une version antérieure si vous ne souhaitez pas prendre en charge des chemins de conversion spécifiques.

La séquence ci-dessous montre le flux quand un client Azure change de forfait dans l’expérience SaaS :

![Flux de changement du forfait d’un client](media/saas-metering-service-integration-flow-b.png)

La séquence ci-dessous montre le flux lorsqu’un client Azure change de forfait dans le magasin en ligne de Microsoft :

![Flux de changement de forfait dans le magasin en ligne client](media/saas-metering-service-integration-flow-c.png)

### <a name="how-can-the-azure-user-unsubscribe-from-the-plan-associated-with-azure-subscription"></a>Comment l’utilisateur Azure peut-il se désabonner du forfait associé à un abonnement Azure ?

Un utilisateur Azure peut se désabonner d'une offre SaaS achetée directement dans l'expérience SaaS ou via la plate-forme Microsoft. Lorsque l'utilisateur se désabonne, il ne sera plus facturé à partir du prochain cycle de facturation.

La séquence ci-dessous montre le flux lorsqu'un client Azure se désabonne de l’offre SaaS dans l’expérience SaaS :

![Le client se désabonne dans l'expérience SaaS](media/saas-metering-service-integration-flow-d.png)

La séquence ci-dessous est la capture du flux quand un utilisateur Azure se désabonne via le magasin en ligne de Microsoft :

![Un client se désabonne via le magasin en ligne de Microsoft](media/saas-metering-service-integration-flow-e.png)

## <a name="next-steps"></a>Étapes suivantes

[API de service de mesure de la Place de marché](./marketplace-metering-service-apis.md)
