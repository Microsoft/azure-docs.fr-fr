---
title: "Tutoriel : Configurer Code42 pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Apprenez à approvisionner automatiquement des comptes d'utilisateur d'Azure AD vers Code42 et à annuler cet approvisionnement.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: ddcb950b-3f9a-4ebb-bf78-4ec42d16d52d
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/23/2020
ms.author: Zhchia
ms.openlocfilehash: ffae286437a476d5ebe08c3ee442d0d52af84eb7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96179336"
---
# <a name="tutorial-configure-code42-for-automatic-user-provisioning"></a>Tutoriel : Configurer Code42 pour l'approvisionnement automatique d'utilisateurs

Ce tutoriel décrit les étapes à suivre dans Code42 et Azure Active Directory (Azure AD) pour configurer l'approvisionnement automatique d'utilisateurs. Une fois configuré, Azure AD procède automatiquement à l'approvisionnement ou à l'annulation de l'approvisionnement des utilisateurs et des groupes pour [Code42](https://www.crashplan.com/) à l'aide du service d'approvisionnement d'Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Code42
> * Supprimer des utilisateurs dans Code42 lorsqu'ils n'ont plus besoin d'accès
> * Maintenir la synchronisation des attributs utilisateur entre Azure AD et Code42
> * Approvisionner des groupes et des appartenances à des groupes dans Code42
> * [Authentification unique](./code42-tutorial.md) auprès de Code42 (recommandée)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md)
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, administrateur d’application, administrateur d’application Cloud, propriétaire d’application ou administrateur général).
* Un locataire Code42 sur lequel la Gestion des identités est activée.
* Un compte d'utilisateur Code42 doté de l'autorisation [Administrateur de cloud client](https://support.code42.com/Administrator/Cloud/Monitoring_and_managing/Roles_reference#Customer_Cloud_Admin).

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Code42](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-code42-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Code42 pour prendre en charge l'approvisionnement avec Azure AD

Cette section explique comment configurer Azure AD en tant que fournisseur d'approvisionnement dans la section Gestion des identités de la console Code42. Cela permettra à Code42 de recevoir en toute sécurité les demandes d'approvisionnement d'Azure AD. Nous vous recommandons de consulter la [documentation relative à Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD) avant de commencer à approvisionner avec Azure AD.

### <a name="to-create-a-provisioning-provider-in-code42s-console"></a>Pour créer un fournisseur d'approvisionnement à partir de la console de Code42 :

1. Connectez-vous à votre console Code42. Sélectionnez **Administration** pour développer le menu de navigation. Sélectionnez **Paramètres** puis **Gestion des identités**.
2. Sélectionnez l’onglet **Approvisionnement**. Développez ensuite le menu **Ajouter un fournisseur d'approvisionnement** et sélectionnez **Ajouter un fournisseur SCIM**.
3. Dans le champ **Nom d'affichage**, entrez un nom unique pour le fournisseur d'approvisionnement. Définissez le **Type d'informations d'identification d'authentification** sur **Jeton OAuth**. Sélectionnez **Suivant** pour générer les informations d'identification.

> [!NOTE]
>* Laissez cette fenêtre ouverte jusqu'à ce que vous soyez invité à fournir l'**URL de base** et le **Jeton** requis dans les étapes suivantes.
>* Vous pouvez également copier ces informations à un emplacement temporaire pour y revenir ultérieurement.

## <a name="step-3-add-code42-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Code42 à partir de la galerie d'applications Azure AD

Ajoutez Code42 à partir de la galerie d'applications Azure AD afin de commencer à gérer l'approvisionnement pour Code42. Si vous avez déjà configuré Code42 pour l'authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* Lorsque vous approvisionnez des utilisateurs et des groupes pour Code42, vous devez sélectionner un rôle autre que **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires.

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).


## <a name="step-5-configure-automatic-user-provisioning-to-code42"></a>Étape 5. Configurer l'approvisionnement automatique d'utilisateurs pour Code42

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-code42-in-azure-ad"></a>Pour configurer l'approvisionnement automatique d'utilisateurs pour Code42 dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Code42**.

    ![Lien Code42 dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Approvisionnement](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Onglet Provisionnement automatique](common/provisioning-automatic.png)

5. Dans la section **Informations d'identification de l'administrateur**, entrez les valeurs de l'**URL de base SCIM 2.0 et du Jeton d'accès** récupérées précédemment sur Code42 dans **URL de locataire** et **Jeton secret**, respectivement. Cliquez sur **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à Code42. Si la connexion échoue, vérifiez que votre compte Code42 dispose des autorisations d'administrateur et réessayez.

    ![par jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Code42**.

9. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Code42. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d'utilisateur Code42 en vue de mises à jour ultérieures. Si vous choisissez de modifier l'[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l'API Code42 prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |userName|String|
   |active|Boolean|
   |title|String|
   |emails[type eq "work"].value|String|
   |name.givenName|String|
   |name.familyName|String|
   |addresses[type eq "work"].locality|String|
   |addresses[type eq "work"].region|String|
   |addresses[type eq "work"].country|String|
   |externalId|String|
   |userType|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Informations de référence|

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Code42**.

11. Dans la section **Mappage d'attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Code42. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés afin de faire correspondre les groupes dans Code42 pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |displayName|String|
      |externalId|String|
      |membres|Informations de référence|

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d'approvisionnement Azure AD pour Code42, définissez le paramètre **État d'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Code42 en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

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
* [Configurer les affectations des organisations en fonction des groupes SCIM dans Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_6:_Choose_an_organization_mapping_method)
* [Configurer les attributions de rôle en fonction des groupes SCIM dans Code42](https://support.code42.com/Administrator/Cloud/Configuring/Introduction_to_SCIM_provisioning/How_to_provision_users_to_Code42_from_Azure_AD#Step_7:_Configure_role_mapping)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)