---
title: Créer un groupe de base et ajouter des membres - Azure Active Directory | Microsoft Docs
description: Instruction sur la création d'un groupe de base avec Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 06/05/2020
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18, contperf-fy20q4
ms.collection: M365-identity-device-management
ms.openlocfilehash: f8041656a11689d0cca64c3edae1af2e70409028
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027711"
---
# <a name="create-a-basic-group-and-add-members-using-azure-active-directory"></a>Créer un groupe de base et ajouter des membres avec Azure Active Directory
Vous pouvez créer un groupe de base à l’aide du portail Azure Active Directory (Azure AD). Pour les besoins de cet article, un groupe de base est ajouté à une seule ressource par le propriétaire de la ressource (administrateur) ; il inclut les membres spécifiques (employés) qui doivent accéder à cette ressource. Pour des scénarios plus complexes, notamment la création de règles et les appartenances dynamiques, consultez la [documentation Gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).

## <a name="group-and-membership-types"></a>Types d’appartenance et de groupe
Il existe plusieurs types de groupe et d’appartenance. Les informations suivantes expliquent chaque type de groupe et d’appartenance, et pourquoi ils sont utilisés. Vous pourrez ainsi déterminer les options à utiliser lorsque vous créez un groupe.

### <a name="group-types"></a>Types de groupe :
- **Sécurité**. Utilisé pour gérer l’accès de membres et d’ordinateurs aux ressources partagées d’un groupe d’utilisateurs. Par exemple, vous pouvez créer un groupe de sécurité pour une stratégie de sécurité particulière. En procédant ainsi, vous pouvez donner un ensemble d’autorisations à tous les membres à la fois, au lieu de devoir ajouter des autorisations individuellement à chaque membre. Un groupe de sécurité peut avoir des utilisateurs, des appareils, des groupes et des principaux de service comme membres ainsi que des utilisateurs et des principaux de service comme propriétaires. Pour plus d’informations sur la gestion de l’accès aux ressources, consultez [Gérer l’accès aux ressources avec des groupes Azure Active Directory](active-directory-manage-groups.md).
- **Microsoft 365**. Fournit des opportunités de collaboration en donnant aux membres l’accès à des éléments partagés : une boîte aux lettres, un calendrier, des fichiers, un site SharePoint et bien plus encore. Cette option vous permet également de donner à des personnes extérieures à votre organisation un accès au groupe. Un groupe Microsoft 365 ne peut contenir que des utilisateurs comme membres. Les utilisateurs et les principaux de service peuvent être propriétaires d’un groupe Microsoft 365. Pour plus d’informations sur les groupes Microsoft 365, consultez [En savoir plus sur les groupes Microsoft 365](https://support.office.com/article/learn-about-office-365-groups-b565caa1-5c40-40ef-9915-60fdb2d97fa2).

### <a name="membership-types"></a>Types d’appartenance :
- **Affecté.** Vous permet d’ajouter des utilisateurs spécifiques pour qu’ils soient membres de ce groupe et qu’ils disposent d’autorisations uniques. Pour les besoins de cet article, nous utilisons cette option.
- **Utilisateur dynamique.** Vous permet d’utiliser des règles d’appartenance dynamique pour ajouter et supprimer automatiquement des membres. Si les attributs d’un membre changent, le système examine vos règles de groupe dynamique par rapport à l’annuaire, et voit si le membre répond aux exigences des règles (il est ajouté) ou s’il ne répond plus aux exigences de ces règles (il est supprimé).
- **Appareil dynamique.** Vous permet d’utiliser des règles de groupe dynamique pour ajouter et supprimer automatiquement des appareils. Si les attributs d’un appareil changent, le système examine vos règles de groupe dynamique par rapport à l’annuaire, et voit si l’appareil répond aux exigences des règles (il est ajouté) ou s’il ne répond plus aux exigences de ces règles (il est supprimé).

    > [!IMPORTANT]
    > Vous pouvez créer un groupe dynamique pour des appareils ou pour des utilisateurs, mais pas pour les deux. Vous ne pouvez pas non plus créer de groupe d’appareils d’après les attributs des propriétaires d’appareils. Les règles d’appartenance d’appareil peuvent référencer uniquement des attributions d’appareils. Pour plus d’informations sur la création d’un groupe dynamique d’utilisateurs ou d’appareils, consultez [Créer un groupe dynamique et vérifier l’état](../enterprise-users/groups-create-rule.md).

## <a name="create-a-basic-group-and-add-members"></a>Créer un groupe de base et ajouter des membres
Vous pouvez créer un groupe de base et ajouter vos membres en même temps. Pour créer un groupe de base et ajouter des membres, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général pour le répertoire.

1. Recherchez et sélectionnez **Azure Active Directory**.

1. Sur la page **Active Directory**, sélectionnez **Groupes**, puis **Nouveau groupe**.

    ![Page Azure AD, avec affichage des groupes](media/active-directory-groups-create-azure-portal/group-full-screen.png)

1. Sur la page **Nouveau groupe** qui s’affiche, fournissez les informations obligatoires.

    ![Page de création d’un groupe, renseignée avec les informations de l’exemple](media/active-directory-groups-create-azure-portal/new-group-blade.png)

1. Sélectionnez un **type de groupe** prédéfini. Pour plus d’informations sur les types de groupe, consultez [Types de groupe et d’appartenance](#group-types).

1. Créez et ajoutez un **nom de groupe**. Choisissez un nom dont vous vous souviendrez et qui est logique pour le groupe. Une vérification sera effectuée pour déterminer si le nom est déjà utilisé par un autre groupe. Si le nom est déjà en cours d’utilisation, pour éviter les noms en double, vous devrez modifier le nom de votre groupe.

1. Ajoutez une **adresse e-mail de groupe** pour le groupe ou conservez l’adresse e-mail qui est renseignée automatiquement.

1. **Description du groupe**. Ajoutez une description facultative à votre groupe.

1. Sélectionnez un **type d’appartenance (obligatoire)** prédéfini. Pour plus d’informations sur les types d’appartenance, consultez [Types de groupe et d’appartenance](#membership-types).

1. Sélectionnez **Create** (Créer). Votre groupe est créé et prêt pour l’ajout de membres.

1. Sélectionnez la zone **Membres** dans la page **Groupe**, puis commencez à rechercher les membres à ajouter à votre groupe à partir de la page **Sélectionner des membres**.

    ![Sélection de membres pour votre groupe pendant le processus de création du groupe](media/active-directory-groups-create-azure-portal/select-members-create-group.png)

1. Lorsque vous avez terminé l’ajout de membres, choisissez **Sélectionner**.

    La page **Présentation du groupe** est mise à jour pour afficher le nombre de membres désormais ajoutés au groupe.

    ![Page de la vue d’ensemble du groupe avec le nombre de membres mis en évidence](media/active-directory-groups-create-azure-portal/group-overview-blade-number-highlight.png)

## <a name="turn-on-or-off-group-welcome-email"></a>Activer ou désactiver l’e-mail de bienvenue dans le groupe

Lors de la création d'un groupe Microsoft 365, que l'appartenance soit dynamique ou statique, une notification de bienvenue est envoyée à tous les utilisateurs qui sont ajoutés au groupe. Lorsqu’un attribut d’utilisateur ou d’appareil change, toutes les règles de groupe dynamique au sein de l’organisation sont traitées pour des modifications d’appartenance potentielles. Les utilisateurs qui sont alors ajoutés reçoivent également la notification de bienvenue. Vous pouvez désactiver ce comportement dans [Exchange PowerShell](/powershell/module/exchange/users-and-groups/Set-UnifiedGroup). 

## <a name="next-steps"></a>Étapes suivantes

- [Gérer l’accès aux applications SaaS en utilisant des groupes](../enterprise-users/groups-saasapps.md)
- [Gérer des groupes au moyen de commandes PowerShell](../enterprise-users/groups-settings-v2-cmdlets.md)
