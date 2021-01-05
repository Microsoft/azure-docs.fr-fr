---
title: Modification sur place de la version SQL Server
description: Découvrez comment modifier la version de votre machine virtuelle SQL Server dans Azure.
services: virtual-machines-windows
documentationcenter: na
author: ramakoni1
manager: ramakoni1
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/08/2020
ms.author: RamaKoni
ms.reviewer: sqlblt, daleche
ms.custom: seo-lt-2019
ms.openlocfilehash: 5dbd5725c666afe04d57d4432f0a4798fcb34c03
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358841"
---
# <a name="in-place-change-of-sql-server-version-on-azure-vm"></a>Modification sur place de la version de SQL Server sur une machine virtuelle Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment modifier la version de Microsoft SQL Server sur une machine virtuelle Windows dans Microsoft Azure.

## <a name="prerequisites"></a>Prérequis

Pour effectuer une mise à niveau sur place de SQL Server, les conditions suivantes s’appliquent :

- Le support d’installation de la version souhaitée de SQL Server est requis. Les clients qui disposent de la [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) peuvent obtenir leur support d’installation à partir du [Centre de gestion des licences en volume](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Les clients qui ne disposent pas de la Software Assurance peuvent utiliser le support d’installation d’une image de machine virtuelle SQL Server de la Place de marché Azure correspondant à la version souhaitée (généralement située dans C:\SQLServerFull).
- Les mises à niveau d’édition doivent suivre les [chemins de mise à niveau de support](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15).

## <a name="planning-for-version-change"></a>Planification du changement de version

Nous vous recommandons de passer en revue les éléments suivants avant d’effectuer la modification de la version :

1. Vérifiez les nouveautés de la version que vous envisagez de mettre à niveau vers :

   - Nouveautés dans [SQL 2019](/sql/sql-server/what-s-new-in-sql-server-ver15)
   - Nouveautés dans [SQL 2017](/sql/sql-server/what-s-new-in-sql-server-2017)
   - Nouveautés dans [SQL 2016](/sql/sql-server/what-s-new-in-sql-server-2016)


1. Nous vous recommandons de vérifier la [certification de compatibilité](/sql/database-engine/install-windows/compatibility-certification) pour la version que vous allez modifier afin de pouvoir utiliser les modes de compatibilité de la base de données afin de réduire l’effet de la mise à niveau.
1. Vous pouvez évaluer les articles suivants pour vous aider à obtenir un résultat correct :

   - [Vidéo : Modernisation de SQL Server | PAM Lahoud & Pedro Lopes | 20 ans de réussite](https://www.youtube.com/watch?v=5RPkuQHcxxs&feature=youtu.be)
   - [Assistant Expérimentation de base de données pour le test AB](/sql/dea/database-experimentation-assistant-overview)
   - [Mise à niveau des bases de données à l’aide de l’Assistant Paramétrage de requête](/sql/relational-databases/performance/upgrade-dbcompat-using-qta)
   - [Modifier le niveau de compatibilité de la base de données et utiliser le Magasin des requêtes](/sql/database-engine/install-windows/change-the-database-compatibility-mode-and-use-the-query-store)

## <a name="upgrade-sql-version"></a>Mettre à niveau la version SQL

> [!WARNING]
> La mise à niveau de la version SQL Server a pour effet de redémarrer le service pour SQL Server ainsi que tous les services associés, à l’instar d’Analysis Services et des Services R.

Pour mettre à niveau la version de SQL Server, procurez-vous le support d’installation SQL Server pour la version ultérieure qui [prend en charge le chemin de mise à niveau](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15) de SQL Server et procédez comme suit :

1. Sauvegardez les bases de données, y compris le système (à l’exception de tempdb) et les bases de données utilisateur, avant de démarrer le processus. Vous pouvez également créer une sauvegarde de niveau machine virtuelle cohérente avec les applications à l’aide des services de Sauvegarde Azure.
1. Démarrez Setup.exe à partir du support d’installation de SQL Server.
1. L'assistant d’installation débute le Centre d'installation de SQL Server. Pour mettre à niveau une instance existante de SQL Server, sélectionnez **Installation** dans le volet de navigation, puis sélectionnez **Mettre à niveau à partir d’une version antérieure de SQL Server**.

   :::image type="content" source="./media/change-sql-server-version/upgrade.png" alt-text="Sélection pour la mise à niveau de la version de SQL Server":::

1. Dans la page **Clé de produit**, cliquez sur une option pour indiquer si vous effectuez une mise à niveau vers une édition gratuite de SQL Server ou si vous disposez d'une clé PID pour une version de production du produit. Pour plus d’informations, consultez [Éditions et composants de SQL Server 2019 (15.x)](/sql/sql-server/editions-and-components-of-sql-server-version-15) et [Mises à niveau de version et d’édition prises en charge (SQL Server 2016)](/sql/database-engine/install-windows/supported-version-and-edition-upgrades).
1. Sélectionnez **Suivant** jusqu'à accéder à la page **Prêt pour la mise à niveau**, puis sélectionnez **Mettre à niveau**. La fenêtre d’installation peut cesser de répondre pendant quelques minutes, le temps que la modification prenne effet. Une page **Terminé** confirme que la mise à niveau est terminée. Pour obtenir une procédure pas à pas de mise à niveau, consultez [la procédure complète](/sql/database-engine/install-windows/upgrade-sql-server-using-the-installation-wizard-setup#procedure).

   :::image type="content" source="./media/change-sql-server-version/complete-page.png" alt-text="Terminer la page":::

Si vous avez modifié l’édition de SQL Server en plus de modifier la version, mettez également à jour l’édition et reportez-vous à la section **vérifier la version et l’édition dans le portail** pour modifier l’instance de machine virtuelle SQL.

   :::image type="content" source="./media/change-sql-server-version/change-portal.png" alt-text="Obtenir les métadonnées de version":::

## <a name="downgrade-the-version-of-sql-server"></a>Passer à une version antérieure de SQL Server

Pour passer à une version antérieure de SQL Server, vous devez complètement désinstaller SQL Server et le réinstaller avec le support d’installation de la version souhaitée. Cela est similaire à une nouvelle installation de SQL Server, car vous ne pourrez pas restaurer la base de données antérieure d’une version ultérieure à la version antérieure installée. Les bases de données devront être recréées à partir de zéro. Si vous avez également modifié l’édition de SQL Server pendant la mise à niveau, remplacez la valeur de la propriété **Édition** de la machine virtuelle SQL Server dans le Portail Azure par la nouvelle valeur d’édition. Les métadonnées et la facturation associées à cette machine virtuelle sont alors mises à jour.

> [!WARNING]
> Un passage à la version ultérieure sur place de SQL Server n’est pas prise en charge.

Pour passer à une version antérieure de SQL Server, procédez comme suit :

1. Assurez-vous que vous n’utilisez aucune fonctionnalité [disponible dans la version ultérieure uniquement](https://social.technet.microsoft.com/wiki/contents/articles/24222.find-enterprise-only-features-in-your-database.aspx).
1. Sauvegardez les bases de données, y compris le système (à l’exception de tempdb) et les bases de données utilisateur.
1. Exportez tous les objets de niveau serveur nécessaires (tels que les déclencheurs de serveur, les rôles, les connexions, les serveurs liés, les travaux, les informations d’identification et les certificats).
1. Si vous n’avez pas de scripts pour recréer vos bases de données utilisateur sur la version antérieure, vous devez générer un script pour tous les objets et exporter toutes les données à l’aide de BCP. exe, SSIS ou DACPAC.

   Veillez à sélectionner les options appropriées lorsque vous créez un script de tels éléments comme version cible, objets dépendants et options avancées.

   :::image type="content" source="./media/change-sql-server-version/scripting-options.png" alt-text="Options de script":::

1. Désinstallez complètement SQL Server et tous les services associés.
1. Redémarrez la machine virtuelle.
1. Installez SQL Server en utilisant le média correspondant à la version souhaitée du programme.
1. Installez les derniers Service Packs et mises à jour cumulatives.
1. Importez tous les objets de niveau serveur nécessaires (qui ont été exportés à l’étape 3).
1. Recréez toutes les bases de données utilisateur nécessaires à partir de zéro (à l’aide de scripts créés ou des fichiers de l’étape 4).

## <a name="verify-the-version-and-edition-in-the-portal"></a>Vérifier la version et l’édition dans le portail

Après avoir modifié la version de SQL Server, inscrivez de nouveau votre machine virtuelle SQL Server auprès de [l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) afin de pouvoir utiliser le portail Azure pour afficher la version de SQL Server. Le numéro de version indiqué doit maintenant refléter la version et l’édition récemment mises à niveau de votre installation SQL Server.

:::image type="content" source="./media/change-sql-server-version/verify-portal.png" alt-text="Vérifier la version":::

> [!NOTE]
> Si vous êtes déjà inscrit auprès de l’extension SQL IaaS Agent, [désinscrivez-vous de l’extension](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) puis [inscrivez à nouveau la ressource de machine virtuelle SQL](sql-agent-extension-manually-register-single-vm.md#register-with-extension) afin qu’elle détecte la version et l’édition appropriées de SQL Server installée sur la machine virtuelle. Les métadonnées et la facturation associées à cette machine virtuelle sont alors mises à jour.

## <a name="remarks"></a>Notes

- Nous vous recommandons d’initier des sauvegardes/de mettre à jour des statistiques/de reconstruire des index/vérifier la cohérence une fois la mise à niveau terminée. Vous pouvez également vérifier les niveaux de compatibilité individuels de chaque base de données pour vous assurer qu’ils reflètent le niveau souhaité.
- Une fois SQL Server mis à jour sur la machine virtuelle, assurez-vous que la propriété **Édition** de SQL Server dans le Portail Azure correspond au numéro de l’édition installée pour la facturation.
- La possibilité de [changer d’édition](change-sql-server-edition.md#change-edition-in-portal) est une fonctionnalité de l’extension SQL IaaS Agent. Pendant le déploiement d’une image de Place de marché Azure via le portail Azure, une machine virtuelle SQL Server est inscrite automatiquement auprès de l’extension. Toutefois, les clients qui installent automatiquement SQL Server doivent manuellement [inscrire leur machine virtuelle SQL Server](sql-agent-extension-manually-register-single-vm.md).
- Si vous supprimez votre ressource de machine virtuelle SQL Server, vous restaurez au paramètre d'édition codé en dur de l’image.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
- [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
- [Notes de publication pour SQL Server sur une machine virtuelle Windows](doc-changes-updates-release-notes.md)