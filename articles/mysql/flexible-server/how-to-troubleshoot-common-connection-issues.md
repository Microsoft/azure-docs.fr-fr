---
title: Résoudre les problèmes de connexion - Azure Database pour MySQL - Serveur flexible
description: Découvrez comment résoudre les problèmes de connexion au serveur flexible Azure Database pour MySQL.
keywords: connexion mysql,chaîne de connexion,problèmes de connectivité,erreur persistante,erreur de connexion
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: troubleshooting
ms.date: 9/21/2020
ms.openlocfilehash: b328da01cd3b2ecb3eb5f183296eab808de27399
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929850"
---
# <a name="troubleshoot-connection-issues-to-azure-database-for-mysql---flexible-server"></a>Résoudre les problèmes de connexion à Azure Database pour MySQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Les problèmes de connexion peuvent avoir l’une des causes suivantes :

* Paramètres du pare-feu
* Expiration du délai de connexion
* Informations de connexion incorrectes
* Dépassement de la limite maximale sur certaines ressources du serveur flexible Azure Database pour MySQL

Dans cet article, nous allons expliquer comment corriger certaines erreurs courantes.

## <a name="troubleshoot-common-errors"></a>Résolution des erreurs courantes

Si l’application ne parvient pas à se connecter au serveur flexible Azure Database pour MySQL, cela indique généralement un problème avec l’un des éléments suivants :

* Connexion chiffrée à l’aide du protocole TLS/SSL : Le serveur flexible prend uniquement en charge les connexions chiffrées à l’aide du protocole TLS (Transport Layer Security) 1.2 et toutes **les connexions entrantes qui utilisent les protocoles TLS 1.0 et TLS 1.1 sont refusées**. Vous ne pouvez pas désactiver ni changer la version du protocole TLS. Découvrez-en plus sur la [connectivité chiffrée à l’aide du protocole TLS (Transport Layer Security) 1.2 dans Azure Database pour MySQL - Serveur flexible](./how-to-connect-tls-ssl.md).
- Serveur flexible dans *Accès privé (intégration au réseau virtuel)*  : Veillez à vous connecter depuis le même réseau virtuel que celui du serveur flexible. Reportez-vous à [Réseau virtuel du serveur flexible Azure Database pour MySQL]<!--(./concepts-networking-virtual-network.md)-->
- Serveur flexible avec *Accès public (adresses IP autorisées)*  : Vérifiez que le pare-feu est configuré pour autoriser les connexions à partir de votre client. Reportez-vous à [Créer et gérer des règles de pare-feu de serveur flexible à l’aide du portail Azure](./how-to-manage-firewall-portal.md).
* Configuration du pare-feu côté client : le pare-feu du client doit autoriser les connexions au serveur de base de données. Les adresses IP et les ports du serveur auxquels vous ne pouvez pas vous connecter doivent être autorisés ainsi que les noms d’application telles que MySQL sur certains pare-feu.
* Erreur utilisateur : Vous avez peut-être mal tapé les paramètres de connexion, comme le nom du serveur dans la chaîne de connexion.

### <a name="resolve-connectivity-issues"></a>Résoudre les problèmes de connectivité

* Reportez-vous à la [connectivité chiffrée à l’aide du protocole TLS (Transport Layer Security) 1.2 dans Azure Database pour MySQL - Serveur flexible](./how-to-connect-tls-ssl.md) pour en savoir plus sur les connexions chiffrées.
* Si vous utilisez l’option **Accès public (adresses IP autorisées)** , configurez des [règles de pare-feu](./how-to-manage-firewall-portal.md) pour autoriser l’adresse IP du client. Définissez une règle de pare-feu avec 0.0.0.0 pour adresse IP initiale et 255.255.255.255 pour adresse IP finale à des fins de test temporaire uniquement. Cette opération ouvrira le serveur à toutes les adresses IP. Si elle résout votre problème de connectivité, supprimez cette règle et créez une règle de pare-feu pour une adresse ou une plage d’adresses IP correctement bornée.
* Sur tous les pare-feu situés entre le client et Internet, assurez-vous que le port 3306 est ouvert pour les connexions sortantes.
* Vérifiez votre chaîne de connexion et d’autres paramètres de connexion. Reportez-vous aux chaînes de connexion prédéfinies dans la page **Chaînes de connexion** disponible pour votre serveur dans le portail Azure pour connaître les langages courants.

## <a name="next-steps"></a>Étapes suivantes
- [Utilisation de MySQL Workbench pour vous connecter et interroger des données sur le serveur flexible Azure Database pour MySQL](./connect-workbench.md).
- [Utilisation de PHP pour vous connecter et interroger des données sur le serveur flexible Azure Database pour MySQL](./connect-php.md).
- [Utilisation de Python pour vous connecter et interroger des données sur le serveur flexible Azure Database pour MySQL](./connect-python.md).
