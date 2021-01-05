---
title: Supprimer la fonctionnalité Suivi des modifications et inventaire d’Azure Automation
description: Cet article explique comment arrêter d’utiliser Suivi des modifications et inventaire et dissocier un compte Automation de l’espace de travail Log Analytics.
services: automation
ms.subservice: change-inventory-management
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 2e3e5abdfbb2bf2e9d7a12a677422adc67336775
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209130"
---
# <a name="remove-change-tracking-and-inventory-from-automation-account"></a>Supprimer Suivi des modifications et inventaire à partir d’un compte Automation

Après avoir activé la gestion de vos machines virtuelles à l’aide d’Azure Automation Suivi des modifications et inventaire, vous pouvez décider de ne plus l’utiliser et de supprimer la configuration du compte et de l’espace de travail Log Analytics lié. Cet article explique comment supprimer complètement Suivi des modifications et inventaire des machines virtuelles gérées, de votre compte Automation et de l’espace de travail Log Analytics.

## <a name="sign-into-the-azure-portal"></a>Se connecter au portail Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="remove-management-of-vms"></a>Supprimer la gestion des machines virtuelles

Avant de supprimer Suivi des modifications et inventaire, vous devez arrêter la gestion de vos machines virtuelles. Consultez [Supprimer des machines virtuelles du Suivi des modifications](remove-vms-from-change-tracking.md) pour les désinscrire de cette fonctionnalité.

## <a name="remove-changetracking-solution"></a>Supprimer la solution changeTracking

Pour pouvoir dissocier le compte Automation de l’espace de travail, vous devez effectuer les étapes suivantes afin de supprimer complètement Suivi des modifications et inventaire. Vous allez supprimer la solution **ChangeTracking** de l’espace de travail.

1. Dans le portail Azure, sélectionnez **Tous les services** . Dans la liste de ressources, saisissez **Log Analytics** . Au fur et à mesure de la saisie, la liste filtre les suggestions. Sélectionnez **Log Analytics** .

2. Dans votre liste d’espaces de travail Log Analytics, sélectionnez celui que vous avez choisi quand vous avez activé Suivi des modifications et inventaire.

3. Sur la gauche, sélectionnez **Solutions** .  

4. Dans la liste des solutions, sélectionnez **ChangeTracking(nom de l’espace de travail)** . Dans la page **Vue d’ensemble** pour la solution, sélectionnez **Supprimer** . À l’invite de confirmation, sélectionnez **Oui** .

## <a name="unlink-workspace-from-automation-account"></a>Dissocier un espace de travail d’un compte Automation

1. Dans le portail Azure, sélectionnez **Comptes Automation** .

2. Accédez à votre compte Automation et sélectionnez **Espace de travail lié** sous **Ressources associées** sur la gauche.

3. Dans la page **Dissocier l’espace de travail** , sélectionnez **Dissocier l’espace de travail** et répondez aux invites.

   ![Page Dissocier l’espace de travail](media/remove-feature/automation-unlink-workspace-blade.png)

Vous pouvez suivre la progression de la dissociation de l’espace de travail Log Analytics en sélectionnant **Notifications** dans le menu.

Vous pouvez également dissocier votre espace de travail Log Analytics de votre compte Automation à partir de l’espace de travail :

1. Dans le portail Azure, sélectionnez **Log Analytics** .

2. Dans l’espace de travail, sélectionnez **Compte Automation** sous **Ressources connexes** .

3. Sur la page du compte Automation, sélectionnez **Dissocier le compte** .

Vous pouvez suivre la progression de la dissociation du compte Automation en sélectionnant **Notifications** dans le menu.

## <a name="next-steps"></a>Étapes suivantes

Pour réactiver cette fonctionnalité, consultez [Activer Suivi des modifications et inventaire à partir d’un compte Automation](enable-from-automation-account.md), [Activer Suivi des modifications et inventaire à partir du portail Azure](enable-from-portal.md), [Activer Suivi des modifications et inventaire à partir d’un runbook](enable-from-runbook.md) ou [Activer Suivi des modifications et inventaire à partir d’une machine virtuelle Azure](enable-from-vm.md).
