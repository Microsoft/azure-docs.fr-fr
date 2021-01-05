---
title: Gérer un serveur de processus pour la récupération d’urgence des machines virtuelles VMware/serveurs physiques dans Azure Site Recovery
description: Cet article explique comment gérer un serveur de processus pour la récupération d’urgence de machines virtuelles VMware et de serveurs physiques à l’aide d’Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/28/2019
ms.author: ramamill
ms.openlocfilehash: a547a874c42d06d8453b154847561d8b5f0dabb8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019183"
---
# <a name="manage-process-servers"></a>Gérer des serveurs de processus

Cet article décrit les tâches courantes de gestion du serveur de processus Site Recovery.

Le serveur de processus permet de recevoir, d’optimiser et d’envoyer les données de réplication à Azure. Il effectue également une installation push du service Mobilité sur les machines virtuelles VMware et les serveurs physiques que vous souhaitez répliquer, ainsi que la détection automatique des machines locales. Par défaut, le serveur de processus est installé sur l’ordinateur serveur de configuration pour la réplication des machines virtuelles VMware locales ou des serveurs physiques dans Azure. 

- Pour les déploiements à grande échelle, il peut être nécessaire de faire évoluer la capacité de vos serveurs de processus locaux.
- Pour la restauration automatique locale à partir d’Azure, vous devez configurer un serveur de processus temporaire dans Azure. Vous pourrez supprimer cette machine virtuelle une fois la restauration terminée. 

En savoir plus sur le serveur de processus.


## <a name="upgrade-a-process-server"></a>Mettre à niveau un serveur de processus

Lorsque vous déployez un serveur de processus en local ou en tant que machine virtuelle Azure pour la restauration automatique, la dernière version du serveur de processus est installée. L’équipe Site Recovery publie régulièrement des correctifs et des améliorations, aussi nous vous recommandons de mettre à jour vos serveurs de processus. Vous pouvez mettre à niveau un serveur de processus comme suit :

[!INCLUDE [site-recovery-vmware-upgrade -process-server](../../includes/site-recovery-vmware-upgrade-process-server-internal.md)]


## <a name="move-vms-to-balance-the-process-server-load"></a>Déplacer des machines virtuelles pour équilibrer la charge du serveur de processus

Équilibrez la charge en déplaçant les machines virtuelles entre deux serveurs de processus, comme suit :

1. Dans le coffre, sous **Gérer** cliquez sur **Infrastructure Site Recovery**. Sous **Pour machines physiques et VMware**, cliquez sur **Serveurs de configuration**.
2. Cliquez sur le serveur de configuration avec lequel les serveurs de processus sont inscrits.
3. Cliquez sur le serveur de processus dont vous souhaitez équilibrer la charge du trafic.

    ![Capture d’écran montrant un serveur de traitement dont vous pouvez équilibrer la charge du trafic.](media/vmware-azure-manage-process-server/LoadBalance.png)

4. Cliquez sur **Équilibrer la charge**, puis sélectionnez le serveur de processus cible vers lequel vous souhaitez déplacer des machines. Cliquez ensuite sur **OK**.

    ![Capture d’écran montrant le volet Équilibrer la charge avec l’option Sélectionner un serveur de traitement cible sélectionnée.](media/vmware-azure-manage-process-server/LoadPS.PNG)

2. Cliquez sur **Sélectionner des machines**, puis choisissez les machines que vous souhaitez déplacer du serveur de processus actuel vers le serveur de processus cible. Des détails de la moyenne de modification des données sont affichés sur chaque machine virtuelle. Cliquez ensuite sur **OK**. 
3. Dans le coffre, suivez la progression du travail sous **Surveillance** > **Travaux Site Recovery**.

