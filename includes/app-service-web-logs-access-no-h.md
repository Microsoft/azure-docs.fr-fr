---
title: Fichier include
description: Fichier include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 03/27/2019
ms.author: cephalin
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 7803ac9009af1657e7f162d656898492a694e28f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997908"
---
Pour accéder aux journaux de la console générés à l’intérieur du code de votre application dans App Service, activez la journalisation des diagnostics en exécutant la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp log config --resource-group <resource-group-name> --name <app-name> --application-logging true --level Verbose
```

Les valeurs possibles pour `--level` sont : `Error`, `Warning`, `Info` et `Verbose`. Chaque niveau suivant comprend le niveau précédent. Par exemple : `Error` comprend uniquement les messages d’erreur et `Verbose` comprend tous les messages.

Une fois la journalisation des diagnostics activée, exécutez la commande suivante pour voir le flux de journal :

```azurecli-interactive
az webapp log tail --resource-group <resource-group-name> --name <app-name>
```

Si vous ne voyez pas les journaux d’activité de la console, attendez 30 secondes et vérifiez à nouveau.

> [!NOTE]
> Vous pouvez également inspecter les fichiers journaux à partir du navigateur sur `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Pour arrêter le streaming des journaux à tout moment, appuyez sur `Ctrl`+`C`.
