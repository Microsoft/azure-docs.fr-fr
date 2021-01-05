---
title: 'FAQ sur les API du service de contrôle : Place de marché commerciale de Microsoft'
description: Foire aux questions sur les API de service de contrôle des offres SaaS dans Microsoft AppSource et la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 06/01/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: ddec23f695396b8322d51da62158a97456096ae8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87319998"
---
# <a name="marketplace-metered-billing-apis---faq"></a>API de facturation à la consommation de la Place de marché - FAQ

Quand un utilisateur s’abonne à un service SaaS ou Azure Application avec un plan Applications gérées qui comprend la facturation à la consommation, vous devez effectuer le suivi de la consommation pour chaque dimension de facturation utilisée.  Si la consommation dépasse les quantités incluses définies pour le terme sélectionné par le client, votre service émet des événements d’utilisation à Microsoft.

## <a name="for-both-saas-offers-and-managed-apps"></a>Pour les offres SaaS et les applications gérées

### <a name="how-often-is-it-expected-to-emit-usage"></a>À quelle fréquence est-il supposé émettre l’utilisation ?

Dans l’idéal, vous devez émettre l’utilisation toutes les heures pour la dernière heure, uniquement s’il existe une utilisation au cours de l’heure précédente.

### <a name="is-there-a-maximal-period-between-one-emission-and-the-next-one"></a>Existe-t-il une période maximale entre chaque émission ?

Cette limitation n'existe pas. Émettez uniquement l’utilisation lorsqu’elle se produit. Par exemple, si vous ne devez envoyer qu’une unité d’utilisation par durée de vie d’abonnement, vous pouvez le faire.

### <a name="what-is-the-maximum-delay-between-the-time-an-event-occurs-and-the-time-a-usage-event-is-emitted-to-microsoft"></a>Quel est le délai maximal entre le moment où un événement se produit et le moment où un événement d’utilisation est émis à Microsoft ?

Dans l’idéal, l’événement d’utilisation est émis toutes les heures pour les événements qui se sont produits au cours de la dernière heure. Toutefois, des retards sont attendus. Le délai maximal autorisé est de 24 heures, après quoi les événements d’utilisation ne sont pas acceptés. La meilleure pratique consiste à collecter l’utilisation horaire et à émettre comme un seul événement à la fin de l’heure.

Par exemple, si un événement d’utilisation se produit à 13 h dans une journée, vous avez jusqu’à 13 h le lendemain pour émettre un événement d’utilisation associé à cet événement.  Si le système émetteur de l’utilisation est arrêté, il peut récupérer puis envoyer l’événement d’utilisation pour l’intervalle d’heure au cours duquel l’utilisation a eu lieu, sans perte de fidélité.

Si 24 heures se sont écoulées après l’utilisation réelle, vous pouvez toujours émettre les unités consommées avec des événements d’utilisation ultérieurs.  Mais cette pratique peut nuire à la crédibilité des rapports d’événements de facturation pour le client final.  Nous vous recommandons d’éviter d’envoyer des émissions avec compteur une fois par jour/semaine/mois.  Il sera plus difficile de comprendre l’utilisation réelle d’un client et de résoudre les problèmes ou questions susceptibles d’être soulevés concernant les événements d’utilisation.

Une autre raison d’envoyer l’utilisation toutes les heures est d’éviter les situations dans lesquelles l’utilisateur annule l’abonnement avant que l’éditeur envoie l’événement d’émission quotidien/hebdomadaire/mensuel.

### <a name="what-happens-when-you-send-more-than-one-usage-event-in-the-same-hour"></a>Que se passe-t-il lorsque vous envoyez plusieurs événements d’utilisation dans la même heure ?

Un seul événement d’utilisation est accepté pour l’intervalle d’une heure. L’intervalle d’heure commence à la minute 0 et se termine à la minute 59.  Si plusieurs événements d’utilisation sont émis dans la même heure, tous les événements d’utilisation suivants sont abandonnés en tant que doublons.

### <a name="what-happens-when-the-customer-cancels-the-purchase-within-the-time-allowed-by-the-cancellation-policy"></a>Que se passe-t-il lorsque le client annule l’achat dans le délai autorisé par la politique d’annulation ?

Le montant forfaitaire ne sera pas facturé, mais le dépassement d’utilisation le sera.

### <a name="can-custom-meter-plans-be-used-for-one-time-payments"></a>Les plans avec compteur personnalisés peuvent-ils être utilisés pour les paiements ponctuels ?

Oui, vous pouvez définir une dimension personnalisée comme unité de paiement ponctuel et l’émettre une seule fois pour chaque client.

### <a name="can-custom-meter-plans-be-used-to-tiered-pricing-model"></a>Les plans avec compteur personnalisés peuvent-ils être utilisés pour hiérarchiser le modèle de tarification ?

