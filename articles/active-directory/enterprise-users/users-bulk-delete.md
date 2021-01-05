---
title: Supprimer des utilisateurs en bloc dans le portail Azure Active Directory | Microsoft Docs
description: Supprimer des utilisateurs en bloc dans le centre d’administration Azure dans Azure Active Directory
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 12/02/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: b57d675003c1ebeb29927b86338f95cf7dd68090
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574461"
---
# <a name="bulk-delete-users-in-azure-active-directory"></a>Supprimer des utilisateurs en bloc dans Azure Active Directory.

En utilisant le portail Azure Active Directory (Azure AD), vous pouvez supprimer un grand nombre de membres d’un groupe en utilisant un fichier de valeurs séparées par des virgules (CSV) pour supprimer en bloc les utilisateurs.

## <a name="understand-the-csv-template"></a>Comprendre le modèle CSV

Téléchargez et renseignez le modèle CSV pour vous aider à supprimer les utilisateurs Azure AD en bloc. Le modèle CSV que vous téléchargez peut se présenter comme dans l’exemple suivant :

![Feuille de calcul pour le chargement et les appels expliquant l’objectif et les valeurs de chaque ligne et colonne](./media/users-bulk-delete/understand-template.png)

### <a name="csv-template-structure"></a>Structure du modèle CSV

Les lignes d’un modèle CSV téléchargé sont les suivantes :

- **Numéro de version** : La première ligne contenant le numéro de version doit être incluse dans le fichier CSV chargé.
- **En-têtes de colonne** : Le format des en-têtes de colonne est &lt;*Nom d’élément*&gt; [PropertyName] &lt;*Obligatoire ou vide*&gt;. Par exemple : `User name [userPrincipalName] Required`. Certaines anciennes versions du modèle peuvent avoir de légères variations.
- **Exemples de lignes** : Nous avons inclus dans le modèle une ligne d’exemples de valeurs acceptables pour chaque colonne. Vous devez supprimer la ligne des exemples et la remplacer par vos propres entrées.

### <a name="additional-guidance"></a>Conseils supplémentaires

- Les deux premières lignes du modèle chargé ne doivent pas être supprimées ou modifiées, ou le chargement ne pourra pas être traité.
- Les colonnes requises sont répertoriées en premier.
- Nous vous déconseillons d’ajouter de nouvelles colonnes au modèle. Toutes les colonnes supplémentaires que vous ajoutez sont ignorées et ne sont pas traitées.
- Nous vous recommandons de télécharger la version la plus récente du modèle CSV aussi souvent que possible.

## <a name="to-bulk-delete-users"></a>Pour supprimer des utilisateurs en bloc

1. [Connectez-vous à votre organisation Azure AD](https://aad.portal.azure.com) avec un compte Administrateur d’utilisateurs de celle-ci.
1. Dans Azure AD, sélectionnez **Utilisateurs** > **Supprimer en bloc**.
1. Dans la page **Supprimer des utilisateurs en bloc**, sélectionnez **Télécharger** pour recevoir un fichier CSV de propriétés utilisateur valide.

   ![Sélectionner un fichier CSV local dans lequel vous répertoriez les utilisateurs à supprimer](./media/users-bulk-delete/bulk-delete.png)

1. Ouvrez le fichier CSV et ajoutez une ligne pour chaque utilisateur à supprimer. La seule valeur obligatoire est **Nom d’utilisateur principal**. Puis enregistrez le fichier.

   ![Le fichier CSV contient les noms et les ID des utilisateurs à supprimer.](./media/users-bulk-delete/delete-csv-file.png)

1. Dans la page **Supprimer des utilisateurs en bloc**, sous **Charger votre fichier csv**, accédez au fichier. Quand vous sélectionnez le fichier et cliquez sur Envoyer, la validation du fichier CSV démarre.
1. Quand le contenu du fichier est validé, un message indique **Fichier chargé**. Si des erreurs sont présentes, vous devez les corriger avant de pouvoir envoyer le travail.
1. Lorsque votre fichier réussit la validation, sélectionnez **Envoyer** pour démarrer l’opération en bloc Azure qui supprime les utilisateurs.
1. Une fois l’opération de suppression terminée, vous recevez une notification indiquant que l’opération en bloc a réussi.

Si des erreurs se produisent, vous pouvez télécharger et consulter le fichier de résultats dans la page **Résultats de l’opération en bloc**. Le fichier contient la raison de chaque erreur.

## <a name="check-status"></a>Vérification du statut

Vous pouvez voir l’état de toutes vos demandes d’opération en bloc en attente dans la page **Résultats de l’opération en bloc**.

   [![Check delete status in the Bulk Operations Results page.](./media/users-bulk-delete/bulk-center.png)](./media/users-bulk-delete/bulk-center.png#lightbox)

Ensuite, vous pouvez vérifier si les utilisateurs que vous avez supprimés n’existent plus au sein de l’organisation Azure AD via le portail Azure ou à l’aide de PowerShell.

## <a name="verify-deleted-users-in-the-azure-portal"></a>Vérifier la suppression d’utilisateurs via le portail Azure

1. Connectez-vous au portail Azure en utilisant un compte d’administrateur d’utilisateurs de l’organisation.
1. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Utilisateurs**.
1. Sous **Afficher**, sélectionnez **Tous les utilisateurs** et vérifiez que les utilisateurs que vous avez supprimés ne sont plus répertoriés.

### <a name="verify-deleted-users-with-powershell"></a>Vérifier la suppression d’utilisateurs à l’aide de PowerShell

Exécutez la commande suivante :

``` PowerShell
Get-AzureADUser -Filter "UserType eq 'Member'"
```

Vérifiez que les utilisateurs que vous avez supprimés ne sont plus répertoriés.

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter des utilisateurs en bloc](users-bulk-add.md)
- [Télécharger une liste d’utilisateurs](users-bulk-download.md)
- [Restaurer des utilisateurs en bloc](users-bulk-restore.md)
