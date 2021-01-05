---
title: Plans et facturation
description: En savoir plus sur les plans et la facturation pour Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/18/2016
ms.openlocfilehash: e821036ff4ddb5a9786bc4f4537bb81539ab2c87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78898471"
---
# <a name="plans-and-billing-for-azure-scheduler"></a>Plans et facturation pour Azure Scheduler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

## <a name="job-collection-plans"></a>Plans de collections de travaux

Dans Azure Scheduler, une collection de travaux contient un nombre spécifique de travaux. La collection de travaux constitue l’entité facturable et elle est fournie dans des plans Standard, P10 Premium et P20 Premium. Ils sont décrits ci-dessous : 

| Plan de collections de travaux | Nombre max. de travaux par collection | Périodicité maximale | Nombre max. de collections de travaux par abonnement | limites | 
|:--- |:--- |:--- |:--- |:--- |
| **Standard** | 50 travaux par collection | Une fois par minute. Impossible d’exécuter les travaux plus souvent qu’une fois par minute. | Chaque abonnement Azure peut comporter jusqu'à 100 collections de travaux Standard. | Accès à l'ensemble complet des fonctionnalités de Scheduler | 
| **P10 Premium** | 50 travaux par collection | Une fois par minute. Impossible d’exécuter les travaux plus souvent qu’une fois par minute. | Chaque abonnement Azure peut comporter jusqu'à 10 000 collections de travaux P10 Premium. Pour plus de collections, <a href="mailto:wapteams@microsoft.com">contactez-nous</a>. | Accès à l'ensemble complet des fonctionnalités de Scheduler |
| **P20 Premium** | 1 000 travaux par collection | Une fois par minute. Impossible d’exécuter les travaux plus souvent qu’une fois par minute. | Chaque abonnement Azure peut comporter jusqu'à 5 000 collections de travaux P20 Premium. Pour plus de collections, <a href="mailto:wapteams@microsoft.com">contactez-nous</a>. | Accès à l'ensemble complet des fonctionnalités de Scheduler |
|||||| 

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur la tarification, consultez [Tarification de Scheduler](https://azure.microsoft.com/pricing/details/scheduler/).

## <a name="upgrade-or-downgrade-plans"></a>Mettre à niveau ou rétrograder des plans

À tout moment, vous pouvez mettre à niveau ou rétrograder un plan de collection de travaux dans le cadre des plans Standard, P10 Premium et P20 Premium.

## <a name="active-status-and-billing"></a>Facturation et état Actif

Les collections de travaux sont toujours actives, sauf si votre abonnement Azure entier passe temporairement à un état désactivé en raison de problèmes de facturation. Bien que vous puissiez désactiver tous les travaux d’une collection de travaux en une seule opération, cette action ne modifie pas l’état de facturation de la collection de travaux et celle-ci sera *encore* facturée. Les collections de travaux vides sont considérées actives et sont facturées.

Pour vous assurer qu'une collection de travaux n’est pas facturée, vous devez supprimer la collection de travaux.

## <a name="standard-billable-units"></a>Unités facturables standard

Une unité facturable standard peut inclure jusqu'à 10 collections de travaux Standard. Dans la mesure où une collection de travaux Standard peut contenir jusqu'à 50 travaux par collection, une unité de facturation standard permet à votre abonnement Azure de contenir jusqu'à 500 travaux, ou jusqu'à presque 22 *millions* d'exécutions de travaux par mois. Cette liste explique comment divers nombres de collections de travaux Standard vous sont facturés :

* Si vous avez entre 1 et 10 collections de travaux Standard, une seule unité de facturation standard vous est facturée. 

* Si vous avez entre 11 et 20 collections de travaux Standard, deux unités de facturation standard vous sont facturées. 

* Si vous avez entre 21 et 30 collections de travaux standard, trois unités de facturation standard vous sont facturées, etc.

## <a name="p10-premium-billable-units"></a>Unités facturables P10 premium

Une unité facturable P10 premium peut comporter jusqu'à 10 000 collections de travaux P10 Premium. Dans la mesure où une collection de travaux P10 Premium peut contenir jusqu'à 50 travaux par collection, une unité de facturation P10 premium permet à votre abonnement Azure de contenir jusqu'à 500 000 travaux ou jusqu'à presque 22 *milliards* d'exécutions de travaux par mois. 

Les collections de travaux P10 Premium fournissent les mêmes fonctionnalités que les collections de travaux Standard, mais elles fournissent une rupture des prix pour les applications qui nécessitent de nombreuses collections de travaux et elles fournissent une meilleure extensibilité. Cette liste explique comment divers nombres de collections de travaux P10 Premium vous sont facturés :

* Si vous avez entre 1 et 10 000 collections de travaux P10 Premium, une seule unité de facturation P10 premium vous est facturée. 

* SSi vous avez entre 10 001 et 20 000 collections de travaux P10 Premium, 2 unités de facturation P10 premium vous sont facturées, etc.

## <a name="p20-premium-billable-units"></a>Unités facturables P20 premium

Une unité facturable P20 premium peut comporter jusqu'à 5 000 collections de travaux P20 Premium. Dans la mesure où une collection de travaux P20 Premium peut contenir jusqu'à 1 000 travaux par collection de travaux, une unité de facturation P20 premium permet à votre abonnement Azure de contenir jusqu'à 5 000 000 travaux ou jusqu'à presque 220 *milliards* d'exécutions de travaux par mois.

Les collections de travaux P20 Premium proposent les mêmes fonctionnalités que les collections de travaux P10 Premium, mais elles prennent également en charge un plus grand nombre de travaux par collection, ainsi qu’un plus grand nombre total de travaux, ce qui favorise une plus grande évolutivité.

## <a name="plan-comparison"></a>Comparaison des plans

* Si vous avez plus de 100 collections de travaux Standard (10 unités de facturation standard), il est plus intéressant de regrouper toutes les collections de travaux dans un plan Premium.

* Si vous disposez d’une collection de travaux Standard et d’une collection de travaux Premium, une unité de facturation standard *et* une unité de facturation premium vous sont facturées.

  Le service Scheduler facture en fonction du nombre de collections de travaux actives qui sont standard ou premium.

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Limites, valeurs par défaut et codes d’erreur d’Azure Scheluler](scheduler-limits-defaults-errors.md)