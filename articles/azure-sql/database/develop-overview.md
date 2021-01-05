---
title: Vue d’ensemble du développement d’applications
description: En savoir plus sur les bibliothèques de connectivité disponibles et les meilleures pratiques pour les applications qui utilisent une connexion à la base de données SQL.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: genemi
ms.date: 11/14/2019
ms.custom: sqldbrb=2
ms.openlocfilehash: 7ea3456dbd1d0942cba48c97d70982ae088d73f9
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782975"
---
# <a name="application-development-overview---sql-database--sql-managed-instance"></a>Vue d’ensemble du développement des applications | SQL Database et SQL Managed Instance

[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Cet article explique les aspects de base qu’un développeur doit prendre en compte lors de l’écriture d’un code permettant d’établir une connexion à votre base de données dans Azure. Cet article s’applique à Azure SQL Database et Azure SQL Managed Instance.

## <a name="language-and-platform"></a>Langage et plateforme

Vous pouvez utiliser différents [langages de programmation et plateformes](connect-query-content-reference-guide.md) pour vous connecter à une base de données Azure SQL et l’interroger. Vous trouverez des [exemples d’applications](https://azure.microsoft.com/resources/samples/?service=sql-database&sort=0) qui peuvent vous permettre de vous connecter à la base de données.

Vous pouvez tirer parti des outils open source comme [cheetah](https://github.com/wunderlist/cheetah), [sql-cli](https://www.npmjs.com/package/sql-cli) et [VS Code](https://code.visualstudio.com/). En outre, Azure SQL Database fonctionne avec des outils Microsoft, tels que [Visual Studio](https://www.visualstudio.com/downloads/) et [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms). Le Portail Azure, PowerShell et les API REST peuvent également contribuer à accroître votre productivité.

## <a name="authentication"></a>Authentification

L’accès à Azure SQL Database est protégé par des connexions et des pare-feu. Azure SQL Database prend en charge les utilisateurs et les connexions de SQL Server et de [l’authentification Azure Active Directory](authentication-aad-overview.md). Les connexions Azure Active Directory ne sont disponibles que dans SQL Managed Instance. 

En savoir plus sur la [gestion des accès et des connexions aux bases de données](logins-create-manage.md).

## <a name="connections"></a>Connexions

Dans votre logique de connexion client, définissez le délai d’expiration sur 30 secondes. La valeur par défaut de 15 secondes est trop courte pour les connexions qui reposent sur Internet.

Si vous utilisez un [pool de connexions](/dotnet/framework/data/adonet/sql-server-connection-pooling), veillez à fermer la connexion dès que votre programme ne l’utilise plus activement et qu’il ne se prépare pas à le réutiliser.

Évitez les transactions de longue durée, car tout échec de connexion ou d’infrastructure peut restaurer la transaction. Si possible, fractionnez la transaction en plusieurs transactions plus petites et utilisez le [traitement par lot pour améliorer les performances](../performance-improve-use-batching.md).

## <a name="resiliency"></a>Résilience

Azure SQL Database est un service cloud où vous pouvez attendre des erreurs temporaires qui se produisent dans l’infrastructure sous-jacente ou dans la communication entre les entités du cloud. Bien qu’Azure SQL Database résiste aux échecs d’infrastructure transitive, ces échecs peuvent affecter votre connectivité. Lorsqu’une erreur temporaire se produit au moment de la connexion à SQL Database, votre code doit effectuer une [nouvelle tentative d’appel](troubleshoot-common-connectivity-issues.md). Nous vous recommandons d’utiliser une logique de nouvelle tentative basée sur une logique d’interruption afin d’éviter que le service ne soit inondé de tentatives simultanées de plusieurs clients. La logique de nouvelle tentative dépend des [messages d’erreur pour les programmes clients SQL Database](troubleshoot-common-errors-issues.md).

Pour plus d’informations sur la préparation aux événements de maintenance planifiée sur Azure SQL Database, consultez [Planification des événements de maintenance Azure dans Azure SQL Database](planned-maintenance.md).

## <a name="network-considerations"></a>Considérations relatives au réseau

- Assurez-vous que le pare-feu de l’ordinateur qui héberge votre programme client autorise les communications TCP sortantes sur le port 1433.  Plus d’informations : [Configurer un pare-feu Azure SQL Database](firewall-configure.md).
- Si votre programme client se connecte à la base de données SQL pendant que votre client s’exécute sur une machine virtuelle Azure, vous devez ouvrir certaines plages de ports sur la machine virtuelle. Plus d’informations : [Ports au-delà de 1433 pour ADO .NET 4.5 et SQL Database](adonet-v12-develop-direct-route-ports.md).
- Parfois, les connexions clientes à Azure SQL Database ignorent le proxy et interagissent directement avec la base de données. Les ports autres que le port 1433 deviennent importants. Pour plus d’informations, consultez [Architecture de connectivité d’Azure SQL Database](connectivity-architecture.md) et [Ports au-delà de 1433 pour ADO.NET 4.5 et SQL Database](adonet-v12-develop-direct-route-ports.md).
- Pour la configuration de mise en réseau d’une instance de SQL Managed Instance, consultez [Configuration réseau de SQL Managed Instance](../managed-instance/how-to-content-reference-guide.md#network-configuration).

## <a name="next-steps"></a>Étapes suivantes

Explorez toutes les fonctionnalités de [SQL Database](sql-database-paas-overview.md) et de [SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md).

Pour démarrer, consultez les guides relatifs à [Azure SQL Database](quickstart-content-reference-guide.md) et [Azure SQL Managed instances](../managed-instance/quickstart-content-reference-guide.md).