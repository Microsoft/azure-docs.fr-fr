---
title: Planifier la capacité pour la reprise d’activité de VMware avec Azure Site Recovery
description: Cet article peut vous aider à planifier la capacité et la mise à l’échelle lorsque vous configurez la récupération d’urgence de machines virtuelles VMware vers Azure à l’aide d’Azure Site Recovery.
author: nsoneji
manager: garavd
ms.service: site-recovery
ms.date: 4/9/2019
ms.topic: conceptual
ms.author: ramamill
ms.openlocfilehash: 4b86d0c189bcf0687a703f2338188df2090feaf0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368024"
---
# <a name="plan-capacity-and-scaling-for-vmware-disaster-recovery-to-azure"></a>Planifier la capacité et la mise à l’échelle pour la récupération d’urgence VMware vers Azure

Cet article explique comment planifier la capacité et la mise à l’échelle lors de la réplication de serveurs physiques et machines virtuelles VMware locaux dans Azure avec [Azure Site Recovery](site-recovery-overview.md).

## <a name="how-do-i-start-capacity-planning"></a>Comment dois-je commencer la planification de la capacité ?

Pour découvrir les exigences de l’infrastructure Azure Site Recovery, collectez des informations sur votre environnement de réplication en exécutant le [Planificateur de déploiement Azure Site Recovery](./site-recovery-deployment-planner.md) pour la réplication VMware. Pour plus d’informations, voir [À propos du planificateur de déploiement Azure Site Recovery pour VMware sur Azure](site-recovery-deployment-planner.md). 

Le Planificateur de déploiement Azure Site Recovery fournit un rapport contenant des informations complètes sur les machines virtuelles compatibles et incompatibles, le nombre de disques par machine virtuelle et l’activité des données par disque. L’outil récapitule également les besoins en bande passante réseau pour satisfaire à l’objectif de point de récupération cible, et l’infrastructure Azure requise pour la réussite de la réplication et du test de basculement.

## <a name="capacity-considerations"></a>Considérations relatives à la capacité

Composant | Détails
--- | ---
**Réplication** | **Taux de modification quotidien maximal** : Un ordinateur protégé ne peut utiliser qu’un seul serveur de processus. Un seul serveur de processus peut prendre en charge un taux de modification quotidien maximal de 2 To. Par conséquent, 2 To est le taux quotidien maximal de modifications de données pris en charge pour une machine protégée.<br /><br /> **Débit maximal** : un ordinateur répliqué peut appartenir à un compte de stockage dans Azure. Un compte de stockage Azure standard peut gérer au maximum 20 000 requêtes par seconde. Nous vous recommandons de limiter le nombre d’opérations d’entrée/sortie par seconde (IOPS) sur une machine source à 20 000. Par exemple, si vous disposez d’une machine source équipée de cinq disques donc chacun génère 120 IOPS (d’une taille de 8 Ko) sur la machine source, celle-ci respecte la limite Azure de 500 IOPS par disque (le nombre de comptes de stockage requis est égal au nombre total d’IOPS de la machine source divisé par 20 000).
**Serveur de configuration** | Le serveur de configuration doit être capable de gérer la capacité de taux de changement quotidien pour toutes les charges de travail exécutées sur des machines protégées. La machine de configuration doit disposer d’une bande passante suffisante pour répliquer en continu les données sur Stockage Azure.<br /><br /> Une meilleure pratique consiste à placer le serveur de configuration sur le même réseau et le même segment de réseau local que les machines à protéger. Vous pouvez placer le serveur de configuration sur un autre réseau, mais les machines que vous souhaitez protéger doivent disposer d’une visibilité de réseau de couche 3.<br /><br /> Les recommandations en matière de taille du serveur de configuration sont résumées dans le tableau de la section suivante.
**Serveur de traitement** | Le premier serveur de processus est installé par défaut sur le serveur de configuration. Vous pouvez déployer des serveurs de processus supplémentaires pour étendre votre environnement. <br /><br /> Le serveur de processus reçoit des données de réplication de machines protégées. Le serveur de processus optimise les données à l’aide d’une mise en cache, d’une compression et d’un chiffrement. Ensuite, le serveur de processus envoie les données à Azure. Le serveur de processus doit disposer de ressources suffisantes pour effectuer ces tâches.<br /><br /> Le serveur de processus utilise un cache disque. Utilisez un disque de cache distinct d’au minimum 600 Go pour gérer les modifications de données stockées en cas de goulot d’étranglement ou de panne.

