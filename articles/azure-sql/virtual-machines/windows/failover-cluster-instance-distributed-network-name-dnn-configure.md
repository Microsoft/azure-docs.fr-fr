---
title: Configurer DNN pour l’instance de cluster de basculement
description: Découvrez comment configurer un nom de réseau distribué (DNN) pour acheminer le trafic vers votre serveur SQL sur une instance de cluster de basculement de machine virtuelle Azure.
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
ms.openlocfilehash: 8549592ace00e712929ebc76045a32531b9db659
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358314"
---
# <a name="configure-a-dnn-for-failover-cluster-instance"></a>Configurer un DNN pour une instance de cluster de basculement
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Sur les machines virtuelles Azure, le nom de réseau distribué (DNN) achemine le trafic vers la ressource en cluster appropriée. Il offre un moyen plus simple de se connecter à l’instance de cluster de basculement (FCI) de Microsoft SQL Server que le nom de réseau virtuel (VNN), sans avoir à utiliser Azure Load Balancer. 

Cet article vous apprend à configurer une ressource DNN pour acheminer le trafic vers votre instance de cluster de basculement avec Microsoft SQL Server sur des machines virtuelles Azure pour la haute disponibilité et la récupération d’urgence (HADR). 

La fonctionnalité DNN est actuellement uniquement disponible sur Microsoft SQL Server 2019 CU2 et versions ultérieures, et sur le Serveur Windows 2016 et versions ultérieures. 

Pour une autre option de connectivité, utilisez plutôt un nom de réseau virtuel [et Azure Load Balancer](failover-cluster-instance-vnn-azure-load-balancer-configure.md). 

## <a name="overview"></a>Vue d’ensemble

Le nom de réseau distribué (DNN) remplace le nom de réseau virtuel (VNN) comme point de connexion lorsqu’il est utilisé avec un [une instance de cluster de basculement Always On sur les machines virtuelles SQL Server](failover-cluster-instance-overview.md). Cela annule la nécessité pour un Azure Load Balancer d’acheminer le trafic vers le VNN, ce qui simplifie le déploiement, la maintenance et l’amélioration du basculement. 

Avec un déploiement FCI, le VNN existe toujours, mais le client se connecte au nom DNS DNN au lieu du nom du VNN. 

## <a name="prerequisites"></a>Prérequis 

