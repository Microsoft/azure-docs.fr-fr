---
title: Bienvenue dans l’application Wingtips
description: Obtenez des informations sur les modèles de client de base de données, ainsi que l’exemple d’application Wingtips SaaS pour Azure SQL Database dans l’environnement cloud.
keywords: didacticiel sur les bases de données SQL
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: a2969ce6ceda0d1b71ec991b32f5b10acf9bfa12
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780374"
---
# <a name="the-wingtip-tickets-saas-application"></a>Application SaaS Wingtip Tickets
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La même application *Wingtip Tickets* SaaS est implémentée dans chacun des trois exemples. L’application est une simple application SaaS de réservation et de liste d’événements ciblant les lieux de petite taille (cinémas, clubs, etc.). Chaque lieu correspond à un locataire de l’application et dispose de ses propres données : détails sur le lieu, listes d’événements, clients, commandes de tickets, etc.  L’application, ainsi que les scripts de gestion et les didacticiels, présentent un scénario SaaS de bout en bout. Celui-ci inclut le provisionnement des locataires, le monitoring et la gestion des performances, la gestion des schémas et les rapports et analytiques entre locataires.

## <a name="three-saas-application-and-tenancy-patterns"></a>3 modèles d’application SaaS et de clients

Trois versions de l’application sont disponibles, chacune explorant un modèle différent d’architecture multilocataire de base de données sur Azure SQL Database.  Le premier utilise une application autonome par client avec sa propre base de données. La deuxième utilise une application multi-locataire avec une base de données par locataire. La troisième utilise une application multi-locataire avec des bases de données multi-locataires partitionnées.

![Trois modèles d’architecture mutualisée][image-three-tenancy-patterns]

 Chaque exemple inclut le code de l’application, et des scripts et didacticiels qui explorent plusieurs modèles de conception et de gestion.  Chaque exemple se déploie en moins de cinq minutes.  Les trois exemples peuvent être déployés côte à côte. Vous pouvez alors comparer les différences de conception et de gestion.

## <a name="standalone-application-per-tenant-pattern"></a>Modèle d’application autonome par client

Le modèle d’application autonome par client utilise une application à locataire unique avec une base de données pour chaque locataire. Chaque application du locataire, base de données comprise, est déployée dans un groupe de ressources Azure distinct. Le groupe de ressources peut être déployé dans l’abonnement du fournisseur de services ou dans celui du locataire qui est alors géré par le fournisseur au nom du locataire. Le modèle d’application autonome par client fournit l’isolation des locataires la plus grande, mais il est généralement le plus onéreux, car il n’existe aucune possibilité de partager des ressources entre plusieurs locataires.  Ce modèle convient parfaitement aux applications qui peuvent être plus complexes et qui sont déployées sur de plus petits nombres de clients.  Avec les déploiements autonomes, l’application peut être personnalisée pour chaque client plus facilement que dans d’autres modèles.  

Consultez les [didacticiels][docs-tutorials-for-wingtip-sa] et le code sur GitHub  [.../Microsoft/WingtipTicketsSaaS-StandaloneApp][github-code-for-wingtip-sa].

## <a name="database-per-tenant-pattern"></a>Modèle avec une base de données par locataire

Le modèle de base de données par locataire est efficace pour les fournisseurs de services qui sont concernés par l’isolation des locataires et souhaitent exécuter un service centralisé qui permet une utilisation rentable de ressources partagées. Une base de données est créée pour chaque locataire ou lieu, et toutes les bases de données sont gérées centralement. Les bases de données peuvent être hébergées dans des pools élastiques pour fournir une gestion des performances rentable et facile, qui tire parti des modèles de charge de travail imprévisibles des locataires. Une base de données de catalogue contient le mappage entre les locataires et leurs bases de données. Ce mappage est géré à l’aide des fonctionnalités de gestion de carte de partitions de la [bibliothèque cliente de base de données élastique](elastic-database-client-library.md), ce qui garantit une gestion efficace des connexions à l’application.

Consultez les [didacticiels][docs-tutorials-for-wingtip-dpt] et le code sur GitHub [.../Microsoft/WingtipTicketsSaaS-DbPerTenant][github-code-for-wingtip-dpt].

## <a name="sharded-multi-tenant-database-pattern"></a>Modèle de base de données multi-locataire partitionnée

Les bases de données multi-locataires sont efficaces pour les fournisseurs de services qui veulent un coût inférieur par locataire et n’ont besoin d’une forte isolation des locataires. Ce modèle permet la compression de gros volumes de locataires dans une base de données individuelle, réduisant ainsi le coût par locataire. Une échelle presque infinie est possible en partitionnant les locataires entre plusieurs bases de données. Une base de données de catalogue mappe à nouveau les locataires aux bases de données.  

Ce modèle autorise également un modèle *hybride* dans lequel vous pouvez optimiser les coûts avec plusieurs locataires dans une base de données, ou optimiser l’isolation avec un seul locataire dans sa propre base de données. Le choix peut être effectué locataire par locataire, soit lorsque le locataire est provisionné, soit ultérieurement, sans aucun impact sur l’application.  Ce modèle peut être utilisé efficacement lorsque des groupes de clients doivent être traités différemment. Par exemple, les clients à bas coût peuvent avoir des bases de données partagées, tandis que les clients premium peuvent être affectés à leurs propres bases de données. 

Consultez les [didacticiels][docs-tutorials-for-wingtip-mt] et le code sur GitHub [.../Microsoft/WingtipTicketsSaaS-MultiTenantDb][github-code-for-wingtip-mt].

## <a name="next-steps"></a>Étapes suivantes

#### <a name="conceptual-descriptions"></a>Descriptions conceptuelles

- Une explication plus détaillée des modèles d’architecture mutualisée d’application est disponible à l’adresse [Modèles de location de base de données SaaS multi-locataire][saas-tenancy-app-design-patterns-md]

#### <a name="tutorials-and-code"></a>Didacticiels et code

- Application autonome par client :
    - [Didacticiels pour une application autonome][docs-tutorials-for-wingtip-sa].
    - [Code pour une application autonome, sur GitHub][github-code-for-wingtip-sa].

- Base de données par locataire :
    - [Didacticiels pour les bases de données par locataire][docs-tutorials-for-wingtip-dpt].
    - [Code pour la base de données par locataire, sur GitHub][github-code-for-wingtip-dpt].

- Multi-locataire partitionné :
    - [Didacticiels pour un multi-locataire partitionné][docs-tutorials-for-wingtip-mt].
    - [Code pour un multi-locataire partitionné, sur GitHub][github-code-for-wingtip-mt].



<!-- Image references. -->

[image-three-tenancy-patterns]: media/saas-tenancy-welcome-wingtip-tickets-app/three-tenancy-patterns.png "Trois modèles d’architecture mutualisée."

<!-- Docs.ms.com references. -->

[saas-tenancy-app-design-patterns-md]: saas-tenancy-app-design-patterns.md

<!-- WWWeb http references. -->

[docs-tutorials-for-wingtip-sa]: ./saas-standaloneapp-get-started-deploy.md
[github-code-for-wingtip-sa]: https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp

[docs-tutorials-for-wingtip-dpt]: ./saas-dbpertenant-wingtip-app-overview.md
[github-code-for-wingtip-dpt]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant

[docs-tutorials-for-wingtip-mt]: ./saas-multitenantdb-get-started-deploy.md
[github-code-for-wingtip-mt]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb