---
title: Configurer Advanced Threat Protection
description: Advanced Threat Protection détecte les activités de base de données anormales indiquant des menaces de sécurité potentielles pour la base de données dans Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-dt-2019, sqldbrb=1
ms.topic: how-to
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 12/01/2020
ms.openlocfilehash: 1425003c718ca52c0bea712e9d25cd3e4c035cf1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453979"
---
# <a name="configure-advanced-threat-protection-for-azure-sql-database"></a>Configurer Advanced Threat Protection pour Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Advanced Threat Protection](threat-detection-overview.md) pour Azure SQL Database détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données. Advanced Threat Protection peut identifier une **injection SQL potentielle**, un **accès à partir d'un emplacement ou d'un centre de données inhabituel**, un **accès à partir d'un principal inconnu ou d'une application potentiellement dangereuse** et des **informations d'identification SQL par force brute**. Pour plus d'informations, consultez [Alertes de protection avancée des menaces](threat-detection-overview.md#alerts).

Vous pouvez recevoir des notifications à propos des menaces détectées par l’intermédiaire de [notifications par e-mail](threat-detection-overview.md#explore-detection-of-a-suspicious-event) ou du [portail Azure](threat-detection-overview.md#explore-alerts-in-the-azure-portal).

[Advanced Threat Protection](threat-detection-overview.md) fait partie de l’offre [Azure Defender pour SQL](azure-defender-for-sql.md). Il s’agit d’un package unifié pour les fonctionnalités de sécurité SQL avancées. Advanced Threat Protection est accessible et gérable par le biais du portail Azure Defender pour SQL central.

## <a name="set-up-advanced-threat-protection-in-the-azure-portal"></a>Configurer la protection avancée contre les menaces dans le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à la page de configuration du serveur que vous voulez protéger. Dans les paramètres de sécurité, sélectionnez **Security Center**.
3. Sur la page de configuration **Azure Defender pour SQL** :

   - Activez **Azure Defender pour SQL** sur le serveur.
   - Dans **Paramètres de protection avancée contre les menaces**, fournissez la liste d’adresses e-mail devant recevoir des alertes de sécurité lors de la détection d’activités de base de données anormales dans la zone de texte **Envoyer les alertes à**.
   
   :::image type="content" source="media/azure-defender-for-sql/set-up-advanced-threat-protection.png" alt-text="Configurer la protection avancée contre les menaces":::

## <a name="set-up-advanced-threat-protection-using-powershell"></a>Configurer Advanced Threat Protection avec PowerShell

Pour obtenir un exemple de script, consultez [Configurer l’audit et Advanced Threat Protection avec PowerShell](scripts/auditing-threat-detection-powershell-configure.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Advanced Threat Protection](threat-detection-overview.md)
- Découvrez-en plus sur [Advanced Threat Protection dans SQL Managed Instance](../managed-instance/threat-detection-configure.md).  
- En savoir plus sur [Azure Defender pour SQL](azure-defender-for-sql.md).
- En savoir plus sur l'[audit](../../azure-sql/database/auditing-overview.md)
- En savoir plus sur [Azure Security Center](../../security-center/security-center-introduction.md)
- Pour plus d'informations sur la tarification, consultez la [page de tarification SQL Database](https://azure.microsoft.com/pricing/details/sql-database/)