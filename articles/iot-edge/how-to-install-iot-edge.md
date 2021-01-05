---
title: Installer Azure IoT Edge | Microsoft Docs
description: Instructions d’installation d’Azure IoT Edge sur des appareils Windows ou Linux
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: kgremban
ms.openlocfilehash: a7794bcdfa4f82698fdc5875bc94dcf52b70166e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185094"
---
# <a name="install-or-uninstall-the-azure-iot-edge-runtime"></a>Installer ou désinstaller le runtime Azure IoT Edge

Le runtime Azure IoT Edge est ce qui transforme un appareil en appareil IoT Edge. Le runtime peut être déployé sur un appareil de petite taille comme un Raspberry Pi ou de grande taille comme un serveur industriel. Une fois qu’un appareil est configuré avec le runtime IoT Edge, vous pouvez commencer à déployer une logique métier sur celui-ci à partir du cloud. Pour en savoir plus, consultez [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md).

La configuration d’un appareil IoT Edge présente deux étapes. La première étape consiste à installer le runtime et ses dépendances, ce qui est abordé dans cet article. La deuxième étape consiste à connecter l’appareil à son identité dans le cloud et à configurer l’authentification avec IoT Hub. Ces étapes sont décrites dans les articles suivants.

Cet article répertorie les étapes à suivre pour installer le runtime Azure IoT Edge sur des appareils Linux ou Windows. Pour les appareils Windows, vous avez en outre la possibilité d’utiliser des conteneurs Linux ou des conteneurs Windows. Actuellement, les conteneurs Windows sur Windows sont recommandés pour les scénarios de production. Les conteneurs Linux sur Windows sont utiles pour les scénarios de développement et de test, en particulier si vous développez sur un PC Windows afin de déployer sur des appareils Linux.

## <a name="prerequisites"></a>Prérequis

