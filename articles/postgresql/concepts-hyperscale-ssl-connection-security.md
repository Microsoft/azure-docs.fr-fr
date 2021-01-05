---
title: Transport Layer Security (TLS) – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Instructions et informations de configuration pour Azure Database pour PostgreSQL (Hyperscale (Citus)) et les applications associées afin d’utiliser correctement les connexions TLS.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 07/16/2020
ms.openlocfilehash: 659f86a107e4b08db4ec5195635ea32d2260d677
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87071460"
---
# <a name="configure-tls-in-azure-database-for-postgresql---hyperscale-citus"></a>Configurer TLS dans Azure Database pour PostgreSQL - Hyperscale (Citus)
Le nœud coordinateur Hyperscale (Citus) requiert que les applications clientes se connectent à l’aide du protocole TLS (Transport Layer Security). L’application du protocole TLS entre le serveur de base de données et les applications clientes permet de préserver la confidentialité des données en transit. Les paramètres de vérification supplémentaires décrits ci-dessous protègent également d’attaques de l’intercepteur.

## <a name="enforcing-tls-connections"></a>Appliquer les connexions TLS
Les applications utilisent une « chaîne de connexion » pour identifier la base de données de destination et les paramètres d’une connexion. Des clients différents nécessitent des réglages différents. Pour afficher la liste des chaînes de connexion utilisées par les clients courants, consultez la section **Chaînes de connexion** de votre groupe de serveurs dans le portail Azure.

Les paramètres TLS `ssl` et `sslmode` varient en fonction des capacités du connecteur, par exemple `ssl=true`, `sslmode=require` ou `sslmode=required`.

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Vérification que votre application ou votre infrastructure prend en charge les connexions TLS
Certaines infrastructures d’application n’activent pas TLS par défaut pour les connexions PostgreSQL. Toutefois, sans connexion sécurisée, une application ne peut pas se connecter à un nœud coordinateur Hyperscale (Citus). Consultez la documentation de votre application pour savoir comment activer les connexions TLS.

## <a name="applications-that-require-certificate-verification-for-tls-connectivity"></a>Applications nécessitant la vérification du certificat pour la connectivité TLS
Dans certains cas, les applications nécessitent un fichier de certificat local généré à partir du fichier de certificat (.cer) d’une autorité de certification (AC) approuvée pour se connecter en toute sécurité. Vous trouverez le certificat permettant la connexion à une base de données Azure pour PostgreSQL - Hyperscale (Citus) sur https://cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem. Télécharger le fichier du certificat et enregistrez-le à l’emplacement par défaut.

> [!NOTE]
>
> Pour vérifier l’authenticité du certificat, vous pouvez vérifier son empreinte digitale SHA-256 à l’aide de l’outil en ligne de commande OpenSSL :
>
> ```sh
> openssl x509 -in DigiCertGlobalRootCA.crt.pem -noout -sha256 -fingerprint
>
> # should output:
> # 43:48:A0:E9:44:4C:78:CB:26:5E:05:8D:5E:89:44:B4:D8:4F:96:62:BD:26:DB:25:7F:89:34:A4:43:C7:01:61
> ```

### <a name="connect-using-psql"></a>Connexion via psql
L’exemple suivant montre comment se connecter au nœud coordinateur Hyperscale (Citus) à l’aide de l’utilitaire de ligne de commande psql. Utilisez le paramètre de chaîne de connexion `sslmode=verify-full` pour appliquer la vérification du certificat TLS. Passez le chemin d’accès du fichier de certificat local au paramètre `sslrootcert`.

Voici un exemple de chaîne de connexion pqsl :
```
psql "sslmode=verify-full sslrootcert=DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=citus user=citus password=your_pass"
```
> [!TIP]
> Vérifiez que la valeur passée à `sslrootcert` correspond au chemin de fichier du certificat que vous avez enregistré.

## <a name="next-steps"></a>Étapes suivantes
Renforcez la sécurité avec les [règles de pare-feu dans Azure Database pour PostgreSQL - Hyperscale (Citus)](concepts-hyperscale-firewall-rules.md).
