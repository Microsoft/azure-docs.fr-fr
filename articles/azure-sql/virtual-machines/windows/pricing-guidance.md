---
title: Aide sur les prix et gestion des coûts
description: Fournit des meilleures pratiques pour choisir le bon modèle de tarification de machine virtuelle SQL Server.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/09/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: bd58f18959df3aa290083f907be8ab703366a7c6
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97360099"
---
# <a name="pricing-guidance-for-sql-server-on-azure-vms"></a>Guide des prix de SQL Server sur machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article fournit un guide des prix pour [SQL Server sur machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md). Plusieurs facteurs ont un impact sur le coût, et il est important de choisir le bon équilibre entre les coûts et les besoins de l’entreprise.

> [!TIP]
> Si vous cherchez seulement à obtenir une estimation de coût pour une combinaison spécifique d’édition de SQL Server et de taille de machine virtuelle, consultez la page de tarification pour [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows) ou [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux). Sélectionnez votre plateforme et l’édition de SQL Server dans la liste **Système d’exploitation/Logiciel**.
>
> ![Interface utilisateur sur la page de tarification des machines virtuelles](./media/pricing-guidance/virtual-machines-pricing-ui.png)
>
> Ou utilisez la [Calculatrice de prix](https://azure.microsoft.com/pricing/#explore-cost) pour ajouter et configurer une machine virtuelle. 

## <a name="free-licensed-sql-server-editions"></a>Éditions SQL Server sous licence libre

Si vous souhaitez développer, tester ou générer une preuve de concept, utilisez l’édition **SQL Server Développeur** sous licence libre. Cette édition offre toutes les fonctionnalités de SQL Server Entreprise, ce qui vous permet de générer et de tester n’importe quel type d’application. Toutefois, vous ne pouvez pas exécuter l’édition Développeur en production. Une machine virtuelle exécutant SQL Server Développeur engendre uniquement des frais liés au coût de la machine virtuelle, car aucun coût de licence n’est associé à cette édition.

Si vous souhaitez exécuter une charge de travail légère en production (< 4 cœurs, < 1 Go de mémoire, < 10 Go/base de données), utilisez l’édition **SQL Server Express** sous licence gratuite. Là encore, une machine virtuelle exécutant SQL Server Express engendre uniquement des frais liés au coût de la machine virtuelle.

Pour ces charges de travail de développement/test et de production légère, vous pouvez également faire des économies en choisissant une machine virtuelle plus petite qui correspond à ces charges de travail. DS1v2 peut s’avérer un bon choix dans certains scénarios.

Pour créer une machine virtuelle Azure exécutant SQL Server 2017 avec l’une de ces images, consultez les liens suivants :

| Plateforme | Images sous licence libre |
|---|---|
| Windows Server 2016 | [Machine virtuelle Azure SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonWindowsServer2016)<br/>[Machine virtuelle Azure SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonWindowsServer2016) |
| Red Hat Enterprise Linux | [Machine virtuelle Azure SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonRedHatEnterpriseLinux74)<br/>[Machine virtuelle Azure SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonRedHatEnterpriseLinux74) |
| SUSE Linux Enterprise Server | [Machine virtuelle Azure SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonSLES12SP2)<br/>[Machine virtuelle Azure SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonSLES12SP2) |
| Ubuntu | [Machine virtuelle Azure SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonUbuntuServer1604LTS)<br/>[Machine virtuelle Azure SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonUbuntuServer1604LTS) |

## <a name="paid-sql-server-editions"></a>Éditions SQL Server payantes

Si votre charge de travail de production n’est pas légère, utilisez l’une des éditions suivantes de SQL Server :

| Édition SQL Server | Charge de travail |
|-----|-----|
| Web | Petits sites web |
| Standard | Charges de travail faibles ou moyennes |
| Entreprise | Charges de travail volumineuses ou critiques|

Pour ces éditions, vous pouvez vous acquitter de la licence SQL Server de deux manières : par un *paiement à l’utilisation* ou par la méthode *BYOL (apportez votre propre licence)* .

## <a name="pay-per-usage"></a>Paiement à l’utilisation

**Le paiement d’une licence SQL Server à l’utilisation** (ou **paiement au fur et à mesure**) signifie que le coût par seconde de l’exécution de la machine virtuelle Azure comprend le coût de la licence SQL Server. Les tarifs des différentes éditions de SQL Server (Web, Standard, Entreprise) sont indiqués dans la page de tarification des machines virtuelles Azure pour [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows) ou [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux).

Le coût est le même pour toutes les versions de SQL Server (de 2012 SP3 à 2019). Le coût de licence par seconde varie selon le nombre de processeurs virtuels de machine virtuelle.

Le paiement de la licence SQL Server à l’utilisation est recommandé pour :

- **Les charges de travail temporaires ou périodiques**. Par exemple, une application qui doit soutenir un événement pendant quelques mois chaque année, ou une analyse métier réalisée tous les lundis.

- **Les charges de travail dont la durée de vie ou la mise à l’échelle ne sont pas connues**. Par exemple, une application qui peut ne pas être requise dans quelques mois, ou qui peut nécessiter plus ou moins de puissance de calcul, en fonction de la demande.

Pour créer une machine virtuelle Azure exécutant SQL Server 2017 avec l’une de ces images assorties d’un paiement à l’utilisation, consultez les liens suivants :

| Plateforme | Images sous licence |
|---|---|
| Windows Server 2016 | [Machine virtuelle Azure Web SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonWindowsServer2016)<br/>[Machine virtuelle Azure Standard SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonWindowsServer2016)<br/>[Machine virtuelle Azure Entreprise SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseWindowsServer2016) |
| Red Hat Enterprise Linux | [Machine virtuelle Azure Web SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonRedHatEnterpriseLinux74)<br/>[Machine virtuelle Azure Standard SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonRedHatEnterpriseLinux74)<br/>[Machine virtuelle Azure Entreprise SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonRedHatEnterpriseLinux74) |
| SUSE Linux Enterprise Server | [Machine virtuelle Azure Web SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonSLES12SP2)<br/>[Machine virtuelle Azure Standard SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonSLES12SP2)<br/>[Machine virtuelle Azure Entreprise SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonSLES12SP2) |
| Ubuntu | [Machine virtuelle Azure Web SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonUbuntuServer1604LTS)<br/>[Machine virtuelle Azure Standard SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonUbuntuServer1604LTS)<br/>[Machine virtuelle Azure Entreprise SQL Server 2017](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonUbuntuServer1604LTS) |

> [!IMPORTANT]
> Lorsque vous créez une machine virtuelle SQL Server dans le portail Azure, la fenêtre **Choisir une taille** affiche un coût estimé. Il est important de noter que cette estimation correspond uniquement aux frais de calcul liés à l’exécution de la machine virtuelle, ainsi qu’aux coûts de licence du système d’exploitation (Windows ou système d’exploitation Linux tiers).
>
> ![Panneau Choisir la taille de la machine virtuelle](./media/pricing-guidance/sql-vm-choose-size-pricing-estimate.png)
>
>N’inclut pas les coûts de licence SQL Server supplémentaires pour les éditions Web, Standard et Enterprise. Pour obtenir une estimation plus précise des tarifs, sélectionnez votre système d’exploitation et l’édition SQL Server dans la page des tarifs pour [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) ou [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

> [!NOTE]
> Vous pouvez désormais changer de modèle de licence et passer du paiement à l’utilisation à une licence BYOL (apportez votre propre licence), et inversement. Pour plus d'informations, consultez [Guide pratique pour changer le modèle de licence d'une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). 

## <a name="bring-your-own-license-byol"></a><a id="byol"></a> BYOL (apportez votre propre licence)

L’**apport de votre propre licence SQL Server par le biais de License Mobility**, également appelé **BYOL**, implique l’utilisation d’une licence en volume SQL Server existante avec Software Assurance dans une machine virtuelle Azure. Une machine virtuelle SQL Server qui a recours à la méthode BYOL facture uniquement le coût d’exécution de la machine virtuelle, et non la licence SQL Server, étant donné que vous avez déjà acquis les licences et Software Assurance dans le cadre d’un programme de licence en volume.

> [!IMPORTANT]
> Les images BYOL impliquent un contrat entreprise avec Software Assurance. Elles ne sont pas disponibles dans le cadre du partenariat Fournisseur de solutions Azure Cloud (CSP) pour l’instant. Les clients CSP peuvent utiliser leur propre licence en déployant une image avec paiement à l’utilisation, puis en activant [Azure Hybrid Benefit](licensing-model-azure-hybrid-benefit-ahb-change.md).

> [!NOTE]
> Les images BYOL ne sont actuellement disponibles que pour les machines virtuelles Windows. Toutefois, vous pouvez installer manuellement SQL Server sur une machine virtuelle Linux uniquement. Reportez-vous aux instructions disponibles dans le [FAQ concernant SQL Server sur une machine virtuelle Linux](../linux/frequently-asked-questions-faq.md).

L’apport de votre propre licence SQL au travers de la mobilité de licence est recommandé pour :

- **Les charges de travail continues**. Par exemple, une application qui doit prendre en charge les opérations de l’entreprise 24 h/24, 7 j/7.

- **Les charges de travail dont la durée de vie et la mise à l’échelle sont connues**. Par exemple, une application qui sera requise pendant toute l’année et dont la demande a été prévue.

Pour utiliser la méthode BYOL avec une machine virtuelle SQL Server, vous devez posséder une licence SQL Server Standard ou Enterprise ainsi que [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default.aspx#tab=1), qui est une option obligatoire pour certains programmes de licence en volume, et une option facultative pour d’autres. Le niveau de tarification fourni par les programmes de licence en volume varie selon le type de contrat et la quantité et/ou l’engagement vis-à-vis de SQL Server. Mais en règle générale, l’apport de votre propre licence pour les charges de travail continues présente les avantages suivants :

| Avantages de la méthode BYOL | Description |
|-----|-----|
| **Réduction des coûts** | [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) permet d'économiser jusqu'à 55 %. Pour plus d’informations, consultez [Changer de modèle de licence](licensing-model-azure-hybrid-benefit-ahb-change.md) |
| **Réplica secondaire passif gratuit** | Autre avantage de l’apport de votre propre licence : la [licence gratuite pour un réplica secondaire passif](https://azure.microsoft.com/pricing/licensing-faq/) par SQL Server à des fins de haute disponibilité. Cela réduit de moitié le coût de licence d’un déploiement SQL Server à haute disponibilité (par exemple, à l’aide de groupes de disponibilité AlwaysOn). Les droits d’exécution du réplica secondaire passif sont fournis au travers des avantages procurés par Software Assurance pour les serveurs de basculement. |

Pour créer une machine virtuelle Azure exécutant SQL Server 2017 avec l’une de ces images BYOL, reportez-vous aux machines virtuelles dont le nom contient le préfixe « {BYOL} » :

- [Machine virtuelle Azure Entreprise SQL Server 2017](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017EnterpriseWindowsServer2016)
- [Machine virtuelle Azure Standard SQL Server 2017](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017StandardonWindowsServer2016)

> [!IMPORTANT]
> Veuillez nous indiquer sous 10 jours le nombre de licences SQL Server que vous utilisez dans Azure. Les liens vers les images précédentes comportent des instructions sur la manière de procéder.

> [!NOTE]
> Vous pouvez désormais changer de modèle de licence et passer du paiement à l’utilisation à une licence BYOL (apportez votre propre licence), et inversement. Pour plus d'informations, consultez [Guide pratique pour changer le modèle de licence d'une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). 



## <a name="reduce-costs"></a>Réduire les coûts

Pour éviter des coûts inutiles, choisissez une taille de machine virtuelle optimale et envisagez des arrêts intermittents pour les charges de travail non continues.

### <a name="correctly-size-your-vm"></a><a id="machinesize"></a> Dimensionner correctement votre machine virtuelle

Le coût de la licence SQL Server est directement lié au nombre de processeurs virtuels. Choisissez une taille de machine virtuelle qui correspond à vos besoins en termes de processeur, mémoire, stockage et bande passante d’E/S. Pour obtenir la liste complète des options de taille de machine virtuelle, consultez [Tailles de machine virtuelle Windows](../../../virtual-machines/sizes.md) et [Tailles de machine virtuelle Linux](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Il existe de nouvelles tailles de machine virtuelle qui fonctionnent correctement avec certains types de charges de travail SQL Server. Ces tailles de machine virtuelle ont des niveaux élevés de mémoire, de stockage et de bande passante d’E/S, mais elles possèdent un nombre de cœurs virtualisés inférieur. Considérez l’exemple suivant :

| Taille de la machine virtuelle | Processeurs virtuels | Mémoire | Disques max | Débit d’E/S max | Coûts de licence SQL Server | Total des coûts (calcul + licence) |
|---|---|---|---|---|---|---|
| **Standard_DS14v2** | 16 | 112 Go | 32 | 51 200 IOPS ou 768 Mo/s | | |
| **Standard_DS14-4v2** | 4 | 112 Go | 32 | 51 200 IOPS ou 768 Mo/s | 75 % inférieur | 57 % inférieur |

> [!IMPORTANT]
> Il s’agit d’un exemple ponctuel. Pour les spécifications les plus récentes, consultez les articles sur les tailles de machine virtuelle et la page de tarification Azure pour [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) et [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

Dans l’exemple précédent, vous pouvez voir que les spécifications de **Standard_DS14v2** et **Standard_DS14-4v2** sont identiques à l’exception des processeurs virtuels. Le suffixe **-4v2** à la fin de la taille de machine **Standard_DS14-4v2** indique le nombre de processeurs virtuels actifs. Étant donné que les coûts de licence SQL Server sont liés au nombre de processeurs virtuels, cela réduit considérablement le coût de la machine virtuelle dans les scénarios où les processeurs virtuels supplémentaires ne sont pas nécessaires. Il s’agit d’un seul exemple. Il existe de nombreuses tailles de machine avec des processeurs virtuels limités qui sont identifiées par ce modèle de suffixe. Pour plus d’informations, consultez le billet de blog [Announcing new Azure VM sizes for more cost-effective database work](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/).

### <a name="shut-down-your-vm-when-possible"></a>Arrêter votre machine virtuelle lorsque cela est possible

Si vous utilisez des charges de travail qui ne s’exécutent pas en continu, envisagez d’arrêter la machine virtuelle pendant les périodes d’inactivité. Vous paierez uniquement pour ce que vous utiliserez.

Par exemple, si vous testez simplement SQL Server sur une machine virtuelle Azure, vous ne souhaitez pas engendrer de coûts en le laissant accidentellement s’exécuter pendant des semaines. Pour remédier à ce problème, il est possible de recourir à la [fonction d’arrêt automatique](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/).

![Arrêt automatique de machine virtuelle SQL Server](./media/pricing-guidance/sql-vm-auto-shutdown.png)

L’arrêt automatique fait partie d’un ensemble plus important de fonctionnalités similaires fournies par [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab).

Pour les autres flux de travail, envisagez l’arrêt et le redémarrage automatiques des machines virtuelles Azure avec une solution de script, comme [Azure Automation](https://azure.microsoft.com/services/automation/).

> [!IMPORTANT]
> L’arrêt et la libération de votre machine virtuelle constituent la seule façon d’éviter des frais. Le simple arrêt ou l’utilisation des options d’alimentation pour arrêter la machine virtuelle entraînent toujours des frais d’utilisation.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une assistance globale sur la tarification Azure, consultez [Éviter les coûts inattendus avec la gestion de la facturation et des coûts dans Azure](../../../cost-management-billing/cost-management-billing-overview.md). Pour voir les tarifs les plus récents des machines virtuelles Azure, notamment ceux de SQL Server, consultez la page de tarification des machines virtuelles Azure pour [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) et [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

Pour obtenir une vue d’ensemble de SQL Server sur des machines virtuelles Azure, consultez les articles suivants :

- [Vue d’ensemble de SQL Server sur des machines virtuelles Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Vue d’ensemble de SQL Server sur des machines virtuelles Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)