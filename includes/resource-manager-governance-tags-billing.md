---
title: Fichier Include
description: Fichier Include
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/19/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 226079b1970508004c55c54cd3de6d2e9475aa08
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75970125"
---
Après avoir appliqué des balises aux ressources, vous pouvez afficher les coûts des ressources à l’aide de ces balises. L’analyse des coûts prend un certain temps pour afficher les dernières données d’utilisation. Il est donc possible que vous ne voyiez pas encore les coûts. Lorsqu’ils sont disponibles, vous pouvez afficher les coûts des ressources des différents groupes de ressources de votre abonnement. Pour voir les coûts, les utilisateurs doivent avoir un [accès de niveau d’abonnement aux informations de facturation](../articles/cost-management-billing/manage/manage-billing-access.md).

Pour afficher les coûts par balise dans le portail, sélectionnez votre abonnement, puis sélectionnez **Analyse du coût**.

![Analyse des coûts](./media/resource-manager-governance-tags-billing/select-cost-analysis.png)

Ensuite, filtrez par valeur de balise, puis sélectionnez **Appliquer**.

![Afficher les coûts par balise](./media/resource-manager-governance-tags-billing/view-costs-by-tag.png)

Vous pouvez également utiliser les [API Facturation Azure](../articles/cost-management-billing/manage/usage-rate-card-overview.md) pour afficher les coûts par programmation.
