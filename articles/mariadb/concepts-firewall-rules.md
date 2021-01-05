---
title: Règles de pare-feu - Azure Database for MariaDB
description: Découvrez comment utiliser des règles de pare-feu pour activer les connexions à votre serveur Azure Database pour MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 7/17/2020
ms.openlocfilehash: 452ef4787812dbdf88eb541cf5a164f2888dddcc
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905432"
---
# <a name="azure-database-for-mariadb-server-firewall-rules"></a>Règles de pare-feu d’un serveur Azure Database for MariaDB
Le pare-feu empêche tout accès à votre serveur de base de données jusqu’à ce que vous spécifiiez les ordinateurs qui disposent d’autorisations. Le pare-feu octroie l’accès au serveur en fonction de l’adresse IP d’origine de chaque demande.

Pour configurer un pare-feu, créez des règles de pare-feu qui spécifient les plages d’adresses IP acceptables. Vous pouvez créer des règles de pare-feu au niveau du serveur.

**Règles de pare-feu :** ces règles permettent aux clients d’accéder à l’ensemble de votre serveur Azure Database for MariaDB, c’est-à-dire à toutes les bases de données qui se trouvent sur le même serveur logique. Les règles de pare-feu au niveau du serveur peuvent être configurées à l’aide du Portail Azure ou des commandes d’Azure CLI. Pour créer des règles de pare-feu au niveau du serveur, vous devez être le propriétaire de l’abonnement ou collaborateur.

## <a name="firewall-overview"></a>Présentation du pare-feu
Tous les accès au serveur Azure Database for MariaDB sont par défaut bloqués par le pare-feu. Pour pouvoir utiliser votre serveur à partir d’un autre ordinateur, vous devez spécifier une ou plusieurs règles de pare-feu au niveau du serveur afin de permettre l’accès à votre serveur. Utilisez les règles de pare-feu pour spécifier les plages d’adresses IP d’Internet à autoriser. L’accès au site web du Portail Azure proprement dit n’est pas affecté par les règles de pare-feu.

Les tentatives de connexion à partir d’Internet et d’Azure doivent franchir le pare-feu pour pouvoir atteindre votre base de données Azure Database for MariaDB, comme l’illustre le diagramme suivant :

![Exemple de flux de fonctionnement du pare-feu](./media/concepts-firewall-rules/1-firewall-concept.png)

## <a name="connecting-from-the-internet"></a>Connexion à partir d’Internet
Les règles de pare-feu au niveau du serveur s’appliquent à toutes les bases de données qui se trouvent sur le serveur Azure Database for MariaDB.

Si l’adresse IP de la demande appartient à une des plages spécifiées dans les règles de pare-feu au niveau du serveur, la connexion est accordée.

Si l’adresse IP de la demande n’appartient pas aux plages spécifiées dans les règles de pare-feu au niveau du serveur ou de la base de données, la demande de connexion échoue.

## <a name="connecting-from-azure"></a>Connexion à partir d’Azure
Il est recommandé de trouver l’adresse IP sortante d’une application ou d’un service et d’autoriser explicitement l’accès à ces plages ou adresses IP individuelles. Par exemple, vous pouvez trouver l’adresse IP sortante d’un service Azure App Service ou utiliser une adresse IP publique liée à une machine virtuelle ou à une autre ressource (voir ci-dessous pour plus d’informations sur la connexion avec l’adresse IP privée d’une machine virtuelle sur les points de terminaison de service). 

Si aucune adresse IP sortante fixe n’est disponible pour votre service Azure, vous pouvez envisager d’activer les connexions à partir de toutes les adresses IP du centre de données Azure. Ce paramètre peut être activé à partir du portail Azure en affectant à l’option **Autoriser l’accès aux services Azure** la valeur **ACTIVÉ** à partir du volet **Sécurité de la connexion**, puis en cliquant sur **Enregistrer**. À partir de l’interface de ligne de commande Azure, un paramètre de règle de pare-feu avec une adresse de début et de fin égale à 0.0.0.0 effectue l’opération équivalente. Si la tentative de connexion n’est pas autorisée, la demande n’atteint pas le serveur Azure Database for MariaDB.

