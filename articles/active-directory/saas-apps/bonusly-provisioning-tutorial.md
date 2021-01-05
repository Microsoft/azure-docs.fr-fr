---
title: 'Tutoriel : Configurer Bonusly pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour attribuer et supprimer automatiquement des comptes d’utilisateur dans Bonusly.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: d8c3f64e5cb5269bfe7e555615f874ac3443c6eb
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357825"
---
# <a name="tutorial-configure-bonusly-for-automatic-user-provisioning"></a>Tutoriel : Configurer Bonusly pour l’attribution automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Bonusly et Azure Active Directory (Azure AD) dans le but de configurer Azure AD pour l’attribution et la suppression automatiques d’utilisateurs et/ou de groupes dans Bonusly.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Conditions préalables requises

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* un locataire Azure AD ;
* Un [locataire Bonusly](https://bonus.ly/pricing)
* Un compte d’utilisateur Bonusly avec des autorisations d’administrateur

> [!NOTE]
> L’intégration du provisionnement Azure AD repose sur [l’API REST Bonusly](https://konghq.com/solutions/gateway/), qui est disponible pour les développeurs Bonusly.

## <a name="adding-bonusly-from-the-gallery"></a>Ajout de Bonusly à partir de la galerie

Avant de configurer Bonusly pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter Bonusly à votre liste d’applications SaaS managées, à partir de la galerie d’applications Azure AD.

**Pour ajouter Bonusly à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Bonusly** , sélectionnez **Bonusly** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Bonusly dans la liste des résultats](common/search-new-app.png)

## <a name="assigning-users-to-bonusly"></a>Attribution d’utilisateurs à Bonusly

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs et/ou les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à Bonusly. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Bonusly en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-bonusly"></a>Conseils importants pour l’attribution d’utilisateurs dans Bonusly

* Dans un premier temps, il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Bonusly afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Bonusly, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-bonusly"></a>Configuration de l’attribution automatique d’utilisateurs dans Bonusly

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Bonusly en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Bonusly en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Bonusly](bonus-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-bonusly-in-azure-ad"></a>Si vous souhaitez configurer l’attribution automatique d’utilisateurs pour Bonusly dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Bonusly**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Bonusly**.

    ![Lien Bonusly dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningTab.png" alt-text="Capture d’écran de Bonusly - onglet Approvisionnement. Sous Gérer, Approvisionnement est mis en surbrillance." border="false":::

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningCredentials.png" alt-text="Capture d’écran montrant une zone de liste du mode d’approvisionnement, avec Automatique sélectionné et mis en surbrillance." border="false":::

5. Sous la section **Informations d’identification de l’administrateur** , entrez le **jeton secret** de votre compte Bonusly, comme expliqué à l’étape 6.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/secrettoken.png" alt-text="Capture d’écran de la section Informations d’identification de l’administrateur. La zone Jeton secret est vide, mais la zone est mise en surbrillance." border="false":::

6. Le **jeton secret** de votre compte se trouve dans **Admin > Company > Integrations** (Admin > Société > Intégrations). Dans la section **If you want to code** (Si vous souhaitez programmer), cliquez sur **API > Create New API Access Token** (API > Créer un jeton d’accès d’API) pour créer un jeton secret.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/BonuslyIntegrations.png" alt-text="Capture d’écran du menu Bonusly. Sous Admin, Company (Société) est mis en surbrillance. Sous Company (Société), Integrations (Intégrations) est mis en surbrillance." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/BonsulyRestApi.png" alt-text="Capture d’écran de la section If you want to code (Si vous souhaitez programmer) du site Bonusly, avec API mis en surbrillance." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/CreateToken.png" alt-text="Capture d’écran du site Bonusly. L’onglet Services est ouvert. Sous Your API access tokens (Vos jetons d’accès d’API), Create new API access token (Créer un jeton d’accès d’API) est mis en surbrillance." border="false":::

7. Dans l’écran suivant, tapez un nom pour le jeton d’accès dans la zone de texte fournie, puis appuyez sur **Create Api Key** (Créer la clé d’API). Le nouveau jeton d’accès s’affiche pendant quelques secondes dans une fenêtre contextuelle.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/Token01.png" alt-text="Capture d’écran de la page New access token (Nouveau jeton d’accès) du site Bonusly. Une zone sans étiquette contient My Token (Mon jeton), et le bouton Create API key (Créer une clé d’API) est mis en surbrillance." border="false":::

    :::image type="content" source="./media/bonusly-provisioning-tutorial/Token02.png" alt-text="Capture d’écran du site Bonusly. Une notification indiquant qu’un nouveau jeton d’accès est créé s’affiche, suivi d’un jeton indéchiffrable." border="false":::

8. Après avoir renseigné les champs indiqués à l’étape 5, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Bonusly. Si la connexion échoue, vérifiez que votre compte Bonusly dispose des autorisations d’administrateur, puis réessayez.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/TestConnection.png" alt-text="Capture d’écran de la section Informations d’identification de l’administrateur du portail Azure. Le bouton Tester la connexion est mis en surbrillance." border="false":::

9. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/EmailNotification.png" alt-text="Capture d’écran montrant une boîte de messagerie de notification vide. Une option intitulée Envoyer une notification par e-mail en cas de défaillance s’affiche lorsqu’une défaillance se produit." border="false":::

10. Cliquez sur **Enregistrer**.

11. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Bonusly**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/UserMappings.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec Bonusly est mis en surbrillance." border="false":::

12. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Bonusly. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur Bonusly en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/UserAttributeMapping.png" alt-text="Capture d’écran de la page Mappages des attributs. Un tableau répertorie les attributs Azure Active Directory, les attributs Bonusly correspondants et l’état de correspondance." border="false":::

13. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

14. Pour activer le service de provisionnement Azure AD pour Bonusly, définissez le paramètre **État de la configuration** sur **Activé** dans la section **Paramètres**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ProvisioningStatus.png" alt-text="Capture d’écran de la page Paramètres. Le bouton bascule État de la configuration est défini sur Désactivé." border="false":::

15. Définissez les utilisateurs et/ou groupes que vous souhaitez attribuer à Bonusly en choisissant les valeurs souhaitées dans **Étendue** , dans la section **Paramètres**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/ScopeSync.png" alt-text="Capture d’écran montrant la zone de liste Étendue. Synchroniser uniquement les utilisateurs et groupes assignés est sélectionné dans la zone." border="false":::

16. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    :::image type="content" source="./media/bonusly-provisioning-tutorial/SaveProvisioning.png" alt-text="Capture d’écran de la page Bonusly - Approvisionnement, avec le bouton Enregistrer mis en surbrillance." border="false":::

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD dans Bonusly.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/bonusly-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/bonusly-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/bonusly-provisioning-tutorial/tutorial_general_03.png