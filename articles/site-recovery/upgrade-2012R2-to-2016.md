---
title: Mettre à niveau Windows Server/System Center VMM 2012 R2 vers Windows Server 2016 - Azure Site Recovery
description: Découvrez comment mettre à niveau des hôtes Windows Server 2012 R2 et des serveurs SCVMM 2012 R2 configurés avec Azure Site Recovery vers Windows Server 2016 et SCVMM 2016.
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.topic: conceptual
ms.service: site-recovery
ms.date: 12/03/2018
ms.author: sharrai
ms.openlocfilehash: b9869ae7dfbf5afd6b8d3b870a2ad4e56fd54c1a
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91250064"
---
# <a name="upgrade-windows-server-serversystem-center-2012-r2-vmm-to-windows-servervmm-2016"></a>Mettre à niveau Windows Server/System Center 2012 R2 VMM vers Windows Server/VMM 2016 

Cet article explique comment mettre à niveau des hôtes Windows Server 2012 R2 et des serveurs SCVMM 2012 R2 configurés avec Azure Site Recovery vers Windows Server 2016 et SCVMM 2016.

Site Recovery contribue à votre stratégie de continuité d’activité et reprise d’activité (BCDR). Le service veille à ce que vos charges de travail de machine virtuelle restent disponibles en cas d’interruption attendue ou inattendue.

> [!IMPORTANT]
> Pour procéder à la mise à niveau d'hôtes Windows Server 2012 R2 déjà configurés pour la réplication avec Azure Site Recovery, vous devez suivre les étapes mentionnées dans ce document. Le choix de toute autre méthode de mise à niveau peut se traduire par des états non pris en charge, et entraîner une interruption de la réplication ou l'impossibilité d'opérer le basculement.


Dans cet article, vous allez apprendre à mettre à niveau les configurations suivantes au sein de votre environnement :

> [!div class="checklist"]
> * **Hôtes Windows Server 2012 R2 non managés par un serveur SCVMM** 
> * **Hôtes Windows Server 2012 R2 managés par un serveur SCVMM 2012 R2 autonome** 
> * **Hôtes Windows Server 2012 R2 managés par un serveur SCVMM 2012 R2 à haute disponibilité**


## <a name="prerequisites--factors-to-consider"></a>Conditions préalables et facteurs à prendre en compte

Avant de procéder à la mise à jour, notez ce qui suit :

- Si vous disposez d'hôtes Windows Server 2012 R2 non managés par un serveur SCVMM ainsi que d'une configuration d'environnement autonome, toute tentative de mise à niveau se traduira par une interruption de la réplication.
- Si vous avez sélectionné « *Ne pas stocker mes clés dans Active Directory sous Gestion distribuée de clés* » lors de l'installation de SCVMM 2012 R2, les mises à niveau n'aboutiront pas.

- Si vous utilisez SCVMM 2012 R2 : 

    - Vérifiez les informations de base de données sur le serveur VMM : **Console VMM** -> **Paramètres** -> **Général** -> **Connexion de base de données**.
    - Vérifiez les comptes de service utilisés pour le service Agent System Center Virtual Machine Manager.
    - Vérifiez que vous disposez d'une sauvegarde de la base de données VMM.
    - Notez le nom de la base de données des serveurs SCVMM impliqués. Pour ce faire, accédez à **Console VMM** -> **Paramètres** -> **Général** -> **Connexion de base de données**.
    - Notez l'ID VMM des serveurs VMM 2012 R2 principal et de récupération. L'ID VMM se trouve dans le registre « HKLM:\SOFTWARE\Microsoft\Microsoft System Center Virtual Machine Manager Server\Setup ».
    - Vérifiez que les nouveaux serveurs SCVMM que vous ajoutez au cluster portent les mêmes noms qu'auparavant. 

- Si vous procédez à une réplication entre deux de vos sites managés des deux côtés par des serveurs SCVMM, veillez à mettre à niveau le côté récupération avant de mettre à niveau le côté principal.
  > [!WARNING]
  > Lors de la mise à niveau d'un serveur SCVMM 2012 R2, sous Gestion distribuée de clés, choisissez de **stocker les clés de chiffrement dans Active Directory**. Choisissez avec soin les paramètres du compte de service et de la gestion distribuée de clés. Selon votre sélection, les données chiffrées telles que les mots de passe des modèles peuvent ne plus être disponibles après la mise à niveau, et peuvent potentiellement affecter la réplication avec Azure Site Recovery.

