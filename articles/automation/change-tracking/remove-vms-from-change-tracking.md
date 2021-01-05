---
title: Supprimer des machines virtuelles de la fonctionnalité Suivi des modifications et inventaire d’Azure Automation
description: Cet article explique comment supprimer des machines virtuelles de la fonctionnalité Suivi des modifications et inventaire.
services: automation
ms.subservice: change-inventory-management
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 0b79fa22d3203504e63161aba03b32830d74d016
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93131273"
---
# <a name="remove-vms-from-change-tracking-and-inventory"></a>Supprimer des machines virtuelles de Change Tracking et Inventory

Une fois que vous avez terminé le suivi des modifications sur les machines virtuelles de votre environnement, vous pouvez cesser de les gérer avec la fonctionnalité [Suivi des modifications et inventaire](overview.md) . Pour arrêter leur gestion, vous allez modifier la requête de recherche enregistrée `MicrosoftDefaultComputerGroup` dans l’espace de travail Log Analytics qui est lié à votre compte Automation.

## <a name="sign-into-the-azure-portal"></a>Se connecter au portail Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="to-remove-your-vms"></a>Pour supprimer vos machines virtuelles

1. Dans le portail Azure, lancez **Cloud Shell** dans le volet de navigation en haut du portail. Si vous ne connaissez pas Azure Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../../cloud-shell/overview.md).

2. Utilisez la commande suivante pour identifier l’UUID d’une machine que vous souhaitez supprimer de la gestion.

    ```azurecli
    az vm show -g MyResourceGroup -n MyVm -d
    ```

3. Dans le portail Azure, accédez à **Espaces de travail Log Analytics**. Sélectionnez votre espace de travail dans la liste.

4. Dans votre espace de travail Log Analytics, sélectionnez **Journaux** puis, dans le menu d’actions en haut, choisissez **Explorateur de requêtes**.

5. Dans **Explorateur de requêtes** dans le volet de droite, développez **Requêtes enregistrées/Mises à jour** et sélectionnez la requête de recherche enregistrée `MicrosoftDefaultComputerGroup` pour la modifier.

6. Dans l’éditeur de requêtes, passez en revue la requête et recherchez l’UUID de la machine virtuelle. Supprimez l’UUID de la machine virtuelle et répétez les étapes pour toutes les machines virtuelles que vous souhaitez supprimer.

7. Sauvegardez la recherche enregistrée une fois que vous avez fini de la modifier en sélectionnant **Enregistrer** dans la barre supérieure.

>[!NOTE]
>Les machines s’affichent toujours une fois que vous les avez désinscrites, car nous effectuons des rapports pour toutes les machines évaluées au cours des dernières 24 heures. Une fois la machine supprimée, vous devez attendre 24 heures avant qu’elle ne soit plus listée.

## <a name="next-steps"></a>Étapes suivantes

Pour réactiver cette fonctionnalité, consultez [Activer Suivi des modifications et inventaire à partir d’un compte Automation](enable-from-automation-account.md), [Activer Suivi des modifications et inventaire à partir du portail Azure](enable-from-portal.md), [Activer Suivi des modifications et inventaire à partir d’un runbook](enable-from-runbook.md) ou [Activer Suivi des modifications et inventaire à partir d’une machine virtuelle Azure](enable-from-vm.md).