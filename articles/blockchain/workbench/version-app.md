---
title: Gestion des versions des applications blockchain - Azure Blockchain Workbench
description: Comment utiliser les versions des applications dans Azure Blockchain Workbench Preview.
ms.date: 11/20/2019
ms.topic: how-to
ms.reviewer: brendal
ms.openlocfilehash: 2a70112fd0ab6e2f664ca48265c121936b01e58b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85209876"
---
# <a name="azure-blockchain-workbench-preview-application-versioning"></a>Versions des applications Azure Blockchain Workbench Preview

Vous pouvez créer et utiliser plusieurs versions d’une application Azure Blockchain Workbench Preview. Si plusieurs versions de la même application sont chargées, un historique des versions est disponible et permet aux utilisateurs de choisir la version qu’ils veulent utiliser.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* Un déploiement Blockchain Workbench. Pour plus de détails sur le déploiement, consultez [Azure Blockchain Workbench deployment](deploy.md) (Déploiement d’Azure Blockchain Workbench).
* Une application blockchain déployée dans Blockchain Workbench. Voir [Créer une application blockchain dans Azure Blockchain Workbench](create-app.md)

## <a name="add-an-app-version"></a>Ajouter une version d’application

Pour ajouter une nouvelle version, chargez la nouvelle configuration et les fichiers de contrat intelligent sur Blockchain Workbench.

1. Dans un navigateur web, accédez à l’adresse web de Blockchain Workbench. Par exemple : `https://{workbench URL}.azurewebsites.net/`. Pour plus d’informations sur la façon de trouver l’adresse web de Blockchain Workbench, consultez [URL web de Blockchain Workbench](deploy.md#blockchain-workbench-web-url)
2. Connectez-vous en tant [qu’administrateur Blockchain Workbench](manage-users.md#manage-blockchain-workbench-administrators).
3. Sélectionnez l’application de blockchain que vous voulez mettre à jour avec une autre version.
4. Sélectionnez **Ajouter une version**. Le volet **Ajouter une version** s’affiche.
5. Sélectionnez la configuration du contrat et les fichiers de code du contrat de la nouvelle version à charger. Le fichier de configuration est automatiquement validé. Corrigez les erreurs de validation avant de déployer l’application.
6. Sélectionnez **Ajouter une version** pour ajouter la nouvelle version de l’application blockchain.

    ![Ajouter une nouvelle version](media/version-app/add-version.png)

Le déploiement de l’application blockchain peut prendre quelques minutes. Une fois le déploiement terminé, actualisez la page de l’application. Le fait de choisir l’application et de sélectionner le bouton **Historique des versions** affiche l’historique des versions de l’application.

> [!IMPORTANT]
> Les versions précédentes de l’application sont désactivées. Vous pouvez réactiver des versions antérieures individuellement.
>
> Il peut être nécessaire d’ajouter à nouveau des membres aux rôles d’application si des modifications ont été apportées aux rôles de l’application dans la nouvelle version.

## <a name="using-app-versions"></a>Utilisation des versions d’une application

Par défaut, la dernière version activée de l’application est utilisée dans Blockchain Workbench. Si vous voulez utiliser une version antérieure d’une application, vous devez d’abord choisir la version dans la page de l’application.

1. Dans la section des applications Blockchain Workbench, cochez la case correspondant à l’application qui contient le contrat que vous voulez utiliser. Si des versions antérieures sont activées, le bouton Historique des versions est disponible.
2. Sélectionnez le bouton **Historique des versions**.
3. Dans le volet d’historique des versions, choisissez la version de l’application en sélectionnant le lien dans la colonne *Date de modification*.

    ![Choisir une version antérieure](media/version-app/use-version.png)

    Vous pouvez créer des contrats ou effectuer des actions sur les contrats d’une version antérieure. La version de l’application s’affiche après le nom de l’application, et un avertissement s’affiche à propos de la version plus ancienne.

## <a name="next-steps"></a>Étapes suivantes

* [Résolution des problèmes d’Azure Blockchain Workbench](troubleshooting.md)
