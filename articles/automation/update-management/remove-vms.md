---
title: Supprimer des machines virtuelles d’Azure Automation Update Management
description: Cet article explique comment supprimer des machines gérées avec Update Management.
services: automation
ms.topic: conceptual
ms.date: 09/09/2020
ms.custom: mvc
ms.openlocfilehash: 774dbe29cbb6b4d063d3619d0c710efb1949b99a
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221564"
---
# <a name="remove-vms-from-update-management"></a>Supprimer des machines virtuelles d’Update Management

Lorsque vous avez terminé de gérer les mises à jour sur vos machines virtuelles dans votre environnement, vous pouvez arrêter de gérer les machines virtuelles grâce à la fonctionnalité [Update Management](overview.md). Pour arrêter leur gestion, vous allez modifier la requête de recherche enregistrée `MicrosoftDefaultComputerGroup` dans l’espace de travail Log Analytics qui est lié à votre compte Automation.

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

Pour réactiver la gestion de votre machine virtuelle, consultez [Activer Update Management à partir du portail Azure](enable-from-portal.md) ou [Activer Update Management à partir d’une machine virtuelle Azure](enable-from-vm.md).