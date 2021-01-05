---
title: Importer et exécuter des blocs-notes à partir d’un dépôt GitHub dans Azure Cosmos DB
description: Découvrez comment vous connecter à GitHub et importer les blocs-notes d’un dépôt GitHub dans votre compte Azure Cosmos. Après l’importation, vous pouvez les exécuter et les modifier, ainsi qu’enregistrer les modifications dans GitHub.
author: deborahc
ms.author: dech
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/19/2020
ms.openlocfilehash: 8315369d2100036a50aae770267aa04bceb2dfb0
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339799"
---
# <a name="import-notebooks-from-a-github-repo-into-azure-cosmos-db"></a>Importer des blocs-notes à partir d’un dépôt GitHub dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Après avoir [activé la prise en charge des bloc-notes](enable-notebooks.md) pour vos comptes Azure Cosmos, vous pouvez créer des blocs-notes, en charger de nouveaux à partir de votre ordinateur local, ou importer les blocs-notes existants à partir de vos comptes GitHub. Cet article explique comment connecter votre espace de travail de blocs-notes à GitHub et importer les blocs-notes d’un dépôt GitHub dans votre compte Azure Cosmos. Après l’importation, vous pouvez les exécuter et leur apporter des modifications, ainsi qu’enregistrer celles-ci sur GitHub.

## <a name="get-notebooks-from-github"></a>Obtenir des blocs-notes à partir de GitHub

Vous pouvez vous connecter à vos propres dépôts GitHub ou à d’autres dépôts GitHub publics pour lire, créer et partager des blocs-notes dans Azure Cosmos DB. Pour vous connecter à un compte GitHub, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez à votre compte Azure Cosmos.

1. Ouvrez le volet **Explorateur de données**. Cet onglet affiche l’ensemble de vos bases de données, conteneurs et blocs-notes existants.

1. Sélectionnez l’élément de menu **Se connecter à GitHub**.

1. Un onglet s’ouvre, dans lequel vous pouvez choisir de vous connecter uniquement aux **Dépôts publics** ou aux **Dépôts publics et privés**.  Après avoir choisi l’option requise, sélectionnez **Autoriser l’accès**. Une autorisation est requise pour qu’Azure Cosmos DB puisse accéder aux dépôts de votre compte GitHub.

   :::image type="content" source="./media/import-github-notebooks/authorize-access-github.png" alt-text="Autoriser Azure Cosmos DB à accéder à vos dépôts GitHub":::

1. Vous êtes redirigé vers la page web « github.com », dans laquelle vous pouvez confirmer l’autorisation. Sélectionnez le bouton **autoriser AzureCosmosDBNotebooks** , puis entrez le mot de passe de votre compte GitHub dans l’invite.

1. Une fois l’autorisation accordée, vous êtes redirigé vers votre compte Azure Cosmos. Vous pouvez alors voir tous les dépôts publics/privés de votre compte GitHub. Vous pouvez sélectionner un dépôt dans la liste disponible, ou en ajouter un directement à l’aide de son URL.

1. Une fois que vous avez sélectionné le dépôt requis, son entrée passe de la section des **dépôt désépinglés** à celle des **dépôts épinglés**. Si nécessaire, vous pouvez également choisir une branche spécifique de ce dépôt à partir de laquelle importer les blocs-notes.

   :::image type="content" source="./media/import-github-notebooks/choose-repo-branch.png" alt-text="Choisir un dépôt et une branche":::

1. Sélectionnez **OK** pour terminer l’opération d’importation. Tous les blocs-notes disponibles dans la branche sélectionnée de votre dépôt sont importés dans votre compte Azure Cosmos.

Une fois l’intégration avec un compte GitHub opérée, vous seul pouvez voir la liste des dépôts et blocs-notes de votre compte Azure Cosmos. Cela reste vrai même si plusieurs utilisateurs se connectent au compte Azure Cosmos DB et y ajoutent leurs propres comptes. Autrement dit, plusieurs utilisateurs peuvent utiliser le même compte Azure Cosmos pour connecter leur espace de travail de blocs-notes à GitHub. Toutefois, chaque utilisateur voit uniquement la liste des dépôts et blocs-notes qu’il a importés. Vous ne pouvez pas voir les blocs-notes importés par d’autres.

Pour déconnecter votre compte GitHub de l’espace de travail des blocs-notes, ouvrez l’onglet **Explorateur de données** , sélectionnez `…` en regard de **Dépôts GitHub** , puis sélectionnez **Se déconnecter de GitHub**.

## <a name="edit-a-notebook-and-push-changes-to-github"></a>Modifier un bloc-notes et envoyer (push) les modifications à GitHub

Vous pouvez modifier un bloc-notes existant dans le dépôt ou ajouté à celui-ci, puis enregistrer les modifications dans GitHub.

Après avoir modifié un bloc-notes existant, sélectionnez **Enregistrer**. Une boîte de dialogue s’ouvre, dans laquelle vous pouvez entrer le message de validation relatif aux modifications que vous avez apportées. Sélectionnez **Valider** pour mettre à jour le bloc-notes dans GitHub. Vous pouvez valider les mises à jour en vous connectant à votre compte GitHub, puis en vérifiant l’historique des validations.

Dans le flux GitHub normal, après avoir validé les modifications, vous expédiez (push) généralement les modifications à distance. Toutefois, dans ce cas, l’option Valider sert à mettre en lots, valider, puis pousser vos mises à jour vers GitHub.

:::image type="content" source="./media/import-github-notebooks/commit-changes-github.png" alt-text="Modifier des blocs-notes et valider les modifications sur GitHub":::

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les avantages des [blocs-notes Jupyter d’Azure Cosmos DB](cosmosdb-jupyter-notebooks.md).

