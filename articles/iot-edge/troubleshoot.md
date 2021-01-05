---
title: Résoudre les problèmes d’Azure IoT Edge | Microsoft Docs
description: Cet article vous permettra d’acquérir des compétences de diagnostic standard concernant Azure IoT Edge, telles que la récupération des journaux d’activité et de l’état des composants.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/12/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 035cf5be4471cad7ac11eb8ce9a8a0ecb13a68da
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462373"
---
# <a name="troubleshoot-your-iot-edge-device"></a>Résoudre les problèmes de votre appareil IoT Edge

Si vous rencontrez des problèmes lors de l’exécution d’Azure IoT Edge dans votre environnement, consultez cet article qui vous guidera pour la résolution des problèmes et les diagnostics.

## <a name="run-the-check-command"></a>Exécuter la commande « check »

La première étape dans la résolution des problèmes relatifs à IoT Edge consiste à utiliser la commande `check` qui exécute une collection de tests de configuration et de connectivité à la recherche de problèmes courants. La commande `check` est disponible dans la [version 1.0.7](https://github.com/Azure/azure-iotedge/releases/tag/1.0.7) et supérieure.

>[!NOTE]
>L’outil de dépannage ne peut pas exécuter de vérifications de connectivité si l’appareil IoT Edge se trouve derrière un serveur proxy.

Vous pouvez exécuter la commande `check` comme suit, ou inclure l’indicateur `--help` pour afficher une liste complète des options :

Sur Linux :

```bash
sudo iotedge check
```

Sur Windows :

```powershell
iotedge check
```

L’outil de résolution des problèmes exécute de nombreuses vérifications qui sont triées dans les trois catégories suivantes :

* La *Vérification de configuration* examine les détails pouvant empêcher les appareils IoT Edge de se connecter au cloud, y compris les problèmes avec *config.yaml* et le moteur du conteneur.
* La *Vérification de connexion* vérifie que le runtime IoT Edge peut accéder aux ports sur l’appareil hôte et que tous les composants IoT Edge peuvent se connecter à IoT Hub. Cet ensemble de vérifications renvoie des erreurs si l’appareil IoT Edge se trouve derrière un proxy.
* La *Vérification de disponibilité de la production* recherche les meilleures pratiques de production recommandées, comme l’état des certificats d’autorité de l’appareil et la configuration du fichier journal du module.

L’outil de vérification IoT Edge utilise un conteneur pour exécuter ses diagnostics. L’image conteneur, `mcr.microsoft.com/azureiotedge-diagnostics:latest`, est disponible via [Microsoft Container Registry](https://github.com/microsoft/containerregistry). S’il vous faut exécuter une vérification portant sur un appareil sans accès direct à Internet, vos appareils doivent pouvoir accéder à l’image conteneur.

Pour plus d’informations sur chacune des vérifications de diagnostic exécutées par cet outil, notamment ce qu’il faut faire si vous obtenez une erreur ou un avertissement, consultez [Vérifications de dépannage IoT Edge](https://github.com/Azure/iotedge/blob/master/doc/troubleshoot-checks.md).

## <a name="gather-debug-information-with-support-bundle-command"></a>Collecter les informations de débogage avec la commande « support-bundle »

Pour collecter des journaux à partir d’un appareil IoT Edge, la méthode la plus pratique consiste à utiliser la commande `support-bundle`. Par défaut, cette commande collecte les journaux des modules, du gestionnaire de sécurité IoT Edge et du moteur de conteneur, la sortie JSON de la commande `iotedge check` et d’autres informations de débogage utiles. Elle les compresse dans un fichier unique pour faciliter le partage. La commande `support-bundle` est disponible dans la [version 1.0.9](https://github.com/Azure/azure-iotedge/releases/tag/1.0.9) et supérieure.

Exécutez la commande `support-bundle` avec l’indicateur `--since` pour spécifier la période pour laquelle vous souhaitez récupérer les journaux. Par exemple `6h` obtiendra les journaux des six dernières heures, `6d` des six derniers jours, `6m` des six dernières minutes, et ainsi de suite. Incluez l’indicateur `--help` pour voir la liste complète des options.

Sur Linux :

```bash
sudo iotedge support-bundle --since 6h
```

Sur Windows :

```powershell
iotedge support-bundle --since 6h
```

Vous pouvez également utiliser un appel de [méthode directe](how-to-retrieve-iot-edge-logs.md#upload-support-bundle-diagnostics) à destination de votre appareil pour charger la sortie de la commande support-bundle dans Stockage Blob Azure.

> [!WARNING]
> La sortie de la commande `support-bundle` peut contenir des noms d’hôte, d’appareil et de module, des informations journalisées par vos modules, etc. Soyez conscient de cela si vous partagez la sortie dans un forum public.

## <a name="check-your-iot-edge-version"></a>Vérifier votre version d’IoT Edge

Si vous exécutez une version antérieure d’IoT Edge, une mise à niveau peut résoudre votre problème. L’outil `iotedge check` vérifie que le démon de sécurité IoT Edge est la version la plus récente, mais ne vérifie pas les versions des modules de hub et d’agent IoT Edge. Pour vérifier la version des modules de runtime sur votre appareil, utilisez les commandes `iotedge logs edgeAgent` et `iotedge logs edgeHub`. Le numéro de version est déclaré dans les journaux au démarrage du module.

Pour obtenir des instructions sur la mise à jour de votre appareil, consultez [Mettre à jour le runtime et le démon de sécurité IoT Edge](how-to-update-iot-edge.md).

## <a name="verify-the-installation-of-iot-edge-on-your-devices"></a>Vérifier l’installation d’IoT Edge sur vos appareils

Vous pouvez vérifier l’installation d’IoT Edge sur vos appareils en [surveillant le jumeau de module edgeAgent](./how-to-monitor-module-twins.md).

Pour obtenir le dernier jumeau de module, exécutez la commande suivante depuis [Azure Cloud Shell](https://shell.azure.com/) :

   ```azurecli-interactive
   az iot hub module-twin show --device-id <edge_device_id> --module-id $edgeAgent --hub-name <iot_hub_name>
   ```

Cette commande renvoie toutes les [propriétés edgeAgent signalées](./module-edgeagent-edgehub.md). Vous trouverez ci-dessous quelques propriétés utiles pour surveiller l’état de l’appareil :

* état du runtime
* heure de début du runtime
* heure de la dernière sortie du runtime
* nombre de redémarrages du runtime

## <a name="check-the-status-of-the-iot-edge-security-manager-and-its-logs"></a>Vérifier l’état du gestionnaire de sécurité IoT Edge et de ses journaux

Le [gestionnaire de sécurité IoT Edge](iot-edge-security-manager.md) est responsable d’opérations telles que l’initialisation du système IoT Edge au démarrage et l’approvisionnement des appareils. Si IoT Edge ne démarre pas, les journaux du gestionnaire de sécurité peuvent fournir des informations utiles.

Sur Linux :

* Consultez l’état du gestionnaire de sécurité IoT Edge :

   ```bash
   sudo systemctl status iotedge
   ```

* Consultez les journaux du gestionnaire de sécurité IoT Edge :

    ```bash
    sudo journalctl -u iotedge -f
    ```

* Consultez les journaux plus détaillés du gestionnaire de sécurité IoT Edge :

  * Modifiez les paramètres du démon IoT Edge :

      ```bash
      sudo systemctl edit iotedge.service
      ```

  * Mettez à jour les lignes suivantes :

      ```bash
      [Service]
      Environment=IOTEDGE_LOG=edgelet=debug
      ```

  * Redémarrez le démon de sécurité IoT Edge :

      ```bash
      sudo systemctl cat iotedge.service
      sudo systemctl daemon-reload
      sudo systemctl restart iotedge
      ```

Sur Windows :

* Consultez l’état du gestionnaire de sécurité IoT Edge :

   ```powershell
   Get-Service iotedge
   ```

* Consultez les journaux du gestionnaire de sécurité IoT Edge :

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
   ```

* Affichez uniquement les 5 dernières minutes des journaux du gestionnaire de sécurité IoT Edge :

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog -StartTime ([datetime]::Now.AddMinutes(-5))
   ```

* Consultez les journaux plus détaillés du gestionnaire de sécurité IoT Edge :

  * Ajouter une variable d’environnement au niveau du système

      ```powershell
      [Environment]::SetEnvironmentVariable("IOTEDGE_LOG", "debug", [EnvironmentVariableTarget]::Machine)
      ```

  * Redémarrez le démon de sécurité IoT Edge :

      ```powershell
      Restart-Service iotedge
      ```

### <a name="if-the-iot-edge-security-manager-is-not-running-verify-your-yaml-configuration-file"></a>Si le gestionnaire de sécurité IoT Edge n’est pas en cours d’exécution, vérifiez votre fichier de configuration yaml.

> [!WARNING]
> Les fichiers YAML ne peuvent pas contenir de tabulations en guise de mise en retrait. Utilisez 2 espaces à la place. Les éléments de niveau supérieur ne doivent pas avoir d’espaces de début.

Sur Linux :

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

Sur Windows :

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

### <a name="restart-the-iot-edge-security-manager"></a>Redémarrer le gestionnaire de sécurité IoT Edge

Si le problème persiste, vous pouvez essayer de redémarrer le gestionnaire de sécurité IoT Edge.

Sur Linux :

   ```cmd
   sudo systemctl restart iotedge
   ```

Sur Windows :

   ```powershell
   Stop-Service iotedge -NoWait
   sleep 5
   Start-Service iotedge
   ```

## <a name="check-container-logs-for-issues"></a>Vérifier si les journaux d’activité des conteneurs indiquent des problèmes

Une fois que le démon de sécurité IoT Edge s’exécute, vérifiez si les journaux des conteneurs signalent des problèmes. Commencez par les conteneurs déployés, puis examinez les conteneurs qui composent le runtime IoT Edge : Edge Agent et Edge Hub. En général, les journaux d’activité de l’agent IoT Edge fournissent des informations sur le cycle de vie de chaque conteneur. Les journaux d’activité du hub IoT Edge fournissent des informations sur la messagerie et le routage.

```cmd
iotedge logs <container name>
```

Vous pouvez également utiliser un appel de [méthode directe](how-to-retrieve-iot-edge-logs.md#upload-module-logs) à destination d’un modèle de votre appareil pour charger les journaux de ce module dans Stockage Blob Azure.

## <a name="view-the-messages-going-through-the-iot-edge-hub"></a>Afficher les messages acheminés via hub IoT Edge

Vous pouvez afficher les messages acheminés via le hub IoT Edge et recueillir des insights à partir des journaux d’activité détaillés issus des conteneurs du runtime. Pour activer les journaux d’activité détaillés sur ces conteneurs, définissez `RuntimeLogLevel` dans votre fichier de configuration yaml. Pour ouvrir le fichier :

Sur Linux :

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

Sur Windows :

   ```cmd
   notepad C:\ProgramData\iotedge\config.yaml
   ```

Par défaut, l’élément `agent` se présente comme dans l’exemple suivant :

   ```yaml
   agent:
     name: edgeAgent
     type: docker
     env: {}
     config:
       image: mcr.microsoft.com/azureiotedge-agent:1.0
       auth: {}
   ```

Remplacez `env: {}` par :

   ```yaml
   env:
     RuntimeLogLevel: debug
   ```

   > [!WARNING]
   > Les fichiers YAML ne peuvent pas contenir de tabulations en guise de mise en retrait. Utilisez 2 espaces à la place. Les éléments de niveau supérieur ne peuvent pas avoir d’espace blanc de début.

Enregistrez le fichier et redémarrez le gestionnaire de sécurité IoT Edge.

Vous pouvez également vérifier les messages envoyés entre IoT Hub et les appareils IoT Edge. Affichez ces messages à l’aide de l’extension [Azure IoT Hub pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit). Pour plus d’informations, consultez [Handy tool when you develop with Azure IoT](https://blogs.msdn.microsoft.com/iotdev/2017/09/01/handy-tool-when-you-develop-with-azure-iot/).

## <a name="restart-containers"></a>Redémarrer les conteneurs

Après avoir examiné les journaux d’activité et les messages pour obtenir plus d’informations, vous pouvez essayer de redémarrer les conteneurs :

```cmd
iotedge restart <container name>
```

Redémarrez les conteneurs du runtime IoT Edge :

```cmd
iotedge restart edgeAgent && iotedge restart edgeHub
```

## <a name="check-your-firewall-and-port-configuration-rules"></a>Vérifier vos règles de configuration de pare-feu et de port

Azure IoT Edge permet la communication entre un serveur local et le cloud Azure au moyen des protocoles IoT Hub pris en charge (voir [Choisir un protocole de communication](../iot-hub/iot-hub-devguide-protocols.md)). Pour renforcer la sécurité, les canaux de communication entre Azure IoT Edge et Azure IoT Hub sont toujours configurés pour être sortants. Cette configuration est basée sur le [modèle de communication assistée par des services](/archive/blogs/clemensv/service-assisted-communication-for-connected-devices), ce qui réduit la surface d’attaque qu’une entité malveillante pourrait explorer. Les communications entrantes sont uniquement requises pour des scénarios spécifiques où Azure IoT Hub a besoin d’envoyer (push) des messages à l’appareil Azure IoT Edge. Les messages cloud-à-appareil sont protégés à l’aide de canaux TLS sécurisés, protection qui peut être renforcée à l’aide de certificats X.509 et de modules d’appareil TPM. Le Gestionnaire de sécurité Azure IoT Edge régit la façon dont cette communication peut être établie (voir [Gestionnaire de sécurité IoT Edge](../iot-edge/iot-edge-security-manager.md)).

Bien que IoT Edge assure une configuration améliorée pour la sécurisation du runtime Azure IoT Edge et des modules déployés, il dépend toujours de la configuration du réseau et de l’ordinateur sous-jacents. Il est donc indispensable de vérifier que les règles de pare-feu et de réseau sont bien configurées pour assurer la sécurisation de la communication Edge vers Cloud. Vous pouvez utiliser le tableau suivant pour configurer les règles de pare-feu pour les serveurs sous-jacents hébergeant le runtime Azure IoT Edge :

|Protocol|Port|Entrant|Sortant|Assistance|
|--|--|--|--|--|
|MQTT|8883|BLOQUÉ (par défaut)|BLOQUÉ (par défaut)|<ul> <li>Configurez Sortant sur Ouvert lorsque vous utilisez MQTT comme protocole de communication.<li>1883 pour MQTT n’est pas pris en charge par IoT Edge. <li>Les connexions entrantes doivent être bloquées.</ul>|
|AMQP|5671|BLOQUÉ (par défaut)|OUVERT (par défaut)|<ul> <li>Protocole de communication par défaut pour IoT Edge. <li> Doit être configuré sur Ouvert si Azure IoT Edge n’est pas configuré pour les autres protocoles pris en charge ou si AMQP est le protocole de communication souhaité.<li>5672 pour AMQP n’est pas pris en charge par IoT Edge.<li>Bloquez ce port quand Azure IoT Edge utilise un autre protocole IoT Hub pris en charge.<li>Les connexions entrantes doivent être bloquées.</ul></ul>|
|HTTPS|443|BLOQUÉ (par défaut)|OUVERT (par défaut)|<ul> <li>Configurez la connexion sortante pour être Ouverte sur le port 443 pour le provisionnement d’IoT Edge. Cette configuration est requise en cas d’utilisation de scripts manuels ou du service Azure IoT Device Provisioning. <li>La connexion entrante ne peut être Ouverte que dans certaines situations uniquement : <ul> <li>  Si vous disposez d’une passerelle transparente avec des appareils de nœud terminal pouvant envoyer des requêtes de méthode. Dans ce cas, le port 443 n’a pas besoin d’être ouvert aux réseaux externes pour se connecter à IoTHub ou fournir des services IoTHub via Azure IoT Edge. La règle entrante peut donc être limitée uniquement à l’ouverture du trafic entrant en provenance du réseau interne. <li> Pour les scénarios Client vers Appareil (C2D).</ul><li>80 pour HTTP n’est pas pris en charge par IoT Edge.<li>Si les protocoles non-HTTP (par exemple, AMQP ou MQTT) ne peuvent pas être configurés dans l’entreprise, les messages peuvent être envoyés sur WebSockets. Dans ce cas, le port 443 sera utilisé pour la communication WebSocket.</ul>|

## <a name="next-steps"></a>Étapes suivantes

Vous pensez que vous avez trouvé un bogue dans la plateforme IoT Edge ? [Soumettez un problème](https://github.com/Azure/iotedge/issues) afin que nous puissions poursuivre les améliorations.

Si vous avez d'autres questions, créez une [Support request](https://portal.azure.com/#create/Microsoft.Support) pour obtenir de l'aide.