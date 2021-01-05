---
title: Résoudre les problèmes de gestion des droits d’utilisation – Azure AD
description: Découvrez les points à vérifier pour résoudre les problèmes de gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: bca78195d4fb7bb265c4a7fd8d78ea49a35335b6
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746709"
---
# <a name="troubleshoot-azure-ad-entitlement-management"></a>Résoudre les problèmes de gestion des droits d’utilisation Azure AD

Cet article décrit certains éléments que vous devriez vérifier pour vous aider à résoudre les problèmes de gestion des droits d’utilisation Azure Active Directory (Azure AD).

## <a name="administration"></a>Administration

* Si vous obtenez un message d’accès refusé lors de la configuration de la gestion des droits d’utilisation, et que vous êtes administrateur général, assurez-vous que votre annuaire comporte une [licence Azure AD Premium P2 (ou EMS E5)](entitlement-management-overview.md#license-requirements).

* Si vous obtenez un message d’accès refusé lors de la création ou de la consultation de packages d’accès, et que vous êtes membre d’un groupe Créateur de catalogue, vous devez [créer un catalogue](entitlement-management-catalog-create.md) avant de créer votre premier package d’accès.

## <a name="resources"></a>Ressources

* Les rôles des applications sont définis par l’application elle-même, et gérés dans Azure AD. Si une application n’a pas de rôle de ressource, la gestion des droits d’utilisation affecte aux utilisateurs un rôle **Accès par défaut**.

    Notez que le portail Azure peut également afficher les principaux de service des services qui ne peuvent pas être sélectionnés en tant qu’applications.  Plus particulièrement, **Exchange Online** et **SharePoint Online** sont des services, pas des applications disposant de rôles de ressources dans l’annuaire, ils ne peuvent donc pas être dans un package d’accès.  Utilisez plutôt la gestion des licences par groupe pour établir une licence appropriée, destinée à un utilisateur qui a besoin d’accéder à ces services.

* Pour qu’un groupe soit une ressource dans un package d’accès, il doit pouvoir être modifiable dans Azure AD.  Les groupes issus d’une instance Active Directory locale ne peuvent pas être attribués en tant que ressources, car leurs attributs de propriétaire ou de membre ne sont pas modifiables dans Azure AD.   Les groupes qui proviennent d’Exchange Online en tant que groupes de distribution ne peuvent pas être modifiés dans Azure AD. 

* Les documents individuels et les bibliothèques SharePoint Online ne peuvent pas être ajoutés en tant que ressources.  Créez plutôt un [groupe de sécurité Azure AD](../fundamentals/active-directory-groups-create-azure-portal.md), ajoutez ce groupe et un rôle de site dans le package d’accès, puis utilisez ce groupe dans SharePoint Online pour contrôler l’accès au document ou à la bibliothèque de documents.

* Si des utilisateurs ont déjà été affectés à une ressource que vous souhaitez gérer avec un package d’accès, assurez-vous qu’ils sont affectés au package d’accès avec une stratégie appropriée. Par exemple, vous envisagez d’inclure un groupe dans un package d’accès qui contient déjà des utilisateurs dans le groupe. Si ces utilisateurs dans le groupe nécessitent un accès permanent, ils doivent avoir une stratégie appropriée pour les packages d’accès, afin de ne pas perdre leur accès au groupe. Vous pouvez affecter le package d’accès soit en indiquant aux utilisateurs qu’ils demandent le package d’accès contenant cette ressource, soit en les affectant directement au package d’accès. Pour plus d’informations, consultez [Changer les paramètres de demande et d’approbation pour un package d’accès](entitlement-management-access-package-request-policy.md).

* Lorsque vous supprimez un membre d’une équipe, il est également supprimé du groupe Microsoft 365. La suppression de la fonctionnalité de conversation de l’équipe peut être retardée. Pour plus d’informations, consultez l’article [Appartenance au groupe](/microsoftteams/office-365-groups#group-membership).

* Vérifiez que votre annuaire n’est pas en configuration multigéographique. Actuellement, la gestion des droits d’utilisation ne prend pas en charge les emplacements multigéographiques pour SharePoint Online. Les sites SharePoint Online doivent se trouver à l’emplacement géographique par défaut pour être régis par la gestion des droits d’utilisation. Pour plus d’informations, voir [Fonctionnalités multigéographiques dans OneDrive et SharePoint Online](/Microsoft 365/Enterprise/multi-geo-capabilities-in-onedrive-and-sharepoint-online-in-office-365).

## <a name="access-packages"></a>Packages d’accès

* Si vous tentez de supprimer un package d’accès ou une stratégie et qu’un message d’erreur apparaît, indiquant qu’il existe des affectations actives alors que vous ne voyez aucun utilisateur doté d’affectations, regardez si certains des utilisateurs supprimés récemment possèdent encore des affectations. Pendant la fenêtre de 30 jours qui suit la suppression d’un utilisateur, le compte d’utilisateur peut être restauré.   

## <a name="external-users"></a>Utilisateurs externes

* Lorsqu’un utilisateur externe souhaite demander l’accès à un package d’accès, veillez à ce qu’il utilise le **lien du portail Mon Accès** pour le package. Pour plus d’informations, consultez [Partager le lien pour demander un package d’accès](entitlement-management-access-package-settings.md). Si un utilisateur externe visite simplement **myaccess.microsoft.com** sans utiliser le lien complet du portail Mon Accès, il verra les packages d’accès disponibles dans sa propre organisation et non dans la vôtre.

* Si un utilisateur externe n’est pas en mesure de demander l’accès à un package d’accès ou ne peut pas accéder aux ressources, vérifiez vos [paramètres pour les utilisateurs externes](entitlement-management-external-users.md#settings-for-external-users).

* Si un utilisateur externe, qui ne s’est pas connecté avant à votre annuaire, reçoit un package d’accès, y compris un site SharePoint Online, son package d’accès s’affiche comme n’étant pas entièrement remis tant que son compte ne sera pas provisionné dans SharePoint Online. Pour plus d’informations sur le partage des paramètres, voir [Vérifier les paramètres de partage externe SharePoint Online](entitlement-management-external-users.md#review-your-sharepoint-online-external-sharing-settings).

## <a name="requests"></a>Demandes

* Lorsqu’un utilisateur souhaite demander l’accès à un package d’accès, assurez-vous qu’il utilise le **lien du portail Mon accès** pour le package d’accès. Pour plus d’informations, consultez [Partager le lien pour demander un package d’accès](entitlement-management-access-package-settings.md).

* Si vous ouvrez le portail Mon Accès avec votre navigateur en mode privé ou incognito, il peut se produire un conflit avec le comportement de connexion. Nous vous recommandons de ne pas utiliser le mode privé ou incognito de votre navigateur lorsque vous visitez le portail Mon Accès.

* Lorsqu’un utilisateur qui ne figure pas encore dans votre annuaire se connecte au portail Mon Accès pour demander un package d’accès, assurez-vous qu’il s’authentifie à l’aide de son compte professionnel ou scolaire. Le compte professionnel ou scolaire peut être un compte présent dans l’annuaire de ressources, ou dans un annuaire qui est inclus dans l’une des stratégies du package d’accès. Si le compte de l’utilisateur n’est pas un compte professionnel ou scolaire, ou si l’annuaire auquel il s’authentifie n’est pas inclus dans la stratégie, l’utilisateur ne voit pas le package d’accès. Pour plus d’informations, consultez [Demander l’accès à un package d’accès](entitlement-management-request-access.md).

* Si un utilisateur est empêché de se connecter à l’annuaire de ressources, il est dans l’incapacité de demander l’accès dans le portail Mon accès. Avant que l’utilisateur puisse demander l’accès, vous devez supprimer le bloc de connexion du profil de l’utilisateur. Pour supprimer le bloc de connexion, dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Utilisateurs** ; cliquez ensuite sur l’utilisateur concerné, puis sur **Profil**. Modifiez la section **Paramètres** et remplacez **Bloquer la connexion** par **Non**. Pour plus d’informations, consultez [Ajouter ou mettre à jour les informations du profil utilisateur avec Azure Active Directory](../fundamentals/active-directory-users-profile-azure-portal.md).  Vous pouvez également vérifier si l’utilisateur a été bloqué à cause d’une [Stratégie dIdentity Protection](../identity-protection/howto-identity-protection-remediate-unblock.md).

* Dans le portail Mon accès, si un utilisateur est en même temps demandeur et approbateur, il ne peut pas voir sa demande pour un package d’accès dans la page **Approbations**. Ce comportement est intentionnel : un utilisateur ne peut pas approuver sa propre demande. Vérifiez que le package d’accès demandé compte des approbateurs supplémentaires configurés sur la stratégie. Pour plus d’informations, consultez [Changer les paramètres de demande et d’approbation pour un package d’accès](entitlement-management-access-package-request-policy.md).

### <a name="view-a-requests-delivery-errors"></a>Afficher les erreurs de remise d’une requête

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Demandes**.

1. Sélectionnez la requête à afficher.

    Si la requête contient des erreurs de remise, l’état de la demande sera **Non remis** ou **Partiellement remis**.

    Le cas échéant, un décompte des erreurs de livraison sera affiché dans le volet Détails de la demande.

1. Cliquez sur ce nombre pour voir toutes les erreurs de livraison de la requête.

### <a name="reprocess-a-request"></a>Retraiter une demande

Si une erreur est rencontrée après le déclenchement d’une demande de retraitement d’un package d’accès, vous devez patienter pendant que le système retraite la demande. Le système essaie de retraiter à plusieurs reprises et sur une période de plusieurs heures, vous ne pouvez donc pas forcer le retraitement pendant cette période. 

Vous ne pouvez retraiter qu’une demande dont l’état est **Échec de livraison** ou **Partiellement remis** avec une date de fin inférieure à une semaine. Dans le cas contraire, le bouton **Retraiter** est grisé.

![Bouton Retraiter grisé](./media/entitlement-management-troubleshoot/cancel-reprocess-grayedout.png)

- Si l’erreur est corrigée dans la fenêtre des essais, l’état de la demande passe à **Livraison en cours**. La demande sera retraitée sans aucune action supplémentaire de la part de l’utilisateur.

- Si l’erreur n’a pas été corrigée dans la fenêtre des essais, l’état de la demande peut être **Échec de livraison** ou **Partiellement remis**. Vous pouvez ensuite utiliser le bouton **Retraiter**. Vous aurez sept jours pour retraiter la demande.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Demandes**.

1. Cliquez sur la demande à retraiter.

1. Dans le volet des détails de la requête, cliquez sur **Retraiter la demande**.

    ![Retraiter une demande ayant échoué](./media/entitlement-management-troubleshoot/reprocess-request.png)

### <a name="cancel-a-pending-request"></a>Annuler une demande en attente

Vous pouvez uniquement annuler une requête en attente qui n’a pas encore été autorisée ou dont la livraison a échoué. Dans le cas contraire, le bouton **Annuler** est grisé.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Demandes**.

1. Cliquez sur la requête à annuler.

1. Dans le volet des détails de la requête, cliquez sur **Annuler la demande**.

## <a name="multiple-policies"></a>Stratégies multiples

* La gestion des droits d’utilisation suit les meilleures pratiques des privilèges minimum. Lorsqu’un utilisateur demande l’accès à un package d’accès auquel plusieurs stratégies s’appliquent, la gestion des droits d’utilisation comporte une logique qui permet de garantir que les stratégies plus strictes ou plus spécifiques sont prioritaires par rapport aux stratégies génériques. Si une stratégie est générique, la gestion des droits d’utilisation est susceptible de ne pas la présenter au demandeur ou d’en sélectionner automatiquement une plus stricte.

* Prenons par exemple un package d’accès comportant deux stratégies pour les employés internes, qui s’appliquent toutes deux au demandeur. La première concerne des utilisateurs spécifiques, dont fait partie le demandeur. La deuxième porte sur tous les utilisateurs d’un répertoire dont le demandeur est membre. Dans ce scénario, la première stratégie est automatiquement sélectionnée pour le demandeur, car elle est plus stricte. Le demandeur n’a pas la possibilité de sélectionner la deuxième.

* Lorsque plusieurs stratégies s’appliquent, la stratégie sélectionnée automatiquement ou les stratégies présentées au demandeur suivent la logique de priorité suivante :

    | Priorité de la stratégie | Étendue |
    | --- | --- |
    | P1 | Utilisateurs et groupes spécifiques du répertoire OU Organisations connectées spécifiques |
    | P2 | Tous les membres du répertoire (à l’exception des invités) |
    | P3 | Tous les utilisateurs du répertoire (y compris les invités) OU Organisations connectées spécifiques |
    | P4 | Toutes les organisations connectées configurées OU Tous les utilisateurs (toutes les organisations connectées + tous les nouveaux utilisateurs externes) |
    
    Si une stratégie se trouve dans une catégorie de priorité plus élevée, les catégories de priorité inférieure sont ignorées. Pour savoir par le biais d’un exemple comment plusieurs stratégies de même priorité sont présentées au demandeur, voir [Sélectionner une stratégie](entitlement-management-request-access.md#select-a-policy).

## <a name="next-steps"></a>Étapes suivantes

- [Régir l’accès des utilisateurs externes](entitlement-management-external-users.md)
- [Afficher les rapports sur la façon dont les utilisateurs ont eu accès à la gestion des droits d’utilisation](entitlement-management-reports.md)