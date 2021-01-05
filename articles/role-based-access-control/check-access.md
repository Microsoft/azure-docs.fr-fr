---
title: Démarrage rapide – Vérifier l’accès d’un utilisateur aux ressources Azure – RBAC Azure
description: Dans ce guide de démarrage rapide, vous découvrez comment vérifier votre accès ou celui d’un autre utilisateur aux ressources Azure avec le portail Azure et le contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: role-based-access-control
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: quickstart
ms.workload: identity
ms.date: 12/09/2020
ms.author: rolyon
ms.custom: contperf-fy21q2
ms.openlocfilehash: 7cf4020ad38224b25ea8bb7dc7f0fdea7dd6f3b1
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97034001"
---
# <a name="quickstart-check-access-for-a-user-to-azure-resources"></a>Démarrage rapide : Vérifier l’accès d’un utilisateur aux ressources Azure

Parfois, vous devez vérifier l’accès dont dispose un utilisateur à un ensemble de ressources Azure. Vous vérifiez son accès en listant ses affectations. Un moyen rapide de vérifier l’accès d’un seul utilisateur consiste à utiliser la fonctionnalité **Vérifier l’accès** dans la page **Contrôle d’accès (IAM)** .

## <a name="step-1-open-the-azure-resources"></a>Étape 1 : Ouvrir les ressources Azure

Pour vérifier l’accès d’un utilisateur, vous devez d’abord ouvrir les ressources Azure pour lesquelles vous souhaitez vérifier l’accès. Les ressources Azure sont organisées en niveaux, qui sont généralement appelés l’*étendue*. Dans Azure, vous pouvez spécifier une étendue à quatre niveaux, allant du plus large au plus petit : groupe d’administration, abonnement, groupe de ressources et ressource.

![Niveaux de l’étendue de RBAC Azure](../../includes/role-based-access-control/media/scope-levels.png)

Suivez ces étapes pour ouvrir l’ensemble des ressources Azure pour lesquelles vous souhaitez vérifier l’accès.

1. Ouvrez le [portail Azure](https://portal.azure.com).

1. Ouvrez l’ensemble de ressources Azure, comme **Groupes d’administration**, **Abonnements**, **Groupes de ressources** ou une ressource particulière.

1. Cliquez sur la ressource spécifique dans cette étendue.

    Voici l’illustration d’un exemple de groupe de ressources.

    ![Vue d’ensemble du groupe de ressources](./media/check-access/rg-overview.png)

## <a name="step-2-check-access-for-a-user"></a>Étape 2 : Vérifier l’accès pour un utilisateur

Suivez ces étapes pour vérifier l’accès d’un utilisateur, groupe, principal de service ou d’une identité managée aux ressources Azure précédemment sélectionnées.

1. Cliquez sur **Contrôle d’accès (IAM)**.

    Voici un exemple de la page Contrôle d’accès (IAM) pour un groupe de ressources.

    ![Contrôle d’accès du groupe de ressources – Onglet Vérifier l’accès](./media/check-access/rg-access-control.png)

1. Sous l’onglet **Vérifier l’accès**, dans la liste **Rechercher**, sélectionnez l’utilisateur, le groupe, le principal de service ou l’identité managée dont vous souhaitez vérifier l’accès.

1. Dans la zone de recherche, entrez une chaîne afin de rechercher, dans le répertoire, des noms d’affichage, des adresses e-mail ou des identificateurs d’objet.

    ![Liste de sélection Vérifier l’accès](./media/shared/rg-check-access-select.png)

1. Cliquez sur le principal de sécurité pour ouvrir le volet **Affectations**.

    Dans ce volet, vous pouvez voir l’accès du principal de sécurité sélectionné pour cette étendue et celui hérité de cette étendue. Les affectations pour les étendues enfants ne sont pas listées. Les affectations suivantes s’affichent :

    - Attributions de rôle ajoutées avec RBAC Azure.
    - Affectations de refus ajoutées en utilisant Azure Blueprints ou des applications managées Azure.
    - Affectations de l’administrateur de service classique ou de coadministrateurs pour les déploiements classiques. 

    ![Volet des affectations de rôle et de refus pour un utilisateur](./media/shared/rg-check-access-assignments-user.png)

## <a name="step-3-check-your-access"></a>Étape 3 : Vérifier votre accès

Suivez ces étapes pour vérifier votre accès aux ressources Azure précédemment sélectionnées.

1. Cliquez sur **Contrôle d’accès (IAM)**.

1. Sous l’onglet **Vérifier l’accès**, cliquez sur le bouton **Afficher mon accès**.

    Un volet des affectations s’affiche, dans lequel est listé votre accès pour cette étendue et celui hérité de cette étendue. Les affectations pour les étendues enfants ne sont pas listées.

    ![Volet des affectations de rôle et de refus](./media/check-access/rg-check-access-assignments.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Répertorier les attributions de rôle Azure à l’aide du portail Azure](role-assignments-list-portal.md)
