---
title: Restaurer en bloc des utilisateurs supprimés sur le portail Azure Active Directory | Microsoft Docs
description: Restaurer des utilisateurs supprimés en bloc dans le centre d’administration Azure AD dans Azure Active Directory
services: active-directory
author: curtand
ms.author: curtand
manager: mtillman
ms.date: 12/02/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: be3947e3de18f8ccaf47382c4035e187521ac710
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571502"
---
# <a name="bulk-restore-deleted-users-in-azure-active-directory"></a>Restaurer en bloc des utilisateurs supprimés dans Azure Active Directory

Azure Active Directory (Azure AD) prend en charge les opérations de restauration en bloc d’utilisateurs, ainsi que le téléchargement de listes d’utilisateurs, de groupes et de membres de groupes.

## <a name="understand-the-csv-template"></a>Comprendre le modèle CSV

Téléchargez et renseignez le modèle CSV pour vous aider à restaurer les utilisateurs Azure AD en bloc. Le modèle CSV que vous téléchargez peut se présenter comme dans l’exemple suivant :

![Feuille de calcul pour le chargement et les appels expliquant l’objectif et les valeurs de chaque ligne et colonne](./media/users-bulk-restore/understand-template.png)

### <a name="csv-template-structure"></a>Structure du modèle CSV

Les lignes d’un modèle CSV téléchargé sont les suivantes :

- **Numéro de version** : La première ligne contenant le numéro de version doit être incluse dans le fichier CSV chargé.
- **En-têtes de colonne** : Le format des en-têtes de colonne est &lt;*Nom d’élément*&gt; [PropertyName] &lt;*Obligatoire ou vide*&gt;. Par exemple : `Object ID [objectId] Required`. Certaines anciennes versions du modèle peuvent avoir de légères variations.
- **Exemples de lignes** : Nous avons inclus dans le modèle une ligne d’exemples de valeurs acceptables pour chaque colonne. Vous devez supprimer la ligne des exemples et la remplacer par vos propres entrées.

### <a name="additional-guidance"></a>Conseils supplémentaires

- Les deux premières lignes du modèle chargé ne doivent pas être supprimées ni modifiées, sinon, le chargement ne pourra pas être traité.
- Les colonnes obligatoires sont listées en premier.
- Nous vous déconseillons d’ajouter des colonnes au modèle. Les colonnes que vous ajouterez seront ignorées et ne seront pas traitées.
- Nous vous recommandons de télécharger la version la plus récente du modèle CSV aussi souvent que possible.

## <a name="to-bulk-restore-users"></a>Pour restaurer des utilisateurs en bloc

1. [Connectez-vous à votre organisation Azure AD](https://aad.portal.azure.com) avec un compte administrateur d’utilisateurs de celle-ci.
1. Dans Azure AD, sélectionnez **Utilisateurs** > **Supprimés**.
1. Dans la page **Utilisateurs supprimés**, sélectionnez **Restaurer en bloc** pour charger un fichier CSV valide de propriétés des utilisateurs à restaurer.

    ![Sélectionner la commande Restaurer en bloc dans la page Utilisateurs supprimés](./media/users-bulk-restore/bulk-restore.png)

1. Ouvrez le modèle CSV et ajoutez une ligne pour chaque utilisateur à restaurer. La seule valeur obligatoire est **ObjectID**. Puis enregistrez le fichier.

    :::image type="content" source="./media/users-bulk-restore/upload-button.png" alt-text="Sélectionner un fichier CSV local dans lequel vous répertoriez les utilisateurs à ajouter":::

1. Sur la page **Restauration en bloc**, sous **Charger votre fichier csv**, accédez au fichier. Quand vous sélectionnez le fichier et cliquez sur **Envoyer**, la validation du fichier CSV démarre.
1. Quand le contenu du fichier est validé, un message indique **Fichier chargé**. Si des erreurs sont présentes, vous devez les corriger avant de pouvoir envoyer le travail.
1. Lorsque votre fichier réussit la validation, sélectionnez **Envoyer** pour démarrer l’opération en bloc Azure qui restaure les utilisateurs.
1. Une fois l’opération de restauration terminée, vous recevez une notification indiquant que l’opération en bloc a réussi.

Si des erreurs se produisent, vous pouvez télécharger et consulter le fichier de résultats dans la page **Résultats de l’opération en bloc**. Le fichier contient la raison de chaque erreur.

## <a name="check-status"></a>Vérification du statut

Pour connaître l'état de toutes vos demandes d'opérations en bloc en attente, consultez la page **Résultats des opérations en bloc**.

[![Vérifier l'état sur la page Résultats des opérations en bloc.](./media/users-bulk-restore/bulk-center.png)](./media/users-bulk-restore/bulk-center.png#lightbox)

Ensuite, vous pouvez vérifier que les utilisateurs que vous avez restaurés existent bien au sein de l’organisation Azure AD via le portail Azure ou à l’aide de PowerShell.

## <a name="view-restored-users-in-the-azure-portal"></a>Afficher les utilisateurs restaurés sur le portail Azure

1. [Connectez-vous au centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte Administrateur d’utilisateurs de votre organisation.
1. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Utilisateurs**.
1. Sous **Afficher**, sélectionnez **Tous les utilisateurs**, puis vérifiez que les utilisateurs que vous avez restaurés sont répertoriés.

### <a name="view-users-with-powershell"></a>Afficher les utilisateurs avec PowerShell

Exécutez la commande suivante :

``` PowerShell
Get-AzureADUser -Filter "UserType eq 'Member'"
```

Les utilisateurs que vous avez restaurés devraient être répertoriés.

## <a name="next-steps"></a>Étapes suivantes

- [Importer des utilisateurs en bloc](users-bulk-add.md)
- [Supprimer des utilisateurs en bloc](users-bulk-delete.md)
- [Télécharger une liste d’utilisateurs](users-bulk-download.md)
