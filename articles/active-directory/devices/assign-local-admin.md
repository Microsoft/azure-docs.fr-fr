---
title: Guide pratique pour gérer les administrateurs locaux sur les appareils joints à Azure AD
description: Découvrez comment attribuer des rôles Azure au groupe Administrateurs local d’un appareil Windows.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: ravenn
ms.collection: M365-identity-device-management
ms.openlocfilehash: cfd7b5ac981fcb87d0fc929d944205dec9432b74
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575820"
---
# <a name="how-to-manage-the-local-administrators-group-on-azure-ad-joined-devices"></a>Guide pratique pour gérer le groupe Administrateurs local sur des appareils joints à Azure AD

Pour gérer un appareil Windows, vous devez être membre du groupe Administrateurs local. Dans le cadre du processus de jonction à Azure Active Directory (Azure AD), Azure AD met à jour l’appartenance de ce groupe sur un appareil. Vous pouvez personnaliser la mise à jour de l’appartenance pour répondre aux besoins de votre entreprise. Une mise à jour de l’appartenance peut par exemple être utile si vous souhaitez autoriser le personnel du support technique à effectuer des tâches nécessitant des droits d’administrateur sur un appareil.

Cet article explique comment la mise à jour de l’appartenance aux administrateurs locaux fonctionne et comment la personnaliser pendant une jonction à Azure AD. Le contenu de cet article ne s’applique pas à des appareils **joints Azure AD hybride**.

## <a name="how-it-works"></a>Fonctionnement

Quand vous connectez un appareil Windows avec Azure AD à l’aide d’une jonction à Azure AD, Azure AD ajoute les principaux de sécurité suivants au groupe Administrateurs local sur l’appareil :

- Rôle Administrateur général Azure AD
- Rôle Administrateur d’appareil Azure AD 
- Utilisateur effectuant la jonction à Azure AD   

En ajoutant des rôles Azure AD au groupe Administrateurs local, vous pouvez mettre à jour les utilisateurs qui peuvent gérer un appareil à tout moment dans Azure AD, sans modifier quoi que ce soit sur l’appareil. À l’heure actuelle, vous ne pouvez pas affecter de groupes à un rôle d’administrateur.
Azure AD ajoute également le rôle d’administrateur d’appareil Azure AD au groupe Administrateurs local pour prendre en charge le principe des privilèges minimum. En plus des administrateurs généraux, vous pouvez également autoriser les utilisateurs ayant *uniquement* reçu le rôle d’administrateur d’appareil à gérer un appareil. 

## <a name="manage-the-global-administrators-role"></a>Gérer le rôle Administrateur général

Pour afficher et mettre à jour l’appartenance du rôle Administrateur général, consultez :

- [Afficher tous les membres d’un rôle d’administrateur dans Azure Active Directory](../roles/manage-roles-portal.md)
- [Attribuer des rôles d’administrateur à un utilisateur dans Azure Active Directory](../fundamentals/active-directory-users-assign-role-azure-portal.md)


## <a name="manage-the-device-administrator-role"></a>Gérer le rôle Administrateur d’appareil 

Dans le portail Azure, vous pouvez gérer le rôle d’administrateur d’appareil dans la page **Appareils**. Pour ouvrir la page **Appareils** :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général.
1. Recherchez et sélectionnez *Azure Active Directory*.
1. Dans la section **Gérer**, cliquez sur **Appareils**.
1. Dans la page **Appareils**, cliquez sur **Paramètres de l’appareil**.

Pour modifier le rôle d’administrateur d’appareils, configurez **Administrateurs locaux supplémentaires sur les appareils joints à Azure AD**.  

![Administrateurs locaux supplémentaires](./media/assign-local-admin/10.png)

>[!NOTE]
> Cette option nécessite un locataire Azure AD Premium. 

Les administrateurs d’appareil sont affectés à toutes les appareils joints à Azure AD. Vous ne pouvez pas limiter l’étendue des administrateurs d’appareil à un jeu spécifique d’appareils. La mise à jour du rôle d’administrateur d’appareil n’a pas nécessairement un impact immédiat sur les utilisateurs affectés. Sur les appareils où un utilisateur est déjà connecté, l’élévation des privilèges a lieu lorsque les *deux* actions ci-dessous se produisent :

- Jusqu’à 4 heures ont passé pour qu’Azure AD émette un nouveau jeton d’actualisation principal avec les privilèges appropriés. 
- L’utilisateur se déconnecte, puis se reconnecte, sans verrouillage/déverrouillage, pour actualiser son profil.

>[!NOTE]
> Les actions ci-dessus ne s’appliquent pas aux utilisateurs qui ne se sont pas connectés précédemment à l’appareil approprié. Dans ce cas, les privilèges d’administrateur sont appliqués immédiatement après leur première connexion à l’appareil. 

## <a name="manage-administrator-privileges-using-azure-ad-groups-preview"></a>Gérer les privilèges d’administrateur à l’aide de groupes Azure AD (préversion)

>[!NOTE]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire.


À compter de la mise à jour Windows 10 2004, vous pouvez utiliser des groupes Azure AD pour gérer les privilèges d’administrateur sur les appareils joints à Azure AD à l’aide de la stratégie MDM [Groupes restreints](/windows/client-management/mdm/policy-csp-restrictedgroups). Cette stratégie vous permet d’affecter des utilisateurs individuels ou des groupes Azure AD au groupe Administrateurs local sur un appareil joint à Azure AD. Vous disposez ainsi de la précision nécessaire pour configurer des administrateurs distincts en fonction des différents groupes d’appareils. 

