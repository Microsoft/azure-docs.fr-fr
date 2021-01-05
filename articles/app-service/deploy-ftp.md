---
title: Déployer du contenu avec FTP/S
description: Découvrez comment déployer votre application dans Azure App Service avec FTP ou FTPS. Améliorez la sécurité d’un site web en désactivant le protocole FTP non chiffré.
ms.assetid: ae78b410-1bc0-4d72-8fc4-ac69801247ae
ms.topic: article
ms.date: 09/18/2019
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: 9884b109db3f3a34ceb323bef9fba1d5bfc23147
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150264"
---
# <a name="deploy-your-app-to-azure-app-service-using-ftps"></a>Déployer votre application dans Azure App Service avec FTP/S

Cet article vous explique comment utiliser FTP ou FTPS pour déployer votre application web, votre backend d’applications mobiles ou votre application API dans [Azure App Service](./overview.md).

Le point de terminaison FTP/S de votre application est déjà actif. Aucune configuration n’est nécessaire pour activer le déploiement FTP/S.

## <a name="open-ftp-dashboard"></a>Ouvrir le tableau de bord FTP

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**.

    ![Recherchez App Services.](media/app-service-continuous-deployment/search-for-app-services.png)

2. Sélectionnez l’application web à déployer.

    ![Sélectionnez votre application.](media/app-service-continuous-deployment/select-your-app.png)

3. Sélectionnez **Centre de déploiement** > **FTP** > **Tableau de bord**.

    ![Ouvrir le tableau de bord FTP](./media/app-service-deploy-ftp/open-dashboard.png)

## <a name="get-ftp-connection-information"></a>Obtention des informations de connexion FTP

Dans le tableau de bord FTP, sélectionnez **Copier** pour copier le point de terminaison FTPS et les informations d’identification de l’application.

![Copier les informations FTP](./media/app-service-deploy-ftp/ftp-dashboard.png)

Il est recommandé d’utiliser les **informations d’identification de l’application** pour le déploiement sur votre application, car elles sont propres à chaque application. Toutefois, si vous cliquez sur **Informations d’identification de l’utilisateur**, vous pouvez définir des informations d’identification au niveau de l’utilisateur, que vous pouvez utiliser pour la connexion FTP/S à toutes les applications App Service de votre abonnement.

> [!NOTE]
> L’authentification auprès d’un point de terminaison FTP/FTPS à l’aide d’informations d’identification de niveau utilisateur nécessite un nom d’utilisateur au format suivant : 
>
>`<app-name>\<user-name>`
>
> Dans la mesure où les informations d’identification au niveau utilisateur sont liées à l’utilisateur et non à une ressource spécifique, le nom d’utilisateur doit respecter ce format pour que l’action de connexion soit dirigée vers le point de terminaison d’application approprié.
>

## <a name="deploy-files-to-azure"></a>Déployer des fichiers sur Azure

