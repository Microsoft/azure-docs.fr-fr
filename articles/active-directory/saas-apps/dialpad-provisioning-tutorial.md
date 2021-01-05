---
title: 'Tutoriel : Configurer Dialpad pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes utilisateur sur Dialpad.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/28/2019
ms.author: zhchia
ms.openlocfilehash: b88e618da3f8a23c0517aaeb251e54bf559fc468
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014514"
---
# <a name="tutorial-configure-dialpad-for-automatic-user-provisioning"></a>Tutoriel : Configurer Dialpad pour l’attribution automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Dialpad et Azure Active Directory (Azure AD) afin de configurer Azure AD pour le provisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Dialpad.

> [!NOTE]
>  Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

> Ce connecteur est en préversion. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire Dialpad](https://www.dialpad.com/pricing/).
* Un compte d’utilisateur dans Dialpad avec des autorisations d’administration.

## <a name="assign-users-to-dialpad"></a>Attribuer des utilisateurs à Dialpad
Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Dialpad. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Dialpad en suivant les instructions fournies ici :
 
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md) 

 ## <a name="important-tips-for-assigning-users-to-dialpad"></a>Conseils importants pour l’attribution d’utilisateurs à Dialpad

 * Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Dialpad afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Dialpad, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="setup-dialpad-for-provisioning"></a>Configurer Dialpad pour le provisionnement

Avant de configurer Dialpad pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez récupérer certaines informations de provisionnement à partir de Dialpad.

1. Connectez-vous à votre [console d’administration Dialpad](https://dialpadbeta.com/login) et sélectionnez **Admin settings** (Paramètres d’administration). Vérifiez que l’option **My Company** (Ma société) est sélectionnée dans la liste déroulante. Accédez à **Authentication > API Keys** (Authentification > Clés API).

    :::image type="content" source="media/dialpad-provisioning-tutorial/dialpad01.png" alt-text="Capture d’écran de la console d’administration Dialpad, avec l’icône des paramètres, My Company (Ma société), Authentication (Authentification) et API keys (Clés API) mis en surbrillance, et My Company (Ma société) sélectionné." border="false":::

2. Générez une nouvelle clé en cliquant sur **Add a key** (Ajouter une clé) et en configurant les propriétés du jeton de votre secret.

    :::image type="content" source="media/dialpad-provisioning-tutorial/dialpad02.png" alt-text="Capture d’écran de la page API keys (Clés API) de la console d’administration Dialpad. Add a key (Ajouter une clé) est mis en surbrillance." border="false":::

    :::image type="content" source="media/dialpad-provisioning-tutorial/dialpad03.png" alt-text="Capture d’écran de la page Edit API key (Modifier une clé API) de la console d’administration Dialpad. Le bouton Enregistrer est en surbrillance." border="false":::

3. Cliquez sur le bouton **Click to show value** (Cliquer pour afficher la valeur) associé à votre clé API récemment créée et copiez la valeur affichée. Cette valeur devra être entrée dans le champ **Jeton secret** sous l’onglet Provisionnement de votre application Dialpad dans le portail Azure. 

    ![Dialpad - Créer le jeton](media/dialpad-provisioning-tutorial/dialpad04.png)

## <a name="add-dialpad-from-the-gallery"></a>Ajouter Dialpad à partir de la galerie

Avant de configurer Dialpad pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter Dialpad à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS gérées.

**Pour ajouter Dialpad à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Dialpad**, puis sélectionnez **Dialpad** dans le volet de résultats.
    ![Dialpad dans la liste des résultats](common/search-new-app.png)

5. Accédez à l’**URL** mise en évidence ci-dessous dans un autre navigateur. 

    :::image type="content" source="media/dialpad-provisioning-tutorial/dialpad05.png" alt-text="Capture d’écran d’une page affichant des informations sur l’application Dialpad. Sous URL, une adresse est répertoriée et mise en surbrillance." border="false":::

6. Dans le coin supérieur droit, sélectionnez **Log In > Use Dialpad online** (Se connecter> Utiliser Dialpad en ligne).

    :::image type="content" source="media/dialpad-provisioning-tutorial/dialpad06.png" alt-text="Capture d’écran du site web de Dialpad. Log in (Se connecter) est mis en surbrillance et l’onglet Log in (Se connecter) est ouvert. Use Dialpad online (Utiliser Dialpad en ligne) est également mis en surbrillance." border="false":::

7. Comme Dialpad est une application OpenIDConnect, choisissez de vous connecter à Dialpad à l’aide de votre compte professionnel Microsoft.

    :::image type="content" source="media/dialpad-provisioning-tutorial/loginpage.png" alt-text="Capture d’écran de la page Start making calls (Commencer à passer des appels) sur le site web de Dialpad. Le bouton Log in with Office 365 (Se connecter avec Office 365) est mis en surbrillance." border="false":::

8. Une fois l’authentification réussie, acceptez l’invite de consentement pour la page de consentement. L’application est alors automatiquement ajoutée à votre locataire et vous serez redirigé vers votre compte Dialpad.

    :::image type="content" source="media/dialpad-provisioning-tutorial/redirect.png" alt-text="Capture d’écran montrant une page d’authentification Microsoft indiquant que l’application Dialpad a demandé l’accès à certaines données. Le bouton Accept (Accepter) est mis en surbrillance." border="false":::

 ## <a name="configure-automatic-user-provisioning-to-dialpad"></a>Configurer l’attribution automatique d’utilisateurs sur Dialpad

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Dialpad en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-dialpad-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Dialpad dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Dialpad**.

    ![Lien Dialpad dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://dialpad.com/scim` dans **URL de locataire**. Entrez la valeur que vous avez récupérée et enregistrée à partir de Dialpad dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Dialpad. Si la connexion échoue, vérifiez que votre compte Dialpad dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Dialpad**.

    ![Dialpad - Mappages d’utilisateur](media/dialpad-provisioning-tutorial/dialpad-user-mappings-new.png)

9. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Dialpad. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondance des comptes d’utilisateur dans Dialpad dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Dialpad](media/dialpad-provisioning-tutorial/dialpad07.png)

10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service de provisionnement Azure AD pour Dialpad, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous souhaitez provisionner sur Dialpad en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour superviser la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Dialpad.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md)
##  <a name="connector-limitations"></a>Limitations du connecteur
* Dialpad ne prend pas en charge le renommage des groupes pour l’instant. Cela signifie que toute modification apportée à la propriété **displayName** d’un groupe dans Azure AD ne sera pas mise à jour et reflétée dans Dialpad.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
