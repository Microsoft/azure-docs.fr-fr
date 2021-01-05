---
title: Automatiser le service Mobility pour la récupération d’urgence de l’installation dans Azure Site Recovery
description: Comment installer automatiquement le service Mobility pour la récupération d’urgence d’un serveur VMware/physique avec Azure Site Recovery.
author: Rajeswari-Mamilla
ms.topic: how-to
ms.date: 2/5/2020
ms.author: ramamill
ms.openlocfilehash: 2159ab8c2639f0f87fd53e8559dad518a3daa663
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92544815"
---
# <a name="automate-mobility-service-installation"></a>Automatiser l’installation du service mobilité

Cet article explique comment automatiser l’installation et les mises à jour de l’agent du service Mobility dans [Azure Site Recovery](site-recovery-overview.md).

Lorsque vous déployez Site Recovery pour la récupération d’urgence de machines virtuelles VMware et de serveurs physiques locaux sur Azure, vous installez l’agent du service Mobility sur chaque ordinateur que vous souhaitez répliquer. Le service Mobility enregistre les écritures de données sur la machine et les transmet au serveur de traitement Site Recovery à des fins de réplication. Vous pouvez déployer le service Mobility de différentes manières :

- **Installation Push**  : Laissez Site Recovery installer l’agent du service Mobility lorsque vous activez la réplication pour un ordinateur dans le Portail Azure.
- **Installation manuelle**  : Installez le service Mobility manuellement sur chaque ordinateur. [En savoir plus](vmware-physical-mobility-service-overview.md) sur l’installation Push et manuelle.
- **Déploiement automatisé**  : Automatisez l’installation avec des outils de déploiement de logiciels tels que Microsoft Endpoint Configuration Manager ou des outils tiers tels que JetPatch.

L’installation et la mise à jour automatisées fournissent une solution dans les cas suivants :

- Votre organisation n’autorise pas l’installation Push sur des serveurs protégés.
- La stratégie de votre entreprise exige que les mots de passe soient modifiés régulièrement. Vous devez spécifier un mot de passe pour l’installation Push.
- Votre stratégie de sécurité n’autorise pas l’ajout d’exceptions de pare-feu pour des ordinateurs spécifiques.
- Vous agissez en tant que fournisseur de services d’hébergement et ne souhaitez pas fournir les informations d’identification de l’ordinateur client nécessaires à l’installation Push avec Site Recovery.
- Vous devez mettre à l’échelle les installations de l’agent sur un grand nombre de serveurs simultanément.
- Vous souhaitez planifier des installations et des mises à niveau pendant les fenêtres de maintenance planifiée.

## <a name="prerequisites"></a>Prérequis

Pour automatiser l'installation, vous avez besoin des éléments suivants :

