---
title: Cadence des mises à jour correctives du système d’exploitation et des runtimes
description: Découvrez comment Azure App Service met à jour le système d’exploitation et les runtimes, les runtimes et le niveau de correctif dont disposent vos applications, et comment obtenir les annonces de mise à jour.
ms.topic: article
ms.date: 02/02/2018
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 8b52223aea0f0bdfecf58906ac192e893da3b47d
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558485"
---
# <a name="os-and-runtime-patching-in-azure-app-service"></a>Système d’exploitation et mise à jour corrective du runtime dans Azure App Service

Cet article vous explique comment obtenir certaines informations de version concernant le système d’exploitation ou les logiciels dans [App Service](overview.md). 

App Service est une plateforme en tant que service (PaaS), ce qui signifie que le système d’exploitation et la pile d’applications sont gérés automatiquement par Azure ; vous ne gérez que votre application et ses données. Vous pouvez obtenir plus de contrôle sur le système d’exploitation et la pile d’applications dans [Machines virtuelles Azure](../virtual-machines/index.yml). Dans cette optique, il est néanmoins utile pour vous, en tant qu’utilisateur App Service, d’obtenir plus d’informations, telles que :

-   Comment et quand sont appliquées les mises à jour du système d’exploitation ?
-   Comment est appliqué un correctif à App Service pour le protéger contre les vulnérabilités importantes (notamment le risque zero-day) ?
-   Quelles versions du système d’exploitation et du runtime exécutent vos applications ?

Pour des raisons de sécurité, certains détails sur les informations de sécurité ne sont pas publiés. Toutefois, l’article vise à réduire les problèmes en optimisant la transparence du processus, et vous explique comment rester à jour grâce à des annonces relatives à la sécurité ou des mises à jour du runtime.

## <a name="how-and-when-are-os-updates-applied"></a>Comment et quand sont appliquées les mises à jour du système d’exploitation ?

Azure gère l’application de correctifs au système d’exploitation à deux niveaux : les serveurs physiques et les machines virtuelles invitées qui exécutent les ressources App Service. Ces deux niveaux sont mis à jour tous les mois, conformément à la planification mensuelle [Patch Tuesday](/security-updates/). Ces mises à jour sont appliquées automatiquement, d’une façon qui garantit le contrat de niveau de service (SLA) Azure. 

