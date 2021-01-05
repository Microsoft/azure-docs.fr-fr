---
title: Vue d’ensemble de SQL Server sur les machines virtuelles Azure pour Linux | Microsoft Docs
description: Découvrez comment exécuter les éditions complètes de SQL Server sur les machines virtuelles Azure pour Linux. Obtenez des liens directs vers toutes les images de machine virtuelle SQL Server Linux et le contenu associé.
services: virtual-machines-linux
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.service: virtual-machines-sql
ms.topic: overview
ms.workload: iaas-sql-server
ms.date: 04/10/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: c394fe2aa7639d32e5d79bcb22a01151f7666f5b
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96324616"
---
# <a name="overview-of-sql-server-on-azure-virtual-machines-linux"></a>Présentation de SQL Server sur les machines virtuelles Azure (Linux)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
> * [Linux](sql-server-on-linux-vm-what-is-iaas-overview.md)

SQL Server sur les machines virtuelles Azure vous permet d’utiliser des versions complètes de SQL Server dans le cloud sans devoir gérer du matériel local. Les machines virtuelles SQL Server simplifient également les coûts de licence lorsque vous payez à l’utilisation.

Les machines virtuelles Azure sont exécutées dans différentes [régions géographiques](https://azure.microsoft.com/regions/) du monde entier. Elles offrent également un éventail de [tailles de machine](../../../virtual-machines/sizes.md). La galerie d’images de machine virtuelle vous permet de créer une machine virtuelle SQL Server avec la version, l’édition et le système d’exploitation appropriés. Les machines virtuelles constituent ainsi une bonne solution pour différentes charges de travail SQL Server. 

Si vous ne connaissez pas Azure SQL, regardez la vidéo *Vue d’ensemble de SQL Server sur les machines virtuelles Azure*, qui fait partie de notre [série de vidéos Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner) approfondies :
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/SQL-Server-on-Azure-VM-Overview-4-of-61/player]

## <a name="get-started-with-sql-server-vms"></a><a id="create"></a> Démarrer avec les machines virtuelles SQL Server

Pour commencer, choisissez une image de machine virtuelle SQL Server avec la version, l’édition et le système d’exploitation requis. Les sections suivantes fournissent des liens directs vers le portail Azure pour les images de la galerie de machines virtuelles SQL Server.

> [!TIP]
> Pour mieux comprendre les tarifs des images SQL Server, consultez la [page des tarifs des machines virtuelles Linux exécutant SQL Server](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

| Version | Système d’exploitation | Édition |
| --- | --- | --- |
| **SQL Server 2019** | Ubuntu 18.04 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-ubuntu1804sqldev-ARM) | 
| **SQL Server 2019** | Red Hat Enterprise Linux (RHEL) 8 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-rhel8sqldev-ARM)|
| **SQL Server 2019** | SUSE Linux Enterprise Server (SLES) v12 SP5 | [Enterprise](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5enterprise-ARM), [Standard](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5standard-ARM), [Web](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5web-ARM), [Developer](https://ms.portal.azure.com/#create/microsoftsqlserver.sql2019-sles12sp5sqldev-ARM)|
| **SQL Server 2017** | Red Hat Enterprise Linux (RHEL) 7.4 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonRedHatEnterpriseLinux74), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonRedHatEnterpriseLinux74), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonRedHatEnterpriseLinux74), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonRedHatEnterpriseLinux74), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonRedHatEnterpriseLinux74) |
| **SQL Server 2017** | SUSE Linux Enterprise Server (SLES) v12 SP2 |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonSLES12SP2), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonSLES12SP2), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonSLES12SP2), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonSLES12SP2), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonSLES12SP2) |
| **SQL Server 2017** | Ubuntu 16.04 LTS |[Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonUbuntuServer1604LTS), [Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonUbuntuServer1604LTS), [Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonUbuntuServer1604LTS), [Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonUbuntuServer1604LTS), [Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonUbuntuServer1604LTS) |

> [!NOTE]
> Pour voir les images de machines virtuelles SQL Server pour Windows disponibles, consultez [Présentation de SQL Server sur les machines virtuelles Azure (Windows)](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md).

## <a name="installed-packages"></a><a id="packages"></a> Packages installés

Quand vous configurez SQL Server sur Linux, vous installez le package Moteur de base de données et plusieurs packages facultatifs selon vos besoins. Les images de machines virtuelles Linux pour SQL Server installent automatiquement la plupart des packages pour vous. Le tableau suivant montre les packages installés pour chaque distribution.

| Distribution | [Moteur de base de données](/sql/linux/sql-server-linux-setup) | [outils](/sql/linux/sql-server-linux-setup-tools) | [Agent SQL Server](/sql/linux/sql-server-linux-setup-sql-agent) | [Recherche en texte intégral](/sql/linux/sql-server-linux-setup-full-text-search) | [SSIS](/sql/linux/sql-server-linux-setup-ssis) | [Module complémentaire HA](/sql/linux/sql-server-linux-business-continuity-dr) |
|---|---|---|---|---|---|---|
| RHEL | ![RHEL et moteur de base de données](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL et outils](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL et agent SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL et recherche en texte intégral](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL et SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![RHEL et module complémentaire HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) |
| SLES | ![SLES et moteur de base de données](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES et outils](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES et agent SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES et recherche en texte intégral](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![SLES et SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/no.png) | ![SLES et module complémentaire HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png)|
| Ubuntu | ![Ubuntu et moteur de base de données](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu et outils](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu et agent SQL Server](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu et recherche en texte intégral](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu et SSIS](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) | ![Ubuntu et module complémentaire HA](./media/sql-server-on-linux-vm-what-is-iaas-overview/yes.png) |

## <a name="related-products-and-services"></a>Produits et services associés

### <a name="linux-virtual-machines"></a>Machines virtuelles Linux

* [Présentation des machines virtuelles Azure](../../../virtual-machines/linux/overview.md)

### <a name="storage"></a>Stockage

* [Introduction à Microsoft Azure Storage](../../../storage/common/storage-introduction.md)

### <a name="networking"></a>Mise en réseau

* [Présentation du réseau virtuel](../../../virtual-network/virtual-networks-overview.md)
* [Adresses IP dans Azure](../../../virtual-network/public-ip-addresses.md)
* [Créer un nom de domaine complet dans le portail Azure](../../../virtual-machines/create-fqdn.md)

### <a name="sql"></a>SQL

* [Documentation relative à SQL Server sur Linux](/sql/linux)
* [Comparaison Azure SQL Database](../../azure-sql-iaas-vs-paas-what-is-overview.md)

## <a name="next-steps"></a>Étapes suivantes

Démarrer avec les machines virtuelles SQL Server sur Linux :

* [Créer une machine virtuelle SQL Server dans le portail Azure](sql-vm-create-portal-quickstart.md)

Obtenez des réponses aux questions fréquentes à propos des machines virtuelles SQL sur Linux :

* [Forum Aux Questions (FAQ) concernant SQL Server sur les machines virtuelles Azure](frequently-asked-questions-faq.md)