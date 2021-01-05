---
title: Configurer l’équilibreur de charge pour l’écouteur VNN AG
description: Apprenez à configurer Azure Load Balancer pour acheminer le trafic vers l’écouteur de nom de réseau virtuel (VNN) pour votre groupe de disponibilité avec Microsoft SQL Server sur des machines virtuelles Azure pour la haute disponibilité et la récupération d’urgence (HADR).
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 2d89759438cb625a0e220af10ab6b287096f6390
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359878"
---
# <a name="configure-load-balancer-for-ag-vnn-listener"></a>Configurer l’équilibreur de charge pour l’écouteur VNN AG
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Sur Machines virtuelles Microsoft Azure, les clusters utilisent un équilibreur de charge pour conserver une adresse IP qui doit se trouver sur un nœud de cluster à la fois. Dans cette solution, l’équilibreur de charge contient l’adresse IP de l’écouteur de nom de réseau virtuel (VNN) pour le groupe de disponibilité Always On. 

Cet article vous apprend à configurer un équilibreur de charge à l’aide du service Azure Load Balancer. L’équilibreur de charge achemine le trafic vers votre [écouteur de groupe de disponibilité](availability-group-overview.md) avec Microsoft SQL Server sur des machines virtuelles Azure pour la haute disponibilité et la récupération d’urgence (HADR). 

Pour une autre option de connectivité pour les clients qui se trouvent sur Microsoft SQL Server 2019 CU8 et versions ultérieures, envisagez plutôt un écouteur [DNN](availability-group-vnn-azure-load-balancer-configure.md) pour simplifier la configuration et améliorer le basculement.  



## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Avoir décidé qu’Azure Load Balancer est [l’option de connectivité appropriée pour votre solution HADR](hadr-cluster-best-practices.md#connectivity).
- Configurez [votre écouteur de groupe de disponibilité](availability-group-overview.md).
- Avoir installé la version la plus récente de [PowerShell](/powershell/azure/install-az-ps). 


## <a name="create-load-balancer"></a>Créer un équilibreur de charge

