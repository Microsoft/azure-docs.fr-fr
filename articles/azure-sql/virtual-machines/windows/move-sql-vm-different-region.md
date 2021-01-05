---
title: Déplacer une machine virtuelle vers une autre région (Azure Site Recovery)
description: Découvrez comment vous pouvez migrer votre machine virtuelle SQL Server d’une région à l’autre dans Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: aa5bf144-37a3-4781-892d-e0e300913d03
ms.service: virtual-machines-sql
ms.subservice: migration
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 07/30/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 789554121af1c83d9077e6153ca9db01477bde25
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97360150"
---
# <a name="move-a-sql-server-vm-to-another-region-within-azure-with-azure-site-recovery"></a>Déplacer une machine virtuelle SQL Server vers une autre région au sein d’Azure à l’aide d’Azure Site Recovery
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment utiliser Azure Site Recovery pour migrer votre machine virtuelle SQL Server d’une région à l’autre dans Azure. 

Pour déplacer une machine virtuelle SQL Server vers une autre région, vous devez effectuer les opérations suivantes :
1. [Préparation](#prepare-to-move) : Vérifiez que votre machine virtuelle SQL Server source et votre région cible sont correctement préparées pour le déplacement. 
1. [Configuration](#configure-azure-site-recovery-vault) : Le déplacement de votre machine virtuelle SQL Server nécessite qu’elle soit un objet répliqué au sein du coffre Azure Site Recovery. Vous devez ajouter votre machine virtuelle SQL Server au coffre Azure Site Recovery. 
1. [Test](#test-move-process) : La migration de la machine virtuelle SQL Server nécessite son basculement de la région source vers la région cible répliquée. Pour garantir la réussite du processus de déplacement, vous devez d’abord tester si votre machine virtuelle SQL Server peut basculer correctement vers la région cible. Ceci permet de faire apparaître les éventuels problèmes et de les éviter lors du déplacement réel. 
1. [Déplacement](#move-the-sql-server-vm) : Une fois que le test de basculement a réussi et que vous savez que vous pouvez migrer sans problèmes votre machine virtuelle SQL Server, vous pouvez effectuer le déplacement de la machine virtuelle vers la région cible. 
1. [Nettoyage](#clean-up-source-resources) : Pour éviter des coûts, supprimez la machine virtuelle SQL Server du coffre ainsi que les ressources non nécessaires laissées dans le groupe de ressources. 

## <a name="verify-prerequisites"></a>Vérifier la configuration requise 

- Vérifiez que le déplacement depuis votre région source vers votre région cible [est pris en charge](../../../site-recovery/azure-to-azure-support-matrix.md#region-support).  
- Passez en revue l’[architecture et les composants](../../../site-recovery/azure-to-azure-architecture.md) du scénario ainsi que les [limitations et les exigences en matière de prise en charge](../../../site-recovery/azure-to-azure-support-matrix.md). 
- Vérifiez les autorisations du compte. Si vous avez créé votre compte Azure gratuit, vous êtes l’administrateur de votre abonnement. Si vous n’êtes pas administrateur de l’abonnement, collaborez avec l’administrateur pour affecter les autorisations dont vous avez besoin. Pour activer la réplication d’une machine virtuelle et copier les données avec Azure Site Recovery, vous devez avoir les autorisations suivantes : 
    - Autorisations de créer une machine virtuelle. Le rôle prédéfini *Contributeur de machines virtuelles* dispose de ces autorisations, à savoir : 
        - Autorisations de créer une machine virtuelle dans le groupe de ressources sélectionné. 
        - Autorisations de créer une machine virtuelle dans le réseau virtuel sélectionné. 
        - Autorisations d’écrire sur le compte de stockage sélectionné. 
      - Des autorisations pour gérer les opérations Azure Site Recovery. Le rôle *Contributeur Site Recovery* a toutes les autorisations nécessaires pour gérer les opérations Site Recovery dans un coffre Recovery Services.  

## <a name="prepare-to-move"></a>Préparer le déplacement
Préparez la machine virtuelle SQL Server source et la région cible pour le déplacement. 

### <a name="prepare-the-source-sql-server-vm"></a>Préparer la machine virtuelle SQL Server source

- Vérifiez que tous les certificats racines les plus récents se trouvent sur la machine virtuelle SQL Server à déplacer. Si les certificats racines les plus récents ne sont pas présents, les contraintes de sécurité empêcheront la copie des données vers la région cible. 
- Pour les machines virtuelles Windows, installez toutes les mises à jour de Windows les plus récentes afin que tous les certificats racines approuvés s’y trouvent. Dans un environnement déconnecté, suivez les processus standard de Windows Update et de mise à jour des certificats au sein de votre organisation. 
- Pour des machines virtuelles Linux, suivez les instructions fournies par votre distributeur Linux pour obtenir les certificats racines approuvés les plus récents et la dernière liste de révocation de certificats sur la machine virtuelle. 
- N’utilisez pas de proxy d’authentification pour contrôler la connectivité réseau pour les machines virtuelles que vous voulez déplacer. 
- Si la machine virtuelle que vous voulez déplacer n’a pas d’accès à Internet, ou si elle utilise un proxy de pare-feu pour contrôler l’accès sortant, vérifiez la configuration requise. 
- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cela comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les adresses IP publiques. 

### <a name="prepare-the-target-region"></a>Préparer la région cible

- Vérifiez que votre abonnement Azure vous permet de créer des machines virtuelles dans la région cible utilisée pour la reprise d’activité. Contactez le support pour activer le quota requis. 
- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge des machines virtuelles de tailles correspondant à celles de vos machines virtuelles sources. Si vous l’utilisez pour copier des données vers la cible, Site Recovery choisit une machine virtuelle cible de la même taille ou de la taille la plus proche possible. 
- Veillez à créer une ressource cible pour chaque composant identifié dans la topologie du réseau source. Cette étape est importante pour garantir que vos machines virtuelles offrent dans la région cible toutes les fonctionnalités et capacités dont vous disposiez dans la région source. 
    - Azure Site Recovery détecte et crée automatiquement un réseau virtuel quand vous activez la réplication pour la machine virtuelle source. Vous pouvez également créer au préalable un réseau et l’affecter à la machine virtuelle dans le flux de l’utilisateur pour activer la réplication. Vous devez créer manuellement les autres ressources dans la région cible.
- Pour créer les ressources réseau courantes dont vous avez besoin, en fonction de la configuration de la machine virtuelle source, consultez la documentation suivante : 
    - [Groupes de sécurité réseau](../../../virtual-network/tutorial-filter-network-traffic.md) 
    - [Équilibreur de charge](../../../load-balancer/quickstart-load-balancer-standard-internal-portal.md)
    - [Adresse IP publique](../../../virtual-network/virtual-network-public-ip-address.md)
    - Pour tous les autres composants réseau, consultez la [documentation sur les réseaux](../../../virtual-network/virtual-networks-overview.md).
- Créez manuellement un réseau hors production dans la région cible si vous voulez tester la configuration avant d’effectuer le déplacement final vers la région cible. Nous recommandons cette étape car elle garantit une interférence minimale avec le réseau de production. 

## <a name="configure-azure-site-recovery-vault"></a>Configurer le coffre Azure Site Recovery

Les étapes ci-dessous montrent comment copier les données vers la région cible à l’aide d’Azure Site Recovery. Créez le coffre Recovery Services dans n’importe quelle région autre que la région source. 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Choisissez **Créer une ressource** dans le coin supérieur gauche du volet de navigation. 
1. Sélectionnez **Outils Informatique et gestion**, puis sélectionnez **Sauvegarde et Site Recovery**. 
1. Sous l’onglet **De base**, sous **Détails du projet**, créez un groupe de ressources dans la région cible ou sélectionnez-y un groupe de ressources existant. 
1. Sous **Détails de l’instance**, spécifiez un nom pour votre coffre, puis sélectionnez votre **Région** cible dans la liste déroulante. 
1. Sélectionnez **Vérifier + créer** pour créer votre coffre Recovery Services. 
1. Sélectionnez **Tous les services** dans le coin supérieur gauche du volet de navigation puis, dans la zone de recherche, tapez `recovery services`. 
1. (Facultatif) Sélectionnez l’étoile en regard de **Coffres Recovery Services** pour l’ajouter à votre barre de navigation rapide. 
1. Sélectionnez **Coffres Recovery Services**, puis sélectionnez le coffre Recovery Services que vous avez créé. 
1. Dans le volet **Vue d’ensemble**, sélectionnez **Répliquer**. 

   ![Configurer la réplication](./media/move-sql-vm-different-region/configure-replication.png)

1. Sélectionnez **Source**, puis sélectionnez **Azure** comme source. Sélectionnez les valeurs appropriées pour les autres champs de liste déroulante, comme l’emplacement de vos machines virtuelles sources. Seuls les groupes de ressources situés dans la région **Emplacement source** sont visibles dans le champ **Groupe de ressources source**. 
1. Sélectionnez **Machines virtuelles**, puis choisissez les machines virtuelles que vous voulez migrer. Sélectionnez **OK** pour enregistrer votre sélection de machines virtuelles. 
1. Sélectionnez **Paramètres**, puis choisissez votre **Emplacement cible** dans la liste déroulante. Il doit s’agir du groupe de ressources que vous avez préparé précédemment. 
1. Une fois que vous avez personnalisé la réplication, sélectionnez **Créer les ressources cibles** pour créer les ressources au nouvel emplacement. 
1. Une fois la création des ressources terminée, sélectionnez **Activer la réplication** pour démarrer la réplication de votre machine virtuelle SQL Server depuis la source vers la région cible.
1. Vous pouvez vérifier l’état de la réplication en accédant à votre coffre de récupération, en sélectionnant **Éléments répliqués** et en visualisant l’**État** de votre machine virtuelle SQL Server. L’état **Protégé** indique que la réplication est terminée. 

   ![Vérifier l’état de la réplication](./media/move-sql-vm-different-region/check-replication-status.png)

## <a name="test-move-process"></a>Tester le processus de déplacement
Les étapes suivantes vous montrent comment utiliser Azure Site Recovery pour tester le processus de déplacement. 

1. Accédez à votre **Coffre de Recovery Services** dans le [portail Azure](https://portal.azure.com) et sélectionnez **Éléments répliqués**. 
1. Sélectionnez la machine virtuelle SQL Server que vous souhaitez déplacer, vérifiez que l’**Intégrité de la réplication** indique **Sain**,puis sélectionnez **Tester le basculement**. 

   ![Tester le basculement pour votre machine virtuelle](./media/move-sql-vm-different-region/test-failover-of-replicated-vm.png)

1. Dans la page **Tester le basculement**, sélectionnez le point de récupération **Dernier point de cohérence des applications** à utiliser pour le basculement, car il s’agit du seul type d’instantané pouvant garantir la cohérence des données SQL Server. 
1. Sélectionnez le réseau virtuel sous **Réseau virtuel Azure**, puis cliquez sur **OK** pour tester le basculement. 
   
   >[!IMPORTANT]
   > Nous vous recommandons d’utiliser un réseau de machines virtuelles Azure distinct pour le test de basculement. N’utilisez pas le réseau de production qui a été configuré quand vous avez activé la réplication et vers lequel vous souhaitez déplacer vos machines virtuelles. 

1. Pour superviser la progression, accédez à votre coffre, sélectionnez **Travaux Site Recovery** sous **Supervision**, puis sélectionnez le travail **Test de basculement** qui est en cours.

   ![Superviser la progression du test de basculement](./media/move-sql-vm-different-region/monitor-failover-test-job.png)

1. Une fois le test terminé, accédez à **Machines virtuelles** dans le portail, puis examinez la machine virtuelle nouvellement créée. Vérifiez que la machine virtuelle SQL Server est en cours d’exécution, qu’elle est correctement dimensionnée et qu’elle est connectée au réseau approprié. 
1. Supprimez la machine virtuelle créée dans le cadre du test, car l’option **Basculement** reste grisée jusqu’à ce que les ressources du test de basculement soient nettoyées. Revenez au coffre, sélectionnez **Éléments répliqués**, sélectionnez la machine virtuelle SQL Server, puis sélectionnez **Nettoyer le test de basculement**. Notez et enregistrez les observations associées au test dans la section **Notes**, puis cochez la case en regard de l’option **Le test est terminé. Supprimez la ou les machines virtuelles de la migration de test**. Sélectionnez **OK** pour nettoyer les ressources après le test. 

   ![Nettoyer les éléments après le test de basculement](./media/move-sql-vm-different-region/cleanup-test-items.png)

## <a name="move-the-sql-server-vm"></a>Déplacer la machine virtuelle SQL Server 
Les étapes suivantes vous montrent comment déplacer la machine virtuelle SQL Server de votre région source vers votre région cible. 

1. Accédez au coffre **Recovery Services**, sélectionnez **Éléments répliqués**, sélectionnez la machine virtuelle, puis sélectionnez **Basculement**. 

   ![Lancer le basculement](./media/move-sql-vm-different-region/initiate-failover.png)

1. Sélectionnez le point de récupération **Dernier point de cohérence des applications** sous **Point de récupération**. 
1. Cochez la case de l’option **Arrêter la machine avant de commencer le basculement**. Site Recovery tente d’arrêter la machine virtuelle source avant de déclencher le basculement. Le basculement continue même en cas d’échec de l’arrêt. 
1. Sélectionnez **OK** pour démarrer le basculement.
1. Vous pouvez superviser le processus de basculement à partir de la même page **Travaux Site Recovery** que celle que vous avez consultée lors de la supervision du test de basculement dans la section précédente. 
1. Une fois le travail terminé, vérifiez que la machine virtuelle SQL Server apparaît bien dans la région cible comme prévu. 
1. Revenez au coffre, sélectionnez **Éléments répliqués**, sélectionnez la machine virtuelle SQL Server, puis sélectionnez **Valider** pour terminer le processus de déplacement vers la région cible. Attendez la fin de la tâche de validation. 
1. Inscrivez votre machine virtuelle SQL Server dans l’extension SQL IaaS Agent pour activer la gestion des **Machines virtuelles SQL** dans le portail Azure et les fonctionnalités associées à l’extension. Pour plus d’informations, consultez [Inscrire une machine virtuelle SQL Server avec l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md). 

  > [!WARNING]
  > La cohérence des données de SQL Server est garantie seulement avec les instantanés de cohérence des applications. L’instantané **traité en dernier** ne peut pas être utilisé pour le basculement SQL Server, car un instantané de récupération après incident ne peut pas garantir la cohérence des données de SQL Server. 

## <a name="clean-up-source-resources"></a>Nettoyer les ressources sources
Pour éviter des coûts, supprimez la machine virtuelle SQL Server du coffre ainsi que les ressources associées non nécessaires. 

1. Revenez au coffre **Site Recovery**, sélectionnez **Éléments répliqués**, puis sélectionnez la machine virtuelle SQL Server. 
1. Sélectionnez **Désactiver la réplication**. Sélectionnez la raison de la désactivation de la protection, puis sélectionnez **OK** pour désactiver la réplication. 

   >[!IMPORTANT]
   > Il est important d’effectuer cette étape pour éviter d’être facturé pour la réplication Azure Site Recovery. 

1. Si vous n’avez pas l’intention de réutiliser les ressources de la région source, supprimez toutes les ressources réseau pertinentes et les comptes de stockage correspondants. 


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) relatives à SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
* [Guide des tarifs de SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication de SQL Server sur une machine virtuelle Windows](doc-changes-updates-release-notes.md)