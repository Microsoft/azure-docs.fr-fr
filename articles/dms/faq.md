---
title: FAQ – Azure Database Migration Service
description: Questions fréquemment posées sur l’utilisation d’Azure Database Migration Service pour effectuer des migrations de bases de données.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: bf6e61ef3dfb1c50166cf17168b4deeb21e958d7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962907"
---
# <a name="faq-about-using-azure-database-migration-service"></a>Questions fréquentes (FAQ) sur l’utilisation d’Azure Database Migration Service

Cet article répertorie les questions fréquemment posées sur l’utilisation d’Azure Database Migration Service, ainsi que les réponses associées.

## <a name="overview"></a>Vue d’ensemble

**Q. Qu’est-ce qu’Azure Database Migration Service ?**
Azure Database Migration Service est un service complètement managé conçu pour permettre la migration homogène de plusieurs sources de base de données vers des plateformes de données Azure avec un temps d’arrêt minimal. Le service est actuellement en disponibilité générale, les efforts de développement en cours étant axés sur les aspects suivants :

* Fiabilité et performances.
* Ajout itératif de paires source/cible.
* Investissement continu dans les migrations sans problèmes.

**Q. Quelles sont les paires sources/cibles actuellement prises en charge par Azure Database Migration Service ?**
Le service prend actuellement en charge différentes paires sources/cibles, ou différents scénarios de migration. Pour obtenir la liste complète des états de chaque scénario de migration disponible, consultez l’article [État des scénarios de migration pris en charge par Azure Database Migration Service](./resource-scenario-status.md).

D’autres scénarios de migration sont disponibles en préversion et nécessitent l’envoi d’une demande via le site DMS Preview. Pour obtenir la liste complète des scénarios en préversion et vous inscrire pour participer à une de ces offres, consultez le [site DMS Preview](https://aka.ms/dms-preview/).

**Q. Quelles sont les versions de SQL Server prises en charge comme sources par Azure Database Migration Service ?**
En ce qui concerne la migration à partir de SQL Server, les sources prises en charge par Azure Database Migration Service vont de SQL Server 2005 à SQL Server 2019.

**Q : Lors de l’utilisation d’Azure Database Migration Service, quelle est la différence entre une migration en ligne et une migration hors connexion ?**
Vous pouvez utiliser Azure Database Migration Service pour effectuer des migrations en ligne et des migrations hors connexion. Dans le cas d’une migration *hors connexion*, le temps d’arrêt de l’application commence quand la migration commence. Dans le cas d’une migration *en ligne*, le temps d’arrêt est limité à la durée du basculement à la fin de la migration. Nous vous suggérons de tester une migration hors connexion pour déterminer si le temps d’arrêt est acceptable ; dans le cas contraire, privilégiez une migration en ligne.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium. Pour plus d’informations, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/database-migration/) du service Azure Database Migration Service.

