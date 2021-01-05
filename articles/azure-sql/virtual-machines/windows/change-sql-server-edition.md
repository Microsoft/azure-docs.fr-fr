---
title: Modification sur place de l’édition SQL Server
description: Découvrez comment modifier l’édition de votre machine virtuelle SQL Server dans Azure afin de réduire les coûts ou mettre à niveau pour activer davantage de fonctionnalités.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/14/2020
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: b6ea6f947b2cd8d7d9c38a0e5e440ce5fd273bb7
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358824"
---
# <a name="in-place-change-of-sql-server-edition-on-azure-vm"></a>Modification sur place de l’édition SQL Server sur une machine virtuelle Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment modifier l’édition de SQL Server sur une machine virtuelle Windows dans Azure. 

L’édition de SQL Server est déterminée par la clé de produit (Product Key) et spécifiée pendant le processus d’installation à l’aide du support d’installation. L’édition détermine les [fonctionnalités](/sql/sql-server/editions-and-components-of-sql-server-2017) qui sont disponibles dans le produit SQL Server. Vous pouvez modifier l’édition de SQL Server avec le support d’installation, et passer à une version antérieure pour réduire les coûts ou à une version ultérieure pour activer plus de fonctionnalités.

Une fois que l’édition de SQL Server a été modifiée en interne sur la machine virtuelle SQL Server, vous devez ensuite mettre à jour la propriété d’édition de SQL Server dans le Portail Azure à des fins de facturation. 

## <a name="prerequisites"></a>Prérequis

Pour effectuer une modification sur place de l’édition de SQL Server, voici ce dont vous avez besoin : 

- Un [abonnement Azure](https://azure.microsoft.com/free/).
- Une [machine virtuelle SQL Server sur Windows](./create-sql-vm-portal.md) inscrite auprès de [l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md).
- Le support d’installation correspondant à l’**édition de SQL Server souhaitée**. Les clients qui disposent de la [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) peuvent obtenir leur support d’installation à partir du [Centre de gestion des licences en volume](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Les clients qui ne disposent pas de la Software Assurance peuvent utiliser le support d’installation d’une image de machine virtuelle SQL Server de la Place de marché Azure correspondant à l’édition souhaitée (généralement située dans `C:\SQLServerFull`). 


## <a name="upgrade-an-edition"></a>Mettre à niveau une édition

> [!WARNING]
> La mise à niveau de l’édition de SQL Server a pour effet de redémarrer le service pour SQL Server ainsi que tous les services associés, à l’instar d’Analysis Services et de R Services. 

Pour mettre à niveau l’édition de SQL Server, procurez-vous le support d’installation de SQL Server correspondant à l'édition souhaitée, puis procédez comme suit :

1. Ouvrez Setup.exe à partir du support d’installation de SQL Server. 
1. Accédez à **Maintenance** et choisissez l’option **Mise à niveau d’édition**. 

   ![Sélection pour la mise à niveau de l’édition de SQL Server](./media/change-sql-server-edition/edition-upgrade.png)

1. Sélectionnez **Suivant** jusqu'à accéder à la page **Prêt pour la mise à niveau de l'édition**, puis sélectionnez **Mettre à niveau**. La fenêtre d’installation peut cesser de répondre pendant quelques minutes, le temps que la modification prenne effet. Une page **Terminé** confirme que la mise à niveau de l’édition est terminée. 

Une fois que l’édition de SQL Server est mise à niveau, modifiez la propriété d’édition de la machine virtuelle SQL Server sur le Portail Azure. Les métadonnées et la facturation associées à cette machine virtuelle sont alors mises à jour.

## <a name="downgrade-an-edition"></a>Passer à une édition antérieure

Pour passer à une édition antérieure de SQL Server, vous devez complètement désinstaller SQL Server et le réinstaller avec le support d’installation de l'édition souhaitée. 

> [!WARNING]
> La désinstallation de SQL Server peut occasionner des temps d’arrêt supplémentaires. 

Pour passer à une édition antérieure de SQL Server, procédez comme suit :

1. Sauvegardez toutes les bases de données, y compris les bases de données système. 
1. Déplacez des bases de données système (maître, modèle et msdb) vers un nouvel emplacement. 
1. Désinstallez complètement SQL Server et tous les services associés. 
1. Redémarrez la machine virtuelle. 
1. Installez SQL Server en utilisant le support correspondant à l’édition souhaitée de SQL Server.
1. Installez les derniers Service Packs et mises à jour cumulatives.  
1. Remplacez les nouvelles bases de données système créées pendant l’installation par les bases de données système que vous avez déplacées antérieurement vers un autre emplacement. 

Une fois que l’édition de SQL Server a été passée à une version antérieure, modifiez la propriété d’édition de la machine virtuelle SQL Server sur le Portail Azure. Les métadonnées et la facturation associées à cette machine virtuelle sont alors mises à jour.

## <a name="change-edition-in-portal"></a>Modifier l’édition dans le portail 

Une fois que vous avez modifié l’édition de SQL Server à l’aide du support d’installation et que vous avez inscrit votre machine virtuelle SQL Server auprès de [l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md), vous pouvez utiliser le portail Azure pour modifier la propriété Édition de la machine virtuelle SQL Server à des fins de facturation. Pour ce faire, procédez comme suit : 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Accédez à votre ressource Machine virtuelle SQL Server. 
1. Sous **Paramètres**, sélectionnez **Configurer**. Sélectionnez ensuite l’édition souhaitée de SQL Server dans la liste déroulante située en dessous de **Édition**. 

   ![Modifier les métadonnées de l'édition](./media/change-sql-server-edition/edition-change-in-portal.png)

1. Lisez l’avertissement qui indique que vous devez d’abord modifier l’édition de SQL Server et que la propriété d’édition doit correspondre à l’édition de SQL Server. 
1. Sélectionnez **Appliquer** pour appliquer les modifications des métadonnées de l'édition. 


## <a name="remarks"></a>Notes

- La propriété d’édition de la machine virtuelle SQL Server doit correspondre à l’édition de l’instance SQL Server installée pour toutes les machines virtuelles SQL Server, y compris les types de licence « paiement à l’utilisation » et BYOL (apportez votre propre licence).
- Si vous supprimez votre ressource de machine virtuelle SQL Server, vous revenez au paramètre d'édition codé en dur de l’image.
- La possibilité de changer d’édition est une fonctionnalité de l’extension SQL IaaS Agent. Pendant le déploiement d’une image de Place de marché Azure via le portail Azure, une machine virtuelle SQL Server est inscrite automatiquement auprès de l’extension SQL IaaS Agent. Toutefois, les clients qui installent automatiquement SQL Server doivent manuellement [inscrire leur machine virtuelle SQL Server](sql-agent-extension-manually-register-single-vm.md).
- L’ajout d’une machine virtuelle SQL Server à un groupe à haute disponibilité implique la recréation de la machine virtuelle. Les machines virtuelles ajoutées à un groupe à haute disponibilité repassent à l’édition par défaut et l’édition doit à nouveau être modifiée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
* [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication pour SQL Server sur une machine virtuelle Windows](doc-changes-updates-release-notes.md)