La répercussion des modifications dans le portail prend environ 15 minutes. Pour obtenir un effet plus rapide, [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="switch-an-entire-workload-to-another-process-server"></a>Basculer d’une charge de travail entière vers un autre serveur de processus

Déplacez la charge de travail entière gérée par un serveur de processus vers un autre serveur de processus, comme suit :

1. Dans le coffre, sous **Gérer** cliquez sur **Infrastructure Site Recovery**. Sous **Pour machines physiques et VMware**, cliquez sur **Serveurs de configuration**.
2. Cliquez sur le serveur de configuration avec lequel les serveurs de processus sont inscrits.
3. Cliquez sur le serveur de processus à partir duquel vous souhaitez basculer la charge de travail.
4. Cliquez sur **Basculer**, puis sélectionnez le serveur de processus cible vers lequel vous souhaitez déplacer la charge de travail. Cliquez ensuite sur **OK**.

    ![Capture d’écran montrant le volet Sélectionner un serveur de traitement cible.](media/vmware-azure-manage-process-server/Switch.PNG)

5. Dans le coffre, suivez la progression du travail sous **Surveillance** > **Travaux Site Recovery**.

La répercussion des modifications dans le portail prend environ 15 minutes. Pour obtenir un effet plus rapide, [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="register-a-master-target-server"></a>Inscrire un serveur cible maître

Le serveur cible maître réside sur le serveur de configuration et les serveurs de processus Scale-out. Il doit être inscrit auprès du serveur de configuration. En cas d’échec de cette inscription, cela peut avoir un impact sur l’intégrité des éléments protégés. Pour inscrire un serveur cible maître auprès du serveur de configuration, connectez-vous au serveur de configuration/serveur de processus Scale-out spécifique sur lequel l’inscription est requise. Accédez au dossier **%PROGRAMDATA%\ASR\Agent** et exécutez la commande suivante sur l’invite de commandes Administrateur.

   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="reregister-a-process-server"></a>Réinscrire un serveur de processus

Réinscrivez un serveur de processus exécuté en local ou sur une machine virtuelle Azure avec le serveur de configuration comme suit :

[!INCLUDE [site-recovery-vmware-register-process-server](../../includes/site-recovery-vmware-register-process-server.md)]

Une fois les paramètres enregistrés, procédez comme suit :

1. Sur le serveur de processus, ouvrez une invite de commandes administrateur.
2. Accédez au dossier **%PROGRAMDATA%\ASR\Agent**, puis exécutez la commande suivante :

    ```
    cdpcli.exe --registermt
    net stop obengine
    net start obengine
    ```

## <a name="modify-proxy-settings-for-an-on-premises-process-server"></a>Modifier les paramètres de proxy pour un serveur de processus local

Si un serveur de processus local utilise un proxy pour se connecter à Azure, vous pouvez modifier les paramètres de proxy comme suit :

1. Connectez-vous à l’ordinateur du serveur de processus. 
2. Ouvrez une fenêtre de commande d’administration PowerShell et exécutez la commande suivante :
   ```powershell
   $pwd = ConvertTo-SecureString -String MyProxyUserPassword
   Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $pwd
   net stop obengine
   net start obengine
   ```
2. Accédez au dossier **%PROGRAMDATA%\ASR\Agent**, puis exécutez cette commande :
   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="remove-a-process-server"></a>Supprimer un serveur de processus

[!INCLUDE [site-recovery-vmware-unregister-process-server](../../includes/site-recovery-vmware-unregister-process-server.md)]

## <a name="exclude-folders-from-anti-virus-software"></a>Exclure des dossiers du logiciel antivirus

Si un logiciel antivirus s’exécute sur un serveur de processus scale-out (ou un serveur cible maître), excluez les dossiers suivants des opérations antivirus :


- C:\Program Files\Microsoft Azure Recovery Services Agent
- C:\ProgramData\ASR
- C:\ProgramData\ASRLogs
- C:\ProgramData\ASRSetupLogs
- C:\ProgramData\LogUploadServiceLogs
- C:\ProgramData\Microsoft Azure Site Recovery
- Répertoire d’installation du serveur de processus. Par exemple : C:\Program Files (x86)\Microsoft Azure Site Recovery