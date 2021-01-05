---
title: Dépanner le serveur de traitement Azure Site Recovery
description: Cet article décrit comment résoudre les problèmes avec le serveur de traitement Azure Site Recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: troubleshooting
ms.date: 09/09/2019
ms.author: raynew
ms.openlocfilehash: ad1bec66edaa3fcc6049f4911684f6e6d6c3e366
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999182"
---
# <a name="troubleshoot-the-process-server"></a>Dépanner le serveur de traitement

Le serveur de traitement [Site Recovery](site-recovery-overview.md) est utilisé pour la configuration d’une reprise après sinistre vers Azure de machines virtuelles VMware et de serveurs physiques. Cet article décrit comment résoudre les problèmes avec le serveur de traitement, y compris les problèmes de réplication et de connectivité.

[En savoir plus](vmware-physical-azure-config-process-server-overview.md) sur le serveur de traitement.

## <a name="before-you-start"></a>Avant de commencer

Avant de commencer le dépannage :

1. Assurez-vous de bien comprendre comment [surveiller les serveurs de traitement](vmware-physical-azure-monitor-process-server.md).
2. Consultez les bonnes pratiques ci-dessous.
3. Veuillez tenir compte des [considérations sur la capacité](site-recovery-plan-capacity-vmware.md#capacity-considerations) et utiliser le guide de dimensionnement du [serveur de configuration](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-configuration-server-and-inbuilt-process-server) ou des [serveurs de traitement autonomes](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-process-server).

## <a name="best-practices-for-process-server-deployment"></a>Bonnes pratiques de déploiement du serveur de traitement

Pour des performances optimales des serveurs de traitement, nous avons résumé un certain nombre de bonnes pratiques générales.

**Bonne pratique** | **Détails**
--- |---
**Utilisation** | Assurez-vous que le serveur de configuration/serveur de traitement autonome n'est utilisé que pour l'usage prévu. Ne lancez aucune autre tâche sur la machine.
**Adresse IP** | Assurez-vous que le serveur de traitement comporte une adresse IPv4 statique et n'a aucun réseau NAT configuré.
**Contrôle de l’utilisation de la mémoire et du processeur** |Maintenez l'utilisation du processeur et de la mémoire en dessous de 70 %.
**Garantir suffisamment d’espace libre** | L'espace libre fait référence à l'espace disque du cache sur le serveur de traitement. Les données de réplication sont stockées dans le cache avant d'être chargées vers Azure.<br/><br/> Conservez un espace libre supérieur à 25 %. S’il descend en dessous de 20 %, la réplication est limitée pour les machines répliquées associées au serveur de traitement.

## <a name="check-process-server-health"></a>Vérifier l'état d’intégrité du serveur de traitement

La première étape du dépannage consiste à vérifier l’intégrité et l’état du serveur de traitement. Pour cela, consultez toutes les alertes, vérifiez que les services requis sont en cours d'exécution et que le serveur de traitement émet des pulsations. Ces étapes sont résumées dans le graphique suivant, suivies de procédures pour vous aider à les exécuter.

![Détecter un problème d’intégrité du serveur de traitement](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-process-server-health.png)

## <a name="step-1-troubleshoot-process-server-health-alerts"></a>Étape 1 : Résoudre les alertes d’intégrité du serveur de traitement

Le serveur de traitement génère un certain nombre d’alertes d’intégrité. Ces alertes et les mesures recommandées sont résumées dans le tableau suivant.

**Type d’alerte** | **Error** | **Résolution des problèmes**
--- | --- | --- 
![Healthy][green] | None  | Le serveur de processus est connecté et sain.
![Avertissement][yellow] | Les services spécifiés ne fonctionnent pas. | 1. Vérifiez que les services sont en cours d’exécution.<br/> 2. Si les services fonctionnent comme prévu, suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).
![Avertissement][yellow]  | Utilisation du processeur > 80 % durant les 15 dernières minutes. | 1. N'ajoutez pas de nouvelles machines.<br/>2. Vérifiez que le nombre de machines virtuelles utilisant le serveur de traitement correspond aux [limites définies](site-recovery-plan-capacity-vmware.md#capacity-considerations), et installez un [serveur de traitement supplémentaire](vmware-azure-set-up-process-server-scale.md) si nécessaire.<br/>3. Suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).
![Critique][red] |  Utilisation du processeur > 95 % durant les 15 dernières minutes. | 1. N'ajoutez pas de nouvelles machines.<br/>2. Vérifiez que le nombre de machines virtuelles utilisant le serveur de traitement correspond aux [limites définies](site-recovery-plan-capacity-vmware.md#capacity-considerations), et installez un [serveur de traitement supplémentaire](vmware-azure-set-up-process-server-scale.md) si nécessaire.<br/>3. Suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).<br/> 4. Si le problème persiste, exécutez le [Planificateur de déploiement](./site-recovery-deployment-planner.md) pour la réplication VMware ou du serveur physique.
![Avertissement][yellow] | Utilisation de la mémoire > 80 % durant les 15 dernières minutes. |  1. N'ajoutez pas de nouvelles machines.<br/>2. Vérifiez que le nombre de machines virtuelles utilisant le serveur de traitement correspond aux [limites définies](site-recovery-plan-capacity-vmware.md#capacity-considerations), et installez un [serveur de traitement supplémentaire](vmware-azure-set-up-process-server-scale.md) si nécessaire.<br/>3. Suivez toutes les instructions liées à l'avertissement.<br/> 4. Si le problème persiste, suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).
![Critique][red] | Utilisation de la mémoire > 95 % durant les 15 dernières minutes. | 1. N'ajoutez pas de nouvelles machines et installez un [serveur de traitement supplémentaire](vmware-azure-set-up-process-server-scale.md) si nécessaire.<br/> 2. Suivez toutes les instructions liées à l'avertissement.<br/> 3. 4. Si le problème persiste, suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).<br/> 4. Si le problème persiste, exécutez le [Planificateur de déploiement](./site-recovery-deployment-planner.md) pour résoudre les problèmes de réplication VMware ou du serveur physique.
![Avertissement][yellow] | Espace libre du dossier cache < 30 % durant les 15 dernières minutes. | 1. N'ajoutez pas de nouvelles machines et installez un [serveur de traitement supplémentaire](vmware-azure-set-up-process-server-scale.md) si nécessaire.<br/>2. Vérifiez que le nombre de machines virtuelles utilisant le serveur de traitement correspond aux [instructions](site-recovery-plan-capacity-vmware.md#capacity-considerations).<br/> 3. Suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).
![Critique][red] |  Espace libre < 25 % durant les 15 dernières minutes | 1. Suivez les instructions liées à l'avertissement pour ce problème.<br/> 2. 3. Suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).<br/> 3. Si le problème persiste, exécutez le [Planificateur de déploiement](./site-recovery-deployment-planner.md) pour la réplication VMware ou du serveur physique.
![Critique][red] | Le serveur de traitement n’a émis aucune pulsation pendant 15 minutes ou plus. Le service tmansvs ne communique pas avec le serveur de configuration. | 1) Vérifiez que le serveur de traitement est opérationnel.<br/> 2. Vérifiez que le service tmassvc est en cours d’exécution sur le serveur de traitement.<br/> 3. Suivez les instructions ci-dessous pour [détecter les problèmes de connectivité et de réplication ](#check-connectivity-and-replication).


![Clé de table](./media/vmware-physical-azure-troubleshoot-process-server/table-key.png)


## <a name="step-2-check-process-server-services"></a>Étape 2 : Vérifier les services du serveur de traitement

Les services qui devraient être exécutés sur le serveur de traitement sont résumés dans le tableau suivant. Il existe de légères différences entre les services, selon la façon dont le serveur de traitement est déployé. 

Pour tous les services à l'exception de Microsoft Azure Recovery Services Agent (obengine), vérifiez que StartType est défini sur **Automatique** ou **Automatique (démarrage différé)** .
 
**Déploiement** | **Services en cours d’exécution**
--- | ---
**Serveur de traitement sur le serveur de configuration** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Log Upload Service (LogUpload); InMage Scout Application Service; Microsoft Azure Recovery Services Agent (obengine); InMage Scout VX Agent-Sentinel/Outpost (svagents); tmansvc; World Wide Web Publishing Service (W3SVC); MySQL; Microsoft Azure Site Recovery Service (dra)
**Serveur de traitement fonctionnant en tant que serveur autonome** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Log Upload Service (LogUpload); InMage Scout Application Service; Microsoft Azure Recovery Services Agent (obengine); InMage Scout VX Agent-Sentinel/Outpost (svagents); tmansvc.
**Serveur de traitement déployé sur Azure à des fins de restauration automatique** | ProcessServer; ProcessServerMonitor; cxprocessserver; InMage PushInstall; Log Upload Service (LogUpload)


## <a name="step-3-check-the-process-server-heartbeat"></a>Étape 3 : Vérifier les pulsations du serveur de traitement

Si le serveur de traitement n’émet aucune pulsation (code d'erreur 806), procédez comme suit :

1. Vérifiez que la machine virtuelle du serveur de traitement est opérationnelle.
2. Recherchez d’éventuelles erreurs dans ces journaux d’activité.

    C:\ProgramData\ASR\home\svsystems\eventmanager *.log  C\ProgramData\ASR\home\svsystems\monitor_protection*.log

## <a name="check-connectivity-and-replication"></a>Vérifier la connectivité et la réplication

 Les défaillances de réplication initiales et en cours résultent souvent de problèmes de connectivité entre les machines source et le serveur de traitement, ou entre ce dernier et Azure. Ces étapes sont résumées dans le graphique suivant, suivies de procédures pour vous aider à les exécuter.

![Organigramme présentant les étapes de la résolution des problèmes liés à la connectivité et à la réplication.](./media/vmware-physical-azure-troubleshoot-process-server/troubleshoot-connectivity-replication.png)


## <a name="step-4-verify-time-sync-on-source-machine"></a>Étape 4 : Vérifier la synchronisation de l'heure sur la machine source

Assurez-vous que la date et l’heure système de la machine répliquée sont synchronisées. [En savoir plus](/windows-server/networking/windows-time-service/accurate-time)

## <a name="step-5-check-anti-virus-software-on-source-machine"></a>Étape 5 : Vérifier le logiciel antivirus de la machine source

Vérifiez qu'aucun logiciel antivirus sur la machine répliquée ne bloque Site Recovery. Si vous devez exclure Site Recovery des programmes antivirus, consultez [cet article](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program).

## <a name="step-6-check-connectivity-from-source-machine"></a>Étape 6 : Vérifier la connectivité de la machine source


1. Installez le [client Telnet](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771275(v=ws.10)) sur la machine source si nécessaire. N’utilisez pas Ping.
2. Depuis la machine source, effectuez un test ping sur le serveur de traitement sur le port HTTPS avec Telnet. Par défaut, le port 9443 est le port HTTPS pour le trafic de réplication.

    `telnet <process server IP address> <port>`

3. Vérifiez si la connexion a réussi.


**Connectivité** | **Détails** | **Action**
--- | --- | ---
**Réussi** | Telnet affiche un écran vide, et le serveur de traitement est accessible. | Aucune action supplémentaire n’est requise.
**Échec** | Vous n’arrivez pas à vous connecter | Vérifiez que le port d'entrée 9443 est autorisé sur le serveur de traitement. Par exemple, si vous utilisez un réseau de périmètre ou un sous-réseau filtré. Revérifiez la connectivité.
**Réussite partielle** | Vous pouvez vous connecter, mais la machine source signale que le serveur de traitement est inaccessible. | Passez à la procédure de dépannage suivante.

## <a name="step-7-troubleshoot-an-unreachable-process-server"></a>Étape 7 : Dépanner un serveur de traitement inaccessible

Si le serveur de traitement est inaccessible depuis la machine source, l'erreur 78186 s’affiche. Si le problème n’est pas résolu, les points de récupération cohérents après incident et cohérents au niveau application ne sont pas générés comme prévu.

Pour résoudre ce problème, vérifiez si la machine source peut atteindre l'adresse IP du serveur de traitement, puis exécutez l'outil cxpsclient sur la machine source pour tester la connexion de bout en bout.


### <a name="check-the-ip-connection-on-the-process-server"></a>Vérifier la connexion IP sur le serveur de traitement

Si telnet réussit mais que la machine source signale que le serveur de traitement est inaccessible, vérifiez si vous pouvez atteindre l'adresse IP de ce serveur.

1. Dans un navigateur Web, essayez d'atteindre l'adresse IP https://<IP_PS>:<Port_Données_PS>/.
2. Cette vérification génère une erreur de certificat HTTPS ? C'est normal. Si vous ignorez l'erreur, une erreur « 400 - Requête incorrecte » devrait apparaître. Cela signifie que le serveur ne peut pas répondre à la requête du navigateur, et que la connexion HTTPS standard fonctionne correctement.
3. Si cette vérification ne fonctionne pas, notez le message d'erreur du navigateur. Par exemple, une erreur 407 indiquera un problème d'authentification proxy.

### <a name="check-the-connection-with-cxpsclient"></a>Vérifier la connexion avec cxpsclient

Vous pouvez également exécuter l'outil cxpsclient pour vérifier la connexion de bout en bout.

1. Exécutez l'outil comme suit :

    ```
    <install folder>\cxpsclient.exe -i <PS_IP> -l <PS_Data_Port> -y <timeout_in_secs:recommended 300>
    ```

2. Sur le serveur de traitement, vérifiez les journaux d’activité générés dans ces dossiers :

    C:\ProgramData\ASR\home\svsystems\transport\log\cxps.err  C:\ProgramData\ASR\home\svsystems\transport\log\cxps.xfer



### <a name="check-source-vm-logs-for-upload-failures-error-78028"></a>Vérifier si les journaux d’activité des machines virtuelles source contiennent des erreurs de chargement (erreur 78028)

Un problème de blocage des chargements de données depuis les machines sources vers le service de traitement peut empêcher la génération de points de récupération cohérents après incident et cohérents au niveau application. 

1. Pour résoudre les problèmes de chargement sur le réseau, vous pouvez rechercher d’éventuelles erreurs dans ce journal d’activité :

    C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\svagents*.log 

2. Les autres procédures de cet article peuvent vous aider à résoudre les problèmes de chargement des données.



## <a name="step-8-check-whether-the-process-server-is-pushing-data"></a>Étape 8 : Vérifier si le serveur de traitement transmet activement des données

Vérifiez si le serveur de traitement transmet activement des données à Azure.

  1. Sur le serveur de processus, ouvrez le Gestionnaire des tâches (appuyez sur Ctrl+Maj+Échap).
  2. Sélectionnez l’onglet **Performances** > **Ouvrir le Moniteur de ressources**.
  3. Dans la page **Moniteur de ressources**, sélectionnez l’onglet **Réseau**. Sous **Processus avec activité réseau**, vérifiez si cbengine.exe envoie activement un volume important de données.

       ![Capture d’écran montrant un grand nombre de volumes sous des processus avec activité réseau.](./media/vmware-physical-azure-troubleshoot-process-server/cbengine.png)

  Si cbengine.exe n’envoie pas un volume important de données, procédez de la manière décrite dans les sections suivantes.

## <a name="step-9-check-the-process-server-connection-to-azure-blob-storage"></a>Étape 9 : Vérifier la connexion du serveur de traitement au Stockage Blob Azure

1. Dans Moniteur de ressources, sélectionnez **cbengine.exe**.
2. Sous **Connexions TCP**, vérifiez s’il y a une connectivité du serveur de traitement au Stockage Azure.

  ![Capture d’écran montrant la connectivité entre cbengine.exe et l’URL de Stockage Blob Azure.](./media/vmware-physical-azure-troubleshoot-process-server/rmonitor.png)

### <a name="check-services"></a>Vérifier les services

À défaut de connectivité du serveur de traitement à l’URL du Stockage Blob Azure, vérifiez que les services sont en cours d’exécution.

1. Dans le Panneau de configuration, sélectionnez **Services**.
2. Vérifiez que les services suivants sont en cours d’exécution :

    - cxprocessserver
    - InMage Scout VX Agent – Sentinel/Outpost
    - Agent Microsoft Azure Recovery Services
    - Service Microsoft Azure Site Recovery
    - tmansvc

3. Démarrez ou redémarrez tout service qui n’est pas en cours d’exécution.
4. Vérifiez que le serveur de traitement est connecté et accessible. 

## <a name="step-10-check-the-process-server-connection-to-azure-public-ip-address"></a>Étape 10 : vérifier la connexion du serveur de traitement à l'adresse IP publique Azure

1. Sur le serveur de traitement, dans **%programfiles%\Microsoft Azure Recovery Services Agent\Temp**, ouvrez le dernier fichier CBEngineCurr.errlog.
2. Dans le fichier, recherchez **443** ou la chaîne **Échec de la tentative de connexion**.

  ![Journaux d'erreurs dans le dossier Temp](./media/vmware-physical-azure-troubleshoot-process-server/logdetails1.png)

3. En cas de problèmes, vous trouverez votre adresse IP publique Azure dans le fichier CBEngineCurr.currLog via le port 443 :

  `telnet <your Azure Public IP address as seen in CBEngineCurr.errlog>  443`

5. Dans la ligne de commande du serveur de traitement, utilisez Telnet pour effectuer un test ping sur votre adresse IP publique Azure.
6. Si vous ne parvenez pas à vous connecter, procédez comme suit.

## <a name="step-11-check-process-server-firewall-settings"></a>Étape 11 : Vérifier les paramètres du pare-feu du serveur de traitement. 

Vérifiez si le pare-feu basé sur l’adresse IP sur le serveur de processus bloque l’accès.

1. Pour les règles de pare-feu basées sur une adresse IP :

    a) Téléchargez la liste complète des [plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).

    b) Ajoutez les plages d’adresses IP à la configuration de votre pare-feu pour vous assurer que celui-ci autorise les communications vers Azure (et le port HTTPS par défaut, 443).

    c) Autorisez les plages d’adresses IP correspondant à la région Azure de votre abonnement et à la région Azure USA Ouest (utilisée pour le contrôle d’accès et la gestion des identités).