>[!NOTE]
> À partir de la mise à jour 20H2 de Windows 10, nous vous recommandons d’utiliser la stratégie [Utilisateurs et groupes locaux](/windows/client-management/mdm/policy-csp-localusersandgroups) plutôt que la stratégie Groupes restreints


Il n’existe pas d’IU dans Intune permettant de gérer ces stratégies, qui doivent être configurées à l’aide de [paramètres OMA-URI personnalisés](/mem/intune/configuration/custom-settings-windows-10). Voici quelques considérations à prendre en compte pour utiliser l’une de ces stratégies : 

- L’ajout de groupes Azure AD via la stratégie nécessite le SID du groupe, lequel peut être obtenu par l’exécution de l’[API Microsoft Graph API pour les groupes](/graph/api/resources/group?view=graph-rest-beta). Le SID est défini par la propriété `securityIdentifier` dans la réponse de l’API.
- Quand la stratégie Groupes restreints est appliquée, tout membre actuel du groupe qui ne figure pas dans la liste Membres est supprimé. Ainsi, l’application de cette stratégie à de nouveaux membres ou groupes entraîne la suppression des administrateurs existants, à savoir l’utilisateur qui a joint l’appareil, le rôle Administrateur et le rôle Administrateur général de l’appareil. Pour éviter de supprimer des membres existants, vous devez les configurer dans le cadre de la liste Membres de la stratégie Groupes restreints. Cette limitation est traitée si vous utilisez la stratégie Utilisateurs et groupes locaux qui autorise les mises à jour incrémentielles de l’appartenance à un groupe.
- Les privilèges d’administrateur utilisant ces deux stratégies sont uniquement évalués pour les groupes bien connus suivants sur un appareil Windows 10 : Administrateurs, Utilisateurs, Invités, Utilisateurs avec pouvoir, Utilisateurs du Bureau à distance et Utilisateurs de gestion à distance. 
- La gestion des administrateurs locaux à l’aide des groupes Azure AD ne s’applique pas aux appareils faisant l’objet d’une jointure Azure AD Hybride ou aux appareils inscrits auprès d’Azure AD.
- Même si la stratégie Groupes restreints existait avant la mise à jour Windows 10 2004, elle ne prenait pas en charge les groupes Azure AD en tant que membres du groupe Administrateurs local d’un appareil. 

## <a name="manage-regular-users"></a>Gérer les utilisateurs réguliers

Par défaut, Azure AD ajoute l’utilisateur qui effectue la jonction à Azure AD au groupe Administrateurs sur l’appareil. Si vous souhaitez empêcher les utilisateurs réguliers de devenir des administrateurs locaux, vous disposez des options suivantes :

- [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot) : Windows Autopilot offre une option permettant d’empêcher l’utilisateur principal qui effectue la jonction de devenir un administrateur local. Pour cela, vous pouvez [créer un profil Autopilot](/intune/enrollment-autopilot#create-an-autopilot-deployment-profile).
- [Inscription en bloc](/intune/windows-bulk-enroll) : une jonction à Azure AD qui est effectuée dans le contexte d’une inscription en bloc se produit dans le contexte d’un utilisateur créé automatiquement. Les utilisateurs qui se connectent après la jonction d’un appareil ne sont pas ajoutés au groupe Administrateurs.   

## <a name="manually-elevate-a-user-on-a-device"></a>Élever manuellement un utilisateur sur un appareil 

Outre l’utilisation du processus de jonction à Azure AD, vous pouvez également élever manuellement un utilisateur régulier pour qu’il devienne un administrateur local sur un appareil spécifique. Cette étape nécessite que vous soyez déjà membre du groupe Administrateurs local. 

À compter de la version **Windows 10 1709**, vous pouvez effectuer cette tâche depuis **Paramètres -> Comptes -> Autres utilisateurs**. Sélectionnez **Ajouter un utilisateur professionnel ou scolaire**, entrez l’UPN de l’utilisateur sous **Compte d’utilisateur**, puis sélectionnez *Administrateur* sous **type de compte**  
 
Vous pouvez également ajouter des utilisateurs à l’invite de commandes :

- Si vos utilisateurs locataires sont synchronisés à partir d’Active Directory en local, utilisez `net localgroup administrators /add "Contoso\username"`.
- Si vos utilisateurs locataires sont créés dans Azure AD, utilisez `net localgroup administrators /add "AzureAD\UserUpn"`.

## <a name="considerations"></a>Considérations 

Vous ne pouvez pas affecter de groupes au rôle d’administrateur d’appareil. Seuls les utilisateurs individuels sont autorisés.

Les administrateurs d’appareil sont affectés à tous les appareils joints à Azure AD. Ils ne peuvent pas être limités à un jeu spécifique d’appareils.

Quand vous supprimez des utilisateurs du rôle Administrateur d’appareil, ils disposent toujours du privilège d’administrateur local sur un appareil tant qu’ils sont connectés à celui-ci. Le privilège est révoqué la prochaine fois qu’ils se connectent quand un nouveau jeton d’actualisation principal est émis. Cette révocation, similaire à l’élévation des privilèges, peut prendre jusqu’à 4 heures.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble de la gestion des appareils dans le portail Azure, consultez [Gestion des appareils via le portail Azure](device-management-azure-portal.md).
- Pour en savoir plus sur l’accès conditionnel basé sur les appareils, consultez [Configurer les stratégies d’accès conditionnel basé sur les appareils Azure Active Directory](../conditional-access/require-managed-devices.md).
