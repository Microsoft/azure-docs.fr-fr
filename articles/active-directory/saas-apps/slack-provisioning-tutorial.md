---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour Slack – Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Slack.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: arvinh
ms.openlocfilehash: 92c2ae13b840d7a73d86365ce88584bcafc878e8
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181456"
---
# <a name="tutorial-configure-slack-for-automatic-user-provisioning"></a>Tutoriel : Configurer Slack pour l’attribution automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans Slack et Azure AD pour approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers Slack. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Slack
> * Supprimer des utilisateurs dans Slack quand ils ne nécessitent plus d’accès
> * Conserver les attributs utilisateur synchronisés entre Azure AD et Slack
> * Approvisionner des groupes et des appartenances aux groupes dans Slack
> * [Authentification unique](./slack-tutorial.md) auprès de Slack (recommandé)


## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md).
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général).
* Un client Slack avec le [forfait Plus](https://aadsyncfabric.slack.com/pricing) ou mieux activé.
* Un compte d’utilisateur dans Slack avec les autorisations d’administrateur d’équipe.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Slack](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-add-slack-from-the-azure-ad-application-gallery"></a>Étape 2. Ajouter Slack à partir de la galerie d’applications Azure AD

Ajoutez Slack à partir de la galerie d’applications Azure AD pour commencer à gérer l’approvisionnement pour Slack. Si vous avez déjà configuré Slack pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-3-define-who-will-be-in-scope-for-provisioning"></a>Étape 3. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Quand vous attribuez des utilisateurs et des groupes à Slack, vous devez sélectionner un rôle différent du rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

## <a name="step-4-configure-automatic-user-provisioning-to-slack"></a>Étape 4. Configurer le provisionnement automatique d’utilisateurs sur Slack 

Cette section vous guide à travers la connexion de votre instance d’Azure AD au compte d’utilisateur fournissant l’API et la configuration du service de provisionnement pour créer, mettre à jour et désactiver les comptes utilisateur affectés dans Slack en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

### <a name="to-configure-automatic-user-account-provisioning-to-slack-in-azure-ad"></a>Pour configurer l’approvisionnement automatique de comptes utilisateur vers Slack dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Slack**.

    ![Lien Slack dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **informations d’identification de l’administrateur**, cliquez sur **Autoriser**. Une boîte de dialogue d’autorisation Slack s’ouvre dans une nouvelle fenêtre de navigateur.

    ![La capture d’écran affiche le bouton Autoriser les informations d’identification administrateur.](media/slack-provisioning-tutorial/authorization.png)


6. Dans la nouvelle fenêtre, connectez-vous à Slack à l’aide de votre compte d’administrateur d’équipe. Dans la boîte de dialogue d’autorisation qui s’affiche, sélectionnez l’équipe Slack pour laquelle vous souhaitez activer l’approvisionnement, puis sélectionnez **Autoriser**. Une fois cela terminé, revenez au portail Azure pour terminer la configuration de l’approvisionnement.

    ![Boîte de dialogue d’autorisation](./media/slack-provisioning-tutorial/slackauthorize.png)

7. Dans le portail Azure, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à votre application Slack. Si la connexion échoue, vérifiez que votre compte Slack dispose des autorisations d’administrateur d’équipe et recommencez l’étape « Autoriser ».

8. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Sélectionnez **Enregistrer**.

10. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Slack**.

11. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui seront synchronisés d’Azure AD vers Slack. Notez que les attributs sélectionnés en tant que propriétés de **Correspondance** seront utilisés pour faire correspondre les comptes d’utilisateur dans Slack pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |active|Boolean|
   |externalId|String|
   |displayName|String|
   |name.familyName|String|
   |name.givenName|String|
   |title|String|
   |emails[type eq "work"].value|String|
   |userName|String|
   |nickName|String|
   |addresses[type eq "untyped"].streetAddress|String|
   |addresses[type eq "untyped"].locality|String|
   |addresses[type eq "untyped"].region|String|
   |addresses[type eq "untyped"].postalCode|String|
   |addresses[type eq "untyped"].country|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "work"].value|String|
   |roles[primary eq "True"].value|String|
   |locale|String|
   |name.honorificPrefix|String|
   |photos[type eq "photo"].value|String|
   |profileUrl|String|
   |timezone|String|
   |userType|String|
   |urn:scim:schemas:extension:enterprise:1.0.department|String|
   |urn:scim:schemas:extension:enterprise:1.0.manager|Informations de référence|
   |urn:scim:schemas:extension:enterprise:1.0.employeeNumber|String|
   |urn:scim:schemas:extension:enterprise:1.0.costCenter|String|
   |urn:scim:schemas:extension:enterprise:1.0.organization|String|
   |urn:scim:schemas:extension:enterprise:1.0.division|String|

12. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Slack**.

13. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe qui seront synchronisés d’Azure AD vers Slack. Notez que les attributs sélectionnés en tant que propriétés de **Correspondance** seront utilisés pour faire correspondre les groupes dans Slack pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |displayName|String|
      |membres|Informations de référence|

14. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service d’approvisionnement Azure AD pour Slack, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

16. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Slack en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

17. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-5-monitor-your-deployment"></a>Étape 5. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

1. Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
2. Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
3. Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="troubleshooting-tips"></a>Conseils de dépannage

* Quand vous configurez l’attribut **displayName** de Slack, tenez compte des comportements suivants :

  * Les valeurs ne sont pas entièrement uniques (par exemple, 2 utilisateurs peuvent avoir le même nom complet).

  * Les caractères non anglais, les espaces et les majuscules sont pris en charge. 
  
  * Les signes de ponctuation autorisés sont notamment les points, traits de soulignement, traits d’union, apostrophes, crochets (par exemple, **( [ { } ] )** ) et séparateurs (par exemple, **, / ;** ).
  
  * La propriété displayName ne peut pas comporter de caractère « @ ». Si un « @ » est inclus, il se peut que vous trouviez un événement ignoré dans les journaux d’approvisionnement avec la description « AttributeValidationFailed ».

  * La mise à jour n’est effectuée que si ces deux paramètres sont configurés dans l’espace de travail/organisation de Slack : **Profile syncing is enabled** (La synchronisation des profils est activée) et **Users cannot change their display name** (Les utilisateurs ne peuvent pas changer leur nom d’affichage).

* L’attribut **userName** ne doit pas dépasser 21 caractères et sa valeur doit être unique.

* Slack n’autorise que la correspondance avec les attributs **userName** et **email**.  
  
* Les codes d’erreur courants sont documentés dans la documentation officielle de Slack : https://api.slack.com/scim#errors.

## <a name="change-log"></a>Journal des modifications

* 16/06/2020 : attribut DisplayName modifié pour être mis à jour uniquement lors de la création d’un nouvel utilisateur.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)