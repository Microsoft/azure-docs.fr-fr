---
title: 'Tutoriel : Configurer myPolicies pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et supprimer automatiquement des comptes d’utilisateur dans myPolicies.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: 221f63ab9a7eb3f71a4c730a11565dda64c9edc9
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353582"
---
# <a name="tutorial-configure-mypolicies-for-automatic-user-provisioning"></a>Tutoriel : Configurer myPolicies pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans myPolicies et Azure Active Directory (Azure AD) dans le but de configurer Azure AD pour l’approvisionnement et la suppression automatiques d’utilisateurs et/ou de groupes dans myPolicies.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire myPolicies](https://mypolicies.com/index.html#section10).
* Un compte d’utilisateur dans myPolicies avec des autorisations d’administrateur.

## <a name="assigning-users-to-mypolicies"></a>Attribution d’utilisateurs à myPolicies

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à myPolicies. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à myPolicies en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-mypolicies"></a>Conseils importants pour l’attribution d’utilisateurs dans myPolicies

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à myPolicies pour tester la configuration de l’approvisionnement automatique des utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à myPolicies, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-mypolicies-for-provisioning"></a>Configurez myPolicies pour l’approvisionnement

Avant de configurer myPolicies pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer le l’approvisionnement SCIM sur myPolicies.

1. Contactez votre représentant myPolicies à l’adresse **support@mypolicies.com** pour obtenir le jeton secret nécessaire à la configuration de l’approvisionnement SCIM.

2.  Enregistrez la valeur du jeton fournie par le représentant myPolicies. Vous devrez entrer cette valeur dans le champ **Jeton secret** dans l’onglet Approvisionnement de votre application myPolicies dans le portail Microsoft Azure.

## <a name="add-mypolicies-from-the-gallery"></a>Ajoutez myPolicies depuis la galerie

Avant de configurer myPolicies pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter myPolicies depuis la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter myPolicies depuis la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **myPolicies** , sélectionnez **myPolicies** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![myPolicies dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-mypolicies"></a>Configuration de l’approvisionnement automatique d’utilisateurs dans myPolicies 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans myPolicies en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour myPolicies en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour myPolicies](mypolicies-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-mypolicies-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs myPolicies dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **myPolicies**.

    ![Lien myPolicies dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://<myPoliciesCustomDomain>.mypolicies.com/scim` dans **URL du locataire** où `<myPoliciesCustomDomain>` se trouve votre domaine personnalisé myPolicies. Vous pouvez récupérer votre domaine client myPolicies à partir de votre URL.
Exemple : `<demo0-qa>`. mypolicies.com.

6. Dans **Jeton secret**, entrez la valeur du jeton que vous avez préalablement récupérée. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à myPolicies. Si la connexion échoue, vérifiez que votre compte myPolicies dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

7. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

8. Cliquez sur **Enregistrer**.

9. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur myPolicies**.

    :::image type="content" source="media/mypolicies-provisioning-tutorial/usermapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec customappsso est visible." border="false":::

10. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et myPolicies. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur myPolicies en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |userName|String|
   |active|Boolean|
   |emails[type eq "work"].value|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |externalId|String|
   |addresses[type eq "work"].country|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Informations de référence|


11. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Pour activer le service d’approvisionnement d’utilisateurs Azure AD pour myPolicies, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

13. Définissez les utilisateurs et/ou les groupes que vous aimeriez approvisionner dans myPolicies en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

14. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD dans myPolicies.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* myPolicies requiert toujours le **nom d’utilisateur**, **l’e-mail** et **externalId**.
* myPolicies ne prend pas en charge les suppressions définitives des attributs utilisateur.

## <a name="change-log"></a>Journal des modifications

* 15/09/2020 : ajout de la prise en charge de l’attribut « country » pour les utilisateurs.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
