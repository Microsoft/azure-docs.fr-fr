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
ms.openlocfilehash: e2c5794e5ce6e23b60bff513562f69c9333d6e34
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92743782"
---
Vous pouvez accéder aux journaux de la console générés à partir du conteneur.

Activez d’abord la journalisation du conteneur en exécutant la commande suivante :

```azurecli-interactive
az webapp log config --name <app-name> --resource-group <resource-group-name> --docker-container-logging filesystem
```

Remplacez `<app-name>` et `<resource-group-name>` par les noms appropriés pour votre application web.

Une fois la journalisation du conteneur activée, exécutez la commande suivante pour voir le flux de journal :

```azurecli-interactive
az webapp log tail --name <app-name> --resource-group <resource-group-name>
```

Si vous ne voyez pas les journaux d’activité de la console, attendez 30 secondes et vérifiez à nouveau.

Pour arrêter le streaming de journaux à tout moment, appuyez sur **Ctrl**+**C** .

Vous pouvez également inspecter les fichiers journaux dans un navigateur sur `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
