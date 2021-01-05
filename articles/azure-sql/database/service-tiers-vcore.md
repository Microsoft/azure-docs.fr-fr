---
title: Vue d’ensemble du modèle d’achat vCore
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Le modèle d’achat vCore permet de mettre à l’échelle les ressources de calcul et de stockage indépendamment les unes des autres, d’égaler les performances en local et d’optimiser les coûts pour Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake
ms.date: 09/30/2020
ms.openlocfilehash: b4473ea304176615c35205494f342922869b71ea
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793141"
---
# <a name="vcore-model-overview---azure-sql-database-and-azure-sql-managed-instance"></a>Vue d’ensemble du modèle vCore – Azure SQL Database et SQL Managed Instance 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Le modèle d’achat vCore (« cœur virtuel ») utilisé par Azure SQL Database et Azure SQL Managed Instance offre plusieurs avantages :

- limites de calcul, de mémoire, d’E/S et de stockage plus élevées ;
- contrôle de la génération du matériel pour mieux répondre aux besoins de calcul et de mémoire de la charge de travail ;
- remises sur le tarif [d’Azure Hybrid Benefit (AHB)](../azure-hybrid-benefit.md) et de [l’Instance réservée (RI)](reserved-capacity-overview.md) ;
- plus grande transparence des informations sur le matériel qui fait tourner les calculs, ce qui facilite la planification des migrations à partir de déploiements locaux.

## <a name="service-tiers"></a>Niveaux de service

Les options de niveau de service du modèle vCore sont les suivantes : usage général, critique pour l’entreprise et hyperscale. Le niveau de service définit généralement l’architecture de stockage, les limites d’espace et d’E/S et les options de continuité d’activité liées à la disponibilité et à la reprise d’activité.

