---
title: Modifier les paramètres de demande d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD - Azure Active Directory
description: Apprenez à modifier les paramètres de demande d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 09/16/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: b7b70e8a408c22be0331bfd0dcbe01830b072ab8
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91449966"
---
# <a name="change-request-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Modifier les paramètres de demande d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD

En tant que gestionnaire de package d’accès, vous pouvez modifier la liste des utilisateurs qui peuvent demander un package d’accès à tout moment en éditant la stratégie ou en ajoutant une nouvelle stratégie. Cet article explique comment modifier les paramètres de demande d'un package d'accès existant.

## <a name="choose-between-one-or-multiple-policies"></a>Choisir entre une ou plusieurs stratégies

Pour choisir les personnes autorisées à demander un package d'accès, utilisez une stratégie. Avant de créer une nouvelle stratégie ou de modifier une stratégie existante dans un package d'accès, vous devez déterminer le nombre de stratégies dont le package d'accès a besoin. 

Lorsque vous créez un package d'accès, vous spécifiez le paramètre de demande qui crée une stratégie. La plupart des packages d’accès sont associés à une seule stratégie, mais un même package d’accès peut posséder plusieurs stratégies. Vous pouvez créer plusieurs stratégies pour un package d'accès si vous souhaitez permettre à différents groupes d'utilisateurs de se voir attribuer des affectations avec différents paramètres de requête et d’approbation. 

Par exemple, une stratégie unique ne permet pas d’affecter des utilisateurs internes et externes à un même package d'accès. En revanche, vous pouvez créer deux stratégies dans un même package d'accès, une pour les utilisateurs internes et une autre pour les utilisateurs externes. Si plusieurs stratégies s'appliquent à un utilisateur, celui-ci sera invité, au moment de sa requête, à sélectionner la stratégie à laquelle il souhaite être affecté. Le diagramme suivant montre un package d’accès avec deux stratégies.

![Plusieurs stratégies dans un package d’accès](./media/entitlement-management-access-package-request-policy/access-package-policy.png)

### <a name="how-many-policies-will-i-need"></a>De combien de stratégies ai-je besoin ?

| Scénario | Nombre de stratégies |
| --- | --- |
| Je souhaite que tous les utilisateurs de mon annuaire aient les mêmes paramètres de demande et d’approbation pour un package d’accès | Une |
| Je souhaite que tous les utilisateurs de certaines organisations connectées puissent demander un package d’accès | Une |
| Je souhaite autoriser les utilisateurs de mon annuaire et également les utilisateurs en dehors de mon annuaire à demander un package d’accès | Multiple |
| Je souhaite spécifier différents paramètres d’approbation pour certains utilisateurs | Multiple |
| Je veux que certains utilisateurs accèdent aux affectations de packages d’accès qui expirent alors que d’autres utilisateurs peuvent étendre leur accès | Multiple |

Pour plus d’informations sur la logique de priorité utilisée lorsque plusieurs stratégies s’appliquent, consultez [Stratégies multiples](entitlement-management-troubleshoot.md#multiple-policies
).

## <a name="open-an-existing-access-package-and-add-a-new-policy-of-request-settings"></a>Ouvrir un package d'accès existant et ajouter une nouvelle stratégie de paramètres de demande

Si vous disposez d’un ensemble d’utilisateurs qui doivent avoir des paramètres de demande et d’approbation différents, vous devrez probablement créer une nouvelle stratégie. Suivez ces étapes pour commencer à ajouter une nouvelle stratégie à un package d'accès existant :

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Stratégies**, puis sur **Ajouter une stratégie**.

1. Vous commencez par l'onglet **Paramètres de base**. Entrez un nom et une description pour la stratégie.

    ![Créer une stratégie avec un nom et une description](./media/entitlement-management-access-package-request-policy/policy-name-description.png)

1. Cliquez sur **Suivant** pour ouvrir l’onglet **Requêtes**.

