---
title: Exporter les certificats de l’émulateur Azure Cosmos DB
description: Découvrez comment exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec des applications Java, Python et Node.js. Les certificats doivent être exportés et utilisés pour les langages et les environnements d’exécution qui n’utilisent pas le Magasin de certificats Windows.
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/17/2020
author: deborahc
ms.author: dech
ms.custom: devx-track-python, devx-track-java, contperf-fy21q1
ms.openlocfilehash: 1c111ca23ce7cf749dc4a7d5ad2ad39756236ab1
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028204"
---
# <a name="export-the-azure-cosmos-db-emulator-certificates-for-use-with-java-python-and-nodejs-apps"></a>Exporter les certificats de l’émulateur Azure Cosmos DB pour une utilisation avec des applications Java, Python et Node.js
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement. L’émulateur Azure Cosmos prend uniquement en charge la communication sécurisée via des connexions TLS.

Les certificats de l’émulateur local Azure Cosmos DB sont générés lors de la première exécution de l’émulateur. Il existe deux certificats. L’une d’entre elles est utilisée pour se connecter à l’émulateur local et l’autre pour gérer le chiffrement par défaut des données de l’émulateur au sein de l’émulateur. Le certificat à exporter est le certificat de connexion présentant le nom convivial DocumentDBEmulatorCertificate.

Lorsque vous utilisez l’émulateur pour développer des applications dans différents langages, tels que Java, Python ou Node.js, vous devez exporter le certificat de l’émulateur et l’importer dans le magasin de certificats requis.

Le langage .NET et le runtime utilisent le magasin de certificats Windows pour se connecter en toute sécurité à l’émulateur local Azure Cosmos DB lorsque l’application est exécutée sur un hôte avec système d’exploitation Windows. D’autres langages ont leur propre méthode de gestion et de l’utilisation des certificats. Par exemple, Java utilise son propre [magasin de certificats](https://docs.oracle.com/cd/E19830-01/819-4712/ablqw/index.html), Python utilise les [wrappers de sockets](https://docs.python.org/2/library/ssl.html), et Node.js utilise [tlsSocket](https://nodejs.org/api/tls.html#tls_tls_connect_options_callback).

Cet article explique comment exporter les certificats TLS/SSL en vue de les utiliser dans différents langages et environnements d’exécution qui ne s’intègrent pas au Magasin de certificats Windows. Pour en savoir plus sur l’émulateur, consultez l’article [Utilisation de l’émulateur Azure Cosmos DB pour le développement et le test](./local-emulator.md).

## <a name="export-the-azure-cosmos-db-tlsssl-certificate"></a><a id="export-emulator-certificate"></a>Exporter le certificat TLS/SSL Azure Cosmos DB

Vous devez exporter le certificat de l’émulateur pour utiliser correctement le point de terminaison de l’émulateur à partir de langages et d’environnements d’exécution qui ne s’intègrent pas au Magasin de certificats Windows. Vous pouvez exporter le certificat à l’aide du gestionnaire de Windows Certificate Manager. Utilisez les instructions pas à pas suivantes pour exporter le certificat « DocumentDBEmulatorCertificate » en tant que fichier X.509 (.cer) encodé en BASE 64 :

1. Démarrez le gestionnaire de certificats Windows en exécutant certlm.msc, puis accédez au dossier Personnel > Certificats et ouvrez le certificat portant le nom convivial **DocumentDbEmulatorCertificate**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-1.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 1)":::

1. Cliquez sur **Détails**, puis sur **OK**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-2.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 2)":::

1. Cliquez sur **Copier dans un fichier...** .

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-3.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 3)":::

1. Cliquez sur **Suivant**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-4.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 4)":::

1. Cliquez sur l’option de **refus d’exportation de la clé privée**, puis sur **Suivant**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-5.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 5)":::

1. Cliquez sur **X.509 encodé en base 64 (.cer)** , puis sur **Suivant**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-6.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 6)":::

1. Attribuez un nom au certificat. Ici, cliquez sur **documentdbemulatorcert**, puis sur **Suivant**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-7.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 7)":::

1. Cliquez sur **Terminer**.

    :::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-export-step-8.png" alt-text="Émulateur local Azure Cosmos DB – Exporter (étape 8)":::

## <a name="use-the-certificate-with-java-apps"></a>Utiliser le certificat avec des applications Java

