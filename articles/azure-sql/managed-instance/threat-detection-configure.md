---
title: Configurer Advanced Threat Protection
titleSuffix: Azure SQL Managed Instance
description: Advanced Threat Protection détecte les activités de base de données anormales indiquant des menaces de sécurité potentielles pour la base de données dans Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
ms.date: 12/01/2020
ms.openlocfilehash: 69bebcf872f55055117acf5cef410d1f89eafe34
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446901"
---
# <a name="configure-advanced-threat-protection-in-azure-sql-managed-instance"></a>Configurer Advanced Threat Protection dans Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Advanced Threat Protection](../database/threat-detection-overview.md) pour une instance [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données. Advanced Threat Protection peut identifier une **injection SQL potentielle**, un **accès à partir d'un emplacement ou d'un centre de données inhabituel**, un **accès à partir d'un principal inconnu ou d'une application potentiellement dangereuse** et des **informations d'identification SQL par force brute**. Pour plus d'informations, consultez [Alertes de protection avancée des menaces](../database/threat-detection-overview.md#alerts).

Vous pouvez recevoir des notifications à propos des menaces détectées par l’intermédiaire de [notifications par e-mail](../database/threat-detection-overview.md#explore-detection-of-a-suspicious-event) ou du [portail Azure](../database/threat-detection-overview.md#explore-alerts-in-the-azure-portal).

[Advanced Threat Protection](../database/threat-detection-overview.md) fait partie de l’offre [Azure Defender pour SQL](../database/azure-defender-for-sql.md), package qui réunit les fonctionnalités de sécurité SQL avancées. Advanced Threat Protection est accessible et gérable par le biais du portail Azure Defender pour SQL central.

##  <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Accédez à la page de configuration de l’instance SQL Managed Instance que vous voulez protéger. Sous **Sécurité**, sélectionnez **Security Center**.
3. Sur la page de configuration Azure Defender pour SQL
   - **Activez** Azure Defender pour SQL.
   - Entrez votre adresse e-mail dans le champ **Envoyer des alertes à** pour recevoir des alertes de sécurité en cas de détection d'activités anormales sur la base de données.
   - Sélectionnez le **compte de stockage Azure** dans lequel sont enregistrés les enregistrements d’audit des menaces anormales.
   - Sélectionnez les **types d’Advanced Threat Protection** que vous souhaitez configurer. En savoir plus sur les [alertes Advanced Threat Protection](../database/threat-detection-overview.md)
4. Cliquez sur **Enregistrer** pour enregistrer la stratégie Azure Defender pour SQL, nouvelle ou mise à jour.

   :::image type="content" source="../database/media/azure-defender-for-sql/set-up-advanced-threat-protection-mi.png" alt-text="Configurer la protection avancée contre les menaces":::

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Advanced Threat Protection](../database/threat-detection-overview.md)
- Pour en savoir plus sur les instances gérées, consultez [Qu’est-ce qu’une instance Azure SQL Managed Instance ?](sql-managed-instance-paas-overview.md).
- En savoir plus sur [Advanced Threat Protection pour Azure SQL Database](../database/threat-detection-configure.md).
- En savoir plus sur [l’audit SQL Managed Instance](./auditing-configure.md).
- Découvrez plus en détail [Azure Security Center](../../security-center/security-center-introduction.md).