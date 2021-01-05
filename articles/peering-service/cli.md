---
title: Inscrire une connexion Peering Service en préversion à l’aide d’Azure CLI
description: Découvrez comment inscrire une connexion Peering Service à l’aide d’Azure CLI.
services: peering-service
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 05/2/2020
ms.author: derekol
ms.openlocfilehash: e7b696ba052b2aca9e14628327c07275845607ad
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540584"
---
# <a name="register-a-peering-service-connection-by-using-the-azure-cli"></a>Inscrire une connexion Peering Service à l’aide d’Azure CLI.

Azure Peering Service est un service réseau qui améliore la connectivité entre le client et les services cloud Microsoft, comme Microsoft 365, Dynamics 365, les services SaaS, Azure ou les services Microsoft accessibles via l’Internet public. Cet article explique comment inscrire une connexion Peering Service en utilisant Azure CLI.

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Exécutez [az version](/cli/azure/reference-index#az_version) pour rechercher la version et les bibliothèques dépendantes installées. Pour effectuer une mise à niveau vers la dernière version, exécutez [az upgrade](/cli/azure/reference-index#az_upgrade).

## <a name="prerequisites"></a>Prérequis 

Vous devez disposer des éléments suivants :

### <a name="azure-account"></a>Compte Azure

Vous devez avoir un compte Microsoft Azure actif et valide. Sinon, vous ne pourrez pas configurer la connexion Peering Service. Peering Service est une ressource fournie dans les abonnements Azure.

### <a name="connectivity-provider"></a>Fournisseur de connectivité

Vous pouvez faire appel à un fournisseur de services Internet ou un partenaire d’échange Internet pour obtenir une connectivité Peering Service qui vous permettra de connecter votre réseau au réseau Microsoft.

Assurez-vous que les fournisseurs de connectivité sont des partenaires de Microsoft.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

### <a name="1-select-your-subscription"></a>Sélectionnez votre abonnement

Sélectionnez l’abonnement pour lequel vous souhaitez inscrire la connexion Peering Service.

```azurecli-interactive
az account set --subscription "<subscription ID>"
```

Si vous n’avez pas encore de groupe de ressources, vous devez en créer un avant d’inscrire votre connexion Peering Service. Vous pouvez créer un groupe de ressources à l’aide de la commande suivante :

```azurecli-interactive
az group create -n MyResourceGroup -l "West US"
```

### <a name="2-register-your-subscription-with-the-resource-provider-and-feature-flag"></a>2. Inscrire votre abonnement auprès du fournisseur de ressources et de l’indicateur de fonctionnalité

Avant de passer aux étapes d’inscription de la connexion Peering Service à l’aide d’Azure CLI, inscrivez votre abonnement auprès du fournisseur de ressources et de l’indicateur de fonctionnalité à l’aide d’Azure CLI. Les commandes Azure CLI sont spécifiées ici :

```azurecli-interactive

az feature register --namespace Microsoft.Peering --name AllowPeeringService

```

### <a name="3-register-the-peering-service-connection"></a>3. Inscrire la connexion Peering Service

Inscrivez la connexion Peering Service à l’aide de l’ensemble de commandes suivant dans Azure CLI. Cet exemple inscrit la connexion Peering Service nommée myPeeringService.

```azurecli-interactive
az peering service create : Create peering service\
  --location -l \
  --name myPeeringService\
  --resource-group -g MyResourceGroup\
  --peering-service-location\
  --peering-service-provider\
  --tags
```

### <a name="4-register-the-prefix"></a>4. Inscrire le préfixe

Inscrivez le préfixe fourni par le fournisseur de connectivité en exécutant les commandes suivantes dans Azure CLI. Cet exemple inscrit le préfixe nommé myPrefix.

```azurecli-interactive
az peering service prefix create \
  --name  myPrefix\
  --peering-service-name myPeeringService\
  --resource-group  -g myResourceGroup\
```

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).
- Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).
- Pour inscrire la connexion par le biais d’Azure PowerShell, consultez [Inscrire une connexion Peering Service - Azure PowerShell](powershell.md).
- Pour inscrire la connexion à l’aide du portail Azure, consultez [Inscrire une connexion Peering Service - Portail Azure](azure-portal.md).
