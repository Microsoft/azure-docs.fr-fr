---
title: Connectivité chiffrée via TLS/SSL dans Azure Database pour PostgreSQL - Serveur flexible
description: Instructions et informations sur la connexion à l’aide du protocole TLS/SSL dans Azure Database pour PostgreSQL - Serveur flexible.
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 0ba04ff14e62cd411515a765dc95ef7a3e72b709
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930143"
---
# <a name="encrypted-connectivity-using-transport-layer-security-in-azure-database-for-postgresql---flexible-server"></a>Connectivité chiffrée à l’aide du protocole TLS dans Azure Database pour PostgreSQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Azure Database pour PostgreSQL - Serveur flexible préfère connecter vos applications clientes au service PostgreSQL en utilisant le protocole TLS (Transport Layer Security), anciennement appelé SSL (Secure Sockets Layer). TLS est un protocole standard qui garantit la sécurité des connexions réseau entre votre serveur de base de données et vos applications clientes, ce qui vous permet de respecter les exigences de conformité.

Azure Database pour PostgreSQL - Serveur flexible prend uniquement en charge les connexions chiffrées à l’aide du protocole TLS 1.2+ et toutes les connexions entrantes qui utilisent les protocoles TLS 1.0 et TLS 1.1 sont refusées. Pour tous les serveurs flexibles, la mise en œuvre des connexions TLS est activée et il est impossible de désactiver le protocole TLS/SSL pour la connexion à un serveur flexible.

## <a name="applications-that-require-certificate-verification-for-tlsssl-connectivity"></a>Applications nécessitant la vérification du certificat pour la connectivité TSL/SSL
Dans certains cas, les applications nécessitent un fichier de certificat local généré à partir du fichier de certificat d’une autorité de certification (AC) approuvée pour se connecter en toute sécurité. Azure Database pour PostgreSQL - Serveur flexible utilise l’*autorité de certification racine globale DigiCert*. Téléchargez ce certificat nécessaire pour communiquer par le biais du protocole SSL auprès de l’[autorité de certification racine globale DigiCert](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) et enregistrez le fichier du certificat à l’emplacement qui vous convient. Par exemple, ce tutoriel utilise `c:\ssl`.


### <a name="connect-using-psql"></a>Connexion via psql
Si vous avez créé votre serveur flexible avec l’option *Accès privé (intégration au réseau virtuel)* , vous avez besoin de vous connecter à votre serveur à partir d’une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l’ajouter au réseau virtuel créé avec votre serveur flexible.

Si vous avez créé votre serveur flexible avec l’option *Accès public (adresses IP autorisées)* , vous pouvez ajouter votre adresse IP locale à la liste des règles de pare-feu sur votre serveur.

L’exemple suivant montre comment vous connecter à votre serveur à l’aide de l’interface de ligne de commande psql. Utilisez le paramètre de chaîne de connexion `sslmode=verify-full` pour appliquer la vérification du certificat TLS/SSL. Passez le chemin d’accès du fichier de certificat local au paramètre `sslrootcert`.

```bash
 psql "sslmode=verify-full sslrootcert=c:\ssl\DigiCertGlobalRootCA.crt.pem host=mydemoserver.postgres.database.azure.com dbname=postgres user=myadmin"
```
> [!Note]
> Vérifiez que la valeur passée à `sslrootcert` correspond au chemin de fichier du certificat que vous avez enregistré.

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Vérification que votre application ou votre infrastructure prend en charge les connexions TLS

Certaines infrastructures d’applications courantes qui utilisent PostgreSQL pour leurs services de base de données n’activent pas le protocole TLS par défaut lors de l’installation. Votre serveur PostgreSQL applique des connexions TLS, mais si l’application n’est pas configurée pour cela, cette dernière risque de ne pas pouvoir se connecter à votre serveur de base de données. Consultez la documentation de votre application pour savoir comment activer les connexions TLS.

## <a name="next-steps"></a>Étapes suivantes
- [Créer et gérer un réseau virtuel Azure Database pour PostgreSQL - Serveur flexible à l’aide de Azure CLI](./how-to-manage-virtual-network-cli.md).
- En savoir plus sur la [mise en réseau dans Azure Database pour PostgreSQL - Serveur flexible](./concepts-networking.md).
- En savoir plus sur les [règles de pare-feu pour Azure Database pour PostgreSQL - Serveur flexible](./concepts-networking.md#public-access-allowed-ip-addresses).
