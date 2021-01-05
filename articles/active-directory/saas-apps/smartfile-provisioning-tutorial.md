---
title: "Tutoriel : Configurer SmartFile pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Apprenez à configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d'utilisateur sur SmartFile.
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
ms.openlocfilehash: 03c7efd390d4714680ed2ccd54d297b8137aaafc
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348668"
---
# <a name="tutorial-configure-smartfile-for-automatic-user-provisioning"></a>Tutoriel : Configurer SmartFile pour l'approvisionnement automatique d'utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans SmartFile et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur SmartFile.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire SmartFile](https://www.SmartFile.com/pricing/).
* Un compte d'utilisateur SmartFile doté d'autorisations d'administrateur.

## <a name="assigning-users-to-smartfile"></a>Attribuer des utilisateurs à SmartFile

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer l'approvisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs et/ou groupes d'Azure AD auront besoin d'accéder à SmartFile. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à SmartFile en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-smartfile"></a>Conseils importants pour l'attribution d'utilisateurs à SmartFile

* Il est recommandé de n'attribuer qu'un seul utilisateur Azure AD à SmartFile afin de tester la configuration de l'approvisionnement automatique d'utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à SmartFile, vous devez sélectionner un rôle valide spécifique à l'application (si disponible) dans la boîte de dialogue d'attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-smartfile-for-provisioning"></a>Configurer SmartFile pour l'approvisionnement

Avant de configurer SmartFile pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez activer l'approvisionnement SCIM sur SmartFile et collecter les informations supplémentaires nécessaires.

1. Connectez-vous à votre Console d'administration SmartFile. Accédez au coin supérieur droit de la Console d'administration SmartFile. Sélectionnez **Clé du produit**.

    ![Console d'administration SmartFile](media/smartfile-provisioning-tutorial/login.png)

2. Pour générer un jeton de porteur, copiez la **Clé du produit** et le **Mot de passe du produit**. Collez-les dans un bloc-notes en entrant deux-points entre eux.
    
     ![Capture d’écran de la section Clé du produit avec les zones de texte Clé du produit et Mot de passe du produit en évidence.](media/smartfile-provisioning-tutorial/auth.png)

    ![Capture d’écran d’un texte en clair montrant la clé de produit et le mot de passe du produit séparés par un deux-points.](media/smartfile-provisioning-tutorial/key.png)

## <a name="add-smartfile-from-the-gallery"></a>Ajouter SmartFile à partir de la galerie

Avant de configurer SmartFile pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez ajouter SmartFile à votre liste d'applications SaaS gérées à partir de la galerie d'applications Azure AD.

**Pour ajouter SmartFile à partir de la galerie d'applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **SmartFile**, sélectionnez **SmartFile** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l'application.

    ![SmartFile dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-smartfile"></a>Configurer l'approvisionnement automatique d'utilisateurs sur SmartFile 

Cette section vous guide tout au long des étapes de configuration du service d'approvisionnement d'Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans SmartFile en fonction des attributions d'utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour SmartFile en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour SmartFile](SmartFile-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-smartfile-in-azure-ad"></a>Pour configurer l'approvisionnement automatique d'utilisateurs pour SmartFile dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SmartFile**.

    ![Lien SmartFile dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5.  Dans la section **Informations d’identification de l’administrateur**, entrez `https://<SmartFile sitename>.smartfile.com/ftp/scim` dans **URL de locataire**. Par exemple, `https://demo1test.smartfile.com/ftp/scim`. Entrez la valeur du **Jeton du porteur** (ProductKey: ProductPassword) précédemment récupérée dans le champ **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à SmartFile. Si la connexion échoue, vérifiez que votre compte SmartFile dispose d'autorisations d'administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec SmartFile**.

    ![Mappages d'utilisateurs SmartFile](media/smartfile-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappage des attributs**, passez en revue les attributs d'utilisateurs synchronisés entre Azure AD et SmartFile. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d'utilisateur dans SmartFile pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur SmartFile](media/smartfile-provisioning-tutorial/userattribute.png)

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec SmartFile**.

    ![Mappages de groupes SmartFile](media/smartfile-provisioning-tutorial/groupmapping.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs de groupes qui sont synchronisés entre Azure AD et SmartFile. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans SmartFile pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs de groupe SmartFile](media/smartfile-provisioning-tutorial/groupattribute.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d'approvisionnement Azure AD pour SmartFile, définissez le paramètre **État d'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes à approvisionner sur SmartFile en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

    Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et suivre les liens vers le rapport d'activité d'approvisionnement, qui décrit toutes les actions effectuées par le service d'approvisionnement Azure AD sur SmartFile.

    Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md)
    
## <a name="connector-limitations"></a>Limitations du connecteur

* SmartFile prend uniquement en charge les suppressions définitives. 

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

 [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
