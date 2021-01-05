---
title: 'Tutoriel : Configurer un répertoire fédéré pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner des comptes d’utilisateur dans le répertoire fédéré.
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
ms.openlocfilehash: 8ca7654d930247f70d85cbc20fbbeb961223f05f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998360"
---
# <a name="tutorial-configure-federated-directory-for-automatic-user-provisioning"></a>Tutoriel : Configurer un répertoire fédéré pour l’attribution automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans le répertoire fédéré et dans Azure Active Directory (Azure AD) dans le but de configurer Azure AD pour l’attribution et la suppression automatiques d’utilisateurs et/ou de groupes dans le répertoire fédéré.

> [!NOTE]
>  Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un répertoire fédéré](https://www.federated.directory/pricing).
* Un compte d’utilisateur de répertoire fédéré avec des autorisations d’administrateur.

## <a name="assign-users-to-federated-directory"></a>Affecter des utilisateurs au répertoire fédéré
Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder au répertoire fédéré. Une fois que vous avez fait votre choix, vous pouvez attribuer ces utilisateurs et/ou groupes à votre répertoire fédéré en suivant les instructions fournies ici :

 * [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md) 
 
 ## <a name="important-tips-for-assigning-users-to-federated-directory"></a>Conseils importants pour l’attribution d’utilisateurs au répertoire fédéré
 * Dans un premier temps, il est recommandé de n’attribuer qu’un seul utilisateur Azure AD au répertoire fédéré afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur au répertoire fédéré, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.
    
 ## <a name="set-up-federated-directory-for-provisioning"></a>Configurer un répertoire fédéré pour l’approvisionnement

Avant de configurer un répertoire fédéré pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur le répertoire fédéré.

1. Connectez-vous à la [console d’administration de votre répertoire fédéré](https://federated.directory/of)

    :::image type="content" source="media/federated-directory-provisioning-tutorial/companyname.png" alt-text="Capture d’écran de la console d’administration Répertoire fédéré montrant un champ permettant d’entrer un nom de société. Les boutons de connexion sont également visibles." border="false":::

2. Accédez à **Répertoires > Répertoires utilisateur** et sélectionnez votre locataire. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/ad-user-directories.png" alt-text="Capture d’écran de la console d’administration Répertoire fédéré, avec Répertoires et le Test du répertoire fédéré Azure AD mis en évidence." border="false":::

3.  Pour générer un jeton du porteur permanent, accédez à **Clés de répertoire > Créer une nouvelle clé.** 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated01.png" alt-text="Capture d’écran de la page Clés de répertoire de la console d’administration Répertoire fédéré. Le bouton Créer une nouvelle clé est mis en évidence." border="false":::

4. Créez une clé de répertoire. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated02.png" alt-text="Capture d’écran de la page Créer une clé de répertoire de la console d’administration Répertoire fédéré, avec les champs Nom et Description et un bouton Créer une clé." border="false":::
    

5. Copiez la valeur **Jeton d’accès**. Cette valeur devra être entrée dans le champ **Jeton secret** dans l’onglet Approvisionnement de votre application de répertoire fédéré dans le portail Azure. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated03.png" alt-text="Capture d’écran d’une page dans la console d’administration Répertoire fédéré. Un espace réservé de jeton d’accès et un nom de clé, une description et un émetteur sont visibles." border="false":::
    
## <a name="add-federated-directory-from-the-gallery"></a>Ajouter un répertoire fédéré à partir de la galerie

Avant de configurer un répertoire fédéré pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter un répertoire fédéré à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter un répertoire fédéré à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **« répertoire fédéré »** , sélectionnez **Répertoire fédéré** dans le volet de résultats.

    ![Répertoire fédéré dans la liste des résultats](common/search-new-app.png)

5. Accédez à l’**URL** mise en évidence ci-dessous dans un autre navigateur. 

    :::image type="content" source="media/federated-directory-provisioning-tutorial/loginpage1.png" alt-text="Capture d’écran d’une page du portail Azure qui affiche des informations relatives au répertoire fédéré. La valeur URL est mise en évidence." border="false":::

6. Cliquez sur **Ouvrir une session**.

    :::image type="content" source="media/federated-directory-provisioning-tutorial/federated04.png" alt-text="Capture d’écran du menu principal sur le site Répertoire fédéré. Le bouton Connexion est en évidence." border="false":::

7.  Comme Répertoire fédéré est une application OpenIDConnect, choisissez de vous connecter au répertoire fédéré à l’aide de votre compte professionnel Microsoft.
    
    :::image type="content" source="media/federated-directory-provisioning-tutorial/loginpage3.png" alt-text="Capture d’écran de la page du test SCIM AD sur le site Répertoire fédéré. Connectez-vous avec votre compte Microsoft est mis en évidence." border="false":::
 
8. Une fois l’authentification réussie, acceptez l’invite de consentement pour la page de consentement. L’application est alors automatiquement ajoutée à votre client et vous serez redirigé vers votre compte de répertoire fédéré.

    ![répertoire fédéré Ajouter SCIM](media/federated-directory-provisioning-tutorial/premission.png)



## <a name="configuring-automatic-user-provisioning-to-federated-directory"></a>Configuration de l’approvisionnement automatique d’utilisateurs dans le répertoire fédéré 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans le répertoire fédéré en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-federated-directory-in-azure-ad"></a>Si vous souhaitez configurer l’attribution automatique d’utilisateurs pour le répertoire fédéré dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Répertoire fédéré**.

    ![Lien vers le répertoire fédéré dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d’identification de l’administrateur**, entrez `https://api.federated.directory/v2/` dans URL de locataire. Entrez la valeur que vous avez récupérée et enregistrée précédemment à partir du répertoire fédéré dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter au répertoire fédéré. Si la connexion échoue, vérifiez que votre compte de répertoire fédéré dispose des autorisations d’administrateur, puis réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

8. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec un répertoire fédéré**.

    :::image type="content" source="media/federated-directory-provisioning-tutorial/user-mappings.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec Répertoire fédéré est mis en évidence." border="false":::
    
    
11. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et le répertoire fédéré. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans le répertoire fédéré à des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    :::image type="content" source="media/federated-directory-provisioning-tutorial/user-attributes.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau répertorie les attributs Azure Active Directory et Répertoire fédéré et l’état de correspondance." border="false":::
    

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’attribution d’utilisateurs Azure AD pour le répertoire fédéré, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes que vous souhaitez attribuer au répertoire fédéré en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité de provisionnement, qui décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur le répertoire fédéré.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md)
## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
