---
title: 'Tutoriel : Configurer iPass SmartConnect pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner des comptes d’utilisateur dans iPass SmartConnect.
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
ms.openlocfilehash: 405a7bc3b653ca7bca026d3318763a4922244e88
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093704"
---
# <a name="tutorial-configure-ipass-smartconnect-for-automatic-user-provisioning"></a>Tutoriel : Configurer iPass SmartConnect pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à suivre dans iPass SmartConnect et Azure Active Directory (Azure AD) afin de configurer Azure AD pour approvisionner et déprovisionner automatiquement des utilisateurs et/ou des groupes dans iPass SmartConnect.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire iPass SmartConnect](https://www.ipass.com/buy-ipass/).
* Un compte d’utilisateur dans iPass SmartConnect avec des autorisations d’administrateur.

## <a name="assigning-users-to-ipass-smartconnect"></a>Affectation d’utilisateurs à iPass SmartConnect

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à iPass SmartConnect. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à iPass SmartConnect en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-ipass-smartconnect"></a>Conseils importants pour l’affectation d’utilisateurs à iPass SmartConnect

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à iPass SmartConnect afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à iPass SmartConnect, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-ipass-smartconnect-for-provisioning"></a>Configurer iPass SmartConnect pour l’approvisionnement

Avant de configurer iPass SmartConnect pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez récupérer les informations de configuration à partir de la console d’administration iPass SmartConnect :

1. Pour récupérer le jeton du porteur nécessaire pour l’authentification auprès de votre point de terminaison iPass SmartConnect SCIM, reportez-vous à la première fois où vous configurez iPass SmartConnect, car cette valeur n’est fournie qu’à ce moment-là. 
2. Si vous ne disposez pas du jeton du porteur, contactez l'[équipe du support technique d'iPass SmartConnect](mailto:help@ipass.com) pour en récupérer un nouveau.

## <a name="add-ipass-smartconnect-from-the-gallery"></a>Ajouter iPass SmartConnect à partir de la galerie

Avant de configurer iPass SmartConnect pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter iPass SmartConnect à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter iPass SmartConnect à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **iPass SmartConnect**, sélectionnez **iPass SmartConnect** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![iPass SmartConnect dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-ipass-smartconnect"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur iPass SmartConnect 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans iPass SmartConnect en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
>  Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour iPass SmartConnect en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour iPass SmartConnect](ipasssmartconnect-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-ipass-smartconnect-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour iPass SmartConnect dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **iPass SmartConnect**.

    ![Lien iPass SmartConnect dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://openmobile.ipass.com/moservices/scim/v1` dans **URL de locataire**. Entrez le jeton du porteur récupéré précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à iPass SmartConnect. Si la connexion échoue, vérifiez que votre compte iPass SmartConnect dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur iPass SmartConnect**.

    :::image type="content" source="media/ipass-smartconnect-provisioning-tutorial/usermapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec iPass SmartConnect est visible." border="false":::

9. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers iPass SmartConnect dans la section **Mappage d’attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans iPass SmartConnect pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    :::image type="content" source="media/ipass-smartconnect-provisioning-tutorial/userattribute.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau liste les attributs Azure Active Directory et iPass SmartConnect et la priorité de correspondance." border="false":::


10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service d’approvisionnement Azure AD pour iPass SmartConnect, accédez à la section **Paramètres** et définissez l’**État de l’approvisionnement** sur **Activé**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous aimeriez approvisionner dans iPass SmartConnect en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité d’approvisionnement, qui décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur iPass SmartConnect.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* iPass SmartConnect accepte uniquement les noms d’utilisateur dont les domaines sont inscrits dans la console d’administration iPass SmartConnect.  

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
