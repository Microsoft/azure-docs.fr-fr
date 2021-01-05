---
title: Accès SSH pour conteneurs Linux
description: Vous pouvez ouvrir une session SSH sur un conteneur Linux dans Azure App Service. Les conteneurs Linux personnalisés sont pris en charge en apportant quelques modifications à votre image personnalisée.
keywords: azure app service, application web, linux, oss
author: msangapu-msft
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.topic: article
ms.date: 02/25/2019
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: 7178b8079bbb2411e0b38c3ef59a9981fb2d55be
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97005237"
---
# <a name="open-an-ssh-session-to-a-linux-container-in-azure-app-service"></a>Ouvrir une session SSH sur un conteneur Linux dans Azure App Service

[Secure Shell (SSH)](https://wikipedia.org/wiki/Secure_Shell) est couramment utilisé pour exécuter des commandes d’administration à distance à partir d’un terminal de ligne de commande. App Service sur Linux assure la prise en charge de SSH dans le conteneur d’application. 

![SSH App Service Linux](./media/configure-linux-open-ssh-session/app-service-linux-ssh.png)

Vous pouvez également vous connecter au conteneur directement à partir de votre machine de développement locale par SSH et SFTP.

## <a name="open-ssh-session-in-browser"></a>Ouvrir une session SSH dans un navigateur

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-no-h.md)]

## <a name="use-ssh-support-with-custom-docker-images"></a>Utiliser la prise en charge SSH avec des images Docker personnalisées

Consultez [Configurer SSH dans un conteneur personnalisé](configure-custom-container.md#enable-ssh).

## <a name="open-ssh-session-from-remote-shell"></a>Ouvrir une session SSH à partir d’un interpréteur de commandes à distance

> [!NOTE]
> Cette fonctionnalité est actuellement en préversion.
>

À l’aide du tunneling TCP, vous pouvez créer une connexion réseau entre votre machine de développement et Web App pour conteneurs via une connexion WebSocket authentifiée. Il vous permet d’ouvrir une session SSH avec votre conteneur en cours d’exécution dans App Service à partir du client de votre choix.

Pour commencer, vous devez installer [Azure CLI](/cli/azure/install-azure-cli). Pour voir comment il fonctionne sans installer Azure CLI, ouvrez [Azure Cloud Shell](../cloud-shell/overview.md). 

Ouvrez une connexion à distance vers votre application à l’aide de la commande [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection#ext-webapp-az-webapp-remote-connection-create). Spécifiez _\<subscription-id>_ , _\<group-name>_ et \_\<app-name>_ pour votre application.

```azurecli-interactive
az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &
```

> [!TIP]
> `&` à la fin de la commande est simplement présent pour des raisons pratiques si vous utilisez Cloud Shell. Il exécute le processus en arrière-plan afin que vous puissiez exécuter la commande suivante dans le même interpréteur de commandes.

La sortie de commande vous donne les informations dont vous avez besoin pour ouvrir une session SSH.

```output
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

Ouvrez une session SSH avec votre conteneur au moyen du client de votre choix, en utilisant le port local. L’exemple suivant utilise la commande [ssh](https://ss64.com/bash/ssh.html) par défaut :

```bash
ssh root@127.0.0.1 -p <port>
```

Lorsque cela vous est demandé, tapez `yes` pour poursuivre la connexion. Vous êtes alors invité à entrer le mot de passe. Utilisez `Docker!`, qui vous a été indiqué précédemment.

<pre>
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
</pre>

Une fois authentifié, vous devez voir l’écran d’accueil de session.

<pre>
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  &gt;
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
</pre>

Vous êtes maintenant connecté à votre connecteur.  

Essayez d’exécuter la commande [top](https://ss64.com/bash/top.html). Vous devez être en mesure de voir les processus de votre application dans la liste des processus. Dans l’exemple ci-dessous, il s’agit de celui avec `PID 263`.

<pre>
Mem: 1578756K used, 127032K free, 8744K shrd, 201592K buff, 341348K cached
CPU:   3% usr   3% sys   0% nic  92% idle   0% io   0% irq   0% sirq
Load average: 0.07 0.04 0.08 4/765 45738
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1528   0%   0   0% /sbin/init
  235     1 root     S     632m  38%   0   0% PM2 v2.10.3: God Daemon (/root/.pm2)
  263   235 root     S     630m  38%   0   0% node /home/site/wwwroot/app.js
  482   291 root     S     7368   0%   0   0% sshd: root@pts/0
45513   291 root     S     7356   0%   0   0% sshd: root@pts/1
  291     1 root     S     7324   0%   0   0% /usr/sbin/sshd
  490   482 root     S     1540   0%   0   0% -ash
45539 45513 root     S     1540   0%   0   0% -ash
45678 45539 root     R     1536   0%   0   0% top
45733     1 root     Z        0   0%   0   0% [init]
45734     1 root     Z        0   0%   0   0% [init]
45735     1 root     Z        0   0%   0   0% [init]
45736     1 root     Z        0   0%   0   0% [init]
45737     1 root     Z        0   0%   0   0% [init]
45738     1 root     Z        0   0%   0   0% [init]
</pre>

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez poser des questions et signaler vos préoccupations sur le [forum Azure](/answers/topics/azure-webapps.html).

Pour plus d’informations sur Web App pour conteneurs, consultez :

* [Présentation du débogage distant des applications Node.js sur Azure App Service à partir de VS Code](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0)
* [Démarrage rapide : Exécuter un conteneur personnalisé sur App Service](quickstart-custom-container.md?pivots=container-linux)
* [Utiliser Ruby dans Azure App Service sur Linux](quickstart-ruby.md)
* [Forum aux questions sur Azure App Service Web App for Containers](faq-app-service-linux.md)
