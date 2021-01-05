---
title: Répliquer des machines virtuelles Azure exécutant des espaces de stockage direct avec Azure Site Recovery
description: Découvrez comment répliquer des machines virtuelles Azure exécutant des espaces de stockage direct avec Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 01/29/2019
ms.openlocfilehash: f438fadb73f7e3bd25cd7ab9aef0bc46285e30e2
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424828"
---
# <a name="replicate-azure-vms-running-storage-spaces-direct-to-another-region"></a>Répliquer des machines virtuelles Azure exécutant des espaces de stockage direct dans une autre région

Cet article décrit comment activer la récupération d’urgence pour des machines virtuelles Azure exploitant des espaces de stockage directs.

>[!NOTE]
>Seuls des points de récupération offrant une cohérence en cas d’incident sont pris en charge pour les clusters d’espaces de stockage direct.
>

Les [espaces de stockage direct](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct) constituent un stockage à définition logicielle qui offre un moyen de créer des [clusters invités](https://techcommunity.microsoft.com/t5/failover-clustering/bg-p/FailoverClustering) sur Azure.  Un cluster invité dans Microsoft Azure est un cluster de basculement constitué de machines virtuelles IaaS. Il permet à des charges de travail de machine virtuelle hébergées de basculer vers les clusters invités et ainsi d’atteindre une disponibilité en termes de niveau de service pour les applications supérieure à celle qu’une machine virtuelle Azure unique peut fournir. Il s’avère utile dans les scénarios où une machine virtuelle héberge une application critique telle que SQL ou un serveur de fichiers de scale-out.

## <a name="disaster-recovery-with-storage-spaces-direct"></a>Reprise d’activité avec des espaces de stockage direct

Dans un scénario classique, vous pouvez avoir cluster invité de machines virtuelles sur Azure pour accroître la résilience de votre application, tel qu’un serveur de fichiers avec scale-out. Si cela peut fournir à votre application une disponibilité supérieure, vous aimeriez protéger ces applications à l’aide de Site Recovery en cas de défaillance au niveau de la région. Site Recovery réplique les données d’une région Azure vers une autre et active le cluster dans la région de récupération d’urgence en cas de basculement.

Le diagramme ci-dessous montre un cluster de basculement de machine virtuelle Azure à deux nœuds qui utilise des espaces de stockage direct.

![storagespacesdirect](./media/azure-to-azure-how-to-enable-replication-s2d-vms/storagespacedirect.png)


- Deux machines virtuelles Azure dans un cluster de basculement Windows, chacune disposant d’au moins deux disques de données.
- La technologie S2D synchronise les données sur le disque de données et présente le stockage synchronisé en tant que pool de stockage.
- Le pool de stockage se présente comme un volume partagé de cluster au cluster de basculement.
- Le cluster de basculement utilise le fichier volume partagé de cluster pour les lecteurs de données.

**Considérations relatives à la récupération d’urgence**

1. Quand vous configurez un [témoin cloud](/windows-server/failover-clustering/deploy-cloud-witness#CloudWitnessSetUp) pour le cluster, conservez le témoin dans la région de récupération d’urgence.
2. Si vous comptez basculer les machines virtuelles vers le sous-réseau sur une région de récupération d’urgence différente de la région source, l’adresse IP du cluster doit être modifiée après basculement.  Pour modifier l’adresse IP du cluster, vous devez utiliser le [script de plan de récupération](./site-recovery-runbook-automation.md) de Site Recovery.</br>
[Exemple de script](https://github.com/krnese/azure-quickstart-templates/blob/master/asr-automation-recovery/scripts/ASR-Wordpress-ChangeMysqlConfig.ps1) pour exécuter une commande à l’intérieur d’une machine virtuelle à l’aide d’un extension de script personnalisé 

### <a name="enabling-site-recovery-for-s2d-cluster"></a>Activation de Site Recovery pour un cluster S2D :

1. À l’intérieur du coffre Recovery Services, cliquez sur « +replicate »
1. Sélectionnez tous les nœuds du cluster et faites-en une partie d’un [groupe de cohérence multimachine virtuelle](./azure-to-azure-common-questions.md#multi-vm-consistency)
1. Sélectionnez la stratégie de réplication avec une cohérence des applications off* (seule la prise en charge de la cohérence en cas d’incident est disponible)
1. Activer la réplication

   ![Capture d'écran montrant où configurer les paramètres de réplication.](./media/azure-to-azure-how-to-enable-replication-s2d-vms/multivmgroup.png)

2. Accédez aux éléments répliqués pour voir l’état des deux machines virtuelles.
3. Les deux machines virtuelles sont protégées et font partie d’un groupe de cohérence multimachine virtuelle.

   ![Capture d'écran montrant que les machines virtuelles sont protégées et font partie d'un groupe de cohérence multimachine virtuelle.](./media/azure-to-azure-how-to-enable-replication-s2d-vms/storagespacesdirectgroup.PNG)

## <a name="creating-a-recovery-plan"></a>Création d’un plan de récupération
Lors d’un basculement, un plan de récupération prend en charge le séquencement des différents niveaux d’une application multiniveau. La mise en séquence permet d’assurer la cohérence de l’application. Lorsque vous créez un plan de récupération pour une application web multiniveau, suivez les étapes décrites dans [Créer un plan de récupération à l’aide de Site Recovery](site-recovery-create-recovery-plans.md).

### <a name="adding-virtual-machines-to-failover-groups"></a>Ajout de machines virtuelles à des groupes de basculement

1.  Créez un plan de récupération en ajoutant les machines virtuelles.
2.  Cliquez sur « Personnaliser » pour regrouper les machines virtuelles. Par défaut, toutes les machines virtuelles font partie du « Groupe 1 ».


### <a name="add-scripts-to-the-recovery-plan"></a>Ajouter des scripts au plan de récupération
Afin de vous assurer du bon fonctionnement de vos applications, vous pouvez être amené à effectuer certaines opérations sur les machines virtuelles Azure après le basculement ou pendant un test de basculement. Vous pouvez automatiser certaines opérations après le basculement. Par exemple, ici, nous attachons un équilibreur de charge et modifions l’adresse IP du cluster.


### <a name="failover-of-the-virtual-machines"></a>Basculement des machines virtuelles 
Les deux nœuds des machines virtuelles doivent basculer à l’aide du [plan de récupération](./site-recovery-create-recovery-plans.md) de Site Recovery. 

![Protection storagespacesdirect](./media/azure-to-azure-how-to-enable-replication-s2d-vms/recoveryplan.PNG)

## <a name="run-a-test-failover"></a>Exécuter un test de basculement
1.  Dans le portail Azure, sélectionnez votre coffre Recovery Services.
2.  Sélectionnez le plan de récupération que vous avez créé.
3.  Sélectionnez **Test de basculement**.
4.  Pour démarrer le test de basculement, sélectionnez le point de récupération et le réseau virtuel Azure.
5.  Lorsque l’environnement secondaire est opérationnel, procédez aux validations.
6.  Lorsque les validations sont terminées, pour nettoyer l’environnement de basculement, sélectionnez **Nettoyer le test de basculement**.

Pour plus d’informations, consultez [Tester le basculement vers Azure dans Site Recovery](site-recovery-test-failover-to-azure.md).

## <a name="run-a-failover"></a>Exécuter un basculement

1.  Dans le portail Azure, sélectionnez votre coffre Recovery Services.
2.  Sélectionnez le plan de récupération que vous avez créé pour les applications SAP.
3.  Sélectionnez **Basculement**.
4.  Pour démarrer le processus de basculement, sélectionnez le point de récupération.

Pour plus d’informations, consultez [Basculement dans Site Recovery](site-recovery-failover.md).
## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](./azure-to-azure-tutorial-failover-failback.md) sur l’exécution d’une restauration automatique.
