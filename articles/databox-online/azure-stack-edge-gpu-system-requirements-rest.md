---
title: Exigences en matière de stockage d’objets BLOB Microsoft Azure Stack Edge | Microsoft Docs
description: En savoir plus sur les versions prises en charge pour les API, les kits de développement logiciel et les bibliothèques clientes pour le stockage d’objets BLOB Azure Stack Edge
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: alkohli
ms.openlocfilehash: f99a3110880626b3a809e6bab5edc02398094547
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426218"
---
# <a name="azure-stack-edge-blob-storage-requirements"></a>Exigences en matière de stockage d’objets BLOB Azure Stack Edge

Cet article répertorie les versions des API Azure, les bibliothèques clientes Azure et les outils pris en charge avec le stockage d’objets BLOB Edge Azure Stack. Le stockage d’objets BLOB Azure Stack Edge fournit des fonctionnalités de gestion d’objets BLOB dont la sémantique est cohérente avec Azure. Cet article résume également les différences connues en matière de stockage d’objets BLOB Azure Stack Edge à partir des services de stockage Azure.

Nous vous recommandons de consulter attentivement les informations avant de vous connecter au stockage d’objets BLOB Edge Azure Stack, puis de vous y référer si nécessaire.

## <a name="storage-differences"></a>différences entre les stockages

|     Fonctionnalité                                             |     Stockage Azure                                     |     Stockage d’objets BLOB Azure Stack Edge |
|---------------------------------------------------------|-------------------------------------------------------|---------------------------|
|    Présentation du stockage de fichiers                                   |    Partages de fichiers SMB sur le cloud pris en charge              |    Non pris en charge      |
|    Type de compte de stockage                                 |    Comptes de stockage à usage général et comptes de stockage d’objets blob Azure    |    Comptes de stockage à usage général v1 uniquement|
|    Nom de l’objet blob                                            |    1 024 caractères (2 048 octets)                     |    880 caractères (1 760 octets)|
|    Taille maximale des objets blob de blocs                              |    4,75 To (100 Mo X 50 000 blocs)                   |    4,75 To (100 Mo x 50 000 blocs) pour Azure Stack Edge|
|    Taille maximale des objets blob de pages                               |    8 To                                               |    1 To                   |
|    Taille de page d’un objet blob de pages                                  |    512 octets                                          |    4 Ko                   |

## <a name="supported-api-versions"></a>Versions d'API prises en charge

Les versions suivantes des API de service de stockage Azure sont prises en charge avec le stockage d’objets BLOB Edge Azure Stack.

### <a name="azure-stack-edge-2113772170-onwards"></a>Azure Stack Edge 2.1.1377.2170 et versions ultérieures

[!INCLUDE [data-box-rest-supported-api-versions](../../includes/data-box-rest-supported-api-versions.md)]

## <a name="supported-azure-client-libraries"></a>Bibliothèques clientes Azure prises en charge

Pour le stockage d’objets BLOB Azure Stack Edge, il existe des bibliothèques clientes spécifiques et des exigences spécifiques en matière de suffixe de point de terminaison. Les points de terminaison du stockage d'objets BLOB Azure Stack Edge ne présentent pas de parité complète avec la dernière version de l'API REST Stockage Blob Azure. Consultez les [versions d'API prises en charge pour Azure Stack Edge](#supported-api-versions). Par conséquent, pour les bibliothèques clientes de stockage, vous devez connaître la version compatible avec l’API REST.

### <a name="azure-stack-edge-2113772170-onwards"></a>Azure Stack Edge 2.1.1377.2170 et versions ultérieures

Les versions de la bibliothèque cliente Azure suivantes sont prises en charge pour le stockage d’objets BLOB Edge Azure Stack.

[!INCLUDE [data-box-rest-supported-azure-client-libraries](../../includes/data-box-rest-supported-azure-client-libraries.md)]

### <a name="install-the-php-client-via-composer---current"></a>Installer le client PHP via Composer - Actuel

Pour installer le client PHP via Composer :

1. Créez un fichier nommé composer.json à la racine du projet avec le code suivant (l'exemple utilise le service Azure Storage Blob).

    ```
    {
    "require": {
    "Microsoft/azure-storage-blob":"1.2.0"
    }
    ```

2. Téléchargez `composer.phar` à la racine du projet.

3. Exécutez : php composer.phar install.


## <a name="endpoint-declaration"></a>Déclaration de point de terminaison

Dans le kit de développement logiciel (SDK) de stockage d’objets BLOB Azure Stack Edge, le suffixe de point de terminaison- `<device serial number>.microsoftdatabox.com` identifie le domaine Azure Stack Edge. Pour plus d’informations sur le point de terminaison du service BLOB, accédez à [transférer des données par le biais de comptes de stockage des GPU Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-add-storage-accounts.md).


## <a name="examples"></a>Exemples

### <a name="net"></a>.NET

Pour le stockage d’objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans le fichier `app.config`  :

```
<add key="StorageConnectionString"
value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;
EndpointSuffix=<<serial no. of the device>.microsoftdatabox.com  />
```

### <a name="java"></a>Java

Pour le stockage d’objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans le programme d’installation de la chaîne de connexion :

```
public static final String storageConnectionString =
    "DefaultEndpointsProtocol=http;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key;" +
    "EndpointSuffix=<serial no. of the device>.microsoftdatabox.com ";
```

### <a name="nodejs"></a>Node.js

Pour le stockage d’objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans l’instance de déclaration :

```
var blobSvc = azure.createBlobService('myaccount', 'mykey',
'myaccount.blob. <serial no. of the device>.microsoftdatabox.com ');
```

### <a name="c"></a>C++

Pour le stockage d'objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;
AccountName=your_storage_account;
AccountKey=your_storage_account_key;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com "));
```

### <a name="php"></a>PHP

Pour le stockage d'objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
$connectionString = 'BlobEndpoint=http://<storage account name>.blob.<serial no. of the device>.microsoftdatabox.com /;
AccountName=<storage account name>;AccountKey=<storage account key>'
```

### <a name="python"></a>Python

Pour le stockage d’objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans l’instance de déclaration :

```
block_blob_service = BlockBlobService(account_name='myaccount',
account_key='mykey',
endpoint_suffix=’<serial no. of the device>.microsoftdatabox.com’)
```

### <a name="ruby"></a>Ruby

Pour le stockage d'objets BLOB Azure Stack Edge, le suffixe du point de terminaison est spécifié dans la configuration de la chaîne de connexion :

```
set
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;
AccountName=myaccount;
AccountKey=mykey;
EndpointSuffix=<serial no. of the device>.microsoftdatabox.com
```

## <a name="next-steps"></a>Étapes suivantes

* [Préparer le déploiement de Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-deploy-prep.md)
