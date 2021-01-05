---
title: Questions fréquentes (FAQ) sur les machines virtuelles SQL Server sur Linux | Microsoft Docs
description: Cet article fournit des réponses aux questions fréquemment posées sur l’exécution des machines virtuelles SQL Server sur Linux.
services: virtual-machines-linux
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.service: virtual-machines-sql
ms.topic: troubleshooting
ms.workload: iaas-sql-server
ms.date: 12/13/2017
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: ad991974df30060e552d21a44d5796cd2ba165e2
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792546"
---
# <a name="frequently-asked-questions-for-sql-server-on-linux-virtual-machines"></a>Questions fréquentes (FAQ) sur les machines virtuelles SQL Server sur Linux
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Windows](../windows/frequently-asked-questions-faq.md)
> * [Linux](frequently-asked-questions-faq.md)

Cet article fournit des réponses à certaines des questions les plus courantes sur l’exécution des [machines virtuelles SQL Server sur Linux](sql-server-on-linux-vm-what-is-iaas-overview.md).

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="images"></a><a id="images"></a> Images

1. **Quelles images de galerie de machines virtuelles SQL Server sont disponibles ?**

   Azure gère les images de machines virtuelles pour toutes les versions majeures prises en charge de SQL Server sur toutes les éditions Linux et Windows. Pour plus d’informations, consultez la liste complète des [images de machines virtuelles Linux](sql-server-on-linux-vm-what-is-iaas-overview.md#create) et [images de machines virtuelles Windows](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo).

1. **Les images de galerie de machines virtuelles SQL Server sont-elles mises à jour?**

   Tous les deux mois, les images SQL Server dans la galerie de machines virtuelles sont mises à jour avec les dernières mises à jour Linux et Windows. Pour les images Linux, cela inclut les dernières mises à jour système. Pour les images Windows, sont comprises toutes les mises à jour marquées comme importantes dans Windows Update, dont les Service Packs et mises à jour de sécurité SQL Server importants. Les mises à jour cumulatives SQL Server sont gérées différemment pour Linux et Windows. Pour Linux, les mises à jour cumulatives SQL Server figurent également dans l’actualisation. Toutefois, à ce stade, les machines virtuelles Windows ne sont pas mises à jour avec les mises à jour cumulatives SQL Server ou Windows Server.

1. **Quels sont les packages SQL Server associés également installés ?**

   Pour voir les packages SQL Server installés par défaut sur les machines virtuelles SQL Server sur Linux, consultez [Packages installés](sql-server-on-linux-vm-what-is-iaas-overview.md#packages).

1. **Est-ce que des images de machines virtuelles SQL Server peuvent être supprimées de la galerie ?**

   Oui. Azure gère uniquement une image par version majeure et édition. Par exemple, quand un nouveau Service Pack SQL Server est publié, Azure ajoute une nouvelle image à la galerie pour ce Service Pack. L’image SQL Server pour le Service Pack précédent est immédiatement supprimée du portail Azure. Toutefois, elle est toujours disponible pour le provisionnement à partir de PowerShell pour les trois prochains mois. Après trois mois, l’image du Service Pack précédent n’est plus disponible. Cette stratégie de suppression s’applique également si une version SQL Server n’est plus prise en charge quand elle atteint la fin de son cycle de vie.

## <a name="creation"></a>Création

1. **Comment créer une machine virtuelle Linux avec SQL Server ?**

   La solution la plus simple consiste à créer une machine virtuelle Linux qui comprend SQL Server. Pour obtenir un tutoriel sur l’inscription à Azure et la création d’une machine virtuelle SQL Server à partir du portail, consultez [Provisionner une machine virtuelle Linux exécutant SQL Server dans le portail Azure](sql-vm-create-portal-quickstart.md). Vous avez également la possibilité d’installer manuellement SQL Server sur une machine virtuelle avec une version sous licence gratuite (Developer ou Express) ou en réutilisant une licence locale. Si vous utilisez votre propre licence, vous devez posséder [License Mobility via Software Assurance sur Azure](https://azure.microsoft.com/pricing/license-mobility).

1. **Pourquoi ne puis-je pas configurer une machine virtuelle SQL Server RHEL ou SLES avec un abonnement Azure associé à une limite de dépense ?**

   Les machines virtuelles RHEL et SLES exigent un abonnement sans aucune limite de dépense et un mode de paiement vérifié (généralement une carte de crédit) associé à l’abonnement. Si vous configurez une machine virtuelle RHEL ou SLES sans supprimer la limite de dépense, votre abonnement est désactivé et toutes les machines virtuelles/tous les services sont arrêtés. Si vous parvenez à cette situation, pour réactiver l’abonnement, [supprimez la limite de dépense](https://account.windowsazure.com/subscriptions). Vos crédits restants sont restaurés pour le cycle de facturation en cours, mais un surcoût pour l’image de machine virtuelle RHEL ou SLES est débité sur votre carte de crédit si vous choisissez de redémarrer votre abonnement et de continuer à l’exécuter.

## <a name="licensing"></a>Licence

1. **Comment installer ma copie sous licence de SQL Server sur une machine virtuelle Azure ?**

   Commencez par créer une machine virtuelle uniquement Linux. Puis exécutez les [étapes d’installation de SQL Server](/sql/linux/sql-server-linux-setup#platforms) pour votre distribution Linux. À moins que vous n’installiez l’une des éditions sous licence gratuite de SQL Server, vous devez aussi disposer d’une licence SQL Server et de [License Mobility via Software Assurance sur Azure](https://azure.microsoft.com/pricing/license-mobility/).

1. **Existe-t-il des images de machine virtuelle Linux BYOL (apportez votre propre licence) pour SQL Server ?**

   À ce stade, il n’existe aucune image de machine virtuelle Linux BYOL pour SQL Server. Toutefois, vous pouvez installer manuellement SQL Server sur une machine virtuelle Linux uniquement comme indiqué dans les questions précédentes.

1. **Puis-je modifier une machine virtuelle pour utiliser ma propre licence SQL Server si elle a été créée à partir de l’une des images de la galerie avec paiement à l’utilisation ?**

   Non. Vous ne pouvez pas passer du mode de licence avec paiement à la seconde à de votre propre licence. Vous devez créer une machine virtuelle Linux, installer SQL Server et faire migrer vos données. Consultez la question précédente pour plus d’informations sur l’apport de votre propre licence.

## <a name="administration"></a>Administration

1. **Est-ce que je peux gérer une machine virtuelle Linux exécutant SQL Server avec SSMS (SQL Server Management Studio) ?**

   Oui, mais SSMS est pour l’instant un outil Windows uniquement. Vous devez vous connecter à distance à partir d’un ordinateur Windows pour utiliser SSMS avec les machines virtuelles Linux exécutant SQL Server. Localement sur Linux, le nouvel outil [mssql-conf](/sql/linux/sql-server-linux-configure-mssql-conf) peut effectuer de nombreuses tâches d’administration. Pour afficher un outil de gestion de bases de données multiplateforme, consultez [Azure Data Studio](/sql/azure-data-studio/what-is).

1. **Puis-je supprimer complètement SQL Server d’une machine virtuelle SQL Server ?**

   Oui, mais vous serez toujours facturé pour votre machine virtuelle SQL Server de la manière décrite dans [Tarification des machines virtuelles SQL Server Azure](../windows/pricing-guidance.md?toc=%2fazure%2fvirtual-machines%2flinux%2fsql%2ftoc.json). Si vous n’avez plus besoin de SQL Server, vous pouvez déployer une nouvelle machine virtuelle et y migrer les données et les applications. Vous pouvez ensuite supprimer la machine virtuelle SQL Server.

## <a name="updating-and-patching"></a>Mise à jour et mise à jour corrective

1. **Comment mettre à niveau vers une nouvelle version/édition de SQL Server dans une machine virtuelle Azure ?**

   Il n'existe actuellement aucune mise à niveau sur place pour une instance SQL Server exécutée sur une machine virtuelle Azure. Créez une machine virtuelle Azure avec la version/édition de SQL Server souhaitée, puis migrez vos bases de données vers le nouveau serveur à l’aide des [techniques de migration de données standard](/sql/linux/sql-server-linux-migrate-overview).

## <a name="general"></a>Général

1. **Les solutions de haute disponibilité SQL Server sont-elles prises en charge sur les machines virtuelles Azure ?**

   Pas pour l'instant. Les groupes de disponibilité AlwaysOn et le clustering de basculement nécessitent une solution de clustering dans Linux, telle que Pacemaker. Les distributions Linux prises en charge pour SQL Server ne gèrent pas les modules complémentaires de haute disponibilité dans le cloud.

## <a name="resources"></a>Ressources

**Machines virtuelles Linux**  :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Linux](sql-server-on-linux-vm-what-is-iaas-overview.md)
* [Provisionner SQL Server sur une machine virtuelle Linux](sql-vm-create-portal-quickstart.md)
* [Documentation relative à SQL Server sur Linux](/sql/linux/sql-server-linux-overview)

**Machines virtuelles Windows**  :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](../windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Provisionner SQL Server sur une machine virtuelle Windows](../windows/sql-vm-create-portal-quickstart.md)
* [Forum Aux Questions (Windows)](../windows/frequently-asked-questions-faq.md)