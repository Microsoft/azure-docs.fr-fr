---
title: 'Tutoriel : Configurer Oracle Cloud Infrastructure Console pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment provisionner et déprovisionner automatiquement des comptes d’utilisateur entre Azure AD et Oracle Cloud Infrastructure Console.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: Zhchia
ms.openlocfilehash: 94de0ca0a5393c891e567e558cbbadd0ca1f453b
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832077"
---
# <a name="tutorial-configure-oracle-cloud-infrastructure-console-for-automatic-user-provisioning"></a>Tutoriel : Configurer Oracle Cloud Infrastructure Console pour le provisionnement automatique d’utilisateurs

Ce tutoriel décrit les étapes à suivre dans Oracle Cloud Infrastructure Console et Azure Active Directory (Azure AD) pour configurer le provisionnement automatique d’utilisateurs. Une fois configuré, Azure AD provisionne et déprovisionne automatiquement des utilisateurs et des groupes sur [Oracle Cloud Infrastructure Console](https://www.oracle.com/cloud/free/?source=:ow:o:p:nav:0916BCButton&intcmp=:ow:o:p:nav:0916BCButton) en utilisant le service de provisionnement Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Fonctionnalités prises en charge
> [!div class="checklist"]
> * Créer des utilisateurs dans Oracle Cloud Infrastructure Console
> * Supprimer des utilisateurs dans Oracle Cloud Infrastructure Console quand ils ne nécessitent plus d’accès
> * Conserver les attributs utilisateur synchronisés entre Azure AD et Oracle Cloud Infrastructure Console
> * Provisionner des groupes et des appartenances de groupe dans Oracle Cloud Infrastructure Console
> * [Authentification unique](./oracle-cloud-tutorial.md) pour Oracle Cloud Infrastructure Console(recommandé)

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* [Un locataire Azure AD](../develop/quickstart-create-new-tenant.md) 
* Un compte d’utilisateur dans Azure AD avec l’[autorisation](../roles/permissions-reference.md) de configurer l’approvisionnement (par exemple, Administrateur d’application, Administrateur d’application cloud, Propriétaire d’application ou Administrateur général). 
* Un [locataire](https://www.oracle.com/cloud/sign-in.html?intcmp=OcomFreeTier&source=:ow:o:p:nav:0916BCButton) Oracle Cloud Infrastructure Console.
* Un compte d’utilisateur dans Oracle Cloud Infrastructure Console avec des autorisations d’administrateur.

## <a name="step-1-plan-your-provisioning-deployment"></a>Étape 1. Planifier votre déploiement de l’approvisionnement
1. En savoir plus sur le [fonctionnement du service d’approvisionnement](../app-provisioning/user-provisioning.md).
2. Déterminez qui sera dans l’[étendue pour l’approvisionnement](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Déterminez les données à [mapper entre Azure AD et Oracle Cloud Infrastructure Console](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-oracle-cloud-infrastructure-console-to-support-provisioning-with-azure-ad"></a>Étape 2. Configurer Oracle Cloud Infrastructure Console pour la prise en charge du provisionnement automatique avec Azure AD

1. Connectez-vous au portail d’administration d’Oracle Cloud Infrastructure Console. Dans le coin supérieur gauche de l’écran, accédez à **Identité >Fédération**.

    ![Administrateur Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/identity.png)

2. Cliquez sur l’URL affichée sur la page à côté d’Oracle Identity Cloud Service Console.

    ![URL Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/url.png)

3. Cliquez sur **Ajouter un fournisseur d’identité** pour créer un fournisseur d’identité. Enregistrez l’ID du fournisseur d’identité à utiliser dans le cadre de l’URL du locataire. Cliquez sur l’icône avec le signe plus en regard de l’onglet **Applications** pour créer un client OAuth et accorder le rôle d’application à l’administrateur de domaine d’identité IDCS.

    ![Icône du cloud Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/add.png)

4. Suivez les captures d’écran ci-dessous pour configurer votre application. Une fois la configuration terminée, cliquez sur **Enregistrer**.

    ![Configuration d’Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/configuration.png)

    ![Stratégie d’émission de jetons Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/token-issuance.png)

5. Sous l’onglet Configurations de votre application, développez l’option **Informations générales** pour récupérer l’ID de client et le secret du client.

    ![Génération de jetons Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/general-information.png)

6. Pour générer un jeton secret, encodez en Base64 l’ID de client et le secret du client au format **ID de client:Secret du client**. Enregistrez le jeton secret. Vous devrez entrer cette valeur dans le champ **Jeton secret** dans l’onglet Provisionnement de votre application Oracle Cloud Infrastructure Console dans le portail Azure.

## <a name="step-3-add-oracle-cloud-infrastructure-console-from-the-azure-ad-application-gallery"></a>Étape 3. Ajouter Oracle Cloud Infrastructure Console à partir de la galerie d’applications Azure AD

Ajoutez Oracle Cloud Infrastructure Console à partir de la galerie d’applications Azure AD pour commencer à gérer le provisionnement pour Oracle Cloud Infrastructure Console. Si vous avez déjà configuré Oracle Cloud Infrastructure Console pour l’authentification unique, vous pouvez utiliser la même application. Toutefois, il est recommandé de créer une application distincte lors du test initial de l’intégration. En savoir plus sur l’ajout d’une application à partir de la galerie [ici](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Étape 4. Définir qui sera dans l’étendue pour l’approvisionnement 

Le service d’approvisionnement Azure AD vous permet de définir l’étendue des utilisateurs approvisionnés en fonction de l’affectation à l’application et/ou en fonction des attributs de l’utilisateur/groupe. Si vous choisissez de définir l’étendue de l’approvisionnement pour votre application en fonction de l’attribution, vous pouvez utiliser les étapes de [suivantes](../manage-apps/assign-user-or-group-access-portal.md) pour affecter des utilisateurs et des groupes à l’application. Si vous choisissez de définir l’étendue de l’approvisionnement en fonction uniquement des attributs de l’utilisateur ou du groupe, vous pouvez utiliser un filtre d’étendue comme décrit [ici](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Lors de l’affectation d’utilisateurs et de groupes à Oracle Cloud Infrastructure Console, vous devez sélectionner un rôle autre que **Accès par défaut**. Les utilisateurs disposant du rôle Accès par défaut sont exclus de l’approvisionnement et sont marqués comme non autorisés dans les journaux de configuration. Si le seul rôle disponible dans l’application est le rôle d’accès par défaut, vous pouvez [mettre à jour le manifeste de l’application](../develop/howto-add-app-roles-in-azure-ad-apps.md) pour ajouter des rôles supplémentaires. 

* Commencez progressivement. Testez avec un petit ensemble d’utilisateurs et de groupes avant d’effectuer un déploiement général. Lorsque l’étendue de l’approvisionnement est définie sur les utilisateurs et les groupes attribués, vous pouvez contrôler cela en affectant un ou deux utilisateurs ou groupes à l’application. Lorsque l’étendue est définie sur tous les utilisateurs et groupes, vous pouvez spécifier un [filtre d’étendue basé sur l’attribut](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-oracle-cloud-infrastructure-console"></a>Étape 5. Configurer le provisionnement automatique d’utilisateurs pour Oracle Cloud Infrastructure Console 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans TestApp en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-oracle-cloud-infrastructure-console-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour Oracle Cloud Infrastructure Console dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Oracle Cloud Infrastructure Console**.

    ![Lien d’Oracle Cloud Infrastructure Console dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification d’administrateur**, entrez l’**URL de locataire** au format `https://<IdP ID>.identity.oraclecloud.com/admin/v1`. Par exemple, `https://idcs-0bfd023ff2xx4a98a760fa2c31k92b1d.identity.oraclecloud.com/admin/v1`. Entrez la valeur du jeton secret récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Oracle Cloud Infrastructure Console. Si la connexion échoue, vérifiez que votre compte Oracle Cloud Infrastructure Console dispose d’autorisations d’administrateur, puis réessayez.

    ![Capture d’écran de la boîte de dialogue Informations d’identification de l’administrateur permettant d’entrer l’URL du locataire et le jeton secret](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/provisioning.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d’erreur de provisionnement et sélectionnez la case à cocher **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Sous la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Oracle Cloud Infrastructure Console**.

9. Dans la section **Mappages des attributs**, passez en revue les attributs des utilisateurs qui sont synchronisés entre Azure AD et Oracle Cloud Infrastructure Console. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour mettre en correspondance les comptes d’utilisateur dans Oracle Cloud Infrastructure Console pour les opérations de mise à jour. Si vous choisissez de changer l’[attribut cible correspondant](../app-provisioning/customize-application-attributes.md), vous devez vérifier que l’API Oracle Cloud Infrastructure Console prend en charge le filtrage des utilisateurs en fonction de cet attribut. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    |Attribut|Type|
    |---|---|
    |displayName|String|
    |userName|String|
    |active|Boolean|
    |title|String|
    |emails[type eq "work"].value|String|
    |preferredLanguage|String|
    |name.givenName|String|
    |name.familyName|String|
    |addresses[type eq "work"].formatted|String|
    |addresses[type eq "work"].locality|String|
    |addresses[type eq "work"].region|String|
    |addresses[type eq "work"].postalCode|String|
    |addresses[type eq "work"].country|String|
    |addresses[type eq "work"].streetAddress|String|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|String|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|String|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|String|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|String|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Informations de référence|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|String|
    |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:bypassNotification|Boolean|
    |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:isFederatedUser|Boolean|

10. Sous la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Oracle Cloud Infrastructure Console**.

11. Dans la section **Mappages des attributs**, passez en revue les attributs des groupes qui sont synchronisés entre Azure AD et Oracle Cloud Infrastructure Console. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour mettre en correspondance les groupes dans Oracle Cloud Infrastructure Console pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    | Attribut | Type |
    |--|--|
    | displayName | String |
    | externalId | String |
    | membres | Informations de référence |

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service Provisionnement Azure AD pour Oracle Cloud Infrastructure Console, définissez **État du provisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous voulez provisionner sur Oracle Cloud Infrastructure Console en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre le cycle de synchronisation initiale de tous les utilisateurs et groupes définis dans **Étendue** dans la section **Paramètres**. Le cycle de synchronisation initiale prend plus de temps que les cycles de synchronisation suivants, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. 

## <a name="step-6-monitor-your-deployment"></a>Étape 6. Surveiller votre déploiement
Une fois que vous avez configuré l’approvisionnement, utilisez les ressources suivantes pour surveiller votre déploiement :

* Utilisez les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) pour déterminer quels utilisateurs ont été configurés avec succès ou échoué.
* Consultez la [barre de progression](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) pour afficher l’état du cycle d’approvisionnement et quand il se termine
* Si la configuration de l’approvisionnement semble se trouver dans un état non sain, l’application passe en quarantaine. Pour en savoir plus sur les états de quarantaine, cliquez [ici](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