2. Pour les pare-feu basés sur une URL, ajoutez les URL répertoriées dans le tableau suivant à la configuration du pare-feu.

    [!INCLUDE [site-recovery-URLS](../../includes/site-recovery-URLS.md)]  


## <a name="step-12-verify-process-server-proxy-settings"></a>Étape 12 : Vérifier les paramètres de proxy du serveur de traitement 

1. Si vous utilisez un serveur proxy, vérifiez que son nom est résolu par le serveur DNS. Vérifiez la valeur que vous avez fournie lors de la configuration du serveur de configuration dans la clé de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Azure Site Recovery\ProxySettings**.
2. Assurez-vous que les mêmes paramètres sont utilisés par l’agent Azure Site Recovery pour l’envoi de données.

    a) Recherchez **Sauvegarde Microsoft Azure**.

    b) Ouvrez **Sauvegarde Microsoft Azure**, puis sélectionnez **Action** > **Modifier les propriétés**.

    c) Dans l’onglet **Configuration du proxy**, l’adresse du proxy doit être identique à celle indiquée dans les paramètres du Registre. Si ce n’est pas le cas, modifiez-la.

## <a name="step-13-check-bandwidth"></a>Étape 13 : Vérifier la bande passante

Augmentez la bande passante entre le serveur de traitement et Azure, puis vérifiez si le problème persiste.


## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’aide supplémentaire, publiez votre question sur la [page de questions Microsoft Q&R sur Azure Site Recovery](/answers/topics/azure-site-recovery.html). 

[green]: ./media/vmware-physical-azure-troubleshoot-process-server/green.png
[yellow]: ./media/vmware-physical-azure-troubleshoot-process-server/yellow.png
[red]: ./media/vmware-physical-azure-troubleshoot-process-server/red.png