> [!IMPORTANT]
> Veuillez vous reporter aux [conditions préalables](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#requirements-and-limitations) détaillées dans la documentation SCVMM.

## <a name="windows-server-2012-r2-hosts-which-arent-managed-by-scvmm"></a>Hôtes Windows Server 2012 R2 non managés par un serveur SCVMM 
La liste des étapes mentionnées ci-dessous s'applique à la configuration utilisateur des [hôtes Hyper-V sur Azure](./hyper-v-azure-architecture.md) exécutée en suivant ce [tutoriel](./hyper-v-prepare-on-premises-tutorial.md).

> [!WARNING]
> Comme indiqué dans les conditions préalables, ces étapes s'appliquent uniquement à un environnement en cluster et non à une configuration d'hôte Hyper-V autonome.

1. Suivez les étapes de [mise à niveau de cluster propagée](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process) pour exécuter le processus de mise à niveau de cluster propagée.
2. À chaque nouvel hôte Windows Server 2016 introduit dans le cluster, supprimez d'Azure Site Recovery la référence d'un hôte Windows Server 2012 R2 en procédant comme mentionné [ici]. Il doit s'agir de l'hôte que vous avez choisi de retirer du cluster.
3. Une fois la commande *Update-VMVersion* exécutée pour toutes les machines virtuelles, les mises à niveau sont terminées. 
4. Suivez les étapes décrites [ici](./hyper-v-azure-tutorial.md#set-up-the-source-environment) pour inscrire le nouvel hôte Windows Server 2016 sur Azure Site Recovery. Veuillez noter que le site Hyper-V est déjà actif et qu'il vous suffit d'enregistrer le nouvel hôte dans le cluster. 
5.  Accédez au portail Azure et vérifiez l'état d'intégrité répliqué dans Recovery Services.

## <a name="upgrade-windows-server-2012-r2-hosts-managed-by-stand-alone-scvmm-2012-r2-server"></a>Mettre à niveau des hôtes Windows Server 2012 R2 managés par un serveur SCVMM 2012 R2 autonome
Avant de mettre à niveau vos hôtes Windows Server 2012 R2, vous devez procéder à la mise à niveau de SCVMM 2012 R2 vers SCVMM 2016. Procédez comme suit :

**Mise à niveau d'un serveur SCVMM 2012 R2 autonome vers SCVMM 2016**

1.  Désinstallez le fournisseur ASR en sélectionnant Panneau de configuration -> Programmes -> Programmes et fonctionnalités -> Microsoft Azure Site Recovery, puis en cliquant sur Désinstaller.
2. [Conservez la base de données SCVMM et mettez à niveau le système d'exploitation](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#back-up-and-upgrade-the-operating-system).
3. Dans **Ajout/Suppression de programmes**, sélectionnez **VMM** > **Désinstaller**. b. Sélectionnez **Supprimer des fonctionnalités**, puis **Serveur d'administration VMM et Console VMM**. c. Dans **Options de la base de données**, sélectionnez **Conserver la base de données**. d. Consultez le résumé, puis cliquez sur **Désinstaller**.

4. [Installez VMM 2016](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#install-vmm-2016).
5. Lancez SCVMM et de vérifiez l'état de chaque hôte sous l'onglet **Structures**. Cliquez sur **Actualiser** pour obtenir l'état le plus récent. L'état « Nécessite une attention » doit être affiché. 
17. Installez la dernière version du [fournisseur Microsoft Azure Site Recovery](https://aka.ms/downloaddra) sur le serveur SCVMM.
16. Installez la dernière version de l'[agent Microsoft Azure Recovery Service (MARS)](https://aka.ms/latestmarsagent) sur chacun des hôtes du cluster. Actualisez pour vous assurer que le serveur SCVMM est en mesure d'interroger les hôtes.

**Mise à niveau d'hôtes Windows Server 2012 R2 vers Windows Server 2016**

1. Suivez les étapes mentionnées [ici](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process) pour exécuter le processus de mise à niveau de cluster propagée. 
2. Une fois le nouvel hôte ajouté au cluster, actualisez-le à partir de la console SCVMM pour installer l'agent VMM sur cet hôte mis à jour.
3. Exécutez la commande *Update-VMVersion* pour mettre à jour les versions VM des machines virtuelles. 
4.  Accédez au portail Azure et vérifiez l'état d'intégrité répliqué des machines virtuelles du coffre Recovery Services. 

## <a name="upgrade-windows-server-2012-r2-hosts-are-managed-by-highly-available-scvmm-2012-r2-server"></a>Mettre à niveau des hôtes Windows Server 2012 R2 managés par un serveur SCVMM 2012 R2 à haute disponibilité
Avant de mettre à niveau vos hôtes Windows Server 2012 R2, vous devez procéder à la mise à niveau de SCVMM 2012 R2 vers SCVMM 2016. Les modes de mise à niveau suivants sont pris en charge lors de la mise à niveau des serveurs SCVMM 2012 R2 configurés avec Azure Site Recovery - Mode mixte sans serveurs VMM supplémentaires et mode mixte avec des serveurs VMM supplémentaires.

**Mise à niveau de SCVMM 2012 R2 vers SCVMM 2016**

1.  Désinstallez le fournisseur ASR en sélectionnant Panneau de configuration -> Programmes -> Programmes et fonctionnalités -> Microsoft Azure Site Recovery, puis en cliquant sur Désinstaller.
2. Suivez les étapes mentionnées [ici](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#upgrade-a-standalone-vmm-server) en fonction du mode de mise à niveau que vous souhaitez exécuter.
3. Lancez la console SCVMM et vérifiez l'état de chaque hôte sous l'onglet **Structures**. Cliquez sur **Actualiser** pour obtenir l'état le plus récent. L'état « Nécessite une attention » doit être affiché.
4. Installez la dernière version du [fournisseur Microsoft Azure Site Recovery](https://aka.ms/downloaddra) sur le serveur SCVMM.
5. Installez la dernière version de l'[agent MARS (Microsoft Azure Recovery Service)](https://aka.ms/latestmarsagent) sur chacun des hôtes du cluster. Actualisez pour vous assurer que le serveur SCVMM est en mesure d'interroger les hôtes.


**Mise à niveau d'hôtes Windows Server 2012 R2 vers Windows Server 2016**

1. Suivez les étapes mentionnées [ici](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process) pour exécuter le processus de mise à niveau de cluster propagée.
2. Une fois le nouvel hôte ajouté au cluster, actualisez-le à partir de la console SCVMM pour installer l'agent VMM sur cet hôte mis à jour.
3. Exécutez la commande *Update-VMVersion* pour mettre à jour les versions VM des machines virtuelles. 
4.  Accédez au portail Azure et vérifiez l'état d'intégrité répliqué des machines virtuelles du coffre Recovery Services. 

## <a name="next-steps"></a>Étapes suivantes
Une fois les hôtes mis à niveau, vous pouvez procéder à un [test de basculement](tutorial-dr-drill-azure.md) pour tester l'état de votre réplication et de votre récupération d'urgence.

