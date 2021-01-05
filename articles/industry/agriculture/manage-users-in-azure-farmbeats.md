---
title: Gérer les utilisateurs dans Azure FarmBeats
description: Cet article explique comment gérer les utilisateurs dans Azure FarmBeats.
author: uhabiba04
ms.topic: article
ms.date: 12/02/2019
ms.author: v-umha
ms.openlocfilehash: e2769e97dde7f0b99bd29da9c6eb65954bc84dd2
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677560"
---
# <a name="manage-users"></a>Gestion des utilisateurs

Azure FarmBeats inclut la gestion des utilisateurs pour les personnes qui font partie de votre instance Azure Active Directory (Azure AD). Vous pouvez ajouter des utilisateurs à votre instance Azure FarmBeats pour accéder aux API, afficher les cartes générées et accéder aux données de télémétrie des capteurs de l’exploitation agricole.

## <a name="prerequisites"></a>Prérequis

- Une installation d’Azure FarmBeats est nécessaire. Pour plus d’informations, consultez [Installer Azure FarmBeats](install-azure-farmbeats.md).
- Les ID d’e-mail des utilisateurs que vous souhaitez ajouter ou supprimer dans votre instance Azure FarmBeats.

## <a name="manage-azure-farmbeats-users"></a>Gérer les utilisateurs Azure FarmBeats

Azure FarmBeats utilise Azure AD pour l’authentification, le contrôle d’accès et les rôles. Vous pouvez ajouter des utilisateurs dans le locataire Azure AD en tant qu’utilisateurs d’Azure FarmBeats.

> [!NOTE]
> Si un utilisateur n’est pas un utilisateur du locataire Azure AD, suivez les instructions de la section **Ajouter des utilisateurs Azure AD** pour effectuer l’installation.

Azure FarmBeats prend en charge deux types de rôle utilisateur :

 - **Administrateur**  : accès complet aux API Azure FarmBeats Datahub. Les utilisateurs ayant ce rôle peuvent interroger tous les objets Azure FarmBeats Datahub et effectuer toutes les opérations à partir de l’accélérateur FarmBeats.
 - **Lecture seule**  : accès en lecture seule aux API FarmBeats Datahub. Les utilisateurs peuvent afficher les API Datahub, les tableaux de bord de l’accélérateur et les cartes. Les utilisateurs disposant d’un accès en lecture seule ne peuvent pas effectuer certaines opérations comme générer des cartes, associer des appareils ou créer des exploitations agricoles.

## <a name="add-users-to-azure-farmbeats"></a>Ajouter des utilisateurs à Azure FarmBeats

Pour ajouter des utilisateurs à Azure FarmBeats :

1. Connectez-vous à l’accélérateur, puis sélectionnez l’icône **Settings** (Paramètres).
2. Sélectionnez **Contrôle d’accès** .

    ![Volet Farms Settings (Paramètres des exploitations agricoles)](./media/create-farms-in-azure-farmbeats/settings-users-1.png)

3. Entrez l’ID d’e-mail de l’utilisateur auquel vous souhaitez accorder l’accès.
4. Sélectionnez le rôle souhaité, **Admin** (Administrateur) ou **Read-Only** (Lecture seule).
5. Sélectionnez **Ajouter un rôle** .

L’utilisateur ajouté peut désormais accéder à Azure FarmBeats (Datahub et l’accélérateur).

## <a name="delete-users-from-azure-farmbeats"></a>Supprimer des utilisateurs d’Azure FarmBeats

Pour supprimer des utilisateurs du système Azure FarmBeats :

1. Connectez-vous à l’accélérateur, puis sélectionnez l’icône **Settings** (Paramètres).
2. Sélectionnez **Contrôle d’accès** .
3. Sélectionnez **Supprimer** .

   L’utilisateur est supprimé du système. Vous recevez alors le message de confirmation suivant :

   ![Message de confirmation d’Azure FarmBeats](./media/create-farms-in-azure-farmbeats/manage-users-2.png)

## <a name="add-azure-ad-users"></a>Ajouter des utilisateurs Azure AD

> [!NOTE]
> Les utilisateurs d’Azure FarmBeats doivent exister dans le locataire Azure AD pour pouvoir leur affecter des applications et des rôles. Si un utilisateur n’existe pas dans le locataire Azure AD, suivez les instructions de cette section. Ignorez les instructions si un utilisateur existe déjà dans le locataire Azure AD.

Suivez les étapes pour ajouter des utilisateurs à Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. En haut à droite, sélectionnez votre compte, puis basculez vers le locataire Azure AD associé à FarmBeats.
3. Sélectionnez **Azure Active Directory** > **Utilisateurs** .

    La liste des utilisateurs Azure AD s’affiche.

4. Pour ajouter un utilisateur à l’annuaire, sélectionnez **Nouvel utilisateur** . Pour ajouter un utilisateur externe, sélectionnez **Nouvel utilisateur invité** .

    ![Volet « Tous les utilisateurs »](./media/create-farms-in-azure-farmbeats/manage-users-3.png)

5. Sélectionnez le nom du nouvel utilisateur, puis complétez les champs obligatoires pour cet utilisateur.
6. Sélectionnez **Create** (Créer).

Pour plus d’informations sur la gestion des utilisateurs Azure AD, consultez [Ajouter ou supprimer des utilisateurs dans Azure AD](../../active-directory/fundamentals/add-users-azure-active-directory.md).

## <a name="next-steps"></a>Étapes suivantes

Vous êtes parvenu à ajouté des utilisateurs à votre instance Azure FarmBeats. À présent, découvrez comment [créer et gérer des exploitations agricoles](manage-farms-in-azure-farmbeats.md#create-farms).