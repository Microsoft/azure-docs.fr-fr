---
title: 'Tutoriel : Configurer Keeper Password Manager & Digital Vault pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d’utilisateur sur Keeper Password Manager & Digital Vault.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2019
ms.author: jeedes
ms.openlocfilehash: 2670dc0cb56805a2afa966bee1d2aa52b6c8e46a
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358977"
---
# <a name="tutorial-configure-keeper-password-manager--digital-vault-for-automatic-user-provisioning"></a>Tutoriel : Configurer Keeper Password Manager & Digital Vault pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Keeper Password Manager & Digital Vault et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur Keeper Password Manager & Digital Vault.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Keeper Password Manager & Digital Vault](https://keepersecurity.com/pricing.html?t=e)
* Un compte d’utilisateur Keeper Password Manager & Digital Vault avec des autorisations d’administrateur.

## <a name="add-keeper-password-manager--digital-vault-from-the-gallery"></a>Ajouter de Keeper Password Manager & Digital Vault à partir de la galerie

Avant de configurer Keeper Password Manager & Digital Vault pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Keeper Password Manager & Digital Vault à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Keeper Password Manager & Digital Vault à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Keeper Password Manager & Digital Vault** , sélectionnez **Keeper Password Manager & Digital Vault** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Keeper Password Manager & Digital Vault dans la liste des résultats](common/search-new-app.png)

## <a name="assigning-users-to-keeper-password-manager--digital-vault"></a>Affectation d’utilisateurs à Keeper Password Manager & Digital Vault

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Keeper Password Manager & Digital Vault. Une fois que vous avez choisi, vous pouvez affecter ces utilisateurs et/ou groupes à Keeper Password Manager & Digital Vault en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-keeper-password-manager--digital-vault"></a>Conseils importants pour l’affectation d’utilisateurs à Keeper Password Manager & Digital Vault

* Il est recommandé de n’affecter qu’un seul utilisateur Azure AD à Keeper Password Manager & Digital Vault afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous affectez un utilisateur à Keeper Password Manager & Digital Vault, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-keeper-password-manager--digital-vault"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Keeper Password Manager & Digital Vault 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Keeper Password Manager & Digital Vault en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Keeper Password Manager & Digital Vault en suivant les instructions fournies dans le [Didacticiel d’authentification unique de Keeper Password Manager & Digital Vault](keeperpasswordmanager-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-keeper-password-manager--digital-vault-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Keeper Password Manager & Digital Vault dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Keeper Password Manager & Digital Vault**.

    ![Lien Keeper Password Manager & Digital Vault dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez les valeurs **URL de locataire** et **Jeton secret** de votre compte Keeper Password Manager & Digital Vault, comme cela est décrit dans l’étape 6.

6. Connectez-vous à votre [console d’administration Keeper](https://keepersecurity.com/console/#login). Cliquez sur **Administrateur** et sélectionnez un nœud existant ou créez-en un. Accédez à l’onglet **Approvisionnement** et sélectionnez **Ajouter une méthode**.

    ![Console d’administration Keeper](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-admin-console.png)

    Sélectionnez **SCIM (System for Cross-Domain Identity Management)** .

    ![Keeper Ajouter SCIM](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-add-scim.png)

    Cliquez sur **Create Provisioning Token** (Créer un jeton d’approvisionnement).

    ![Créer un point de terminaison Keeper](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-endpoint.png)

    Copiez les valeurs pour **URL** et **Jeton** et collez-les dans **URL de locataire** et **Jeton secret** dans Azure AD. Cliquez sur **Enregistrer** pour terminer la configuration de l’approvisionnement sur Keeper.

    ![Keeper Créer un jeton](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-create-token.png)

7. Après avoir renseigné les champs indiqués à l’étape 5, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à Keeper Password Manager & Digital Vault. Si la connexion échoue, vérifiez que votre compte Keeper Password Manager & Digital Vault dispose d’autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

8. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Keeper Password Manager & Digital Vault**.

    ![Keeper Mappages d’utilisateurs](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-mappings.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs d’utilisateurs qui sont synchronisés entre Azure AD et Keeper Password Manager & Digital Vault. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Keeper Password Manager & Digital Vault pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Keeper Attributs utilisateur](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-user-attributes.png)

12. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Keeper Password Manager & Digital Vault**.

    ![Keeper Mappages de groupes](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-mappings.png)

13. Dans la section **Mappages des attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Keeper Password Manager & Digital Vault. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Keeper Password Manager & Digital Vault pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Keeper Attributs de groupe](media/keeper-password-manager-digitalvault-provisioning-tutorial/keeper-group-attributes.png)

14. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service d’approvisionnement Azure AD pour Keeper Password Manager & Digital Vault, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

16. Définissez les utilisateurs et/ou groupes que vous aimeriez approvisionner sur Keeper Password Manager & Digital Vault en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

17. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour analyser la progression et suivre les liens vers les rapports d’activité d’approvisionnement, qui décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Keeper Password Manager & Digital Vault.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* Dans Keeper Password Manager & Digital Vault, **emails** et **userName** doivent avoir la même valeur source, car les mises à jour d’un de ces attributs modifieront l’autre valeur.
* Keeper Password Manager & Digital Vault ne prend pas en charge les suppressions d’utilisateur, uniquement les désactivations. Les utilisateurs désactivés apparaissent comme verrouillé dans l’interface utilisateur de la console d’administration Keeper.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