Pour obtenir les informations les plus récentes sur les systèmes d’exploitation actuellement pris en charge pour les scénarios de production, consultez [Systèmes pris en charge par Azure IoT Edge](support.md#operating-systems).

# <a name="linux"></a>[Linux](#tab/linux)

Vous devez avoir un appareil Linux x64, ARM32 ou ARM64. Microsoft fournit des packages d’installation pour les systèmes d’exploitation Ubuntu Server 16.04, Ubuntu Server 18.04 et Raspbian Stretch.

>[!NOTE]
>La prise en charge des appareils ARM64 est en [préversion publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Préparez votre appareil pour accéder aux packages d’installation de Microsoft.

1. Installez la configuration du référentiel qui correspond au système d’exploitation de votre appareil.

   * **Ubuntu Server 16.04** :

     ```bash
     curl https://packages.microsoft.com/config/ubuntu/16.04/multiarch/prod.list > ./microsoft-prod.list
     ```

   * **Ubuntu Server 18.04** :

     ```bash
     curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
     ```

   * **Raspberry Pi OS Stretch** :

     ```bash
     curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
     ```

2. Copiez la liste générée dans le répertoire sources.list.d.

   ```bash
   sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   ```

3. Installez la clé publique Microsoft GPG.

   ```bash
   curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   ```

# <a name="windows"></a>[Windows](#tab/windows)

### <a name="windows-version"></a>Version de Windows

IoT Edge avec des conteneurs Windows requiert Windows version 1809/Build 17762, qui est la [build prise en charge à long terme de Windows](/windows/release-information/)la plus récente. Pour les scénarios de développement et de test, tout SKU (Pro, Entreprise, Serveur, etc.) qui prend en charge la fonctionnalité des conteneurs fonctionnera. Toutefois, veillez à consulter la liste des systèmes pris en charge avant de passer en production.

IoT Edge avec des conteneurs Linux peut s’exécuter sur n’importe quelle version de Windows qui respecte la [configuration requise pour Docker Desktop](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install).

### <a name="container-engine-requirements"></a>Configuration requise du moteur de conteneur

Azure IoT Edge s’appuie sur un moteur de conteneur [compatible avec OCI](https://www.opencontainers.org/). Assurez-vous que votre appareil peut prendre en charge les conteneurs.

Si vous installez IoT Edge sur une machine virtuelle, activez la virtualisation imbriquée et allouez au moins 2 Go de mémoire. Pour Hyper-V, la virtualisation imbriquée est activée par défaut sur les machines virtuelles de deuxième génération. Pour VMware, un bouton bascule permet d’activer cette fonctionnalité sur votre machine virtuelle.

Si vous installez IoT Edge sur un appareil IoT Core, utilisez la commande suivante dans une [session PowerShell à distance](/windows/iot-core/connect-your-device/powershell) pour vérifier que les conteneurs Windows sont pris en charge sur votre appareil :

```powershell
Get-Service vmcompute
```

---

Les packages logiciels Azure IoT Edge sont soumis aux termes du contrat de licence situés dans chaque package (`usr/share/doc/{package-name}` ou dans le répertoire `LICENSE`). Lisez les termes du contrat de licence avant d’utiliser un package. Le fait d’installer et d’utiliser un package revient à accepter ces termes. Si vous n’acceptez pas les termes du contrat de licence, n’utilisez pas le package en question.

## <a name="install-a-container-engine"></a>Installer un moteur de conteneur

Azure IoT Edge s’appuie sur un runtime de conteneur compatible avec OCI. Pour les scénarios de production, nous vous recommandons d’utiliser le moteur Moby. Le moteur Moby est le seul moteur de conteneur officiellement pris en charge avec Azure IoT Edge. Les images conteneur Docker CE/EE sont compatibles avec le runtime Moby.

# <a name="linux"></a>[Linux](#tab/linux)

Mettez à jour les listes de packages sur votre appareil.

   ```bash
   sudo apt-get update
   ```

Installez le moteur Moby.

   ```bash
   sudo apt-get install moby-engine
   ```

Si des erreurs surviennent lors de l’installation du moteur de conteneur Moby, vérifiez la compatibilité de votre noyau Linux avec Moby. Certains fabricants d’appareils embarqués livrent des images d’appareils qui contiennent des noyaux Linux personnalisés sans les fonctionnalités nécessaires à la compatibilité du moteur de conteneur. Exécutez la commande suivante, qui utilise le [script check-config](https://github.com/moby/moby/blob/master/contrib/check-config.sh) fourni par Moby pour vérifier la configuration de votre noyau :

   ```bash
   curl -sSL https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh -o check-config.sh
   chmod +x check-config.sh
   ./check-config.sh
   ```

Dans la sortie du script, vérifiez que tous les éléments sous `Generally Necessary` et `Network Drivers` sont activés. Si vous ne disposez pas de certaines fonctionnalités, activez-les en régénérant votre noyau à partir des sources et en sélectionnant les modules associés à inclure dans le noyau .config approprié. De même, si vous utilisez un générateur de configuration du noyau comme `defconfig` ou `menuconfig`, trouvez et activez les fonctionnalités respectives et recompilez votre noyau en conséquence. Une fois que vous avez déployé votre nouveau noyau modifié, exécutez à nouveau le script check-config pour vérifier que toutes les fonctionnalités requises ont été activées avec succès.

# <a name="windows"></a>[Windows](#tab/windows)

Pour les scénarios de production, utilisez le moteur Moby inclus dans le script d’installation. Aucune étape supplémentaire n’est nécessaire pour installer le moteur.

Pour IoT Edge avec des conteneurs Linux, vous devez fournir votre propre runtime de conteneur. Installez [Docker Desktop](https://docs.docker.com/docker-for-windows/install/) sur votre appareil et configurez-le pour qu’il [utilise des conteneurs Linux](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) avant de continuer.

---

## <a name="install-the-iot-edge-security-daemon"></a>Installer le démon de sécurité IoT Edge

Le démon de sécurité IoT Edge fournit et gère les standards de sécurité sur l’appareil IoT Edge. Le démon se lance à chaque démarrage et amorce l’appareil en démarrant le reste du runtime IoT Edge.

Les étapes de cette section représentent le processus classique d’installation de la version la plus récente sur un appareil disposant d’une connexion Internet. Si vous devez installer une version spécifique, comme une préversion, ou si vous devez installer en mode hors connexion, suivez les étapes d’[installation d’une version hors connexion ou spécifique](#offline-or-specific-version-installation) dans la section suivante.

# <a name="linux"></a>[Linux](#tab/linux)

Mettez à jour les listes de packages sur votre appareil.

   ```bash
   sudo apt-get update
   ```

Vérifiez les versions d’IoT Edge disponibles.

   ```bash
   apt list -a iotedge
   ```

Si vous souhaitez installer la version la plus récente du démon de sécurité, utilisez la commande suivante, qui installe également la dernière version du package **libiothsm-std** :

   ```bash
   sudo apt-get install iotedge
   ```

Si vous souhaitez installer une version spécifique du démon de sécurité, spécifiez la version dans la sortie de liste apt. Spécifiez également la même version pour le package **libiothsm-std**, qui, dans le cas contraire, installe sa dernière version. Par exemple, la commande suivante installe la version la plus récente de la mise en production 1.0.8 :

   ```bash
   sudo apt-get install iotedge=1.0.8* libiothsm-std=1.0.8*
   ```

Si la version que vous souhaitez installer ne figure pas dans la liste, suivez les étapes d’[installation d’une version hors connexion ou spécifique](#offline-or-specific-version-installation) dans la section suivante. Cette section vous montre comment cibler une version antérieure du démon de sécurité IoT Edge ou des versions Release Candidate.

# <a name="windows"></a>[Windows](#tab/windows)

>[!TIP]
>Pour les appareils IoT Core, nous vous recommandons d’exécuter les commandes d’installation à l’aide d’une session PowerShell à distance. Pour plus d’informations, consultez [Utilisation de PowerShell pour Windows IoT](/windows/iot-core/connect-your-device/powershell).

1. Exécutez PowerShell ISE en tant qu’administrateur.

   Utilisez une session AMD64 de PowerShell, et non PowerShell (x86). Si vous ne savez pas quel type de session vous utilisez, exécutez la commande suivante :

   ```powershell
   (Get-Process -Id $PID).StartInfo.EnvironmentVariables["PROCESSOR_ARCHITECTURE"]
   ```

2. Exécutez la commande [Deploy-IoTEdge](reference-windows-scripts.md#deploy-iotedge) qui effectue les tâches suivantes :

   * Vérifie que votre ordinateur Windows est sous une version prise en charge.
   * Active la fonctionnalité des conteneurs.
   * Télécharge le moteur Moby et le runtime IoT Edge.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge
   ```

   Par défaut, la commande `Deploy-IoTEdge` utilise des conteneurs Windows. Si vous souhaitez utiliser des conteneurs Linux, ajoutez le paramètre `ContainerOs` :

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge -ContainerOs Linux
   ```

3. À ce stade, les appareils IoT Core peuvent redémarrer automatiquement. Les appareils Windows 10 ou Windows Server peuvent vous inviter à redémarrer. Si c’est le cas, redémarrez votre appareil maintenant.

Quand vous installez IoT Edge sur un appareil, vous pouvez utiliser des paramètres supplémentaires pour modifier le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Pointer le programme d’installation vers un répertoire local pour une installation hors connexion.

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur Windows](reference-windows-scripts.md).

---

Maintenant que le moteur de conteneur et le runtime IoT Edge sont installés sur votre appareil, vous êtes prêt pour l’étape suivante, qui consiste à inscrire votre appareil auprès d’IoT Hub et à configurer l’appareil avec ses informations d’identité et d’authentification cloud.

Choisissez l’article suivant en fonction du type d’authentification que vous souhaitez utiliser :

* [L’authentification par clé symétrique](how-to-manual-provision-symmetric-key.md) est plus rapide pour commencer.
* [L’authentification par certificat X.509](how-to-manual-provision-x509.md) est plus sécurisée pour les scénarios de production.

## <a name="offline-or-specific-version-installation"></a>Installation d’une version hors connexion ou spécifique

Les étapes de cette section concernent les scénarios non couverts par les étapes d’installation standard. Cela peut inclure :

* Installer IoT Edge en mode hors connexion
* Installer une version Release candidate
* Sur Windows, installer une version autre que la version la plus récente

# <a name="linux"></a>[Linux](#tab/linux)

Utilisez les étapes de cette section si vous souhaitez installer une version spécifique du runtime Azure IoT Edge, qui n’est pas disponible via `apt-get install`. La liste des packages Microsoft contient uniquement un ensemble limité de versions récentes et leurs sous-versions. Par conséquent, ces étapes sont destinées à toute personne souhaitant installer une version antérieure ou une version Release Candidate.

À l’aide de commandes curl, vous pouvez cibler les fichiers de composants directement à partir du référentiel GitHub IoT Edge.

<!-- TODO: Offline installation? -->

1. Accédez aux [versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases)et recherchez celle que vous souhaitez cibler.

2. Développez la section **Ressources** pour cette version.

3. Chaque version devrait avoir de nouveaux fichiers pour le démon de sécurité IoT Edge et le hsmlib. Utilisez les commandes suivantes pour mettre à jour ces composants.

   1. Trouvez le fichier **libiothsm-std** qui correspond à l’architecture de votre périphérique IoT Edge. Cliquez avec le bouton droit de la souris sur le lien de fichier et copiez l’adresse du lien.

   2. Utilisez le lien copié dans la commande suivante pour installer cette version du hsmlib :

      ```bash
      curl -L <libiothsm-std link> -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb
      ```

   3. Trouvez le fichier **iotedge** qui correspond à l’architecture de votre périphérique IoT Edge. Cliquez avec le bouton droit de la souris sur le lien de fichier et copiez l’adresse du lien.

   4. Utilisez le lien copié dans la commande suivante pour installer cette version du démon de sécurité de l’IoT Edge.

      ```bash
      curl -L <iotedge link> -o iotedge.deb && sudo dpkg -i ./iotedge.deb
      ```

# <a name="windows"></a>[Windows](#tab/windows)

Lors de l’installation, trois fichiers sont téléchargés :

* Un script PowerShell, qui contient les instructions d’installation
* Le fichier cab Microsoft Azure IoT Edge, contenant le démon de sécurité IoT Edge (iotedged), le moteur du conteneur Moby et Moby CLI
* Programme d’installation du package Visual C++ Redistribuable (Runtime VC)

Si votre appareil est hors connexion pendant l’installation, ou si vous souhaitez installer une version spécifique d’IoT Edge, vous pouvez télécharger à l’avance ces fichiers sur l’appareil. Quand il est temps d’effectuer l’installation, pointez le script d’installation sur le répertoire contenant les fichiers téléchargés. Le programme d’installation commence par vérifier le répertoire, puis télécharge uniquement les composants qui ne s’y trouvent pas. Si tous les fichiers sont disponibles hors connexion, vous pouvez procéder à l’installation sans connexion Internet.

1. Pour accéder aux derniers fichiers d’installation IoT Edge ainsi qu’aux versions précédentes, voir les [publications d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

2. Recherchez la version que vous souhaitez installer, puis téléchargez les fichiers suivants de la section **Ressources** des notes de publication vers votre appareil IoT :

   * IoTEdgeSecurityDaemon.ps1
   * Microsoft-Azure-IoTEdge-amd64.cab à partir des versions 1.0.9 ou ultérieures, ou Microsoft-Azure-IoTEdge.cab à partir des versions 1.0.8 ou antérieures.

   Microsoft-Azure-IotEdge-arm32.cabcab est également disponible à partir de 1.0.9 à des fins de test uniquement. IoT Edge n’est actuellement pas pris en charge sur les appareils Windows ARM32.

   Il est important d’utiliser le script PowerShell de la même version que le fichier .cab que vous utilisez, car les fonctionnalités changent pour prendre en charge les fonctionnalités de chaque version.

3. Si le fichier .cab que vous avez téléchargé est doté d’un suffixe d’architecture, renommez le fichier uniquement **Microsoft-Azure-IoTEdge.cab**.

4. Éventuellement, téléchargez un programme d’installation pour Visual C++ Redistributable. Par exemple, le script PowerShell utilise cette version : [vc_redist.x64. exe](https://download.microsoft.com/download/0/6/4/064F84EA-D1DB-4EAA-9A5C-CC2F0FF6A638/vc_redist.x64.exe). Enregistrez le programme d’installation dans le même dossier que les fichiers IoT Edge sur votre appareil.

5. Pour installer avec des composants hors connexion, [effectuez un appel de source de type « dot source »](/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) de la copie locale du script PowerShell. 

6. Exécutez la commande [Deploy-IoTEdge](reference-windows-scripts.md#deploy-iotedge) avec le paramètre `-OfflineInstallationPath`. Indiquez le chemin d’accès absolu au répertoire de fichiers. Par exemple,

   ```powershell
   . <path>\IoTEdgeSecurityDaemon.ps1
   Deploy-IoTEdge -OfflineInstallationPath <path>
   ```

   La commande de déploiement utilise les composants trouvés dans le répertoire de fichiers local fourni. S’il manque le fichier .cab ou le programme d'installation de Visual C++, elle tente de les télécharger.

---

Maintenant que le moteur de conteneur et le runtime IoT Edge sont installés sur votre appareil, vous êtes prêt à passer à l’étape suivante, qui consiste à [authentifier un appareil IoT Edge dans IoT Hub](how-to-manual-provision-symmetric-key.md).

## <a name="uninstall-iot-edge"></a>Désinstaller IoT Edge

Si vous souhaitez supprimer l’installation d’IoT Edge de votre appareil, utilisez les commandes suivantes.

# <a name="linux"></a>[Linux](#tab/linux)

Supprimez le runtime IoT Edge.

```bash
sudo apt-get remove --purge iotedge
```

Lors de la suppression du runtime IoT Edge, tous les conteneurs qu’il a créés sont arrêtés, mais continuent d’exister sur votre appareil. Affichez tous les conteneurs pour voir ceux qui restent.

```bash
sudo docker ps -a
```

Supprimez les conteneurs de votre appareil, dont les deux conteneurs de runtime.

```bash
sudo docker rm -f <container name>
```

Enfin, supprimez le runtime du conteneur de votre appareil.

```bash
sudo apt-get remove --purge moby-cli
sudo apt-get remove --purge moby-engine
```

# <a name="windows"></a>[Windows](#tab/windows)

Si vous souhaitez supprimer l’installation d’IoT Edge de votre appareil Windows, utilisez la commande [Uninstall-IoTEdge](reference-windows-scripts.md#uninstall-iotedge) à partir d’une fenêtre PowerShell d’administration. Cette commande supprime le runtime IoT Edge, ainsi que votre configuration existante et les données du moteur Moby.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
Uninstall-IoTEdge
```

La commande `Uninstall-IoTEdge` ne fonctionne pas sur Windows IoT Core. Pour supprimer IoT Edge, vous devez redéployer votre image Windows IoT Core.

Pour plus d’informations sur les options de désinstallation, utilisez la commande `Get-Help Uninstall-IoTEdge -full`.

---

## <a name="next-steps"></a>Étapes suivantes

Après avoir installé le runtime IoT Edge, configurez votre appareil pour qu’il se connecte à IoT Hub. Les articles suivants expliquent comment inscrire un nouvel appareil dans le cloud, puis lui fournir ses informations d’identité et d’authentification.

Choisissez l’article suivant en fonction du type d’authentification que vous souhaitez utiliser :

* **Clé symétrique** : IoT Hub et l’appareil IoT Edge possèdent tous deux une copie d’une clé sécurisée. Lorsque l’appareil se connecte à IoT Hub, il vérifie que les clés correspondent. Cette méthode d’authentification est plus rapide pour commencer, mais moins sécurisée que l’autre.

  [Configurer un appareil Azure IoT Edge avec une authentification par clé symétrique](how-to-manual-provision-symmetric-key.md)

* **Certificat autosigné X.509** : L’appareil IoT Edge est doté de certificats d’identité X.509, et IoT Hub reçoit l’empreinte des certificats. Lorsque l’appareil se connecte à IoT Hub, il compare le certificat par rapport à son empreinte. Cette méthode d’authentification étant plus sécurisée, elle est recommandée dans les scénarios de production.

  [Configurer un appareil Azure IoT Edge avec une authentification par certificat X.509](how-to-manual-provision-x509.md)