Lors de l’exécution d’applications Java ou MongoDB qui utilisent un client Java, il est plus simple d’installer le certificat dans le magasin de certificats Java par défaut que de transmettre les indicateurs `-Djavax.net.ssl.trustStore=<keystore> -Djavax.net.ssl.trustStorePassword="<password>"`. Par exemple, l’application de démonstration Java incluse (`https://localhost:8081/_explorer/index.html`) dépend du magasin de certificats par défaut.

Suivez les instructions figurant dans [Ajout d'un certificat au magasin de certificats Java](/azure/developer/java/sdk/java-sdk-add-certificate-ca-store) pour importer le certificat X.509 dans le magasin de certificats Java par défaut. Gardez à l’esprit que vous travaillerez dans le répertoire *%JAVA_HOME%* durant l’exécution de keytool. Une fois le certificat importé dans le magasin de certificats, les clients de l’API SQL et Azure Cosmos DB pour MongoDB peuvent se connecter à l’émulateur Azure Cosmos DB.

Vous pouvez également exécuter le script bash suivant pour importer le certificat :

```bash
#!/bin/bash

# If emulator was started with /AllowNetworkAccess, replace the below with the actual IP address of it:
EMULATOR_HOST=localhost
EMULATOR_PORT=8081
EMULATOR_CERT_PATH=/tmp/cosmos_emulator.cert
openssl s_client -connect ${EMULATOR_HOST}:${EMULATOR_PORT} </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > $EMULATOR_CERT_PATH
# delete the cert if already exists
sudo $JAVA_HOME/bin/keytool -cacerts -delete -alias cosmos_emulator
# import the cert
sudo $JAVA_HOME/bin/keytool -cacerts -importcert -alias cosmos_emulator -file $EMULATOR_CERT_PATH
```

Une fois que le certificat TLS/SSL « Azure Cosmos DBEmulatorCertificate » est installé, votre application doit être en mesure de se connecter et d’utiliser l’émulateur local Azure Cosmos DB. Si vous rencontrez des problèmes, vous pouvez suivre l’article [Débogage des connexions SSL/TLS](https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html). Dans la plupart des cas, le certificat n’est pas installé dans le magasin *%JAVA_HOME%/jre/lib/security/cacerts*. Par exemple, si vous disposez de plusieurs versions de Java installées, votre application peut utiliser un magasin cacaerts différent de celui mis à jour.

## <a name="use-the-certificate-with-python-apps"></a>Utiliser le certificat avec des applications Python

Lors de la connexion à l’émulateur à partir des applications Python, la vérification TLS est désactivée. Par défaut, le [SDK Python (version 2.0.0 ou ultérieure)](sql-api-sdk-python.md) pour l’API SQL ne tente pas d’utiliser le certificat TLS/SSL lors de la connexion à l’émulateur local. Si vous souhaitez utiliser la validation TLS, vous pouvez suivre les exemples de la documentation des [wrappers de socket Python](https://docs.python.org/2/library/ssl.html).

## <a name="how-to-use-the-certificate-in-nodejs"></a>Comment utiliser le certificat en Node.js

Lors de la connexion à l’émulateur à partir des SDK Node.js, la vérification TLS est désactivée. Par défaut, le [SDK Node.js (version 1.10.1 ou ultérieure)](sql-api-sdk-node.md) pour l’API SQL ne tente pas d’utiliser le certificat TLS/SSL lors de la connexion à l’émulateur local. Si vous souhaitez utiliser la validation TLS, vous pouvez suivre les exemples de la [documentation Node.js](https://nodejs.org/api/tls.html#tls_tls_connect_options_callback).

## <a name="rotate-emulator-certificates"></a>Faire pivoter les certificats de l’émulateur

Vous pouvez forcer la régénération des certificats de l’émulateur en sélectionnant **Réinitialiser les données** à partir de l’émulateur Azure Cosmos DB s’exécutant dans la barre d’état Windows. Notez que cette action efface également toutes les données stockées localement par l’émulateur.

:::image type="content" source="./media/local-emulator-export-ssl-certificates/database-local-emulator-reset-data.png" alt-text="Émulateur local Azure Cosmos DB – Réinitialiser les données":::

Si vous avez installé les certificats dans le Magasin de certificats Java ou si vous les avez utilisés ailleurs, vous devez les réimporter à l’aide des certificats actuels. Votre application ne peut pas se connecter à l’émulateur local tant que vous n’avez pas mis à jour les certificats.

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des paramètres de ligne de commande et des commandes PowerShell pour contrôler l’émulateur](emulator-command-line-parameters.md)
* [Problèmes de débogage avec l’émulateur](troubleshoot-local-emulator.md)
