---
title: SSL/TLS – Azure Database pour PostgreSQL – Serveur unique
description: Instructions et informations sur la façon de configurer la connectivité TSL pour Azure Database pour PostgreSQL – Serveur unique.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: conceptual
ms.date: 07/08/2020
ms.openlocfilehash: c98ee8f747975d4237c2906be2060eddbc7b9990
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000944"
---
# <a name="configure-tls-connectivity-in-azure-database-for-postgresql---single-server"></a>Configurer la connectivité TLS dans Azure Database pour PostgreSQL – Serveur unique

Azure Database pour PostgreSQL préfère connecter vos applications clientes au service PostgreSQL en utilisant le protocole TLS (Transport Layer Security), anciennement appelé SSL (Secure Sockets Layer). L’application de connexions TLS entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application

Par défaut, le service de base de données PostgreSQL est configuré de façon à exiger une connexion TLS. Vous pouvez désactiver l’exigence d’utilisation du protocole TLS si votre application cliente ne prend pas en charge la connectivité TLS.

>[!NOTE]
> Sur la base des commentaires des clients, nous avons étendu la dépréciation du certificat racine pour notre autorité de certification racine Baltimore existante jusqu’au 15 février 2021 (15/2/2021).

> [!IMPORTANT] 
> Le certificat racine SSL est configuré pour expirer à partir du 15 février 2021 (15/2/2021). Mettez à jour votre application pour utiliser le [nouveau certificat](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem). Pour plus d’informations, consultez [Mises à jour de certificat planifiées](concepts-certificate-rotation.md)


## <a name="enforcing-tls-connections"></a>Appliquer les connexions TLS

Pour tous les serveurs Azure Database pour PostgreSQL approvisionnés via le Portail Azure et Azure CLI, l’application de connexions TLS est activée par défaut. 

De même, les chaînes de connexion prédéfinies dans les paramètres « Chaînes de connexion » du serveur sur le Portail Azure incluent les paramètres requis par les langages courants pour se connecter au serveur de base de données via TLS. Le paramètre TLS varie en fonction du connecteur, par exemple « ssl=true », « sslmode=require » ou « sslmode=require » et d’autres variations.

## <a name="configure-enforcement-of-tls"></a>Configurer l’application du protocole TLS

Vous pouvez désactiver l’application de la connectivité protocole TLS. Microsoft Azure recommande de toujours activer le paramètre **Appliquer une connexion SSL** pour renforcer la sécurité.

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Accédez à votre serveur de base de données Azure pour PostgreSQL et cliquez sur **Sécurité de la connexion**. Utilisez le bouton bascule pour activer ou désactiver le paramètre **Appliquer une connexion SSL**. Ensuite, cliquez sur **Enregistrer**.

:::image type="content" source="./media/concepts-ssl-connection-security/1-disable-ssl.png" alt-text="Sécurité de connexion – Désactiver l’application du protocole TLS/SSL":::

Vous pouvez vérifier le paramètre en consultant la page **Vue d’ensemble** pour voir si l’indicateur d **’état de d’application SSL** est affiché.

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

Vous pouvez activer ou désactiver le paramètre **ssl-mise en œuvre** en utilisant respectivement les valeurs `Enabled` ou `Disabled` dans Azure CLI.

```azurecli
az postgres server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Enabled
```

## <a name="ensure-your-application-or-framework-supports-tls-connections"></a>Vérification que votre application ou votre infrastructure prend en charge les connexions TLS

Certaines infrastructures d’applications courantes qui utilisent PostgreSQL pour leurs services de base de données n’activent pas le protocole TLS par défaut lors de l’installation. Si votre serveur PostgreSQL applique des connexions TLS, mais que l’application n’est pas configurée pour cela, cette dernière risque de ne pas pouvoir se connecter à votre serveur de base de données. Consultez la documentation de votre application pour savoir comment activer les connexions TLS.

## <a name="applications-that-require-certificate-verification-for-tls-connectivity"></a>Applications nécessitant la vérification du certificat pour la connectivité TLS

Dans certains cas, les applications nécessitent un fichier de certificat local généré à partir du fichier de certificat d’une autorité de certification (AC) approuvée pour se connecter en toute sécurité. Vous trouverez le certificat permettant la connexion à un serveur Azure Database pour PostgreSQL ici : https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem. Télécharger le fichier du certificat et enregistrez-le à l’emplacement par défaut. 

