---
author: IEvangelist
ms.author: dapine
ms.date: 06/25/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 2bfb700ac5c220b780c05c8d415a4506c7a2f871
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001161"
---
Les paramètres `Logging` gèrent la prise en charge de la journalisation ASP.NET Core pour votre conteneur. Vous pouvez utiliser pour votre conteneur les mêmes paramètres et valeurs de configuration que ceux d’une application ASP.NET Core. 

Le conteneur prend en charge les fournisseurs de journalisation suivants :

|Fournisseur|Objectif|
|--|--|
|[Console](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#console-provider)|Fournisseur de journalisation `Console` ASP.NET Core. Tous les paramètres de configuration ASP.NET Core et les valeurs par défaut de ce fournisseur de journalisation sont pris en charge.|
|[Déboguer](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1#debug-provider)|Fournisseur de journalisation `Debug` ASP.NET Core. Tous les paramètres de configuration ASP.NET Core et les valeurs par défaut de ce fournisseur de journalisation sont pris en charge.|
|[Disque](#disk-logging)|Fournisseur de journalisation JSON. Ce fournisseur de journalisation écrit les données de journal dans le montage de sortie.|

Cette commande de conteneur stocke des informations de journalisation au format JSON dans le montage de sortie :

```bash
docker run --rm -it -p 5000:5000 \
--memory 2g --cpus 1 \
--mount type=bind,src=/home/azureuser/output,target=/output \
<registry-location>/<image-name> \
Eula=accept \
Billing=<endpoint> \
ApiKey=<api-key> \
Logging:Disk:Format=json
```

Cette commande de conteneur affiche des informations de débogage, avec le préfixe `dbug`, tandis que le conteneur s’exécute :

```bash
docker run --rm -it -p 5000:5000 \
--memory 2g --cpus 1 \
<registry-location>/<image-name> \
Eula=accept \
Billing=<endpoint> \
ApiKey=<api-key> \
Logging:Console:LogLevel:Default=Debug
```

### <a name="disk-logging"></a>Journalisation Disk

Le fournisseur de journalisation `Disk` prend en charge les paramètres de configuration suivants :

| Name | Type de données | Description |
|------|-----------|-------------|
| `Format` | String | Format de sortie des fichiers journaux.<br/> **Remarque :** Cette valeur doit être définie sur `json` pour activer le fournisseur de journalisation. Si cette valeur est spécifiée sans que le montage de sortie soit aussi spécifié pendant l’instanciation d’un conteneur, une erreur se produit. |
| `MaxFileSize` | Integer | Taille maximale, en mégaoctets (Mo), d’un fichier journal. Dès que la taille du fichier journal actif atteint ou dépasse cette valeur, un nouveau fichier journal est commencé par le fournisseur de journalisation. Si la valeur -1 est spécifiée, la taille du fichier journal est limitée uniquement par la taille de fichier maximale, le cas échéant, pour le montage de sortie. La valeur par défaut est 1. |

Pour plus d’informations sur la configuration de la prise en charge de la journalisation ASP.NET Core, consultez [Configuration d’un fichier de paramètres](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1).