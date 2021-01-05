---
title: 'Tutoriel : Configuration de Contentful pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory'
description: Découvrez comment approvisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure Active Directory (Azure AD) pour Contentful.
services: active-directory
documentationcenter: ''
author: zchia
manager: beatrizd
ms.assetid: 3b761984-a9a0-4519-b23e-563438978de5
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/11/2020
ms.author: zhchia
ms.openlocfilehash: c9d19624d90b1228b2a44caeff7d103af3172ed9
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516347"
---
# <a name="tutorial-configure-contentful-for-automatic-user-provisioning"></a>Tutoriel : Configuration de Contentful pour l’attribution automatique d’utilisateurs

Cet article décrit la procédure à suivre dans Contentful et Azure Active Directory (Azure AD) pour configurer l’approvisionnement automatique d’utilisateurs. Une fois configuré, Azure AD approvisionne et déprovisionne automatiquement des utilisateurs et des groupes sur [Contentful](https://www.contentful.com/) à l’aide du service d’approvisionnement d’Azure AD. Pour obtenir des informations importantes sur l’utilisation et le fonctionnement de ce service, ainsi que des réponses aux questions fréquentes, consultez [Automatiser le provisionnement et le déprovisionnement d’utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 

## <a name="capabilities-supported"></a>Fonctionnalités prises en charge

> [!div class="checklist"]
> * Créer des utilisateurs dans Contentful
> * Supprimer des utilisateurs dans Contentful lorsqu’ils n’ont plus besoin d’accès
> * Maintenir la synchronisation des attributs utilisateur entre Azure AD et Contentful
> * Provisionner des groupes et des appartenances aux groupes dans Contentful
> * [Authentification unique](contentful-tutorial.md) auprès de Contentful (recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md). 
* Un compte d’utilisateur dans Azure AD [autorisé](../roles/permissions-reference.md) à configurer l’approvisionnement (par exemple Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général). 
* Un compte d’organisation Contentful avec un abonnement qui prend en charge l’approvisionnement SCIM (System for Cross-domain Identity Management). Si vous avez des questions concernant l’abonnement de votre organisation, contactez le [support technique de Contentful](mailto:support@contentful.com).
 
## <a name="plan-your-provisioning-deployment"></a>Planifier votre déploiement de l’approvisionnement

1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Contentful](../app-provisioning/customize-application-attributes.md). 

## <a name="configure-contentful-to-support-provisioning-with-azure-ad"></a>Configurer Contentful pour prendre en charge le provisionnement avec Azure AD

1. Dans Contentful, créez un compte **Utilisateur de service**. Toutes les autorisations d’approvisionnement pour Azure sont accordées par l’intermédiaire de ce compte. Il est recommandé de choisir le rôle d’organisation **Propriétaire** pour ce compte.

2. Connectez-vous à Contentful en tant qu’**utilisateur de service**.

3. Dans le menu de gauche, sélectionnez **Paramètres de l’organisation** > **Outils d’accès** > **Attribution d’utilisateurs**.

   ![Capture d’écran du menu Paramètres de l’organisation dans Contentful, avec Attribution d’utilisateur mis en évidence sous Outils d’accès.](media/contentful-provisioning-tutorial/access.png)

4. Copiez et enregistrez **l’URL SCIM**. Vous entrez cette valeur dans le portail Azure, dans l’onglet **Approvisionnement** de votre application Contentful.

5. Sélectionnez **Générer un jeton d’accès personnel**.

    ![url](media/contentful-provisioning-tutorial/generate.png)

6. Dans la fenêtre modale, entrez un nom pour votre jeton d’accès personnel, puis sélectionnez **Générer**.

7. L’URL SCIM et le jeton secret sont générés. Copiez et enregistrez ces valeurs. Vous entrez ces valeurs dans l’onglet **Approvisionnement** de votre application Contentful dans le portail Azure.

    ![Capture d’écran du volet Jeton d’accès personnel, avec CFPAT et le nom de l’espace réservé du jeton mis en évidence.](media/contentful-provisioning-tutorial/token.png)


Si vous avez des questions lors de la configuration de l’approvisionnement dans la console d’administration Contentful, contactez le [support technique de Contentful](mailto:support@contentful.com).

## <a name="add-contentful-from-the-azure-ad-application-gallery"></a>Ajouter Contentful à partir de la galerie d’applications Azure AD

Pour gérer l’approvisionnement vers Contentful, ajoutez Contentful à partir de la galerie d’applications Azure AD. Si vous avez déjà configuré Contentful pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, nous vous recommandons de créer une application distincte pour tester l’intégration dans un premier temps. Découvrez comment [ajouter une application de la galerie](../manage-apps/add-application-portal.md). 

