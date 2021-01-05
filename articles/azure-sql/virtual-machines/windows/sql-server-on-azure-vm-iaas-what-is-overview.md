---
title: Présentation de SQL Server sur les machines virtuelles Azure | Microsoft Docs
description: Découvrez comment exécuter des éditions complètes de SQL Server sur les machines virtuelles Azure dans le cloud sans devoir gérer du matériel local.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.assetid: c505089e-6bbf-4d14-af0e-dd39a1872767
ms.service: virtual-machines-sql
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/27/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: b4b1234b88eafedbd03b78241baaa40231a6cbd6
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356801"
---
# <a name="what-is-sql-server-on-azure-virtual-machines-windows"></a>Qu’est-ce que SQL Server sur les machines virtuelles Azure (Windows) ?
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
> * [Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)

[SQL Server sur les machines virtuelles Azure](https://azure.microsoft.com/services/virtual-machines/sql-server/) vous permet d’utiliser des versions complètes de SQL Server dans le cloud sans devoir gérer du matériel local. Les machines virtuelles SQL Server simplifient également les coûts de licence quand vous payez à l’utilisation.

Les machines virtuelles Azure sont exécutées dans différentes [régions géographiques](https://azure.microsoft.com/regions/) du monde entier. Elles offrent également un éventail de [tailles de machine](../../../virtual-machines/sizes.md). La galerie d’images de machine virtuelle vous permet de créer une machine virtuelle SQL Server avec la version, l’édition et le système d’exploitation appropriés. Les machines virtuelles constituent ainsi une bonne solution pour différentes charges de travail SQL Server.

Si vous ne connaissez pas SQL Server sur les machines virtuelles Azure, regardez la vidéo *Vue d’ensemble de SQL Server sur les machines virtuelles Azure*, qui fait partie de notre [série de vidéos Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner) approfondies :
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/SQL-Server-on-Azure-VM-Overview-4-of-61/player]

## <a name="automated-updates"></a>Mises à jour automatisées

SQL Server sur machines virtuelles Azure peut utiliser la fonctionnalité [Mise à jour corrective automatisée](automated-patching.md) afin de planifier une fenêtre de maintenance pour l’installation automatique des mises à jour importantes de Windows et de SQL Server.

## <a name="automated-backups"></a>Sauvegardes automatisées

SQL Server sur machines virtuelles Azure peut tirer parti de la fonctionnalité [Sauvegarde automatisée](automated-backup.md), qui crée à intervalles réguliers des sauvegardes de votre base de données dans le stockage blob. Vous pouvez utiliser cette technique manuellement. Pour plus d’informations, voir [Utilisation du stockage Azure pour la sauvegarde et la restauration de SQL Server](azure-storage-sql-server-backup-restore-use.md).

Azure offre également une solution de sauvegarde de classe entreprise pour les instances SQL Server s’exécutant sur des machines virtuelles Azure. Solution de sauvegarde entièrement managée, elle prend en charge les groupes de disponibilité Always On, la rétention à long terme, la récupération jusqu`à une date et heure, ainsi que des fonctions de gestion et de surveillance centralisées. Pour plus d’informations, consultez [Sauvegarde Azure pour SQL Server dans des machines virtuelles Azure](../../../backup/backup-azure-sql-database.md).
  

## <a name="high-availability"></a>Haute disponibilité

Si vous avez besoin d’une haute disponibilité, pensez à configurer les groupes de disponibilité SQL Server. Cela implique plusieurs instances de SQL Server sur machines virtuelles Azure. Vous pouvez configurer votre solution de haute disponibilité manuellement, ou vous pouvez utiliser des modèles dans le portail Azure pour une configuration automatique. Pour une vue d’ensemble des options de haute disponibilité, consultez [Haute disponibilité et reprise d’activité pour SQL Server sur des machines virtuelles Azure](business-continuity-high-availability-disaster-recovery-hadr-overview.md).

## <a name="performance"></a>Performances

Les machines virtuelles Azure présentent différentes tailles pour répondre aux diverses demandes de charge de travail. Les machines virtuelles SQL Server fournissent également une configuration de stockage automatisée, qui est optimisée pour vos besoins en termes de performances. Pour plus d’informations sur la configuration du stockage des machines virtuelles SQL Server, consultez [Configuration du stockage pour les machines virtuelles SQL Server](storage-configuration.md). Pour ajuster les performances, consultez [Bonnes pratiques relatives aux performances de SQL Server sur les machines virtuelles Azure](performance-guidelines-best-practices.md).

## <a name="get-started-with-sql-server-vms"></a>Démarrer avec les machines virtuelles SQL Server

Pour commencer, choisissez une image de machine virtuelle SQL Server avec la version, l’édition et le système d’exploitation requis. Les sections suivantes fournissent des liens directs vers le portail Azure pour les images de la galerie de machines virtuelles SQL Server.

> [!TIP]
> Pour en savoir plus sur les tarifs des images SQL Server, consultez [Tarifs de SQL Server sur machines virtuelles Azure](pricing-guidance.md). 

### <a name="pay-as-you-go"></a><a id="payasyougo"></a> Paiement à l’utilisation
Le tableau suivant fournit une matrice des images SQL Server de paiement à l’utilisation.

| Version | Système d’exploitation | Édition |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019enterprise), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019standard), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019web), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019sqldev) | 
| **SQL Server 2017** |Windows Server 2016 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseWindowsServer2016), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonWindowsServer2016), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonWindowsServer2016), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonWindowsServer2016), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2EnterpriseWindowsServer2016), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2StandardWindowsServer2016), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2016SP2WebWindowsServer2016), [Express](https://portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP2ExpressWindowsServer2016), [Developer](https://portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP2DeveloperWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2StandardWindowsServer2012R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2WebWindowsServer2012R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2014SP2ExpressWindowsServer2012R2) |
| **SQL Server 2012 SP4** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4StandardWindowsServer2012R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4WebWindowsServer2012R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2012SP4ExpressWindowsServer2012R2) |
| **SQL Server 2008 R2 SP3** |Windows Server 2008 R2|[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3EnterpriseWindowsServer2008R2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3StandardWindowsServer2008R2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3WebWindowsServer2008R2), [Express](https://portal.azure.com/#create/Microsoft.SQLServer2008R2SP3ExpressWindowsServer2008R2) |

Pour afficher les images de machines virtuelles SQL Server sur Linux disponibles, consultez la [présentation de SQL Server sur les machines virtuelles Azure (Linux)](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md).

> [!NOTE]
> Vous pouvez désormais changer de modèle de licence pour les machines virtuelles SQL Server avec paiement à l’utilisation, et utiliser votre propre licence. Pour plus d'informations, consultez [Guide pratique pour changer le modèle de licence d'une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). 

### <a name="bring-your-own-license"></a><a id="BYOL"></a> BYOL (apportez votre propre licence)
Vous pouvez également apporter votre propre licence (modèle BYOL). Dans ce scénario, vous payez uniquement pour la machine virtuelle sans frais supplémentaires pour la gestion de licences SQL Server.  Apporter votre propre licence peut être avantageux sur le long terme pour des charges de travail de production continue. Pour la configuration requise pour utiliser cette option, consultez [Tarification des machines virtuelles SQL Server Azure](pricing-guidance.md#byol).

Pour utiliser votre propre licence, vous pouvez soit convertir une machine virtuelle SQL Server avec paiement à l’utilisation, soit déployer une image avec le préfixe **{BYOL}** . Pour plus d’informations sur le passage du modèle de paiement à l’utilisation au modèle BYOL (apportez votre propre licence), consultez [Guide pratique pour modifier le modèle de licence d’une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). 

| Version | Système d’exploitation | Édition |
| --- | --- | --- |
| **SQL Server 2019** | Windows Server 2019 | [BYOL Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019-byolenterprise), [BYOL Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ws2019-byolstandard)| 
| **SQL Server 2017** |Windows Server 2016 |[BYOL Enterprise](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017EnterpriseWindowsServer2016), [BYOL Standard](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017StandardonWindowsServer2016) |
| **SQL Server 2016 SP2** |Windows Server 2016 |[BYOL Enterprise](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP2EnterpriseWindowsServer2016), [BYOL Standard](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP2StandardWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[BYOL Enterprise](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2014SP2EnterpriseWindowsServer2012R2), [BYOL Standard](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2014SP2StandardWindowsServer2012R2) |
| **SQL Server 2012 SP4** |Windows Server 2012 R2 |[BYOL Enterprise](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2012SP4EnterpriseWindowsServer2012R2), [BYOL Standard](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2012SP4StandardWindowsServer2012R2) |

Avec PowerShell, il est possible de déployer une image plus ancienne de SQL Server qui n'est pas disponible sur le portail Azure. Pour voir toutes les images disponibles avec PowerShell, utilisez la commande suivante :

  ```powershell
  Get-AzVMImageOffer -Location $Location -Publisher 'MicrosoftSQLServer'
  ```

Pour plus d’informations sur le déploiement de machines virtuelles SQL Server à l’aide de PowerShell, consultez [Guide pratique pour provisionner des machines virtuelles SQL Server à l’aide d’Azure PowerShell](create-sql-vm-powershell.md).


### <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle
Lorsque vous avez créé votre machine virtuelle SQL Server, connectez-vous à celle-ci à partir d’applications ou d’outils, tels que SQL Server Management Studio (SSMS). Pour obtenir des instructions, consultez [Se connecter à une machine virtuelle SQL Server sur Azure](ways-to-connect-to-sql.md).

### <a name="migrate-your-data"></a>Migration de vos données
Si vous disposez d’une base de données existante, vous voudrez la déplacer vers la machine virtuelle SQL Server récemment provisionnée. Pour obtenir la liste des options de migration ainsi que de l’aide, voir [Migration d’une base de données vers SQL Server sur une machine virtuelle Azure](migrate-to-vm-from-sql-server.md).

## <a name="create-and-manage-azure-sql-resources-with-the-azure-portal"></a>Créer et gérer des ressources Azure SQL avec le portail Azure

Le portail Azure fournit une page unique sur laquelle vous pouvez gérer [toutes vos ressources Azure SQL](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Sql%2Fazuresql), y compris vos machines virtuelles SQL.

Pour accéder à la page **Ressources Azure SQL**, dans le menu du portail Azure, sélectionnez **Azure SQL**, ou recherchez et sélectionnez **Azure SQL** depuis n’importe quelle page.

![Rechercher Azure SQL](./media/sql-server-on-azure-vm-iaas-what-is-overview/search-for-azure-sql.png)

> [!NOTE]
> Azure SQL permet d’accéder rapidement et facilement à l’ensemble de vos bases de données Azure SQL, pools élastiques, serveurs logiques, instances managées et machines virtuelles. Azure SQL n'est pas un service ou une ressource. 

Pour gérer les ressources existantes, sélectionnez l'élément souhaité dans la liste. Pour créer des ressources Azure SQL, sélectionnez **+ Ajouter**. 

![Créer une ressource Azure SQL](./media/sql-server-on-azure-vm-iaas-what-is-overview/create-azure-sql-resource.png)

Après avoir sélectionné **+ Ajouter**, affichez des informations supplémentaires sur les différentes options en sélectionnant **Afficher les détails** sur une vignette.

![Détails de la vignette Bases de données](./media/sql-server-on-azure-vm-iaas-what-is-overview/sql-vm-details.png)

Pour plus d’informations, consultez :

- [Créer une base de données unique](../../database/single-database-create-quickstart.md)
- [Créer un pool élastique](../../database/elastic-pool-overview.md#creating-a-new-sql-database-elastic-pool-using-the-azure-portal)
- [Créer une instance managée](../../managed-instance/instance-create-quickstart.md)
- [Créer une machine virtuelle SQL Server](sql-vm-create-portal-quickstart.md)

## <a name="sql-server-vm-image-refresh-policy"></a><a id="lifecycle"></a> Stratégie d’actualisation des images de machine virtuelle SQL Server
Azure conserve uniquement une image de machine virtuelle pour chaque combinaison de système d’exploitation, de version et d’édition prise en charge. Cela signifie qu’au fil du temps, les images sont actualisées et les images plus anciennes sont supprimées. Pour plus d’informations, consultez la section **Images** des [FAQ relatifs aux machines virtuelles SQL Server](frequently-asked-questions-faq.md#images).

## <a name="customer-experience-improvement-program-ceip"></a>Programme d’amélioration du produit (CEIP)
Le Programme d’amélioration du produit est activé par défaut. Il transmet régulièrement des rapports à Microsoft afin de contribuer à améliorer SQL Server. Aucune tâche de gestion n’est requise dans le cadre de ce programme, sauf si vous souhaitez le désactiver après sa mise en service. Vous pouvez personnaliser ou désactiver le CEIP en vous connectant à la machine virtuelle avec le Bureau à distance. Exécutez ensuite l’utilitaire **Rapports d’erreurs et d’utilisation SQL Server** . Suivez les instructions pour désactiver la création de rapports. Pour plus d’informations sur la collecte de données, consultez la [Déclaration de confidentialité de SQL Server](/sql/sql-server/sql-server-privacy).

## <a name="related-products-and-services"></a>Produits et services associés
### <a name="windows-virtual-machines"></a>Machines virtuelles Windows
* [Présentation des machines virtuelles Azure](../../../virtual-machines/windows/overview.md)

### <a name="storage"></a>Stockage
* [Introduction à Microsoft Azure Storage](../../../storage/common/storage-introduction.md)

### <a name="networking"></a>Mise en réseau
* [Présentation du réseau virtuel](../../../virtual-network/virtual-networks-overview.md)
* [Adresses IP dans Azure](../../../virtual-network/public-ip-addresses.md)
* [Créer un nom de domaine complet dans le portail Azure](../../../virtual-machines/create-fqdn.md)

### <a name="sql"></a>SQL
* [Documentation SQL Server](/sql/index)
* [Comparaison Azure SQL Database](../../azure-sql-iaas-vs-paas-what-is-overview.md)

## <a name="next-steps"></a>Étapes suivantes

Démarrez avec SQL Server sur des machines virtuelles Azure :

* [Créer une machine virtuelle SQL Server dans le portail Azure](sql-vm-create-portal-quickstart.md)

Obtenez des réponses aux questions fréquentes à propos des machines virtuelles SQL :

* [Forum Aux Questions (FAQ) concernant SQL Server sur les machines virtuelles Azure](frequently-asked-questions-faq.md)

Afficher les architectures de référence pour l’exécution d’applications multiniveau sur SQL Server dans IaaS

* [Application multiniveau Windows sur Azure avec SQL Server](/azure/architecture/reference-architectures/n-tier/n-tier-sql-server)
* [Exécuter une application multiniveau dans plusieurs régions Azure pour une haute disponibilité](/azure/architecture/reference-architectures/n-tier/multi-region-sql-server)