- Une solution d’installation de logiciels déployée telle que [Configuration Manager](/configmgr/) ou [JetPatch](https://jetpatch.com/microsoft-azure-site-recovery/).
- Les conditions préalables au déploiement en place dans [Azure](tutorial-prepare-azure.md) et [en local](vmware-azure-tutorial-prepare-on-premises.md) pour la récupération d’urgence de VMware, ou pour la récupération d’urgence du [serveur physique](physical-azure-disaster-recovery.md). Passez en revue les [conditions de prise en charge](vmware-physical-azure-support-matrix.md) en matière de récupération d’urgence.

## <a name="prepare-for-automated-deployment"></a>Préparer le déploiement automatisé

Le tableau suivant récapitule les outils et les processus pour l’automatisation du déploiement du service Mobility.

**Outil** | **Détails** | **Instructions**
--- | --- | ---
**Gestionnaire de configuration** | 1. Vérifiez que les [conditions préalables](#prerequisites) mentionnées ci-dessus sont remplies. <br/><br/> 2. Déployez la récupération d’urgence en configurant l’environnement source, y compris le téléchargement d’un fichier OVA pour déployer le serveur de configuration Site Recovery en tant que machine virtuelle VMware à l’aide d’un modèle OVF.<br/><br/> 3. Vous inscrivez le serveur de configuration auprès du service Site Recovery, vous configurez l’environnement Azure cible et vous configurez une stratégie de réplication.<br/><br/> 4. Pour le déploiement du service Mobility automatisé, vous créez un partage réseau contenant les fichiers de phrase secrète du serveur de configuration et les fichiers d’installation du service Mobility.<br/><br/> 5. Vous créez un package Configuration Manager contenant l’installation ou les mises à jour, et vous préparez le déploiement du service Mobility.<br/><br/> 6. Vous pouvez ensuite activer la réplication vers Azure pour les ordinateurs sur lesquels le service Mobility est installé. | [Automatiser avec Configuration Manager](#automate-with-configuration-manager)
**JetPatch** | 1. Vérifiez que les [conditions préalables](#prerequisites) mentionnées ci-dessus sont remplies. <br/><br/> 2. Déployez la récupération d’urgence en configurant l’environnement source, y compris le téléchargement et le déploiement de JetPatch Agent Manager pour Azure Site Recovery dans votre environnement Site Recovery, à l’aide d’un modèle OVF.<br/><br/> 3. Vous inscrivez le serveur de configuration auprès de Site Recovery, vous configurez l’environnement Azure cible et vous configurez une stratégie de réplication.<br/><br/> 4. Pour un déploiement automatisé, initialisez et terminez la configuration de JetPatch Agent Manager.<br/><br/> 5. Dans JetPatch, vous pouvez créer une stratégie Site Recovery pour automatiser le déploiement et la mise à niveau de l’agent du service Mobility. <br/><br/> 6. Vous pouvez ensuite activer la réplication vers Azure pour les ordinateurs sur lesquels le service Mobility est installé. | [Automatiser avec JetPatch Agent Manager](https://jetpatch.com/microsoft-azure-site-recovery-deployment-guide/)<br/><br/> [Résoudre les problèmes d’installation de l’agent dans JetPatch](https://kc.jetpatch.com/hc/articles/360035981812)

## <a name="automate-with-configuration-manager"></a>Automatiser avec Configuration Manager

### <a name="prepare-the-installation-files"></a>Préparer les fichiers d’installation

1. Assurez-vous que les conditions préalables sont remplies.
1. Créez un partage de fichiers réseau sécurisé (partage SMB) qui est accessible par l’ordinateur exécutant le serveur de configuration.
1. Dans Configuration Manager, [classez les serveurs](/sccm/core/clients/manage/collections/automatically-categorize-devices-into-collections) sur lesquels vous souhaitez installer ou mettre à jour le service Mobility. Une collection doit contenir tous les serveurs Windows, l’autre, tous les serveurs Linux.
1. Sur le partage réseau, créez un dossier :

   - Pour une installation sur des ordinateurs Windows, créez un dossier nommé _MobSvcWindows_ .
   - Pour une installation sur des ordinateurs Windows, créez un dossier nommé _MobSvcWindows_ .

1. Connectez-vous à la machine du serveur de configuration.
1. Sur la machine du serveur de configuration, ouvrez une invite de commandes d’administration.
1. Pour générer le fichier de phrase secrète, exécutez la commande suivante :

    ```Console
    cd %ProgramData%\ASR\home\svsystems\bin
    genpassphrase.exe -v > MobSvc.passphrase
    ```

1. Copiez le fichier _MobSvc.passphrase_ dans le dossier Windows et le dossier Linux.
1. Pour accéder au dossier contenant les fichiers d’installation, exécutez la commande suivante :

    ```Console
    cd %ProgramData%\ASR\home\svsystems\pushinstallsvc\repository
    ```

1. Copiez ces fichiers d’installation sur le partage réseau :

   - Pour Windows, copiez _Microsoft-ASR_UA_version_Windows_GA_date_Release.exe_ dans _MobSvcWindows_ .
   - Pour Linux, copiez les fichiers suivants dans _MobSvcLinux_  :
     - _Microsoft-ASR_UARHEL6-64release.tar.gz_
     - _Microsoft-ASR_UARHEL7-64release.tar.gz_
     - _Microsoft-ASR_UASLES11-SP3-64release.tar.gz_
     - _Microsoft-ASR_UASLES11-SP4-64release.tar.gz_
     - _Microsoft-ASR_UAOL6-64release.tar.gz_
     - _Microsoft-ASR_UAUBUNTU-14.04-64release.tar.gz_

1. Comme décrit dans les procédures suivantes, copiez le code dans les dossiers Windows ou Linux. Nous supposons que :

   - L’adresse IP du serveur de configuration est `192.168.3.121`.
   - Le partage de fichiers réseau sécurisé est `\\ContosoSecureFS\MobilityServiceInstallers`.

### <a name="copy-code-to-the-windows-folder"></a>Copier le code dans le dossier Windows

Copiez le code suivant :

- Enregistrez le code dans le dossier _MobSvcWindows_ en tant que _install.bat_ .
- Remplacez les espaces réservés `[CSIP]` dans le script ci-dessous par les valeurs réelles de l’adresse IP de votre serveur de configuration.
- Le script prend en charge les nouvelles installations de l’agent du service Mobility et les mises à jour sur les agents déjà installés.

```DOS
Time /t >> C:\Temp\logfile.log
REM ==================================================
REM ==== Clean up the folders ========================
RMDIR /S /q %temp%\MobSvc
MKDIR %Temp%\MobSvc
MKDIR C:\Temp
REM ==================================================

REM ==== Copy new files ==============================
COPY M*.* %Temp%\MobSvc
CD %Temp%\MobSvc
REN Micro*.exe MobSvcInstaller.exe
REM ==================================================

REM ==== Extract the installer =======================
MobSvcInstaller.exe /q /x:%Temp%\MobSvc\Extracted
REM ==== Wait 10s for extraction to complete =========
TIMEOUT /t 10
REM =================================================

REM ==== Perform installation =======================
REM =================================================

CD %Temp%\MobSvc\Extracted
whoami >> C:\Temp\logfile.log
SET PRODKEY=HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
REG QUERY %PRODKEY%\{275197FC-14FD-4560-A5EB-38217F80CBD1}
IF NOT %ERRORLEVEL% EQU 0 (
    echo "Product is not installed. Goto INSTALL." >> C:\Temp\logfile.log
    GOTO :INSTALL
) ELSE (
    echo "Product is installed." >> C:\Temp\logfile.log

    echo "Checking for Post-install action status." >> C:\Temp\logfile.log
    GOTO :POSTINSTALLCHECK
)

:POSTINSTALLCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "PostInstallActions" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Post-install actions succeeded. Checking for Configuration status." >> C:\Temp\logfile.log
        GOTO :CONFIGURATIONCHECK
    ) ELSE (
        echo "Post-install actions didn't succeed. Goto INSTALL." >> C:\Temp\logfile.log
        GOTO :INSTALL
    )

:CONFIGURATIONCHECK
    REG QUERY "HKLM\SOFTWARE\Wow6432Node\InMage Systems\Installed Products\5" /v "AgentConfigurationStatus" | Find "Succeeded"
    If %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded. Goto UPGRADE." >> C:\Temp\logfile.log
        GOTO :UPGRADE
    ) ELSE (
        echo "Configuration didn't succeed. Goto CONFIGURE." >> C:\Temp\logfile.log
        GOTO :CONFIGURE
    )


:INSTALL
    echo "Perform installation." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Role MS /InstallLocation "C:\Program Files (x86)\Microsoft Azure Site Recovery" /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Installation has succeeded." >> C:\Temp\logfile.log
        (GOTO :CONFIGURE)
    ) ELSE (
        echo "Installation has failed." >> C:\Temp\logfile.log
        GOTO :ENDSCRIPT
    )

:CONFIGURE
    echo "Perform configuration." >> C:\Temp\logfile.log
    cd "C:\Program Files (x86)\Microsoft Azure Site Recovery\agent"
    UnifiedAgentConfigurator.exe  /CSEndPoint "[CSIP]" /PassphraseFilePath %Temp%\MobSvc\MobSvc.passphrase
    IF %ERRORLEVEL% EQU 0 (
        echo "Configuration has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Configuration has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:UPGRADE
    echo "Perform upgrade." >> C:\Temp\logfile.log
    UnifiedAgent.exe /Platform "VmWare" /Silent
    IF %ERRORLEVEL% EQU 0 (
        echo "Upgrade has succeeded." >> C:\Temp\logfile.log
    ) ELSE (
        echo "Upgrade has failed." >> C:\Temp\logfile.log
    )
    GOTO :ENDSCRIPT

:ENDSCRIPT
    echo "End of script." >> C:\Temp\logfile.log
```

### <a name="copy-code-to-the-linux-folder"></a>Copier le code dans le dossier Linux

Copiez le code suivant :

- Enregistrez le code dans le dossier _MobSvcLinux_ en tant que _install_linux.sh_ .
- Remplacez les espaces réservés `[CSIP]` dans le script ci-dessous par les valeurs réelles de l’adresse IP de votre serveur de configuration.
- Le script prend en charge les nouvelles installations de l’agent du service Mobility et les mises à jour sur les agents déjà installés.

```Bash
#!/usr/bin/env bash

rm -rf /tmp/MobSvc
mkdir -p /tmp/MobSvc
INSTALL_DIR='/usr/local/ASR'
VX_VERSION_FILE='/usr/local/.vx_version'

echo "=============================" >> /tmp/MobSvc/sccm.log
echo `date` >> /tmp/MobSvc/sccm.log
echo "=============================" >> /tmp/MobSvc/sccm.log

if [ -f /etc/oracle-release ] && [ -f /etc/redhat-release ]; then
    if grep -q 'Oracle Linux Server release 6.*' /etc/oracle-release; then
        if uname -a | grep -q x86_64; then
            OS="OL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *OL6*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/redhat-release ]; then
    if grep -q 'Red Hat Enterprise Linux Server release 6.* (Santiago)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 6.* (Final)' /etc/redhat-release || \
        grep -q 'CentOS release 6.* (Final)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL6-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL6*.tar.gz /tmp/MobSvc
        fi
    elif grep -q 'Red Hat Enterprise Linux Server release 7.* (Maipo)' /etc/redhat-release || \
        grep -q 'CentOS Linux release 7.* (Core)' /etc/redhat-release; then
        if uname -a | grep -q x86_64; then
            OS="RHEL7-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *RHEL7*.tar.gz /tmp/MobSvc
                fi
    fi
elif [ -f /etc/SuSE-release ] && grep -q 'VERSION = 11' /etc/SuSE-release; then
    if grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 3' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP3-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP3*.tar.gz /tmp/MobSvc
        fi
    elif grep -q "SUSE Linux Enterprise Server 11" /etc/SuSE-release && grep -q 'PATCHLEVEL = 4' /etc/SuSE-release; then
        if uname -a | grep -q x86_64; then
            OS="SLES11-SP4-64"
            echo $OS >> /tmp/MobSvc/sccm.log
            cp *SLES11-SP4*.tar.gz /tmp/MobSvc
        fi
    fi
elif [ -f /etc/lsb-release ] ; then
    if grep -q 'DISTRIB_RELEASE=14.04' /etc/lsb-release ; then
       if uname -a | grep -q x86_64; then
           OS="UBUNTU-14.04-64"
           echo $OS >> /tmp/MobSvc/sccm.log
           cp *UBUNTU-14*.tar.gz /tmp/MobSvc
       fi
    fi
else
    exit 1
fi

if [ -z "$OS" ]; then
    exit 1
fi

Install()
{
    echo "Perform Installation." >> /tmp/MobSvc/sccm.log
    ./install -q -d ${INSTALL_DIR} -r MS -v VmWare
    RET_VAL=$?
    echo "Installation Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Installation has succeeded. Proceed to configuration." >> /tmp/MobSvc/sccm.log
        Configure
    else
        echo "Installation has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Configure()
{
    echo "Perform configuration." >> /tmp/MobSvc/sccm.log
    ${INSTALL_DIR}/Vx/bin/UnifiedAgentConfigurator.sh -i [CSIP] -P MobSvc.passphrase
    RET_VAL=$?
    echo "Configuration Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Configuration has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Configuration has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

Upgrade()
{
    echo "Perform Upgrade." >> /tmp/MobSvc/sccm.log
    ./install -q -v VmWare
    RET_VAL=$?
    echo "Upgrade Returncode: $RET_VAL" >> /tmp/MobSvc/sccm.log
    if [ $RET_VAL -eq 0 ]; then
        echo "Upgrade has succeeded." >> /tmp/MobSvc/sccm.log
    else
        echo "Upgrade has failed." >> /tmp/MobSvc/sccm.log
        exit $RET_VAL
    fi
}

cp MobSvc.passphrase /tmp/MobSvc
cd /tmp/MobSvc

tar -zxvf *.tar.gz

if [ -e ${VX_VERSION_FILE} ]; then
    echo "${VX_VERSION_FILE} exists. Checking for configuration status." >> /tmp/MobSvc/sccm.log
    agent_configuration=$(grep ^AGENT_CONFIGURATION_STATUS "${VX_VERSION_FILE}" | cut -d"=" -f2 | tr -d " ")
    echo "agent_configuration=$agent_configuration" >> /tmp/MobSvc/sccm.log
     if [ "$agent_configuration" == "Succeeded" ]; then
        echo "Agent is already configured. Proceed to Upgrade." >> /tmp/MobSvc/sccm.log
        Upgrade
    else
        echo "Agent is not configured. Proceed to Configure." >> /tmp/MobSvc/sccm.log
        Configure
    fi
else
    Install
fi

cd /tmp

```

### <a name="create-a-package"></a>Créer un package

1. Connectez-vous à la console Configuration Manager et accédez à **Bibliothèque de logiciels** > **Gestion des applications** > **Packages** .
1. Cliquez avec le bouton droit sur **Packages** > **Créer un package** .
1. Fournissez les détails du package, notamment un nom, une description, le fabricant, la langue et la version.
1. Sélectionnez **Ce package contient des fichiers sources** .
1. Cliquez sur **Parcourir** et sélectionnez le partage réseau et le dossier contenant le programme d’installation approprié ( _MobSvcWindows_ ou _MobSvcLinux_ ). Ensuite, sélectionnez **Suivant** .

   ![Capture d’écran de l’assistant Création d’un package et d’un programme](./media/vmware-azure-mobility-install-configuration-mgr/create_sccm_package.png)

1. Dans la page **Choisissez le type de programme que vous voulez créer** , sélectionnez **Programme standard** > **Suivant** .

   ![Capture d’écran de l’assistant Création d’un package et d’un programme qui affiche l’option Programme standard.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-standard-program.png)

1. Dans la page **Spécifier les informations concernant ce programme standard** , spécifiez les valeurs suivantes :

    **Paramètre** | **Valeur Windows** | **Valeur Linux**
    --- | --- | ---
    **Nom** | Installer le service Mobilité de Microsoft Azure (Windows) | Installer le service Mobility de Microsoft Azure (Linux)
    **Ligne de commande** | install.bat | ./install_linux.sh
    **Le programme peut s’exécuter** | Qu’un utilisateur ait ouvert une session ou non | Qu’un utilisateur ait ouvert une session ou non
    **Autres paramètres** | Utiliser le paramètre par défaut | Utiliser le paramètre par défaut

   ![Capture d’écran montrant les informations que vous pouvez spécifier pour le programme standard.](./media/vmware-azure-mobility-install-configuration-mgr/sccm-program-properties.png)

1. Dans **Spécifier les conditions requises pour ce programme standard** , exécutez les tâches suivantes :

   - Pour les ordinateurs Windows, sélectionnez **Ce programme ne peut s’exécuter que sur des plateformes spécifiées** . Ensuite, sélectionnez les [systèmes d’exploitation Windows pris en charge](vmware-physical-azure-support-matrix.md#replicated-machines), puis **Suivant** .
   - Pour les ordinateurs Linux, sélectionnez **Ce programme peut être exécuté sur n'importe quelle plateforme** . Sélectionnez ensuite **Suivant** .

1. Terminez l’Assistant.

### <a name="deploy-the-package"></a>Déploiement du package

1. Dans la console Configuration Manager, cliquez avec le bouton droit sur le package et sélectionnez **Distribuer du contenu** .

   ![Capture d’écran de la console Configuration Manager](./media/vmware-azure-mobility-install-configuration-mgr/sccm_distribute.png)

1. Sélectionnez les points de distribution vers lesquels les packages doivent être copiés. [Plus d’informations](/sccm/core/servers/deploy/configure/install-and-configure-distribution-points)
1. Terminez l'Assistant. Le package démarre ensuite la réplication sur les points de distribution spécifiés.
1. Une fois la distribution du package terminée, cliquez avec le bouton droit sur le package > **Déployer** .

   ![Capture d’écran de la console Configuration Manager montrant l’option de menu Déployer.](./media/vmware-azure-mobility-install-configuration-mgr/sccm_deploy.png)

1. Sélectionnez la collection d’appareils Windows ou Linux que vous avez créé précédemment.
1. Dans la page **Spécifier la destination du contenu** , sélectionnez **Points de distribution** .
1. Dans la page **Spécifier un paramètre pour contrôler la manière dont ce logiciel est déployé** , définissez **Objectif** sur **Requis** .

   ![Capture d’écran de l’assistant Déploiement du logiciel](./media/vmware-azure-mobility-install-configuration-mgr/sccm-deploy-select-purpose.png)

1. Dans **Spécifier la planification de ce déploiement** , configurez une planification. [Plus d’informations](/sccm/apps/deploy-use/deploy-applications#bkmk_deploy-sched)

   - Le service Mobility est installé conformément à la planification que vous spécifiez.
   - Pour éviter les redémarrages inutiles, planifiez l’installation du package pendant votre fenêtre de maintenance mensuelle ou votre fenêtre de mises à jour logicielles.

1. Dans la page **Points de distribution** , configurez les paramètres et terminez l’Assistant.
1. Surveillez la progression du déploiement dans la console Configuration Manager. Accédez à **Supervision** > **Déploiements** >  _\<your package name\>_ .

### <a name="uninstall-the-mobility-service"></a>Désinstaller le service Mobility

Vous pouvez créer des packages Configuration Manager pour désinstaller le service Mobility. Par exemple, le script suivant désinstalle le service Mobility :

```DOS
Time /t >> C:\logfile.log
REM ==================================================
REM ==== Check if Mob Svc is already installed =======
REM ==== If not installed no operation required ========
REM ==== Else run uninstall command =====================
REM ==== {275197FC-14FD-4560-A5EB-38217F80CBD1} is ====
REM ==== guid for Mob Svc Installer ====================
whoami >> C:\logfile.log
NET START | FIND "InMage Scout Application Service"
IF  %ERRORLEVEL% EQU 1 (GOTO :INSTALL) ELSE GOTO :UNINSTALL
:NOOPERATION
                echo "No Operation Required." >> c:\logfile.log
                GOTO :ENDSCRIPT
:UNINSTALL
                echo "Uninstall" >> C:\logfile.log
                MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
:ENDSCRIPT
```

## <a name="next-steps"></a>Étapes suivantes

[Activez la réplication](vmware-azure-enable-replication.md) pour les machines virtuelles.