## <a name="define-who-will-be-in-scope-for-provisioning"></a>Définir qui sera dans l’étendue pour l’approvisionnement 

Vous pouvez utiliser le service d’approvisionnement Azure AD pour définir l’étendue des utilisateurs qui seront approvisionnés en fonction de l’affectation à l’application ou en fonction des attributs de l’utilisateur ou du groupe. 

Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, suivez les étapes pour [affecter des utilisateurs et des groupes à l’application](../manage-apps/assign-user-or-group-access-portal.md).

Si vous choisissez de définir l’étendue des utilisateurs qui seront approvisionnés uniquement en fonction des attributs de l’utilisateur ou du groupe, utilisez un filtre d’étendue afin de [définir des règles conditionnelles pour l’approvisionnement des comptes d’utilisateur](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Lorsque vous attribuez des utilisateurs et des groupes à Contentful, vous devez sélectionner un rôle autre que le rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux d’approvisionnement. Si le seul rôle disponible sur l’application est le rôle Accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter plus de rôles. 
* Commencez progressivement. Effectuez un test avec un petit ensemble d’utilisateurs et de groupes avant de procéder à un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler l’étendue en affectant un ou deux utilisateurs ou groupes à l’application. Quand l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur les attributs](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="configure-automatic-user-provisioning-to-contentful"></a>Configurer l’attribution automatique d’utilisateurs pour Contentful 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs et des groupes dans une application de test en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

### <a name="configure-automatic-user-provisioning-for-contentful-in-azure-ad"></a>Configurer l’approvisionnement automatique d’utilisateurs pour Contentful dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

   ![Capture d’écran montrant le menu Applications d’entreprise dans le portail Azure, avec l’option Toutes les applications mise en surbrillance.](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Contentful**.

   ![Capture d’écran montrant les 20 premiers résultats retournés dans la liste Applications.](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

   ![Capture d’écran de l’onglet Approvisionnement en surbrillance dans la section Gérer du menu de gauche.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

   ![Capture d’écran montrant les options du mode d’approvisionnement, avec Automatique mis en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification d’administration**, entrez l’URL et le jeton secret de votre locataire Contentful. Pour vérifier qu’Azure AD peut se connecter à Contentful, sélectionnez **Tester la connexion**. Si la connexion échoue, vérifiez que votre compte Contentful dispose d’autorisations d’administrateur, puis réessayez.

   ![Capture d’écran montrant les zones de texte URL du locataire et Jeton secret, avec le bouton Tester la connexion mis en évidence.](common/provisioning-testconnection-tenanturltoken.png)

6. Dans **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

   ![Capture d’écran montrant la zone de texte E-mail de notification.](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Contentful**.

9. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur synchronisés entre Azure AD et Contentful. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Contentful dans le cadre des opérations de mise à jour. Si vous choisissez de modifier [l’attribut cible de correspondance](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l’API Contentful prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|Pris en charge pour le filtrage|
   |---|---|---|
   |userName|String|&check;|
   |name.givenName|String|
   |name.familyName|String|

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Contentful**.

11. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe synchronisés entre Azure AD et Contentful. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Contentful dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    |Attribut|Type|Pris en charge pour le filtrage|
    |---|---|---|
    |displayName|String|&check;|
    |membres|Informations de référence|

12. Pour configurer des filtres d’étendue, effectuez les étapes décrites dans le [tutoriel relatif au filtre d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Contentful, dans la section **Paramètres**, définissez **État d’approvisionnement** sur **Activé**.

    ![Capture d’écran montrant le bouton bascule Activé ou Désactivé du paramètre État d’approvisionnement.](common/provisioning-toggle-on.png)

14. Pour définir les utilisateurs ou les groupes que vous souhaitez approvisionner dans Contentful, dans la section **Paramètres**, sélectionnez l’option appropriée pour **Étendue**.

    ![Capture d’écran montrant les options que vous pouvez sélectionner dans le volet Étendue.](common/provisioning-scope.png)

15. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Capture d’écran montrant le bouton Enregistrer et le bouton Annuler.](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** sous **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="monitor-your-deployment"></a>Surveiller votre déploiement

Après avoir configuré l’approvisionnement, utilisez les ressources suivantes pour superviser votre déploiement :

* Pour déterminer quels utilisateurs ont été configurés avec succès ou ceux pour laquelle la procédure a échoué, consultez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md).
* Pour afficher l’état d’avancement du cycle d’approvisionnement et le moment où il se terminera, consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md).
* Si la configuration de l’approvisionnement semble être dans un état non sain, l’application passe en quarantaine. En savoir plus sur les [états de quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Gérer l’attribution de comptes d’utilisateurs pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