Oui, ils peuvent être implémentés avec chaque dimension personnalisée représentant un niveau de prix tarifaire.

Par exemple, Contoso souhaite facturer 0,5 $ par e-mail pour les 1 000 premiers e-mails, 0,4 $ par e-mail entre 1 000 et 5 000 e-mails, et 0,2 $ par e-mail au-delà de 5 000 e-mails. Ils peuvent définir trois dimensions personnalisées correspondant aux trois niveaux de tarification par e-mail. Émettez les unités de la première dimension tant que le nombre d’e-mails reste en dessous de 1 000, puis les unités de la deuxième dimension lorsque le nombre d’e-mails est compris entre 1 000 et 5 000, et enfin, les unités de la troisième dimension au-delà de 5 000 e-mails.

### <a name="what-happens-if-the-marketplace-metering-service-has-an-outage"></a>Que se passe-t-il si le service de mesure de la Place de marché connaît une interruption ?

Si l’ISV envoie un compteur personnalisé et reçoit une erreur, peut-être provoquée par un problème côté Microsoft (généralement lorsque des événements similaires ont été acceptés auparavant sans erreur), l’ISV doit attendre et réessayer l’émission.

Si l’erreur persiste, ce compteur personnalisé doit être renvoyé l’heure suivante (quantité cumulée). Poursuivez ce processus jusqu’à ce que vous receviez une réponse sans erreur.

## <a name="for-saas-offers-only"></a>Pour les offres SaaS uniquement

### <a name="what-happens-when-you-emit-usage-for-a-saas-subscription-that-has-been-unsubscribed-already"></a>Que se passe-t-il lorsque vous émettez une utilisation pour un abonnement SaaS qui a déjà été annulé ?

Tout événement d’utilisation émis sur la plateforme de la Place du marché n’est pas accepté après la suppression d’un abonnement SaaS.

L’utilisation peut être émise uniquement pour les abonnements dont l’état indique Abonné (et non pour les abonnements à l’état `PendingFulfillmentStart`, `Suspended` ou `Unsubscribed`).

La seule exception est la création de rapports pour le temps écoulé avant l’annulation de l’abonnement SaaS.

Par exemple, le client a annulé l’abonnement SaaS aujourd’hui à 15h. Il est maintenant 17h et l’éditeur peut toujours émettre l’utilisation pour la période comprise entre 18h hier et 15h aujourd’hui, pour cet abonnement SaaS.

### <a name="can-you-get-a-list-of-all-saas-subscriptions-including-active-and-unsubscribed-subscriptions"></a>Pouvez-vous obtenir une liste de tous les abonnements SaaS, y compris les abonnements actifs et désabonnés ?

Oui, lorsque vous appelez l’[API Obtenir la liste des abonnements](pc-saas-fulfillment-api-v2.md#subscription-apis), car cela inclut une liste de tous les abonnements SaaS. Le champ d’état dans la réponse pour chaque abonnement SaaS indique si l’abonnement est actif ou non.

### <a name="are-the-start-and-end-dates-of-saas-subscription-term-and-overage-usage-emission-connected"></a>Les dates de début et de fin de la période d’abonnement SaaS et l’émission de dépassement d’utilisation sont-elles connectées ?

Les événements de dépassement peuvent être émis à tout moment pour l’abonnement SaaS existant à l’état *Abonné*. Il incombe à l’éditeur d’émettre des événements d’utilisation en fonction de la stratégie définie dans le plan de facturation. Le dépassement doit être calculé en fonction des dates définies dans le terme de l’abonnement SaaS. 

Par exemple, si l’éditeur définit un plan SaaS qui inclut 1 000 e-mails pour 100 $ à un tarif forfaitaire mensuel, chaque e-mail au-dessus de 1 000 est facturé 1 $ via une dimension personnalisée.

Si le client achète et active l’abonnement le 6 janvier, les 1 000 e-mails inclus dans le tarif forfaitaire seront comptabilisés à partir de cette date. Ainsi, si jusqu’au 5 février (fin du premier mois de l’abonnement), seuls 900 e-mails ont été envoyés, le client ne paiera le tarif forfaitaire que pour le premier mois de cet abonnement, et aucun événement d’utilisation de dépassement ne sera émis par l’éditeur entre le 6 janvier et le 5 février. Le 6 février, l’abonnement sera automatiquement renouvelé et le décompte sera réinitialisé. Si, le 15 février, le client a atteint 1 000 e-mails envoyés, le reste des e-mails envoyés jusqu’au 5 mars sera facturé comme un dépassement (1 $ par e-mail) en fonction des événements de dépassement d’utilisation émis par l’éditeur.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations, consultez [API de service de mesure de la Place de marché](./marketplace-metering-service-apis.md).
