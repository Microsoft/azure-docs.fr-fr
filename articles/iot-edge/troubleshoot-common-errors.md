---
title: Erreurs courantes - Azure IoT Edge | Microsoft Docs
description: Cet article vous donne des conseils pour résoudre des problèmes courants rencontrés au moment du déploiement d’une solution IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 98ee865a3ddf6c26ffe9cb77767f3872b42018d8
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442359"
---
# <a name="common-issues-and-resolutions-for-azure-iot-edge"></a>Problèmes courants et résolutions pour Azure IoT Edge

Cet article explique les étapes à suivre pour résoudre des problèmes courants que vous êtes susceptible de rencontrer pendant le déploiement de solutions IoT Edge. Si vous souhaitez savoir comment trouver les journaux et les erreurs sur votre appareil IoT Edge, consultez [Problèmes courants et résolutions pour Azure IoT Edge](troubleshoot.md).

## <a name="iot-edge-agent-stops-after-about-a-minute"></a>L’agent IoT Edge s’arrête après environ une minute

**Comportement observé :**

Le module edgeAgent démarre et s’exécute avec succès pendant environ une minute, puis s’arrête. Les journaux d’activité indiquent que l’agent IoT Edge tente de se connecter à IoT Hub via AMQP, puis essaie de se connecter à l’aide d’AMQP sur WebSocket. Lorsque cette tentative échoue, l’agent IoT Edge se ferme.

Exemples de journaux d’activité edgeAgent :

```output
2017-11-28 18:46:19 [INF] - Starting module management agent.
2017-11-28 18:46:19 [INF] - Version - 1.0.7516610 (03c94f85d0833a861a43c669842f0817924911d5)
2017-11-28 18:46:19 [INF] - Edge agent attempting to connect to IoT Hub via AMQP...
2017-11-28 18:46:49 [INF] - Edge agent attempting to connect to IoT Hub via AMQP over WebSocket...
```

**Cause racine :**

Une configuration de mise en réseau sur le réseau hôte empêche l’agent IoT Edge d’atteindre le réseau. L’agent tente de se connecter via AMQP (port 5671) en premier. Si la connexion échoue, il essaie via les WebSockets (port 443).

Le runtime IoT Edge configure un réseau pour chacun des modules sur lesquels communiquer. Sur Linux, ce réseau est un réseau de pont. Sur Windows, il utilise NAT. Ce problème est plus courant sur les appareils Windows utilisant des conteneurs Windows qui utilisent le réseau NAT.

**Résolution :**

Assurez-vous qu’il existe un itinéraire vers Internet pour les adresses IP affectées à ce réseau pont/NAT. Parfois, une configuration VPN sur l’hôte remplace le réseau IoT Edge.

## <a name="iot-edge-agent-cant-access-a-modules-image-403"></a>L’agent IoT Edge ne peut pas accéder à l’image d’un module (403)

**Comportement observé :**

Un conteneur ne s’exécute pas et les journaux d’activité edgeAgent comportent une erreur 403.

**Cause racine :**

L’agent IoT Edge ne possède pas les autorisations pour accéder à l’image d’un module.

**Résolution :**

Vérifiez que les informations d’identification du Registre sont correctement spécifiées dans le manifeste de déploiement.

## <a name="edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device"></a>Le module Edge Agent rapporte le message « fichier config vide » et aucun module ne démarre sur l’appareil

**Comportement observé :**

L’appareil rencontre des difficultés pour démarrer les modules définis dans le déploiement. Seul edgeAgent s’exécute mais rapporte en continu le message « fichier config vide ».

**Cause racine :**

Par défaut, IoT Edge démarre les modules dans leur réseau de conteneur isolé. L’appareil peut rencontrer des difficultés avec la résolution de noms DNS au sein de ce réseau privé.

**Résolution :**

**Option n°1 : Définissez le serveur DNS dans les paramètres du moteur de conteneur**

Spécifiez le serveur DNS de votre environnement dans les paramètres du moteur de conteneur qui s’appliquent à tous les modules de conteneur démarrés par le moteur. Créez un fichier nommé `daemon.json` en spécifiant le serveur DNS à utiliser. Par exemple :

```json
{
    "dns": ["1.1.1.1"]
}
```

L’exemple ci-dessus définit le serveur DNS sur un service DNS accessible publiquement. Si l’appareil de périphérie ne peut pas accéder à cette adresse IP depuis son environnement, remplacez l’adresse par l’adresse du serveur DNS qui est accessible.

Placez `daemon.json` au bon emplacement de votre plateforme :

| Plateforme | Emplacement |
| --------- | -------- |
| Linux | `/etc/docker` |
| Hôte Windows avec des conteneurs Windows | `C:\ProgramData\iotedge-moby\config` |

