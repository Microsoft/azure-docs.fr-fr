---
title: 'Tutoriel : Configurer Playvox pour l’approvisionnement automatique d’utilisateurs à l’aide d’Azure Active Directory | Microsoft Docs'
description: Apprenez à approvisionner et déprovisionner automatiquement des comptes d'utilisateur entre Azure AD et Playvox.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: c31c20ab-f6cd-40e1-90ad-fa253ecbc0f8
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/18/2020
ms.author: Zhchia
ms.openlocfilehash: 3c7efca5e052c2d0680aa7ca3e1b6d27bfdd7d11
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862477"
---
# <a name="tutorial-configure-playvox-for-automatic-user-provisioning"></a>Tutoriel : Configurer Playvox pour l'approvisionnement automatique d'utilisateurs

Ce tutoriel décrit les étapes à suivre dans Playvox et Azure Active Directory (Azure AD) pour configurer l’approvisionnement automatique d’utilisateurs. Une fois la configuration effectuée, Azure AD approvisionne et déprovisionne automatiquement les utilisateurs ou les groupes dans [Playvox](https://www.playvox.com) à l’aide du service d’approvisionnement d’Azure AD. Pour obtenir des informations importantes sur l’utilisation et le fonctionnement de ce service, ainsi que des réponses aux questions fréquentes, consultez [Automatiser l’approvisionnement et le déprovisionnement d’utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Playvox.
> * Supprimer des utilisateurs dans Playvox lorsqu’ils n’ont plus besoin d’accès.
> * Maintenir la synchronisation des attributs utilisateur entre Azure AD et Playvox.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des éléments suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md).
* Un compte d’utilisateur dans Azure AD disposant d’une [autorisation](../roles/permissions-reference.md) pour configurer le provisionnement. Par exemple, un compte peut avoir le rôle Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général.
* Un compte d'utilisateur [Playvox](https://www.playvox.com) doté d'autorisations de super administrateur

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1 : Planifier votre déploiement de l’approvisionnement

1. Connaissez le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).

2. Déterminez qui sera [concerné par l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

3. Déterminez les données à [mapper entre Azure AD et Playvox](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-playvox-to-support-provisioning-by-using-azure-ad"></a>Étape 2 : Configurer Playvox pour prendre en charge l’approvisionnement avec Azure AD

1. Connectez-vous à la console d’administration Playvox et accédez à **Paramètres > Clés API**.

2. Sélectionnez **Créer une clé API**.

    ![Capture d’écran partielle montrant l’emplacement du bouton Créer une clé API dans l’interface utilisateur de Playvox.](media/playvox-provisioning-tutorial/create.png)

3. Entrez un nom explicite pour la clé API, puis sélectionnez **Enregistrer**. Une fois la clé API générée, sélectionnez **Fermer**.

4. Sur la clé API que vous avez créée, sélectionnez l’icône **Détails**.

    ![Capture d’écran partielle montrant l’emplacement de l’icône Détails, représentant une loupe, dans l’interface utilisateur de Playvox.](media/playvox-provisioning-tutorial/api.png)

5. Copiez et enregistrez la valeur **CLÉ BASE64**. Plus tard, dans le portail Azure, vous devrez entrer cette valeur dans la zone de texte **Jeton secret** de l’onglet **Approvisionnement** de votre application Playvox.

    ![Capture d’écran de la boîte de message Détails de la clé API, avec la valeur CLÉ BASE64 mise en évidence.](media/playvox-provisioning-tutorial/token.png)

## <a name="step-3-add-playvox-from-the-azure-ad-application-gallery"></a>Étape 3 : Ajouter Playvox à partir de la galerie d'applications Azure AD

Pour commencer à gérer l’approvisionnement vers Playvox, ajoutez Playvox à votre locataire Azure AD à partir de la galerie d’applications. Pour en savoir plus, consultez [Démarrage rapide : Ajouter une application à votre locataire Azure Active Directory (Azure AD)](../manage-apps/add-application-portal.md).

Si vous avez déjà configuré Playvox pour l’authentification unique (SSO), vous pouvez utiliser la même application. Toutefois, nous vous recommandons de créer une application distincte lors du test initial de l’intégration.

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4 : Définir qui sera dans l’étendue pour l’approvisionnement

Vous utilisez le service d’approvisionnement Azure AD pour délimiter les utilisateurs qui seront attribués en fonction de l’affectation à l’application ou des attributs de l’utilisateur ou du groupe. Pour délimiter les utilisateurs qui seront attribués à votre application en fonction de l’affectation, consultez [Gérer l’attribution d’utilisateurs pour une application dans Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md) pour savoir comment affecter des utilisateurs ou des groupes à l’application. Pour délimiter les utilisateurs qui seront attribués uniquement en fonction des attributs de l’utilisateur ou du groupe, utilisez un filtre d’étendue comme décrit dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

Souvenez-vous des points suivants :

* Lorsque vous attribuez des utilisateurs et des groupes à Playvox, vous devez sélectionner un rôle autre que Accès par défaut. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si « Accès par défaut » est le seul rôle disponible sur l’application, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter d’autres rôles.

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs ou de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs ou groupes affectés, vous pouvez contrôler la taille du jeu en attribuant seulement un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue d’approvisionnement comprend tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur les attributs](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md) pour limiter la taille de votre jeu de test.

## <a name="step-5-configure-automatic-user-provisioning-to-playvox"></a>Étape 5 : Configurer l'approvisionnement automatique d'utilisateurs sur Playvox

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou groupes en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

Si vous souhaitez configurer l'approvisionnement automatique d'utilisateurs pour Playvox dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Capture d’écran partielle du portail Azure, avec les éléments Applications d’entreprise et Toutes les applications mis en surbrillance](common/enterprise-applications.png)

