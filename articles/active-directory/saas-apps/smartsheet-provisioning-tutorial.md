---
title: 'Tutoriel : Configurer Smartsheet pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner des comptes d’utilisateur dans Smartsheet.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/07/2019
ms.author: jeedes
ms.openlocfilehash: aa4e8c2baaaa0c8ccc9bcdda595f040fac72682f
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181439"
---
# <a name="tutorial-configure-smartsheet-for-automatic-user-provisioning"></a>Tutoriel : Configurer Smartsheet pour l’attribution automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à suivre dans Smartsheet et Azure Active Directory (Azure AD) afin de configurer Azure AD pour provisionner et annuler automatiquement des utilisateurs et/ou des groupes dans [Smartsheet](https://www.smartsheet.com/pricing). Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Smartsheet
> * Supprimer des utilisateurs dans Smartsheet quand ils ne nécessitent plus d’accès
> * Maintenir la synchronisation des attributs utilisateur entre Azure AD et Smartsheet
> * Authentification unique auprès de Smartsheet (recommandé)

> [!NOTE]
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md).
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général).
* [Un locataire Smartsheet](https://www.smartsheet.com/pricing).
* Un compte d’utilisateur sur un plan Smartsheet Enterprise ou Enterprise Premier avec des autorisations d’administrateur système.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Smartsheet](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-smartsheet-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Smartsheet pour prendre en charge le provisionnement avec Azure AD

Avant de configurer Smartsheet pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez activer le provisionnement SCIM sur Smartsheet.

1. Connectez-vous en tant que **SysAdmin** dans le **[portail Smartsheet](https://app.smartsheet.com/b/home)** et accédez à l’**administrateur de compte**.

    ![Administrateur de compte Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-accountadmin.png)

2. Accédez à **Security Controls (Contrôles de sécurité) > User Auto Provisioning (Attribution automatique d’utilisateurs) > Edit (Modifier)** .

    ![Contrôles de sécurité Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-securitycontrols.png)

3. Ajoutez et validez les domaines de courrier pour les utilisateurs que vous envisagez d’approvisionner à partir d’Azure AD dans Smartsheet. Choisissez **Not Enabled (Non activé)** pour vous assurer que toutes les actions de provisionnement proviennent uniquement d’Azure AD et également pour vous assurer que votre liste d’utilisateurs Smartsheet est synchronisée avec les attributions d’Azure AD.

    ![Attribution d’utilisateurs Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-userprovisioning.png)

4. Une fois la validation terminée, vous devez activer le domaine. 

    ![Activation du domaine Smartsheet](media/smartsheet-provisioning-tutorial/smartsheet-activatedomain.png)

5. Générez le **jeton secret** requis pour configurer l’attribution automatique d’utilisateurs avec Azure AD en accédant à **Apps and Integrations (Applications et intégrations)** .

    ![Capture d’écran de la page d’administration de Smartsheet avec l’avatar de l’utilisateur et l’option Apps and Integrations (Applications et intégrations) en évidence.](media/smartsheet-provisioning-tutorial/Smartsheet05.png)

6. Choisissez **Accès d’API**. Cliquez sur **Generate new access token (Générer un nouveau jeton d'accès)** .

    ![Capture d’écran de la boîte de dialogue Personal Settings (Paramètres personnels) avec les options API Access (Accès à l’API) et Generate new access token (Générer un nouveau jeton d’accès) en évidence.](media/smartsheet-provisioning-tutorial/Smartsheet06.png)

7. Définissez le nom du jeton d’accès d’API. Cliquez sur **OK**.

    ![Capture d’écran de Step 1 of 2: Generate new access token (Étape 1 sur 2 : Générer un nouveau jeton d’accès) avec l’option OK en évidence.](media/smartsheet-provisioning-tutorial/Smartsheet07.png)

8. Copiez le jeton d’accès d’API et enregistrez-le car il s’agit de la seule fois où vous pouvez le consulter. Il est requis dans le champ **Jeton secret**, dans Azure AD.

    ![Jeton Smartsheet](media/smartsheet-provisioning-tutorial/Smartsheet08.png)

## <a name="step-3-add-smartsheet-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Smartsheet à partir de la galerie d’applications Azure AD

Ajoutez Smartsheet à partir de la galerie d’applications Azure AD pour commencer à gérer le provisionnement vers Smartsheet. Si vous avez déjà configuré Smartsheet pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Quand vous affectez des utilisateurs et des groupes à Smartsheet, vous devez sélectionner un rôle différent du rôle **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Pour garantir la parité dans les attributions de rôles d’utilisateur entre Smartsheet et Azure AD, il est recommandé d’utiliser les attributions de rôles renseignées dans la liste complète des utilisateurs Smartsheet. Pour récupérer cette liste d’utilisateurs à partir de Smartsheet, accédez à **Account Admin (Administrateur de compte) > User Management (Gestion des utilisateurs) > More Actions (Autres Actions) > Download User List (csv) (Télécharger la liste des utilisateurs (csv))** .

* Pour accéder à certaines fonctionnalités de l’application, Smartsheet exige d’un utilisateur qu’il possède plusieurs rôles. Pour en savoir plus sur les types d’utilisateurs et les autorisations dans Smartsheet, accédez à [User Types and Permissions (Types d’utilisateurs et autorisations)](https://help.smartsheet.com/learning-track/shared-users/user-types-and-permissions).

*  Si un utilisateur a plusieurs rôles dans Smartsheet, vous **DEVEZ** garantir que ces attributions de rôles sont répliquées dans Azure AD afin d’éviter un scénario où les utilisateurs pourraient perdre définitivement l’accès aux objets Smartsheet. Chaque rôle unique dans Smartsheet **DOIT** être attribué à un groupe différent dans Azure AD. L’utilisateur **DOIT** ensuite être ajouté à chacun des groupes correspondant aux rôles que vous souhaitez. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="step-5-configure-automatic-user-provisioning-to-smartsheet"></a>Étape 5. Configurer l’attribution automatique d’utilisateurs dans Smartsheet 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Smartsheet en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-smartsheet-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Smartsheet dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Smartsheet**.

    ![Lien Smartsheet dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification de l’administrateur**, entrez les valeurs d’URL de base **SCIM 2.0 et de Jeton du porteur** récupérées précédemment sur Smartsheet dans **URL de locataire** et **Jeton secret**, respectivement. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Smartsheet. Si la connexion échoue, vérifiez que votre compte Smartsheet dispose d’autorisations d’administrateur système et réessayez.

    ![par jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Sous la section **Mappages**, sélectionnez **Synchronize Azure Active Directory Users to Smartsheet (Synchroniser les utilisateurs Azure Active Directory dans Smartsheet)** .

9. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers Smartsheet dans la section **Mappage d’attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour mettre en correspondance les comptes d’utilisateur dans Smartsheet pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

   |Attribut|Type|
   |---|---|
   |active|Boolean|
   |title|String|
   |userName|String|
   |name.givenName|String|
   |name.familyName|String|
   |phoneNumbers[type eq "work"].value|String|
   |phoneNumbers[type eq "mobile"].value|String|
   |phoneNumbers[type eq "fax"].value|String|
   |externalId|String|
   |roles[primary eq "True"].display|String|
   |roles[primary eq "True"].type|String|
   |roles[primary eq "True"].value|String|
   |roles|String|
   urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|String|


10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service de provisionnement Azure AD pour Smartsheet, modifiez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous aimeriez provisionner dans Smartsheet en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

1. Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
2. Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
3. Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Limitations du connecteur

* Smartsheet ne prend pas en charge les suppressions réversibles. Lorsque l’attribut **actif** d’un utilisateur est défini sur False, Smartsheet supprime l’utilisateur de façon permanente.

## <a name="change-log"></a>Journal des modifications

* 16/06/2020-ajout de la prise en charge des attributs d’extension d’entreprise « Centre de coûts », « Division », « Manager » et « Départment » pour les utilisateurs.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)