---
title: Vérifier votre facture d’abonnement Azure
description: Apprenez à comprendre votre facture et votre utilisation des ressources, et à vérifier les frais facturés pour votre abonnement Azure, y compris le paiement à l’utilisation.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: tutorial
ms.date: 10/26/2020
ms.author: banders
ms.custom: contperf-fy21q2
ms.openlocfilehash: ef04c964f0037fcf36fe376084df75ffa0034957
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027558"
---
# <a name="tutorial-review-your-individual-azure-subscription-bill"></a>Tutoriel : Vérifier votre facture d’abonnement Azure

Cet article vous aide à comprendre et à vérifier la facture de votre abonnement Azure avec paiement à l’utilisation ou Visual Studio. Pour chaque période de facturation, vous recevez normalement une facture par e-mail. Cette facture est une représentation de votre facture Azure. Les informations de coût figurant sur la facture sont disponibles dans le portail Azure. Dans ce tutoriel, vous allez comparer votre facture avec le fichier détaillé sur l’utilisation quotidienne et l’analyse des coûts disponible dans le portail Azure.

Ce tutoriel s’applique uniquement aux clients Azure disposant d’un abonnement individuel. Les abonnements individuels courants sont ceux avec un paiement à l’utilisation, achetés directement sur le site web Azure.