2. Dans la liste des applications, recherchez et sélectionnez **Playvox**.

    ![Capture d’écran partielle de la liste des applications, avec la zone de recherche de l’application en évidence.](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran partielle montrant l’élément de menu Approvisionnement.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran partielle de l’onglet Approvisionnement, avec l’option Automatique sélectionnée dans la zone de liste déroulante Mode d’approvisionnement.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez votre **URL de locataire** Playvox comme suit :

    `https://{tenant}.playvox.com/scim/v1`

    Saisissez le **jeton secret** que vous avez copié plus tôt à l’étape 2. Sélectionnez ensuite **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Playvox. Si la connexion échoue, vérifiez que votre compte Playvox dispose des autorisations d’administrateur et réessayez.

    ![Capture d’écran partielle montrant la section Informations d’identification de l’administrateur, notamment les zones de texte URL du locataire et Jeton secret, avec le lien Tester la connexion mis en évidence.](common/provisioning-testconnection-tenanturltoken.png)

6. Dans la zone de texte **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement. Cochez ensuite la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![Capture d’écran partielle montrant la zone de texte E-mail de notification et la case à cocher de notification par e-mail.](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Playvox**.

9. Dans la section **Mappages d'attributs**, passez en revue les attributs d'utilisateur synchronisés entre Azure AD et Playvox. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d'utilisateur Playvox en vue de mises à jour ultérieures. Si vous choisissez de modifier l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vérifiez que l’API Playvox prend en charge le filtrage des utilisateurs en fonction de cet attribut. Sélectionnez **Enregistrer** pour valider les modifications.

   |Attribut|Type|Pris en charge pour le filtrage|
   |---|---|---|
   |userName|String|&check;|
   |active|Boolean|
   |displayName|String|
   |emails[type eq "work"].value|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |externalId|String|

10. Pour configurer des filtres d’étendue, consultez les instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Si vous souhaitez activer le service d'approvisionnement Azure AD pour Playvox, définissez le paramètre **État de l'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![Capture d’écran partielle de la section Paramètres avec l’option État de l’approvisionnement définie sur Activé.](common/provisioning-toggle-on.png)

12. Toujours dans **Paramètres**, définissez les utilisateurs ou les groupes à approvisionner sur Playvox en choisissant les valeurs souhaitées dans **Étendue**.

    ![Capture d’écran partielle de la section Paramètres, avec la zone de liste déroulante Étendue.](common/provisioning-scope.png)

13. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Capture d’écran partielle montrant les options Enregistrer et Annuler.](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle initial prend plus de temps que les cycles ultérieurs. Les cycles ultérieurs se produisent toutes les 40 minutes environ, à condition que le service d’approvisionnement Azure AD soit en cours d’exécution.

## <a name="step-6-monitor-your-deployment"></a>Étape 6 : Surveiller votre déploiement

Après avoir configuré l’approvisionnement, utilisez les ressources suivantes pour superviser votre déploiement :

* Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou ceux pour laquelle la procédure a échoué.
* Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état d’avancement du cycle d’approvisionnement et le moment où il se terminera.
* Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, consultez [Provisionnement d’application en état de quarantaine](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)