Si l’emplacement contient déjà le fichier `daemon.json`, ajoutez la clé **dns** et enregistrez-le.

Redémarrez le moteur de conteneur pour que les mises à jour prennent effet.

| Plateforme | Commande |
| --------- | -------- |
| Linux | `sudo systemctl restart docker` |
| Windows (Admin PowerShell) | `Restart-Service iotedge-moby -Force` |

**Option n°2 : Définissez le serveur DNS dans le déploiement IoT Edge par module**

Vous pouvez définir le serveur DNS pour *createOptions* de chaque module dans le déploiement IoT Edge. Par exemple :

```json
"createOptions": {
  "HostConfig": {
    "Dns": [
      "x.x.x.x"
    ]
  }
}
```

Veillez à définir cette configuration également pour les modules *edgeAgent* et *edgeHub*.

## <a name="iot-edge-hub-fails-to-start"></a>Edge Hub ne parvient pas à démarrer

**Comportement observé :**

Le module edgeHub ne démarre pas. Vous pouvez voir un message semblable à l’une des erreurs suivantes dans les journaux :

```output
One or more errors occurred.
(Docker API responded with status code=InternalServerError, response=
{\"message\":\"driver failed programming external connectivity on endpoint edgeHub (6a82e5e994bab5187939049684fb64efe07606d2bb8a4cc5655b2a9bad5f8c80):
Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is already allocated\"}\n)
```

ou

```output
info: edgelet_docker::runtime -- Starting module edgeHub...
warn: edgelet_utils::logging -- Could not start module edgeHub
warn: edgelet_utils::logging --     caused by: failed to create endpoint edgeHub on network nat: hnsCall failed in Win32:  
        The process cannot access the file because it is being used by another process. (0x20)
```

**Cause racine :**

Un autre processus sur la machine hôte a lié un port que le module edgeHub tente de lier. Le hub IoT Edge mappe les ports 443, 5671 et 8883 pour une utilisation dans les scénarios de passerelle. Le module ne démarre pas si un autre processus a déjà lié l’un de ces ports.

**Résolution :**

Vous pouvez résoudre ce problème de deux manières :

Si l’appareil IoT Edge fonctionne en tant qu’appareil de passerelle, vous devez rechercher et arrêter le processus qui utilise actuellement le port 443, 5671 ou 8883. Une erreur sur le port 443 signifie généralement que l’autre processus est un serveur web.

Si vous n’avez pas besoin d’utiliser l’appareil IoT Edge en tant que passerelle, supprimez les liaisons de port dans les options de création de module edgeHub. Vous pouvez modifier ces options de création dans le Portail Azure, ou directement dans le fichier deployment.json.

Dans le portail Azure :

1. Accédez à votre hub IoT, puis sélectionnez **IoT Edge**.

2. Sélectionnez l’appareil IoT Edge que vous souhaitez modifier.

3. Sélectionnez **Définir modules**.

4. Sélectionnez **Paramètres du runtime**.

5. Dans les paramètres du module **Edge Hub**, supprimez tout le contenu de la zone de texte **Options de création**.

6. Enregistrez vos modifications et créez le déploiement.

Dans le fichier deployment.json :

1. Ouvrez le fichier deployment.json que vous avez appliqué à votre appareil IoT Edge.

2. Recherchez les paramètres `edgeHub` dans la section des propriétés souhaitées pour edgeAgent :

   ```json
   "edgeHub": {
       "settings": {
           "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
           "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
       },
       "type": "docker",
       "status": "running",
       "restartPolicy": "always"
   }
   ```

3. Supprimez la ligne `createOptions` ainsi que la virgule à la fin de la ligne `image` au-dessus :

   ```json
   "edgeHub": {
       "settings": {
           "image": "mcr.microsoft.com/azureiotedge-hub:1.0"
       },
       "type": "docker",
       "status": "running",
       "restartPolicy": "always"
   }
   ```

4. Enregistrez le fichier et réappliquez-le à votre appareil IoT Edge.

## <a name="iot-edge-security-daemon-fails-with-an-invalid-hostname"></a>Le démon de sécurité IoT Edge échoue avec un nom d’hôte non valide

**Comportement observé :**

