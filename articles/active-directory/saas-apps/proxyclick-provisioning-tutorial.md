---
title: 'Tutoriel : Configurer Proxyclick pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour attribuer et retirer automatiquement des comptes d’utilisateurs sur Proxyclick.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/3/2019
ms.author: jeedes
ms.openlocfilehash: f7d2a6f01e891a7fb1c14cde552d66679e474139
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359168"
---
# <a name="tutorial-configure-proxyclick-for-automatic-user-provisioning"></a>Tutoriel : Configurer Proxyclick pour l’attribution automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Proxyclick et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’attribution et le retrait automatiques d’utilisateurs et/ou de groupes sur Proxyclick.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* un [locataire Proxyclick](https://www.proxyclick.com/pricing) ;
* un compte d’utilisateur dans Proxyclick avec des autorisations d’administrateur.

## <a name="add-proxyclick-from-the-gallery"></a>Ajouter Proxyclick à partir de la galerie

Avant de configurer Proxyclick pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter Proxyclick à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Proxyclick à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Proxyclick** , sélectionnez **Proxyclick** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Proxyclick dans la liste des résultats](common/search-new-app.png)

## <a name="assigning-users-to-proxyclick"></a>Attribution d’utilisateurs à Proxyclick

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Proxyclick. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Proxyclick en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-proxyclick"></a>Conseils importants pour l’attribution d’utilisateurs à Proxyclick

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Proxyclick afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Proxyclick, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-proxyclick"></a>Configuration de l’attribution automatique d’utilisateurs sur Proxyclick 

Cette section vous guide tout au long des étapes de configuration du service d’attribution d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Proxyclick en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Proxyclick en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Proxyclick](proxyclick-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-proxyclick-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Proxyclick dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Proxyclick**.

    ![Lien Proxyclick dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Pour récupérer l’ **URL de locataire** et le **jeton secret** de votre compte Proxyclick, suivez la procédure pas à pas, comme décrit à l’étape 6.

6. Connectez-vous à votre [Console d’administration Proxyclick](https://app.proxyclick.com/login//?destination=%2Fdefault). Accédez à **Paramètres** > **Intégrations** > **Browse Marketplace (Parcourir la Place de marché)** .

    ![Paramètres Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick09.png)

    ![Intégrations Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick01.png)

    ![Place de marché Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick02.png)

    Sélectionnez ensuite **Azure AD**. Cliquez sur **Install now (Installer maintenant)** .

    ![Proxyclick Azure AD](media/proxyclick-provisioning-tutorial/proxyclick03.png)

    ![Installation de Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick04.png)

    Sélectionnez **Attribution d’utilisateurs** et cliquez sur **Start integration (Démarrer l’intégration)** . 

    ![Attribution d’utilisateurs Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick05.png)

    Les options de configuration de paramètres appropriées doivent maintenant apparaître sous **Paramètres** > **Intégrations**. Sélectionnez **Paramètres** sous **Azure AD (attribution d’utilisateurs)** .

    ![Création Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick06.png)

    Vous pouvez trouver l’ **URL de locataire** et le **jeton secret** ici.

    ![Création de jeton Proxyclick](media/proxyclick-provisioning-tutorial/proxyclick07.png)

7. Après avoir renseigné les champs indiqués à l’étape 5, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à Proxyclick. Si la connexion échoue, vérifiez que votre compte Proxyclick dispose des autorisations d’administrateur et réessayez.

    ![par jeton](common/provisioning-testconnection-tenanturltoken.png)

8. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Users to Proxyclick** (Synchroniser les utilisateurs Azure Active Directory avec Proxyclick).

    ![Proxyclick Mappages d’utilisateurs](media/proxyclick-provisioning-tutorial/Proxyclick-user-mappings.png)

11. Dans la section **Mappage d’attributs** , passez en revue les attributs d’utilisateur qui sont synchronisés entre Azure AD et Proxyclick. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Proxyclick pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs d’utilisateur Proxyclick](media/proxyclick-provisioning-tutorial/Proxyclick-user-attribute.png)

13. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

14. Pour activer le service d’attribution Azure AD pour Proxyclick, modifiez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

15. Définissez les utilisateurs et/ou groupes que vous souhaitez attribuer sur Proxyclick en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

16. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’attribution, qui décrivent toutes les actions effectuées par le service d’attribution Azure AD sur Proxyclick.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* Dans Proxyclick, **emails** et **userName** doivent avoir la même valeur source. Les mises à jour d’attributs modifient l’autre valeur.
* Proxyclick ne prend pas en charge l’attribution pour les groupes.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

