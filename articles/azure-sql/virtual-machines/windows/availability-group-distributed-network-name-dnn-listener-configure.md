---
title: Configurer un écouteur DNN pour le groupe de disponibilité
description: Découvrez comment configurer un écouteur de nom de réseau distribué (DNN) pour remplacer votre écouteur de nom de réseau virtuel (VNN) et acheminer le trafic vers votre groupe de disponibilité Always On sur Microsoft SQL Server sur une machine virtuelle Azure.
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
ms.date: 10/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 07ce01304f27ded4e0a566777fcf7027f7a15e4b
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359436"
---
# <a name="configure-a-dnn-listener-for-an-availability-group"></a>Configurer un écouteur DNN pour un groupe de disponibilité
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Avec Microsoft SQL Server sur des machines virtuelles Azure, le nom de réseau distribué (DNN) achemine le trafic vers la ressource en cluster appropriée. Il permet de se connecter plus facilement à un groupe de disponibilité Always On qu’avec l’écouteur de nom de réseau virtuel (VNN), sans avoir à utiliser Azure Load Balancer. 

Cet article vous apprend à configurer un écouteur DNN pour remplacer l’écouteur VNN et à acheminer le trafic vers votre groupe de disponibilité avec Microsoft SQL Server sur des machines virtuelles Azure pour la haute disponibilité et la récupération d’urgence (HADR). 

La fonctionnalité d’écouteur DNN est actuellement uniquement disponible à partir de SQL Server 2019 CU8 sur Windows Server 2016 et versions ultérieures. 

Pour une autre option de connectivité, utilisez plutôt un [écouteur VNN et Azure Load Balancer](availability-group-vnn-azure-load-balancer-configure.md). 

## <a name="overview"></a>Vue d’ensemble

Un écouteur DNN (Distributed Network Name) remplace l’écouteur du groupe de disponibilité VNN (nom de réseau virtuel traditionnel) lorsqu’il est utilisé avec des [groupes de disponibilité Always On sur des machines virtuelles SQL Server](availability-group-overview.md). Cela annule la nécessité d’un Azure Load Balancer pour acheminer le trafic, ce qui simplifie le déploiement, la maintenance et l’amélioration du basculement. 

