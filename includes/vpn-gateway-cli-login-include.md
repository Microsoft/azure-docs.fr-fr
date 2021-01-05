---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 2ffd8b42bf43658ba488f4c1c92bb7af5b88339e
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92755326"
---
Connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/) et suivez les instructions à l’écran. Pour plus d’informations sur la connexion, consultez [Bien démarrer avec Azure CLI](/cli/azure/get-started-with-azure-cli).

```azurecli
az login
```

Si vous disposez de plusieurs abonnements Azure, répertoriez les abonnements associés au compte.

```azurecli
az account list --all
```

Spécifiez l’abonnement à utiliser.

```azurecli
az account set --subscription <replace_with_your_subscription_id>
```
