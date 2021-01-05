---
title: Mettre à jour la version IoT Edge sur les appareils - Azure IoT Edge | Microsoft Docs
description: Guide pratique pour mettre à jour des appareils IoT Edge afin qu’ils exécutent les dernières versions du démon de sécurité et le runtime IoT Edge
keywords: ''
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/22/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 797b5f569f081065eb950f7c10bf6449002f733b
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436978"
---
# <a name="update-the-iot-edge-security-daemon-and-runtime"></a>Mettre à jour le runtime et le démon de sécurité IoT Edge

Chaque fois que le service IoT Edge publiera de nouvelles versions, vous pourrez mettre à jour vos appareils IoT Edge pour obtenir les dernières fonctionnalités et améliorations de la sécurité. Cet article fournit des informations sur la façon de mettre à jour vos appareils IoT Edge quand une nouvelle version est disponible.

Deux composants d’un appareil IoT Edge doivent être mis à jour si vous souhaitez passer à une version plus récente. Le premier est le démon de sécurité qui s’exécute sur l’appareil et démarre les modules du runtime au démarrage de l’appareil. Le démon de sécurité ne peut être mis à jour qu’à partir de l’appareil lui-même. Le second composant est le runtime, constitué des modules de l’agent IoT Edge et du hub IoT Edge. Selon la façon dont vous structurez votre déploiement, le runtime peut être mis à jour à partir de l’appareil ou à distance.

