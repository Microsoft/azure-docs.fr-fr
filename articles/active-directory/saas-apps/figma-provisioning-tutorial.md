---
title: 'Tutoriel : Configurer Figma pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Figma.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: ec509cc04db87602b3e29230253d9b363a5e30d5
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353916"
---
# <a name="tutorial-configure-figma-for-automatic-user-provisioning"></a>Tutoriel : Configurer Figma pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Figma et Azure Active Directory (Azure AD) afin de configurer Azure AD pour le provisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur Figma.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire Figma](https://www.figma.com/pricing/).
* un compte d’utilisateur dans Figma avec des autorisations d’administration.

## <a name="assign-users-to-figma"></a>Affectez des utilisateurs à Figma.
Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Figma. Une fois que vous avez choisi, vous pouvez assigner ces utilisateurs et/ou groupes à votre application Figma en suivant les instructions fournies ici :
 
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)
## <a name="important-tips-for-assigning-users-to-figma"></a>Conseils importants pour l’affectation d’utilisateurs à Figma

 * Il est recommandé de n’affecter qu’un seul utilisateur Azure AD à Figma afin de tester la configuration du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous assignez un utilisateur à Figma, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="set-up-figma-for-provisioning"></a>Configurer Figma pour l’approvisionnement

Avant de configurer Figma pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devrez récupérer certaines informations de configuration à partir de Figma.

1. Connectez-vous à votre [Console d’administration Figma](https://www.Figma.com/). Cliquez sur l’icône d’engrenage à côté de votre locataire.

    :::image type="content" source="media/Figma-provisioning-tutorial/image0.png" alt-text="Capture d’écran de la console d’administration Figma. Un locataire nommé AAD Scim Test est visible. À côté du locataire, une icône d’engrenage est mise en évidence." border="false":::

2. Accédez à **Général > Mettre à jour les paramètres de connexion**.

    :::image type="content" source="media/Figma-provisioning-tutorial/figma03.png" alt-text="Capture d’écran de l’onglet General de la console d’administration Figma. Sous Log in and provisioning, Update log in settings est mis en évidence." border="false":::

3. Copiez **l’ID de locataire**. Cette valeur sera utilisée pour construire l’URL de point de terminaison SCIM à entrer dans le champ **URL de locataire** dans l’onglet Approvisionnement de votre application Figma dans le portail Azure.

    :::image type="content" source="media/Figma-provisioning-tutorial/figma-tenantid.png" alt-text="Capture d’écran de la section SAMLSSO dans la console d’administration Figma. Une étiquette d’ID de locataire et un lien adjacent indiquant Copy sont mis en évidence." border="false":::

4. Défilez vers le bas, puis cliquez sur **Générer un jeton d’API**.

    :::image type="content" source="media/Figma-provisioning-tutorial/token.png" alt-text="Capture d’écran de la section de provisionnement SCIM dans la console d’administration Figma. Un lien intitulé Generate API token est mis en évidence." border="false":::

5. Copiez la valeur du **Jeton d’API**. Cette valeur devra être entrée dans le champ **Jeton secret** dans l’onglet Approvisionnement de votre application Figma dans le portail Azure. 

    :::image type="content" source="media/Figma-provisioning-tutorial/figma04.png" alt-text="Capture d’écran d’une page dans la console d’administration Figma. Sous Your provisioning API token, un espace réservé pour le jeton est mis en évidence." border="false":::

## <a name="add-figma-from-the-gallery"></a>Ajouter Figma depuis la galerie

Avant de configurer Figma pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Figma à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Figma**, sélectionnez **Figma** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Figma dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-figma"></a>Configuration du provisionnement automatique d’utilisateurs sur Figma 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Figma en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour Figma en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour Figma](figma-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-figma--in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour Figma dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Figma**.

    ![Lien Figma dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification administrateur**, entrez `https://www.figma.com/scim/v2/<TenantID>` dans **URL de locataire**, où **TenantID** est la valeur que vous avez récupérée à partir de Figma. Entrez la valeur du **jeton d’API** dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Figma. Si la connexion échoue, vérifiez que votre compte Figma dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

8. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Figma**.

    ![Figma - Mappages d’utilisateurs](media/Figma-provisioning-tutorial/figma05.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Figma. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateurs dans Figma pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Figma](media/Figma-provisioning-tutorial/figma06.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service de provisionnement Azure AD pour Figma, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous souhaitez provisionner sur Figma en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Figma.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)