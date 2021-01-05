---
title: 'Tutoriel : Configurer G Suite pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment approvisionner et annuler automatiquement l’approvisionnement des comptes d’utilisateur d’Azure AD vers G Suite.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/06/2020
ms.author: Zhchia
ms.openlocfilehash: 4851dfb4a96ab2ca19ba6ea67139772f9c091a69
ms.sourcegitcommit: 799f0f187f96b45ae561923d002abad40e1eebd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97763642"
---
# <a name="tutorial-configure-g-suite-for-automatic-user-provisioning"></a>Tutoriel : Configurer G Suite pour l’attribution automatique d’utilisateurs

Ce tutoriel décrit les étapes à suivre dans G Suite et Azure Active Directory (Azure AD) pour configurer l’attribution automatique d’utilisateurs. Une fois la configuration effectuée, Azure AD attribue automatiquement les utilisateurs et les groupes (et supprime automatiquement les attributions) dans [G Suite](https://gsuite.google.com/) à l’aide du service d’attribution Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

> [!NOTE]
> Le connecteur G Suite a été récemment mis à jour en octobre 2019. Les modifications apportées au connecteur G Suite sont les suivantes :
>
> * Ajout de la prise en charge d’autres attributs d’utilisateur et de groupe G Suite.
> * Les noms d’attributs cibles G Suite mis à jour correspondent à ce qui est défini [ici](https://developers.google.com/admin-sdk/directory).
> * Mappages d’attributs par défaut mis à jour.

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Création d’utilisateurs dans G Suite
> * Suppression d’utilisateurs dans G Suite quand ils n’ont plus besoin d’accès
> * Maintien de la synchronisation des attributs utilisateur entre Azure AD et G Suite
> * Attribution de groupes et d’appartenances de groupe dans G Suite
> * [Authentification unique](./google-apps-tutorial.md) auprès de G Suite (recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md) 
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général). 
* [Un abonné G suite](https://gsuite.google.com/pricing.html)
* Un compte d’utilisateur dans G Suite avec des autorisations d’administrateur.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et G Suite](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-g-suite-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer S Suite pour prendre en charge l’attribution avec Azure AD

Avant de configurer G Suite pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM dans G Suite.

1. Connectez-vous à la [Console d’administration de G Suite](https://admin.google.com/) avec votre compte d’administrateur, puis sélectionnez **Sécurité**. Si le lien ne s’affiche pas, il est peut-être masqué par le menu **Autres contrôles** situé en bas de l’écran.

    ![G Suite – Sécurité](./media/google-apps-provisioning-tutorial/gapps-security.png)

2. Dans la page **Sécurité**, sélectionnez **Informations de référence sur l’API**.

    ![G Suite – API](./media/google-apps-provisioning-tutorial/gapps-api.png)

3. Sélectionnez **Activer l'accès à l'API**.

    ![G Suite – API activée](./media/google-apps-provisioning-tutorial/gapps-api-enabled.png)

    > [!IMPORTANT]
   > Le nom d’utilisateur dans Azure AD de chaque utilisateur que vous souhaitez approvisionner dans G Suite **doit** être associé à un domaine personnalisé. Par exemple, les noms d’utilisateur qui ressemblent à bob@contoso.onmicrosoft.com ne sont pas acceptés par G Suite. En revanche, bob@contoso.com est accepté. Vous pouvez modifier le domaine d’un utilisateur existant en suivant les instructions [ici](../fundamentals/add-custom-domain.md).

4. Une fois que vous avez ajouté et vérifié les domaines personnalisés souhaités avec Azure AD, vous devez les vérifier à nouveau avec G Suite. Pour vérifier les domaines dans G Suite, reportez-vous aux étapes suivantes :

    a. Dans la [Console d’administration de G Suite](https://admin.google.com/), sélectionnez **Domaines**.

    ![G Suite – Domaines](./media/google-apps-provisioning-tutorial/gapps-domains.png)

    b. Sélectionnez **Ajouter un domaine ou un alias de domaine**.

    ![G Suite – Ajouter un domaine](./media/google-apps-provisioning-tutorial/gapps-add-domain.png)

    c. Sélectionnez **Ajouter un autre domaine**, puis tapez le nom du domaine que vous souhaitez ajouter.

    ![G Suite – Ajouter un autre domaine](./media/google-apps-provisioning-tutorial/gapps-add-another.png)

    d. Sélectionnez **Continuer et vérifier la propriété du domaine**. Puis suivez les étapes pour vérifier que vous possédez le nom de domaine. Pour obtenir des instructions complètes sur la vérification de votre domaine avec Google, consultez [Vérification de votre propriété de site](https://support.google.com/webmasters/answer/35179).

    e. Répétez la procédure précédente pour tous les domaines supplémentaires que vous souhaitez ajouter à G Suite.

5. Déterminez ensuite le compte d’administrateur à utiliser pour gérer l’approvisionnement des utilisateurs dans G Suite. Accédez à **Rôles d’administrateur** .

    ![G Suite – Administration](./media/google-apps-provisioning-tutorial/gapps-admin.png)

6. Pour le **Rôle d’administrateur** de ce compte, modifiez les **Privilèges** pour ce rôle. Veillez à activer tous les **Privilèges d’administrateur d’API** pour que ce compte puisse être utilisé pour l’approvisionnement.

    ![G Suite – Privilèges Administrateur](./media/google-apps-provisioning-tutorial/gapps-admin-privileges.png)

## <a name="step-3-add-g-suite-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter G Suite à partir de la galerie d’applications Azure AD

Ajoutez G Suite à partir de la galerie d’applications Azure AD pour pouvoir gérer l’attribution dans G Suite. Si vous avez déjà configuré G Suite pour l’authentification SSO, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Quand vous attribuez des utilisateurs et des groupes à G Suite, vous devez sélectionner un autre rôle que le rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-g-suite"></a>Étape 5. Configurer l’attribution automatique d’utilisateurs sur G Suite 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison de l’API d’annuaire G Suite, reportez-vous à l’[API Directory](https://developers.google.com/admin-sdk/directory).

### <a name="to-configure-automatic-user-provisioning-for-g-suite-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour G Suite dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**. Les utilisateurs devront se connecter à portal.azure.com et ne pourront pas utiliser aad.portal.azure.com

    ![Panneau Applications d’entreprise](./media/google-apps-provisioning-tutorial/enterprise-applications.png)

    ![Panneau Toutes les applications](./media/google-apps-provisioning-tutorial/all-applications.png)

2. Dans la liste des applications, sélectionnez **G Suite**.

    ![Lien G Suite dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**. Cliquez sur **Prise en main**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

      ![Panneau Prise en main](./media/google-apps-provisioning-tutorial/get-started.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification de l’administrateur**, cliquez sur **Autoriser**. Une boîte de dialogue d’autorisation Google s’ouvre dans une nouvelle fenêtre du navigateur.

      ![G Suite – Autoriser](./media/google-apps-provisioning-tutorial/authorize-1.png)

6. Confirmez que vous souhaitez autoriser Azure AD à apporter des modifications à votre abonné G Suite. Sélectionnez **Accepter**.

     ![G Suite – Authentification du locataire](./media/google-apps-provisioning-tutorial/gapps-auth.png)

7. Sur le Portail Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à G Suite. Si la connexion échoue, vérifiez que votre compte G Suite dispose des autorisations d’administrateur et réessayez. Ensuite, réessayez d’effectuer l’étape **Autoriser**.

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Attribuer les utilisateurs Azure Active Directory**.

9. Dans la section **Mappages des attributs**, vérifiez les attributs d’utilisateurs qui sont synchronisés d’Azure AD vers G Suite. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans G Suite pour les opérations de mise à jour. Si vous choisissez de modifier [l’attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vérifiez que l’API G Suite prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |primaryEmail|String|
   |relations.[type eq "manager"].value|String|
   |name.familyName|String|
   |name.givenName|String|
   |interrompu|String|
   |externalIds.[type eq "custom"].value|String|
   |externalIds.[type eq "organization"].value|String|
   |addresses.[type eq "work"].country|String|
   |addresses.[type eq "work"].streetAddress|String|
   |addresses.[type eq "work"].region|String|
   |addresses.[type eq "work"].locality|String|
   |addresses.[type eq "work"].postalCode|String|
   |emails.[type eq "work"].address|String|
   |organizations.[type eq "work"].department|String|
   |organizations.[type eq "work"].title|String|
   |phoneNumbers.[type eq "work"].value|String|
   |phoneNumbers.[type eq "mobile"].value|String|
   |phoneNumbers.[type eq "work_fax"].value|String|
   |emails.[type eq "work"].address|String|
   |organizations.[type eq "work"].department|String|
   |organizations.[type eq "work"].title|String|
   |phoneNumbers.[type eq "work"].value|String|
   |phoneNumbers.[type eq "mobile"].value|String|
   |phoneNumbers.[type eq "work_fax"].value|String|
   |addresses.[type eq "home"].country|String|
   |addresses.[type eq "home"].formatted|String|
   |addresses.[type eq "home"].locality|String|
   |addresses.[type eq "home"].postalCode|String|
   |addresses.[type eq "home"].region|String|
   |addresses.[type eq "home"].streetAddress|String|
   |addresses.[type eq "other"].country|String|
   |addresses.[type eq "other"].formatted|String|
   |addresses.[type eq "other"].locality|String|
   |addresses.[type eq "other"].postalCode|String|
   |addresses.[type eq "other"].region|String|
   |addresses.[type eq "other"].streetAddress|String|
   |addresses.[type eq "work"].formatted|String|
   |changePasswordAtNextLogin|String|
   |emails.[type eq "home"].address|String|
   |emails.[type eq "other"].address|String|
   |externalIds.[type eq "account"].value|String|
   |externalIds.[type eq "custom"].customType|String|
   |externalIds.[type eq "customer"].value|String|
   |externalIds.[type eq "login_id"].value|String|
   |externalIds.[type eq "network"].value|String|
   |gender.type|String|
   |GeneratedImmutableId|String|
   |Identificateur|String|
   |ims.[type eq "home"].protocol|String|
   |ims.[type eq "other"].protocol|String|
   |ims.[type eq "work"].protocol|String|
   |includeInGlobalAddressList|String|
   |ipWhitelisted|String|
   |organizations.[type eq "school"].costCenter|String|
   |organizations.[type eq "school"].department|String|
   |organizations.[type eq "school"].domain|String|
   |organizations.[type eq "school"].fullTimeEquivalent|String|
   |organizations.[type eq "school"].location|String|
   |organizations.[type eq "school"].name|String|
   |organizations.[type eq "school"].symbol|String|
   |organizations.[type eq "school"].title|String|
   |organizations.[type eq "work"].costCenter|String|
   |organizations.[type eq "work"].domain|String|
   |organizations.[type eq "work"].fullTimeEquivalent|String|
   |organizations.[type eq "work"].location|String|
   |organizations.[type eq "work"].name|String|
   |organizations.[type eq "work"].symbol|String|
   |OrgUnitPath|String|
   |phoneNumbers.[type eq "home"].value|String|
   |phoneNumbers.[type eq "other"].value|String|
   |websites.[type eq "home"].value|String|
   |websites.[type eq "other"].value|String|
   |websites.[type eq "work"].value|String|
   

10. Dans la section **Mappages**, sélectionnez **Attribuer les groupes Azure Active Directory**.

11. Dans la section **Mappage des attributs**, vérifiez les attributs de groupes qui sont synchronisés d’Azure AD vers G Suite. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondre des groupes dans G Suite dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

      |Attribut|Type|
      |---|---|
      |email|String|
      |Membres|String|
      |name|String|
      |description|String|

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour G Suite, passez l’**État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur G Suite en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution.

> [!NOTE]
> Si les utilisateurs disposent déjà d’un compte personnel/consommateur existant à l’aide de l’adresse e-mail de l’utilisateur Azure AD, cela peut entraîner un problème qui peut être résolu à l’aide de l’outil Google Transfer avant d’effectuer la synchronisation d’annuaires.

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