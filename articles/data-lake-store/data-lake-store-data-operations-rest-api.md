---
title: 'API REST : opérations de gestion du système de fichiers sur Azure Data Lake Storage Gen1 | Microsoft Docs'
description: Utiliser les API REST de WebHDFS pour effectuer des opérations de gestion du système de fichiers sur Azure Data Lake Storage Gen1
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 51e0fb2ffa7b573ecfeda163d9ad99597ff735a2
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109202"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-rest-api"></a>Opérations de gestion du système de fichiers sur Azure Data Lake Storage Gen1 à l’aide d’une API REST
> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET](data-lake-store-data-operations-net-sdk.md)
> * [Kit SDK Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

Dans cet article, vous allez découvrir comment utiliser les API REST de WebHDFS et de Data Lake Storage Gen1 pour effectuer des opérations de gestion du système de fichiers sur Azure Data Lake Storage Gen1. Pour obtenir des instructions sur l’exécution des opérations de gestion des comptes sur Data Lake Storage Gen1 à l’aide de l’API REST, consultez la section relative aux [opérations de gestion des comptes sur Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-get-started-rest-api.md).

## <a name="prerequisites"></a>Prérequis
* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Compte Azure Data Lake Storage Gen1**. Suivez les instructions de [Prise en main d’Azure Data Lake Storage Gen1 avec le portail Azure](data-lake-store-get-started-portal.md).

* **[cURL](https://curl.haxx.se/)** . Cet article utilise cURL pour montrer comment effectuer des appels d’API REST sur un compte Data Lake Storage Gen1.

## <a name="how-do-i-authenticate-using-azure-active-directory"></a>Comment s’authentifier à l’aide d’Azure Active Directory ?
Vous avez le choix entre deux méthodes pour vous authentifier à l’aide d’Azure Active Directory :

* Pour en savoir plus sur l’authentification des utilisateurs accédant à votre application (interactive), consultez la section relative à l’[authentification de l’utilisateur avec Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-rest-api.md).
* Pour en savoir plus sur l’authentification de service à service pour votre application (non interactive), consultez la section relative à l’[authentification de service à service avec Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-service-to-service-authenticate-rest-api.md).


## <a name="create-folders"></a>Créez les dossiers.
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Make_a_Directory).

Utilisez la commande cURL suivante. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/?op=MKDIRS'
```

Dans la commande précédente, remplacez \<`REDACTED`\> par le jeton d’autorisation que vous avez récupéré. Cette commande crée un répertoire appelé **mytempdir** sous le dossier racine de votre compte Azure Data Lake Storage Gen1.

Si l’opération s’est déroulée correctement, vous devez voir une réponse similaire à l’extrait de code suivant :

```output
{"boolean":true}
```

## <a name="list-folders"></a>Répertorier les dossiers
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#List_a_Directory).

Utilisez la commande cURL suivante. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -X GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS'
```

Dans la commande précédente, remplacez \<`REDACTED`\> par le jeton d’autorisation que vous avez récupéré.

Si l’opération s’est déroulée correctement, vous devez voir une réponse similaire à l’extrait de code suivant :

```output
{
"FileStatuses": {
    "FileStatus": [{
        "length": 0,
        "pathSuffix": "mytempdir",
        "type": "DIRECTORY",
        "blockSize": 268435456,
        "accessTime": 1458324719512,
        "modificationTime": 1458324719512,
        "replication": 0,
        "permission": "777",
        "owner": "<GUID>",
        "group": "<GUID>"
    }]
}
}
```

## <a name="upload-data"></a>Télécharger des données
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Create_and_Write_to_a_File).

Utilisez la commande cURL suivante. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -X PUT -L -T 'C:\temp\list.txt' -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE'
```

Dans la syntaxe précédente, le paramètre **-T** correspond à l’emplacement du fichier que vous chargez.

Le résultat ressemble à l’extrait de code suivant :
   
```output
HTTP/1.1 307 Temporary Redirect
...
Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/list.txt?op=CREATE&write=true
...
Content-Length: 0

HTTP/1.1 100 Continue

HTTP/1.1 201 Created
...
```

## <a name="read-data"></a>Lire les données
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Open_and_Read_a_File).

La lecture des données d’un compte Azure Data Lake Storage Gen1 s’effectue en deux étapes.

* D’abord, envoyez une demande GET sur le point de terminaison `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN`. Cet appel retourne un emplacement où envoyer la demande GET suivante.
* Ensuite, envoyez la deuxième demande GET sur le point de terminaison `https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN&read=true`. Cet appel affiche le contenu du fichier.

Toutefois, comme les paramètres d’entrée sont les mêmes pour la première et la deuxième étape, vous pouvez utiliser le paramètre `-L` pour envoyer la première demande. L’option `-L` permet essentiellement de combiner les deux demandes en une seule et d’indiquer à cURL de réexécuter la demande sur le nouvel emplacement. Le résultat final de tous les appels de la demande s’affiche, comme illustrée dans l’extrait de code suivant. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -L GET -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=OPEN'
```

Le résultat doit ressembler à l’extrait de code suivant :

```output
HTTP/1.1 307 Temporary Redirect
...
Location: https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/somerandomfile.txt?op=OPEN&read=true
...

HTTP/1.1 200 OK
...

Hello, Data Lake Store user!
```

## <a name="rename-a-file"></a>Renommer un fichier
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Rename_a_FileDirectory).

Utilisez la commande cURL suivante pour renommer un fichier. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -d "" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile.txt?op=RENAME&destination=/mytempdir/myinputfile1.txt'
```

Le résultat doit ressembler à l’extrait de code suivant :

```output
HTTP/1.1 200 OK
...

{"boolean":true}
```

## <a name="delete-a-file"></a>Supprimer un fichier
Cette opération est basée sur l’appel de l’API REST de WebHDFS défini [ici](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Delete_a_FileDirectory).

Utilisez la commande cURL suivante pour supprimer un fichier. Remplacez **\<yourstorename>** par le nom de votre compte Data Lake Storage Gen1.

```console
curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" 'https://<yourstorename>.azuredatalakestore.net/webhdfs/v1/mytempdir/myinputfile1.txt?op=DELETE'
```

Un résultat similaire à ce qui suit s’affiche normalement :

```output
HTTP/1.1 200 OK
...

{"boolean":true}
```

## <a name="next-steps"></a>Étapes suivantes
* [Opérations de gestion des comptes sur Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-get-started-rest-api.md).

## <a name="see-also"></a>Voir aussi
* [Référence relative à l’API REST d’Azure Data Lake Storage Gen1](/rest/api/datalakestore/)
* [Ouvrir des applications Big Data open source compatibles avec Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)