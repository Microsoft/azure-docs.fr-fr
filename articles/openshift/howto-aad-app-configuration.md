---
title: Intégration d’Azure Active Directory pour Azure Red Hat OpenShift
description: Apprenez comment créer un groupe de sécurité et un utilisateur Azure AD pour tester des applications sur votre cluster Microsoft Azure Red Hat OpenShift.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: ee8613d0300a941f80577c98be106173d5d3ced1
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220701"
---
# <a name="azure-active-directory-integration-for-azure-red-hat-openshift"></a>Intégration d’Azure Active Directory pour Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Si vous n’avez pas encore créé un locataire Azure Active Directory (Azure AD), suivez les instructions dans [Créer un locataire Azure AD pour Azure Red Hat OpenShift](howto-create-tenant.md)avant de poursuivre avec ces instructions.

Microsoft Azure Red Hat OpenShift a besoin de permissions pour exécuter des tâches au nom de votre cluster. Si votre organisation n’a pas déjà un utilisateur Azure AD, un groupe de sécurité Azure AD ou un enregistrement d’application Azure AD à utiliser comme principal du service, suivez ces instructions pour les créer.

## <a name="create-a-new-azure-active-directory-user"></a>Création d’un nouvel utilisateur Azure Active Directory

Dans le [Portail Microsoft Azure](https://portal.azure.com), assurez-vous que votre locataire apparaît sous votre nom d’utilisateur dans le coin supérieur droit du portail :

![Capture d’écran du portail avec le locataire en haut à droite](./media/howto-create-tenant/tenant-callout.png). Si le mauvais locataire est affiché, cliquez sur votre nom d’utilisateur en haut à droite, puis cliquez sur **Commuter répertoire** , et sélectionnez le bon locataire dans la liste **Tous les répertoires** .

Créez un « propriétaire » Azure Active Directory pour vous connecter à votre cluster Azure Red Hat OpenShift.

1. Accédez au panneau [Utilisateurs-Tous les utilisateurs](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers).
2. Cliquez sur **+Nouvel utilisateur** pour ouvrir le volet **Utilisateur** .
3. Entrez un **Nom** pour cet utilisateur.
4. Créez un **nom d’utilisateur** basé sur le nom du locataire que vous avez créé, avec `.onmicrosoft.com` ajouté à la fin. Par exemple : `yourUserName@yourTenantName.onmicrosoft.com`. Notez ce nom d’utilisateur. Vous en aurez besoin pour vous connecter à votre cluster.
5. Cliquez sur **Rôle d’annuaire** pour ouvrir le volet de rôle d’annuaire, sélectionnez **Propriétaire** , puis cliquez sur **OK** en bas du volet.
6. Dans le volet **Utilisateur** , cliquez sur **Afficher le mot de passe** et enregistrez le mot de passe temporaire. Après vous être connecté la première fois, vous serez invité à le réinitialiser.
7. En bas du volet, cliquez sur **Créer** pour créer l’utilisateur.

## <a name="create-an-azure-ad-security-group"></a>Créer un groupe de sécurité Azure AD

Pour accorder l’accès à l’administrateur du cluster, les membres d’un groupe de sécurité Azure AD sont synchronisés dans le groupe OpenShift « osa-customer-admins ». S’il n’est pas spécifié, aucun accès administrateur du cluster ne sera accordé.

1. Ouvrez le panneau [Groupes Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups).
2. Cliquez sur **+Nouveau groupe** .
3. Entrez un nom et une description pour le groupe.
4. Définissez **Type de groupe** sur **Sécurité** .
5. Définissez **Type d’appartenance** sur **Affecté** .

    Ajoutez à ce groupe de sécurité l’utilisateur Azure AD que vous avez créé à l’étape précédente.

6. Cliquez sur **Membres** pour ouvrir le volet **Sélectionner des membres** .
7. Dans la liste des membres, sélectionnez l’utilisateur Azure AD que vous avez créé précédemment.
8. En bas du portail, cliquez sur **Sélectionner** , puis **Créer** pour créer le groupe de sécurité.

    Notez la valeur ID de groupe.

9. Lorsque le groupe est créé, vous le verrez dans la liste de tous les groupes. Cliquez sur le nouveau groupe.
10. Dans la page qui s’affiche, notez l’ **ID d’objet** . Nous nous référerons à cette valeur en tant que `GROUPID` dans le didacticiel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

> [!IMPORTANT]
> Pour synchroniser ce groupe avec le groupe OpenShift osa-customer-admins, créez le cluster en utilisant Azure CLI. Le portail Azure ne dispose pas pour l’instant d’un champ pour définir ce groupe.

## <a name="create-an-azure-ad-app-registration"></a>Créer une inscription d’application Azure AD

Vous pouvez créer automatiquement un client d’inscription d’application Azure Active Directory (Azure AD) dans le cadre de la création du cluster en omettant l’indicateur `--aad-client-app-id` de la commande `az openshift create`. Ce didacticiel vous montre comment créer l’inscription d’application Azure AD par souci d’exhaustivité.

Si votre organisation n’a pas encore d’inscription d’application Azure Active Directory (Azure AD) à utiliser en tant que service principal, suivez ces instructions pour créer une.

1. Ouvrez le panneau [Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview), puis cliquez sur **+Nouvelle inscription** .
2. Sur la page **Inscrire une application** , entrez un nom pour votre inscription d’application.
3. Vérifiez que sous **Types de comptes pris en charge** , l’option **Comptes dans cet annuaire organisationnel uniquement** est sélectionnée. Il s’agit de l’option la plus sûre.
4. Nous allons ajouter un URI de redirection ultérieurement, lorsque nous saurons l’URI du cluster. Cliquez sur le bouton **S’inscrire** permettant de créer l’inscription de l’application Azure AD.
5. Dans la page qui s’affiche, notez l’ **ID d’application (client)** . Nous nous référerons à cette valeur en tant que `APPID` dans le didacticiel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

![Capture d’écran de la page Objet de l’application](./media/howto-create-tenant/get-app-id.png)

### <a name="create-a-client-secret"></a>Créer une clé secrète client

Générez une clé secrète client pour authentifier votre application sur Azure Active Directory.

1. Dans la section **Gérer** de la page Inscriptions d’application, cliquez sur **Certificats et clés secrètes** .
2. Dans le volet **Certificats et clés secrètes** , cliquez sur **+Créer une clé secrète client** .  La page **Ajouter une clé secrète client** s’affiche.
3. Entrez une **Description** .
4. Définissez le paramètre **Expires** sur la durée de votre choix, par exemple **Dans 2 ans** .
5. Cliquez sur **Ajouter** et la valeur de la clé s’affiche dans la section **Clés secrètes client** de la page.
6. Copiez la valeur de la clé. Nous nous référerons à cette valeur en tant que `SECRET` dans le didacticiel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

![Capture d’écran du volet Certificats et clés secrètes](./media/howto-create-tenant/create-key.png)

Pour plus d’informations sur les objets Azure Application, veuillez consulter [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Pour plus d’informations sur la création d’une application Azure AD, consultez [Inscrire une application auprès du point de terminaison Azure Active Directory v1.0](../active-directory/develop/quickstart-register-app.md).

## <a name="add-api-permissions"></a>Ajouter des autorisations d’API

[//]: # (Ne remplacez pas par Microsoft Graph. Cela ne fonctionne pas avec Microsoft Graph.)
1. Dans la section **Gérer** , cliquez sur **Autorisations des API** .
2. Cliquez sur **Ajouter une autorisation** , sélectionnez **Azure Active Directory Graph** , puis **Autorisations déléguées** .
> [!NOTE]
> Veillez à sélectionner la vignette « Azure Active Directory Graph » et non « Microsoft Graph ».

3. Développez **Utilisateur** dans la liste ci-dessous et activez l’autorisation **User.Read** . Si l’autorisation **User.Read** est activée par défaut, vérifiez qu’il s’agit de l’autorisation **User.Read** **Azure Active Directory Graph** .
4. Faites défiler la fenêtre vers le haut, puis sélectionnez **Autorisations de l’application** .
5. Développez **Directory** (Répertoire) dans la liste ci-dessous et activez **Directory.ReadAll** .
6. Cliquez sur **Ajouter des autorisations** pour accepter les modifications.
7. Le panneau Autorisations des API doit désormais afficher *User.Read* et *Directory.ReadAll* . Veuillez noter l’avertissement dans la colonne **consentement administrateur requis** en regard de *Directory.ReadAll* .
8. Si vous êtes l’ *administrateur d’abonnement Azure* , cliquez sur **Accorder le consentement administrateur pour *Nom de l’abonnement*** ci-dessous. Si vous n’êtes pas l’ *administrateur d’abonnement Azure* , veuillez demander le consentement à votre administrateur.

![Capture d’écran du panneau Autorisations des API. Les autorisations User.Read et Directory.ReadAll sont ajoutées, le consentement de l’administrateur est requis pour Directory.ReadAll](./media/howto-aad-app-configuration/permissions-required.png)

> [!IMPORTANT]
> La synchronisation du groupe d’administrateurs de cluster fonctionne uniquement lorsque l’autorisation a pas été accordée. Vous verrez un cercle vert avec une coche et un message « Autorisations accordées pour *Nom de l’abonnement*  » dans la colonne *Consentement de l’administrateur requis* .

Pour plus d’informations sur la gestion des administrateurs et d’autres rôles, consultez [Ajout ou modification des administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="resources"></a>Ressources

* [Objets d’applications et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md)
* [Démarrage rapide : Inscrire une application auprès du point de terminaison Azure Active Directory v1.0](../active-directory/develop/quickstart-register-app.md)

## <a name="next-steps"></a>Étapes suivantes

Si vous avez rempli toutes les [conditions préalables à l’utilisation d’Azure Red Hat OpenShift](howto-setup-environment.md), vous pouvez créer votre premier cluster !

Essayez le tutoriel :
> [!div class="nextstepaction"]
> [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md)