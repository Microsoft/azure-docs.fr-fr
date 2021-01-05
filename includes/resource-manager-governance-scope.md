---
title: Fichier Include
description: Fichier Include
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/21/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 14ac8ff54f5627509fe10a90bfdd0a01c07762d9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028499"
---
Avant de créer des éléments, passons en revue le concept de portée. Azure fournit quatre niveaux de gestion : des groupes d’administration, d’abonnement, des groupes de ressources et des ressources. Les [groupes d’administration](../articles/governance/management-groups/overview.md) sont en préversion. L’image suivante représente un exemple de ces couches.

![Étendue](./media/resource-manager-governance-scope/scope-levels.png)

Vous appliquez les paramètres de gestion à tous ces niveaux de l’étendue. Le niveau que vous sélectionnez détermine à quel point le paramètre est appliqué. Les niveaux inférieurs héritent des paramètres des niveaux supérieurs. Lorsque vous appliquez un paramètre à l’abonnement, ce paramètre est appliqué à tous les groupes de ressources et les ressources de votre abonnement. Lorsque vous appliquez un paramètre sur le groupe de ressources, ce paramètre est appliqué sur le groupe de ressources et toutes ses ressources. Toutefois, un autre groupe de ressources ne dispose pas de ce paramètre.

En règle générale, il est judicieux d’appliquer les paramètres critiques à des niveaux supérieurs et les exigences spécifiques au projet à des niveaux inférieurs. Par exemple, vous voudrez vous assurer que toutes les ressources de votre organisation sont déployées dans certaines régions. Pour satisfaire cette exigence, appliquez une stratégie à l’abonnement qui spécifie les emplacements autorisés. Lorsque les autres utilisateurs de votre organisation ajouteront de nouveaux groupes de ressources et des ressources, les emplacements autorisés seront automatiquement appliqués.