Pour obtenir de l’aide pour comprendre des frais inattendus, voir [Analyser les frais inattendus](analyze-unexpected-charges.md). Si vous devez annuler votre abonnement Azure, voir [Annulation de votre abonnement Azure](../manage/cancel-azure-subscription.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Comparer les frais facturés avec le fichier d’utilisation.
> * Comparer les frais et l’utilisation dans l’analyse des coûts.

## <a name="prerequisites"></a>Prérequis

Vous devez disposer d’un compte de facturation *Programme Microsoft Online Services*. Le compte est créé quand vous vous inscrivez à Azure par le biais du site web Azure. C’est par exemple le cas si vous disposez d’un compte avec des tarifs de paiement à l’utilisation ou si vous êtes un abonné Visual Studio.

Les factures pour les comptes gratuits Azure sont créées uniquement quand le montant du crédit mensuel est dépassé.

Vous devez être inscrit à Azure depuis plus de 30 jours. Azure vous facture à la fin de votre période de facturation.

## <a name="sign-in-to-azure"></a>Connexion à Azure

- Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="compare-billed-charges-with-your-usage-file"></a>Comparer les frais facturés et ceux dans votre fichier d’utilisation

<a name="charges"></a>

La première étape pour comparer l’utilisation et les coûts consiste à télécharger votre facture et vos fichiers d’utilisation. Le fichier CSV sur l’utilisation détaillée indique vos frais par période de facturation et l’utilisation quotidienne. Il n’inclut aucune information sur les taxes. Pour télécharger les fichiers, vous devez être administrateur de compte ou détenir le rôle de propriétaire.

Dans le portail Azure, tapez *abonnements* dans la zone de recherche, puis cliquez sur **Abonnements**.

[![Accéder aux abonnements](./media/review-individual-bill/navigate-subscriptions.png)](./media/review-individual-bill/navigate-subscriptions.png#lightbox)

Dans la liste des abonnements, cliquez sur l’abonnement concerné.

Sous **Facturation**, cliquez sur **Factures**.

Dans la liste des factures, recherchez celle que vous souhaitez télécharger, puis cliquez sur le symbole de téléchargement. Vous devrez peut-être changer l’intervalle de temps pour voir les factures les plus anciennes. La génération du fichier de détails d’utilisation et de la facture peut prendre quelques minutes.

![Capture d’écran qui montre les périodes de facturation, l’option de téléchargement et le total des frais pour chaque période de facturation](./media/review-individual-bill/download-invoice.png)

Dans la fenêtre Télécharger l’utilisation + les frais, cliquez sur **Télécharger le fichier csv** et **Télécharger la facture**.

![Capture d’écran qui montre la page Télécharger les données sur l’utilisation et la facturation](./media/review-individual-bill/usageandinvoice.png)

Si la page indique **Non disponible**, il peut y avoir plusieurs raisons pour lesquelles vous ne voyez pas les détails d’utilisation ou une facture :

- Cela fait moins de 30 jours que vous êtes inscrit à Azure.
- Il n’y a pas d’utilisation pour la période de facturation.
- Une facture n’a pas encore été générée. Patientez jusqu’à la fin de la période de facturation.
- Vous n’êtes pas autorisé à afficher les factures. Vous ne verrez peut-être pas les anciennes factures, sauf si vous êtes l’administrateur du compte.
- Si vous bénéficiez d’un essai gratuit ou avez un crédit mensuel avec votre abonnement que vous n’avez pas dépassé, vous n’obtenez pas de facture, sauf si vous avez un Contrat client Microsoft.

Ensuite, passez en revue les frais. Votre facture affiche des valeurs pour les taxes et vos frais d’utilisation.

![Exemple de facture Azure](./media/review-individual-bill/invoice-usage-charge.png)

Ouvrez le fichier d’utilisation CSV que vous avez téléchargé. À la fin du fichier, additionnez les valeurs de tous les éléments de la colonne *Cost*.

![Exemple de fichier d’utilisation avec coût total](./media/review-individual-bill/usage-file-usage-charges.png)

 La valeur additionnée de *Cost* doit correspondre précisément au coût de *frais d’utilisation* sur votre facture.

Vos frais d’utilisation sont affichés au niveau du compteur. Les termes suivants ont la même signification sur la facture et sur le fichier sur l’utilisation détaillée. Par exemple, le cycle de facturation de la facture est identique à la période de facturation indiquée dans le fichier relatif à l’utilisation détaillée.

| Facture (PDF) | Utilisation détaillée (CSV)|
| --- | --- |
|Cycle de facturation | BillingPeriodStartDate BillingPeriodEndDate |
|Nom |Catégorie du compteur |
|Type |Sous-catégorie du compteur |
|Ressource |MeterName |
|Région |MeterRegion |
|Consommé | Quantité |
|Inclus |Quantité incluse |
|Facturable |Quantité de dépassement |
|Tarif | EffectivePrice|
| Valeur | Coût |

La section **Frais d’utilisation** de votre facture affiche la valeur totale (coût) de chaque compteur consommé pendant la période de facturation. Par exemple, l’image suivante montre des frais d’utilisation pour le service Stockage Azure pour la ressource *P10 Disks*.

![Frais d’utilisation indiqués sur la facture](./media/review-individual-bill/invoice-usage-charges.png)

Dans votre fichier d’utilisation CSV, filtrez par *MeterName* pour la ressource correspondante indiquée sur votre facture. Ensuite, additionnez les valeurs *Cost* pour les éléments de la colonne. Voici un exemple qui se focalise sur le nom du compteur (disques P10) correspondant à la même ligne sur la facture.

![Valeur additionnée pour MeterName dans le fichier d’utilisation](./media/review-individual-bill/usage-file-usage-charge-resource.png)

La valeur additionnée de *Cost* doit correspondre précisément au coût de *frais d’utilisation* de la ressource individuelle facturé sur votre facture.

## <a name="compare-billed-charges-and-usage-in-cost-analysis"></a>Comparer les frais et l’utilisation facturés dans l’analyse des coûts

L’analyse des coûts dans le portail Azure peut également vous aider à vérifier vos frais. Pour obtenir un aperçu rapide de l’utilisation et des frais facturés, sélectionnez votre abonnement dans la page Abonnements du portail Azure. Ensuite, cliquez sur **Analyse des coûts** puis, dans la liste des vues, cliquez sur **Détails de la facture**.

![Exemple montrant la sélection des détails de la facture](./media/review-individual-bill/cost-analysis-select-invoice-details.png)

Ensuite, dans la liste de plage de dates, sélectionnez une période pour votre facture. Ajoutez un filtre pour le numéro de facture, puis sélectionnez la valeur InvoiceNumber qui correspond au numéro de votre facture. L’analyse des coûts affiche les détails des coûts de vos éléments facturés.

![Exemple montrant les détails des coûts facturés dans l’analyse des coûts](./media/review-individual-bill/cost-analysis-service-usage-charges.png)

Les coûts indiqués dans l’analyse des coûts doivent correspondre précisément au coût de *frais d’utilisation* de la ressource individuelle facturé sur votre facture.

![Frais d’utilisation indiqués sur la facture](./media/review-individual-bill/invoice-usage-charges.png)

## <a name="external-marketplace-services-are-billed-separately"></a>Les services externes de la Place de marché sont facturés séparément

<a name="external"></a>

Les services externes, ou frais de la Place de marché, concernent les ressources créées par des fournisseurs de logiciels tiers. Ces ressources sont disponibles sur la Place de marché Azure. Par exemple, un pare-feu Barracuda est une ressource de la Place de marché Azure offerte par un tiers. Tous les frais du pare-feu et les compteurs correspondants apparaissent en tant que frais de services externes.

Les frais de services externes sont facturés séparément. Les frais ne s’affichent pas sur votre facture Azure.

### <a name="resources-are-billed-by-usage-meters"></a>Les ressources sont facturées d’après les compteurs d’utilisation

Azure ne facture pas directement en fonction du coût de la ressource. Les frais d’une ressource sont calculés à l’aide d’un ou de plusieurs compteurs. Les compteurs permettent de suivre l’utilisation d’une ressource tout au long de sa durée de vie. Ces compteurs sont ensuite utilisés pour calculer la facture.

Quand vous créez une ressource Azure, par exemple une machine virtuelle, une ou plusieurs instances de compteur sont créées. Les compteurs sont utilisés pour suivre l’utilisation de la ressource au fil du temps. Chaque compteur émet des enregistrements d’utilisation qui sont ensuite utilisés par Azure pour calculer la facture.

Par exemple, les compteurs suivants peuvent être créés pour suivre l’utilisation d’une machine virtuelle créée dans Azure :

- Heures de calcul
- Heures d’adresse IP
- Transfert de données entrant
- Transfert de données sortant
- Disques managés Standard
- Opérations disque managées Standard
- E/S standard - Disque
- E/S standard - Lecture d’objet blob
- E/S standard - Écriture d’objet blob
- E/S standard - Suppression d’objet blob

Une fois la machine virtuelle créée, chaque compteur commence à émettre des enregistrements d’utilisation. Cette utilisation et le tarif du compteur font l’objet d’un suivi dans le système de comptabilisation d’Azure.

Vous pouvez voir les compteurs qui ont été utilisés pour calculer votre facture dans le fichier CSV d’utilisation, comme dans l’exemple précédent.

## <a name="pay-your-bill"></a>Régler votre facture

<a name="payment"></a>

Si vous avez configuré une carte de crédit comme moyen de paiement, le paiement est débité automatiquement dans les 10 jours qui suivent la fin de la période de facturation. Sur votre relevé de carte de crédit, le poste doit indiquer **MSFT Azure**.

Pour changer la carte de crédit utilisée, consultez [Ajouter, mettre à jour ou supprimer une carte de crédit pour Azure](../manage/change-credit-card.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Comparer les frais facturés avec le fichier d’utilisation.
> * Comparer les frais et l’utilisation dans l’analyse des coûts.

Suivez le guide de démarrage rapide pour commencer à utiliser l’analyse des coûts.

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts](../costs/quick-acm-cost-analysis.md)