Utilisez l’écouteur DNN pour remplacer un écouteur VNN existant, ou utilisez-le conjointement avec un écouteur VNN existant afin que votre groupe de disponibilité dispose de deux points de connexion distincts : l’un utilisant le nom de l’écouteur VNN (et le port s’il n’est pas défini par défaut) et l’autre à l’aide du nom et du port de l’écouteur DNN. 

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Microsoft SQL Server 2019 sur CU8 ou version ultérieure, sur Serveur Windows 2016 et versions ultérieures
- Avoir décidé que le nom du réseau distribué est l’option de [connectivité appropriée pour votre solution HADR](hadr-cluster-best-practices.md#connectivity).
- Avoir configuré votre [groupe de disponibilité Always On](availability-group-overview.md). 
- Avoir installé la version la plus récente de [PowerShell](/powershell/azure/install-az-ps). 


## <a name="create-script"></a>Création du script

Utilisez PowerShell pour créer la ressource de nom de réseau distribué (DNN) et l’associer à votre groupe de disponibilité. 

Pour ce faire, procédez comme suit : 

1. Ouvrez un éditeur de texte, tel que le Bloc-notes. 
1. Copiez et collez le script suivant : 

   ```powershell
   param (
      [Parameter(Mandatory=$true)][string]$Ag,
      [Parameter(Mandatory=$true)][string]$Dns,
      [Parameter(Mandatory=$true)][string]$Port
   )
   
   Write-Host "Add a DNN listener for availability group $Ag with DNS name $Dns and port $Port"
   
   $ErrorActionPreference = "Stop"
   
   # create the DNN resource with the port as the resource name
   Add-ClusterResource -Name $Port -ResourceType "Distributed Network Name" -Group $Ag 
   
   # set the DNS name of the DNN resource
   Get-ClusterResource -Name $Port | Set-ClusterParameter -Name DnsName -Value $Dns 
   
   # start the DNN resource
   Start-ClusterResource -Name $Port
   
   
   $Dep = Get-ClusterResourceDependency -Resource $Ag
   if ( $Dep.DependencyExpression -match '\s*\((.*)\)\s*' )
   {
   $DepStr = "$($Matches.1) or [$Port]"
   }
   else
   {
   $DepStr = "[$Port]"
   }
   
   Write-Host "$DepStr"
   
   # add the Dependency from availability group resource to the DNN resource
   Set-ClusterResourceDependency -Resource $Ag -Dependency "$DepStr"
   
   
   #bounce the AG resource
   Stop-ClusterResource -Name $Ag
   Start-ClusterResource -Name $Ag
   ```

1. Enregistrez le script sous la forme d’un fichier de `.ps1`, tel que `add_dnn_listener.ps1`. 


## <a name="execute-script"></a>Exécutez le script

Pour créer l’écouteur DNN, exécutez le script en passant les paramètres pour le nom du groupe de disponibilité, le nom de l’écouteur et le port. 

Par exemple, en supposant que le nom du groupe de disponibilité `ag1`, le nom de l’écouteur `dnnlsnr`et le port de l’écouteur `6789`, procédez comme suit : 

1. Ouvrez un outil d’interface de ligne de commande, tel que l’invite de commandes ou PowerShell. 
1. Accédez à l’emplacement où vous avez enregistré le script `.ps1` , tel que c:\Documents. 
1. Exécutez le script : ```add_dnn_listener.ps1 <ag name> <listener-name> <listener port>```. Par exemple : 

   ```console
   c:\Documents> add_dnn_listener.ps1 ag1 dnnlsnr 6789
   ```

## <a name="verify-listener"></a>Vérifiez l’écouteur

Utilisez SQL Server Management Studio ou Transact-SQL pour confirmer que votre écouteur DNN est correctement créé. 

### <a name="sql-server-management-studio"></a>SQL Server Management Studio

Développez **écouteurs de groupe de disponibilité** dans [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) pour afficher votre écouteur DNN : 

:::image type="content" source="media/availability-group-distributed-network-name-dnn-listener-configure/dnn-listener-in-ssms.png" alt-text="Affichez l’écouteur DNN sous les écouteurs de groupe de disponibilité dans SQL Server Management Studio (SSMS)":::

### <a name="transact-sql"></a>Transact-SQL

Utilisez Transact-SQL pour afficher l’état de l’écouteur DNN : 

```sql
SELECT * FROM SYS.AVAILABILITY_GROUP_LISTENERS
```

La valeur `1` pour `is_distributed_network_name` indique que l’écouteur est un écouteur DNN (Distributed Network Name) : 

:::image type="content" source="media/availability-group-distributed-network-name-dnn-listener-configure/dnn-listener-tsql.png" alt-text="Utilisez sys.availability_group_listeners pour identifier les écouteurs DNN qui ont une valeur de 1 dans is_distributed_network_name":::


## <a name="update-connection-string"></a>Mettre à jour une chaîne de connexion

Mettez à jour les chaînes de connexion pour les applications afin qu’elles se connectent à l’écouteur DNN. Pour garantir une connectivité rapide lors du basculement, ajoutez `MultiSubnetFailover=True` à la chaîne de connexion si le client SQL le prend en charge. 

## <a name="test-failover"></a>Test de basculement

Testez le basculement du groupe de disponibilité pour garantir la fonctionnalité. 

Pour tester le basculement, procédez comme suit : 

1. Connectez-vous à l’écouteur DNN ou à l’un des réplicas à l’aide de [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms). 
1. Développez **Groupe de disponibilité Always On** dans **Explorateur d’objets**. 
1. Cliquez avec le bouton droit sur le groupe de disponibilité, puis choisissez **Type de basculement** pour ouvrir l’**Assistant basculement**. 
1. Suivez les invites pour choisir une cible de basculement et faire basculer le groupe de disponibilité sur un réplica secondaire. 
1. Vérifiez que la base de données est dans un état synchronisé sur le nouveau réplica principal. 
1. (Facultatif) Effectuez une restauration automatique vers le réplica principal d’origine ou un autre réplica secondaire. 

## <a name="test-connectivity"></a>Tester la connectivité

Pour tester la connectivité à votre écouteur DNN, procédez comme suit :

1. Ouvrez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
1. Connectez-vous à votre écouteur DNN. 
1. Ouvrez une nouvelle fenêtre de requête et vérifiez le réplica auquel vous êtes connecté en exécutant `SELECT @@SERVERNAME`. 
1. Faites basculer le groupe de disponibilité sur un autre réplica.
1. Après un laps de temps raisonnable, exécutez `SELECT @@SERVERNAME` pour confirmer que votre groupe de disponibilité est maintenant hébergé sur un autre réplica. 


## <a name="limitations"></a>Limites

- Actuellement, un écouteur DNN pour un groupe de disponibilité est uniquement pris en charge pour Microsoft SQL Server 2019 CU8 et versions ultérieures sur le Serveur Windows 2016 et versions ultérieures. 
- Il peut y avoir plus de points à prendre en compte lorsque vous travaillez avec d’autres fonctionnalités de Microsoft SQL Server et un groupe de disponibilité avec un DNN. Pour plus d’informations, consultez [AG avec l’interopérabilité de DNN](availability-group-dnn-interoperability.md). 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités de SQL Server HADR dans Azure, consultez [Groupes de disponibilité](availability-group-overview.md) et [Instance de cluster de basculement](failover-cluster-instance-overview.md). Vous pouvez également apprendre les [meilleures pratiques](hadr-cluster-best-practices.md) en matière de configuration de votre environnement pour la haute disponibilité et la reprise d’activité. 


