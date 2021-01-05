---
author: zhangmanling
ms.service: azure-cdn
ms.topic: include
ms.date: 11/21/2018
ms.author: mazha
ms.openlocfilehash: 4967991b0edaa854acbf6b308596859d662311fe
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993359"
---
## <a name="prerequisites"></a>Prérequis
Avant d’écrire le code de gestion CDN, vous devez effectuer certaines tâches de préparation pour permettre à ce code d’interagir avec Azure Resource Manager. Ce travail de préparation implique les opérations suivantes :

* Créez un groupe de ressources destiné à héberger le profil CDN créé dans ce didacticiel.
* Configurez Azure Active Directory pour l’authentification de l’application.
* Octroyez des autorisations au groupe de ressources afin que seuls les utilisateurs autorisés de votre locataire Azure AD puissent interagir avec le profil CDN.

### <a name="creating-the-resource-group"></a>Création du groupe de ressources
1. Connectez-vous au [Portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource**.
3. Recherchez **Groupe de ressources**, puis dans le volet Groupe de ressources, cliquez sur **Créer**.

    ![Création d’un groupe de ressources](./media/cdn-app-dev-prep/cdn-new-rg-1-include.png)
3. Attribuez à votre groupe de ressources le nom *CdnConsoleTutorial*.  Sélectionnez votre abonnement et choisissez un emplacement près de chez vous.  Si vous le souhaitez, vous pouvez cocher la case **Épingler au tableau de bord** pour épingler le groupe de ressources au tableau de bord dans le portail.  Une fois épinglé, le groupe sera plus facile à repérer.  Une fois vos sélections effectuées, cliquez sur **Créer**.

    ![Capture d’écran de la boîte de dialogue Groupe de ressources.](./media/cdn-app-dev-prep/cdn-new-rg-2-include.png)
4. Si vous n’avez pas épinglé le groupe de ressources à votre tableau de bord, cliquez sur **Parcourir**, puis sur **Groupes de ressources** pour le rechercher.  Pour ouvrir le groupe de ressources, cliquez sur ce dernier.  Notez votre **ID d’abonnement**. Nous en aurons besoin ultérieurement.

    ![Capture d’écran de la section Tutoriel de la console CDN.](./media/cdn-app-dev-prep/cdn-subscription-id-include.png)

### <a name="creating-the-azure-ad-application-and-applying-permissions"></a>Création d’une application Azure AD et application des autorisations
Il existe deux approches pour authentifier une application avec Azure Active Directory : les utilisateurs individuels ou un principal de service. Un principal de service est similaire à un compte de service dans Windows.  Au lieu d’être octroyées à un utilisateur spécifique, les autorisations d’interagir avec les profils CDN sont accordées au principal de service.  Les principaux de service sont généralement utilisés pour des processus automatisés non interactifs.  Bien que le but de ce didacticiel soit de créer une application console interactive, nous allons nous adopter l’approche du principal de service.

La création d’un principal de service comprend plusieurs étapes, dont la création d’une application Azure Active Directory.  Pour créer cette application, nous allons [suivre ce didacticiel](../articles/active-directory/develop/howto-create-service-principal-portal.md).

> [!IMPORTANT]
> Veillez à respecter toutes les étapes du [didacticiel associé](../articles/active-directory/develop/howto-create-service-principal-portal.md).  Il est *important* que vous suiviez la procédure à la lettre.  N’oubliez pas de noter votre **ID de locataire**, **nom de domaine de locataire** (généralement un domaine *.onmicrosoft.com*, sauf si vous avez spécifié un domaine personnalisé), **ID de client** et **clé d’authentification**, car nous aurons besoin de ces informations ultérieurement.  Veillez à protéger votre **ID de client** et votre **clé d’authentification**, car toute personne peut utiliser ces informations d’identification pour exécuter des opérations en tant que principal de service.
>
> Lorsque vous arrivez à l’étape intitulée Configurer une application mutualisée, sélectionnez **Non**.
>
> Lorsque vous arrivez à l’étape [Affecter l’application à un rôle](../articles/active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application), utilisez le groupe de ressources créé précédemment, *CdnConsoleTutorial*, mais à la place du rôle **Lecteur**, affectez-lui le rôle **Contributeur de profil CDN**.  Après avoir affecté à l’application le rôle **Contributeur du profil CDN** dans votre groupe de ressources, revenez dans ce didacticiel. 
>
>

Une fois que vous avez créé le principal de service et affecté le rôle **Contributeur de profil CDN**, le panneau **Utilisateurs** de votre groupe de ressources doit se présenter comme illustré ci-après.

![Panneau Utilisateurs](./media/cdn-app-dev-prep/cdn-service-principal-include.png)

### <a name="interactive-user-authentication"></a>Authentification interactive des utilisateurs
Si, au lieu d’un principal de service, vous préférez disposer d’une authentification des utilisateurs interactive, ce processus est comparable à celui d’un principal de service.  En fait, la procédure est identique, à quelques légères différences près.

> [!IMPORTANT]
> Ne suivez ces étapes que si vous optez pour l’authentification des utilisateurs au lieu d’un principal de service.
>
>

1. Lors de la création de votre application, au lieu de choisir **Application web**, sélectionnez **Application native**.

    ![Application native](./media/cdn-app-dev-prep/cdn-native-application-include.png)
2. Sur la page suivante, vous êtes invité à spécifier un **URI de redirection**.  Cette URI ne sera pas validée mais notez-la. Vous en aurez besoin ultérieurement.
3. Il est inutile de créer une **clé d’authentification de client**.
4. Au lieu d’affecter le rôle **Contributeur du profil CDN** à un principal de service, nous allons l’attribuer à des utilisateurs ou des groupes.  Dans cet exemple, vous pouvez voir que j’ai attribué le rôle **Contributeur du profil CDN** à l’utilisateur *CDN Demo User*.  

    ![Accès d’utilisateurs individuels](./media/cdn-app-dev-prep/cdn-aad-user-include.png)
