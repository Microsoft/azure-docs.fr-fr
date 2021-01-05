---
title: FAQ sur la résolution des problèmes liés aux rôles attribués aux groupes cloud – Azure Active Directory | Microsoft Docs
description: Découvrez quelques questions courantes et des conseils de dépannage pour l’attribution de rôles à des groupes dans Azure Active Directory.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8735a0d34b9fcf5b86b6592980ffc5c7c3e3073c
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861933"
---
# <a name="troubleshooting-roles-assigned-to-cloud-groups"></a>Résoudre les problèmes de rôles attribués aux groupes cloud

Voici quelques questions courantes et des conseils de résolution des problèmes pour l’attribution de rôles à des groupes dans Azure Active Directory (Azure AD).

**Q :** Je suis administrateur de groupes, mais je ne vois pas le commutateur **Des rôles Azure AD peuvent être attribués au groupe**.

**R :** Seuls les Administrateurs de rôle privilégié ou les Administrateurs généraux peuvent créer un groupe éligible pour l’attribution de rôle. Seuls les utilisateurs de ces rôles voient ce contrôle.

**Q :** Qui peut modifier l’appartenance des groupes qui sont affectés à des rôles Azure AD ?

**R :** Par défaut, seul l’Administrateur de rôle privilégié ou l’Administrateur général peut gérer l’appartenance à un groupe assignable à un rôle, mais vous pouvez déléguer la gestion des groupes assignables à un rôle en leur ajoutant des propriétaires de groupe.

**Q** : Je suis Administrateur du support technique dans mon organisation, mais je ne peux pas mettre à jour le mot de passe d’un utilisateur ayant le rôle Lecteur de répertoire. Pourquoi cela se produit-il ?

**R** : L’utilisateur peut avoir obtenu le rôle Lecteur de répertoire par le biais d’un groupe assignable à un rôle. Tous les membres et propriétaires d’un groupe assignable à un rôle sont protégés. Seuls les utilisateurs ayant les rôles Administrateur d’authentification privilégiée ou Administrateur général peuvent réinitialiser les informations d’identification d’un utilisateur protégé.

**Q :** Je ne parviens pas à mettre à jour le mot de passe d’un utilisateur. Aucun rôle privilégié plus élevé ne lui est attribué. Pourquoi cela se passe-t-il ?

**R :** L’utilisateur peut être propriétaire d’un groupe assignable à un rôle. Nous protégeons les propriétaires de ces groupes afin d’empêcher une élévation de privilèges. Prenons l’exemple du groupe Contoso_Security_Admins auquel attribué au rôle Administrateur de la sécurité, et où Bob est le propriétaire du groupe et Alice l’Administrateur de mot de passe dans l’organisation. Si cette protection n’était pas présente, Alice pourrait réinitialiser les informations d’identification de Bob et prendre le contrôle de son identité. Ensuite, Alice pourrait s’ajouter elle-même ou toute autre personne au groupe Contoso_Security_Admins pour devenir Administrateur de la sécurité dans l’organisation. Pour déterminer si un utilisateur est propriétaire d’un groupe, obtenez la liste des objets détenus de cet utilisateur et vérifiez si isAssignableToRole a la valeur true pour l’un des groupes. Si c’est le cas, cet utilisateur est protégé et le comportement est normal. Reportez-vous à ces documents pour obtenir les objets détenus :

- [Get-AzureADUserOwnedObject](/powershell/module/azuread/get-azureaduserownedobject)  
- [List ownedObjects](/graph/api/user-list-ownedobjects?tabs=http)

**Q :** Puis-je créer une révision d’accès sur des groupes qui peuvent être affectés à des rôles Azure AD (plus précisément, les groupes dont la propriété isAssignableToRole est définie sur true) ?  

