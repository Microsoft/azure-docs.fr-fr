---
title: configurer les informations d’identification de déploiement
description: Découvrez les types d’informations d’identification de déploiement dans Azure App Service et comment les configurer et les utiliser.
ms.topic: article
ms.date: 08/14/2019
ms.reviewer: byvinyal
ms.custom: seodec18
ms.openlocfilehash: e5793d21f27128162095e2d86e13006c5b6e7b7c
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007991"
---
# <a name="configure-deployment-credentials-for-azure-app-service"></a>Configurer les informations d’identification de déploiement pour Azure App Service
[Azure App Service](./overview.md) prend en charge deux types d’informations d’identification pour le [déploiement Git local](deploy-local-git.md) et le [déploiement FTP/S](deploy-ftp.md). Ces informations d’identification ne sont pas les mêmes que les informations d’identification de votre abonnement Azure.

[!INCLUDE [app-service-deploy-credentials](../../includes/app-service-deploy-credentials.md)]

## <a name="configure-user-level-credentials"></a><a name="userscope"></a>Configurer des informations d’identification au niveau de l’utilisateur

Vous pouvez configurer vos informations d’identification au niveau de l’utilisateur dans la [page Ressources](../azure-resource-manager/management/manage-resources-portal.md#manage-resources) d’une application. Quelle que soit l’application dans laquelle vous configurez ces informations d’identification, ces dernières s’appliquent à toutes les applications et à tous les abonnements de votre compte Azure. 

### <a name="in-the-cloud-shell"></a>Dans Cloud Shell

Pour configurer l’utilisateur de déploiement dans [Cloud Shell](https://shell.azure.com), exécutez la commande [az webapp deployment user set](/cli/azure/webapp/deployment/user#az-webapp-deployment-user-set). Remplacez \<username> et \<password> par un nom d’utilisateur et un mot de passe de déploiement. 

- Le nom d’utilisateur doit être unique dans Azure et, pour les push Git locaux, ne doit pas contenir le symbole « @ ». 
- Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles. 

```azurecli-interactive
az webapp deployment user set --user-name <username> --password <password>
```

La sortie JSON affiche le mot de passe comme étant `null`. Si vous obtenez une erreur `'Conflict'. Details: 409`, modifiez le nom d’utilisateur. Si vous obtenez une erreur `'Bad Request'. Details: 400`, utilisez un mot de passe plus fort. 

### <a name="in-the-portal"></a>Sur le portail

Sur le portail Azure, vous devez disposer d’au moins une application avant de pouvoir accéder à la page Informations d’identification de déploiement. Pour configurer les informations d’identification au niveau de l’utilisateur :

1. Sur le [portail Azure](https://portal.azure.com), dans le menu de gauche, sélectionnez **App Services** >  **\<any_app>**  > **Centre de déploiement** > **FTP** > **Tableau de bord**.

    ![Montre comment vous pouvez sélectionner le tableau de bord FTP à partir du Centre de déploiement dans Azure App Services.](./media/app-service-deployment-credentials/access-no-git.png)

    Ou bien, si vous avez déjà configuré le déploiement Git, sélectionnez **App Services** >  **&lt;une_application>**  > **Centre de déploiement** > **FTP/Informations d’identification**.

    ![Montre comment vous pouvez sélectionner le tableau de bord FTP à partir du Centre de déploiement dans Azure App Services pour votre déploiement de Git configuré.](./media/app-service-deployment-credentials/access-with-git.png)

2. Sélectionnez **Informations d’identification de l’utilisateur**, configurez le nom d’utilisateur et le mot de passe, puis sélectionnez **Enregistrer les informations d’identification**.

Une fois que vous avez défini vos informations d’identification de déploiement, vous trouverez le nom d’utilisateur du déploiement *Git* dans la page **Vue d’ensemble** de votre application.

![Montre comment trouver le nom d’utilisateur du déploiement Git sur la page Vue d’ensemble de votre application.](./media/app-service-deployment-credentials/deployment_credentials_overview.png)

Si le déploiement Git est configuré, la page présente un **Nom d’utilisateur Git/Déploiement** ; sinon, un **Nom d’utilisateur FTP/Déploiement** est affiché.

> [!NOTE]
> Azure n’affiche pas votre mot de passe du déploiement au niveau de l’utilisateur. Si vous oubliez le mot de passe, vous pouvez réinitialiser vos informations d’identification en suivant les étapes décrites dans cette section.
>
> 

## <a name="use-user-level-credentials-with-ftpftps"></a>Utiliser les informations d’identification de niveau utilisateur avec FTP/FTPS

L’authentification auprès d’un point de terminaison FTP/FTPS à l’aide d’informations d’identification de niveau utilisateur nécessite un nom d’utilisateur au format suivant : `<app-name>\<user-name>`

Dans la mesure où les informations d’identification au niveau utilisateur sont liées à l’utilisateur et non à une ressource spécifique, le nom d’utilisateur doit respecter ce format pour que l’action de connexion soit dirigée vers le point de terminaison d’application approprié.

## <a name="get-and-reset-app-level-credentials"></a><a name="appscope"></a>Obtenir et réinitialiser les informations d’identification au niveau de l’application
Pour obtenir les informations d’identification au niveau de l’application :

1. Sur le [portail Azure](https://portal.azure.com), dans le menu de gauche, sélectionnez **App Services** >  **&lt;une_application>**  > **Centre de déploiement** > **FTP/Informations d’identification**.

2. Sélectionnez **Informations d’identification de l’application**, puis cliquez sur le lien **Copier** pour copier le nom d’utilisateur ou le mot de passe.

Pour réinitialiser les informations d’identification de niveau application, sélectionnez **Réinitialiser les informations d’identification** dans la même boîte de dialogue.

## <a name="disable-basic-authentication"></a>Désactiver l’authentification de base

Certaines organisations doivent répondre aux exigences de sécurité et préfèrent désactiver l’accès via FTP ou WebDeploy. De cette manière, les membres de l’organisation peuvent accéder aux services d’application uniquement par le biais d’API contrôlées par Azure Active Directory (Azure AD).

### <a name="ftp"></a>FTP

Pour désactiver l’accès FTP au site, exécutez la commande CLI suivante. Remplacez les espaces réservés par les noms de votre groupe de ressources et de votre site. 

```bash
az resource update --resource-group <resource-group> --name ftp --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Pour confirmer que l’accès FTP est bloqué, vous pouvez essayer de vous authentifier à l’aide d’un client FTP tel que FileZilla. Pour récupérer les informations d’identification de publication, accédez au panneau Vue d’ensemble de votre site et cliquez sur Télécharger le profil de publication. Essayez de vous authentifier en utilisant le nom d’hôte FTP du fichier, le nom d’utilisateur et le mot de passe. Vous obtenez une réponse d’erreur 401, indiquant que vous n’êtes pas autorisé.

### <a name="webdeploy-and-scm"></a>WebDeploy et SCM

Pour désactiver l’accès d’authentification de base au port WebDeploy et au site SCM, exécutez la commande CLI suivante. Remplacez les espaces réservés par les noms de votre groupe de ressources et de votre site. 

```bash
az resource update --resource-group <resource-group> --name scm --namespace Microsoft.Web --resource-type basicPublishingCredentialsPolicies --parent sites/<site-name> --set properties.allow=false
```

Pour confirmer que les informations d’identification du profil de publication sont bloquées sur WebDeploy, essayez de [publier une application web à l’aide de Visual Studio 2019](/visualstudio/deployment/quickstart-deploy-to-azure).

### <a name="disable-access-to-the-api"></a>Désactiver l’accès à l’API

L’API de la section précédente est associée au contrôle d’accès en fonction du rôle Azure (Azure RBAC), ce qui signifie que vous pouvez [créer un rôle personnalisé](../role-based-access-control/custom-roles.md#steps-to-create-a-custom-role) et affecter des utilisateurs moins privilégiés au rôle afin qu’ils ne puissent pas activer l’authentification de base sur des sites. Pour configurer le rôle personnalisé, [suivez ces instructions](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#create-a-custom-rbac-role).

Vous pouvez également utiliser [Azure Monitor](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#audit-with-azure-monitor) pour auditer toutes les demandes d’authentification réussies et utiliser [Azure Policy](https://azure.github.io/AppService/2020/08/10/securing-data-plane-access.html#enforce-compliance-with-azure-policy) pour appliquer cette configuration à tous les sites de votre abonnement.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser ces informations d’identification pour déployer votre application à partir de [Git local](deploy-local-git.md) ou à l’aide de [FTP/S](deploy-ftp.md).
