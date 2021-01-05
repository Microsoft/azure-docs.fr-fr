---
title: Commencer à utiliser PIM - Azure Active Directory | Microsoft Docs
description: Découvrez comment activer et bien démarrer avec Azure AD Privileged Identity Management (PIM) dans le portail Azure.
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.topic: how-to
ms.workload: identity
ms.date: 09/15/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5bcfb21ab15355653780355f1b5e459bc806ec8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90600719"
---
# <a name="start-using-privileged-identity-management"></a>Commencer à utiliser Privileged Identity Management

Cet article décrit comment activer et bien démarrer avec Privileged Identity Management (PIM).

Utilisez Privileged Identity Management (PIM) pour gérer, contrôler et analyser des accès au sein de votre organisation Azure Active Directory (Azure AD). La technologie PIM vous permet de fournir un accès en fonction des besoins et juste-à-temps aux ressources Azure et Azure AD, ainsi qu’à d’autres services en ligne Microsoft, tels que Microsoft 365 ou Microsoft Intune.

## <a name="prerequisites"></a>Prérequis

Pour utiliser Privileged Identity Management, vous devez disposer de l’une des licences suivantes :

- Azure AD Premium P2
- Enterprise Mobility + Security (EMS) E5

Pour plus d’informations, consultez [Exigences relatives aux licences pour l’utilisation de Privileged Identity Management](subscription-requirements.md).

> [!Note]
> Lorsqu’un utilisateur qui est actif dans un rôle privilégié dans une organisation Azure AD avec une licence Premium P2 accède à **Rôles et administrateurs** dans Azure AD et sélectionne un rôle (ou accède même simplement à Privileged Identity Management) :
>
> - Nous activons automatiquement PIM pour l’organisation.
> - L’utilisateur peut maintenant affecter une attribution de rôle « normale » ou une attribution de rôle éligible.
>
> Lorsque PIM est activé, cela n’a aucun autre effet sur votre organisation dont vous devez vous préoccuper. Il vous offre des options d’attribution supplémentaires, telles que actif/éligible avec heure de début et de fin. PIM vous permet également de définir l’étendue des attributions de rôles à l’aide d’unités administratives et de rôles personnalisés. Si vous êtes administrateur général ou administrateur de rôle privilégié, vous commencerez peut-être à recevoir quelques e-mails supplémentaires tels que le résumé hebdomadaire PIM. Vous constaterez peut-être aussi la présence du principal de service MS-PIM dans le journal d’audit relatif à l’attribution de rôle. Il s’agit d’un changement prévu qui ne devrait avoir aucun effet sur votre workflow.

## <a name="prepare-pim-for-azure-ad-roles"></a>Préparer PIM pour les rôles Azure AD

Voici les tâches que nous vous recommandons pour préparer Privileged Identity Management et gérer les rôles Azure AD :

1. [Configurez les paramètres de rôle Azure AD](pim-how-to-change-default-settings.md).
1. [Donnez des affectations éligibles](pim-how-to-add-role-to-user.md).
1. [Autorisez les utilisateurs éligibles à activer leur rôle juste-à-temps](pim-how-to-activate-role.md).

## <a name="prepare-pim-for-azure-roles"></a>Préparer PIM pour les rôles Azure

Voici les tâches que nous vous recommandons pour préparer Privileged Identity Management et gérer les rôles Azure pour un abonnement :

1. [Découvrir les ressources Azure](pim-resource-roles-discover-resources.md)
1. [Configurez les paramètres de rôle Azure](pim-resource-roles-configure-role-settings.md).
1. [Donnez des affectations éligibles](pim-resource-roles-assign-roles.md).
1. [Autorisez les utilisateurs éligibles à activer leurs rôles Azure juste-à-temps](pim-resource-roles-activate-your-roles.md).

## <a name="navigate-to-your-tasks"></a>Accédez à vos tâches

Une fois Privileged Identity Management configuré, vous pouvez vous familiariser avec.

![Fenêtre de navigation dans Privileged Identity Management présentant les tâches et les options de gestion](./media/pim-getting-started/pim-quickstart-tasks.png)

| Tâche + Gérer | Description |
| --- | --- |
| **Mes rôles**  | Affiche la liste des rôles éligibles et actifs qui vous sont attribués. C’est là où vous pouvez activer tout rôle éligible attribué. |
| **Mes demandes** | Affiche toutes vos requêtes en attente pour activer les attributions de rôles éligibles. |
| **Approuver les demandes** | Affiche la liste des demandes d’activation des rôles éligibles par utilisateur dans votre annuaire, dont l’approbation vous revient. |
| **Revoir l’accès** | Répertorie les révisions d’accès que vous devez terminer, que vous les effectuiez pour vous-même ou pour quelqu’un d’autre. |
| **Rôles Azure AD** | Affiche un tableau de bord et des paramètres pour les administrateurs de rôle privilégié qui doivent gérer les attributions de rôles Azure AD. Ce tableau de bord est désactivée pour toute personnes n’étant pas un administrateur de rôle privilégié. Ces utilisateurs ont accès à un tableau de bord spécial intitulé Mon affichage. Le tableau de bord Mon affichage présente uniquement des informations sur l’utilisateur qui accède au tableau de bord, sur l’ensemble de l’organisation. |
| **Ressources Azure** | Affiche un tableau de bord et des paramètres pour les administrateurs de rôle privilégié qui doivent gérer les attributions de rôles de ressources Azure. Ce tableau de bord est désactivée pour toute personnes n’étant pas un administrateur de rôle privilégié. Ces utilisateurs ont accès à un tableau de bord spécial intitulé Mon affichage. Le tableau de bord Mon affichage présente uniquement des informations sur l’utilisateur qui accède au tableau de bord, sur l’ensemble de l’organisation. |

## <a name="add-a-pim-tile-to-the-dashboard"></a>Ajouter une vignette PIM au tableau de bord

Pour faciliter l’ouverture de Privileged Identity Management, ajoutez une vignette PIM à votre tableau de bord du portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Cliquez sur **Tous les services**, puis recherchez le service **Azure AD Privileged Identity Management**.

    ![Azure AD Privileged Identity Management dans Tous les services](./media/pim-getting-started/pim-all-services-find.png)

1. Sélectionnez le **Démarrage rapide** Privileged Identity Management.

1. Sélectionnez **Épingler le panneau au tableau de bord** pour épingler la page **Démarrage rapide** Privileged Identity Management au tableau de bord.

    ![Icône de punaise pour épingler la page Privileged Identity Management au tableau de bord](./media/pim-getting-started/pim-quickstart-pin-to-dashboard.png)

    Dans le tableau de bord Azure, vous verrez une vignette comme celle-ci :

    ![Mosaïque Démarrage rapide Privileged Identity Management sur le tableau de bord](./media/pim-getting-started/pim-quickstart-dashboard-tile.png)

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles dans Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Gérer l’accès aux ressources Azure dans Privileged Identity Management](pim-resource-roles-discover-resources.md)
