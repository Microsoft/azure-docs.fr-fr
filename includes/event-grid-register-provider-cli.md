---
title: Fichier Include
description: Fichier Include
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: include
ms.date: 08/17/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 579ea74c4767ba1afb6e0128c9f12ff83f7f3998
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826642"
---
## <a name="enable-the-event-grid-resource-provider"></a>Activer le fournisseur de ressources Event Grid

Si vous n’avez jamais utilisé Event Grid dans votre abonnement Azure, vous risquez de devoir inscrire le fournisseur de ressources Event Grid. Exécutez la commande suivante pour enregistrer le fournisseur :

```azurecli-interactive
az provider register --namespace Microsoft.EventGrid
```

L’inscription peut prendre un certain temps. Pour vérifier l’état, exécutez :

```azurecli-interactive
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

Lorsque `registrationState` est `Registered`, vous êtes prêt à continuer.