1. Dans votre client FTP (par exemple, [Visual Studio](https://www.visualstudio.com/vs/community/), [Cyberduck](https://cyberduck.io/) ou [WinSCP](https://winscp.net/index.php)), utilisez les informations de connexion que vous avez recueillies pour vous connecter à votre application.
2. Copiez vos fichiers et la structure de répertoire qui leur correspond dans le répertoire [ **/site/wwwroot** dans Azure](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure) (ou dans le répertoire **/site/wwwroot/App_Data/Jobs/** pour WebJobs).
3. Accédez à l’URL de votre application pour vérifier que l’application s’exécute correctement. 

> [!NOTE] 
> Contrairement aux [déploiements Git](deploy-local-git.md), un déploiement FTP ne prend pas en charge les automatisations de déploiement suivantes : 
>
> - restauration de dépendances (par exemple, des automatisations NuGet, NPM, PIP et Composer)
> - compilation de fichiers binaires .NET
> - génération du fichier web.config (dont voici un [exemple Node.js](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps))
> 
> Générez ces fichiers requis manuellement sur votre ordinateur local, puis déployez-les avec votre application.
>

## <a name="enforce-ftps"></a>Appliquer le protocole FTPS

Pour une sécurité renforcée, vous devez autoriser FTP sur TLS/SSL uniquement. Vous pouvez également désactiver FTP et FTPS si vous n’utilisez pas de déploiement FTP.

Dans la page des ressources de votre application dans le [portail Azure](https://portal.azure.com), sélectionnez **Configuration** > **Paramètres généraux** dans le volet de navigation gauche.

Pour désactiver le protocole FTP non chiffré, sélectionnez **FTPS uniquement** dans **État FTP**. Pour désactiver complètement les protocoles FTP et FTPS, sélectionnez **Désactivé**. Lorsque vous avez terminé, cliquez sur **Enregistrer**. Si vous utilisez **FTPS uniquement**, appliquez TLS 1.2 ou version ultérieure en accédant au panneau **Paramètres TLS/SSL** de votre application web. TLS 1.0 et 1.1 ne sont pas pris en charge avec **FTPS uniquement**.

![Désactiver le protocole FTP/S](./media/app-service-deploy-ftp/disable-ftp.png)

## <a name="automate-with-scripts"></a>Automatiser des tâches à l’aide de scripts

Pour le déploiement FTP à l’aide d’[Azure CLI](/cli/azure), consultez [Créer une application web et déployer des fichiers par FTP (Azure CLI)](./scripts/cli-deploy-ftp.md).

Pour le déploiement FTP à l’aide d’[Azure PowerShell](/cli/azure), consultez [Télécharger des fichiers vers une application web via FTP (PowerShell)](./scripts/powershell-deploy-ftp.md).

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-ftp-deployment"></a>Résoudre les problèmes d’un déploiement FTP

- [Comment puis-je résoudre les problèmes d’un déploiement FTP ?](#how-can-i-troubleshoot-ftp-deployment)
- [Je ne peux pas envoyer par FTP et publier mon code. Comment puis-je résoudre le problème ?](#im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue)
- [Comment puis-je me connecter à FTP dans Azure App Service via le mode passif ?](#how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode)

### <a name="how-can-i-troubleshoot-ftp-deployment"></a>Comment puis-je résoudre les problèmes d’un déploiement FTP ?

La première étape de résolution des problèmes d’un déploiement FTP consiste à isoler un problème de déploiement d’un problème d’application à l’exécution.

Un problème de déploiement se produit généralement si aucun fichier ou des fichiers incorrects sont déployés sur votre application. Vous pouvez le résoudre en examinant votre déploiement FTP ou en sélectionnant un autre chemin d’accès de déploiement (comme un contrôle de code source).

Un problème d’application à l’exécution se produit généralement si l’ensemble de fichiers approprié est déployé sur votre application, mais que le comportement de l’application est incorrect. Vous pouvez le résoudre en vous concentrant sur le comportement du code lors de l’exécution et en examinant des chemins d’accès d’échec spécifiques.

Pour déterminer un problème de déploiement ou d’exécution, consultez [Deployment vs. runtime issues](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues) (Problèmes de déploiement et d’exécution).

### <a name="im-not-able-to-ftp-and-publish-my-code-how-can-i-resolve-the-issue"></a>Je ne peux pas envoyer par FTP et publier mon code. Comment puis-je résoudre le problème ?
Vérifiez que vous avez entré le nom d’hôte et les [informations d’identification](#open-ftp-dashboard) corrects. Vérifiez également que les ports FTP suivants sur votre machine ne sont pas bloqués par un pare-feu :

- Port de connexion de contrôle FTP : 21
- Port de connexion de données FTP : 989, 10001-10300
 
### <a name="how-can-i-connect-to-ftp-in-azure-app-service-via-passive-mode"></a>Comment puis-je me connecter à FTP dans Azure App Service via le mode passif ?
Azure App Service prend en charge la connexion via les modes actif et passif. Le mode passif est recommandé, car vos machines de déploiement se trouvent généralement derrière un pare-feu (dans le système d’exploitation ou faisant partie d’un réseau domestique ou d’entreprise). Consultez un [exemple dans la documentation de WinSCP](https://winscp.net/docs/ui_login_connection). 

## <a name="next-steps"></a>Étapes suivantes

Pour des scénarios de déploiement plus avancés, consultez [Déploiement Git local vers Azure App Service](deploy-local-git.md). Le déploiement GIT vers Azure autorise le contrôle de version, la restauration du package, MSBuild et bien plus encore.

## <a name="more-resources"></a>Plus de ressources

* [Informations d’identification du déploiement d’Azure App Service](deploy-configure-credentials.md)