La tentative de [vérification des journaux du gestionnaire de sécurité IoT Edge](troubleshoot.md#check-the-status-of-the-iot-edge-security-manager-and-its-logs) échoue, avec le message suivant :

```output
Error parsing user input data: invalid hostname. Hostname cannot be empty or greater than 64 characters
```

**Cause racine :**

Le runtime IoT Edge peut prendre uniquement en charge les noms d’hôte avec moins de 64 caractères. Les machines physiques n’ont généralement pas de noms d’hôte longs, mais le problème est plus courant sur une machine virtuelle. Les noms d’hôtes générés automatiquement pour les machines virtuelles Windows hébergées dans Azure, en particulier, sont généralement longs.

**Résolution :**

Lorsque vous voyez cette erreur, vous pouvez la résoudre par la configuration du nom DNS de votre machine virtuelle, puis en définissant le nom DNS en tant que nom d’hôte dans la commande d’installation.

1. Dans le Portail Azure, accédez au panneau Vue d’ensemble de votre machine virtuelle.
2. Sélectionnez **configurer** sous le nom DNS. Si votre machine virtuelle a déjà un nom DNS configuré, vous n’avez pas besoin d’en configurer un nouveau.

   ![Configurer le nom DNS de la machine virtuelle](./media/troubleshoot/configure-dns.png)

3. Fournissez une valeur pour **l’étiquette de nom DNS** et sélectionnez **Enregistrer**.
4. Copiez le nouveau nom DNS qui doit être au format **\<DNSnamelabel\>.\<vmlocation\>.cloudapp.azure.com**.
5. À l’intérieur de la machine virtuelle, utilisez la commande suivante pour installer le runtime IoT Edge avec votre nom DNS :

   * Sur Linux :

      ```bash
      sudo nano /etc/iotedge/config.yaml
      ```

   * Sur Windows :

      ```cmd
      notepad C:\ProgramData\iotedge\config.yaml
      ```

## <a name="cant-get-the-iot-edge-daemon-logs-on-windows"></a>Impossible d’obtenir les journaux d’activité de démon IoT Edge sur Windows

**Comportement observé :**

Vous obtenez une exception EventLogException quand vous utilisez `Get-WinEvent` sur Windows.

**Cause racine :**

La commande PowerShell `Get-WinEvent` repose sur la présence d’une entrée de Registre pour trouver les journaux d’activité d’après un `ProviderName` spécifique.

**Résolution :**

Définissez une entrée de Registre pour le démon IoT Edge. Créez un fichier **iotedge.reg** avec le contenu suivant, puis importez-le dans le Registre Windows en double-cliquant dessus ou en utilisant la commande `reg import iotedge.reg` :

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
"CustomSource"=dword:00000001
"EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
"TypesSupported"=dword:00000007
```

## <a name="stability-issues-on-smaller-devices"></a>Problèmes de stabilité sur les petits appareils

**Comportement observé :**

Vous rencontrez parfois des problèmes de stabilité sur les appareils avec contraintes de ressources, comme le Raspberry Pi, en particulier quand ils sont utilisés comme passerelles. Les symptômes incluent des exceptions de mémoire insuffisante dans le module hub IoT Edge, l’échec de la connexion d’appareils en aval ou l’échec de l’envoi des messages de télémétrie d’un appareil au bout de quelques heures.

**Cause racine :**

Le hub IoT Edge, qui fait partie du runtime IoT Edge, est optimisé pour les performances par défaut et tente d’allouer de grandes quantités de mémoire. Cette optimisation n’est pas idéale pour les appareils de périphérie limités et peut entraîner des problèmes de stabilité.

**Résolution :**

Pour le hub IoT Edge, affectez la valeur **false** à une variable d’environnement **OptimizeForPerformance**. Il existe deux façons de définir des variables d’environnement :

Dans le portail Azure :

Dans votre hub IoT, sélectionnez votre appareil IoT Edge puis, à partir de la page de détails de l’appareil, sélectionnez **Définir des modules** > **Paramètres du runtime**. Créez une variable d'environnement pour le module hub IoT Edge appelé *OptimizeForPerformance* en la définissant sur *false*.

![OptimizeForPerformance défini sur false](./media/troubleshoot/optimizeforperformance-false.png)

Dans le manifeste de déploiement :

```json
"edgeHub": {
  "type": "docker",
  "settings": {
    "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
    "createOptions": <snipped>
  },
  "env": {
    "OptimizeForPerformance": {
      "value": "false"
    }
  },
```

## <a name="iot-edge-module-fails-to-send-a-message-to-edgehub-with-404-error"></a>Le module IoT Edge ne parvient pas à envoyer de message à edgeHub et retourne l’erreur 404

**Comportement observé :**

Un module IoT Edge personnalisé ne parvient pas à envoyer de message au hub IoT Edge et retourne l’erreur `Module not found` 404. Le démon IoT Edge imprime le message suivant dans les journaux d’activité :

```output
Error: Time:Thu Jun  4 19:44:58 2018 File:/usr/sdk/src/c/provisioning_client/adapters/hsm_client_http_edge.c Func:on_edge_hsm_http_recv Line:364 executing HTTP request fails, status=404, response_buffer={"message":"Module not found"}u, 04 )
```

**Cause racine :**

Pour des raisons de sécurité, le démon IoT Edge applique l’identification du processus à tous les modules se connectant à l’edgeHub. Il vérifie que tous les messages envoyés par un module proviennent de l’ID de processus principal du module. Si un message est envoyé par un module depuis un ID de processus différent de celui qui a été établi initialement, il rejette le message avec un message d’erreur 404.

**Résolution :**

Depuis la version 1.0.7, tous les processus de module sont autorisés à se connecter. Pour plus d’informations, consultez les [Notes de version pour la version 1.0.7](https://github.com/Azure/iotedge/blob/master/CHANGELOG.md#iotedged-1).

Si la mise à niveau vers la version 1.0.7 n’est pas possible, procédez comme suit. Assurez-vous que le même ID de processus est bien toujours utilisé par le module IoT Edge personnalisé pour envoyer des messages au hub de périphérie. Par exemple, utilisez la commande `ENTRYPOINT` au lieu de `CMD` dans votre fichier Docker. La commande `CMD` crée un ID de processus pour le module et un autre ID de processus pour la commande bash exécutant le programme principal, alors que la commande `ENTRYPOINT` créé un ID de processus unique.

## <a name="iot-edge-module-deploys-successfully-then-disappears-from-device"></a>Après son déploiement, le module IoT Edge n’est plus visible sur l’appareil

**Comportement observé :**

Une fois que les modules d’un appareil IoT Edge ont été définis, ils sont déployés, mais au bout de quelques minutes, ils ne sont plus visibles sur l’appareil et ils n’apparaissent plus dans les détails de l’appareil dans le Portail Azure. D’autres modules que ceux définis peuvent également apparaître sur l’appareil.

**Cause racine :**

Si un déploiement automatique cible un appareil, il est prioritaire sur la définition manuelle des modules pour un seul appareil. La fonctionnalité **Définir des modules** dans le Portail Azure ou la fonctionnalité **Créer un déploiement pour un seul appareil** dans Visual Studio Code est appliquée pendant un moment. Vous voyez les modules que vous avez définis démarrer sur l’appareil. Ensuite, la priorité du déploiement automatique est appliquée et remplace les propriétés souhaitées de l’appareil.

**Résolution :**

Utilisez un seul type de mécanisme de déploiement par appareil, qu’il s’agisse d’un déploiement automatique ou de déploiements d’appareils individuels. Si vous avez plusieurs déploiements automatiques ciblant un appareil, vous pouvez changer la priorité ou les descriptions des cibles pour être sûr que le déploiement approprié est effectué sur l’appareil. Vous pouvez également mettre à jour le jumeau d’appareil pour qu’il ne corresponde plus à la description de la cible du déploiement automatique.

Pour plus d’informations, consultez [Comprendre les déploiements automatiques IoT Edge pour un seul ou de nombreux appareils](module-deployment-monitoring.md).

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

## <a name="iot-edge-behind-a-gateway-cannot-perform-http-requests-and-start-edgeagent-module"></a>IoT Edge derrière une passerelle ne peut pas exécuter de requêtes HTTP ni démarrer le module edgeAgent

**Comportement observé :**

Le démon IoT Edge est actif avec un fichier de configuration valide, mais il ne peut pas démarrer le module edgeAgent. La commande `iotedge list` retourne une liste vide. Le démon IoT Edge enregistre le rapport `Could not perform HTTP request`.

**Cause racine :**

Les appareils IoT Edge derrière une passerelle obtiennent leurs images de module à partir de l’appareil IoT Edge parent spécifié dans le champ `parent_hostname` du fichier config.yaml. L’erreur `Could not perform HTTP request` signifie que l’appareil enfant n’est pas en mesure d’atteindre son appareil parent via HTTP.

**Résolution :**

Assurez-vous que l’appareil de IoT Edge parent peut recevoir des demandes entrantes de l’appareil IoT Edge enfant. Ouvrez le trafic réseau sur les ports 443 et 6617 pour les requêtes provenant de l’appareil enfant.

:::moniker-end

## <a name="next-steps"></a>Étapes suivantes

Vous pensez que vous avez trouvé un bogue dans la plateforme IoT Edge ? [Soumettez un problème](https://github.com/Azure/iotedge/issues) afin que nous puissions poursuivre les améliorations.

Si vous avez d'autres questions, créez une [Support request](https://portal.azure.com/#create/Microsoft.Support) pour obtenir de l'aide.