> [!IMPORTANT]
> L’option **Autoriser l’accès aux services Azure** configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.
> 

![Configurer Autoriser l’accès aux services Azure dans le portail](./media/concepts-firewall-rules/allow-azure-services.png)

### <a name="connecting-from-a-vnet"></a>Connexion à partir d’un réseau virtuel
Pour vous connecter de manière sécurisée à votre serveur Azure Database for MariaDB à partir d’un réseau virtuel, vous pouvez utiliser des [points de terminaison de service de réseau virtuel](./concepts-data-access-security-vnet.md). 

## <a name="programmatically-managing-firewall-rules"></a>Gestion par programmation des règles de pare-feu
En dehors du Portail Azure, les règles de pare-feu peuvent être gérées par programmation à l’aide d’Azure CLI. 

Voir aussi [Créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide de l'interface de ligne de commande Azure](./howto-manage-firewall-cli.md).

## <a name="troubleshooting-firewall-issues"></a>Résolution des problèmes de pare-feu
Tenez compte des points suivants quand l’accès au service de serveur Azure Database for MariaDB présente un comportement anormal :

* **Les modifications apportées à la liste d’approbation n’ont pas encore pris effet :** jusqu’à cinq minutes peuvent s’écouler avant que les modifications apportées à la configuration du pare-feu du serveur Azure Database for MariaDB ne soient effectives.

* **La connexion n’est pas autorisée ou un mot de passe incorrect a été utilisé :** si une connexion n’a pas d’autorisations sur le serveur Azure Database for MariaDB ou que le mot de passe est incorrect, la connexion au serveur Azure Database for MariaDB est refusée. Créer un paramètre de pare-feu permet uniquement aux clients de tenter de se connecter à votre serveur ; chaque client doit fournir les informations d’identification de sécurité nécessaires.

* **Adresse IP dynamique :** si vous avez une connexion Internet avec un adressage IP dynamique et que le pare-feu demeure infranchissable, vous pouvez essayer l’une des solutions suivantes :

   * Demandez à votre fournisseur d’accès Internet (FAI) la plage d’adresses IP affectée à vos ordinateurs clients qui accèdent au serveur Azure Database for MariaDB, puis ajoutez cette plage dans une règle de pare-feu.

   * Obtenez un adressage IP statique à la place pour vos ordinateurs clients, puis ajoutez les adresses IP en tant que règles de pare-feu.

* **L’adresse IP du serveur semble être publique :** les connexions au serveur Azure Database for MariaDB sont routées par le biais d’une passerelle Azure accessible publiquement. Toutefois, l’adresse IP réelle du serveur est protégée par le pare-feu. Pour plus d’informations, consultez l’article [Architecture de connectivité](concepts-connectivity-architecture.md). 

* **Impossible de se connecter à partir d’une ressource Azure avec l’adresse IP autorisée :** vérifiez si le point de terminaison de service **Microsoft.Sql** est activé pour le sous-réseau à partir duquel vous vous connectez. Si **Microsoft.Sql** est activé, il signifie que vous souhaitez utiliser uniquement les [règles de point de terminaison de service de réseau virtuel](concepts-data-access-security-vnet.md) sur ce sous-réseau.

   Par exemple, vous pouvez voir l’erreur suivante si vous vous connectez à partir d’une machine virtuelle Azure dans un sous-réseau où **Microsoft.Sql** est activé, mais qui n’a pas de règle de réseau virtuel correspondante : `FATAL: Client from Azure Virtual Networks is not allowed to access the server`.

* **La règle de pare-feu n’est pas disponible pour le format IPv6 :** Les règles de pare-feu doivent être au format IPv4. Si vous spécifiez des règles de pare-feu au format IPv6, l’erreur de validation s’affiche.

## <a name="next-steps"></a>Étapes suivantes
- [Créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide du portail Azure](./howto-manage-firewall-portal.md)
- [Create and manage Azure Database for MariaDB firewall rules using Azure CLI](./howto-manage-firewall-cli.md) (Créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide de l’interface de ligne de commande Azure)
- [Points de terminaison de service de réseau virtuel dans Azure Database for MariaDB](./concepts-data-access-security-vnet.md)
