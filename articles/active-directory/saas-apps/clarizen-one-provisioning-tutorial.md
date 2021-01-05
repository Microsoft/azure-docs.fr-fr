---
title: 'Tutoriel : Configurer Clarizen One pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment provisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure AD dans Clarizen One.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: d8021105-eb5b-4a20-8739-f02e0e22c147
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: Zhchia
ms.openlocfilehash: 3754a56d1520118937a9d4aad406ae657b4dd23b
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673662"
---
# <a name="tutorial-configure-clarizen-one-for-automatic-user-provisioning"></a>Tutoriel : Configurer Clarizen One pour l’approvisionnement automatique d’utilisateurs

Ce tutoriel décrit les étapes à suivre dans Clarizen One et Azure Active Directory (Azure AD) pour configurer l’approvisionnement automatique d’utilisateurs. Une fois configuré, Azure AD attribue et désattribue automatiquement les utilisateurs et les groupes dans [Clarizen One](https://www.clarizen.com/) à l’aide du service de provisionnement Azure AD. Pour plus d’informations sur l’objet et le fonctionnement de ce service, et pour accéder aux questions fréquentes, consultez [Automatisation de l’attribution et de l’annulation d’attribution des utilisateurs pour les applications SaaS (Software as a Service) avec Azure AD](../app-provisioning/user-provisioning.md).

## <a name="capabilities-supported"></a>Fonctionnalités prises en charge

> [!div class="checklist"]
> * Créer des utilisateurs dans Clarizen One.
> * Supprimer des utilisateurs dans Clarizen One quand ils n’ont plus besoin d’accès.
> * Maintenir les attributs utilisateur synchronisés entre Azure AD et Clarizen One.
> * Provisionner des groupes et des appartenances aux groupes dans Clarizen One.
> * L’[authentification unique (SSO)](./clarizen-tutorial.md) auprès de Clarizen One recommandée.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md).
* Un compte d’utilisateur dans Azure AD disposant d’une [autorisation](../roles/permissions-reference.md) pour configurer le provisionnement. Par exemple : Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général.
* Un compte d’utilisateur dans Clarizen One avec les [autorisations](https://success.clarizen.com/hc/articles/360011833079-API-Keys-Support) **Utilisateur d’intégration** et **Administrateur Lite**.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement

1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
1. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Déterminez les données à [mapper entre Azure AD et Clarizen One ](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-clarizen-one-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Clarizen One pour prendre en charge l’approvisionnement avec Azure AD

1. Sélectionnez l’une des quatre URL de locataire suivantes en fonction de votre environnement et centre de données Clarizen One :
      * Centre de données de production USA : https://servicesapp2.clarizen.com/scim/v2
      * Centre de données de production Europe : https://serviceseu1.clarizen.com/scim/v2
      * Centre de données du bac à sable USA : https://servicesapp.clarizentb.com/scim/v2
      * Centre de données du bac à sable Europe : https://serviceseu.clarizentb.com/scim/v2

1. Générez une [clé d’API](https://success.clarizen.com/hc/articles/360011833079-API-Keys-Support). Vous devrez entrer cette valeur dans la zone **Jeton secret** de l’onglet **Provisionnement** de votre application Clarizen One dans le portail Azure.

## <a name="step-3-add-clarizen-one-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Clarizen One à partir de la galerie d’applications Azure AD

Ajoutez Clarizen One à partir de la galerie d’applications Azure AD pour commencer à gérer l’approvisionnement sur Clarizen One. Si vous avez déjà configuré Clarizen One pour l’authentification unique (SSO), vous pouvez utiliser la même application. Quand vous effectuez un test initial de l’intégration, créez une application distincte. Pour en savoir plus sur la façon d’ajouter une application à partir de la galerie, consultez [Ajouter une application à votre locataire Azure AD](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement

Avec le service de provisionnement Azure AD, vous pouvez définir l’étendue des utilisateurs qui seront attribués en fonction de l’affectation à l’application ou en fonction des attributs de l’utilisateur ou du groupe. Si vous choisissez de définir l’étendue des utilisateurs qui seront attribués à votre application en fonction de l’affectation, effectuez les étapes mentionnées dans [Gérer l’attribution d’utilisateurs pour une application dans Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md) afin d’affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue des utilisateurs qui seront attribués uniquement en fonction des attributs de l’utilisateur ou du groupe, utilisez un filtre d’étendue comme décrit dans [Provisionnement d’application basé sur des attributs avec des filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* Quand vous affectez des utilisateurs et des groupes à Clarizen One, vous devez sélectionner un rôle autre que **Accès par défaut**. Les utilisateurs disposant du rôle d’accès par défaut sont exclus de l’attribution et sont marqués comme non autorisés dans les journaux de provisionnement. Si le seul rôle disponible sur l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste d’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter plus de rôles.
* Commencez progressivement. Effectuez un test avec un petit ensemble d’utilisateurs et de groupes avant de procéder à un déploiement général. Quand l’étendue de l’attribution est définie sur les utilisateurs et les groupes affectés, vous pouvez garder le contrôle en affectant un ou deux utilisateurs ou groupes à l’application. Quand l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur les attributs](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

## <a name="step-5-configure-automatic-user-provisioning-to-clarizen-one"></a>Étape 5. Configurer l’approvisionnement automatique d’utilisateurs sur Clarizen One

Cette section vous guide tout au long des étapes de configuration du service de provisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans TestApp en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

### <a name="configure-automatic-user-provisioning-for-clarizen-one-in-azure-ad"></a>Configurer le provisionnement automatique d’utilisateurs pour Clarizen One dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

      ![Capture d’écran montrant le volet Applications d’entreprise.](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Clarizen One**.

      ![Capture d’écran montrant le lien Clarizen One dans la liste Applications.](common/all-applications.png)

1. Sélectionnez l’onglet **Approvisionnement**.

      ![Capture d’écran montrant l’onglet Provisionnement.](common/provisioning.png)

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

      ![Capture d’écran montrant l’option Automatique de l’onglet Provisionnement.](common/provisioning-automatic.png)

1. Sous la section **Informations d’identification de l’administrateur**, entrez l’**URL du locataire** et le **Jeton secret** de votre compte Clarizen One. Sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Clarizen One. Si la connexion échoue, vérifiez que votre compte Clarizen One dispose d’autorisations d’administrateur, puis réessayez.

    ![Capture d’écran montrant la zone Jeton secret.](common/provisioning-testconnection-tenanturltoken.png)

1. Dans la zone **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![Capture d’écran montrant la zone E-mail de notification.](common/provisioning-notification-email.png)

1. Sélectionnez **Enregistrer**.

1. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Clarizen One**.

1. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Clarizen One. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Clarizen One pour les opérations de mise à jour. Si vous changez l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l’API Clarizen One prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |userName|String|
   |displayName|String|
   |active|Boolean|
   |title|String|
   |emails[type eq "work"].value|String|
   |emails[type eq "home"].value|String|
   |emails[type eq "other"].value|String|
   |preferredLanguage|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |name.honorificPrefix|String|
   |name.honorificSuffix|String|
   |addresses[type eq "other"].formatted|String|
   |addresses[type eq "work"].formatted|String|
   |addresses[type eq "work"].country|String|
   |addresses[type eq "work"].region|String|
   |addresses[type eq "work"].locality|String|
   |addresses[type eq "work"].postalCode|String|
   |addresses[type eq "work"].streetAddress|String|
   |phoneNumbers[type eq "work"].value|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "fax"].value|String|
   |phoneNumbers[type eq "home"].value|String|
   |phoneNumbers[type eq "other"].value|String|
   |phoneNumbers[type eq "pager"].value|String|
   |externalId|String|
   |nickName|String|
   |locale|String|
   |roles[primary eq"True".type]|String|
   |roles[primary eq"True".value]|String|
   |timezone|String|
   |userType|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Informations de référence|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|

1. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Clarizen One**.

1. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Clarizen One. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Clarizen One pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |displayName|String|
      |externalId|String|
      |membres|Informations de référence|

1. Pour configurer des filtres d’étendue, consultez les instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Afin d’activer le service de provisionnement Azure AD pour Clarizen One, définissez le paramètre **État de provisionnement** sur **Activé** dans la section **Paramètres**.

      ![Capture d’écran montrant le paramètre État de provisionnement activé.](common/provisioning-toggle-on.png)

1. Définissez les utilisateurs ou groupes à provisionner dans Clarizen One en sélectionnant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

      ![Capture d’écran montrant l’Étendue du provisionnement.](common/provisioning-scope.png)

1. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

      ![Capture d’écran montrant l’enregistrement de la configuration de provisionnement.](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution.

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement

Après avoir configuré le provisionnement, utilisez les ressources suivantes pour superviser votre déploiement.

1. Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou ceux pour laquelle la procédure a échoué.
1. Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état d’avancement du cycle d’approvisionnement et le moment où il se terminera.
1. Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, consultez [Provisionnement d’application en état de quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="troubleshooting-tips"></a>Conseils de dépannage

Quand vous attribuez un utilisateur à l’application de galerie Clarizen One, sélectionnez uniquement le rôle **Utilisateur**. Les rôles suivants ne sont pas valides :

* Administrateur (admin)
* Utilisateur de rapports par e-mail
* Utilisateur externe
* Utilisateur financier
* Utilisateur social
* Superutilisateur
* Utilisateur temps et dépenses

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)