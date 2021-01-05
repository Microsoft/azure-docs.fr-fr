---
title: 'Tutoriel : Configurer Looop pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Looop.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/19/2019
ms.author: Zhchia
ms.openlocfilehash: 528003ac482da6f254bf437321c70c389d23844b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835014"
---
# <a name="tutorial-configure-looop-for-automatic-user-provisioning"></a>Tutoriel : Configurer Looop pour l’affectation automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Looop et Azure Active Directory (Azure AD), afin de configurer Azure AD pour le provisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Looop.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [un locataire Looop](https://www.looop.co/pricing/) ;
* un compte d’utilisateur dans une instance Looop avec des autorisations d’administrateur.

## <a name="assign-users-to-looop"></a>Affecter des utilisateurs à Looop

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Looop. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Looop en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-looop"></a>Conseils importants pour l’affectation d’utilisateurs à Looop

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Looop afin de tester la configuration du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Looop, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-looop-for-provisioning"></a>Configurer Looop pour le provisionnement

Avant de configurer Looop pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez récupérer certaines informations de configuration à partir de Looop.

1. Connectez-vous à votre [Console d’administration Looop](https://app.looop.co/#/login) et sélectionnez **Account** (Compte). Sous **Account Settings**, sélectionnez **Authentication**.

    :::image type="content" source="media/looop-provisioning-tutorial/admin.png" alt-text="Capture d’écran de la console d’administration Looop. L’onglet Account est mis en évidence et ouvert. Sous Account settings, Authentication est mis en évidence." border="false":::

2. Générez un jeton en cliquant sur **Reset Token** (Réinitialiser le jeton) sous **SCIM Integration**.

    :::image type="content" source="media/looop-provisioning-tutorial/resettoken.png" alt-text="Capture d’écran de la section SCIM Integration d’une page dans la console d’administration Looop. Le bouton Reset token est mis en évidence." border="false":::

3. Copiez le **Point de terminaison SCIM** et le **Jeton**. Ces valeurs doivent être entrées dans les champs **URL de locataire** et **Jeton secret** de l’onglet Provisionnement, dans votre application Looop sur le portail Azure. 

    ![Looop - Créer le jeton](media/looop-provisioning-tutorial/token.png)

## <a name="add-looop-from-the-gallery"></a>Ajouter Looop à partir de la galerie

Pour configurer Looop en vue du provisionnement automatique d’utilisateurs avec Azure AD, vous devez, à partir de la galerie d’applications Azure AD, ajouter Looop à votre liste d’applications SaaS managées.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Looop**, puis sélectionnez **Looop** dans le volet de résultats. 

    ![Looop dans la liste des résultats](common/search-new-app.png)

5. Sélectionnez le bouton **S’inscrire à Looop** qui vous redirige vers la page de connexion de Looop. 

    ![Looop - Ajouter OIDC](media/looop-provisioning-tutorial/signup.png)

6. Comme Looop est une application OpenIDConnect, choisissez de vous connecter à Looop au moyen de votre compte professionnel Microsoft.

    ![Looop - Connexion OIDC](media/looop-provisioning-tutorial/msftlogin.png)

7. Une fois l’authentification réussie, acceptez l’invite de consentement pour la page de consentement. L’application est alors automatiquement ajoutée à votre locataire et vous êtes redirigé vers votre compte Looop.

    ![Looop - Consentement OIDC](media/looop-provisioning-tutorial/accept.png)

## <a name="configure-automatic-user-provisioning-to-looop"></a>Configurer le provisionnement automatique d’utilisateurs sur Looop 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Looop, en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-looop-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs destiné à Looop dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Looop**.

    ![Lien Looop dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://<organisation_domain>.looop.co/scim/v2` dans **URL de locataire**. Par exemple : `https://demo.looop.co/scim/v2`. Entrez la valeur que vous avez récupérée et enregistrée précédemment de Looop, dans **Secret Token** (Jeton secret). Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Looop. Si la connexion échoue, vérifiez que votre compte Looop dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Looop**.

    ![Looop - Mappages d’utilisateurs](media/looop-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappage d’attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Looop. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés afin de faire correspondre les comptes d’utilisateur dans Looop pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|Pris en charge pour le filtrage|
   |---|---|---|
   |userName|String|&check;|
   |active|Boolean|
   |emails[type eq "work"].value|String|
   |name.givenName|String|
   |name.familyName|String|
   |externalId|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:area|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_1|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_2|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:custom_3|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:employee_id|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:location|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:position|String|
   |urn:ietf:params:scim:schemas:extension:Looop:2.0:User:startAt|String|

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Meta Networks Connector**.

    ![Looop - Mappages de groupes](media/looop-provisioning-tutorial/groupmappings.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Meta Networks Connector. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Meta Networks Connector pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    |Attribut|Type|Pris en charge pour le filtrage|
    |---|---|---|
    |displayName|String|&check;|
    |membres|Informations de référence|
    |externalId|String|


10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Afin d’activer le service de provisionnement Azure AD pour Looop, définissez le paramètre **État de provisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou les groupes que vous voulez provisionner sur Looop en choisissant les valeurs souhaitées dans **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour superviser la progression et suivre les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Looop.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)


