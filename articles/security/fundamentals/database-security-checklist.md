---
title: Liste de contrôle de la sécurité des bases de données Azure | Microsoft Docs
description: Utilisez la liste de vérification de sécurité de la base de données Azure pour vous assurer de poser les questions importantes relatives à la sécurité du cloud computing.
services: security
documentationcenter: na
author: unifycloud
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: tomsh
ms.openlocfilehash: e9845eaf6d84caad3ddb8c7e615cb72f71c91f3b
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94412831"
---
# <a name="azure-database-security-checklist"></a>Liste de contrôle de la sécurité des bases de données Azure

Pour aider à améliorer la sécurité, les bases de données Azure incluent un nombre de contrôles de sécurité intégrés que vous pouvez utiliser pour limiter et contrôler l’accès.

notamment :

-    Pare-feu qui vous permet de créer des [règles de pare-feu](../../azure-sql/database/firewall-configure.md) limitant la connectivité par adresse IP
-    Pare-feu au niveau du serveur accessible depuis le portail Azure
-    Règles de pare-feu au niveau de la base de données accessibles à partir de SSMS
-    Connectivité sécurisée à votre base de données avec des chaînes de connexion sécurisées
-    Gestion de l’accès des utilisateurs
-    Chiffrement des données
-    Audit de base de données SQL
-    Détection de menaces pour les bases de données SQL

## <a name="introduction"></a>Introduction
Le cloud computing requiert de nouveaux modèles de sécurité avec lesquels de nombreux utilisateurs d’applications, administrateurs de bases de données et programmeurs ne sont pas familiarisés. Par conséquent, certaines organisations hésitent à implémenter une infrastructure de cloud pour la gestion des données en raison des risques de sécurité perçus. Pour une grande part toutefois, ces inquiétudes peuvent être calmées par une meilleure compréhension des fonctionnalités de sécurité intégrées dans Microsoft Azure et Microsoft Azure SQL Database.

## <a name="checklist"></a>Liste de contrôle
Nous vous recommandons de lire l’article [Azure Database Security Best Practices](../../azure-sql/database/security-best-practice.md) (Meilleures pratiques en matière de sécurité des bases de données Azure) avant de passer cette liste de contrôle en revue. Vous pourrez tirer le meilleur parti de cette liste de contrôle après avoir découvert ces meilleures pratiques. Vous pourrez ensuite utiliser cette liste de contrôle pour vous assurer que vous avez géré les problèmes importants en matière de sécurité des bases de données Azure.


|Catégorie de la liste de contrôle| Description|
| ------------ | -------- |
|**Protection les données**||
| <br> Chiffrement en mouvement/transit| <ul><li>[Transport Layer Security](/windows-server/security/tls/transport-layer-security-protocol), pour le chiffrement des données lorsqu’elles se déplacent vers les réseaux.</li><li>Les bases de données requièrent une communication sécurisée depuis les clients, avec l’utilisation du protocole [TDS (Tabular Data Stream)](/openspecs/windows_protocols/ms-tds/893fcc7e-8a39-4b3c-815a-773b7b982c50) sur le protocole TLS (Transport Layer Security).</li></ul> |
|<br>Chiffrement au repos| <ul><li>[Transparent Data Encryption](../../azure-sql/database/transparent-data-encryption-tde-overview.md), lorsque les données inactives sont stockées physiquement dans un format numérique.</li></ul>|
|**Contrôle des accès**||  
|<br> Accès à la base de données | <ul><li>[Authentification](../../azure-sql/database/logins-create-manage.md) AD (Azure Active Directory), qui utilise des identités gérées par Azure Active Directory.</li><li>[Autorisation](../../azure-sql/database/logins-create-manage.md), pour accorder aux utilisateurs les privilèges minimum nécessaires.</li></ul> |
|<br>Accès aux applications| <ul><li>[Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) (avec une stratégie de sécurité, tout en limitant l’accès au niveau des lignes selon l’identité des utilisateurs, leur rôle ou le contexte d’exécution).</li><li>[Masquage des données dynamiques](../../azure-sql/database/dynamic-data-masking-overview.md) (avec autorisation et stratégie, limite l’exposition des données sensibles en les masquant pour les utilisateurs sans privilège).</li></ul>|
|**Surveillance proactive**||  
| <br>Suivi et détection| <ul><li>L’[audit](../../azure-sql/database/auditing-overview.md) suit les événements de base de données et les écrit dans un journal d’audit ou d’activité dans votre [compte de stockage Azure](../../storage/common/storage-account-create.md).</li><li>Suivi de l’intégrité des bases de données Azure à l’aide des [journaux d’activité Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md).</li><li>La [détection des menaces](../../azure-sql/database/threat-detection-configure.md) permet de détecter les activités base de données anormales indiquant la présence potentielle de menaces de sécurité pour la base de données. </li></ul> |
|<br>Azure Security Center| <ul><li>[Supervision des données](../../security-center/security-center-remediate-recommendations.md), avec Azure Security Center comme solution de supervision de la sécurité centralisée pour SQL et d’autres services Azure.</li></ul>|        

## <a name="conclusion"></a>Conclusion
Azure Database est une plateforme robuste de base de données, avec un éventail complet de fonctionnalités de sécurité qui répondent à de nombreuses exigences en matière de conformité réglementaire et organisationnelles. Vous pouvez facilement protéger les données en contrôlant l’accès physique à vos données et à l’aide de diverses options de sécurité des données au niveau des fichiers, des colonnes ou des lignes avec le chiffrement transparent des données, le chiffrement au niveau des cellules ou de la sécurité au niveau des lignes. Always Encrypted active également des opérations sur les données chiffrées, ce qui simplifie le processus des mises à jour d’application. À son tour, l’accès aux journaux d’activité d’audit de l’activité SQL Database vous offre les informations dont vous avez besoin, ce qui vous permet de savoir quand et comment les utilisateurs ont accédé aux données.

## <a name="next-steps"></a>Étapes suivantes
Quelques étapes simples suffisent pour améliorer la protection d’une base de données contre les utilisateurs malveillants ou tout accès non autorisé. Ce didacticiel vous apprend à effectuer les opérations suivantes :

- Définissez des [règles de pare-feu](../../azure-sql/database/firewall-configure.md) pour votre serveur ou base de données.
- Protéger vos données à l’aide du [chiffrement](/sql/relational-databases/security/encryption/sql-server-encryption).
- Activer l’[audit Azure SQL Database](../../azure-sql/database/auditing-overview.md).