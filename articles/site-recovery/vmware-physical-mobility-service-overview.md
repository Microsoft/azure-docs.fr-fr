---
title: À propos du service Mobility pour la reprise d'activité des serveurs physiques et des machines virtuelles VMware à l'aide d'Azure Site Recovery | Microsoft Docs
description: Familiarisez-vous avec l'agent du service Mobility pour la reprise d’activité des serveurs physiques et des machines virtuelles VMware sur Azure à l’aide du service Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: how-to
ms.date: 04/10/2020
ms.author: ramamill
ms.openlocfilehash: 90862a74e5fb6521a95292d50fc5cc11bd0082b5
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547654"
---
# <a name="about-the-mobility-service-for-vmware-vms-and-physical-servers"></a>À propos du service Mobilité pour serveurs physiques et machines virtuelles VMware

Lorsque vous configurez la récupération d’urgence sur des serveurs physiques et des machines virtuelles VMware avec [Azure Site Recovery](site-recovery-overview.md), vous installez le service Mobility de Site Recovery sur l’ensemble des serveurs physiques et des machines virtuelles VMware locaux. Le service Mobilité enregistre les écritures de données sur la machine et les transmet au serveur de traitement Site Recovery. Le service Mobility est installé par le logiciel de l’agent correspondant que vous pouvez déployer à l’aide des méthodes suivantes :

