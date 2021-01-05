---
title: Surveiller la connectivité des appareils à l’aide du IoT Central Explorer d’Azure
description: Surveillez les messages de l’appareil et observez les modifications de jumeau d’appareil via l’interface CLI d’IoT Central Explorer.
author: viv-liu
ms.author: viviali
ms.date: 03/27/2020
ms.topic: how-to
ms.service: iot-central
ms.custom: devx-track-azurecli, device-developer
services: iot-central
manager: corywink
ms.openlocfilehash: b266def0dbe81542714fc6d47986f3cde8235dd0
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92122958"
---
# <a name="monitor-device-connectivity-using-azure-cli"></a>Superviser la connectivité des appareils à l’aide d’Azure CLI

*Cette rubrique s’applique aux développeurs d’appareils et aux créateurs de solutions.*

Utilisez l’extension IoT d’Azure CLI pour consulter les messages que vos appareils envoient à IoT Central et observez les modifications apportées au jumeau d’appareil. Vous pouvez utiliser cet outil pour déboguer et observer la connectivité des appareils. Vous pouvez aussi diagnostiquer les problèmes des messages d’appareil n’atteignant pas le cloud ou les appareils qui ne répondent pas aux modifications du jumeau.

[Pour plus d’informations, consultez la référence sur les extensions d’Azure CLI](/cli/azure/ext/azure-iot/iot/central?view=azure-cli-latest)

## <a name="prerequisites"></a>Prérequis

+ Azure CLI installé et doté de la version 2.7.0 ou ultérieure. Vérifiez la version de votre Azure CLI en exécutant `az --version`. En savoir plus sur l’installation et la mise à jour à l’aide des [docs Azure CLI](/cli/azure/install-azure-cli)
+ Un compte professionnel ou scolaire dans Azure, ajouté en tant qu’utilisateur dans une application IoT Central.

## <a name="install-the-iot-central-extension"></a>Installer l’extension IoT Central

Exécutez la commande suivante dans la console de ligne de commande pour installer :

```azurecli
az extension add --name azure-iot
```

Vérifiez la version de l’extension en exécutant :

```azurecli
az --version
```

La version de l’extension azure-iot doit être 0.9.9 ou ultérieure. Si ce n’est pas le cas, exécutez :

```azurecli
az extension update --name azure-iot
```

## <a name="using-the-extension"></a>En utilisant l’extension

Les sections suivantes décrivent les commandes et les options courantes que vous pouvez utiliser quand vous exécutez `az iot central`. Pour voir l’ensemble complet des commandes et des options, passez `--help` à `az iot central` ou à une de ses sous-commandes.

### <a name="login"></a>Connexion

Commencez par vous connecter à Azure CLI. 

```azurecli
az login
```

### <a name="get-the-application-id-of-your-iot-central-app"></a>Obtenez l’ID d’application de votre application IoT Central
Dans **Administration/Paramètres de l’application**, copiez l’**ID d’application**. Vous utiliserez cette valeur ultérieurement.

### <a name="monitor-messages"></a>Surveiller les messages
Surveillez les messages envoyés à votre application IoT Central à partir de vos appareils. La sortie inclut l’ensemble des en-têtes et annotations.

```azurecli
az iot central diagnostics monitor-events --app-id <app-id> --properties all
```

### <a name="view-device-properties"></a>Voir les propriétés de l’appareil
Affichez les propriétés actuelles de lecture et de lecture/écriture pour un appareil donné.

```azurecli
az iot central device twin show --app-id <app-id> --device-id <device-id>
```

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, nous vous suggérons de vous renseigner sur la [connectivité des appareils dans Azure IoT Central](./concepts-get-connected.md).