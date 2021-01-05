---
title: Migrer vos machines virtuelles Linux vers le stockage Premium Azure avec Azure Site Recovery
description: Migrez vos machines virtuelles existantes vers le stockage Premium Azure à l'aide de Site Recovery. Premium Storage offre une prise en charge très performante et à faible latence des disques pour les charges de travail utilisant beaucoup d'E/S exécutées sur les machines virtuelles Azure.
author: luywang
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/15/2017
ms.author: luywang
ms.subservice: disks
ms.openlocfilehash: c767edca46696bc7d04a1cf101e2bd183f5cf7f9
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91970841"
---
# <a name="migrate-to-premium-storage-by-using-azure-site-recovery"></a>Migrer vers le stockage Premium à l’aide d’Azure Site Recovery

Les [disques SSD Azure Premium](../disks-types.md) offrent une prise en charge des disques haute performance et à faible latence pour les machines virtuelles exécutant des charges de travail qui utilisent beaucoup d’E/S. Ce guide vous aide à migrer vos disques de machine virtuelle d’un compte de stockage standard vers un compte de stockage Premium à l’aide d’[Azure Site Recovery](../../site-recovery/site-recovery-overview.md).

Site Recovery est un service Azure qui participe à votre stratégie de continuité des activités et de récupération d’urgence en orchestrant la réplication des machines virtuelles et des serveurs physiques locaux dans le cloud (Azure) ou dans un centre de données secondaire. Lorsque des pannes se produisent sur votre site principal, vous effectuez un basculement sur le site secondaire pour préserver la disponibilité des applications et des charges de travail. Vous restaurez votre site principal dès lors qu’il retrouve un fonctionnement normal. 

Site Recovery fournit des tests de basculement pour prendre en charge des exercices de récupération d’urgence sans affecter les environnements de production. Vous pouvez exécuter des basculements avec une perte de données minimale (en fonction de la fréquence de réplication) pour les sinistres inattendus. Dans le scénario d’une migration vers le stockage Premium, vous pouvez utiliser le [basculement dans Site Recovery](../../site-recovery/site-recovery-failover.md) pour migrer des disques cibles vers un compte de stockage Premium.

Nous recommandons de migrer vers un stockage Premium à l’aide de Site Recovery, car cette option limite les temps d'arrêt. Elle permet également d’éviter la copie manuelle des disques ainsi que la création manuelle de machines virtuelles. Site Recovery copie systématiquement vos disques et crée de nouvelles machines virtuelles pendant le basculement. 

Site Recovery prend en charge différents types de basculement avec très peu de temps d'arrêt ou aucun temps d'arrêt. Pour planifier votre temps d'arrêt et estimer la perte de données, consultez les [types de basculement dans Site Recovery](../../site-recovery/site-recovery-failover.md). Si vous [préparez des connexions à des machines virtuelles Azure après le basculement](../../site-recovery/vmware-azure-tutorial.md), vous devez pouvoir vous connecter à la machine virtuelle Azure à l'aide de RDP après le basculement.

