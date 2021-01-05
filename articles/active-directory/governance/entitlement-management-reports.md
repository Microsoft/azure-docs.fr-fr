---
title: Afficher les rapports et les journaux dans la gestion des droits d’utilisation - Azure AD
description: Découvrez comment afficher le rapport d’affectations d’utilisateur et les journaux d’audit dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: jocastel-MSFT
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: jocastel
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6eb44c1efd683b6febe9a355ef72c80cc6f2e40d
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746624"
---
# <a name="view-reports-and-logs-in-azure-ad-entitlement-management"></a>Afficher les rapports et les journaux dans la gestion des droits d’utilisation Azure AD

Les rapports de gestion des droits d’utilisation d’Azure AD et le journal d’audit d’Azure AD fournissent des détails supplémentaires sur les ressources auxquelles les utilisateurs ont accès. En tant qu’administrateur, vous pouvez afficher les packages d’accès et les affectations de ressources pour un utilisateur et afficher les journaux des requêtes à des fins d’audit, ou pour déterminer l’état de la requête d’un utilisateur. Cet article explique comment utiliser les rapports de gestion des droits d’utilisation et les journaux d’audit d’Azure AD.

Regardez la vidéo suivante pour savoir comment afficher les ressources auxquelles les utilisateurs ont accès dans la gestion des droits d’utilisation :

>[!VIDEO https://www.youtube.com/embed/omtNJ7ySjS0]

## <a name="view-access-packages-for-a-user"></a>Afficher les packages d’accès d’un utilisateur

Ce rapport vous permet de répertorier tous les packages d’accès qu’un utilisateur peut demander et les packages d’accès qui sont actuellement affectés à l’utilisateur.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

1. Cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu gauche, cliquez sur **Rapports**.

1. Cliquez sur **Afficher les packages d’accès d’un utilisateur**.

1. Cliquez sur **Sélectionner des utilisateurs** pour ouvrir le volet Sélectionner des utilisateurs.

1. Recherchez l’utilisateur dans la liste, puis cliquez sur **Sélectionner**.

    L’onglet **Peut demander** affiche la liste des packages d’accès que l’utilisateur peut demander. Cette liste est déterminée par les [stratégies de demande](entitlement-management-access-package-request-policy.md#for-users-in-your-directory) définies pour les packages d’accès. 

    ![Packages d’accès d’un utilisateur](./media/entitlement-management-reports/access-packages-report.png)

1. S’il existe plusieurs rôles de ressource ou de stratégie pour un package d’accès, cliquez sur l’entrée rôles de ressource ou stratégies pour afficher les détails de la sélection.

1. Cliquez sur l’onglet **Affecté** pour afficher la liste des packages d’accès actuellement affectés à l’utilisateur. Lorsqu’un package d’accès est attribué à un utilisateur, cela signifie que l’utilisateur a accès à tous les rôles de ressource dans le package d’accès.

## <a name="view-resource-assignments-for-a-user"></a>Afficher les attributions de ressources pour un utilisateur

Ce rapport vous permet de répertorier les ressources actuellement attribuées à un utilisateur dans la gestion des droits d’utilisation. Notez que ce rapport est destiné aux ressources gérées avec la gestion des droits d’utilisation. L’utilisateur peut avoir accès à d’autres ressources dans votre répertoire en dehors de la gestion des droits d’utilisation.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

1. Cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu gauche, cliquez sur **Rapports**.

1. Cliquez sur **Attributions de ressources pour un utilisateur**.

1. Cliquez sur **Sélectionner des utilisateurs** pour ouvrir le volet Sélectionner des utilisateurs.

1. Recherchez l’utilisateur dans la liste, puis cliquez sur **Sélectionner**.

    La liste des ressources actuellement affectées à l’utilisateur s’affiche. La liste affiche également le package d’accès et la stratégie à partir desquels ils ont le rôle de ressource, ainsi que la date de début et de fin pour l’accès.
    
    Si un utilisateur a obtenu l’accès à la même ressource dans plusieurs packages, vous pouvez cliquer sur une flèche pour afficher chaque package et chaque stratégie.

    ![Attributions de ressources pour un utilisateur](./media/entitlement-management-reports/resource-assignments-report.png)

## <a name="determine-the-status-of-a-users-request"></a>Déterminer l’état de la demande d’un utilisateur

Pour obtenir plus d’informations sur la façon dont un utilisateur a demandé et reçu l’accès à un package d’accès, vous pouvez utiliser le journal d’audit Azure AD. En particulier, vous pouvez utiliser des enregistrements de journal dans les catégories `EntitlementManagement` et `UserManagement` pour obtenir des détails supplémentaires sur les étapes de traitement pour chaque demande.  

1. Cliquez sur **Azure Active Directory**, puis sur **Journaux d’audit**.

1. En haut, remplacez la **Catégorie** par `EntitlementManagement` ou `UserManagement`, en fonction de l’enregistrement d’audit que vous recherchez.  

1. Cliquez sur **Appliquer**.

1. Cliquez sur **Télécharger** pour télécharger les journaux.

Lorsqu’Azure AD reçoit une nouvelle demande, il écrit un enregistrement d’audit, dont la **catégorie** est `EntitlementManagement` et l’**activité** est généralement `User requests access package assignment`.  Dans le cas d’une affectation directe créée dans le portail Azure, le champ **activité** de l’enregistrement d’audit est `Administrator directly assigns user to access package`, et l’utilisateur effectuant l’affectation est identifiée par **ActorUserPrincipalName**.

Azure AD écrira des enregistrements d’audit supplémentaires lorsque la demande est en cours, y compris :

| Category | Activité | État de la demande |
| :---- | :------------ | :------------ |
| `EntitlementManagement` | `Auto approve access package assignment request` | La demande ne nécessite pas d’approbation |
| `UserManagement` | `Create request approval` | La demande nécessite une approbation |
| `UserManagement` | `Add approver to request approval` | La demande nécessite une approbation |
| `EntitlementManagement` | `Approve access package assignment request` | Demande approuvée |
| `EntitlementManagement` | `Ready to fulfill access package assignment request` |Demande approuvée, ou ne nécessite pas d’approbation |

Lorsqu’un utilisateur se voit affecter l’accès, Azure AD écrit un enregistrement d’audit pour la catégorie `EntitlementManagement` avec l’**activité** `Fulfill access package assignment`.  L’utilisateur qui a reçu l’accès est identifié par le champ **ActorUserPrincipalName**.

Si l’accès n’a pas été affecté, Azure AD écrit un enregistrement d’audit pour la catégorie `EntitlementManagement` avec l’**activité**`Deny access package assignment request`, si la demande a été refusée par un approbateur, ou `Access package assignment request timed out (no approver action taken)`, si la demande a expiré avant l’approbation.

Lorsque l’attribution du package d’accès d’un utilisateur expire, est annulé par l’utilisateur, ou supprimé par un administrateur, Azure AD écrit un enregistrement d’audit pour la catégorie `EntitlementManagement` avec l’**activité** de `Remove access package assignment`.

## <a name="next-steps"></a>Étapes suivantes

- [Archiver les rapports et les journaux](entitlement-management-logs-and-reporting.md)
- [Résoudre les problèmes de gestion des droits d’utilisation Azure AD](entitlement-management-troubleshoot.md)
- [Scénarios courants](entitlement-management-scenarios.md)
