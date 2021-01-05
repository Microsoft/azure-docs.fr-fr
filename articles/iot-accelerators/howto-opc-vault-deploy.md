---
title: Guide pratique pour déployer le service de gestion de certificats OPC Vault - Azure | Microsoft Docs
description: Guide pratique pour déployer le service de gestion de certificats OPC Vault à partir de zéro.
author: mregen
ms.author: mregen
ms.date: 08/16/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 98dc9345d2c8b392fd094458b612857d6d454739
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071487"
---
# <a name="build-and-deploy-the-opc-vault-certificate-management-service"></a>Générer et déployer le service de gestion de certificats OPC Vault

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article explique comment déployer le service de gestion de certificats OPC Vault dans Azure.

> [!NOTE]
> Pour plus d’informations, consultez le [dépôt OPC Vault](https://github.com/Azure/azure-iiot-opc-vault-service) sur GitHub.

## <a name="prerequisites"></a>Prérequis

### <a name="install-required-software"></a>Installer les logiciels nécessaires

Actuellement, l’opération de génération et de déploiement est limitée à Windows.
Les exemples sont tous écrits pour C# .NET Standard, ce dont vous avez besoin pour générer le service et des exemples pour le déploiement.
Tous les outils dont vous avez besoin pour .NET Standard sont fournis avec les outils .NET Core. Consultez [Bien démarrer avec .NET Core](/dotnet/articles/core/getting-started).

1. [Installez .NET Core 2.1+][dotnet-install].
2. [Installez Docker][docker-url] (facultatif, uniquement si la build Docker locale est nécessaire).
4. Installez les [outils en ligne de commande Azure pour PowerShell][powershell-install].
5. Souscrivez à un [abonnement Azure][azure-free].

### <a name="clone-the-repository"></a>Cloner le référentiel

Si vous ne l’avez pas déjà fait, clonez ce dépôt GitHub. Ouvrez une invite de commandes ou un terminal, puis exécutez ce qui suit :

```bash
git clone https://github.com/Azure/azure-iiot-opc-vault-service
cd azure-iiot-opc-vault-service 
```

Vous pouvez aussi cloner le dépôt directement dans Visual Studio 2017.

### <a name="build-and-deploy-the-azure-service-on-windows"></a>Générer et déployer le service Azure sur Windows

Un script PowerShell permet de déployer facilement le microservice OPC Vault et l’application.

1. Ouvrez une fenêtre PowerShell à la racine du dépôt. 
3. Accédez au dossier de déploiement `cd deploy`.
3. Choisissez un nom pour `myResourceGroup` qui ne risque pas de provoquer un conflit avec d’autres pages web déployées. Consultez la section « Nom du site web déjà utilisé » plus loin dans cet article.
5. Lancez le déploiement avec `.\deploy.ps1` pour une installation interactive ou entrez une ligne de commande complète :  
`.\deploy.ps1  -subscriptionName "MySubscriptionName" -resourceGroupLocation "East US" -tenantId "myTenantId" -resourceGroupName "myResourceGroup"`
7. Si vous envisagez de développer avec ce déploiement, ajoutez `-development 1` pour activer l’interface utilisateur Swagger et déployer des builds de débogage.
6. Suivez les instructions contenues dans le script pour vous connecter à votre abonnement et fournir des informations supplémentaires.
9. Après une opération de génération et de déploiement réussie, le message suivant doit s’afficher :
   ```
   To access the web client go to:
   https://myResourceGroup.azurewebsites.net

   To access the web service go to:
   https://myResourceGroup-service.azurewebsites.net

   To start the local docker GDS server:
   .\myResourceGroup-dockergds.cmd

   To start the local dotnet GDS server:
   .\myResourceGroup-gds.cmd
   ```

   > [!NOTE]
   > En cas de problèmes, consultez la section « Résoudre les problèmes de déploiement » plus loin dans cet article.

8. Ouvrez le navigateur de votre choix, puis ouvrez la page de l’application : `https://myResourceGroup.azurewebsites.net`
8. Accordez à l’application web et au microservice OPC Vault quelques minutes de préparation après le déploiement. Il se peut que la page d’accueil web se fige pendant une minute tout au plus, avant que les premières réponses ne vous parviennent.
11. Pour examiner l’API Swagger, ouvrez : `https://myResourceGroup-service.azurewebsites.net`
13. Pour démarrer un serveur GDS local avec dotnet, démarrez `.\myResourceGroup-gds.cmd`. Avec Docker, démarrez `.\myResourceGroup-dockergds.cmd`.

Il est possible de redéployer une build avec exactement les mêmes paramètres. N’oubliez pas qu’une telle opération renouvelle tous les secrets d’application et peut réinitialiser certains paramètres dans les inscriptions d’applications Azure Active Directory (Azure AD).

Il est possible aussi de redéployer uniquement les fichiers binaires de l’application web. Avec le paramètre `-onlyBuild 1`, les nouveaux packages zip du service et l’application sont déployés dans les applications web.

Une fois le déploiement effectué, vous pouvez commencer à utiliser les services. Consultez [Gérer le service de gestion de certificats OPC Vault](howto-opc-vault-manage.md).

## <a name="delete-the-services-from-the-subscription"></a>Supprimer les services de l’abonnement

Voici comment faire :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez au groupe de ressources dans lequel le service a été déployé.
3. Sélectionnez **Supprimer le groupe de ressources** et confirmez.
4. Après quelques instants, tous les composants du service déployés sont supprimés.
5. Accédez à **Azure Active Directory** > **Inscriptions des applications**.
6. Trois inscriptions doivent être listées pour chaque groupe de ressources déployé. Les inscriptions portent les noms suivants : `resourcegroup-client`, `resourcegroup-module`, `resourcegroup-service`. Supprimez chaque inscription séparément.

Maintenant, tous les composants déployés sont supprimés.

## <a name="troubleshooting-deployment-failures"></a>Résoudre les problèmes de déploiement

### <a name="resource-group-name"></a>Nom de groupe ressources

Utilisez un nom de groupe de ressources court et simple. Le nom sert aussi à nommer les ressources et le préfixe de l’URL du service. Par conséquent, il doit être conforme aux exigences de nommage des ressources.  

### <a name="website-name-already-in-use"></a>Nom du site web déjà utilisé

Le nom du site web est peut déjà être utilisé. Vous devez dans ce cas utiliser un autre nom de groupe de ressources. Les noms d’hôtes utilisés par le script de déploiement sont les suivants : https:\//resourcegroupname.azurewebsites.net et https:\//resourgroupname-service.azurewebsites.net.
Les autres noms de services résultent d’une combinaison de hachages de noms courts et ne risquent pas d’entrer en conflit avec d’autres services.

### <a name="azure-ad-registration"></a>inscription Azure AD. 

Le script de déploiement essaie d’inscrire trois applications Azure AD dans Azure AD. En fonction de vos autorisations dans le locataire Azure AD sélectionné, cette opération peut échouer. Nous avons deux options :

- Si vous avez choisi un locataire Azure AD dans une liste de locataires, redémarrez le script et choisissez un autre locataire dans la liste.
- Vous pouvez aussi déployer un locataire Azure AD privé d’un autre abonnement. Redémarrez le script, puis sélectionnez-le pour l’utiliser.

## <a name="deployment-script-options"></a>Options du script de déploiement

Le script utilise les paramètres suivants :


```
-resourceGroupName
```

Il peut s’agir du nom d’un groupe de ressources existant ou nouveau.

```
-subscriptionId
```


Il s’agit de l’ID de l’abonnement où seront déployées les ressources. Il est facultatif.

```
-subscriptionName
```


Vous pouvez aussi utiliser le nom de l’abonnement.

```
-resourceGroupLocation
```


Il s’agit d’un emplacement de groupe de ressources. S’il est spécifié, ce paramètre tente de créer un groupe de ressources à cet emplacement. Ce paramètre est également facultatif.


```
-tenantId
```


Il s’agit du locataire Azure AD à utiliser. 

```
-development 0|1
```

Ce paramètre est utilisé pour déployer à des fins de développement. Utilisez la build de débogage et définissez l’environnement ASP.Net sur Développement. Créez `.publishsettings` pour l’importation dans Visual Studio 2017 et pour permettre un déploiement direct de l’application et du service. Ce paramètre est également facultatif.

```
-onlyBuild 0|1
```

Ce paramètre vise à regénérer et redéployer uniquement les applications web et à regénérer les conteneurs Docker. Ce paramètre est également facultatif.

[azure-free]:https://azure.microsoft.com/free/
[powershell-install]:https://azure.microsoft.com/downloads/#powershell
[docker-url]: https://www.docker.com/
[dotnet-install]: https://www.microsoft.com/net/learn/get-started

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment déployer OPC Vault en partant de rien, vous pouvez :

> [!div class="nextstepaction"]
> [Gérer OPC Vault](howto-opc-vault-manage.md)