## <a name="size-recommendations-for-the-configuration-server-and-inbuilt-process-server"></a>Recommandations de taille pour le serveur de configuration et le serveur de processus intégré

Un serveur de configuration qui utilise un serveur de processus intégré pour protéger la charge de travail peut gérer jusqu’à 200 machines virtuelles en fonction des configurations suivantes :

UC | Mémoire | Taille du disque cache | Taux de modification des données | Machines protégées
--- | --- | --- | --- | ---
8 processeurs virtuels (2 sockets * 4 cœurs \@ 2,5 GHz) | 16 Go | 300 Go | 500 Go ou moins | Permet de répliquer moins de 100 machines.
12 processeurs virtuels (2 sockets * 6 cœurs \@ 2,5 GHz) | 18 Go | 600 Go | 501 Go à 1 To | Permet de répliquer de 100 à 150 machines.
16 processeurs virtuels (2 sockets * 8 cœurs \@ 2,5 GHz) | 32 Go | 1 To | >1 To à 2 To | Permet de répliquer de 151 à 200 machines.
Déployez un autre serveur de configuration à l’aide du [modèle OVF](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template). | | | | Déployez un nouveau serveur de configuration si vous répliquez plus de 200 machines.
Déployez un autre [serveur de processus](vmware-azure-set-up-process-server-scale.md#download-installation-file). | | | > 2 To| Déployez un nouveau serveur de processus de scale-out si le taux total quotidien de modifications est supérieur à 2 To.

Dans ces configurations :

* Chaque machine source comprend trois disques de 100 Go.
* Nous avons utilisé le stockage de référence de huit lecteurs avec signature d’accès partagé de 10 K tr/min avec RAID 10 pour les mesures de disque cache.

## <a name="size-recommendations-for-the-process-server"></a>Recommandations de taille pour le serveur de processus

Le serveur de processus est le composant qui gère le processus de réplication de données dans Azure Site Recovery. Si la quantité de modifications quotidiennes est supérieure à 2 To, vous devez ajouter des serveurs de processus de scale-out pour gérer la charge de réplication. Pour effectuer un scale-out, vous pouvez :

* Augmenter le nombre de serveurs de configuration en déployant à l’aide d’un [modèle OVF](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template). Par exemple, vous pouvez protéger jusqu’à 400 machines avec deux serveurs de configuration.
* Ajoutez des [serveurs de processus de scale-out](vmware-azure-set-up-process-server-scale.md#download-installation-file). Utilisez des serveurs de processus de scale-out pour gérer le trafic de réplication à la place (ou en plus) du serveur de configuration.

Le tableau suivant décrit ce scénario :

* Vous avez configuré un serveur de processus de scale-out.
* Vous avez configuré des machines virtuelles protégées pour qu’elles utilisent le serveur de processus de scale-out.
* Chaque machine source protégée comprend trois disques de 100 Go.

Serveur de processus supplémentaire | Taille du disque cache | Taux de modification des données | Machines protégées
--- | --- | --- | ---
4 processeurs virtuels (2 sockets * 2 cœurs \@ 2,5 GHz), 8 Go de mémoire | 300 Go | 250 Go ou moins | Permet de répliquer jusqu’à 85 machines.
8 processeurs virtuels (2 sockets * 4 cœurs \@ 2,5 GHz), 12 Go de mémoire | 600 Go | 251 Go à 1 To | Permet de répliquer de 86 à 150 machines.
12 processeurs virtuels (2 sockets \@ 6 cœurs @ 2,5 GHz), 24 Go de mémoire | 1 To | >1 To à 2 To | Permet de répliquer de 151 à 225 machines.

Le mode de mise à l’échelle de vos serveurs dépend de votre préférence pour un modèle de montée en puissance ou de montée en charge. Pour monter en puissance, déployez quelques serveurs de configuration et serveurs de processus haut de gamme. Pour effectuer un scale-out, déployez davantage de serveurs comptant moins de ressources. Par exemple, si vous souhaitez protéger 200 machines avec taux global quotidien de modifications de données de 1,5 To, vous pouvez effectuer l’une des actions suivantes :

* Configurer un seul serveur de processus (16 processeurs virtuels, 24 Go de RAM).
* Configurer deux serveurs de processus (2 x 8 processeurs virtuels, 2 * 12 Go de RAM).

## <a name="control-network-bandwidth"></a>Contrôler la bande passante réseau

Après avoir exécuté le [Planificateur de déploiement Azure Site Recovery](site-recovery-deployment-planner.md) pour calculer la bande passante nécessaire pour la réplication (réplication initiale et delta), vous pouvez contrôler la quantité de bande passante utilisée pour la réplication de deux façons :

* **Limiter la bande passante** : le trafic VMware qui est répliqué sur Azure passe par un serveur de processus spécifique. Vous pouvez limiter la bande passante sur les machines qui s’exécutent en tant que serveurs de processus.
* **Influer sur la bande passante** : vous pouvez influer sur la bande passante utilisée pour la réplication à l’aide de deux clés de Registre :
  * La valeur de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\UploadThreadsPerVM** détermine le nombre de threads utilisés pour le transfert des données (réplication initiale ou différentielle) sur un disque. Une valeur plus élevée augmenter la bande passante réseau utilisée pour la réplication.
  * La valeur de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\DownloadThreadsPerVM** détermine le nombre de threads utilisés pour le transfert des données pendant la restauration automatique.

### <a name="throttle-bandwidth"></a>Limiter la bande passante

1. Ouvrez le composant logiciel enfichable MMC Sauvegarde Azure sur la machine qui fait office de serveur de processus. Par défaut, un raccourci vers Sauvegarde est disponible sur le bureau ou dans le dossier suivant : C:\Program Files\Microsoft Azure Recovery Services Agent\bin.
2. Dans le composant logiciel enfichable, sélectionnez **Modifier les propriétés** .

    ![Capture d’écran de l’option de composant logiciel enfichable MMC Sauvegarde Azure pour modifier les propriétés](./media/site-recovery-vmware-to-azure/throttle1.png)
3. Sous l’onglet **Limitation** , sélectionnez la case **Activer la limitation de la bande passante sur Internet pour les opérations de sauvegarde** . Définissez les limites pour les heures ouvrées et non ouvrées. Les plages valides vont de 512 Kbits/s à 1 023 Mbits/s.

    ![Capture d’écran de la boîte de dialogue Propriétés de Sauvegarde Azure](./media/site-recovery-vmware-to-azure/throttle2.png)

Vous pouvez également utiliser l’applet de commande [Set-OBMachineSetting](/previous-versions/windows/powershell-scripting/hh770409(v=wps.640)) pour définir la limitation. Voici un exemple :

```azurepowershell-interactive
$mon = [System.DayOfWeek]::Monday
$tue = [System.DayOfWeek]::Tuesday
Set-OBMachineSetting -WorkDay $mon, $tue -StartWorkHour "9:00:00" -EndWorkHour "18:00:00" -WorkHourBandwidth  (512*1024) -NonWorkHourBandwidth (2048*1024)
```

**Set-OBMachineSetting -NoThrottle** indique qu’aucune limitation n’est requise.

### <a name="alter-the-network-bandwidth-for-a-vm"></a>Modifier la bande passante réseau d’une machine virtuelle

1. Dans le Registre de la machine virtuelle, accédez à **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication** .
   * Pour modifier le trafic de la bande passante sur un disque de réplication, modifiez la valeur de **UploadThreadsPerVM** . Créez la clé si elle n’existe pas.
   * Pour modifier la bande passante utilisée pour le trafic lié à la restauration automatique à partir d’Azure, modifiez la valeur de **DownloadThreadsPerVM** .
2. La valeur par défaut pour chaque clé est **4** . Dans un réseau « surutilisé », ces clés de Registre doivent être modifiées par rapport aux valeurs par défaut. La valeur maximale que vous pouvez utiliser est **32** . Surveillez le trafic pour optimiser la valeur.

## <a name="set-up-the-site-recovery-infrastructure-to-protect-more-than-500-vms"></a>Configurer l’infrastructure Site Recovery pour protéger plus de 500 machines virtuelles

Avant de configurer l’infrastructure Azure Site Recovery, accédez à l’environnement pour mesurer les facteurs suivants : machines virtuelles compatibles, taux quotidien de modifications des données, bande passante réseau requise pour l’objectif de point de récupération à atteindre, nombre de composants Azure Site Recovery nécessaires, temps nécessaire pour la réplication initiale. Pour collecter les informations requises, procédez comme suit :

1. Pour mesurer ces paramètres, exécutez le Planificateur de déploiement Azure Site Recovery dans votre environnement. Pour obtenir des instructions utiles, voir [À propos du planificateur de déploiement Azure Site Recovery pour VMware sur Azure](site-recovery-deployment-planner.md).
2. Déployez un serveur de configuration respectant les [recommandations de taille pour le serveur de configuration](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-configuration-server-and-inbuilt-process-server). Si votre charge de travail de production dépasse 650 machines virtuelles, déployez un autre serveur de configuration.
3. Selon le taux quotidien de modifications de données mesuré, déployez des [serveurs de processus de scale-out](vmware-azure-set-up-process-server-scale.md#download-installation-file) en suivant les [directives concernant la taille](site-recovery-plan-capacity-vmware.md#size-recommendations-for-the-process-server).
4. Si vous pensez que le taux de modifications de données pour une machine virtuelle sur disque dépassera 2 Mbits/s, veillez à utiliser des disques managés Premium. Le Planificateur de déploiement Azure Site Recovery s’exécute pendant une période spécifique. Il se peut que des pics de taux de modifications de données à d’autres moments ne figurent pas dans le rapport.
5. [Définissez la bande passante réseau](site-recovery-plan-capacity-vmware.md#control-network-bandwidth) en fonction de l’objectif de point de récupération à atteindre.
6. Une fois l’infrastructure configurée, activer la récupération d’urgence pour votre charge de travail. Pour savoir comment procéder, voir [Configurer l’environnement source pour la réplication VMware vers Azure](vmware-azure-set-up-source.md).

## <a name="deploy-additional-process-servers"></a>Déployer d’autres serveurs de traitement

Si vous effectuez un scale-out de votre déploiement au-delà de 200 machines sources, ou si votre taux d’activités quotidien dépasse 2 To, vous devez ajouter des serveurs de processus pour gérer le volume du trafic. Nous avons amélioré le produit dans la version 9.24 afin de proposer des [alertes de serveur de traitement](vmware-physical-azure-monitor-process-server.md#process-server-alerts) quand vous configurez un serveur de traitement scale-out. [Configurez le serveur de traitement](vmware-azure-set-up-process-server-scale.md) pour protéger les nouvelles machines sources ou [équilibrer la charge](vmware-azure-manage-process-server.md#move-vms-to-balance-the-process-server-load).

### <a name="migrate-machines-to-use-the-new-process-server"></a>Migrez les machines pour utiliser le nouveau serveur de traitement

1. Sélectionnez **Paramètres** > **Serveurs Site Recovery** . Sélectionnez le serveur de configuration, puis développez **Serveurs de processus** .

    ![Capture d’écran de la boîte de dialogue Serveur de processus](./media/site-recovery-vmware-to-azure/migrate-ps2.png)
2. Cliquez avec le bouton droit sur le serveur de processus en cours d’utilisation, puis sélectionnez **Basculer** .

    ![Capture d’écran de la boîte de dialogue Serveur de configuration](./media/site-recovery-vmware-to-azure/migrate-ps3.png)
3. Dans **Sélectionner un serveur de processus cible** , sélectionnez le nouveau serveur de processus à utiliser. Sélectionnez ensuite les machines virtuelles que le serveur va gérer. Pour obtenir des informations sur le serveur, sélectionnez l’icône Informations. Pour vous aider à prendre des décisions quant à la charge, l’espace moyen requis pour la réplication de chaque machine virtuelle sélectionnée vers le nouveau serveur de processus s’affiche. Cochez la case pour commencer la réplication vers le nouveau serveur de processus.

## <a name="deploy-additional-master-target-servers"></a>Déployer des serveurs maîtres cibles supplémentaires

Dans les scénarios suivants, plusieurs serveurs cibles maîtres sont requis :

*   Vous voulez protéger une machine virtuelle Linux.
*   Le serveur cible maître disponible sur le serveur de configuration n’a pas accès au magasin de données de la machine virtuelle.
*   Le nombre total de disques sur le serveur cible maître (nombre de disques locaux sur le serveur, plus nombre de disques à protéger) est supérieur à 60.

Pour savoir comment ajouter un serveur cible maître pour une machine virtuelle Linux, voir [Installer un serveur cible maître Linux pour la restauration automatique](vmware-azure-install-linux-master-target.md).

Pour ajouter un serveur cible maître pour une machine virtuelle Windows :

1. Accédez à **coffre Recovery Services** > **Infrastructure Site Recovery** > **Serveurs de Configuration** .
2. Sélectionnez le serveur de configuration requis, puis l’option **Serveur cible maître** .

    ![Capture d’écran montrant le bouton Ajouter un serveur cible maître](media/site-recovery-plan-capacity-vmware/add-master-target-server.png)
3. Téléchargez le fichier de configuration unifiée, puis exécutez-le sur la machine virtuelle pour configurer le serveur cible maître.
4. Sélectionnez **Installer le serveur maître cible** > **Suivant** .

    ![Capture d’écran montrant la sélection de l’option Installer le serveur cible maître](media/site-recovery-plan-capacity-vmware/choose-MT.PNG)
5. Sélectionnez l’emplacement d’installation par défaut, puis **Installer** .

     ![Capture d’écran montrant l’emplacement d’installation par défaut](media/site-recovery-plan-capacity-vmware/MT-installation.PNG)
6. Pour inscrire le serveur cible maître auprès du serveur de configuration, sélectionnez **Poursuivre la configuration** .

    ![Capture d’écran montrant le bouton Poursuivre la configuration](media/site-recovery-plan-capacity-vmware/MT-proceed-configuration.PNG)
7. Entrez l’adresse IP du serveur de configuration, puis la phrase secrète. Pour savoir comment générer une phrase secrète, voir [Générer la phrase secrète du serveur de configuration](vmware-azure-manage-configuration-server.md#generate-configuration-server-passphrase). 

    ![Capture d’écran montrant où entrer l’adresse IP et la phrase secrète pour le serveur de configuration](media/site-recovery-plan-capacity-vmware/cs-ip-passphrase.PNG)
8. Sélectionnez **Inscription** . Une fois l’inscription terminée, sélectionnez **Terminer** .

Une fois l’inscription effectuée, le serveur est répertorié sur le portail Azure sous **Coffre Recovery Services** > **Infrastructure Site Recovery** > **Serveurs de configuration** , parmi les serveurs cibles maîtres du serveur de configuration.

 > [!NOTE]
 > Téléchargez la dernière version du [fichier de configuration unifiée du serveur cible maître pour Windows](https://aka.ms/latestmobsvc).

## <a name="next-steps"></a>Étapes suivantes

Télécharger et exécuter le [Planificateur de déploiement Azure Site Recovery](https://aka.ms/asr-deployment-planner).