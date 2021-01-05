---
title: Sécurité - Azure Database pour MySQL
description: Présente une vue d’ensemble des fonctionnalités de sécurité dans Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 90855059461fcd5f8ed8d2733d2b6d4addaccde3
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535042"
---
# <a name="security-in-azure-database-for-mysql"></a>Security dans Azure Database pour MySQL

Il existe plusieurs couches de sécurité qui permettent de protéger les données d’un serveur Azure Database pour MySQL. Cet article décrit ces différentes options de sécurité.

## <a name="information-protection-and-encryption"></a>Protection et chiffrement des informations

### <a name="in-transit"></a>En transit
Azure Database pour MySQL sécurise vos données en chiffrant les données en transit à l’aide du protocole TLS (Transport Layer Security). Le chiffrement (SSL/TLS) est appliqué par défaut.

### <a name="at-rest"></a>Au repos
Le service Azure Database pour MySQL utilise le module de chiffrement conforme à la norme FIPS 140-2 pour chiffrer le stockage des données au repos. Toutes les données, notamment les sauvegardes, sont chiffrées sur le disque, y compris les fichiers temporaires créés durant l’exécution des requêtes. Le service utilise le chiffrement AES 256 bits inclus dans le chiffrement de stockage Azure, et les clés sont gérées par le système. Le chiffrement de stockage est toujours activé et ne peut pas être désactivé.


## <a name="network-security"></a>Sécurité du réseau
Les connexions à un serveur Azure Database pour MySQL sont d’abord routées par le biais d’une passerelle régionale. La passerelle a une adresse IP accessible publiquement, tandis que les adresses IP du serveur sont protégées. Pour plus d’informations sur la passerelle, consultez l’article [Architecture de connectivité](concepts-connectivity-architecture.md).  

Tout serveur Azure Database pour MySQL est créé avec un pare-feu qui bloque toutes les connexions externes. Ces connexions peuvent atteindre la passerelle, mais elles ne permettent pas l’accès au serveur. 

### <a name="ip-firewall-rules"></a>Règles de pare-feu IP
Les règles de pare-feu IP octroient l’accès aux serveurs en fonction de l’adresse IP d’origine de chaque requête. Pour plus d’informations, consultez la [vue d’ensemble des règles de pare-feu](concepts-firewall-rules.md).

### <a name="virtual-network-firewall-rules"></a>Règles de pare-feu de réseau virtuel
Les points de terminaison de service de réseau virtuel étendent votre connectivité de réseau virtuel via le réseau principal Azure. À l’aide de règles de réseau virtuel, vous pouvez configurer votre serveur Azure Database pour MySQL pour autoriser les connexions à partir de sous-réseaux sélectionnés dans un réseau virtuel. Pour plus d’informations, consultez la [vue d’ensemble des points de terminaison de service de réseau virtuel](concepts-data-access-and-security-vnet.md).

### <a name="private-ip"></a>IP privée
Private Link vous permet de vous connecter à votre serveur unique Azure Database pour MySQL via un point de terminaison privé. Azure Private Link intègre essentiellement les services Azure à votre Réseau virtuel privé. Vous pouvez accéder aux ressources PaaS à l’aide de l’adresse IP privée, comme toute autre ressource dans le réseau virtuel. Pour plus d’informations, consultez la [présentation de Private Link](concepts-data-access-security-private-link.md)

## <a name="access-management"></a>Gestion de l’accès

Quand vous créez un serveur Azure Database pour MySQL, vous fournissez les informations d’identification d’un utilisateur administrateur. Cet administrateur peut ensuite créer d’autres utilisateurs MySQL.


## <a name="threat-protection"></a>Protection contre les menaces

Vous pouvez activer le service [Advanced Threat Protection](concepts-data-access-and-security-threat-protection.md) qui détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des serveurs.

La fonctionnalité d’[enregistrement d’audit](concepts-audit-logs.md) vous permet de suivre l’activité dans vos bases de données. 


## <a name="next-steps"></a>Étapes suivantes
- Activer des règles de pare-feu pour les [adresses IP](concepts-firewall-rules.md) ou les [réseaux virtuels](concepts-data-access-and-security-vnet.md)
