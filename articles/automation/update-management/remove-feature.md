---
title: Supprimer la fonctionnalité Azure Automation Update Management
description: Cet article explique comment arrêter d’utiliser Update Management et dissocier un compte Automation de l’espace de travail Log Analytics.
services: automation
ms.date: 07/28/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 1d83f859fce33b9499d01c4b58e69f56fdbbb293
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221567"
---
# <a name="remove-update-management-from-automation-account"></a>Supprimer Update Management d’un compte Automation

Après avoir activé la gestion des mises à jour sur vos machines virtuelles à l’aide d’Azure Automation Update Management, vous pouvez décider de ne plus l’utiliser et de supprimer la configuration du compte et de l’espace de travail Log Analytics lié.  Cet article explique comment supprimer complètement Update Management des machines virtuelles gérées, de votre compte Automation et de l’espace de travail Log Analytics.

## <a name="sign-into-the-azure-portal"></a>Se connecter au portail Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="remove-management-of-vms"></a>Supprimer la gestion des machines virtuelles

Avant de supprimer Update Management, vous devez arrêter la gestion de vos machines virtuelles. Consultez [Supprimer des machines virtuelles d’Update Management](remove-vms.md) pour les désinscrire de cette fonctionnalité.

## <a name="remove-updatemanagement-solution"></a>Supprimer la solution UpdateManagement

Pour pouvoir dissocier le compte Automation de l’espace de travail, vous devez effectuer les étapes suivantes afin de supprimer complètement Update Management. Vous allez supprimer la solution **Mises à jour** de l’espace de travail.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste filtre les suggestions. Sélectionnez **Log Analytics**.

3. Dans votre liste d’espaces de travail Log Analytics, sélectionnez celui que vous avez choisi quand vous avez activé Update Management.

4. Sur la gauche, sélectionnez **Solutions**.  

5. Dans la liste des solutions, sélectionnez **Mises à jour(nom de l’espace de travail)** . Dans la page **Vue d’ensemble** pour la solution, sélectionnez **Supprimer**. À l’invite de confirmation, sélectionnez **Oui**.

## <a name="unlink-workspace-from-automation-account"></a>Dissocier un espace de travail d’un compte Automation

1. Dans le portail Azure, sélectionnez **Comptes Automation**.

2. Accédez à votre compte Automation et sélectionnez **Espace de travail lié** sous **Ressources associées** sur la gauche.

3. Dans la page **Dissocier l’espace de travail**, sélectionnez **Dissocier l’espace de travail** et répondez aux invites.

   ![Page Dissocier l’espace de travail](media/remove-feature/automation-unlink-workspace-blade.png)

Vous pouvez suivre la progression de la dissociation de l’espace de travail Log Analytics en sélectionnant **Notifications** dans le menu.

Vous pouvez également dissocier votre espace de travail Log Analytics de votre compte Automation à partir de l’espace de travail :

1. Dans le portail Azure, sélectionnez **Log Analytics**.

2. Dans l’espace de travail, sélectionnez **Compte Automation** sous **Ressources connexes**.

3. Sur la page du compte Automation, sélectionnez **Dissocier le compte**.

Vous pouvez suivre la progression de la dissociation du compte Automation en sélectionnant **Notifications** dans le menu.

## <a name="cleanup-automation-account"></a>Nettoyer le compte Automation

Si Update Management a été configuré pour prendre en charge des versions antérieures de la solution de supervision SQL Azure, le programme d’installation de la fonctionnalité a peut-être créé des ressources Automation que vous devez supprimer. Pour Update Management, vous pouvez supprimer les éléments suivants qui ne sont plus nécessaires :

   * Planifications des mises à jour : chacune a un nom qui correspond au déploiement de mise à jour que vous avez créé.
   * Groupes Worker hybrides créés pour Update Management : chacun porte un nom similaire à *machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8)* .

## <a name="next-steps"></a>Étapes suivantes

Pour réactiver cette fonctionnalité, consultez [Activer Update Management à partir d’un compte Automation](enable-from-automation-account.md), [Activer Update Management à partir du portail Azure](enable-from-portal.md), [Activer Update Management à partir d’un runbook](enable-from-runbook.md) ou [Activer Update Management à partir d’une machine virtuelle Azure](enable-from-vm.md).