Pour rechercher la dernière version d’Azure IoT Edge, consultez [Versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

## <a name="update-the-security-daemon"></a>Mettre à jour le démon de sécurité

Le démon de sécurité IoT Edge est un composant natif qui doit être mis à jour à l’aide du gestionnaire de package sur l’appareil IoT Edge.

Vérifiez la version du démon de sécurité qui s’exécute sur votre appareil à l’aide de la commande `iotedge version`.

### <a name="linux-devices"></a>Appareils Linux

Sur des appareils Linux x64, utilisez apt-get ou votre gestionnaire de package approprié pour mettre à jour le démon de sécurité vers la dernière version.

Procurez-vous la dernière configuration du référentiel auprès de Microsoft :

* **Ubuntu Server 16.04** :

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/16.04/multiarch/prod.list > ./microsoft-prod.list
   ```

* **Ubuntu Server 18.04** :

   ```bash
   curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
   ```

* **Raspberry Pi OS Stretch** :

   ```bash
   curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
   ```

Copiez la liste générée.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

Installez la clé publique Microsoft GPG.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

Mettez à jour apt.

   ```bash
   sudo apt-get update
   ```

Vérifiez les versions d’IoT Edge disponibles.

   ```bash
   apt list -a iotedge
   ```

Si vous souhaitez effectuer la mise à jour vers la version la plus récente du démon de sécurité, utilisez la commande suivante, qui met également à jour **libiothsm-std** vers la dernière version :

   ```bash
   sudo apt-get install iotedge
   ```

Si vous souhaitez mettre à jour vers une version spécifique du démon de sécurité, spécifiez la version dans la sortie de liste apt. Chaque fois que **iotedge** est mis à jour, il tente automatiquement de mettre à jour le package **libiothsm-std** vers la dernière version, ce qui peut entraîner un conflit de dépendance. Si vous n’allez pas utiliser la version la plus récente, veillez à cibler les deux packages avec la même version. Par exemple, la commande suivante installe une version spécifique de la version 1.0.9 :

   ```bash
   sudo apt-get install iotedge=1.0.9-1 libiothsm-std=1.0.9-1
   ```

Si la version que vous souhaitez installer n’est pas disponible par le biais de apt-get, vous pouvez utiliser curl pour cibler n’importe quelle version du référentiel [IoT Edge](https://github.com/Azure/azure-iotedge/releases). Quelle que soit la version que vous souhaitez installer, localisez les fichiers **libiothsm-STD** et **iotedge** correspondant à votre appareil. Pour chaque fichier, cliquez avec le bouton droit de la souris sur le lien du fichier, puis copiez l’adresse du lien. Utilisez l’adresse du lien pour installer les versions spécifiques de ces composants :

```bash
curl -L <libiothsm-std link> -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb
curl -L <iotedge link> -o iotedge.deb && sudo dpkg -i ./iotedge.deb
```

### <a name="windows-devices"></a>Appareils Windows

Sur les appareils Windows, utilisez le script PowerShell pour mettre à jour le démon de sécurité. Le script extrait automatiquement la dernière version du démon de sécurité.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Update-IoTEdge -ContainerOs <Windows or Linux>
```

La commande Update-IoTEdge supprime et met à jour le démon de sécurité de l’appareil, ainsi que les deux images conteneur du runtime. Le fichier config.yaml est conservé sur l’appareil, de même que les données du moteur de conteneur Moby (dans le cas de conteneurs Windows). Le fait de conserver les informations de configuration évite d’avoir à indiquer de nouveau la chaîne de connexion ou le service Device Provisioning de votre appareil lors du processus de mise à jour.

Si vous souhaitez mettre à jour vers une version spécifique du démon de sécurité, recherchez la version que vous souhaitez cibler à partir de [Versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases). Dans cette version, téléchargez le fichier **Microsoft-Azure-IoTEdge.cab**. Utilisez ensuite le paramètre `-OfflineInstallationPath` pour pointer vers l’emplacement du fichier local. Par exemple :

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Update-IoTEdge -ContainerOs <Windows or Linux> -OfflineInstallationPath <absolute path to directory>
```

>[!NOTE]
>Le paramètre `-OfflineInstallationPath` recherche un fichier nommé **Microsoft-Azure-IoTEdge.cab** dans le répertoire fourni. À partir d’IoT Edge version 1.0.9-rc4, deux fichiers. cab peuvent être utilisés, un pour les appareils AMD64 et l’autre pour ARM32. Téléchargez le fichier approprié pour votre appareil, puis renommez-le pour supprimer le suffixe d’architecture.

Pour plus d’informations sur les options de mise à jour, utilisez la commande `Get-Help Update-IoTEdge -full` ou consultez [Script PowerShell pour IoT Edge sous Windows](reference-windows-scripts.md).

## <a name="update-the-runtime-containers"></a>Mettre à jour les conteneurs du runtime

La façon dont vous mettez à jour les conteneurs de l’agent IoT Edge et du hub IoT Edge diffère selon que vous utilisez des étiquettes évolutives (par exemple, 1.0) ou des étiquettes spécifiques (par exemple, 1.0.7) dans votre déploiement.

Vérifiez la version des modules de l’agent IoT Edge et du hub IoT Edge sur votre appareil à l’aide des commandes `iotedge logs edgeAgent` ou `iotedge logs edgeHub`.

  ![Rechercher la version du conteneur dans les journaux d’activité](./media/how-to-update-iot-edge/container-version.png)

### <a name="understand-iot-edge-tags"></a>Comprendre les étiquettes IoT Edge

Les images de l’agent IoT Edge et du hub IoT Edge sont marquées avec la version IoT Edge à laquelle elles sont associées. Il existe deux façons d’utiliser des étiquettes avec les images de runtime :

* **Étiquettes évolutives** : utilisez uniquement les deux premières valeurs du numéro de version pour obtenir la dernière image qui correspond à ces chiffres. Par exemple, 1.0 est mis à jour à chaque nouvelle version pour pointer vers la dernière version 1.0.x. Si le runtime du conteneur sur votre appareil IoT Edge réextrait l’image, les modules de runtime sont mis à jour vers la dernière version. Cette approche est conseillée à des fins de développement. Les déploiements à partir du portail Azure adoptent par défaut des étiquettes évolutives.

* **Étiquettes spécifiques** : utilisez les trois valeurs du numéro de version pour définir explicitement la version de l’image. Par exemple, la version 1.0.7 ne change pas après sa publication initiale. Vous pouvez déclarer un nouveau numéro de version dans le manifeste de déploiement quand vous êtes prêt à effectuer une mise à jour. Cette approche est conseillée à des fins de production.

### <a name="update-a-rolling-tag-image"></a>Mettre à jour une image avec des étiquettes évolutives

Si vous utilisez des étiquettes évolutives dans votre déploiement (par exemple, mcr.microsoft.com/azureiotedge-hub:**1.0**), vous devez forcer le runtime du conteneur sur votre appareil à extraire la dernière version de l’image.

Supprimez la version locale de l’image de votre appareil IoT Edge. Sur les ordinateurs Windows, la désinstallation du démon de sécurité entraîne également la suppression des images du runtime, ce qui permet de sauter cette étape.

```bash
docker rmi mcr.microsoft.com/azureiotedge-hub:1.0
docker rmi mcr.microsoft.com/azureiotedge-agent:1.0
```

Vous devrez peut-être utiliser l’indicateur `-f` (forcer) pour supprimer les images.

Le service IoT Edge extrait les dernières versions des images de runtime et les démarre automatiquement sur votre appareil à nouveau.

### <a name="update-a-specific-tag-image"></a>Mettre à jour une image avec des étiquettes spécifiques

Si vous utilisez des étiquettes spécifiques dans votre déploiement (par exemple, mcr.microsoft.com/azureiotedge-hub:**1.0.8**), il vous suffit de mettre à jour l’étiquette dans votre manifeste de déploiement et d’appliquer les modifications à votre appareil.

1. Dans l’IoT Hub du portail Azure, sélectionnez votre appareil IoT Edge, puis sélectionnez **Définir les modules**.

1. Dans la section **Modules IoT Edge**, sélectionnez **Paramètres d'exécution**.

   ![Configurer les paramètres d'exécution](./media/how-to-update-iot-edge/configure-runtime.png)

1. Dans **Paramètres d’exécution**, mettez à jour la valeur de l’**image** pour **Edge Hub** avec la version souhaitée. Ne sélectionnez pas encore **Enregistrer**.

   ![Mettre à jour la version de l’image Edge Hub.](./media/how-to-update-iot-edge/runtime-settings-edgehub.png)

1. Réduisez les paramètres **Edge Hub**, ou faites défiler la liste vers le bas et mettez à jour la valeur de l’**image** pour l’**agent Edge** avec la même version souhaitée.

   ![Mettre à jour la version de l’agent Edge Hub](./media/how-to-update-iot-edge/runtime-settings-edgeagent.png)

1. Sélectionnez **Enregistrer**.

1. Sélectionnez **Vérifier + Créer**, vérifiez le déploiement, puis sélectionnez **Créer**.

## <a name="update-offline-or-to-a-specific-version"></a>Mettre à jour hors connexion ou vers une version spécifique

Si vous souhaitez mettre à jour un appareil hors connexion ou effectuer une mise à jour vers une version spécifique d’IoT Edge plutôt que vers la version la plus récente, vous pouvez le faire avec le paramètre `-OfflineInstallationPath`.

Deux composants sont utilisés pour mettre à jour un appareil IoT Edge :

* Un script PowerShell, qui contient les instructions d’installation
* Le fichier cab Microsoft Azure IoT Edge, contenant le démon de sécurité IoT Edge (iotedged), le moteur du conteneur Moby et Moby CLI

1. Pour accéder aux derniers fichiers d’installation IoT Edge ainsi qu’aux versions précédentes, voir les [publications d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

2. Recherchez la version que vous souhaitez installer, puis téléchargez les fichiers suivants de la section **Ressources** des notes de publication vers votre appareil IoT :

   * IoTEdgeSecurityDaemon.ps1
   * Microsoft-Azure-IoTEdge-amd64.cab à partir des versions 1.0.9 ou ultérieures, ou Microsoft-Azure-IoTEdge.cab à partir des versions 1.0.8 ou antérieures.

   Microsoft-Azure-IotEdge-arm32.cabcab est également disponible à partir de 1.0.9 à des fins de test uniquement. IoT Edge n’est actuellement pas pris en charge sur les appareils Windows ARM32.

   Il est important d’utiliser le script PowerShell de la même version que le fichier .cab que vous utilisez, car les fonctionnalités changent pour prendre en charge les fonctionnalités de chaque version.

3. Si le fichier .cab que vous avez téléchargé est doté d’un suffixe d’architecture, renommez le fichier uniquement **Microsoft-Azure-IoTEdge.cab**.

4. Pour mettre à jour avec des composants hors connexion, [effectuez un appel de source de type « dot source »](/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) de la copie locale du script PowerShell. Ensuite, utilisez le paramètre `-OfflineInstallationPath` dans la commande `Update-IoTEdge` et spécifiez le chemin absolu du répertoire des fichiers. Par exemple,

   ```powershell
   . <path>\IoTEdgeSecurityDaemon.ps1
   Update-IoTEdge -OfflineInstallationPath <path>
   ```

## <a name="update-to-a-release-candidate-version"></a>Mettre à jour vers une version Release Candidate

Azure IoT Edge publie régulièrement de nouvelles versions du service IoT Edge. Avant chaque version stable, il y a une ou plusieurs versions Release Candidate (RC). Les versions RC incluent toutes les fonctionnalités planifiées de la version, mais sont encore sujettes aux processus de tests et de validation. Si vous souhaitez tester très tôt une nouvelle fonctionnalité, vous pouvez installer un version RC et envoyer des commentaires via GitHub.

Les versions Release Candidate suivent la même convention de numérotation que les versions publiées, mais **-rc** et un nombre incrémentiel sont ajoutés à la fin. Vous pouvez voir les versions Release Candidate dans la même liste [Versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases) que les versions stables. Recherchez par exemple **1.0.9-rc5** et **1.0.9-rc6**, deux versions Release Candidate mises en place avant la version **1.0.9**. Vous pouvez également voir que les versions RC sont marquées avec des étiquettes **préversion**.

Les modules agent et hub d’IoT Edge ont des versions RC qui sont marquées selon la même convention. Par exemple, **mcr.microsoft.com/azureiotedge-hub:1.0.9-rc6**.

Tout comme les préversions, les versions Release Candidate ne sont pas incluses comme la version la plus récente ciblée par les programmes d’installation réguliers. Au lieu de cela, vous devez cibler manuellement les ressources de la version RC que vous souhaitez tester. Dans la plupart des cas, l’installation ou la mise à jour vers une version RC sont identiques au ciblage d’une autre version d’IoT Edge.

Reportez-vous aux sections de cet article pour savoir comment mettre à jour un appareil IoT Edge vers une version spécifique du démon de sécurité ou des modules d’exécution.

Si vous installez IoT Edge, plutôt que de mettre à niveau une installation existante, suivez les étapes décrites dans [Mettre à jour hors connexion ou vers une version spécifique](how-to-install-iot-edge.md#offline-or-specific-version-installation).

## <a name="next-steps"></a>Étapes suivantes

Déterminez les dernières [versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

Restez informé des dernières mises à jour et annonces en consultant le [blog Internet of Things](https://azure.microsoft.com/blog/topics/internet-of-things/)