Utilisez le [portail Azure](https://portal.azure.com) pour créer l’équilibreur de charge :

1. Dans le Portail Azure, accédez au groupe de ressources contenant les machines virtuelles.

1. Sélectionnez **Ajouter**. Recherchez **Équilibreur de charge** sur Place de marché Azure. Sélectionnez **Équilibreur de charge**.

1. Sélectionnez **Create** (Créer).

1. Configurez l’équilibreur de charge à l’aide des valeurs suivantes :

   - **Abonnement**: Votre abonnement Azure.
   - **Groupe de ressources** : Le groupe de ressources qui contient vos machines virtuelles.
   - **Name** : Nom qui identifie l’équilibreur de charge.
   - **Région** : L’emplacement Azure qui contient vos machines virtuelles.
   - **Type** : Public ou privé. Un équilibrage de charge privé est accessible à partir du réseau virtuel. La plupart des applications Azure peut utiliser un équilibrage de charge privé. Si votre application doit accéder à SQL Server directement via Internet, utilisez un équilibrage de charge public.
   - **SKU** : Standard.
   - **Réseau virtuel** : Le même réseau que les machines virtuelles.
   - **Affectation d'adresses IP** : Statique. 
   - **Adresse IP privée** : Adresse IP que vous avez attribuée à la ressource réseau en cluster.

   L’illustration suivante montre l’interface utilisateur **Créer un équilibreur de charge** :

   ![Configurer l’équilibreur de charge](./media/failover-cluster-instance-premium-file-share-manually-configure/30-load-balancer-create.png)
   

## <a name="configure-backend-pool"></a>Configuration du pool principal

1. Revenez au groupe de ressources Azure contenant les machines virtuelles et recherchez le nouvel équilibrage de charge. Vous devrez peut-être actualiser l’affichage du groupe de ressources. Sélectionnez l’équilibreur de charge.

1. Sélectionnez **Pools principaux**, puis **Ajouter**.

1. Associez le pool principal au groupe à haute disponibilité contenant les machines virtuelles.

1. Sous **Configurations IP du réseau cible**, sélectionnez **MACHINE VIRTUELLE** et choisissez les machines virtuelles qui participent en tant que nœuds de cluster. Veillez à inclure toutes les machines virtuelles qui hébergeront la FCI ou le groupe de disponibilité.

1. Sélectionnez **OK** pour créer le pool principal.

## <a name="configure-health-probe"></a>Configurer le probe d’intégrité

1. Dans le volet de l’équilibreur de charge, sélectionnez **Probes d’intégrité**.

1. Sélectionnez **Ajouter**.

1. Dans le volet **Ajouter un probe d’intégrité**, <span id="probe"></span>définissez les paramètres de probe d’intégrité suivants :

   - **Name** : Nom de la sonde d’intégrité.
   - **Protocole** : TCP.
   - **Port** : Port que vous avez créé dans le pare-feu pour le probe d’intégrité [lors de la préparation de la machine virtuelle](failover-cluster-instance-prepare-vm.md#uninstall-sql-server-1). Dans cet article, l’exemple utilise le port TCP `59999`.
   - **Intervalle** : 5 secondes.
   - **Seuil de défaillance sur le plan de l’intégrité** : 2 défaillances consécutives.

1. Sélectionnez **OK**.

## <a name="set-load-balancing-rules"></a>Définir les règles d’équilibrage de charge

1. Dans le volet de l’équilibreur de charge, sélectionnez **Règles d’équilibrage de charge**.

1. Sélectionnez **Ajouter**.

1. Définissez les paramètres de règles d’équilibrage de charge :

   - **Name** : Nom des règles d’équilibrage de charge.
   - **Adresse IP du serveur frontal** : L’adresse IP de la SQL Server instances FCI ou de la ressource réseau en cluster de l’écouteur GA.
   - **Port** : Port TCP SQL Server. Le port d’instance par défaut est 1433.
   - **Port principal** : Même port que la valeur **Port** lorsque vous activez **Adresse IP flottante (retour direct du serveur)** .
   - **Pool principal** : Le nom du pool principal que vous avez configuré précédemment.
   - **Sonde d’intégrité** : La sonde d’intégrité que vous avez configurée précédemment.
   - **Persistance de session** : Aucun.
   - **Délai d’inactivité (minutes)**  : 4.
   - **Adresse IP flottante (retour direct du serveur)**  : activé.

1. Sélectionnez **OK**.

## <a name="configure-cluster-probe"></a>Configurer le probe du cluster

Définissez le paramètre de port de sonde de cluster dans PowerShell.

Pour définir le paramètre de port de sonde de cluster, mettez à jour les variables dans le script suivant avec des valeurs à partir de votre environnement. Supprimez les crochets pointus (`<` et `>`) du script.

```powershell
$ClusterNetworkName = "<Cluster Network Name>"
$IPResourceName = "<SQL Server FCI / AG Listener IP Address Resource Name>" 
$ILBIP = "<n.n.n.n>" 
[int]$ProbePort = <nnnnn>

Import-Module FailoverClusters

Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
```

Le tableau ci-dessous décrit les valeurs que vous devez mettre à jour :


|**Valeur**|**Description**|
|---------|---------|
|`Cluster Network Name`| Nom du cluster de basculement Windows Server pour le réseau. Dans le **Gestionnaire du cluster de basculement** > **Réseaux**, cliquez avec le bouton droit sur le réseau et sélectionnez **Propriétés**. La valeur correcte est sous **Nom** dans l’onglet **Général**.|
|`AG listener IP Address Resource Name`|Nom de la ressource pour l’adresse IP de la FCI SQL Server ou de l’écouteur de groupe de disponibilité. Dans **Gestionnaire du cluster de basculement** > **Rôles**, sous le rôle de l’instance de cluster de basculement SQL Server, sous **Nom du serveur**, cliquez avec le bouton droit sur la ressource d’adresse IP, puis sélectionnez **Propriétés**. La valeur correcte est sous **Nom** dans l’onglet **Général**.|
|`ILBIP`|Adresse IP de l’équilibreur de charge interne (ILB). Cette adresse est configurée dans le portail Azure en tant qu’adresse frontale de l’équilibreur de charge interne. Il s’agit également de l’adresse IP de la FCI SQL Server. Vous pouvez la trouver dans le **Gestionnaire du cluster de basculement** sur la page de propriétés où se trouve également localisé le `<AG listener IP Address Resource Name>`.|
|`nnnnn`|Port de probe que vous avez configuré dans le probe d’intégrité de l’équilibreur de charge. N’importe quel port TCP inutilisé est valide.|
|« SubnetMask »| Masque de sous-réseau pour le paramètre de cluster. Il doit s’agir de l’adresse de diffusion TCP IP : `255.255.255.255`.| 


Après avoir défini la sonde du cluster, vous pouvez voir tous les paramètres de cluster dans PowerShell. Exécutez ce script :

```powershell
Get-ClusterResource $IPResourceName | Get-ClusterParameter
```


## <a name="test-failover"></a>Test de basculement


Testez le basculement de la ressource en cluster pour valider les fonctionnalités du cluster. 


Procédez comme suit :

1. Ouvrez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et connectez-vous à votre écouteur de groupe de disponibilité. 
1. Développez **Groupe de disponibilité Always On** dans **Explorateur d’objets**. 
1. Cliquez avec le bouton droit sur le groupe de disponibilité et sélectionnez **Basculement**. 
1. Suivez les invites de l’Assistant pour basculer le groupe de disponibilité sur un réplica secondaire. 

Le basculement a lieu lorsque les réplicas changent de rôle et sont synchronisés. 


## <a name="test-connectivity"></a>Tester la connectivité

Pour tester la connectivité, connectez-vous à une autre machine virtuelle sur le même réseau virtuel. Ouvrez **SQL Server Management Studio** et connectez-vous à l’écouteur du groupe de disponibilité.

>[!NOTE]
>Si nécessaire, vous pouvez [télécharger SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités de SQL Server HADR dans Azure, consultez [Groupes de disponibilité](availability-group-overview.md) et [Instance de cluster de basculement](failover-cluster-instance-overview.md). Vous pouvez également apprendre les [meilleures pratiques](hadr-cluster-best-practices.md) en matière de configuration de votre environnement pour la haute disponibilité et la reprise d’activité. 



