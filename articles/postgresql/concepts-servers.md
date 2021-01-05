---
title: Serveurs - Azure Database pour PostgreSQL - Serveur unique
description: Cet article indique les éléments à prendre en considération et fournit des instructions pour configurer et gérer Azure Database pour PostgreSQL - Serveur unique.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: c8ae129cbad56e3bb63e055f9d843cac1da22fb1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710462"
---
# <a name="azure-database-for-postgresql---single-server"></a>Azure Database pour PostgreSQL - Serveur unique
Cet article présente des considérations et des instructions relatives à l’utilisation d’Azure Database pour PostgreSQL - Serveur unique.

## <a name="what-is-an-azure-database-for-postgresql-server"></a>Qu’est-ce qu’un serveur de base de données Azure pour PostgreSQL ?
Un serveur dans l’option de déploiement Azure Database pour PostgreSQL - Serveur unique est un point d’administration central pour plusieurs bases de données. Il s’agit de la structure de serveur PostgreSQL que vous connaissez peut-être en local. Plus précisément, le service PostgreSQL est géré, fournit des garanties de performances et propose des fonctionnalités et des accès au niveau du serveur.

Un serveur de base de données Azure pour PostgreSQL :

- est créé dans un abonnement Azure ;
- représente la ressource parente des bases de données ;
- fournit un espace de noms aux bases de données ;
- constitue un conteneur avec une sémantique à durée de vie longue (la suppression d’un serveur entraîne la suppression des bases de données autonomes qu’il contient) ;
- colocalise les ressources d’une région ;
- fournit un point de terminaison de connexion pour l’accès au serveur et aux bases de données ; 
- fournit l’étendue des stratégies de gestion qui s’appliquent à ses bases de données : connexions, pare-feu, utilisateurs, rôles, configurations, etc. ;
- est disponible dans plusieurs versions (pour plus d’informations, consultez [Versions de bases de données PostgreSQL prises en charge](concepts-supported-versions.md)).
- peut être étendu par les utilisateurs (pour plus d’informations, consultez la page [Extensions de PostgreSQL](concepts-extensions.md)).

Dans une base de données Azure Database pour serveur PostgreSQL, vous pouvez créer une ou plusieurs bases de données. Vous pouvez choisir de créer une seule base de données par serveur pour utiliser toutes les ressources, ou de créer plusieurs bases de données pour partager les ressources. Les tarifs sont structurés par serveur, en fonction de la configuration du niveau tarifaire, des vCores et du stockage (Go). Pour plus d’informations, consultez [Niveaux tarifaires](./concepts-pricing-tiers.md).

## <a name="how-do-i-connect-and-authenticate-to-an-azure-database-for-postgresql-server"></a>Comment se connecter et s’authentifier auprès d’un serveur de base de données Azure pour PostgreSQL ?
Les éléments suivants permettent de garantir un accès sécurisé à votre base de données :

|||
|:--|:--|
| **Authentification et autorisation** | Le serveur de base de données Azure pour PostgreSQL prend en charge l’authentification PostgreSQL native. Vous pouvez vous connecter et vous authentifier auprès du serveur avec les informations de connexion d’administrateur du serveur. |
| **Protocole** | Le service prend en charge un protocole par messages utilisé par PostgreSQL. |
| **TCP/IP** | Le protocole est pris en charge via TCP/IP et des sockets du domaine Unix. |
| **Pare-feu** | Pour renforcer la protection de vos données, une règle de pare-feu empêche tout accès à votre serveur et à ses bases de données tant que vous ne précisez pas quels ordinateurs sont autorisés. Consultez  [Règles de pare-feu d’un serveur Azure Database pour PostgreSQL](concepts-firewall-rules.md). |

## <a name="managing-your-server"></a>Gestion de votre serveur
Vous pouvez gérer des serveurs Azure Database pour PostgreSQL à l’aide du [Portail Azure](https://portal.azure.com) ou [d’Azure CLI](/cli/azure/postgres).

Lors de la création d’un serveur, vous configurez les informations d’identification pour votre utilisateur administrateur. L’utilisateur administrateur est l’utilisateur ayant les privilèges les plus élevés sur le serveur. Il appartient au rôle azure_pg_admin. Ce rôle ne dispose pas de toutes les autorisations du superutilisateur. 

L’attribut de superutilisateur PostgreSQL est affecté à azure_superuser, qui appartient au service managé. Vous n’avez pas accès à ce rôle.

Un serveur Azure Database pour PostgreSQL dispose de bases de données par défaut : 
- **postgres** : une base de données par défaut à laquelle vous pouvez vous connecter une fois que votre serveur est créé.
- **azure_maintenance** : cette base de données est utilisée pour séparer les processus qui fournissent le service managé des actions de l’utilisateur. Vous n’avez pas accès à cette base de données.
- **azure_sys** -Une base de données pour le Magasin des requêtes. Cette base de données n’accumule pas les données lorsque le Magasin des requêtes est désactivé ; il s’agit du paramètre par défaut. Pour plus d’informations, consultez [Vue d’ensemble du Magasin des requêtes](concepts-query-store.md).


## <a name="server-parameters"></a>Paramètres de serveur
Les paramètres de serveur PostgreSQL déterminent la configuration du serveur. Dans Azure Database pour PostgreSQL, la liste des paramètres peut être affichée et modifiée à l’aide du portail Azure ou d’Azure CLI. 

Azure Database pour PostgreSQL étant un service géré pour Postgres, ses paramètres configurables sont un sous-ensemble des paramètres d’une instance locale de Postgres (pour plus d’informations sur les paramètres de Postgres, consultez la [documentation de PostgreSQL](https://www.postgresql.org/docs/9.6/static/runtime-config.html)). Votre serveur Azure Database pour PostgreSQL est activé avec des valeurs par défaut pour chaque paramètre lors de la création. Certains paramètres qui nécessitent un redémarrage du serveur ou un accès de superutilisateur pour que les modifications entrent en vigueur ne peuvent pas être configurés par l’utilisateur.


## <a name="next-steps"></a>Étapes suivantes
- Vous trouverez une vue d’ensemble du service à la page  [Vue d’ensemble d'Azure Database pour PostgreSQL](overview.md).
- Pour obtenir des informations sur des quotas de ressources spécifiques et les limitations associées à votre **niveau de service**, consultez  [Niveaux de service](concepts-pricing-tiers.md).
- Pour plus d’informations sur la connexion au service, consultez la page  [Bibliothèques de connexions d'Azure Database pour PostgreSQL](concepts-connection-libraries.md).
- Affichez et modifiez les paramètres de serveur par le biais du [portail Azure](howto-configure-server-parameters-using-portal.md) ou [d’Azure CLI](howto-configure-server-parameters-using-cli.md).
