---
title: Exigences relatives au stockage Blob Microsoft Azure Data Box | Microsoft Docs
description: Familiarisez-vous avec les versions des API, des kits de développement logiciel (SDK) et des bibliothèques clientes prises en charge pour le stockage Blob Azure Data Box
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 10/05/2020
ms.author: alkohli
ms.openlocfilehash: ac5f2de383066d6ee399dac3b0ad8c365b2e72bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91744092"
---
# <a name="azure-data-box-blob-storage-requirements"></a>Exigences relatives au stockage Blob Azure Data Box

Cet article liste les versions des API Azure, des bibliothèques de client Azure et des outils pris en charge avec le stockage Blob Data Box. Le stockage Blob Data Box fournit des fonctionnalités de gestion des objets blob avec une sémantique Azure cohérente. Cet article récapitule également les différences connues entre le stockage Blob Azure Data Box et les services de stockage Azure.

Nous vous recommandons de lire attentivement les informations suivantes avant de vous connecter au stockage Blob Data Box, puis d'y revenir par la suite si nécessaire.


## <a name="storage-differences"></a>différences entre les stockages

|     Fonctionnalité                                             |     Stockage Azure                                     |     Stockage Blob Data Box |
|---------------------------------------------------------|-------------------------------------------------------|---------------------------|
|    Présentation du stockage de fichiers                                   |    Partages de fichiers SMB sur le cloud pris en charge              |    Non pris en charge      |
|    Service Encryption pour les données au repos                  |    Chiffrement AES 256 bits                             |    Chiffrement AES 256 bits |
|    Type de compte de stockage                                 |    Comptes de stockage à usage général et comptes de stockage d’objets blob Azure    |    Comptes de stockage à usage général v1 uniquement|
|    Nom de l’objet blob                                            |    1 024 caractères (2 048 octets)                     |    880 caractères (1 760 octets)|
|    Taille maximale des objets blob de blocs                              |    4,75 To (100 Mo X 50 000 blocs)                   |    4,75 To (100 Mo x 50 000 blocs) pour Azure Data Box v 3.0 et versions ultérieures.|
|    Taille maximale des objets blob de pages                               |    8 To                                               |    1 To                   |
|    Taille de page d’un objet blob de pages                                  |    512 octets                                          |    4 Ko                   |

## <a name="supported-api-versions"></a>Versions d'API prises en charge

Les versions suivantes des API du service Stockage Azure sont prises en charge avec le stockage Blob Data Box.

### <a name="azure-data-box-30-onwards"></a>Azure Data Box 3.0 et ultérieur

[!INCLUDE [data-box-rest-supported-api-versions](../../includes/data-box-rest-supported-api-versions.md)]

## <a name="supported-azure-client-libraries"></a>Bibliothèques clientes Azure prises en charge

Des bibliothèques clientes spécifiques, de même que des exigences spécifiques en matière de suffixe de point de terminaison, s'appliquent au stockage Blob Data Box. Les points de terminaison du stockage Blob Data Box n’ont pas de parité complète avec la dernière version de l’API REST du Stockage Blob Azure. Consultez la liste des [versions prises en charge pour Azure Data Box v 3.0 et ultérieur](#supported-api-versions). Par conséquent, pour les bibliothèques clientes de stockage, vous devez connaître la version compatible avec l’API REST.

### <a name="azure-data-box-30-onwards"></a>Azure Data Box 3.0 et ultérieur

Les versions suivantes de la bibliothèque de client Azure sont prises en charge pour le stockage Blob Data Box.

[!INCLUDE [data-box-rest-supported-azure-client-libraries](../../includes/data-box-rest-supported-azure-client-libraries.md)]

### <a name="install-php-client-via-composer---current"></a>Installer le client PHP via Composer - actuel

Pour installer via Composer : (prenez blob pour exemple).
1. Créez un fichier nommé composer.json à la racine du projet avec le code suivant :

    ```
    {
    "require": {
    "Microsoft/azure-storage-blob":"1.2.0"
    }
    ```

2. Téléchargez `composer.phar` à la racine du projet.

3. Exécutez : php composer.phar install.

### <a name="endpoint-declaration"></a>Déclaration de point de terminaison

Dans le SDK du stockage Blob Data Box, le suffixe du point de terminaison (`<device serial number>.microsoftdatabox.com`) identifie le domaine Data Box. Pour plus d’informations sur le point de terminaison du service Blob, accédez à [Se connecter via le stockage Blob Data Box](data-box-deploy-copy-data-via-rest.md).
 
## <a name="examples"></a>Exemples

### <a name="net"></a>.NET

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans le fichier `app.config` :

```
<add key="StorageConnectionString"
value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;
EndpointSuffix=<<serial no. of the device>.microsoftdatabox.com  />
```

### <a name="java"></a>Java

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key;" +
    "EndpointSuffix=<serial no. of the device>.microsoftdatabox.com ";
```

### <a name="nodejs"></a>Node.js

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans l'instance de déclaration :

```
var blobSvc = azure.createBlobService('myaccount', 'mykey',
'myaccount.blob. <serial no. of the device>.microsoftdatabox.com ');
```

### <a name="c"></a>C++

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;
AccountName=your_storage_account;
AccountKey=your_storage_account_key;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com "));
```

### <a name="php"></a>PHP

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
$connectionString = 'BlobEndpoint=http://<storage account name>.blob.<serial no. of the device>.microsoftdatabox.com /;
AccountName=<storage account name>;AccountKey=<storage account key>'
```

### <a name="python"></a>Python

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans l'instance de déclaration :

```
block_blob_service = BlockBlobService(account_name='myaccount',
account_key='mykey',
endpoint_suffix=’<serial no. of the device>.microsoftdatabox.com’)
```

### <a name="ruby"></a>Ruby

Pour le stockage Blob Data Box, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
set
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;
AccountName=myaccount;
AccountKey=mykey;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com
```

## <a name="next-steps"></a>Étapes suivantes

* [Déployer votre Azure Data Box](data-box-deploy-ordered.md)
