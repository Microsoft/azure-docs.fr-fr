---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 08/12/2019
ms.author: cephalin
ms.openlocfilehash: 92e39f128e90ba83a919388e217f0edc86f81770
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018423"
---
## <a name="deploy-zip-file-with-rest-apis"></a><a name="rest"></a>Déployer un fichier ZIP avec l’API REST 

Vous pouvez utiliser les [API REST du service de déploiement](https://github.com/projectkudu/kudu/wiki/REST-API) pour déployer le fichier .zip sur votre application dans Azure. Pour déployer, envoyez une demande POST à https://<nom_application>.scm.azurewebsites.net/api/zipdeploy. La demande POST doit contenir le fichier .zip dans le corps du message. Les informations d’identification de déploiement pour votre application sont fournies dans la demande avec l’authentification de base HTTP. Pour plus d’informations, consultez les [informations de référence des envois (push) de fichiers .zip](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file). 

Pour l’authentification HTTP BASIC, vous avez besoin de vos informations d’identification de déploiement App Service. Pour découvrir comment définir les informations d’identification de votre déploiement, consultez [Définir et réinitialiser les informations d’identification de niveau utilisateur](../articles/app-service/deploy-configure-credentials.md#userscope).

### <a name="with-curl"></a>Avec cURL

L’exemple suivant utilise l’outil cURL pour déployer un fichier .zip. Remplacez les espaces réservés `<deployment_user>`, `<zip_file_path>` et `<app_name>`. Quand vous y êtes invité par cURL, tapez le mot de passe.

```bash
curl -X POST -u <deployment_user> --data-binary @"<zip_file_path>" https://<app_name>.scm.azurewebsites.net/api/zipdeploy
```

Cette demande déclenche le déploiement par envoi (push) à partir du fichier .zip chargé. Vous pouvez examiner les déploiements en cours et passés en utilisant le point de terminaison `https://<app_name>.scm.azurewebsites.net/api/deployments`, comme le montre l’exemple cURL suivant. Ici encore, remplacez `<app_name>` par le nom de votre application et `<deployment_user>` par le nom d’utilisateur de vos informations d’identification de déploiement.

```bash
curl -u <deployment_user> https://<app_name>.scm.azurewebsites.net/api/deployments
```

### <a name="with-powershell"></a>Avec PowerShell

L’exemple suivant utilise [Publish-AzWebapp](/powershell/module/az.websites/publish-azwebapp) pour charger le fichier .zip. Remplacez les espaces réservés `<group-name>`, `<app-name>` et `<zip-file-path>`.

```powershell
Publish-AzWebapp -ResourceGroupName <group-name> -Name <app-name> -ArchivePath <zip-file-path>
```

Cette demande déclenche le déploiement par envoi (push) à partir du fichier .zip chargé. 

Pour passer en revue les déploiements en cours et passés, exécutez les commandes suivantes. Là encore, remplacez les valeurs des espaces réservés `<deployment-user>`, `<deployment-password>` et `<app-name>`.

```bash
$username = "<deployment-user>"
$password = "<deployment-password>"
$apiUrl = "https://<app-name>.scm.azurewebsites.net/api/deployments"
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $username, $password)))
$userAgent = "powershell/1.0"
Invoke-RestMethod -Uri $apiUrl -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)} -UserAgent $userAgent -Method GET
```
