---
title: 'Tutoriel : Configurer Meta Networks Connector pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour attribuer et supprimer automatiquement des comptes d’utilisateur dans Meta Networks Connector.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/01/2019
ms.author: Zhchia
ms.openlocfilehash: b6a8f192cd26639431cc9fcb6b43e1bc5e8e2843
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353627"
---
# <a name="tutorial-configure-meta-networks-connector-for-automatic-user-provisioning"></a>Tutoriel : Configurer Meta Networks Connector pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Meta Networks Connector et Azure Active Directory (Azure AD) dans le but de configurer Azure AD pour l’attribution et la suppression automatiques d’utilisateurs et/ou de groupes dans Meta Networks Connector.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Locataire Meta Networks Connector](https://www.metanetworks.com/)
* Un compte d’utilisateur dans Meta Networks Connector avec des autorisations d’administrateur.

## <a name="assigning-users-to-meta-networks-connector"></a>Affectation d’utilisateurs à Meta Networks Connector

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à Meta Networks Connector. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Meta Networks Connector en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-meta-networks-connector"></a>Conseils importants pour l’affectation d’utilisateurs à Meta Networks Connector

* Il est recommandé qu’un seul utilisateur Azure AD soit affecté à Meta Networks Connector pour tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Meta Networks Connector, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-meta-networks-connector-for-provisioning"></a>Configurer Meta Networks Connector pour l’approvisionnement

1. Connectez-vous à la [console d’administration Meta Networks Connector](https://login.metanetworks.com/login/) à l’aide du nom de votre organisation. Navigate to **Administration > Clés API**.

    ![Console d’administration Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/apikey.png)

2.  Cliquez sur le signe plus en haut à droite de l’écran pour créer une **clé API** .

    ![Icône plus Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/plusicon.png)

3.  Définissez le **Nom de la clé API** et la **Description de la clé API**.

    :::image type="content" source="media/meta-networks-connector-provisioning-tutorial/keyname.png" alt-text="Capture d’écran de la console d’administration Meta Networks Connector avec le nom de la clé API et les valeurs de description de la clé API d’Azure AD et une clé API mis en surbrillance." border="false":::

4.  Activez les privilèges **En écriture** pour les **Groupes** et les **Utilisateurs**.

    ![Privilèges Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/privileges.png)

5.  Cliquez sur **Ajouter**. Copiez le **SECRET** et enregistrez-le car il s’agit de la seule fois où vous pouvez le consulter. Cette valeur sera entrée dans le champ Jeton secret de l’onglet Approvisionnement de votre application Meta Networks Connector dans le Portail Azure.

    :::image type="content" source="media/meta-networks-connector-provisioning-tutorial/token.png" alt-text="Capture d’écran d’une fenêtre indiquant aux utilisateurs que la clé API a été ajoutée. La zone Secret contient une valeur indéchiffrable et est mise en surbrillance." border="false":::

6.  Ajoutez un IdP en accédant à **Administration > Paramètres > IdP > Créer**.

    ![Ajouter IdP Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/newidp.png)

7.  Dans la page **Configuration IdP**, vous pouvez **Nommer** votre configuration IdP et choisir une **Icône**.

    ![Nom IdP Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/idpname.png)

    ![Icône IdP Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/icon.png)

8.  Sous **Configurer SCIM**, sélectionnez le nom de clé API créé lors des étapes précédentes. Cliquez sur **Save**(Enregistrer).

    ![Configurer SCIM Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/configure.png)

9.  Accédez à **Administration > Paramètres > Onglet IdP** . Cliquez sur le nom de la configuration IdP créée lors des étapes précédentes pour afficher l’**ID IdP**. Cette **ID**  est ajoutée à la fin de l’**URL de locataire** lors de la saisie de la valeur dans le champ **URL de locataire** dans l’onglet Approvisionnement de votre application Meta Networks Connector dans le Portail Azure.

    ![ID IdP Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/idpid.png)

## <a name="add-meta-networks-connector-from-the-gallery"></a>Ajouter Meta Networks Connector à partir de la galerie

Avant de configurer Meta Networks Connector pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter Meta Networks Connector à votre liste d’applications SaaS managées, à partir de la galerie d’applications Azure AD.

**Pour ajouter Meta Networks Connector à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Meta Networks Connector**, sélectionnez **Meta Networks Connector** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Meta Networks Connector dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-meta-networks-connector"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Meta Networks Connector 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Meta Networks Connector en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour Meta Networks Connector en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour Meta Networks Connector](./metanetworksconnector-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-meta-networks-connector-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs Meta Networks Connector dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Meta Networks Connector**.

    ![Lien Meta Networks Connector dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://api.metanetworks.com/v1/scim/<IdP ID>` dans **URL de locataire**. Saisissez la valeur de **Jeton d’authentification SCIM** récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Meta Networks Connector. Si la connexion échoue, vérifiez que votre compte Meta Networks Connector dispose des autorisations d’administrateur, puis réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Meta Networks Connector**.

    ![Mappage d’utilisateurs Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Meta Networks Connector. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Meta Networks Connector pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs d’utilisateurs Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/userattributes.png)

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Meta Networks Connector**.

    ![Mappage de groupes Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupmappings.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Meta Networks Connector. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Meta Networks Connector pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs de groupes Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupattributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Meta Networks Connector, affectez la valeur **Activé** au paramètre **Statut d’approvisionnement** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous souhaitez approvisionner sur Meta Networks Connector en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD dans Meta Networks Connector.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)