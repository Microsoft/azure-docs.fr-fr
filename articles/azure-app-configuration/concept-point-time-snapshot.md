---
title: Récupérer des paires clé-valeur à un point dans le temps
titleSuffix: Azure App Configuration
description: Récupérez d’anciennes paires clé-valeur à l’aide de captures instantanées de points dans le temps dans Azure App Configuration, qui gère un enregistrement des modifications apportées aux paires clé-valeur.
services: azure-app-configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 08/05/2020
ms.openlocfilehash: fa2dbb11b3b8b9afd90c7f6fe3ffe77e2e57c4e6
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929895"
---
# <a name="point-in-time-snapshot"></a>Capture instantanée à un point dans le temps

Azure App Configuration conserve un enregistrement des modifications apportées aux paires clé-valeur. Cet enregistrement fournit une chronologie des modifications apportées aux paires clé-valeur. Vous pouvez reconstituer l’historique de toute paire clé-valeur et fournir sa valeur passée à tout moment au cours de la période d’historique de la clé (7 jours pour les magasins de niveau Gratuit, ou 30 jours pour les magasins de niveau Standard). Avec cette fonctionnalité, vous pouvez « voyager dans le passé » pour récupérer une ancienne valeur de clé. Par exemple, vous pouvez récupérer les paramètres de configuration qui étaient utilisés avant le dernier déploiement en date, afin de restaurer la précédente configuration de l’application.

## <a name="key-value-retrieval"></a>Récupération de paires clé-valeur

Vous pouvez utiliser le portail Azure ou CLI pour récupérer les paires clé-valeur précédentes. Dans Azure CLI, utilisez `az appconfig revision list` en ajoutant les paramètres permettant de récupérer les valeurs requises.  Spécifiez l’instance Azure App Configuration en fournissant le nom du magasin (`--name <app-config-store-name>`) ou en utilisant une chaîne de connexion (`--connection-string <your-connection-string>`). Limitez la sortie en spécifiant un point dans le temps (`--datetime`) et en spécifiant le nombre maximal d’éléments à retourner (`--top`).

Si Azure CLI n’est pas installé localement, vous pouvez éventuellement utiliser [Azure Cloud Shell](../cloud-shell/overview.md).

Récupérez toutes les modifications enregistrées qui ont été apportées à vos paires clé-valeur.

```azurecli
az appconfig revision list --name <your-app-config-store-name>.
```

Récupérez toutes les modifications enregistrées qui ont été apportées à la clé `environment` et aux étiquettes `test` et `prod`.

```azurecli
az appconfig revision list --name <your-app-config-store-name> --key environment --label test,prod
```

Récupérez toutes les modifications enregistrées qui ont été apportées à l’espace de clé hiérarchique `environment:prod`.

```azurecli
az appconfig revision list --name <your-app-config-store-name> --key environment:prod:* 
```

Récupérez toutes les modifications enregistrées qui ont été apportées à la clé `color` à un point dans le temps.

```azurecli
az appconfig revision list --connection-string <your-app-config-connection-string> --key color --datetime "2019-05-01T11:24:12Z" 
```

Récupérez les 10 dernières modifications enregistrées qui ont été apportées à vos paires clé-valeur, et retournez uniquement les valeurs avec `key`, `label` et l’horodatage `last_modified`.

```azurecli-interactive
az appconfig revision list --name <your-app-config-store-name> --top 10 --fields key label last_modified
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créez une application web ASP.NET Core](./quickstart-aspnet-core-app.md)