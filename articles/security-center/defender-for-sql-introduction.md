---
title: 'Azure Defender pour SQL : avantages et fonctionnalités'
description: Découvrez les avantages et les fonctionnalités d’Azure Defender pour SQL.
author: memildin
ms.author: memildin
ms.date: 12/13/2020
ms.topic: overview
ms.service: security-center
ms.custom: references_regions
manager: rkarlin
ms.openlocfilehash: e9a2259f1e15a94ad1f2a9eabf7d3a2f3ed419f1
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387800"
---
# <a name="introduction-to-azure-defender-for-sql"></a>Présentation d’Azure Defender pour SQL

Azure Defender pour SQL comprend deux plans Azure Defender qui étendent le [package de sécurité des données](../azure-sql/database/azure-defender-for-sql.md) d’Azure Security Center pour sécuriser les bases de données et leurs données où qu’elles se trouvent. 

> [!VIDEO https://www.youtube.com/embed/V7RdB6RSVpc]

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|**Azure Defender pour les serveurs de base de données Azure SQL** – Disponibilité générale (GA)<br>**Azure Defender pour les serveurs SQL sur les machines** - Disponibilité générale |
|Prix :|Les deux plans qui forment **Azure Defender pour SQL** sont facturés conformément aux indications de la [page des tarifs](security-center-pricing.md).|
|Versions de SQL protégées :|[SQL sur machines virtuelles Azure](../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)<br>[Serveurs SQL avec Azure Arc](https://docs.microsoft.com/sql/sql-server/azure-arc/overview)<br>Serveurs SQL locaux sur machines Windows sans Azure Arc<br>[Bases de données uniques](../azure-sql/database/single-database-overview.md) et [pools élastiques](../azure-sql/database/elastic-pool-overview.md) Azure SQL<br>[Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)<br>[Pool SQL dédié Azure Synapse Analytics (anciennement SQL DW)](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) US Gov<br>![Oui](./media/icons/yes-icon.png) China Gov (**Partiel**: Sous-ensemble d’alertes et évaluation des vulnérabilités pour les serveurs SQL. Les protections contre les menaces comportementales ne sont pas disponibles.)|
|||

## <a name="what-does-azure-defender-for-sql-protect"></a>Quels sont les éléments protégés par Azure Defender pour SQL ?

**Azure Defender pour SQL** comprend deux plans Azure Defender distincts :

- **Azure Defender pour les serveurs de base de données Azure SQL** protège :
    - [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md)
    - [Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)
    - [Pool SQL dédié dans Azure Synapse](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

- **Azure Defender pour les serveurs SQL Server sur les machines** étend les protections de vos serveurs SQL Azure natifs pour prendre en charge les environnements hybrides et protéger les serveurs SQL (toutes les versions prises en charge) hébergés dans Azure, d’autres environnements cloud et même les machines locales :
    - [SQL Server sur les machines virtuelles](https://azure.microsoft.com/services/virtual-machines/sql-server/)
    - Serveurs SQL locaux :
        - [SQL Server avec Azure Arc (préversion)](https://docs.microsoft.com/sql/sql-server/azure-arc/overview)
        - [SQL Server exécuté sur des machines Windows sans Azure Arc](../azure-monitor/platform/agent-windows.md)


## <a name="what-are-the-benefits-of-azure-defender-for-sql"></a>Quels sont les avantages d’Azure Defender pour SQL ?

Ces deux plans comprennent des fonctionnalités qui permettent d’identifier et d’atténuer les vulnérabilités potentielles des bases de données, et de détecter les activités anormales susceptibles d’indiquer des menaces pour les bases de données :

- [Évaluation des vulnérabilités](../azure-sql/database/sql-vulnerability-assessment.md) : service d’analyse qui vous permet de découvrir, de suivre les vulnérabilités de base de données potentielles et vous aide à les corriger. Les analyses d’évaluation fournissent une vue d’ensemble de l’état de sécurité de vos machines SQL, ainsi que des détails sur les résultats de la sécurité.

- [Protection avancée contre les menaces](../azure-sql/database/threat-detection-overview.md) : service de détection qui permet d’effectuer un monitoring continu des serveurs SQL pour repérer les menaces telles que l’injection de code SQL, les attaques par force brute et les abus de privilèges. Ce service fournit des alertes de sécurité orientées sur l’action dans Azure Security Center avec des détails sur l’activité suspecte, des conseils sur la façon de limiter les menaces et des options pour poursuivre vos investigations avec Azure Sentinel.


## <a name="what-kind-of-alerts-does-azure-defender-for-sql-provide"></a>Quels sont les types d’alertes fournis par Azure Defender pour SQL ?

Les alertes de sécurité enrichies par le renseignement sur les menaces sont déclenchées dans les cas suivants :

- **Attaques potentielles par injection de code SQL**, y compris les vulnérabilités détectées lorsque les applications génèrent une instruction SQL défectueuse dans la base de données
- **Modèles de requête et d’accès à la base de données anormaux**, par exemple, un nombre anormalement élevé d’échecs de tentatives de connexion avec des informations d’identification différentes (tentative d’attaque par force brute)
- **Activité suspecte de la base de données**, par exemple lorsqu’un utilisateur légitime accède à un ordinateur SQL Server à partir d’un ordinateur compromis qui a communiqué avec un serveur C&C à exploration crypto.

Les alertes fournissent des détails sur l’incident qui les a déclenchées, ainsi que des suggestions pour enquêter et contrer les menaces.



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert Azure Defender pour SQL.

> [!div class="nextstepaction"]
> [Analyser vos serveurs SQL Server à la recherche de vulnérabilités avec Azure Defender](defender-for-sql-usage.md)

Pour des informations connexes, consultez les articles suivants : 

- [Guide pratique pour activer Azure Defender pour les serveurs de base de données SQL](../azure-sql/database/azure-defender-for-sql.md)
- [Liste des alertes de sécurité pour les serveurs SQL](alerts-reference.md#alerts-sql-db-and-warehouse)