1. Modifiez le paramètre **Utilisateurs pouvant demander l'accès**. Suivez les étapes des sections ci-dessous pour définir le paramètre sur l'une des options suivantes : 
    - [Pour les utilisateurs dans votre répertoire](#for-users-in-your-directory) 
    - [Pour les utilisateurs qui ne sont pas dans votre annuaire](#for-users-not-in-your-directory)
    - [Aucune (attributions direct administrateur uniquement)](#none-administrator-direct-assignments-only)

## <a name="for-users-in-your-directory"></a>Pour les utilisateurs dans votre répertoire

Suivez ces étapes si vous souhaitez autoriser les utilisateurs de votre annuaire à demander ce package d’accès. Lors de la définition de la stratégie de demandes, vous pouvez spécifier des utilisateurs individuels ou plus souvent des groupes d’utilisateurs. Par exemple, votre organisation a peut-être déjà un groupe tel que **Tous les employés**.  Si ce groupe est ajouté dans la stratégie pour des utilisateurs qui peuvent demander l’accès, tout membre de ce groupe peut ensuite demander l’accès.

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Pour les utilisateurs dans votre annuaire**.

    Quand vous sélectionnez cette option, de nouvelles options s’affichent pour vous permettre de préciser qui dans votre annuaire peut demander ce package d’accès.

    ![Package d’accès - Demandes - Pour les utilisateurs de votre annuaire](./media/entitlement-management-access-package-request-policy/for-users-in-your-directory.png)

1. Sélectionnez l’une des options suivantes :

    |  |  |
    | --- | --- |
    | **Utilisateurs et groupes spécifiques** | Choisissez cette option si vous souhaitez que seuls les utilisateurs et les groupes de votre annuaire que vous spécifiez puissent demander ce package d’accès. |
    | **Tous les membres (à l’exclusion des invités)** | Choisissez cette option si vous souhaitez que tous les utilisateurs membres de votre annuaire puissent demander ce package d’accès. Cette option n’inclut pas les utilisateurs que vous avez invités dans votre annuaire. |
    | **Tous les utilisateurs (y compris les invités)** | Choisissez cette option si vous souhaitez que tous les utilisateurs membres et les utilisateurs invités de votre annuaire puissent demander ce package d’accès. |

    Les utilisateurs invités font référence à des utilisateurs externes qui ont été invités dans votre annuaire avec [Azure AD B2B](../external-identities/what-is-b2b.md). Pour plus d’informations sur les différences entre les utilisateurs membres et les utilisateurs invités, consultez [Quelles sont les autorisations utilisateur par défaut dans Azure Active Directory ?](../fundamentals/users-default-permissions.md).

1. Si vous avez sélectionné **Utilisateurs et groupes spécifiques**, cliquez sur **Ajouter des utilisateurs et des groupes**.

1. Dans le volet Sélectionnez des utilisateurs et des groupes, sélectionnez les utilisateurs et les groupes à ajouter.

    ![Package d’accès - Demandes - Sélectionner des utilisateurs et des groupes](./media/entitlement-management-access-package-request-policy/select-users-groups.png)

1. Cliquez sur **Sélectionner** pour ajouter les utilisateurs et les groupes.

1. Si vous souhaitez exiger une approbation, suivez les étapes décrites dans [Modifier les paramètres d'approbation d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD](entitlement-management-access-package-approval-policy.md) pour configurer les paramètres d'approbation.

1. Accédez à la section [Activer les demandes](#enable-requests).
 
## <a name="for-users-not-in-your-directory"></a>Pour les utilisateurs qui ne sont pas dans votre répertoire

 **Les utilisateurs absents de votre annuaire** sont des utilisateurs qui se trouvent dans un autre annuaire ou domaine Azure AD. Ces utilisateurs n’ont peut-être pas encore été invités dans votre annuaire. Les annuaires Azure AD doivent être configurés pour autoriser les invitations dans **Restrictions de collaboration**. Pour plus d’informations, consultez [Permettre une collaboration B2B externe et gérer les utilisateurs autorisés à en inviter d’autres](../external-identities/delegate-invitations.md).

> [!NOTE]
> Un compte d’utilisateur invité sera créé pour un utilisateur qui n’est pas encore dans votre répertoire et dont la demande est approuvée ou approuvée automatiquement. L’invité est invité, mais ne recevra pas d’invitation par e-mail. Il recevra un e-mail une fois l’attribution au package d’accès fournie. Par défaut, quand cet utilisateur invité n’a plus aucune attribution de package d’accès (pour cause d’expiration ou d’annulation), son compte ne peut plus se connecter et est supprimé par la suite. Si vous voulez que des utilisateurs invités demeurent indéfiniment dans votre répertoire, même s’ils n’ont plus d’attribution de package d’accès, vous pouvez modifier les paramètres de votre configuration de gestion des droits d’utilisation. Pour plus d’informations sur l’objet utilisateur invité, consultez [Propriétés d’un utilisateur Azure Active Directory B2B Collaboration](../external-identities/user-properties.md).

Suivez ces étapes si vous souhaitez autoriser les utilisateurs absents de votre annuaire à demander ce package d’accès :

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Pour les utilisateurs dans votre annuaire**.

    Quand vous sélectionnez cette option, de nouvelles options s’affichent.

    ![Package d’accès - Demandes - Pour les utilisateurs qui ne sont pas dans votre annuaire](./media/entitlement-management-access-package-request-policy/for-users-not-in-your-directory.png)

1. Sélectionnez l’une des options suivantes :

    |  |  |
    | --- | --- |
    | **Organisations connectées spécifiques** | Choisissez cette option si vous souhaitez sélectionner parmi une liste d’organisations que votre administrateur a ajoutées précédemment. Tous les utilisateurs des organisations sélectionnées peuvent demander ce package d’accès. |
    | **Toutes les organisations connectées configurées** | Choisissez cette option si tous les utilisateurs de toutes les organisations connectées configurées peuvent demander ce package d’accès. Seuls les utilisateurs des organisations connectées configurées peuvent demander des packages d’accès affichés aux utilisateurs de toutes les organisations configurées. |
    | **Tous les utilisateurs (toutes les organisations connectées + tous les nouveaux utilisateurs externes)** | Choisissez cette option si un utilisateur sur Internet doit être en mesure de demander ce package d’accès.  Si l’utilisateur n’appartient pas à une organisation connectée dans votre annuaire, une organisation connectée est automatiquement créée lorsqu’il demande le package. L’organisation connectée créée automatiquement se trouve à l’état **proposé**. Pour plus d’informations sur l’état proposé, consultez [Propriétés d’état des organisations connectées](entitlement-management-organization.md#state-properties-of-connected-organizations). |

    Une organisation connectée est un domaine ou un annuaire Azure AD externe avec lequel vous collaborez.

1. Si vous avez sélectionné **Organisations connectées spécifiques**, cliquez sur **Ajouter des annuaires** pour effectuer une sélection parmi une liste d’organisations connectées que votre administrateur a ajoutées précédemment.

1. Tapez le nom ou le nom de domaine pour rechercher une organisation précédemment connectée.

    ![Package d’accès - Demandes - Sélectionner des annuaires](./media/entitlement-management-access-package-request-policy/select-directories.png)

    Si l’organisation avec laquelle vous souhaitez collaborer ne figure pas dans la liste, vous pouvez demander à votre administrateur de l’ajouter en tant qu’organisation connectée. Pour plus d’informations, voir [Ajouter une organisation connectée](entitlement-management-organization.md).

1. Une fois que vous avez sélectionné toutes les organisations connectées, cliquez sur **Sélectionner**.

    > [!NOTE]
    > Tous les utilisateurs des organisations connectées sélectionnées pourront demander ce package d’accès. Cela comprend les utilisateurs Azure AD de tous les sous-domaines associés à l’organisation, à moins que ces domaines soient bloqués par la liste d’autorisation ou de refus d’Azure B2B. Pour plus d’informations, consultez [Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques](../external-identities/allow-deny-list.md).

1. Si vous souhaitez exiger une approbation, suivez les étapes décrites dans [Modifier les paramètres d'approbation d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD](entitlement-management-access-package-approval-policy.md) pour configurer les paramètres d'approbation.
 
1. Accédez à la section [Activer les demandes](#enable-requests).

## <a name="none-administrator-direct-assignments-only"></a>Aucune (attributions direct administrateur uniquement)

Effectuez ces étapes si vous souhaitez contourner les demandes d’accès et autoriser les administrateurs à attribuer directement des utilisateurs spécifiques à ce package d’accès. Les utilisateurs n’auront pas à demander le package d’accès. Vous pouvez toujours définir des paramètres de cycle de vie, mais il n’y a aucun paramètre de demande.

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Aucune (attributions direct administrateur uniquement)** .

    ![Package d’accès - Demandes - Aucun (attributions directes d’administrateur uniquement)](./media/entitlement-management-access-package-request-policy/none-admin-direct-assignments-only.png)

    Après avoir créé le package d’accès, vous pouvez directement attribuer des utilisateurs internes et externes spécifiques à ce package d’accès. Si vous spécifiez un utilisateur externe, un compte d’utilisateur invité est créé dans votre répertoire. Pour plus d’informations sur l’attribution directe d’un utilisateur, voir [Afficher, ajouter et supprimer les attributions pour un package d’accès](entitlement-management-access-package-assignments.md).

1. Passez à la section [Activer les demandes](#enable-requests).


## <a name="open-and-edit-an-existing-policy-of-request-settings"></a>Ouvrir et modifier une stratégie existante de paramètres de demande

Pour modifier les paramètres de demande et d’approbation d’un package d’accès, vous devez ouvrir la stratégie correspondante. Suivez ces étapes pour ouvrir et modifier les paramètres de demande d'un package d'accès :

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Stratégies**, puis sélectionnez la stratégie à modifier.

    Le volet Détails de la stratégie s'ouvre au bas de la page.

    ![Package d’accès - Volet des détails de la stratégie](./media/entitlement-management-shared/policy-details.png)

1. Cliquez sur **Modifier** pour modifier la stratégie.

    ![Package d'accès - Modifier la stratégie](./media/entitlement-management-shared/policy-edit.png)

1. Cliquez sur l'onglet **Demandes** pour ouvrir les paramètres de demande.

1. Suivez les étapes des sections précédentes pour modifier les paramètres de demande en fonction de vos besoins.

1. Accédez à la section [Activer les demandes](#enable-requests).

## <a name="enable-requests"></a>Activer les demandes

1. Si vous souhaitez que le package d’accès soit immédiatement disponible afin d’être demandé par les utilisateurs dans la stratégie de demandes, déplacez le bouton bascule Activer sur **Oui**.

    Vous pouvez toujours l’activer plus tard, après avoir créé le package d’accès.

    Si vous avez sélectionné **Aucun (attributions directes d’administrateur uniquement)** et que vous affectez la valeur **No** à Activer, les administrateurs ne pourront pas attribuer directement ce package d’accès.

    ![Package d’accès - Stratégie - Paramètre Activer la stratégie](./media/entitlement-management-access-package-approval-policy/enable-requests.png)

1. Cliquez sur **Suivant**.

1. Si vous souhaitez exiger des demandeurs qu'ils fournissent des informations supplémentaires lors de la demande d'accès à un package d'accès, suivez les étapes décrites dans []() pour configurer les informations relatives au demandeur (préversion).

1. Configurez les paramètres de cycle de vie.

1. Si vous modifiez une stratégie, cliquez sur **Mettre à jour**. Si vous ajoutez une nouvelle stratégie, cliquez sur **Créer**.

## <a name="next-steps"></a>Étapes suivantes

- [Modifier les paramètres d'approbation d'un package d'accès](entitlement-management-access-package-approval-policy.md)
- [Changer les paramètres de cycle de vie pour un package d’accès](entitlement-management-access-package-lifecycle-policy.md)
- [Afficher les requêtes d’un package d’accès](entitlement-management-access-package-requests.md)
