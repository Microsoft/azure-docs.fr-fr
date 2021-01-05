---
title: Configurer un serveur de processus avec scale-out pendant la récupération d’urgence des machines virtuelles VMware et serveurs physiques avec Azure Site Recovery | Microsoft Docs
description: Cet article décrit comment configurer un serveur de processus avec scale-out en puissance pendant la récupération d’urgence des machines virtuelles VMware et serveurs physiques.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/23/2019
ms.author: ramamill
ms.openlocfilehash: 40f912122e6ffb9cccbd32a747f6f0d46fd6c330
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019132"
---
# <a name="scale-with-additional-process-servers"></a>Mettre à l’échelle avec d’autres serveurs de processus

Par défaut, lorsque vous répliquez des machines virtuelles VMware ou des serveurs physiques sur Azure avec [Site Recovery](site-recovery-overview.md), un serveur de processus est installé sur le serveur de configuration et permet de coordonner le transfert des données entre Site Recovery et votre infrastructure locale. Pour augmenter la capacité et effectuer un scale-out de votre déploiement de réplication, vous pouvez ajouter des serveurs de processus autonomes supplémentaires. Cet article décrit comment configurer un serveur de processus avec montée en puissance.

## <a name="before-you-start"></a>Avant de commencer

### <a name="capacity-planning"></a>planification de la capacité

Assurez-vous d’avoir effectué la [planification de la capacité](site-recovery-plan-capacity-vmware.md) pour la réplication VMware. Cela vous aide à déterminer comment et quand déployer des serveurs de traitement supplémentaires.

À partir de la version 9.24, des conseils sont ajoutés lors de la sélection du serveur de processus pour les nouvelles réplications. En fonction de certains critères, les états suivants seront attribués au serveur de processus : Sain, Avertissement et Critique. Pour comprendre les différents scénarios qui peuvent influencer l’état du serveur de processus, consultez l’article dédié aux [alertes du serveur de processus](vmware-physical-azure-monitor-process-server.md#process-server-alerts).

> [!NOTE]
> L’utilisation d’un composant de serveur de processus cloné n’est pas prise en charge. Suivez les étapes décrites dans cet article pour chaque mise à l’échelle PS.

### <a name="sizing-requirements"></a>Configuration requise pour le dimensionnement 

Vérifiez la configuration requise pour le dimensionnement, résumée dans le tableau. En général, si vous devez faire évoluer votre déploiement au-delà de 200 machines source ou que votre taux d’évaluation quotidien total dépasse 2 To, vous avez besoin de serveurs de traitement supplémentaires pour gérer le volume de trafic.

| **Serveur de traitement supplémentaire** | **Taille du disque cache** | **Taux de modification des données** | **Machines protégées** |
| --- | --- | --- | --- |
|4 processeurs virtuels (2 sockets * 2 cœurs \@ 2,5 GHz), 8 Go de mémoire |300 Go |250 Go ou moins |Répliquez 85 machines ou moins. |
|8 processeurs virtuels (2 sockets * 4 cœurs \@ 2,5 GHz), 12 Go de mémoire |600 Go |250 Go à 1 To |Répliquez entre 85 et 150 machines. |
|12 processeurs virtuels (2 sockets * 6 cœurs \@ 2,5 GHz), 24 Go de mémoire |1 To |1 To à 2 To |Répliquez entre 150 et 225 machines. |

Où chaque machine source protégée est configurée avec 3 disques de 100 Go chacune.

### <a name="prerequisites"></a>Conditions préalables requises

La configuration requise pour le serveur de traitement supplémentaire est résumée dans le tableau suivant.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

## <a name="download-installation-file"></a>Télécharger le fichier d’installation

Téléchargez le fichier d’installation pour le serveur de processus comme suit :

1. Connectez-vous au portail Azure et accédez à votre coffre Recovery Services.
2. Ouvrez **Infrastructure Site Recovery** > **VMware et machines physiques** > **Serveurs de configuration** (sous Pour VMware et machines physiques).
3. Sélectionnez le serveur de configuration pour explorer les détails du serveur. Cliquez ensuite sur **+ Serveur de processus**.
4. Dans **Ajouter un serveur de processus** >  **Indique où vous souhaitez déployer votre serveur de processus**, sélectionnez **Déployer un serveur de traitement de montée en puissance parallèle local**.

   ![Page Ajouter des serveurs](./media/vmware-azure-set-up-process-server-scale/add-process-server.png)
1. Cliquez sur **Download the Microsoft Azure Site Recovery Unified Setup** (Télécharger le programme d’installation unifiée de Microsoft Azure Site Recovery). Cette option télécharge la dernière version du fichier d’installation.

   > [!WARNING]
   > La version d’installation du serveur de processus doit être identique ou antérieure à la version du serveur de configuration en cours d’exécution. Un moyen simple d’assurer la compatibilité de la version consiste à utiliser le même programme d’installation que celui récemment utilisé pour installer ou mettre à jour votre serveur de configuration.

## <a name="install-from-the-ui"></a>Installer à partir de l’interface utilisateur

Procédez à l’installation comme suit. Après avoir configuré le serveur, vous pouvez migrer les ordinateurs source pour les utiliser.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-add-process-server.md)]


## <a name="install-from-the-command-line"></a>Installer à partir de la ligne de commande

Installez en exécutant la commande suivante :

```
UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMware/NonVMware>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]
```

Avec les paramètres de ligne de commande suivants :

[!INCLUDE [site-recovery-unified-setup-parameters](../../includes/site-recovery-unified-installer-command-parameters.md)]

Par exemple :

```
MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted
cd C:\Temp\Extracted
UNIFIEDSETUP.EXE /AcceptThirdpartyEULA /servermode "PS" /InstallLocation "D:\" /EnvType "VMware" /CSIP "10.150.24.119" /PassphraseFilePath "C:\Users\Administrator\Desktop\Passphrase.txt" /DataTransferSecurePort 443
```
### <a name="create-a-proxy-settings-file"></a>Créer un fichier de paramètres de proxy

Si vous devez configurer un proxy, le paramètre ProxySettingsFilePath prend un fichier en tant qu’entrée. Vous pouvez créer le fichier comme suit, puis le transmettre comme paramètre ProxySettingsFilePath d’entrée.

```
* [ProxySettings]
* ProxyAuthentication = "Yes/No"
* Proxy IP = "IP Address"
* ProxyPort = "Port"
* ProxyUserName="UserName"
* ProxyPassword="Password"
```

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la [gestion des paramètres de serveur de processus](vmware-azure-manage-process-server.md)
