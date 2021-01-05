---
title: Serveurs - Azure Database for MariaDB
description: Cette rubrique comporte des considérations et des instructions relatives à l’utilisation des serveurs Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 4d8293258083ea3e8d0172f510e5b41e91328736
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541060"
---
# <a name="server-concepts-in-azure-database-for-mariadb"></a>Concepts de serveur dans Azure Database for MariaDB
Cet article comporte des considérations et des instructions relatives à l’utilisation des serveurs Azure Database for MariaDB.

## <a name="what-is-an-azure-database-for-mariadb-server"></a>Qu’est-ce qu’un serveur Azure Database for MariaDB ?

Un serveur Azure Database for MariaDB est un point d’administration central pour plusieurs bases de données. Il s’agit de la structure de serveur MariaDB que vous connaissez peut-être en local. Plus précisément, le service Azure Database for MariaDB est géré, fournit des garanties de performances et propose des fonctionnalités et un accès au niveau du serveur.

Un serveur Azure Database for MariaDB :

- est créé dans un abonnement Azure ;
- représente la ressource parente des bases de données ;
- fournit un espace de noms aux bases de données ;
- constitue un conteneur avec une sémantique à durée de vie longue (la suppression d’un serveur entraîne la suppression des bases de données autonomes qu’il contient) ;
- colocalise les ressources d’une région ;
- fournit un point de terminaison de connexion pour l’accès au serveur et aux bases de données ;
- fournit l’étendue des stratégies de gestion qui s’appliquent à ses bases de données : connexions, pare-feu, utilisateurs, rôles, configurations, etc. ;
- est disponible dans le moteur MariaDB version 10.2. Pour plus d’informations, consultez [Versions prises en charge des bases de données Azure Database for MariaDB](./concepts-supported-versions.md).

Dans un serveur Azure Database for MariaDB, vous pouvez créer une ou plusieurs bases de données. Vous pouvez choisir de créer une seule base de données par serveur pour utiliser toutes les ressources, ou de créer plusieurs bases de données pour partager les ressources. Les tarifs sont structurés par serveur, en fonction de la configuration du niveau tarifaire, des vCores et du stockage (Go). Pour plus d’informations, consultez [Niveaux tarifaires](./concepts-pricing-tiers.md).

## <a name="how-do-i-secure-an-azure-database-for-mariadb-server"></a>Comment sécuriser un serveur Azure Database for MariaDB ?

Les éléments suivants permettent de garantir un accès sécurisé à votre base de données.

|||
| :--| :--|
| **Authentification et autorisation** | Le serveur Azure Database for MariaDB prend en charge l’authentification MySQL native. Vous pouvez vous connecter et vous authentifier auprès d’un serveur avec les informations de connexion d’administrateur du serveur. |
| **Protocole** | Le service prend en charge un protocole par messages utilisé par MySQL. |
| **TCP/IP** | Le protocole est pris en charge via TCP/IP et des sockets du domaine Unix. |
| **Pare-feu** | Pour aider à protéger vos données, une règle de pare-feu empêche tout accès à votre serveur de base de données tant que vous ne spécifiez pas les ordinateurs autorisés. Consultez [Règles de pare-feu d’un serveur Azure Database for MariaDB](./concepts-firewall-rules.md). |
| **SSL** | Le service prend en charge l’application des connexions SSL entre vos applications et votre serveur de base de données. Voir [Configuration de la connectivité SSL dans votre application pour se connecter de manière sécurisée à Azure Database for MariaDB](./howto-configure-ssl.md). |

## <a name="stopstart-an-azure-database-for-mariadb-preview"></a>Arrêter/Démarrer Azure Database for MariaDB (préversion)
Azure Database for MariaDB vous donne la possibilité d’**arrêter** le serveur si vous ne l’utilisez pas, et de le **démarrer** lorsque vous reprenez votre activité. Cela permet essentiellement de réduire les coûts sur les serveurs de base de données et de payer uniquement la ressource en cours d’utilisation. Cela devient encore plus important pour les charges de travail de développement et de test, et lorsque vous utilisez uniquement le serveur pour une partie de la journée. Lorsque vous arrêtez le serveur, toutes les connexions actives sont supprimées. Plus tard, lorsque vous souhaiterez remettre le serveur en ligne, vous pourrez utiliser le [portail Azure](../mysql/how-to-stop-start-server.md) ou l’interface [CLI](../mysql/how-to-stop-start-server.md).

Lorsque le serveur se trouve à l’état **Arrêté**, le calcul du serveur n’est pas facturé. Toutefois, le stockage continue à être facturé tant que le stockage du serveur est conservé pour s’assurer que les fichiers de données sont disponibles lors du redémarrage du serveur.

> [!IMPORTANT]
> Lorsque vous **arrêtez** le serveur, il reste dans cet état durant les 7 jours suivants. Si vous ne le **démarrez** pas manuellement pendant cette période, le serveur sera automatiquement démarré à la fin des 7 jours. Vous pouvez choisir de l’**arrêter** de nouveau si vous n’utilisez pas le serveur.

Pendant l’arrêt du serveur, aucune opération de gestion ne peut être effectuée sur le serveur. Afin de modifier les paramètres de configuration sur le serveur, vous devez [démarrer le serveur](../mysql/how-to-stop-start-server.md).

### <a name="limitations-of-stopstart-operation"></a>Limitations de l’opération Arrêter/Démarrer
- Pas de prise en charge avec les configurations de réplica en lecture (source et réplicas).

## <a name="how-do-i-manage-a-server"></a>Comment gérer un serveur ?
Vous pouvez gérer des serveurs Azure Database for MariaDB à l’aide du portail Azure ou de l’interface Azure CLI.

## <a name="next-steps"></a>Étapes suivantes
- Vous trouverez une vue d’ensemble du service dans la page [Vue d’ensemble d’Azure Database for MariaDB](./overview.md).
- Pour plus d’informations sur les quotas de ressources et les limitations associés à votre **niveau de service**, consultez la page [Niveaux de service](./concepts-pricing-tiers.md).

<!-- - For information about connecting to the service, see [Connection libraries for Azure Database for MariaDB](./concepts-connection-libraries.md). -->