Consultez les liens suivants afin d’obtenir des certificats pour les serveurs de clouds souverains : [Azure Government](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem), [Azure Chine](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) et [Azure Allemagne](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt).

### <a name="connect-using-psql"></a>Connexion via psql

L’exemple suivant montre comment vous connecter à votre serveur PostgreSQL à l’aide de l’utilitaire de ligne de commande psql. Utilisez le paramètre de chaîne de connexion `sslmode=verify-full` pour appliquer la vérification du certificat TLS/SSL. Passez le chemin d’accès du fichier de certificat local au paramètre `sslrootcert`.

La commande suivante est un exemple de chaîne de connexion pqsl :

```shell
psql "sslmode=verify-full sslrootcert=BaltimoreCyberTrustRoot.crt host=mydemoserver.postgres.database.azure.com dbname=postgres user=myusern@mydemoserver"
```

> [!TIP]
> Vérifiez que la valeur passée à `sslrootcert` correspond au chemin de fichier du certificat que vous avez enregistré.

## <a name="tls-enforcement-in-azure-database-for-postgresql-single-server"></a>Application de TLS dans Azure Database pour PostgreSQL Serveur unique

Azure Database pour PostgreSQL - Serveur unique prend en charge le chiffrement pour les clients qui se connectent à votre serveur de base de données à l’aide du protocole TLS (Transport Layer Security). TLS est un protocole standard qui garantit la sécurité des connexions réseau entre votre serveur de base de données et vos applications clientes, ce qui vous permet de respecter les exigences de conformité.

### <a name="tls-settings"></a>Paramètres TLS

Azure Database pour PostgreSQL Serveur unique offre la possibilité d’exiger une version TLS pour les connexions clientes. Pour exiger une version TLS, utilisez le paramètre d’option **Version TLS minimale**. Les valeurs suivantes sont autorisées pour ce paramètre d’option :

|  Paramètre TLS minimal             | Version TLS du client prise en charge                |
|:---------------------------------|-------------------------------------:|
| TLSEnforcementDisabled (par défaut) | Aucun protocole TLS requis                      |
| TLS1_0                           | TLS 1.0, TLS 1.1, TLS 1.2 et versions ultérieures |
| TLS1_1                           | TLS 1.1, TLS 1.2 et versions ultérieures          |
| TLS1_2                           | TLS version 1.2 et ultérieures           |


Par exemple, la définition de ce paramètre TLS minimal sur TLS 1.0 signifie que votre serveur autorisera les connexions en provenance des clients utilisant TLS 1.0, 1.1 et 1.2+. Si vous choisissez la valeur 1.2, en revanche, cela signifie que vous autorisez uniquement les connexions en provenance des clients utilisant TLS 1.2+ et que toutes les connexions à l’aide des protocoles TLS 1.0 et TLS 1.1 sont rejetées.

> [!Note] 
> Par défaut, Azure Database pour PostgreSQL n’applique aucune version TLS minimale (le paramètre `TLSEnforcementDisabled`).
>
> Après avoir appliqué une version TLS minimale, vous ne pouvez plus la désactiver.

Pour découvrir comment définir le paramètre TLS pour Azure Database pour PostgreSQL - Serveur unique, consultez le [Guide pratique pour configurer le paramètre TLS](howto-tls-configurations.md).

## <a name="cipher-support-by-azure-database-for-postgresql-single-server"></a>Prise en charge du chiffrement par Azure Database pour PostgreSQL – Serveur unique

Dans le cadre de la communication SSL/TLS, les suites de chiffrement sont validées et seules les suites de chiffrement prises en charge sont autorisées à communiquer avec le serveur de la base de données. La validation de la suite de chiffrement est contrôlée dans la [couche passerelle](concepts-connectivity-architecture.md#connectivity-architecture) et non pas explicitement sur le nœud lui-même. Si les suites de chiffrement ne correspondent pas à l’une des suites listées ci-dessous, les connexions client entrantes seront rejetées.

### <a name="cipher-suite-supported"></a>Suite de chiffrement prise en charge

*   TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
*   TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
*   TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
*   TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les différentes options de connectivité d’application de la rubrique [Bibliothèques de connexions d’Azure Database pour PostgreSQL](concepts-connection-libraries.md).

- Découvrez comment [configurer le protocole TLS](howto-tls-configurations.md).