**R :** Oui, vous pouvez. Si vous utilisez la version la plus récente de la révision d’accès, vos réviseurs sont dirigés vers Mon Accès par défaut, et seuls les Administrateurs généraux peuvent créer des révisions d’accès sur des groupes assignables à un rôle. Toutefois, si vous utilisez une version antérieure de la révision d’accès, vos réviseurs sont dirigés par défaut vers le volet d’accès, et les Administrateurs généraux et les Administrateurs d’utilisateurs peuvent créer des révisions d’accès sur des groupes assignables à un rôle. La nouvelle expérience sera déployée pour tous les clients le 28 juillet 2020, mais, si vous souhaitez procéder à une mise à niveau plus tôt, effectuez une demande sur la page [Révisions d’accès Azure AD – Expérience du réviseur mise à jour dans Inscription à Mon Accès](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

**Q :** Puis-je créer un package d’accès et y placer des groupes qui peuvent être affectés à des rôles Azure AD ?

**R :** Oui, vous pouvez. L’administrateur général et les administrateurs d’utilisateurs ont la possibilité de placer n’importe quel groupe dans un package d’accès. Rien ne change pour l’administrateur général, mais il y a une légère modification des autorisations du rôle Administrateur d’utilisateurs. Pour placer un groupe assignable à un rôle dans un package d’accès, vous devez être un administrateur d’utilisateurs et également le propriétaire du groupe en question. Voici le tableau complet qui indique qui peut créer un package d’accès dans la gestion des licences Entreprise :

Rôle d’annuaire Azure AD | Rôle de gestion des droits d’utilisation | Peut ajouter un groupe de sécurité\* | Peut ajouter un groupe Microsoft 365\* | Peut ajouter une application | Peut ajouter un site SharePoint Online
----------------------- | --------------------------- | ----------------------- | ------------------------- | ----------- |  -----------------------------
Administrateur général | n/a | ✔️ | ✔️ | ✔️  | ✔️
Administrateur d’utilisateurs  | n/a  | ✔️  | ✔️  | ✔️
Administrateur Intune | Propriétaire de catalogue | ✔️  | ✔️  | &nbsp;  | &nbsp;
Administrateur Exchange  | Propriétaire de catalogue  | &nbsp; | ✔️  | &nbsp;  | &nbsp;
Administrateur du service Teams | Propriétaire de catalogue  | &nbsp; | ✔️  | &nbsp;  | &nbsp;
Administrateur SharePoint | Propriétaire de catalogue | &nbsp; | ✔️  | &nbsp;  | ✔️ 
Administrateur d’application | Propriétaire de catalogue  | &nbsp;  | &nbsp; | ✔️  | &nbsp;
Administrateur d’application cloud | Propriétaire de catalogue  | &nbsp;  | &nbsp; | ✔️  | &nbsp;
Utilisateur | Propriétaire de catalogue | Seulement si propriétaire d’un groupe | Seulement si propriétaire d’un groupe | Seulement si propriétaire d’une application  | &nbsp;

\* Le groupe n’est pas assignable à un rôle ; autrement dit, isAssignableToRole = false. Si un groupe est assignable à un rôle, la personne qui crée le package d’accès doit également être propriétaire du groupe en question.

**Q :** Je ne trouve pas l’option « Supprimer l’attribution » dans « Rôles attribués ». Comment supprimer l’attribution de rôle à un utilisateur ?

**R :** Cette réponse s’applique uniquement aux organisations Azure AD Premium P1.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez **Azure Active Directory**.
1. Sélectionnez des utilisateurs et ouvrez un profil utilisateur.
1. Sélectionnez **Rôles attribués**.
1. Sélectionnez l’icône d’engrenage. Un volet s’ouvre et peut fournir ces informations. Il y a un bouton « Supprimer » à côté des attributions directes. Pour supprimer une attribution de rôle indirecte, supprimez l’utilisateur du groupe auquel le rôle a été attribué.

**Q :** Comment puis-je voir tous les groupes assignables à un rôle ?

**R :** Procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez **Azure Active Directory**.
1. Sélectionnez **Groupes** > **Tous les groupes**.
1. Sélectionnez **Ajouter des filtres**.
1. Filtrez sur **Assignable à un rôle**.

**Q :** Comment savoir quel rôle est attribué directement ou indirectement à un principal ?

**R :** Procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez **Azure Active Directory**.
1. Sélectionnez des utilisateurs et ouvrez un profil utilisateur.
1. Sélectionnez **Rôles attribués**, puis :

    - Dans les organisations sous licence Azure AD Premium P1 : Sélectionnez l’icône d’engrenage. Un volet s’ouvre et peut fournir ces informations.
    - Dans les organisations sous licence Azure AD Premium P2 : Vous trouverez des informations sur les licences directes et héritées dans la colonne **Appartenance**.

**Q :** Pourquoi la création d’un nouveau groupe cloud est-elle obligatoire pour l’affecter à un rôle ?  

**R :** Si vous affectez un groupe existant à un rôle, le propriétaire du groupe existant peut ajouter d’autres membres à ce groupe sans que les nouveaux membres ne se rendent compte qu’ils recevront le rôle. Parce que les groupes assignables à un rôle sont puissants, nous plaçons de nombreuses restrictions pour les protéger. Vous ne souhaitez pas que le groupe subisse des modifications qui surprendraient la personne qui le gère.

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des groupes cloud pour gérer les attributions de rôles](groups-concept.md)
- [Créer un groupe assignable à un rôle](groups-create-eligible.md)
