---
title: Fichier include
description: Fichier include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/12/2019
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: add0d392f39ab476c6d75f704d5b2e2e0faaa77c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004340"
---
## <a name="prepare-your-repository"></a>Préparer votre dépôt

Pour obtenir les builds automatiques à partir du serveur de builds Kudu d’Azure App Service, assurez-vous que la racine du référentiel contient les fichiers appropriés de votre projet.

| Runtime | Fichiers du répertoire racine |
|-|-|
| ASP.NET (Windows uniquement) | _*.sln_, _*.csproj_ ou _default.aspx_ |
| ASP.NET Core | _*.sln_ ou _*.csproj_ |
| PHP | _index.php_ |
| Ruby (Linux uniquement) | _Gemfile_ |
| Node.js | _server.js_, _app.js_ ou _package.json_ avec un script de démarrage |
| Python | _\*.py_, _requirements.txt_ ou _runtime.txt_ |
| HTML | _default.htm_, _default.html_, _default.asp_, _index.htm_, _index.html_ ou _iisstart.htm_ |
| WebJobs | _\<job_name>/run.\<extension>_ sous _App\_Data/jobs/continuous_ pour les WebJobs continus ou _App\_Data/jobs/triggered_ pour les WebJobs déclenchés. Pour plus d’informations, consultez la [documentation Kudu relative aux WebJobs](https://github.com/projectkudu/kudu/wiki/WebJobs). |
| Fonctions | Consultez [Déploiement continu pour Azure Functions](../articles/azure-functions/functions-continuous-deployment.md#requirements-for-continuous-deployment). |

Pour personnaliser votre déploiement, vous pouvez inclure un fichier *.deployment* dans la racine du dépôt. Pour plus d’informations, consultez [Personnaliser les déploiements](https://github.com/projectkudu/kudu/wiki/Customizing-deployments) et [Personnaliser un script de déploiement](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script).

> [!NOTE]
> Si vous développez dans Visual Studio, laissez [Visual Studio créer un dépôt pour vous](/azure/devops/repos/git/creatingrepo?view=vsts&tabs=visual-studio). Le projet est immédiatement prêt à être déployé à l’aide de Git.
>

