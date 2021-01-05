---
title: Accéder au Panneau de configuration Avere vFXT - Azure
description: Comment se connecter au cluster vFXT et au Panneau de configuration Avere basé sur le navigateur pour configurer Avere vFXT
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/14/2019
ms.author: rohogue
ms.openlocfilehash: 79e7c5db2a2c445ae740a21744a0bdfe0736c01a
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342431"
---
# <a name="access-the-vfxt-cluster"></a>Accéder au cluster vFXT

Pour modifier les paramètres et superviser le cluster, utilisez le Panneau de configuration Avere. Le Panneau de configuration Avere est une interface graphique basée sur le navigateur pour le cluster.

Étant donné que le cluster vFXT se situe dans un réseau virtuel privé, vous devez créer un tunnel SSH ou utiliser une autre méthode pour atteindre l’adresse IP de gestion du cluster.

Il existe deux étapes de base :

1. Créer une connexion entre votre station de travail et le réseau virtuel privé
1. Charger le panneau de configuration du cluster dans un navigateur web

> [!NOTE]
> Cet article suppose que vous avez défini une adresse IP publique sur le contrôleur de cluster ou sur une autre machine virtuelle à l’intérieur du réseau virtuel de votre cluster. Cet article décrit comment utiliser cette machine virtuelle comme hôte pour accéder au cluster. Si vous utilisez un VPN ou ExpressRoute pour l’accès au réseau virtuel, passez directement à [Se connecter au Panneau de configuration Avere](#connect-to-the-avere-control-panel-in-a-browser).

Avant de vous connecter, vérifiez que la paire de clés publique/privée SSH que vous avez utilisée lors de la création du contrôleur de cluster est installée sur votre ordinateur local. Lisez la documentation sur les clés SSH pour [Windows](../virtual-machines/linux/ssh-from-windows.md) ou pour [Linux](../virtual-machines/linux/mac-create-ssh-keys.md) si vous avez besoin d’aide. Si vous avez utilisé un mot de passe plutôt qu’une clé publique, vous serez invité à l’entrer lors de la connexion.

## <a name="create-an-ssh-tunnel"></a>Création d’un tunnel SSH

Vous pouvez créer un tunnel SSH à partir de la ligne de commande sur un système client Windows 10 ou Linux.

Utilisez une commande de tunneling SSL sous cette forme :

ssh -L *port_local*:*ip_gestion_cluster*:443 *nom_utilisateur_contrôleur*\@*IP_publique_contrôleur*

Cette commande se connecte à l’adresse IP de gestion du cluster via l’adresse IP du contrôleur de cluster.

Exemple :

```sh
ssh -L 8443:10.0.0.5:443 azureuser@203.0.113.51
```

L’authentification est automatique si vous avez utilisé votre clé publique SSH pour créer le cluster et la clé correspondante est installée sur le système client. Si vous avez utilisé un mot de passe, vous serez invité à l'entrer.

## <a name="connect-to-the-avere-control-panel-in-a-browser"></a>Se connecter au Panneau de configuration Avere dans un navigateur

Cette étape utilise un navigateur web pour se connecter à l’utilitaire de configuration installé sur le cluster vFXT.

* Pour une connexion de tunnel SSH, ouvrez votre navigateur web et accédez à `https://127.0.0.1:8443`.

  Vous vous êtes connecté à l’adresse IP du cluster quand vous avez créé le tunnel, vous avez donc simplement besoin d’utiliser l’adresse IP localhost dans le navigateur. Si vous avez utilisé un port local autre que 8443, utilisez à la place votre numéro de port.

* Si vous utilisez une connexion VPN ou ExpressRoute pour accéder au cluster, accédez à l’adresse IP de gestion du cluster dans votre navigateur. Exemple : ``https://203.0.113.51``

En fonction de votre navigateur, vous devrez peut-être cliquer sur **Avancé** et vérifier que vous pouvez accéder à la page sans risque.

Entrez le nom d’utilisateur `admin` et le mot de passe d'administration que vous avez fournis lors de la création du cluster.

![Capture d’écran de la page de connexion Avere renseignée avec le nom d’utilisateur « admin » et un mot de passe](media/avere-vfxt-gui-login.png)

Cliquez sur **Connexion** ou appuyez sur Entrée sur votre clavier.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous êtes connecté au panneau de configuration du cluster, activez le [support](avere-vfxt-enable-support.md).