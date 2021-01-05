---
title: Exemple de script Azure CLI - Réinitialiser vos informations d’identification de compte | Microsoft Docs
description: Utilisez le script Azure CLI pour réinitialiser vos informations d’identification de compte et récupérer les paramètres du fichier app.config.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: da997ec71655231876749d9f3aa65ba06fd3a1f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89295833"
---
# <a name="azure-cli-example-reset-the-account-credentials"></a>Exemple Azure CLI : Réinitialiser les informations d’identification

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Le script Azure CLI de cet article vous montre comment réinitialiser vos informations d’identification de compte et récupérer les paramètres du fichier app.config.

## <a name="prerequisites"></a>Prérequis

[Créer un compte Media Services](./create-account-howto.md).

[!INCLUDE [media-services-cli-instructions.md](../../../includes/media-services-cli-instructions.md)]

## <a name="example-script"></a>Exemple de script

```azurecli-interactive
# Update the following variables for your own settings:
resourceGroup=amsResourceGroup
amsAccountName=amsmediaaccountname

az ams account sp reset-credentials \
  --account-name $amsAccountName \
  --resource-group $resourceGroup
 ```

## <a name="next-steps"></a>Étapes suivantes

* [az ams](/cli/azure/ams)
* [Réinitialiser les informations d’identification](/cli/azure/ams/account/sp#az-ams-account-sp-reset-credentials)
