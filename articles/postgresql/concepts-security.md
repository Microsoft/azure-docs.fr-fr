---
title: Sécurité dans Azure Database pour PostgreSQL – Serveur unique
description: Vue d’ensemble des fonctionnalités de sécurité dans Azure Database pour PostgreSQL – Serveur unique.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 11/22/2019
ms.openlocfilehash: be042a0ec076538cf0f0d155667acea6f1ae19cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710479"
---
# <a name="security-in-azure-database-for-postgresql---single-server"></a>Sécurité dans Azure Database pour PostgreSQL – Serveur unique

Il existe plusieurs couches de sécurité qui permettent de protéger les données de votre serveur Azure Database pour PostgreSQL. Cet article décrit ces différentes options de sécurité.

## <a name="information-protection-and-encryption"></a>Protection et chiffrement des informations

### <a name="in-transit"></a>En transit
Azure Database pour PostgreSQL sécurise vos données en chiffrant les données en transit à l’aide du protocole TLS (Transport Layer Security). Le chiffrement (SSL/TLS) est appliqué par défaut.

### <a name="at-rest"></a>Au repos
Le service Azure Database pour PostgreSQL utilise le module de chiffrement conforme à la norme FIPS 140-2 pour chiffrer le stockage des données au repos. Toutes les données, notamment les sauvegardes, sont chiffrées sur le disque, y compris les fichiers temporaires créés durant l’exécution des requêtes. Le service utilise le chiffrement AES 256 bits inclus dans le chiffrement de stockage Azure, et les clés sont gérées par le système. Le chiffrement de stockage est toujours activé et ne peut pas être désactivé.


## <a name="network-security"></a>Sécurité du réseau
Les connexions à un serveur Azure Database pour PostgreSQL sont d’abord routées via une passerelle régionale. La passerelle a une adresse IP accessible publiquement, tandis que les adresses IP du serveur sont protégées. Pour plus d’informations sur la passerelle, consultez l’article [Architecture de connectivité](concepts-connectivity-architecture.md).  

Tout serveur Azure Database pour PostgreSQL est créé avec un pare-feu qui bloque toutes les connexions externes. Ces connexions peuvent atteindre la passerelle, mais elles ne permettent pas l’accès au serveur. 

### <a name="ip-firewall-rules"></a>Règles de pare-feu IP
Les règles de pare-feu IP octroient l’accès aux serveurs en fonction de l’adresse IP d’origine de chaque requête. Pour plus d’informations, consultez la [vue d’ensemble des règles de pare-feu](concepts-firewall-rules.md).

### <a name="virtual-network-firewall-rules"></a>Règles de pare-feu de réseau virtuel
Les points de terminaison de service de réseau virtuel étendent votre connectivité de réseau virtuel via le réseau principal Azure. En utilisant des règles de réseau virtuel, vous pouvez faire en sorte que votre serveur Azure Database pour PostgreSQL autorise les connexions à partir de certains sous-réseaux d’un réseau virtuel. Pour plus d’informations, consultez la [vue d’ensemble des points de terminaison de service de réseau virtuel](concepts-data-access-and-security-vnet.md).

### <a name="private-ip"></a>IP privée
Private Link vous permet de vous connecter à votre serveur unique Azure Database pour PostgreSQL via un point de terminaison privé. Azure Private Link intègre essentiellement les services Azure à votre Réseau virtuel privé. Vous pouvez accéder aux ressources PaaS à l’aide de l’adresse IP privée, comme toute autre ressource dans le réseau virtuel. Pour plus d’informations, consultez la [présentation de Private Link](concepts-data-access-and-security-private-link.md)


## <a name="access-management"></a>Gestion de l’accès

Quand vous créez un serveur Azure Database pour PostgreSQL, vous fournissez les informations d’identification d’un rôle Administrateur. Ce rôle Administrateur permet de créer d’autres [rôles PostgreSQL](https://www.postgresql.org/docs/current/user-manag.html).

Vous pouvez aussi vous connecter au serveur en utilisant l’[authentification Azure Active Directory (AAD)](concepts-aad-authentication.md).


## <a name="threat-protection"></a>Protection contre les menaces

Vous pouvez activer le service [Advanced Threat Protection](concepts-data-access-and-security-threat-protection.md) qui détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des serveurs.

La fonctionnalité d’[enregistrement d’audit](concepts-audit.md) vous permet de suivre l’activité dans vos bases de données. 

## <a name="migrating-from-oracle"></a>Effectuer une migration depuis Oracle

Oracle prend en charge TDE (Transparent Data Encryption) pour chiffrer les données de table et d’espace disque logique. Dans Azure pour PostgreSQL, les données sont automatiquement chiffrées au niveau de différentes couches. Consultez la section « Au repos » de cette page et reportez-vous également à différents sujets relatifs à la sécurité, notamment les [clés gérées par le client](./concepts-data-encryption-postgresql.md) et [Chiffrement double d’infrastructure](./concepts-infrastructure-double-encryption.md). Vous pouvez également envisager d’utiliser l’extension [pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html), qui est prise en charge dans [Azure pour PostgreSQL](./concepts-extensions.md).

## <a name="next-steps"></a>Étapes suivantes
- Activer des règles de pare-feu pour les [adresses IP](concepts-firewall-rules.md) ou les [réseaux virtuels](concepts-data-access-and-security-vnet.md)
- Découvrir l’[authentification Azure Active Directory](concepts-aad-authentication.md) dans Azure Database pour PostgreSQL
