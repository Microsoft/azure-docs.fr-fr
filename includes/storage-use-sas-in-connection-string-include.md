---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 2f27c50b1d016265c20102521a137bcbb0646115
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177281"
---
Si vous disposez d’une URL de signature d’accès partagé (SAP) qui vous permet d’accéder aux ressources d’un compte de stockage, vous pouvez utiliser la SAP dans une chaîne de connexion. Étant donné que la SAP contient les informations requises pour authentifier la requête, une chaîne de connexion avec une SAP fournit le protocole, le point de terminaison de service et les informations d’identification nécessaires pour accéder à la ressource.

Pour créer une chaîne de connexion incluant une signature d’accès partagé, spécifiez la chaîne au format suivant :

```
BlobEndpoint=myBlobEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
FileEndpoint=myFileEndpoint;
SharedAccessSignature=sasToken
```

Chaque point de terminaison de service est facultatif, mais la chaîne de connexion doit en contenir au moins un.

> [!NOTE]
> Il est recommandé d’utiliser le protocole HTTPS avec une SAS.
>
> Si vous spécifiez une SAS dans une chaîne de connexion dans un fichier de configuration, vous devrez encoder les caractères spéciaux dans l’URL.
>
>

### <a name="service-sas-example"></a>Exemple de SAP de service
Voici un exemple de chaîne de connexion incluant la SAS d’un service pour Blob Storage :

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&sr=b&si=tutorial-policy-635959936145100803&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

Et voici un exemple de la même chaîne de connexion avec encodage de caractères spéciaux :

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&amp;sr=b&amp;si=tutorial-policy-635959936145100803&amp;sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

### <a name="account-sas-example"></a>Exemple de SAP de compte
Voici un exemple de chaîne de connexion incluant la SAS d’un compte pour Blob Storage et File Storage. Notez que les points de terminaison des deux services sont spécifiés :

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&spr=https&st=2016-04-12T03%3A24%3A31Z&se=2016-04-13T03%3A29%3A31Z&srt=s&ss=bf&sp=rwl
```

Et voici un exemple de la même chaîne de connexion avec encodage de l’URL :

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&amp;sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&amp;spr=https&amp;st=2016-04-12T03%3A24%3A31Z&amp;se=2016-04-13T03%3A29%3A31Z&amp;srt=s&amp;ss=bf&amp;sp=rwl
```

