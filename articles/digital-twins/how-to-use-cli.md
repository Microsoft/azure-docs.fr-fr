---
title: Utiliser l’interface CLI Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Découvrez comment utiliser l’interface CLI Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 05/25/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5e4c49e7aea05b6f430860eb6975713f59ad8080
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92635981"
---
# <a name="use-the-azure-digital-twins-cli"></a>Utiliser l’interface CLI Azure Digital Twins

En plus de gérer votre instance Azure Digital Twins dans le portail Azure, vous pouvez utiliser le **jeu de commandes pour [Azure CLI](/cli/azure/what-is-azure-cli)** d’Azure Digital Twins pour effectuer la plupart des actions principales dans le service, notamment :
* Gestion d’une instance Azure Digital Twins
* Gestion des modèles
* Gestion des jumeaux numériques
* Gestion des relations entre les jumeaux
* Configuration de points de terminaison
* Gestion des [routes](concepts-route-events.md)
* Configuration de la [sécurité](concepts-security.md) via le contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Le jeu de commandes est appelé **az dt** et fait partie de l’[extension Azure IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension). Vous pouvez afficher la liste complète des commandes et leur utilisation dans le cadre de la documentation de référence pour l’ensemble de commandes `az iot` : [*az dt* , informations de référence sur la commande](/cli/azure/ext/azure-iot/dt?preserve-view=true&view=azure-cli-latest).

## <a name="uses-deploy-and-validate"></a>Utilisations (déployer et valider)

En plus de la gestion générale de votre instance, l’interface CLI vous permet également d’effectuer des déploiements et des validations.
* Les commandes de plan de contrôle peuvent être utilisées pour rendre le déploiement d’une nouvelle instance reproductible ou automatisé.
* Les commandes de plan de données peuvent être utilisées pour vérifier rapidement les valeurs de votre instance et vérifier que les opérations se sont terminées comme prévu.

## <a name="get-the-command-set"></a>Obtenir le jeu de commandes

Les commandes Azure Digital Twins font partie de l’ [extension Azure IoT pour Azure CLI (azure-iot)](https://github.com/Azure/azure-iot-cli-extension), procédez donc comme suit pour vous assurer que vous disposez de la dernière extension `azure-iot` avec les commandes **az dt** .

### <a name="cli-version-requirements"></a>Version CLI requise

Si vous utilisez Azure CLI avec PowerShell, le package d’extension exige que votre version d’Azure CLI soit **2.3.1** ou une version ultérieure.

Vous pouvez vérifier votre version d’Azure CLI à l’aide de cette commande CLI :
```azurecli
az --version
```

Pour obtenir des instructions sur l’installation ou la mise à jour d’Azure CLI vers une version plus récente, consultez [*Installer Azure CLI*](/cli/azure/install-azure-cli).

### <a name="get-the-extension"></a>Obtient l’extension

Vous pouvez vous assurer de disposer de la version la plus récente de l’extension `azure-iot` en procédant comme suit. Vous pouvez exécuter ces commandes dans [Azure Cloud Shell](../cloud-shell/overview.md) ou dans une [interface de ligne de commande Azure locale](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).

[!INCLUDE [digital-twins-cloud-shell-extensions.md](../../includes/digital-twins-cloud-shell-extensions.md)]

## <a name="next-steps"></a>Étapes suivantes

Explorez l’interface CLI et son ensemble complet de commandes via les documents de référence :
* [*az dt* , informations de référence sur les commandes](/cli/azure/ext/azure-iot/dt?preserve-view=true&view=azure-cli-latest)