Avant d’effectuer les étapes décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Microsoft SQL Server 2019 sur CU2 ou version ultérieure, sur Serveur Windows 2016 et versions ultérieures
- Avoir décidé que le nom du réseau distribué est l’option de [connectivité appropriée pour votre solution HADR](hadr-cluster-best-practices.md#connectivity).
- Avoir configuré vos [instances de cluster de basculement](failover-cluster-instance-overview.md). 
- Avoir installé la version la plus récente de [PowerShell](/powershell/azure/install-az-ps). 

## <a name="create-dnn-resource"></a>Créer une ressource DNN 

La ressource DNN est créée dans le même groupe de clusters que l’instance FCI SQL Server. Utilisez PowerShell pour créer la ressource DNN dans le groupe de clusters FCI. 

La commande PowerShell suivante ajoute une ressource DNN dont le nom de ressource est `<dnnResourceName>` au groupe de clusters de l’instance FCI SQL Server. Le nom de ressource est utilisé pour identifier une ressource de manière unique. Utilisez-en un qui a un sens pour vous et est unique dans le cluster. Le type de ressource doit être `Distributed Network Name`. 

La valeur de `-Group` doit être le nom du groupe de clusters qui correspond à l’instance FCI SQL Server où vous souhaitez ajouter le nom de réseau distribué. Pour une instance par défaut, le format standard est `SQL Server (MSSQLSERVER)`. 


```powershell
Add-ClusterResource -Name <dnnResourceName> `
-ResourceType "Distributed Network Name" -Group "<WSFC role of SQL server instance>"
```

Par exemple, pour créer votre ressource DNN `dnn-demo` pour une instance FCI SQL Server par défaut, utilisez la commande PowerShell suivante :

```powershell
Add-ClusterResource -Name dnn-demo `
-ResourceType "Distributed Network Name" -Group "SQL Server (MSSQLSERVER)"

```

## <a name="set-cluster-dnn-dns-name"></a>Définir le nom DNS de la ressource DNN

Définissez le nom DNS de la ressource DNN dans le cluster. Le cluster utilise ensuite cette valeur pour acheminer le trafic vers le nœud qui héberge actuellement l’instance FCI SQL Server. 

Les clients utilisent le nom DNS pour se connecter à l’instance FCI SQL Server. Vous pouvez choisir une valeur unique. Ou, si vous disposez déjà d’une instance FCI existante et que vous ne souhaitez pas mettre à jour les chaînes de connexion du client, vous pouvez configurer la ressource DNN pour utiliser le nom de réseau virtuel (VNN) déjà actuellement utilisé par les clients. Pour ce faire, vous devez [renommer le VNN](#rename-the-vnn) avant de définir le DNN dans le nom DNS.

Utilisez cette commande pour définir le nom DNS de votre ressource DNN : 

```powershell
Get-ClusterResource -Name <dnnResourceName> | `
Set-ClusterParameter -Name DnsName -Value <DNSName>
```

La valeur `DNSName` est celle utilisée par les clients pour se connecter à l’instance FCI SQL Server. Par exemple, pour que les clients se connectent à `FCIDNN`, utilisez la commande PowerShell suivante :

```powershell
Get-ClusterResource -Name dnn-demo | `
Set-ClusterParameter -Name DnsName -Value FCIDNN
```

Les clients entrent à présent `FCIDNN` dans leur chaîne de connexion lors de la connexion à l’instance FCI SQL Server. 

   > [!WARNING]
   > Ne supprimez pas le nom de réseau virtuel (VNN) actuel, car il s’agit d’un composant nécessaire de l’infrastructure FCI. 


### <a name="rename-the-vnn"></a>Renommez le VNN 

Si vous avez un nom de réseau virtuel existant et que vous voulez que les clients continuent à utiliser cette valeur pour se connecter à l’instance FCI SQL Server, vous devez renommer le VNN actuel avec une valeur d’espace réservé. Après avoir renommé le VNN actuel, vous pouvez lui affecter la valeur du nom DNS. 

Certaines restrictions s’appliquent au changement de nom du VNN. Pour plus d’informations, consultez [Changement de nom d’une instance FCI](/sql/sql-server/failover-clusters/install/rename-a-sql-server-failover-cluster-instance).

Si l’utilisation du VNN actuel n’est pas nécessaire pour votre entreprise, ignorez cette section. Une fois que vous avez renommé VNN, [définissez le nom DNS de la ressource DNN du cluster](#set-cluster-dnn-dns-name). 

   
## <a name="set-dnn-resource-online"></a>Définir la ressource DNN en ligne

Une fois que votre ressource DNN a un nom approprié et que vous avez défini la valeur du nom DNS dans le cluster, utilisez PowerShell pour définir la ressource DNN en ligne dans le cluster : 

```powershell
Start-ClusterResource -Name <dnnResourceName>
```

Par exemple, pour démarrer votre ressource DNN `dnn-demo`, utilisez la commande PowerShell suivante : 

```powershell
Start-ClusterResource -Name dnn-demo
```

## <a name="configure-possible-owners"></a>Configurez les propriétaires possibles

Par défaut, le cluster lie le nom DNS de la ressource DNN à tous les nœuds du cluster. Toutefois, les nœuds du cluster qui ne font pas partie de l’instance FCI SQL Server doivent être exclus de la liste des propriétaires possibles de la ressource DNN. 

Pour mettre à jour les propriétaires possibles, procédez comme suit :

1. Accédez à votre ressource DNN dans le gestionnaire du cluster de basculement. 
1. Cliquez avec le bouton droit sur la ressource DNN et sélectionnez **Propriétés**. 

   :::image type="content" source="media/hadr-distributed-network-name-dnn-configure/fci-dnn-properties.png" alt-text="Menu contextuel de la ressource DNN avec la commande Propriétés mise en surbrillance.":::

1. Désactivez la case à cocher pour les nœuds qui ne font pas partie de l’instance du cluster de basculement. La liste des propriétaires possibles de la ressource DNN doit correspondre à la liste des propriétaires possibles de la ressource d’instance SQL Server. Par exemple, en supposant que Data3 ne participe pas à l’instance FCI, l’image suivante est un exemple de suppression de Data3 de la liste des propriétaires possibles de la ressource DNN : 

   :::image type="content" source="media/hadr-distributed-network-name-dnn-configure/clear-check-for-nodes-not-in-fci.png" alt-text="Désactivez la case à cocher en regard des nœuds qui ne participent pas à l’instance FCI pour les propriétaires possibles de la ressource DNN":::

1. Sélectionnez **OK** pour enregistrer vos paramètres. 


## <a name="restart-sql-server-instance"></a>Redémarrez l’instance SQL Server 

Utilisez le gestionnaire du cluster de basculement pour redémarrer l’instance SQL Server. Procédez comme suit :

1. Accédez à votre ressource SQL Server dans le gestionnaire du cluster de basculement.
1. Cliquez avec le bouton droit sur la ressource SQL Server, puis mettez-la hors connexion. 
1. Une fois que toutes les ressources associées sont hors connexion, cliquez avec le bouton droit sur la ressource SQL Server, puis remettez-la en ligne. 

## <a name="update-connection-string"></a>Mettre à jour une chaîne de connexion

Pour garantir une connectivité rapide lors du basculement, ajoutez `MultiSubnetFailover=True` à la chaîne de connexion si la version du client SQL est antérieure à 4.6.1. 

En outre, si le DNN n’utilise pas le VNN d’origine, les clients SQL qui se connectent au SQL Server FCI devront mettre à jour leur chaîne de connexion avec le nom DNS DNN. Pour éviter cette exigence, vous pouvez mettre à jour la valeur du nom DNS pour qu’elle corresponde au nom du VNN. Toutefois, vous devez d’abord [remplacer le VNN existant par un espace réservé](#rename-the-vnn). 

## <a name="test-failover"></a>Test de basculement


Testez le basculement de la ressource en cluster pour valider les fonctionnalités du cluster. 


Pour tester le basculement, procédez comme suit : 

1. Connectez-vous à l’un des nœuds de cluster SQL Server avec RDP.
1. Ouvrez le **Gestionnaire du cluster de basculement**. Sélectionnez **Rôles**. Notez le nœud qui possède le rôle de l’instance de cluster de basculement SQL Server.
1. Cliquez avec le bouton droit sur le rôle de l’instance de cluster de basculement SQL Server. 
1. Sélectionnez **Déplacer**, puis sélectionnez **Meilleur nœud possible**.

Le **Gestionnaire du cluster de basculement** présente le rôle et ses ressources hors connexion. Ensuite, les ressources sont déplacées et remises en ligne sur l’autre nœud.

## <a name="test-connectivity"></a>Tester la connectivité

Pour tester la connectivité, connectez-vous à une autre machine virtuelle sur le même réseau virtuel. Ouvrez **SQL Server Management Studio** et connectez-vous à l’instance FCI SQL Server à l’aide du nom DNS de la ressource DDN.

Si nécessaire, vous pouvez [télécharger SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).


## <a name="avoid-ip-conflict"></a>Éviter les conflits d’IP

Il s’agit d’une étape facultative pour empêcher l’attribution de l’adresse IP virtuelle (VIP) utilisée par la ressource FCI à une autre ressource dans Azure en tant que doublon. 

Bien que les clients utilisent désormais le DNN pour se connecter à l’instance de cluster de basculement (FCI) SQL Server, le nom du réseau virtuel (VNN) et l’adresse IP virtuelle ne peuvent pas être supprimés, car ils sont des composants nécessaires de l’infrastructure FCI. Toutefois, étant donné qu’il n’y a plus d’équilibrage de charge qui réserve l’adresse IP virtuelle dans Azure, il existe un risque qu’une autre ressource sur le réseau virtuel reçoive la même adresse IP que l’adresse IP virtuelle utilisée par la FCI. Cela peut entraîner un problème de conflit d’adresse IP en double. 

Configurez une adresse APIPA ou une carte réseau dédiée pour réserver l’adresse IP. 

### <a name="apipa-address"></a>Adresse APIPA

Pour éviter d’utiliser des adresses IP dupliquées, configurez une adresse APIPA (également appelée adresse lien-local). Pour cela, exécutez la commande suivante :

```powershell
Get-ClusterResource "virtual IP address" | Set-ClusterParameter 
    –Multiple @{"Address”=”169.254.1.1”;”SubnetMask”=”255.255.0.0”;"OverrideAddressMatch"=1;”EnableDhcp”=0}
```

Dans cette commande, « virtual IP address » est le nom de la ressource d’adresse VIP en cluster et « 169.254.1.1 » est l’adresse APIPA choisie pour l’adresse IP virtuelle. Choisissez l’adresse qui convient le mieux à votre entreprise. Définissez `OverrideAddressMatch=1` pour permettre à l’adresse IP de se trouver sur n’importe quel réseau, y compris l’espace d’adressage APIPA. 

### <a name="dedicated-network-adapter"></a>Carte réseau dédiée

Vous pouvez également configurer une carte réseau dans Azure pour réserver l’adresse IP utilisée par la ressource d’adresse IP virtuelle. Toutefois, cela consomme l’adresse dans l’espace d’adressage du sous-réseau, et il existe une surcharge supplémentaire pour s’assurer que la carte réseau n’est pas utilisée à d’autres fins.

## <a name="limitations"></a>Limites

- Actuellement, un DNN avec FCI est pris en charge uniquement pour Microsoft SQL Server 2019 CU2 et versions ultérieures sur le Serveur Windows 2016 et versions ultérieures. 
- Il peut y avoir plus de points à prendre en compte lorsque vous travaillez avec d’autres fonctionnalités de SQL Server et une instance FCI avec une ressource DNN. Pour plus d’informations, consultez [Instance FCI avec interopérabilité DNN](failover-cluster-instance-dnn-interoperability.md). 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités de SQL Server HADR dans Azure, consultez [Groupes de disponibilité](availability-group-overview.md) et [Instance de cluster de basculement](failover-cluster-instance-overview.md). Vous pouvez également apprendre les [meilleures pratiques](hadr-cluster-best-practices.md) en matière de configuration de votre environnement pour la haute disponibilité et la reprise d’activité. 