**Q. Comment Azure Database Migration Service se distingue des autres outils de migration de bases de données Microsoft, comme l’Assistant Migration de bases de données ou l’Assistant Migration SQL Server ?**
Azure Database Migration Service est la méthode recommandée pour la migration de bases de données à grande échelle vers Microsoft Azure. Pour plus d’informations sur la façon dont Azure Database Migration Service se distingue des autres outils de migration de bases de données Microsoft et pour obtenir des recommandations sur l’utilisation du service pour différents scénarios, consultez le billet de blog [Differentiating Microsoft’s Database Migration Tools and Services](https://techcommunity.microsoft.com/t5/microsoft-data-migration/differentiating-microsoft-s-database-migration-tools-and/ba-p/368529).

**Q. Comment Azure Database Migration Service se distingue du produit Azure Migrate ?**
Azure Migrate facilite la migration de machines virtuelles locales vers Azure IaaS. Le service évalue la pertinence de la migration et le dimensionnement en fonction des performances, et fournit des estimations de coût pour l’exécution de vos machines locales dans Azure. Azure Migrate est particulièrement utile pour les migrations de type lift-and-shift des charges de travail basées sur des machines virtuelles locales vers des machines virtuelles IaaS Azure. Cependant, contrairement à Azure Database Migration Service, Azure Migrate n'est pas un service de migration de bases de données spécialisé pour les plateformes de base de données relationnelle PaaS Azure, comme Azure SQL Database ou Azure SQL Managed Instance.

## <a name="setup"></a>Programme d’installation

**Q. Quels sont les prérequis pour utiliser Azure Database Migration Service ?**
Il existe plusieurs prérequis pour garantir qu’Azure Database Migration Service s’exécute correctement lors de la migration de bases de données. Certaines des conditions préalables s’appliquent à tous les scénarios (paires source-cible) pris en charge par le service, tandis que d’autres sont propres à un scénario spécifique.

Les conditions préalables associées à Azure Database Migration Service communes à tous les scénarios de migration pris en charge incluent le besoin de :

* Créez un Réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Assurez-vous que les règles de groupe de sécurité de votre réseau virtuel ne bloquent pas les ports de communication suivants : 443, 53, 5671 à 5672, 9350 à 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.

Pour obtenir la liste de tous les prérequis permettant des scénarios de migration spécifiques à l’aide d’Azure Database Migration Service, consultez les tutoriels associés dans la [documentation](./dms-overview.md) d’Azure Database Migration Service sur docs.microsoft.com.

**Q. Comment trouver l’adresse IP d’Azure Database Migration Service afin de créer une liste d’autorisations pour les règles de pare-feu utilisées pour accéder à ma base de données source pour la migration ?**
Vous devrez peut-être ajouter des règles de pare-feu autorisant Azure Database Migration Service à accéder à votre base de données source pour la migration. L’adresse IP du service est dynamique, mais si vous utilisez ExpressRoute, cette adresse est attribuée en privé par votre réseau d’entreprise. Le moyen le plus simple d’identifier l’adresse IP appropriée est de regarder dans le même groupe de ressources que votre ressource Azure Database Migration Service provisionnée et y rechercher l’interface réseau associée. Généralement, le nom de la ressource d’interface réseau commence par le préfixe de la carte réseau, et est suivi d’une séquence de caractères et de chiffres unique (par exemple, NIC-jj6tnztnmarpsskr82rbndyp). En sélectionnant cette ressource d’interface réseau, vous pouvez voir l’adresse IP devant être incluse dans la liste d’autorisations sur la page du portail Azure de présentation des ressources.

Par ailleurs, vous devez peut-être inclure la source du port que SQL Server écoute sur la liste d’autorisations. Par défaut, il s’agit du port 1433, mais le SQL Server source peut être configuré pour écouter d’autres ports également. Dans ce cas, vous devez aussi inclure ces ports sur la liste d’autorisations. Vous pouvez déterminer le port écouté par SQL Server à l’aide de la requête de vue de gestion dynamique :

```sql
    SELECT DISTINCT
        local_tcp_port
    FROM sys.dm_exec_connections
    WHERE local_tcp_port IS NOT NULL
```

Vous pouvez également déterminer le port écouté par SQL Server en interrogeant le journal des erreurs SQL Server :

```sql
    USE master
    GO
    xp_readerrorlog 0, 1, N'Server is listening on'
    GO
```

**Q. Comment configurer un Réseau virtuel Microsoft Azure ?**
Même si plusieurs didacticiels Microsoft peuvent vous présenter le processus de configuration d’un réseau virtuel, la documentation officielle est disponible dans l’article [Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).

## <a name="usage"></a>Usage

**Q. Quel est le récapitulatif des étapes nécessaires pour utiliser Azure Database Migration Service afin de migrer une base de données ?**
Lors d’une migration de base de données classique et simple, vous effectuez les opérations suivantes :

1. Créer des bases de données cibles.
2. Évaluer vos bases de données sources.
    * Pour les migrations homogènes, évaluez vos bases de données existantes avec [DMA](https://www.microsoft.com/download/details.aspx?id=53595).
    * Pour les migrations hétérogènes (à partir de sources concurrentes), évaluez vos bases de données existantes avec [SSMA](/sql/ssma/sql-server-migration-assistant). Vous utilisez également SSMA pour convertir des objets de base de données et migrer le schéma vers votre plateforme cible.
3. Créer une instance Azure Database Migration Service.
4. Créer un projet de migration spécifiant la ou les bases de données sources, la ou les bases de données cibles, et les tables à migrer.
5. Démarrer le chargement complet.
6. Choisir la validation ultérieure.
7. Effectuer un basculement manuel de votre environnement de production vers la nouvelle base de données cloud.

## <a name="troubleshooting-and-optimization"></a>Résolution des problèmes et optimisation

**Q. Je configure un projet de migration dans DMS et je rencontre des difficultés pour me connecter à ma base de données source. Que dois-je faire ?**
Si vous avez des difficultés à vous connecter à votre système de base de données source quand vous travaillez sur la migration, créez une machine virtuelle dans le réseau virtuel, avec laquelle vous configurez votre instance DMS. Dans la machine virtuelle, vous devez être en mesure d’effectuer un test de connexion, par exemple en utilisant un fichier UDL pour tester une connexion à SQL Server ou en téléchargeant Robo 3T pour tester les connexions MongoDB. Si le test de connexion réussit, vous ne devriez pas avoir de problème de connexion à votre base de données source. Si le test de connexion échoue, contactez votre administrateur réseau.

**Q. Pourquoi Azure Database Migration Service est arrêté ou indisponible ?**
Si l’utilisateur arrête de manière explicite Azure Database Migration Service (DMS) ou si le service est inactif pendant une période de 24 heures, le service est dans un état arrêté ou de pause automatique. Dans tous les cas, le service est indisponible et dans un état arrêté.  Pour reprendre les migrations actives, redémarrez le service.

**Q. Existe-t-il des recommandations pour optimiser les performances d’Azure Database Migration Service ?**
Vous pouvez effectuer quelques opérations pour accélérer la migration de votre base de données à l’aide du service :

* Utilisez le niveau tarifaire Usage général avec plusieurs processeurs lorsque vous créez votre instance de service pour permettre au service de tirer parti de multiples processeurs virtuels pour la parallélisation et le transfert plus rapide des données.
* Montez temporairement en puissance votre instance cible Azure SQL Database vers la référence SKU de niveau Premium pendant l’opération de migration des données pour réduire la limitation SQL Database pouvant avoir un impact sur les activités de transfert de données lors de l’utilisation de références SKU de niveau inférieur.

## <a name="next-steps"></a>Étapes suivantes

Pour une présentation d’Azure Database Migration Service et de la mise à disponibilité régionale, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](dms-overview.md).