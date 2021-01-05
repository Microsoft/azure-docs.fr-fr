---
title: Groupes dynamiques et B2B Collaboration - Azure Active Directory | Microsoft Docs
description: Cet article montre comme utiliser des groupes dynamiques Azure AD avec Azure Active Directory B2B Collaboration.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/28/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5b820b8b9606795709d03414fa14ec29a1b5c519
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92441553"
---
# <a name="dynamic-groups-and-azure-active-directory-b2b-collaboration"></a>Groupes dynamiques et Azure Active Directory B2B Collaboration

## <a name="what-are-dynamic-groups"></a>Qu'est-ce-que les groupes dynamiques ?
La configuration dynamique de l’appartenance au groupe de sécurité pour Azure Active Directory (Azure AD) est disponible dans le [portail Azure](https://portal.azure.com). Les administrateurs peuvent définir des règles pour remplir les groupes créés dans Azure AD en fonction d’attributs utilisateur (par exemple, userType, le département ou le pays/la région). Les membres peuvent être automatiquement ajoutés ou supprimés d’un groupe de sécurité en fonction de leurs attributs. Ces groupes permettent d’accorder l’accès à des applications ou à des ressources cloud (sites SharePoint, documents) et d’attribuer des licences à des utilisateurs. En savoir plus sur les groupes dynamiques dans [Groupes dédiés dans Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

La [licence Azure AD Premium P1 ou P2](https://azure.microsoft.com/pricing/details/active-directory/) appropriée est nécessaire pour créer et utiliser des groupes dynamiques. Pour en savoir plus, consultez l’article [Créer des règles basées sur les attributs pour l’appartenance à un groupe dynamique dans Azure Active Directory](../enterprise-users/groups-dynamic-membership.md).

## <a name="creating-an-all-users-dynamic-group"></a>Création d'un groupe dynamique « Tous les utilisateurs »
Vous pouvez créer un groupe contenant tous les utilisateurs d’un client à l’aide d’une règle d’appartenance. Lors de l’ajout ou de la suppression ultérieurs d’utilisateurs dans le client, l’appartenance du groupe est ajustée automatiquement.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte attribué au rôle d'administrateur général ou d’administrateur d’utilisateurs dans le locataire.
1. Sélectionnez **Azure Active Directory** .
2. Sous **Gérer** , sélectionnez **Groupes** , puis **Nouveau groupe** .
1. Sur la page **Nouveau groupe** , sous **Type de groupe** , sélectionnez **Sécurité** . Entrez un **nom** et une **description** pour le nouveau groupe. 
2. Sous **Type d'appartenance** , sélectionnez **Utilisateur dynamique** , puis **Ajouter une requête dynamique** . 
4. Au-dessus de la zone de texte **Syntaxe de la règle** , sélectionnez **Modifier** . Sur la page **Modifier la syntaxe de la règle** , entrez l’expression suivante dans la zone de texte :

   ```
   user.objectId -ne null
   ```
1. Sélectionnez **OK** . La règle s’affiche dans la zone Syntaxe de la règle :

   ![Syntaxe de la règle du groupe dynamique « Tous les utilisateurs »](media/use-dynamic-groups/all-user-rule-syntax.png)

1.  Sélectionnez **Enregistrer** . Le nouveau groupe dynamique inclut désormais les utilisateurs invités B2B, ainsi que les utilisateurs membres.


1. Sélectionnez **Créer** sur la page **Nouveau groupe** pour créer le groupe.

## <a name="creating-a-group-of-members-only"></a>Création d’un groupe de membres uniquement

Si vous souhaitez exclure de votre groupe les utilisateurs invités et y inclure uniquement les membres de votre locataire, créez un groupe dynamique comme décrit ci-dessus, mais dans la zone **Syntaxe de la règle** , entrez l’expression suivante :

```
(user.objectId -ne null) and (user.userType -eq "Member")
```

L’image suivante illustre la syntaxe de règle d'un groupe dynamique modifié afin d’inclure uniquement les membres et d’exclure les invités.

![Règle pour laquelle le type utilisateur est Membre](media/use-dynamic-groups/all-member-user-rule-syntax.png)

## <a name="creating-a-group-of-guests-only"></a>Création d’un groupe d’invités uniquement

Vous trouverez peut-être aussi utile de créer un groupe dynamique contenant uniquement les utilisateurs invités, afin de pouvoir leur appliquer des stratégies (telles que des stratégies d’accès conditionnel Azure AD). Créez un groupe dynamique comme décrit ci-dessus, mais dans la zone **Syntaxe de la règle** , entrez l’expression suivante :

```
(user.objectId -ne null) and (user.userType -eq "Guest")
```

L’image suivante illustre la syntaxe de règle d'un groupe dynamique modifié afin d’inclure uniquement les invités et d’exclure les membres.

![Règle pour laquelle le type utilisateur est Invité](media/use-dynamic-groups/all-guest-user-rule-syntax.png)

## <a name="next-steps"></a>Étapes suivantes

- [Propriétés de l’utilisateur B2B Collaboration](user-properties.md)
- [Ajout d’un utilisateur B2B Collaboration à un rôle](add-guest-to-role.md)
- [Accès conditionnel pour les utilisateurs de B2B Collaboration](conditional-access.md)