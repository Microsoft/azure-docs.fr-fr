---
title: Vue d’ensemble des modèles courants de mise à l’échelle automatique
description: Découvrez certains des modèles courants de mise à l’échelle automatique de vos ressources dans Azure.
ms.topic: conceptual
ms.date: 05/07/2017
ms.subservice: autoscale
ms.openlocfilehash: adbfecd05bfd4330e7c7c972cfdb9fe7021eec31
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87073495"
---
# <a name="overview-of-common-autoscale-patterns"></a>Vue d’ensemble des modèles courants de mise à l’échelle automatique
Cet article décrit certains des modèles courants de mise à l’échelle de vos ressources dans Azure.

La mise à l’échelle automatique Azure Monitor s’applique uniquement aux [groupes de machines virtuelles identiques](https://azure.microsoft.com/services/virtual-machine-scale-sets/), aux [services cloud](https://azure.microsoft.com/services/cloud-services/), à [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) et aux [services de gestion des API](../../api-management/api-management-key-concepts.md).

## <a name="lets-get-started"></a>Prise en main

Cet article suppose que vous êtes familiarisé avec la mise à l’échelle automatique. Vous pouvez [entamer la mise à l'échelle de votre ressource ici][1]. Voici quelques modèles de mise à l’échelle courants.

## <a name="scale-based-on-cpu"></a>Mise à l’échelle en fonction du processeur

Vous avez une application web (/VMSS/rôle de service cloud) et

- Vous souhaitez effectuer un scale-out/scale-in sur la base du processeur.
- Vous souhaitez en outre vous assurer qu’il existe un nombre minimal d’instances.
- Vous souhaitez aussi vous assurer que vous définissez une limite maximale pour le nombre d’instances, vers lequel vous pouvez faire évoluer.

![Mise à l’échelle en fonction du processeur][2]

## <a name="scale-differently-on-weekdays-vs-weekends"></a>Mettre à l’échelle différemment durant les week-ends et jours de la semaine

Vous avez une application web (/VMSS/rôle de service cloud) et

- Vous voulez 3 instances par défaut (pour les jours de la semaine)
- Vous ne prévoyez le trafic lors des week-ends et, par conséquent, vous souhaitez réduire de 1 instance les week-ends.

![Mettre à l’échelle différemment durant les week-ends et jours de la semaine][3]

## <a name="scale-differently-during-holidays"></a>Mettre à l’échelle différemment pendant les jours fériés

Vous avez une application web (/VMSS/rôle de service cloud) et

- Vous souhaitez augmenter/diminuer la taille des instances en fonction de l’utilisation du processeur par défaut
- Toutefois, pendant les vacances (ou des jours spécifiques qui sont importants pour votre entreprise), vous souhaitez remplacer les valeurs par défaut et avoir plus de capacité à votre disposition.

![Mettre à l’échelle différemment lors des jours fériés][4]

## <a name="scale-based-on-custom-metric"></a>Mise à l’échelle en fonction de métriques personnalisées

Vous avez un site web frontal et un niveau d’API qui communique avec le serveur principal.

- Vous souhaitez mettre à l'échelle le niveau de l'API en fonction d'événements personnalisés dans le front-end (exemple : vous souhaitez mettre à l'échelle votre processus de paiement en fonction du nombre d'articles contenus dans le panier d'achat).

![Mise à l’échelle en fonction de métriques personnalisées][5]

<!--Reference-->
[1]: ./autoscale-get-started.md
[2]: ./media/autoscale-common-scale-patterns/scale-based-on-cpu.png
[3]: ./media/autoscale-common-scale-patterns/weekday-weekend-scale.png
[4]: ./media/autoscale-common-scale-patterns/holidays-scale.png
[5]: ./media/autoscale-common-scale-patterns/custom-metric-scale.png
