---
title: 'Didacticiel : configurer Box pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Box.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: jeedes
ms.openlocfilehash: e22738f1fff813e5a928b76f8049e810847fe548
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358148"
---
# <a name="tutorial-configure-box-for-automatic-user-provisioning"></a>Didacticiel : configurer Box pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans Box et Azure AD pour approvisionner automatiquement des comptes utilisateur Azure AD dans Box, ainsi que pour annuler leur approvisionnement.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Box, vous avez besoin des éléments suivants :

- un locataire Azure AD ;
- une offre Box pour les entreprises ou mieux.

> [!NOTE]
> Lorsque vous testez les étapes de ce didacticiel, nous *déconseillons* l’utilisation d’un environnement de production.

> [!NOTE]
> Les applications doivent d’abord être activées dans l’application Box.

Pour tester la procédure de ce didacticiel, suivez les recommandations ci-dessous :

- N’utilisez pas votre environnement de production, sauf si cela est nécessaire.
- Si vous n’avez pas d’environnement d’essai Azure AD, vous pouvez [obtenir un essai d’un mois](https://azure.microsoft.com/pricing/free-trial/).

## <a name="assigning-users-to-box"></a>Attribution d’utilisateurs à Box 

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique des comptes d’utilisateur, seuls les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à votre application Box. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs à votre application Box en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="assign-users-and-groups"></a>Affecter des utilisateurs et des groupes
L’onglet **Box > Utilisateurs et groupes** du portail Azure vous permet de spécifier quels utilisateurs et groupes doivent avoir accès à Box. L'affectation d'un utilisateur ou groupe entraîne les événements suivants :

* Azure AD permet à l'utilisateur affecté (par affectation directe ou appartenance au groupe) de s'identifier auprès de Box. Si un utilisateur n’est pas attribué, Azure AD ne lui permet pas de se connecter à Box et retourne une erreur dans la page de connexion Azure AD.
* Une mosaïque d'application pour Box est ajoutée au [lanceur d'applications](../manage-apps/end-user-experiences.md)de cet utilisateur.
* Si la configuration automatique est activée, les utilisateurs et/ou groupes affectés sont ajoutés à la file d'attente d'approvisionnement automatique.
  
  * Si seuls des objets utilisateur ont été configurés pour être attribués, tous les utilisateurs directement assignés sont placés dans la file d’attente d’attribution et tous les utilisateurs membres des groupes assignés sont placés dans la file d’attente d’attribution. 
  * Si des objets de groupe ont été configurés pour être attribués, tous les objets du groupe affectés sont attribués dans Box, tout comme l’ensemble des utilisateurs membres de ces groupes. Les appartenances des groupes et des utilisateurs sont conservées lors de l'écriture dans Box.

Vous pouvez utiliser l'onglet **Attributs > Authentification unique** pour configurer les attributs (ou revendications) de l'utilisateur présentés dans Box pendant l'authentification SAML, et l'onglet **Attributs > Approvisionnement** pour configurer le transfert des attributs utilisateur et groupe d'Azure AD vers Box lors des opérations d'approvisionnement.

### <a name="important-tips-for-assigning-users-to-box"></a>Conseils importants pour l’attribution d’utilisateurs à Box 

*   Dans un premier temps, il est recommandé de n’assigner qu’un seul utilisateur Azure AD à Box pour tester la configuration de l’attribution. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

*   Quand vous assignez un utilisateur à Box, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

## <a name="enable-automated-user-provisioning"></a>Activer l’approvisionnement automatique des utilisateurs

Cette section explique comment connecter Azure AD à l’API d’approvisionnement de comptes d’utilisateur de Box pour créer, mettre à jour et désactiver les comptes d’utilisateur attribués dans Box, en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

Si la configuration automatique est activée, les utilisateurs et/ou groupes affectés sont ajoutés à la file d'attente d'approvisionnement automatique.
    
 * Si seuls des objets utilisateur ont été configurés pour être attribués, les utilisateurs directement assignés sont placés dans la file d’attente d’attribution et tous les utilisateurs membres des groupes assignés sont placés dans la file d’attente d’attribution. 
    
 * Si des objets de groupe ont été configurés pour être attribués, tous les objets du groupe affectés sont attribués dans Box, tout comme l’ensemble des utilisateurs membres de ces groupes. Les appartenances des groupes et des utilisateurs sont conservées lors de l'écriture dans Box.

> [!TIP] 
> Vous pouvez également choisir d’activer l’authentification unique SAML pour Box en suivant les instructions fournies dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="to-configure-automatic-user-account-provisioning"></a>Pour configurer l’approvisionnement automatique des comptes d’utilisateur :

Cette section décrit comment activer l'approvisionnement des comptes d'utilisateurs Active Directory sur Box.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Si vous avez déjà configuré Box pour l’authentification unique, recherchez votre instance de Box à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Box** dans la galerie d’applications. Sélectionnez Box dans les résultats de la recherche, puis ajoutez-le à votre liste d’applications.

3. Sélectionnez votre instance de Box, puis sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**. 

    ![Capture d’écran de l’onglet Approvisionnement pour Box dans le portail Azure. Le mode d’approvisionnement est défini sur Automatique et le bouton Autorisation est mis en évidence dans Informations d’identification de l’administrateur.](./media/box-userprovisioning-tutorial/provisioning.png)

5. Sous la section **Informations d’identification de l’administrateur** , cliquez sur **Autoriser** pour ouvrir la boîte de dialogue de connexion à Box dans une nouvelle fenêtre de navigateur.

6. Dans la page **Login to grant access to Box** (Se connecter pour autoriser l’accès à Box), saisissez les informations d’identification, puis cliquez sur **Authorize** (Autoriser). 
   
    ![Screenshot of the Log in to grant access to box screen, showing entry for Email and Password, and the Authorize button.](./media/box-userprovisioning-tutorial/IC769546.png "Activer l'approvisionnement de l'utilisateur automatique.")

7. Cliquez sur **Grant access to Box** (Accorder l’accès à Box) pour autoriser cette opération et retourner au portail Azure. 
   
    ![Screenshot of the authorize access screen in Box, showing an explanatory message and the Grant access to Box button.](./media/box-userprovisioning-tutorial/IC769549.png "Activer l'approvisionnement de l'utilisateur automatique.")

8. Dans le portail Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application Box. Si la connexion échoue, vérifiez que votre compte Box dispose des autorisations d’administrateur d’équipe et recommencez l’étape **Autoriser**.

9. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case.

10. Cliquez sur **Enregistrer.**

11. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Box**.

12. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Box. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur Box en vue de mises à jour ultérieures. Cliquez sur le bouton Enregistrer pour valider les modifications.

13. Pour activer le service d’approvisionnement Azure AD pour Box, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section Paramètres.

14. Cliquez sur **Enregistrer.**

Cette commande démarre la synchronisation initiale des utilisateurs et/ou des groupes affectés à Box dans la section Utilisateurs et Groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application Box.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

Dans votre locataire Box, les utilisateurs synchronisés sont répertoriés sous **Utilisateurs gérés** dans la **Console d’administration**.

![Statut d’intégration](./media/box-userprovisioning-tutorial/IC769556.png "Statut d’intégration")


## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](box-tutorial.md)