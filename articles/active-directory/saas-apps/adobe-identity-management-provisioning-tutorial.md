---
title: 'Tutoriel : Configurer Adobe Identity Management pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment procéder automatiquement à l’approvisionnement et au déprovisionnement des comptes d’utilisateur d’Azure AD vers Adobe Identity Management.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 6ae05dc7-1265-44b4-a20c-512b5218b9d1
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2020
ms.author: Zhchia
ms.openlocfilehash: aafdbe631426b4b0c8c762a81457369bc474f479
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181120"
---
# <a name="tutorial-configure-adobe-identity-management-for-automatic-user-provisioning"></a>Tutoriel : Configurer Adobe Identity Management pour l’approvisionnement automatique d’utilisateurs

Ce tutoriel décrit les étapes à suivre dans Adobe Identity Management et Azure Active Directory (Azure AD) pour configurer l’approvisionnement automatique d’utilisateurs. Une fois la configuration effectuée, Azure AD approvisionne et déprovisionne automatiquement les utilisateurs et les groupes dans Adobe Identity Management à l’aide du service d’approvisionnement d’Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Adobe Identity Management
> * Supprimer des utilisateurs dans Adobe Identity Management quand ils n’ont plus besoin d’y accéder
> * Conserver les attributs d’utilisateur synchronisés entre Azure AD et Adobe Identity Management
> * Approvisionner des groupes et des appartenances de groupe dans Adobe Identity Management
> * Authentification unique à Adobe Identity Management (recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md).
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, administrateur d’application, administrateur d’application Cloud, propriétaire d’application ou administrateur général). 
* Un répertoire fédéré dans la [console d’administration Adobe](https://adminconsole.adobe.com/) avec des domaines vérifiés.

> [!NOTE]
> Si votre organisation utilise l’outil de synchronisation utilisateur ou une intégration UMAPI, vous devez d’abord suspendre l’intégration. Ensuite, ajoutez l’approvisionnement automatique d’Azure AD pour automatiser la gestion des utilisateurs à partir du portail Azure. Une fois l’approvisionnement automatique d’Azure AD configuré et en cours d’exécution, vous pouvez supprimer complètement l’outil de synchronisation utilisateur ou l’intégration UMAPI.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Adobe Identity Management](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-adobe-identity-management-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Adobe Identity Management pour prendre en charge l’approvisionnement avec Azure AD

1. Connectez-vous à la console [Adobe Admin Console](https://adminconsole.adobe.com/). Accédez à **Settings > Directory Details > Sync** (Paramètres > Détails de répertoire > Synchroniser). 

2. Cliquez sur **Add Sync** (Ajouter une synchronisation).

      ![Ajouter](media/adobe-identity-management-provisioning-tutorial/add-sync.png)

3. Sélectionnez **Sync users from Microsoft Azure** (Synchroniser les utilisateurs à partir de Microsoft Azure) et cliquez sur **Next** (Suivant).

      ![Synchronisation](media/adobe-identity-management-provisioning-tutorial/sync-users.png)

4. Copiez et enregistrez l’**URL de locataire** et le **jeton secret**. Ces valeurs doivent être entrées dans les champs **URL de locataire** et **Jeton secret** de l’onglet Approvisionnement, dans votre application Adobe Identity Management sur le portail Azure.

      ![Synchronisation](media/adobe-identity-management-provisioning-tutorial/token.png)

## <a name="step-3-add-adobe-identity-management-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Adobe Identity Management à partir de la galerie d’applications d’Azure AD

Ajoutez Adobe Identity Management à partir de la galerie d’applications d’Azure AD pour commencer à gérer l’approvisionnement dans Adobe Identity Management. Si vous avez déjà configuré Adobe Identity Management pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Quand vous attribuez des utilisateurs et des groupes à Adobe Identity Management, vous devez sélectionner un rôle différent du rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-adobe-identity-management"></a>Étape 5. Configurer l’approvisionnement automatique d’utilisateurs pour Adobe Identity Management 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-adobe-identity-management-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Adobe Identity Management dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Adobe Identity Management**.

    ![Le lien Adobe Identity Management dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Approvisionnement](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Onglet Provisionnement automatique](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification de l’administrateur**, entrez l’URL de votre locataire Adobe Identity Management et le jeton secret récupéré précédemment à l’étape 2. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Adobe Identity Management. Si la connexion échoue, vérifiez que votre compte Adobe Identity Management dispose des autorisations d’administrateur, puis réessayez.

    ![par jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchronize Azure Active Directory Users to Adobe Identity Management** (Synchroniser les utilisateurs Azure Active Directory avec Adobe Identity Management).

9. Dans la section **Mappages d’attributs**, passez en revue les attributs d’utilisateur qui sont synchronisés entre Azure AD et Adobe Identity Management. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur d’Adobe Identity Management en vue de mises à jour ultérieures. Si vous choisissez de modifier l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vérifiez que l’API Adobe Identity Management prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |userName|String|
   |emails[type eq "work"].value|String|
   |active|Boolean|
   |addresses[type eq "work"].country|String|
   |name.givenName|String|
   |name.familyName|String|
   |urn:ietf:params:scim:schemas:extension:Adobe:2.0:User:emailAliases|String|

10. Dans la section **Mappages**, sélectionnez **Synchronize Azure Active Directory Groups to Adobe Identity Management** (Synchroniser les groupes Azure Active Directory avec Adobe Identity Management).

11. Dans la section **Mappages d’attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Adobe Identity Management. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les groupes d’Adobe Identity Management en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |displayName|String|
      |membres|Informations de référence|

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement d’Azure AD pour Adobe Identity Management, dans la section **Paramètres**, définissez le paramètre **État d’approvisionnement** sur **Activé**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner dans Adobe Identity Management en choisissant les valeurs souhaitées sous **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

1. Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
2. Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
3. Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)