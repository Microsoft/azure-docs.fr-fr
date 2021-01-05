---
title: Résoudre des problèmes de réplication durant la récupération d’urgence de serveurs physiques et de machines virtuelles VMware sur Azure avec Azure Site Recovery | Microsoft Docs
description: Cet article fournit des informations sur la résolution de problèmes de réplication durant la récupération d’urgence de serveurs physiques et de machines virtuelles VMware sur Azure avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 08/2/2019
ms.author: mayg
ms.openlocfilehash: 8b44a1d6119cc658b9460e0a52fa0629f759964a
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91336203"
---
# <a name="troubleshoot-replication-issues-for-vmware-vms-and-physical-servers"></a>Résoudre les problèmes de réplication pour les serveurs physiques et machines virtuelles VMware

Cet article décrit certains problèmes courants et erreurs spécifiques que vous pouvez rencontrer lorsque vous répliquez des machines virtuelles et serveurs physiques VMware locaux sur Azure avec [Site Recovery](site-recovery-overview.md).

## <a name="step-1-monitor-process-server-health"></a>Étape 1 : Surveiller l’intégrité du serveur de processus

Site Recovery utilise le [serveur de processus](vmware-physical-azure-config-process-server-overview.md#process-server) pour recevoir et optimiser les données répliquées, ainsi que pour les envoyer à Azure.

Nous vous recommandons de surveiller l’intégrité des serveurs de processus dans le portail, pour vous assurer qu’ils sont connectés et fonctionnent correctement, et que la réplication progresse pour les machines sources associées au serveur de processus.

- [En savoir plus](vmware-physical-azure-monitor-process-server.md) sur la surveillance des serveurs de processus
- [Passer en revue les meilleures pratiques](vmware-physical-azure-troubleshoot-process-server.md#best-practices-for-process-server-deployment)
- [Détecter un problème](vmware-physical-azure-troubleshoot-process-server.md#check-process-server-health) d’intégrité du serveur de processus

## <a name="step-2-troubleshoot-connectivity-and-replication-issues"></a>Étape 2 : Détecter les problèmes de connectivité et de réplication

Les défaillances de réplication initiales et en cours résultent souvent de problèmes de connectivité entre le serveur source et le serveur de processus, ou entre ce dernier et Azure.

Pour résoudre ces difficultés, [détectez les problèmes de connectivité et de réplication](vmware-physical-azure-troubleshoot-process-server.md#check-connectivity-and-replication).




## <a name="step-3-troubleshoot-source-machines-that-arent-available-for-replication"></a>Étape 3 : Détecter les problèmes de machines sources qui ne sont pas disponibles pour la réplication

Lorsque vous tentez de sélectionner la machine source pour activer la réplication avec Site Recovery, il se peut que cette machine ne soit pas disponible pour l’une des raisons suivantes :

* **Deux machines virtuelles avec le même UUID d’instance** : Si deux machines virtuelles sous le vCenter ont le même UUID d’instance, la première machine découverte par le serveur de configuration apparaît sur le portail Azure. Pour résoudre ce problème, assurez-vous que deux machines virtuelles n’ont pas le même UUID d’instance. Ce scénario est courant dans les cas où une machine virtuelle de sauvegarde devient active et est connectée à nos enregistrements de découverte. Consultez [Azure Site Recovery, WMware vers Azure : comment nettoyer les entrées en double ou périmées](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx) pour résoudre le problème.
* **Informations d’identification de l’utilisateur vCenter incorrectes** : Vérifiez que vous avez ajouté les informations d’identification de vCenter correctes lors de la configuration du serveur de configuration en utilisant le modèle OVF ou une configuration unifiée. Pour vérifier les informations d’identification que vous avez ajoutées pendant la configuration, voir [Modifier les informations d’identification pour la découverte automatique](vmware-azure-manage-configuration-server.md#modify-credentials-for-automatic-discovery).
* **Privilèges insuffisants pour vCenter** : Si les autorisations fournies pour accéder à vCenter ne sont pas les autorisations requises, la découverte des machines virtuelles pourrait échouer. Assurez-vous que les autorisations décrites dans [Préparer un compte pour la découverte automatique](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery) sont ajoutées au compte d’utilisateur de vCenter.
* **Serveurs d’administration Azure Site Recovery** : Si la machine virtuelle est utilisée comme serveur d’administration sous un ou plusieurs de ces rôles (serveur de configuration, serveur de processus avec scale-out, serveur cible maître), vous ne pourrez pas choisir la machine virtuelle à partir du portail. Les serveurs d’administration ne peuvent pas être répliqués.
* **Machine virtuelle déjà protégée/basculée par le biais des services Azure Site Recovery** : si la machine virtuelle est déjà protégée ou basculée par le biais de Site Recovery, vous ne pouvez pas la sélectionner à des fins de protection dans le portail. Vérifiez que la machine virtuelle que vous recherchez sur le portail n’est pas déjà protégée par un autre utilisateur ou sous un autre abonnement.
* **vCenter non connecté** : Vérifiez si vCenter est dans un état connecté. Pour cela, accédez au coffre Recovery Services > Infrastructure Site Recovery > Serveurs de configuration, puis cliquez sur un serveur de configuration. Un panneau s’ouvre à droite avec des détails sur les serveurs associés. Vérifiez si vCenter est connecté. S’il présente l’état « Non connecté », résolvez le problème, puis [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server) dans le portail. Après cela, la machine virtuelle est listée dans le portail.
* **ESXi hors tension** : Si l’hôte ESXi sous lequel la machine virtuelle réside est dans un état hors tension, la machine virtuelle n’est pas listée ou ne peut pas être sélectionnée dans le portail Azure. Mettez l’hôte ESXi sous tension, puis [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server) dans le portail. Après cela, la machine virtuelle est listée dans le portail.
* **Redémarrage en attente** : si un redémarrage est en attente sur la machine virtuelle, vous ne pouvez pas sélectionner la machine dans le portail Azure. Veillez à effectuer les activités de redémarrage en attente, puis [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Après cela, la machine virtuelle est listée dans le portail.
* **IP introuvable** : Si une adresse IP valide n’est pas associée à la machine virtuelle, vous ne pouvez pas sélectionner la machine dans le portail Azure Veillez à affecter une adresse IP valide à la machine virtuelle, puis [actualisez le serveur de configuration](vmware-azure-manage-configuration-server.md#refresh-configuration-server). Après cela, la machine virtuelle est listée dans le portail.

### <a name="troubleshoot-protected-virtual-machines-greyed-out-in-the-portal"></a>Détecter les problèmes de machines virtuelles protégées qui apparaissent en grisé dans le portail

Les machines virtuelles répliquées sous Azure Site Recovery sont indisponibles sur le portail Azure s’il existe des entrées en double dans le système. Pour savoir comment supprimer des entrées périmées et résoudre le problème, consultez [Azure Site Recovery, WMware vers Azure : comment nettoyer les entrées en double ou périmées](https://social.technet.microsoft.com/wiki/contents/articles/32026.asr-vmware-to-azure-how-to-cleanup-duplicatestale-entries.aspx).

## <a name="no-crash-consistent-recovery-point-available-for-the-vm-in-the-last-xxx-minutes"></a>Aucun point de récupération d’incident cohérent disponible pour la machine virtuelle pendant les « XXX » dernières minutes

Certains des problèmes les plus courants sont répertoriés ci-dessous

### <a name="initial-replication-issues-error-78169"></a>Problèmes de réplication initiale [erreur 78169]

Après vous être assuré de l’absence de problèmes de connectivité, de bande passante ou de synchronisation de l’heure, vérifiez les points suivants :

- Aucun antivirus ne bloque Azure Site Recovery. Apprenez-en [plus](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) sur les exclusions de dossier requises pour Azure Site Recovery.

### <a name="source-machines-with-high-churn-error-78188"></a>Machines sources à taux d’évolution élevé [erreur 78188]

Causes possibles :
- Le taux de modification des données (octets écrits/s) sur les disques répertoriés de la machine virtuelle dépasse les [limites prises en charge par Azure Site Recovery](site-recovery-vmware-deployment-planner-analyze-report.md#azure-site-recovery-limits) pour le type de compte de stockage cible de réplication.
- Le taux d’évolution des données connaît un pic soudain à cause duquel un important volume de données est en attente de chargement.

Pour résoudre le problème :
- Assurez-vous que le type de compte de stockage cible (Standard ou Premium) est approvisionné conformément aux exigences de taux d’évolution des données au niveau de la machine source.
- Si vous effectuez déjà une réplication vers un disque managé Premium (de type asrseeddisk), vérifiez que la taille du disque prend en charge le taux d’évolution observé, conformément aux limites Site Recovery. Vous pouvez augmenter la taille du asrseeddisk si nécessaire. Effectuez les étapes suivantes :
    - Accédez au panneau Disques de la machine répliquée concernée et copiez le nom du disque de réplica.
    - Accédez à ce disque managé de réplica.
    - Vous pouvez voir une bannière dans le panneau Vue d’ensemble indiquant qu’une URL de signature d’accès partagé a été générée. Cliquez sur cette bannière et annulez l’exportation. Ignorez cette étape si vous ne voyez pas la bannière.
    - Dès que l’URL de la signature d’accès partagé est révoquée, accédez au panneau Configuration du disque managé et augmentez la taille de manière à ce qu’Azure Site Recovery prenne en charge le débit observé sur le disque source.
- Si le taux d’évolution observé est temporaire, attendez quelques heures que le chargement des données en attente rattrape son retard et crée des points de récupération.
- Si le disque contient des données non critiques, comme des journaux temporaires ou des données de test, déplacez ces données ou excluez l’intégralité de ce disque de la réplication.
- Si le problème persiste, utilisez le [Planificateur de déploiement](site-recovery-deployment-planner.md#overview) Site Recovery pour faciliter la planification de la réplication.

### <a name="source-machines-with-no-heartbeat-error-78174"></a>Machines sources dépourvues de pulsation [erreur 78174]

Cette erreur se produit lorsque l’agent Mobilité Azure Site Recovery sur la machine source ne communique pas avec le serveur de configuration (CS).

Pour résoudre ce problème, procédez comme suit pour vérifier la connectivité réseau entre la machine virtuelle source et le serveur de configuration :

1. Vérifiez que la machine source est en cours d’exécution.
2. Connectez-vous à la machine source à l’aide d’un compte disposant de privilèges d’administrateur.
3. Vérifiez que les services ci-après sont en cours d’exécution et, dans le cas contraire, redémarrez-les :
   - Svagents (InMage Scout VX Agent)
   - Service d’application InMage Scout
4. Sur la machine source, examinez les journaux à l’emplacement ci-après pour obtenir les détails de l’erreur :

    *C:\Program Files (X86)\Microsoft Azure Site Recovery\agent\svagents\*.log*

### <a name="process-server-with-no-heartbeat-error-806"></a>Serveur de processus dépourvu de pulsation [erreur 806]
Si le serveur de processus est dépourvu de pulsation, vérifiez les points suivants :
1. La machine virtuelle du serveur de processus est opérationnelle.
2. Consultez les journaux ci-après sur le serveur de processus pour obtenir les détails de l’erreur :

    *C:\ProgramData\ASR\home\svsystems\eventmanager\*.log*\
    et
    *C:\ProgramData\ASR\home\svsystems\monitor_protection\*.log*

### <a name="master-target-server-with-no-heartbeat-error-78022"></a>Serveur cible maître dépourvu de pulsation [erreur 78022]

Cette erreur se produit lorsque l’agent Mobilité Azure Site Recovery sur la cible maître ne communique pas avec le serveur de configuration.

Pour résoudre ce problème, vérifiez l’état du service en procédant comme suit :

1. Vérifiez que la machine virtuelle cible maître est en cours d’exécution.
2. Connectez-vous à la machine virtuelle cible maître à l’aide d’un compte disposant de privilèges d’administrateur.
    - Vérifiez que le service svagents est en cours d’exécution. S’il l’est, redémarrez-le.
    - Vérifiez les journaux à l’emplacement ci-après pour obtenir les détails de l’erreur :

        *C:\Program Files (X86)\Microsoft Azure Site Recovery\agent\svagents\*.log*
3. Pour inscrire le serveur cible maître auprès du serveur de configuration, accédez au dossier **%PROGRAMDATA%\ASR\Agent**, puis exécutez ce qui suit dans l’invite de commandes :
   ```
   cmd
   cdpcli.exe --registermt

   net stop obengine

   net start obengine

   exit
   ```

## <a name="error-id-78144---no-app-consistent-recovery-point-available-for-the-vm-in-the-last-xxx-minutes"></a>ID d’erreur 78144 : aucun point de récupération cohérent avec l’application disponible pour la machine virtuelle pendant les « XXX » dernières minutes

Des améliorations ont été apportées aux versions [9.23](vmware-physical-mobility-service-overview.md#mobility-service-agent-version-923-and-higher) & [9.27](site-recovery-whats-new.md#update-rollup-39) de l’agent de mobilité pour gérer les comportements d’échec d’installation VSS. Vérifiez que vous disposez des dernières versions pour obtenir les meilleurs conseils en matière de résolution des défaillances VSS.

Certains des problèmes les plus courants sont répertoriés ci-dessous

#### <a name="cause-1-known-issue-in-sql-server-20082008-r2"></a>Cause 1 : Problème connu dans SQL Server 2008/2008 R2
**Procédure de résolution** : Il existe un problème connu dans SQL Server 2008/2008 R2. Référez-vous à cet article de la base de connaissances : [Azure Site Recovery Agent or other non-component VSS backup fails for a server hosting SQL Server 2008 R2](https://support.microsoft.com/help/4504103/non-component-vss-backup-fails-for-server-hosting-sql-server-2008-r2) (L’agent Azure Site Recovery ou une autre sauvegarde VSS sans composant échoue sur un serveur hébergeant une instance SQL Server 2008 R2)

#### <a name="cause-2-azure-site-recovery-jobs-fail-on-servers-hosting-any-version-of-sql-server-instances-with-auto_close-dbs"></a>Cause 2 : Les tâches Azure Site Recovery échouent lorsque des serveurs hébergent les instances de n’importe quelle version de SQL Server avec des bases de données AUTO_CLOSE
**Procédure de résolution** : Référez-vous à cet [article](https://support.microsoft.com/help/4504104/non-component-vss-backups-such-as-azure-site-recovery-jobs-fail-on-ser) de la base de connaissances


#### <a name="cause-3-known-issue-in-sql-server-2016-and-2017"></a>Cause 3 : Problème connu dans SQL Server 2016 et 2017
**Procédure de résolution** : Référez-vous à cet [article](https://support.microsoft.com/help/4493364/fix-error-occurs-when-you-back-up-a-virtual-machine-with-non-component) de la base de connaissances

#### <a name="cause-4-app-consistency-not-enabled-on-linux-servers"></a>Cause 4 : Cohérence des applications non activée sur les serveurs Linux
**Procédure de résolution** : Azure Site Recovery pour le système d’exploitation Linux prend en charge les scripts personnalisés des applications à des fins de cohérence. Le script personnalisé avec options pré et post-script sera utilisé par l’agent Mobilité Azure Site Recovery pour la cohérence des applications. [Voici](./site-recovery-faq.md#replication) les étapes pour l’activer.

### <a name="more-causes-due-to-vss-related-issues"></a>Autres causes provoquées par des problèmes liés à VSS :

Pour mieux résoudre le problème, vérifiez les fichiers sur la machine source pour obtenir le code d’erreur exact de l’échec :

*C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\Application Data\ApplicationPolicyLogs\vacp.log*

Comment localiser les erreurs dans le fichier ?
Recherchez la chaîne « vacpError » en ouvrant le fichier vacp.log dans un éditeur

`Ex: `**`vacpError`**`:220#Following disks are in FilteringStopped state [\\.\PHYSICALDRIVE1=5, ]#220|^|224#FAILED: CheckWriterStatus().#2147754994|^|226#FAILED to revoke tags.FAILED: CheckWriterStatus().#2147754994|^|`

Dans l’exemple ci-dessus, **2147754994** est le code d’erreur qui vous informe de l’échec, comme indiqué ci-dessous

#### <a name="vss-writer-is-not-installed---error-2147221164"></a>L’enregistreur VSS n’est pas installé - erreur 2147221164

*Procédure de résolution* : Pour générer une balise de cohérence d’application, Azure Site Recovery utilise le service VSS (cliché instantané de volume) de Microsoft. Il installe un fournisseur VSS pour que l’opération prenne des clichés instantanés de la cohérence d’application. Ce fournisseur VSS est installé en tant que service. Si le service de fournisseur VSS n’est pas installé, la création de clichés instantanés de la cohérence d’application échoue et l’ID d’erreur 0x80040154 « Classe non inscrite » s’affiche. </br>
Consultez [l’article relatif au dépannage de l’installation de l’enregistreur VSS](./vmware-azure-troubleshoot-push-install.md#vss-installation-failures)

#### <a name="vss-writer-is-disabled---error-2147943458"></a>L’enregistreur VSS est désactivé - erreur 2147943458

**Procédure de résolution** : Pour générer une balise de cohérence d’application, Azure Site Recovery utilise le service VSS (cliché instantané de volume) de Microsoft. Il installe un fournisseur VSS pour que l’opération prenne des clichés instantanés de la cohérence d’application. Ce fournisseur VSS est installé en tant que service. Si le service de fournisseur VSS est désactivé, la création de clichés instantanés de la cohérence d’application échoue et l’ID d’erreur « Le service spécifié est désactivé et ne peut pas être démarré (0x80070422) » s’affiche. </br>

- Si le service VSS est désactivé :
    - Vérifiez que le type de démarrage du service fournisseur VSS est défini sur **Automatique**.
    - Redémarrez les services suivants :
        - Service VSS
        - Fournisseur VSS d’Azure Site Recovery
        - Service VDS (Virtual Disk Service)

####  <a name="vss-provider-not_registered---error-2147754756"></a>VSS PROVIDER NOT_REGISTERED - erreur 2147754756

**Procédure de résolution** : Pour générer une balise de cohérence d’application, Azure Site Recovery utilise le service VSS (cliché instantané de volume) de Microsoft.
Vérifiez si le service de fournisseur VSS d’Azure Site Recovery est installé. </br>

- Réessayez d’installer le fournisseur à l’aide des commandes suivantes :
- Désinstallez le fournisseur existant : C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Uninstall.cmd
- Réinstallez : C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd

Vérifiez que le type de démarrage du service fournisseur VSS est défini sur **Automatique**.
    - Redémarrez les services suivants :
        - Service VSS
        - Fournisseur VSS d’Azure Site Recovery
        - Service VDS (Virtual Disk Service)

## <a name="error-id-95001---insufficient-permissions-found"></a>ID d’erreur 95001 : Autorisations insuffisantes

Cette erreur se produit lorsque vous tentez d’activer la réplication alors que les dossiers d’application ne disposent pas d’autorisations suffisantes.

**Procédure de résolution** : Pour résoudre ce problème, assurez-vous que l’utilisateur IUSR détient le rôle de propriétaire pour tous les dossiers indiqués ci-dessous :

- *C\ProgramData\Microsoft Azure Site Recovery\private*
- Répertoire d’installation. Par exemple, si le répertoire d’installation est le lecteur F, fournissez les autorisations nécessaires pour :
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems*
- Dossier *\pushinstallsvc* dans le répertoire d’installation. Par exemple, si le répertoire d’installation est le lecteur F, fournissez les autorisations nécessaires pour :
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\pushinstallsvc*
- Dossier *\etc* dans le répertoire d’installation. Par exemple, si le répertoire d’installation est le lecteur F, fournissez les autorisations nécessaires pour :
    - *F:\Program Files (x86)\Microsoft Azure Site Recovery\home\svsystems\etc*
- *C:\Temp*
- *C:\thirdparty\php5nts*
- Tous les éléments sur le chemin ci-dessous :
    - *C:\thirdparty\rrdtool-1.2.15-win32-perl58\rrdtool\Release\**

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’aide supplémentaire, publiez votre question sur la [page de questions Microsoft Q&R sur Azure Site Recovery](/answers/topics/azure-site-recovery.html). Nous avons une communauté active et l’un de nos ingénieurs peut vous aider.
