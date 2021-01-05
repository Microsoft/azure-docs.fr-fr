---
title: Gérer des bases de données avec Azure Automation
description: Découvrez comment le service Azure Automation peut être utilisé pour gérer Azure SQL Database à grande échelle.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: juliemsft
ms.author: jrasnick
ms.reviewer: sstein
ms.date: 03/12/2019
ms.openlocfilehash: faf5b1108e28b352a0b8622feed8bdd99264ff16
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91327576"
---
# <a name="manage-databases-in-azure-sql-database-by-using-azure-automation"></a>Gérer des bases de données dans Azure SQL Database avec Azure Automation

[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Ce guide présente le service Azure Automation et vous explique comment l'utiliser pour simplifier la gestion des bases de données dans Azure SQL Database.

## <a name="about-azure-automation"></a>À propos d’Azure Automation

[Azure Automation](https://azure.microsoft.com/services/automation/) est un service Azure destiné à simplifier la gestion du cloud via l'automatisation des processus. Azure Automation permet d'automatiser les tâches fastidieuses, manuelles, propices aux erreurs et répétitives, afin d'augmenter la fiabilité, l'efficacité et le retour sur investissement pour votre organisation. Pour plus d'informations sur la prise en main, consultez [Introduction à Azure Automation](../../automation/automation-intro.md).

Azure Automation fournit un moteur d’exécution de workflow très fiable et très disponible, qui s'adapte à vos besoins à mesure que votre entreprise se développe. Dans Azure Automation, les processus peuvent être lancés manuellement, par des systèmes tiers ou à des intervalles planifiés, afin que les tâches interviennent exactement au moment opportun.

Réduisez les coûts opérationnels et dégagez du temps pour votre personnel informatique/DevOps afin de lui permettre de se concentrer sur des tâches générant une valeur ajoutée pour l'entreprise, en configurant Azure Automation pour exécuter automatiquement vos tâches de gestion de cloud.

## <a name="how-azure-automation-can-help-manage-your-databases"></a>Comment Azure Automation peut vous aider à gérer vos bases de données

Azure Automation vous permet de gérer les bases de données Azure SQL Database à l'aide des [cmdlets PowerShell](/powershell/module/servicemanagement/azure.service/#sql) disponibles dans les [outils Azure PowerShell ](/powershell/azure/). Azure Automation met à votre disposition ces cmdlets PowerShell pour Azure SQL Database afin de vous permettre d'effectuer toutes vos tâches de gestion SQL Database au sein du service. Dans Azure Automation, vous pouvez également associer ces applets de commande à des applets de commande d’autres services Azure, afin d’automatiser des tâches complexes entre des services Azure et des systèmes tiers.

Azure Automation a également la possibilité de communiquer directement avec les serveurs SQL en exécutant des commandes SQL à l'aide de PowerShell.

Les galeries de runbooks et de modules [Azure Automation](../../automation/automation-runbook-gallery.md) fournissent un éventail de runbooks de Microsoft et de la communauté que vous pouvez importer dans Azure Automation. Pour utiliser l’un de ces runbooks, vous pouvez en télécharger un à partir de la galerie, ou importer directement des runbooks à partir de la galerie, ou à partir de votre compte Automation sur le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris les bases d'Azure Automation et que vous savez l'utiliser pour gérer Azure SQL Database, suivez ces liens pour en savoir plus sur Azure Automation.

- [Vue d’ensemble de Microsoft Azure Automation](../../automation/automation-intro.md)
- [Mon premier Runbook](../../automation/learn/automation-tutorial-runbook-graphical.md)