![Diagramme de la récupération d'urgence][1]

## <a name="azure-site-recovery-components"></a>Composants Azure Site Recovery

Ces composants Site Recovery conviennent à ce scénario de migration :

* Un **serveur de configuration** est une machine virtuelle Azure qui coordonne la communication et gère les processus de réplication et de récupération des données. Sur cette machine virtuelle, vous exécutez un fichier d’installation unique pour installer le serveur de configuration ainsi qu’un composant supplémentaire, appelé serveur de processus, comme passerelle de réplication. En savoir plus sur la [configuration requise du serveur](../../site-recovery/vmware-azure-tutorial.md). Vous configurez le serveur de configuration une seule fois et vous l’utilisez ensuite pour toutes les migrations vers la même région.

* Le **serveur de processus** est une passerelle de réplication qui effectue les actions suivantes : 

  1. Reçoit les données de réplication de machines virtuelles sources.
  2. Optimise les données à l’aide de la mise en cache, la compression et le chiffrement.
  3. Envoie les données à un compte de stockage. 

  Il gère également l’installation Push du service Mobilité sur des machines virtuelles source protégées et assure la détection automatique des machines virtuelles source. Le serveur de processus par défaut est installé sur le serveur de configuration. Vous pouvez déployer des serveurs de processus autonomes supplémentaires pour étendre votre déploiement. En savoir plus sur les [meilleures pratiques de déploiement de serveur de processus](https://azure.microsoft.com/blog/best-practices-for-process-server-deployment-when-protecting-vmware-and-physical-workloads-with-azure-site-recovery/) et le [déploiement de serveurs de processus supplémentaires](../../site-recovery/site-recovery-plan-capacity-vmware.md#deploy-additional-process-servers). Vous configurez le serveur de processus une seule fois et vous l’utilisez ensuite pour toutes les migrations vers la même région.

* Le **service Mobilité** est un composant déployé sur chaque machine virtuelle standard à répliquer. Il enregistre les opérations d'écritures de données sur la machine virtuelle standard et les transmet au serveur de processus. Découvrez plus d’informations sur les [prérequis des machines répliquées](../../site-recovery/vmware-azure-tutorial.md).

Ce graphique montre comment ces composants interagissent :

![Interaction des composants Azure Site Recovery][15]

> [!NOTE]
> Site Recovery ne prend pas en charge la migration des disques des espaces de stockage.

Afin d’obtenir des composants supplémentaires pour d’autres scénarios, consultez [Architecture du scénario](../../site-recovery/vmware-azure-tutorial.md).

## <a name="azure-essentials"></a>Éléments principaux d’Azure

Voici les prérequis Azure pour ce scénario de migration :

* Un abonnement Azure.
* Un compte de stockage Premium Azure pour stocker les données répliquées.
* Un réseau virtuel Azure auquel les machines virtuelles se connectent quand elles sont créées au moment du basculement. Le réseau virtuel Azure doit se trouver dans la même région que celle dans laquelle s’exécute Site Recovery.
* Un compte de stockage Azure standard pour stocker les journaux d’activité de réplication. Il peut s'agir du même compte de stockage que celui des disques de machine virtuelle en cours de migration.

## <a name="prerequisites"></a>Prérequis

* Identifiez les composants de scénario de migration pertinents de la section précédente.
* Planifiez votre temps d’arrêt en vous familiarisant avec le [basculement dans Site Recovery](../../site-recovery/site-recovery-failover.md).

## <a name="setup-and-migration-steps"></a>Configuration et étapes de la migration

Vous pouvez utiliser Site Recovery pour migrer des machines virtuelles IaaS Azure entre différentes régions ou au sein de la même région. Les instructions suivantes ont été adaptées à ce scénario de migration à partir de l’article [Répliquer des machines virtuelles VMware ou des serveurs physiques sur Azure](../../site-recovery/vmware-azure-tutorial.md). Suivez les liens pour obtenir des étapes détaillées en plus des instructions de cet article.

### <a name="step-1-create-a-recovery-services-vault"></a>Étape 1 : Créer un coffre Recovery Services

1. Ouvrez le [portail Azure](https://portal.azure.com).
2. Sélectionnez **Créer une ressource** > **Gestion** > **Backup** and **Site Recovery (OMS)** . Vous pouvez aussi sélectionner **Parcourir** > **Coffre Recovery Services** > **Ajouter**. 
3. Spécifiez une région sur laquelle répliquer les machines virtuelles. Pour effectuer une migration au sein de la même région, sélectionnez la région dans laquelle se trouvent vos machines virtuelles source et les comptes de stockage source. 

### <a name="step-2-choose-your-protection-goals"></a>Étape 2 : Sélectionner vos objectifs en matière de protection 

1. Sur la machine virtuelle sur laquelle vous souhaitez installer le serveur de configuration, ouvrez le [portail Azure](https://portal.azure.com).
2. Accédez à **Coffres Recovery Services** > **Paramètres** > **Site Recovery** > **Étape 1 : préparer l’infrastructure** > **Objectif de protection**.

   ![Accès au volet Objectif de protection][2]

3. Sous **Objectif de protection**, dans la première liste déroulante, sélectionnez **Vers Azure**. Dans la deuxième liste déroulante, sélectionnez **Non virtualisé / autre**, puis cliquez sur **OK**.

   ![Volet Objectif de protection avec des zones remplies][3]

### <a name="step-3-set-up-the-source-environment-configuration-server"></a>Étape 3 : Configurer l’environnement source (serveur de configuration)

1. Téléchargez le **fichier d'installation unifiée Azure Site Recovery** et la clé d’inscription du coffre en accédant aux volets **Préparer l'infrastructure** > **Préparer la source** > **Ajouter un serveur**. 
 
   Vous aurez besoin de la clé d’inscription de coffre pour exécuter le programme d’installation unifiée. Une fois générée, la clé reste valide pendant 5 jours.

   ![Accès au volet Ajouter un serveur][4]

2. Dans le volet **Ajouter un serveur**, ajoutez un serveur de configuration.

   ![Volet Ajouter un serveur avec Serveur de configuration sélectionné][5]

3. Sur la machine virtuelle qui vous sert de serveur de configuration, exécutez l’installation unifiée pour installer le serveur de configuration et le serveur de processus. Vous pouvez [parcourir les captures d’écran](../../site-recovery/vmware-azure-tutorial.md) pour exécuter l’installation. Vous pouvez consulter les captures d’écran ci-dessous pour connaître les étapes spécifiées pour ce scénario de migration.

   1. Dans **Avant de commencer**, sélectionnez **Installer le serveur de configuration et le serveur de traitement**.

      ![Page Avant de commencer][6]

   2. Dans **Inscription**, recherchez et sélectionnez la clé d’inscription que vous avez téléchargée à partir du coffre.

      ![Page Inscription][7]

   3. Dans **Détails de l’environnement**, indiquez si vous voulez répliquer des machines virtuelles VMware. Pour ce scénario de migration, choisissez **non**.

      ![Page Détails de l’environnement][8]

4. Une fois l’installation terminée, effectuez ce qui suit dans la fenêtre **Serveur de configuration Microsoft Azure Site Recovery** :
 
   1. Utilisez l'onglet **Gérer les comptes** pour créer le compte que Site Recovery peut utiliser pour la découverte automatique (Dans le scénario sur la protection des machines physiques, la configuration du compte n’est pas pertinente, mais vous avez besoin d'au moins un compte pour activer l’une des étapes suivantes. Dans ce cas, vous pouvez nommer le compte et définir le mot de passe comme vous le souhaitez). 
   2. Utilisez l'onglet **Inscription du coffre** pour télécharger le fichier d’informations d’identification du coffre.

      ![Onglet Inscription du coffre][9]

### <a name="step-4-set-up-the-target-environment"></a>Étape 4 : Configurer l’environnement cible

Sélectionnez **Préparer l'infrastructure** > **Cible** et spécifiez le modèle de déploiement à utiliser pour les machines virtuelles après le basculement. Vous pouvez choisir **Classique** ou **Resource Manager**, en fonction de votre scénario.

![Volet cible][10]

Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles. 

> [!NOTE]
> Si vous utilisez un compte de stockage Premium pour les données répliquées, vous devez configurer un compte de stockage standard supplémentaire pour stocker les journaux d’activité de réplication.

### <a name="step-5-set-up-replication-settings"></a>Étape 5 : Configurer les paramètres de réplication

Pour vérifier que votre serveur de configuration est correctement associé à la stratégie de réplication que vous créez, suivez [Configurer les paramètres de réplication](../../site-recovery/vmware-azure-tutorial.md).

### <a name="step-6-plan-capacity"></a>Étape 6 : Planifier la capacité

1. Utilisez le [Capacity Planner](../../site-recovery/site-recovery-capacity-planner.md) pour évaluer de manière précise la bande passante réseau, le stockage et les autres conditions requises pour répondre à vos besoins en matière de réplication. 
2. Une fois que vous avez terminé, sélectionnez **Oui, je l’ai fait** dans **Avez-vous effectué une planification de la capacité ?** .

   ![Zone de confirmation d’exécution de la planification de capacité][11]

### <a name="step-7-install-the-mobility-service-and-enable-replication"></a>Étape 7 : Installer le service Mobilité et activer la réplication

1. Vous pouvez choisir d’effectuer une [installation push](../../site-recovery/vmware-azure-tutorial.md) sur vos machines virtuelles sources ou d’y [installer manuellement le service Mobilité](../../site-recovery/vmware-azure-install-mobility-service.md). Vous trouverez les conditions requises de l'installation push et le chemin du programme d’installation manuelle dans le lien fourni. Si vous effectuez une installation manuelle, vous devez peut-être utiliser une adresse IP interne pour rechercher le serveur de configuration.

   ![Page Détails du serveur de configuration][12]

   La machine virtuelle basculée a deux disques temporaires : l’un de la machine virtuelle principale et l’autre créé pendant l'approvisionnement de la machine virtuelle dans la région de récupération. Pour exclure le disque temporaire avant la réplication, installez le service Mobilité avant d’activer la réplication. Pour plus d’informations sur l’exclusion du disque temporaire, consultez [Exclure des disques de la réplication](../../site-recovery/vmware-azure-tutorial.md).

2. Activez la réplication comme suit :
   1. Sélectionnez **Répliquer l’application** > **Source**. Après avoir activé la réplication pour la première fois, sélectionnez **+Répliquer** dans le coffre pour activer la réplication de machines supplémentaires.
   2. À l’étape 1, configurez **Source** comme votre serveur de processus.
   3. À l’étape 2, spécifiez le modèle de déploiement après le basculement, un compte de stockage Premium vers lequel effectuer la migration, un compte de stockage standard pour enregistrer les journaux d’activité et un réseau virtuel vers lequel effectuer le basculement.
   4. À l’étape 3, ajoutez des machines virtuelles protégées par leur adresse IP. (Vous avez peut-être besoin d’une adresse IP interne pour les rechercher.)
   5. À l’étape 4, configurez les propriétés en sélectionnant les comptes que vous avez configurés précédemment sur le serveur de processus.
   6. À l’étape 5, choisissez la stratégie de réplication que vous avez créée précédemment dans la section « Étape 5 : Configurer les paramètres de réplication ».
   7. Sélectionnez **OK**.

   > [!NOTE]
   > Lorsqu’une machine virtuelle Azure est libérée et redémarrée, il n’existe aucune garantie qu’elle aura la même adresse IP. Si l’adresse IP du serveur de configuration/serveur de processus ou des machines virtuelles Azure protégées change, la réplication dans ce scénario peut ne pas fonctionner correctement.

   ![Volet Activer la réplication avec Source sélectionné][13]

Lorsque vous concevez votre environnement de stockage Azure, nous vous recommandons d’utiliser des comptes de stockage distincts pour chaque machine virtuelle dans un groupe à haute disponibilité. Nous vous recommandons de suivre la bonne pratique de la couche de stockage concernant l’[utilisation de plusieurs comptes de stockage pour chaque groupe à haute disponibilité](../manage-availability.md). La distribution de disques de machines virtuelles sur plusieurs comptes de stockage permet d’améliorer la disponibilité du stockage et distribue les E/S sur l’infrastructure de stockage Azure.

Si vos machines virtuelles se trouvent dans un groupe à haute disponibilité au lieu de répliquer les disques de toutes les machines virtuelles dans un compte de stockage, nous vous recommandons vivement de migrer plusieurs machines virtuelles plusieurs fois. De cette façon, les machines virtuelles d’un même groupe à haute disponibilité ne partagent pas le même compte de stockage. Utilisez le volet **Activer la réplication** pour configurer un par un des comptes de stockage de destination pour chaque machine virtuelle.
 
Vous pouvez choisir un modèle de déploiement post-basculement en fonction de vos besoins. Si vous choisissez Azure Resource Manager comme modèle de déploiement après le basculement, vous pouvez basculer une machine virtuelle (Resource Manager) sur une machine virtuelle (Resource Manager), ou vous pouvez basculer une machine virtuelle (Classic) sur une machine virtuelle (Resource Manager).

### <a name="step-8-run-a-test-failover"></a>Étape 8 : Exécuter un test de basculement

Pour vérifier si la réplication est terminée, sélectionnez votre instance Site Recovery, puis **Paramètres** > **Éléments répliqués**. Vous verrez l’état et le pourcentage de votre processus de réplication. 

Une fois la réplication initiale effectuée, exécutez un basculement de test pour valider votre stratégie de réplication. Pour obtenir les étapes détaillées d’un basculement de test, consultez [Exécution d'un basculement de test dans Site Recovery](../../site-recovery/vmware-azure-tutorial.md). 

> [!NOTE]
> Avant d’exécuter un basculement, vérifiez que vos machines virtuelles et la stratégie de réplication répondent aux exigences. Pour plus d’informations sur l’exécution d’un basculement de test, consultez [Basculement de test sur Azure dans Site Recovery](../../site-recovery/site-recovery-test-failover-to-azure.md).

Vous pouvez consulter l’état du basculement de test dans **Paramètres** > **Travaux** > *NOM_DE_VOTRE_PLAN_DE_BASCULEMENT*. Dans le volet, vous voyez la répartition des étapes et des résultats (réussite/échec). Si le basculement de test échoue à une étape, sélectionnez l’étape pour vérifier le message d’erreur. 

### <a name="step-9-run-a-failover"></a>Étape 9 : Exécuter un basculement

Une fois le test de basculement terminé, exécutez un basculement pour migrer vos disques vers le Stockage Premium et répliquer les instances de machines virtuelles. Suivez les étapes détaillées dans [Exécuter un basculement](../../site-recovery/site-recovery-failover.md#run-a-failover). 

Vérifiez que vous sélectionnez **Arrêter les machines virtuelles et synchroniser les dernières données**. Cette option spécifie que Site Recovery doit essayer d’arrêter les machines virtuelles protégées et de synchroniser les données pour que la dernière version des données soit basculée. Si vous ne sélectionnez pas cette option ou si la tentative n’aboutit pas, le basculement est effectué à partir du dernier point de récupération disponible pour la machine virtuelle. 

Site Recovery crée une instance de machine virtuelle dont le type est identique ou similaire à celui d’une machine virtuelle compatible avec le stockage Premium. Vous pouvez vérifier les performances et le prix des différentes instances de machine virtuelle en accédant à [Tarification des machines virtuelles Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) ou à [Tarification des machines virtuelles Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

## <a name="post-migration-steps"></a>Étapes de post-migration

1. **Configurer des machines virtuelles répliquées dans le groupe à haute disponibilité, le cas échéant**. Site Recovery ne prend pas en charge la migration de machines virtuelles avec le groupe à haute disponibilité. En fonction du déploiement de votre machine virtuelle répliquée, effectuez l’une des opérations suivantes :
   * Pour une machine virtuelle créée avec le modèle de déploiement classique : ajoutez la machine virtuelle au groupe à haute disponibilité dans le portail Azure. Pour connaître les détails des étapes, accédez à [Ajout d’une machine virtuelle existante à un groupe à haute disponibilité](/previous-versions/azure/virtual-machines/linux/classic/configure-availability-classic).
   * Pour une machine virtuelle créée avec le modèle de déploiement Resource Manager : enregistrez la configuration de la machine virtuelle, puis supprimez et recréez les machines virtuelles dans le groupe à haute disponibilité. Pour ce faire, utilisez le script figurant dans [Définir le groupe à haute disponibilité de machine virtuelle Azure Resource Manager](https://gallery.technet.microsoft.com/Set-Azure-Resource-Manager-f7509ec4). Avant d’exécuter ce script, vérifiez ses limites et planifiez votre temps d’arrêt.

2. **Supprimer les anciennes machines virtuelles et les anciens disques**. Vérifiez que les disques Premium sont cohérents avec les disques sources et que les nouvelles machines virtuelles exécutent la même fonction que les machines virtuelles sources. Supprimez la machine virtuelle et les disques de vos comptes de stockage sources dans le portail Azure. Si un problème survient et que le disque n’est pas supprimé même après la suppression de la machine virtuelle, consultez [Résoudre les erreurs de suppression de ressources de stockage](../troubleshooting/storage-resource-deletion-errors.md).

3. **Nettoyer l’infrastructure Azure Site Recovery**. Si Site Recovery n’est plus nécessaire, vous pouvez nettoyer son infrastructure. Supprimez les éléments répliqués, le serveur de configuration et la stratégie de récupération, puis supprimez le coffre Azure Site Recovery.

## <a name="troubleshooting"></a>Dépannage

* [Surveiller et résoudre les problèmes de protection pour les machines virtuelles et les serveurs physiques](../../site-recovery/site-recovery-monitor-and-troubleshoot.md)
* [Page de questions Microsoft Q&A pour Microsoft Azure Site Recovery](/answers/topics/azure-site-recovery.html)

## <a name="next-steps"></a>Étapes suivantes

Pour des scénarios propres à la migration de machines virtuelles, consultez les ressources suivantes :

* [Migrer des machines virtuelles Azure entre les comptes de stockage](https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/)
* [Télécharger un disque dur virtuel Linux](upload-vhd.md)
* [Migration de machines virtuelles à partir d’Amazon AWS vers Microsoft Azure](https://channel9.msdn.com/Series/Migrating-Virtual-Machines-from-Amazon-AWS-to-Microsoft-Azure)

Consultez également les ressources suivantes pour en savoir plus sur Azure Storage et les machines virtuelles Azure :

* [Stockage Azure](https://azure.microsoft.com/documentation/services/storage/)
* [Machines virtuelles Azure](https://azure.microsoft.com/documentation/services/virtual-machines/)
* [Sélectionner un type de disque pour les machines virtuelles IaaS](../disks-types.md)

[1]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-1.png
[2]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-2.png
[3]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-3.png
[4]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-4.png
[5]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-5.png
[6]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-6.PNG
[7]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-7.PNG
[8]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-8.PNG
[9]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-9.PNG
[10]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-10.png
[11]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-11.PNG
[12]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-12.PNG
[13]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-13.png
[14]:../site-recovery/media/site-recovery-vmware-to-azure/v2a-architecture-henry.png
[15]:./media/migrate-to-premium-storage-using-azure-site-recovery/migrate-to-premium-storage-using-azure-site-recovery-14.png