- [Installation Push](#push-installation) : Lorsque la protection est activée via le portail Azure, Site Recovery installe le service Mobility sur le serveur.
- Installation manuelle : vous pouvez installer le service Mobility manuellement sur chaque machine par le biais de l’[interface utilisateur (IU)](#install-the-mobility-service-using-ui) ou d’une [invite de commandes](#install-the-mobility-service-using-command-prompt).
- [Déploiement automatisé](vmware-azure-mobility-install-configuration-mgr.md) : Vous pouvez automatiser l’installation du service Mobility avec des outils de déploiement de logiciels tels que Configuration Manager.

> [!NOTE]
> Le service Mobility utilise environ 6 à 10 % de la mémoire des machines sources pour les machines virtuelles VMware ou les machines physiques.

## <a name="antivirus-on-replicated-machines"></a>Protection antivirus sur les machines répliquées

Si les machines que vous souhaitez répliquer exécutent un logiciel antivirus, excluez le dossier d'installation du service Mobility _C:\ProgramData\ASR\agent_ des opérations antivirus. Cette exclusion assure le bon fonctionnent de la réplication.

## <a name="push-installation"></a>Installation Push

L’installation Push fait partie intégrante du travail exécuté à partir du portail Azure pour [activer la réplication](vmware-azure-enable-replication.md#enable-replication). Une fois que vous avez choisi l’ensemble de machines virtuelles à protéger et pour lesquelles activer la réplication, le serveur de configuration envoie (push) le service Mobility aux serveurs, installe l’agent et procède à l’inscription de ce dernier auprès du serveur de configuration.

### <a name="prerequisites"></a>Prérequis

- Vérifiez que tous les [prérequis](vmware-azure-install-mobility-service.md) de l’installation push sont satisfaits.
- Assurez-vous que toutes les configurations de serveur répondent aux critères de la [matrice de prise en charge de la récupération d'urgence des machines virtuelles VMware et des serveurs physiques sur Azure](vmware-physical-azure-support-matrix.md).
- À partir de la version 9.36, pour SUSE Linux Enterprise Server 11 SP3, RHEL 5, CentOS 5, Debian 7, vérifiez que le programme d’installation le plus récent est [disponible sur le serveur de configuration et le serveur de processus Scale-out](#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server)

Le workflow de l’installation push est décrit dans les sections suivantes :

### <a name="mobility-service-agent-version-923-and-higher"></a>Agent du service Mobility version 9.23 ou ultérieure

Pour plus d’informations sur la version 9.23, consultez [Correctif cumulatif 35 pour Azure Site Recovery](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery).

Pendant l’installation push du service Mobility, les étapes suivantes sont effectuées :

1. L’agent fait l’objet d’un envoi (push) vers la machine source. La copie de l’agent sur la machine source peut échouer en raison de plusieurs erreurs environnementales. Consultez [nos conseils](vmware-azure-troubleshoot-push-install.md) pour résoudre les échecs d’installation push.
1. Une fois que l’agent est correctement copié sur le serveur, un vérification des prérequis intervient sur le serveur.
   - Si tous les prérequis sont satisfaits, l’installation démarre.
   - L’installation échoue si un [prérequis](vmware-physical-azure-support-matrix.md) ou plus ne sont pas satisfaits.
1. Dans le cadre de l’installation de l’agent, le fournisseur Service VSS pour Azure Site Recovery est installé. Le fournisseur VSS est utilisé pour générer des points de récupération de cohérence d’application. Si l’installation du fournisseur VSS échoue, cette étape est ignorée et l’installation de l’agent se poursuit.
1. Si l’installation de l’agent réussit, alors que celle du fournisseur VSS échoue, l’état du travail est marqué comme **Avertissement**. Ce n'a aucune incidence sur la génération de points de récupération cohérents en cas d’incident.

    - Pour générer des points de récupération de cohérence d’application, reportez-vous à [nos conseils](vmware-physical-manage-mobility-service.md#install-site-recovery-vss-provider-on-source-machine) pour terminer l’installation manuelle du fournisseur VSS Site Recovery.
    - Si vous ne voulez pas que des points de récupération de cohérence d’application soient générés, [modifiez la stratégie de réplication](vmware-azure-set-up-replication.md#create-a-policy) pour les désactiver.

### <a name="mobility-service-agent-version-922-and-below"></a>Agent du service Mobility version 9.22 ou antérieure

1. L’agent fait l’objet d’un envoi (push) vers la machine source. La copie de l’agent sur la machine source peut échouer en raison de plusieurs erreurs environnementales. Consultez [nos conseils](vmware-azure-troubleshoot-push-install.md) pour résoudre les échecs d’installation push.
1. Une fois que l’agent est correctement copié sur le serveur, un vérification des prérequis intervient sur le serveur.
   - Si tous les prérequis sont satisfaits, l’installation démarre.
   - L’installation échoue si un [prérequis](vmware-physical-azure-support-matrix.md) ou plus ne sont pas satisfaits.

1. Dans le cadre de l’installation de l’agent, le fournisseur Service VSS pour Azure Site Recovery est installé. Le fournisseur VSS est utilisé pour générer des points de récupération de cohérence d’application.
   - Si l’installation du fournisseur VSS échoue, l’installation de l’agent échoue également. Pour éviter un échec d'installation de l’agent, utilisez la [version 9.23](https://support.microsoft.com/help/4494485/update-rollup-35-for-azure-site-recovery) ou une version ultérieure afin de générer des points de récupération de cohérence d’incident et procédez à une installation manuelle du fournisseur VSS.

## <a name="install-the-mobility-service-using-ui"></a>Installer le service Mobility à l'aide de l'interface utilisateur

### <a name="prerequisites"></a>Prérequis

- Assurez-vous que toutes les configurations de serveur répondent aux critères de la [matrice de prise en charge de la récupération d'urgence des machines virtuelles VMware et des serveurs physiques sur Azure](vmware-physical-azure-support-matrix.md).
- [Recherchez le programme d’installation](#locate-installer-files) correspondant au système d’exploitation du serveur.

>[!IMPORTANT]
> N’utilisez pas la méthode d’installation de l’interface utilisateur si vous répliquez une machine virtuelle IaaS (infrastructure as a service) Azure d’une région Azure vers une autre. Utilisez l'installation avec [l’invite de commandes](#install-the-mobility-service-using-command-prompt).

1. Copiez le fichier d’installation sur la machine et exécutez-le.
1. Dans **Option d’installation** , sélectionnez **Installer le service Mobilité**.
1. Sélectionnez l’emplacement d’installation, puis **Installer**.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility1.png" alt-text="Page de l’option d’installation du service Mobility":::

1. Surveillez l’installation dans **Progression de l’installation**. Au terme de l’installation, sélectionnez **Passer à la configuration** pour inscrire le service auprès du serveur de configuration.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility3.png" alt-text="Capture d’écran montrant la progression de l’installation et le bouton Poursuivre la configuration actif une fois l’installation terminée.":::

1. Dans **Détails du serveur de configuration** , spécifiez l’adresse IP et la phrase secrète que vous avez configurées.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility4.png" alt-text="Page d’inscription du service Mobility":::

1. Sélectionnez **Inscrire** pour terminer l’inscription.

    :::image type="content" source="./media/vmware-physical-mobility-service-install-manual/mobility5.png" alt-text="Page d’inscription finale du service Mobility":::

## <a name="install-the-mobility-service-using-command-prompt"></a>Installer le service Mobility à l'aide d'une invite de commandes

### <a name="prerequisites"></a>Prérequis

- Assurez-vous que toutes les configurations de serveur répondent aux critères de la [matrice de prise en charge de la récupération d'urgence des machines virtuelles VMware et des serveurs physiques sur Azure](vmware-physical-azure-support-matrix.md).
- [Recherchez le programme d’installation](#locate-installer-files) correspondant au système d’exploitation du serveur.

### <a name="windows-machine"></a>Machine Windows

- À partir d'une invite de commandes, exécutez les commandes suivantes pour copier le programme d’installation dans un dossier local, par exemple _C:\Temp_ , sur le serveur que vous souhaitez protéger. Remplacez le nom de fichier du programme d’installation par le nom de fichier réel.

  ```cmd
  cd C:\Temp
  ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
  MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  ```

- Exécutez cette commande pour installer l'agent.

  ```cmd
  UnifiedAgent.exe /Role "MS" /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
  ```

- Exécutez ces commandes pour inscrire l’agent auprès du serveur de configuration.

  ```cmd
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <CSIP> /PassphraseFilePath <PassphraseFilePath>
  ```

#### <a name="installation-settings"></a>Paramètres d’installation

Paramètre | Détails
--- | ---
Syntaxe | `UnifiedAgent.exe /Role \<MS/MT> /InstallLocation \<Install Location> /Platform "VmWare" /Silent`
Fichiers journaux d’activité d’installation | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentInstaller.log`
`/Role` | Paramètre d’installation obligatoire. Spécifie si le service Mobilité (MS) ou Master Target(MT) doit être installé.
`/InstallLocation`| Paramètre facultatif. Spécifie l’emplacement d’installation du service Mobilité (n’importe quel dossier).
`/Platform` | Mandatory. Spécifie la plateforme sur laquelle le service Mobility est installé : <br/> **VMware** pour les machines virtuelles VMware/serveurs physiques. <br/> **Azure** pour les machines virtuelles Azure.<br/><br/> Si vous traitez des machines virtuelles Azure comme des machines physiques, spécifiez **VMware**.
`/Silent`| facultatif. Indique si le programme d’installation doit être exécuté en mode silencieux.

#### <a name="registration-settings"></a>Paramètres d’inscription

Paramètre | Détails
--- | ---
Syntaxe | `UnifiedAgentConfigurator.exe  /CSEndPoint \<CSIP> /PassphraseFilePath \<PassphraseFilePath>`
Journaux d’activité de configuration de l’agent | `%ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log`
`/CSEndPoint` | Paramètre obligatoire. `<CSIP>` spécifie l’adresse IP du serveur de configuration. Utilisez une adresse IP valide.
`/PassphraseFilePath` |  Mandatory. Emplacement de la phrase secrète. Utilisez n’importe quel chemin d’accès UNC ou local valide.

### <a name="linux-machine"></a>Machine Linux

1. À partir d'une session de terminal, copiez le programme d’installation dans un dossier local, par exemple _/tmp_ , sur le serveur que vous souhaitez protéger. Remplacez le nom de fichier du programme d’installation par le nom réel de votre distribution Linux, puis exécutez les commandes.

   ```shell
   cd /tmp ;
   tar -xvf Microsoft-ASR_UA_version_LinuxVersion_GA_date_release.tar.gz
   ```

2. Procédez à l’installation comme suit :

   ```shell
   sudo ./install -d <Install Location> -r MS -v VmWare -q
   ```

3. Une fois l’installation terminée, le service Mobility doit être inscrit auprès du serveur de configuration. Exécutez la commande suivante pour inscrire le service Mobility auprès du serveur de configuration.

   ```shell
   /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <CSIP> -P /var/passphrase.txt
   ```

#### <a name="installation-settings"></a>Paramètres d’installation

Paramètre | Détails
--- | ---
Syntaxe | `./install -d \<Install Location> -r \<MS/MT> -v VmWare -q`
`-r` | Paramètre d’installation obligatoire. Spécifie si le service Mobilité (MS) ou Master Target(MT) doit être installé.
`-d` | Paramètre facultatif. Spécifie l’emplacement d’installation du service Mobility : `/usr/local/ASR`.
`-v` | Mandatory. Spécifie la plateforme sur laquelle le service Mobility est installé. <br/> **VMware** pour les machines virtuelles VMware/serveurs physiques. <br/> **Azure** pour les machines virtuelles Azure.
`-q` | facultatif. Indique si le programme d’installation doit être exécuté en mode silencieux.

#### <a name="registration-settings"></a>Paramètres d’inscription

Paramètre | Détails
--- | ---
Syntaxe | `cd /usr/local/ASR/Vx/bin<br/><br/> UnifiedAgentConfigurator.sh -i \<CSIP> -P \<PassphraseFilePath>`
`-i` | Paramètre obligatoire. `<CSIP>` spécifie l’adresse IP du serveur de configuration. Utilisez une adresse IP valide.
`-P` |  Mandatory. Chemin d’accès complet du fichier dans lequel la phrase secrète est enregistrée. Utilisez n’importe quel dossier valide.

## <a name="azure-virtual-machine-agent"></a>Agent de machine virtuelle Azure

- **Machines virtuelles Windows**  : à partir de la version 9.7.0.0 du service Mobilité, l’ [agent de machine virtuelle Azure](../virtual-machines/extensions/features-windows.md#azure-vm-agent) est installé par le programme d’installation du service Mobilité. Ainsi, lorsque la machine bascule vers Azure, la machine virtuelle Azure remplit les conditions préalables à l’installation de l'agent pour utiliser une extension de machine virtuelle.
- **Machines virtuelles Linux**  : l’agent [WALinuxAgent](../virtual-machines/extensions/update-linux-agent.md) doit être installé manuellement sur la machine virtuelle Azure après le basculement.

## <a name="locate-installer-files"></a>Localiser les fichiers d’installation

Accédez au dossier _%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository_ sur le serveur de configuration. Vérifiez le programme d’installation dont vous avez besoin en fonction du système d’exploitation. Le tableau suivant récapitule les fichiers d’installation pour chaque système d’exploitation de serveur physique et machine virtuelle VMware. Avant de commencer, n'hésitez pas à consulter les [systèmes d’exploitation pris en charge](vmware-physical-azure-support-matrix.md#replicated-machines).

> [!NOTE]
> Les noms de fichiers utilisent la syntaxe indiquée dans le tableau suivant avec _version_ et _date_ en tant qu'espaces réservés pour les valeurs réelles. Les noms de fichiers réels doivent se présenter comme suit :
> - `Microsoft-ASR_UA_9.30.0.0_Windows_GA_22Oct2019_release.exe`
> - `Microsoft-ASR_UA_9.30.0.0_UBUNTU-16.04-64_GA_22Oct2019_release.tar.gz`

Fichier d’installation | Système d’exploitation (64 bits uniquement)
--- | ---
`Microsoft-ASR_UA_version_Windows_GA_date_release.exe` | Windows Server 2016 </br> Windows Server 2012 R2 </br> Windows Server 2012 </br> Windows Server 2008 R2 SP1
[À télécharger et à placer manuellement dans ce dossier](#rhel-5-or-centos-5-server) | Red Hat Enterprise Linux (RHEL) 5 </br> CentOS 5
`Microsoft-ASR_UA_version_RHEL6-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 6 </br> CentOS 6
`Microsoft-ASR_UA_version_RHEL7-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 7 </br> CentOS 7
`Microsoft-ASR_UA_version_RHEL8-64_GA_date_release.tar.gz` | Red Hat Enterprise Linux (RHEL) 8 </br> CentOS 8
`Microsoft-ASR_UA_version_SLES12-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 12 SP1 </br> Inclut SP2 et SP3.
[À télécharger et à placer manuellement dans ce dossier](#suse-11-sp3-server) | SUSE Linux Enterprise Server 11 SP3
`Microsoft-ASR_UA_version_SLES11-SP4-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 11 SP4
`Microsoft-ASR_UA_version_SLES15-64_GA_date_release.tar.gz` | SUSE Linux Enterprise Server 15 
`Microsoft-ASR_UA_version_OL6-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 6.4 </br> Oracle Enterprise Linux 6.5
`Microsoft-ASR_UA_version_OL7-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 7 
`Microsoft-ASR_UA_version_OL8-64_GA_date_release.tar.gz` | Oracle Enterprise Linux 8
`Microsoft-ASR_UA_version_UBUNTU-14.04-64_GA_date_release.tar.gz` | Ubuntu Linux 14.04
`Microsoft-ASR_UA_version_UBUNTU-16.04-64_GA_date_release.tar.gz` | Serveur Ubuntu Linux 16.04 LTS
`Microsoft-ASR_UA_version_UBUNTU-18.04-64_GA_date_release.tar.gz` | Serveur Ubuntu Linux 18.04 LTS
`Microsoft-ASR_UA_version_UBUNTU-20.04-64_GA_date_release.tar.gz` | Serveur Ubuntu Linux 20.04 LTS
[À télécharger et à placer manuellement dans ce dossier](#debian-7-server) | Debian 7
`Microsoft-ASR_UA_version_DEBIAN8-64_GA_date_release.tar.gz` | Debian 8
`Microsoft-ASR_UA_version_DEBIAN9-64_GA_date_release.tar.gz` | Debian 9

## <a name="download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server"></a>Télécharger la dernière version du programme d’installation de l’agent de mobilité pour le serveur SUSE 11 SP3, RHEL 5, Debian 7

### <a name="suse-11-sp3-server"></a>Serveur SUSE 11 SP3

**condition préalable à la mise à jour ou à la protection des ordinateurs SUSE Linux Enterprise Server 11 SP3** à partir de la version 9.36 :

1. Vérifiez que le dernier programme d’installation de l’agent de mobilité est téléchargé à partir du Centre de téléchargement Microsoft et placé dans le référentiel du programme d’installation push sur le serveur de configuration et tous les serveurs de processus de montée en puissance parallèle
2. [Téléchargez](site-recovery-whats-new.md) le dernier programme d’installation de l’agent SUSE Linux Enterprise Server 11 SP3. La dernière version de l’agent de mobilité est la [9.37](https://support.microsoft.com/help/4582666/)
3. Accédez au serveur de configuration, copiez le programme d’installation de l’agent SUSE Linux Enterprise Server 11 SP3 dans le chemin d'accès INSTALL_DIR\home\svsystems\pushinstallsvc\repository
1. Après la copie du programme d’installation le plus récent, redémarrez le service InMage PushInstall. 
1. À présent, accédez aux serveurs de traitement de processus avec montée en charge associés, répétez les étapes 3 et 4.
1. **Par exemple** , si le chemin d’installation est C:\Program Files (x86) \Microsoft Azure Site Recovery, les répertoires mentionnés ci-dessus seront :
    1. C:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc\repository

### <a name="rhel-5-or-centos-5-server"></a>Serveur RHEL 5 ou CentOS 5

**condition préalable à la mise à jour ou à la protection des ordinateurs RHEL 5** à partir de la version 9.36 :

1. Vérifiez que le dernier programme d’installation de l’agent de mobilité est téléchargé à partir du Centre de téléchargement Microsoft et placé dans le référentiel du programme d’installation push sur le serveur de configuration et tous les serveurs de processus de montée en puissance parallèle
2. [Téléchargez](site-recovery-whats-new.md) le programme d'installation de l’agent RHEL 5 ou CentOS 5 le plus récent. La dernière version de l’agent de mobilité est la [9.37](https://support.microsoft.com/help/4582666/)
3. Accédez au serveur de configuration, copiez le programme d'installation de l’agent RHEL 5 ou CentOS 5 dans le chemin d’accès INSTALL_DIR\home\svsystems\pushinstallsvc\repository
1. Après la copie du programme d’installation le plus récent, redémarrez le service InMage PushInstall. 
1. À présent, accédez aux serveurs de traitement de processus avec montée en charge associés, répétez les étapes 3 et 4.
1. **Par exemple** , si le chemin d’installation est C:\Program Files (x86) \Microsoft Azure Site Recovery, les répertoires mentionnés ci-dessus seront :
    1. C:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc\repository

## <a name="debian-7-server"></a>Serveur Debian 7

**condition préalable à la mise à jour ou à la protection des ordinateurs Debian 7** à partir de la version 9.36 :

1. Vérifiez que le dernier programme d’installation de l’agent de mobilité est téléchargé à partir du Centre de téléchargement Microsoft et placé dans le référentiel du programme d’installation push sur le serveur de configuration et tous les serveurs de processus de montée en puissance parallèle
2. [Téléchargez](site-recovery-whats-new.md) le programme d’installation de l’agent Debian 7 le plus récent. La dernière version de l’agent de mobilité est la [9.37](https://support.microsoft.com/help/4582666/)
3. Accédez au serveur de configuration, copiez le programme d’installation de l’agent Debian 7 dans le chemin d'accès INSTALL_DIR\home\svsystems\pushinstallsvc\repository
1. Après la copie du programme d’installation le plus récent, redémarrez le service InMage PushInstall. 
1. À présent, accédez aux serveurs de traitement de processus avec montée en charge associés, répétez les étapes 3 et 4.
1. **Par exemple** , si le chemin d’installation est C:\Program Files (x86) \Microsoft Azure Site Recovery, les répertoires mentionnés ci-dessus seront :
    1. C:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc\repository

## <a name="next-steps"></a>Étapes suivantes

[Configurer l’installation Push du service Mobilité](vmware-azure-install-mobility-service.md)
