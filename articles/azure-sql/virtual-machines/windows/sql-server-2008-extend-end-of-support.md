---
title: Étendre la prise en charge de SQL Server 2008 et 2008 R2
description: Étendez la prise en charge de SQL Server 2008 et SQL Server 2008 R2 en procédant à la migration de votre instance SQL Server vers Azure, ou en achetant une prise en charge étendue pour conserver les instances en local.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/08/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 4c25adc16d14b4a5fb72ae0103ca05b193b40499
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359164"
---
# <a name="extend-support-for-sql-server-2008-and-sql-server-2008-r2-with-azure"></a>Étendre la prise en charge pour SQL Server 2008 et SQL Server 2008 R2 avec Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

SQL Server 2008 et SQL Server 2008 R2 ont atteint la [fin de leur cycle de vie de prise en charge](https://www.microsoft.com/sql-server/sql-server-2008). Sachant que beaucoup de clients utilisent encore les deux versions, nous proposons plusieurs options pour leur permettre de continuer à bénéficier de la prise en charge. Vous pouvez migrer vos instances SQL Server locales vers les machines virtuelles (VM) Azure, migrer vers Azure SQL Database ou demeurer en local et acheter des mises à jour de sécurité étendues.

À la différence d’une instance managée, la migration vers une machine virtuelle Azure ne nécessite pas de faire certifier de nouveau vos applications. Et contrairement à la solution de rester en local, vous recevrez des correctifs de sécurité étendus gratuits en migrant vers une machine virtuelle Azure.

Le reste de cet article présente des points à prendre en considération pour la migration de votre instance SQL Server vers une machine virtuelle Azure.

Pour plus d’informations sur la fin des options de support technique, consultez [Fin du support](/sql/sql-server/end-of-support/sql-server-end-of-life-overview).

## <a name="provisioning"></a>Approvisionnement

Une image de paiement à l’utilisation **SQL Server 2008 R2 sur Windows Server 2008 R2** est disponible sur la Place de marché Azure.

Les clients qui sont sur SQL Server 2008 devront effectuer une installation automatique ou une mise à niveau vers SQL Server 2008 R2. De la même façon, les clients sur Windows Server 2008 devront déployer leur machine virtuelle à partir d’un disque dur virtuel personnalisé ou effectuer une mise à niveau vers Windows Server 2008 R2.

Les images déployées via la Place de marché Azure sont fournies avec l’extension IaaS SQL préinstallée. L’extension IaaS SQL est obligatoire pour la gestion des licences flexibles et la mise à jour corrective automatisée. Les clients qui déploient des machines virtuelles installées automatiquement doivent installer manuellement l’extension IaaS SQL. L’extension IaaS SQL n’est pas prise en charge sur Windows Server 2008.

> [!NOTE]
> Bien que les panneaux **Créer** et **Gérer** de SQL Server fonctionnent avec l’image SQL Server 2008 R2 sur le portail Azure, les fonctionnalités suivantes ne sont _pas prises en charge_ : les sauvegardes automatiques, l’intégration d’Azure Key Vault, R Services et la configuration du stockage.

## <a name="licensing"></a>Licence
Les déploiements de SQL Server 2008 R2 avec paiement à l’utilisation peuvent être convertis en [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).

Pour convertir une licence Software Assurance (SA) en paiement à l’utilisation, les clients doivent s’inscrire dans l’[extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md). À l’issue de cette inscription, le type de licence SQL est interchangeable entre Azure Hybrid Benefit et le paiement à l’utilisation.

Les instances SQL Server 2008 ou SQL Server 2008 R2 installées automatiquement sur une machine virtuelle Azure peuvent être inscrites avec l’extension SQL IaaS Agent et leur type de licence converti en paiement à l’utilisation.

## <a name="migration"></a>Migration
Vous pouvez migrer des instances SQL Server en fin de support vers une machine virtuelle Azure avec des méthodes de sauvegarde/restauration manuelles. Il s’agit de la méthode la plus courante quand il s’agit de migrer d’un environnement local vers une machine virtuelle Azure.

### <a name="azure-site-recovery"></a>Azure Site Recovery

Pour les migrations en bloc, nous recommandons le service [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md). Avec Azure Site Recovery, les clients peuvent répliquer la machine virtuelle entière, notamment l’instance SQL Server d’un système local vers une machine virtuelle Azure.

Des captures instantanées Azure Site Recovery avec cohérence des applications sont indispensables à SQL Server pour garantir la récupération. Azure Site Recovery prend en charge les captures instantanées avec cohérence des applications et un intervalle minimum d’une heure. L’objectif de point de récupération (RPO) minimum possible pour SQL Server avec les migrations Azure Site Recovery est d’une heure. L’objectif de point de récupération (RTO) est de deux heures auxquelles s’ajoute le temps de récupération de SQL Server.

### <a name="database-migration-service"></a>Database Migration Service

[Azure Database Migration Service](../../../dms/dms-overview.md) représente une option pour les clients qui souhaitent migrer d’un environnement local vers une machine virtuelle Azure en mettant à niveau SQL Server vers la version 2012 ou une version ultérieure.

## <a name="disaster-recovery"></a>Récupération d'urgence

Les solutions de récupération d’urgence pour SQL Server en fin de support sur une machine virtuelle Azure sont les suivantes :

- **Sauvegardes SQL Server** : Utilisez Sauvegarde Azure pour protéger votre instance SQL Server 2008 et 2008 R2 en fin de support contre les ransomwares, une suppression accidentelle et une altération des données à l’aide d’un objectif de point de récupération de 15 minutes et de la récupération jusqu`à une date et heure. Pour plus d’informations, consultez [cet article](../../../backup/sql-support-matrix.md#scenario-support).
- **Copie des journaux de transaction** : Vous pouvez créer un réplica de la copie des journaux de transaction dans une autre zone ou région Azure, avec des restaurations continues pour réduire le RTO. Vous devez configurer manuellement la copie des journaux de transaction.
- **Azure Site Recovery** : Vous pouvez répliquer votre machine virtuelle entre les zones et les régions par l’intermédiaire de la réplication d’Azure Site Recovery. Les captures instantanées avec cohérence des applications sont indispensables à SQL Server pour garantir une récupération en cas de sinistre. Azure Site Recovery offre un RPO d’une heure au minimum et un RTO de deux heures (plus le temps de récupération de SQL Server) pour la récupération d’urgence de SQL Server en fin de support.

## <a name="security-patching"></a>Correctifs de sécurité

Les mises à jour de sécurité étendues pour les machines virtuelles SQL Server sont remises via les canaux Microsoft Update après que la machine virtuelle SQL Server a été inscrite dans l’[extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md). Les correctifs peuvent être téléchargés manuellement ou automatiquement.

*Automated patching* est activée par défaut. La mise à jour corrective automatisée permet à Azure de corriger automatiquement SQL Server et le système d’exploitation. Vous pouvez spécifier un jour de la semaine, une heure et une durée pour la fenêtre de maintenance si l’extension IaaS SQL Server est installée. Azure effectue la mise à jour corrective dans cette fenêtre de maintenance. La planification de la fenêtre de maintenance utilise les paramètres régionaux de la machine virtuelle pour l’heure. Pour plus d’informations, consultez [Mise à jour corrective automatisée pour SQL Server dans les machines virtuelles Azure](automated-patching.md).


## <a name="next-steps"></a>Étapes suivantes

Migrez votre machine virtuelle SQL Server vers Azure :

* [Migrer une base de données SQL Server vers SQL Server dans une machine virtuelle Azure](migrate-to-vm-from-sql-server.md)

Démarrez avec SQL Server sur des machines virtuelles Azure :

* [Créer une machine virtuelle SQL Server dans le portail Azure](sql-vm-create-portal-quickstart.md)

Obtenez des réponses aux questions fréquentes à propos des machines virtuelles SQL :

* [Questions fréquentes (FAQ) pour SQL Server sur les machines virtuelles Azure](frequently-asked-questions-faq.md)

En savoir plus sur les options de fin de support et sur les mises à jour de sécurité étendues :

* [Fin de support](/sql/sql-server/end-of-support/sql-server-end-of-life-overview) & [Mises à jour de sécurité étendues](/sql/sql-server/end-of-support/sql-server-extended-security-updates)