Pour plus d’informations sur la façon dont les mises à jour sont appliquées, consultez [*****Demystifying the magic behind App Service OS updates****](https://azure.github.io/AppService/2018/01/18/Demystifying-the-magic-behind-App-Service-OS-updates.html).

## <a name="how-does-azure-deal-with-significant-vulnerabilities"></a>Comment Azure traite-t-il les vulnérabilités importantes ?

Lorsque de graves vulnérabilités nécessitent une mise à jour corrective immédiate, par exemple des [vulnérabilités zero-day](https://wikipedia.org/wiki/Zero-day_(computing)), les mises à jour à priorité élevée sont gérées au cas par cas.

Restez informé grâce à des annonces de sécurité critique dans Azure en consultant le [blog sur la sécurité Azure](https://azure.microsoft.com/blog/topics/security/). 

## <a name="when-are-supported-language-runtimes-updated-added-or-deprecated"></a>Quand les runtimes de langage pris en charge sont-ils mis à jour, ajoutés ou déconseillés ?

De nouvelles versions stables des runtimes de langage pris en charge (majeures, mineures ou correctifs) sont régulièrement ajoutées aux instances App Service. Certaines mises à jour remplacent l’installation existante, tandis que d’autres sont installées côte à côte avec les versions existantes. Une installation de remplacement implique que votre application s’exécute automatiquement sur le runtime mis à jour. Une installation côte à côte signifie que vous devez migrer manuellement votre application pour tirer parti d’une nouvelle version du runtime. Pour plus d’informations, consultez l’une des sous-sections.

Les mises à jour et les désapprobations de runtime sont annoncées ici :

- https://azure.microsoft.com/updates/?product=app-service 
- https://github.com/Azure/app-service-announcements/issues

> [!NOTE] 
> Ces informations s’appliquent aux runtimes de langage qui sont intégrés à une application App Service. Par exemple, un runtime personnalisé que vous chargez sur App Service reste inchangé, sauf si vous le mettez manuellement à niveau.
>
>

### <a name="new-patch-updates"></a>Nouvelles mises à jour correctives

Les mises à jour correctives de la version .NET, PHP, Java SDK ou Tomcat sont appliquées automatiquement en remplaçant l’installation existante par la dernière version. Les mises à jour correctives Node.js sont installées côte à côte avec les versions existantes (comme pour les versions majeures et mineures à la section suivante). Les nouvelles versions du correctif Python peuvent être installées manuellement par le biais des [extensions de site](https://azure.microsoft.com/blog/azure-web-sites-extensions/), côte à côte avec les installations Python intégrées.

### <a name="new-major-and-minor-versions"></a>Nouvelles versions majeures et mineures

Lorsqu’une nouvelle version majeure ou mineure est ajoutée, elle est installée côte à côte avec les versions existantes. Vous pouvez manuellement mettre à niveau votre application avec la nouvelle version. Si vous avez configuré la version du runtime dans un fichier de configuration (tel que `web.config` et `package.json`), vous devez effectuer la mise à niveau avec la même méthode. Si vous avez utilisé un paramètre App Service pour configurer votre version du runtime, vous pouvez le modifier dans le [portail Azure](https://portal.azure.com) ou en exécutant une commande d[’interface CLI Azure](/cli/azure/get-started-with-azure-cli) dans le [Cloud Shell](../cloud-shell/overview.md), comme indiqué dans les exemples suivants :

```azurecli-interactive
az webapp config set --net-framework-version v4.7 --resource-group <groupname> --name <appname>
az webapp config set --php-version 7.0 --resource-group <groupname> --name <appname>
az webapp config appsettings set --settings WEBSITE_NODE_DEFAULT_VERSION=8.9.3 --resource-group <groupname> --name <appname>
az webapp config set --python-version 3.4 --resource-group <groupname> --name <appname>
az webapp config set --java-version 1.8 --java-container Tomcat --java-container-version 9.0 --resource-group <groupname> --name <appname>
```

### <a name="deprecated-versions"></a>Versions déconseillées  

Lorsqu’une version plus ancienne est déconseillée, la date de suppression est annoncée, ce qui vous permet de planifier en conséquence la mise à niveau de votre version du runtime. 

## <a name="how-can-i-query-os-and-runtime-update-status-on-my-instances"></a>Comment afficher l’état de mise à jour du système d’exploitation et du runtime sur mes instances ?  

Bien que les informations critiques sur le système d’exploitation ne soient pas accessibles (voir [Fonctionnalités de système d’exploitation sur Azure App Service](operating-system-functionality.md)), la [console Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) vous permet d’interroger votre instance App Service pour connaître la version du système d’exploitation et les versions du runtime. 

Le tableau suivant indique comment afficher les versions de Windows et du runtime du langage qui exécutent vos applications :

| Information | Comment y accéder | 
|-|-|
| Version de Windows | Voir `https://<appname>.scm.azurewebsites.net/Env.cshtml` (sous Informations système) |
| Version de .NET | Dans `https://<appname>.scm.azurewebsites.net/DebugConsole`, exécutez la commande suivante dans l’invite de commandes : <br>`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full"` |
| Version .NET Core | Dans `https://<appname>.scm.azurewebsites.net/DebugConsole`, exécutez la commande suivante dans l’invite de commandes : <br> `dotnet --version` |
| Version PHP | Dans `https://<appname>.scm.azurewebsites.net/DebugConsole`, exécutez la commande suivante dans l’invite de commandes : <br> `php --version` |
| Version Node.js par défaut | Dans le [Cloud Shell](../cloud-shell/overview.md), exécutez la commande suivante : <br> `az webapp config appsettings list --resource-group <groupname> --name <appname> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION']"` |
| Version Python | Dans `https://<appname>.scm.azurewebsites.net/DebugConsole`, exécutez la commande suivante dans l’invite de commandes : <br> `python --version` |  
| Version de Java | Dans `https://<appname>.scm.azurewebsites.net/DebugConsole`, exécutez la commande suivante dans l’invite de commandes : <br> `java -version` |  

> [!NOTE]  
> L’accès à l’emplacement de registre `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages`, où sont stockées les informations sur les [correctifs KB](/security-updates/SecurityBulletins/securitybulletins), est verrouillé.
>
>

## <a name="more-resources"></a>Plus de ressources

[Centre de confidentialité - Sécurité](https://www.microsoft.com/en-us/trustcenter/security)  
[ASP.NET Core 64 bits sur Azure App Service](https://gist.github.com/glennc/e705cd85c9680d6a8f1bdb62099c7ac7)