|-|**Usage général**|**Critique pour l’entreprise**|**Hyperscale**|
|---|---|---|---|
|Idéal pour|La plupart des charges de travail d’entreprise. Propose des options de calcul et de stockage équilibrées, évolutives et économiques. |Offre aux applications métier la résilience la plus élevée aux défaillances en utilisant plusieurs réplicas isolés et assure les meilleures performances d’E/S par réplica de base de données.|La plupart des charges de travail métier avec des exigences de stockage et d’échelle lecture à haute scalabilité.  Offre une meilleure résilience aux défaillances en autorisant la configuration de plusieurs réplicas de base de données isolés. |
|Stockage|Utilise le stockage à distance.<br/>**Calcul provisionné par SQL Database**  :<br/>5 Go - 4 To<br/>**Calcul serverless** :<br/>5 Go - 3 To<br/>**SQL Managed Instance**  : 32 Go - 8 To |Utilise le stockage SSD local.<br/>**Calcul provisionné par SQL Database**  :<br/>5 Go - 4 To<br/>**SQL Managed Instance**  :<br/>32 Go - 4 To |Croissance automatique et flexible du stockage en fonction des besoins. Prend en charge jusqu’à 100 To de stockage. Utilise le stockage SSD local pour le cache du pool de mémoires tampons local et le stockage de données local. Utilise le stockage distant Azure comme banque de données finale à long terme. |
|IOPS et débit (approximatif)|**SQL Database**  : Consultez les limites de ressources pour les [bases de données uniques](resource-limits-vcore-single-databases.md) et les [pools élastiques](resource-limits-vcore-elastic-pools.md).<br/>**SQL Managed Instance**  : Consultez [Vue d’ensemble des limites de ressources Azure SQL Managed Instance](../managed-instance/resource-limits.md#service-tier-characteristics).|Consultez les limites de ressources pour les [bases de données uniques](resource-limits-vcore-single-databases.md) et les [pools élastiques](resource-limits-vcore-elastic-pools.md).|L’architecture hyperscale est une architecture à plusieurs niveaux avec une mise en cache sur plusieurs niveaux. L’IOPS et le débit réels dépendront de la charge de travail.|
|Disponibilité|1 réplica, réplicas sans échelle lecture|3 réplicas, 1 [réplica avec échelle lecture](read-scale-out.md),<br/>haute disponibilité (HA) redondante interzone|1 réplica en lecture-écriture, plus 0 à 4 [réplicas avec échelle lecture](read-scale-out.md)|
|Sauvegardes|[Stockage géo-redondant avec accès en lecture (RA-GRS)](../../storage/common/geo-redundant-design.md), 7 à 35 jours (7 jours par défaut)|[RA-GRS](../..//storage/common/geo-redundant-design.md), 7 à 35 jours (7 jours par défaut)|Sauvegardes basées sur des instantanés dans le stockage distant Azure. Les restaurations utilisent ces instantanés pour une récupération rapide. Les sauvegardes sont instantanées et n’ont aucun impact sur les performances d’E/S de calcul. Les restaurations sont rapides et ne sont pas des opérations à l’échelle des données (elles durent quelques minutes plutôt que quelques heures ou jours).|
|En mémoire|Non pris en charge|Prise en charge|Non pris en charge|
|||


### <a name="choosing-a-service-tier"></a>Choix d’un niveau de service

Pour plus d’informations sur la sélection d’un niveau de service pour une charge de travail spécifique, voir les articles suivants :

- [Quand choisir le niveau de service Usage général](service-tier-general-purpose.md#when-to-choose-this-service-tier)
- [Quand choisir le niveau de service Critique pour l’entreprise](service-tier-business-critical.md#when-to-choose-this-service-tier)
- [Quand choisir le niveau de service Hyperscale](service-tier-hyperscale.md#who-should-consider-the-hyperscale-service-tier)


## <a name="compute-tiers"></a>Niveaux de calcul

Les options de niveau de calcul du modèle vCore sont les suivantes : provisionné et serverless.


### <a name="provisioned-compute"></a>Calcul provisionné

Le niveau de calcul provisionné fournit une quantité spécifique de ressources de calcul provisionnées en continu, indépendamment de l’activité de la charge de travail, la quantité de ressources provisionnées étant facturée à un tarif horaire fixe.


### <a name="serverless-compute"></a>Calcul serverless

Le [niveau de calcul serverless](serverless-tier-overview.md) met automatiquement à l’échelle les ressources de calcul en fonction de l’activité de la charge de travail, la quantité de ressources utilisées étant facturée à la seconde.



## <a name="hardware-generations"></a>Génération du matériel

Les options de génération du matériel du modèle vCore sont les suivantes : Gen4/5, série M et série Fsv2. La génération du matériel définit généralement les limites de calcul et de mémoire, ainsi que d’autres caractéristiques qui ont un impact sur les performances de la charge de travail.

### <a name="gen4gen5"></a>Gen4/Gen5

- Le matériel Gen4/Gen5, qui fournit des ressources de calcul et de mémoire équilibrées, convient à la plupart des charges de travail de base de données qui n’ont pas besoin de la quantité de mémoire, du nombre de vCore et de la vitesse de vCore unique offerts par les séries Fsv2 et M.

Pour connaître les régions dans lesquelles Gen4/Gen5 est disponible, voir [Disponibilité Gen4/Gen5](#gen4gen5-1).

### <a name="fsv2-series"></a>Série Fsv2

- La série Fsv2 est une option matérielle optimisée pour le calcul qui assure une faible latence du processeur et une fréquence d’horloge élevée pour les charges de travail les plus exigeantes en ressources de processeur.
- En fonction de la charge de travail, la série Fsv2 peut fournir plus de performances de processeur par vCore que Gen5 ; la taille de 72 vCore peut en offrir plus à moindre coût que 80 vCore sur Gen5. 
- Dans la mesure où Fsv2 fournit moins de mémoire et de tempdb par vCore que d’autres matériels, Gen5 ou la série M peuvent être à envisager pour les charges de travail sensibles à ces limites.  

La série Fsv2 n’est pas prise en charge dans le niveau Usage général. Pour connaître les régions dans lesquelles la série Fsv2 est disponible, voir [Disponibilité de la série Fsv2](#fsv2-series-1).


### <a name="m-series"></a>Série M

- La série M est une option matérielle à mémoire optimisée destinée aux charges de travail réclamant plus de mémoire et des limites de calcul supérieures à celles fournies par Gen5.
- La série M fournit 29 Go par vCore et jusqu’à 128 vCores, ce qui multiplie par 8 la limite de mémoire par rapport à Gen5 (presque 4 To).

La série M est prise en charge uniquement dans le niveau Critique pour l’entreprise et ne prend pas en charge la redondance de zone.  Pour connaître les régions dans lesquelles la série M est disponible, voir [Disponibilité de la série M](#m-series-1).

#### <a name="azure-offer-types-supported-by-m-series"></a>Types d’offres Azure pris en charge par la série M

Pour accéder à la série M, l’abonnement doit être un type d’offre payante comme Paiement à l’utilisation ou Contrat Entreprise (EA).  Pour obtenir la liste complète des types d’offres Azure pris en charge par la série M, consultez [offres actuelles sans limites de dépense](https://azure.microsoft.com/support/legal/offer-details).

<!--
To enable M-series hardware for a subscription and region, a support request must be opened. The subscription must be a paid offer type including Pay-As-You-Go or Enterprise Agreement (EA).  If the support request is approved, then the selection and provisioning experience of M-series follows the same pattern as for other hardware generations. For regions where M-series is available, see [M-series availability](#m-series).
-->

### <a name="compute-and-memory-specifications"></a>Spécifications de calcul et de mémoire


|Génération du matériel  |Calcul  |Mémoire  |
|:---------|:---------|:---------|
|Gen4     |- Processeurs Intel® E5-2673 v3 (Haswell) 2,4 GHz<br>- Provisionnement dans la limite de 24 vCore (1 vCore = 1 cœur physique)  |- 7 Go par vCore<br>- Provisionnement dans la limite de 168 Go|
|Gen5     |**Calcul provisionné**<br>- Processeurs Intel® E5-2673 v4 (Broadwell) 2,3 GHz, Intel® SP-8160 (Skylake)\* et Intel® 8272CL (Cascade Lake) 2,5 GHz\*<br>- Provisionnement dans la limite de 80 vCore (1 vCore = 1 hyper-thread)<br><br>**Calcul serverless**<br>- Processeurs Intel® E5-2673 v4 (Broadwell) cadencé à 2,3 GHz et Intel® SP-8160 (Skylake)*<br>- Mise à l’échelle automatique dans la limite de 40 vCores (1 vCore = 1 hyper-thread)|**Calcul provisionné**<br>- 5,1 Go par vCore<br>- Provisionnement dans la limite de 408 Go<br><br>**Calcul serverless**<br>- Mise à l’échelle automatique dans la limite de 24 Go par vCore<br>- Mise à l’échelle automatique dans la limite de 120 Go|
|Série Fsv2     |- Processeurs Intel® 8168 (Skylake)<br>- Fréquence d’horloge turbo tous cœurs prolongée de 3,4 GHz et fréquence d’horloge turbo monocœur maximale de 3,7 GHz<br>- Provisionnement dans la limite de 72 vCores (1 vCore = 1 hyper-thread)|- 1,9 Go par vCore<br>- Provisionnement dans la limite de 136 Go|
|Série M     |- Processeurs Intel® E7-8890 v3 2,5 GHz et Intel® 8280M 2,7 GHz (Cascade Lake)<br>- Provisionnement dans la limite de 128 vCores (1 vCore = 1 hyper-thread)|- 29 Go par vCore<br>- Provisionnement dans la limite de 3,7 To|

\* Dans la vue de gestion dynamique [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database), la génération de matériel pour les bases de données utilisant des processeurs Intel® SP-8160 (Skylake) apparaît comme Gen6, tandis que la génération de matériel pour les bases de données utilisant des processeurs Intel® 8272CL (Cascade Lake) apparaît comme Gen7. Les limites de ressources pour toutes les bases de données Gen5 sont identiques, quel que soit le type de processeur (Broadwell, Skylake ou Cascade Lake).

Pour plus d’informations sur les limites de ressources, voir [Limites de ressources des bases de données uniques (vCore)](resource-limits-vcore-single-databases.md)ou [Limites de ressources des pools élastiques (vCore)](resource-limits-vcore-elastic-pools.md).

### <a name="selecting-a-hardware-generation"></a>Choisir une génération de matériel

Dans le portail Azure, vous pouvez sélectionner la génération de matériel pour une base de données ou un pool dans SQL Database au moment de la création, ou bien modifier la génération de matériel d’une base de données ou un pool existants.

**Pour sélectionner une génération de matériel lors de la création d’une base de données SQL ou d’un pool**

Pour plus d’informations, consultez [Créer une base de données SQL](single-database-create-quickstart.md).

Sous l’onglet **Informations de base** , sélectionnez le lien **Configurer la base de données** dans la section **Calcul + Stockage** , puis le lien **Modifier la configuration**  :

  ![Configurer la base de données](./media/service-tiers-vcore/configure-sql-database.png)

Sélectionnez la génération du matériel souhaitée :

  ![Sélectionner le matériel](./media/service-tiers-vcore/select-hardware.png)


**Pour changer la génération du matériel d’une base de données SQL ou d’un pool existant**

Pour une base de données, sélectionnez le lien **Niveau tarifaire** sur la page Vue d’ensemble :

  ![Modifier le matériel](./media/service-tiers-vcore/change-hardware.png)

Pour un pool, sélectionnez **Configurer** sur la page Vue d’ensemble.

Suivez les étapes indiquées pour modifier la configuration, puis sélectionnez la génération du matériel comme dans la procédure précédente.

**Pour sélectionner une génération de matériel lors de la création d’une instance SQL Managed Instance**

Pour plus d’informations, consultez [Créer une instance SQL Managed Instance](../managed-instance/instance-create-quickstart.md).

Sous l'onglet **Informations de base** , sélectionnez le lien **Configurer la base de données** dans la section **Calcul + Stockage** , puis sélectionnez la génération de matériel souhaitée :

  ![Configurer une instance SQL Managed Instance](./media/service-tiers-vcore/configure-managed-instance.png)
  
**Pour changer la génération du matériel d’une instance SQL Managed Instance existante**

# <a name="the-azure-portal"></a>[Le portail Azure](#tab/azure-portal)

Dans la page SQL Managed Instance, sélectionnez le lien **Niveau tarifaire** situé sous la section Paramètres

![Modifier le matériel d’une instance SQL Managed Instance](./media/service-tiers-vcore/change-managed-instance-hardware.png)

Dans la page Niveau tarifaire, vous pouvez modifier la génération du matériel comme décrit dans les étapes précédentes.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez le script PowerShell suivant :

```powershell-interactive
Set-AzSqlInstance -Name "managedinstance1" -ResourceGroupName "ResourceGroup01" -ComputeGeneration Gen5
```

Pour plus d’informations, consultez la rubrique relative à la commande [Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance).

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Utilisez la commande CLI suivante :

```azurecli-interactive
az sql mi update -g mygroup -n myinstance --family Gen5
```

Pour plus d’informations, consultez la rubrique relative à la commande [az sql mi update](/cli/azure/sql/mi#az-sql-mi-update).

---

### <a name="hardware-availability"></a>Disponibilité matérielle

#### <a name="gen4gen5"></a><a name="gen4gen5-1"></a> Gen4/Gen5

Le matériel Gen4 est [en cours de retrait](https://azure.microsoft.com/updates/gen-4-hardware-on-azure-sql-database-approaching-end-of-life-in-2020/) et n’est plus disponible pour les nouveaux déploiements. Toutes les nouvelles bases de données doivent être déployées sur le matériel Gen5.

Gen5 est disponible dans la plupart des régions du monde.

#### <a name="fsv2-series"></a>Série Fsv2

La série Fsv2 est disponible dans les régions suivantes : Afrique du Sud Nord, Asie Est, Asie Sud-Est, Australie Centre, Australie Centre 2, Australie Est, Australie Sud-Est, Brésil Sud, Canada Centre, Corée Centre, Corée Sud, Europe Nord, Europe Ouest, France Centre, Inde Centre, Royaume-Uni Ouest, Royaume-Uni Sud, USA Est et USA Ouest 2.


#### <a name="m-series"></a>Série M

La série M est disponible dans les régions suivantes : Europe Nord, Europe Ouest, USA Est et USA Ouest 2.
<!--
M-series may also have limited availability in additional regions. You can request a different region than listed here, but fulfillment in a different region may not be possible.

To enable M-series availability in a subscription, access must be requested by [filing a new support request](#create-a-support-request-to-enable-m-series).


##### Create a support request to enable M-series: 

1. Select **Help + support** in the portal.
2. Select **New support request**.

On the **Basics** page, provide the following:

1. For **Issue type**, select **Service and subscription limits (quotas)**.
2. For **Subscription** = select the subscription to enable M-series.
3. For **Quota type**, select **SQL database**.
4. Select **Next** to go to the **Details** page.

On the **Details** page, provide the following:

1. In the **PROBLEM DETAILS** section select the **Provide details** link. 
2. For **SQL Database quota type** select **M-series**.
3. For **Region**, select the region to enable M-series.
    For regions where M-series is available, see [M-series availability](#m-series).

Approved support requests are typically fulfilled within 5 business days.
-->

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez : 
- [Création d’une base de données SQL à l’aide du portail Azure](single-database-create-quickstart.md)
- [Création d’une instance SQL Managed Instance à l’aide du portail Azure](../managed-instance/instance-create-quickstart.md)

Pour plus d’informations sur les tarifs, voir la [page des tarifs Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/).

Pour plus d’informations sur les tailles de calcul et de stockage spécifiques disponibles dans les niveaux de service Usage général et Critique pour l’entreprise, consultez :

- [Limites de ressources vCore pour Azure SQL Database](resource-limits-vcore-single-databases.md).
- [Limites de ressources vCore pour une instance Azure SQL Database mise en pool](resource-limits-vcore-elastic-pools.md).
- [Limites de ressources vCore pour Azure SQL Managed Instance](../managed-instance/resource-limits.md).