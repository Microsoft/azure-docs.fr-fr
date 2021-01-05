---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 08/14/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: f02fa49b62a2e3d617617a20518810209d3879b7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027098"
---
La configuration suivante a été utilisée pour les étapes ci-dessous :

- Ordinateur : Ubuntu Server 18.04
- Dépendances : strongSwan


Utilisez les commandes suivantes pour installer la configuration strongSwan requise :

```
sudo apt install strongswan
```

```
sudo apt install strongswan-pki
```

```
sudo apt install libstrongswan-extra-plugins
```

Utilisez la commande suivante pour installer l’interface de ligne de commande Azure :

```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

[Instructions supplémentaires sur l’installation de l’interface Azure CLI](/cli/azure/install-azure-cli-apt?view=azure-cli-latest)