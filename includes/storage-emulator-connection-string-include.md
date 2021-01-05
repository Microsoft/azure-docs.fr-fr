---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/17/2020
ms.author: tamram
ms.openlocfilehash: 85e7cb86217340e77a6f597a357c3de1f91fb8d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87070566"
---
Azurite prend en charge uniquement un compte fixe et une clé d’authentification connue pour l’authentification par clé partagée. Ce compte et cette clé sont les seules informations d’identification par clé partagée autorisées pour une utilisation avec Azurite. Il s'agit de :

```
Account name: devstoreaccount1
Account key: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

> [!NOTE]
> La clé d’authentification prise en charge par Azurite est destinée uniquement au test de la fonctionnalité de votre code d’authentification du client. Elle n'offre aucune fonction de sécurité. Vous ne pouvez pas utiliser votre compte et votre clé de stockage de production avec Azurite. Vous ne devez pas utiliser le compte de développement avec des données de production.
> 
> Azurite prend uniquement en charge la connexion via le protocole HTTP. Toutefois, HTTPS est le protocole recommandé pour l’accès aux ressources dans un compte de Stockage Azure de production.
> 

#### <a name="connect-to-the-emulator-account-using-a-shortcut"></a>Se connecter au compte de l’émulateur à l’aide d’un raccourci
La méthode la plus simple pour vous connecter à Azurite à partir de votre application consiste à configurer une chaîne de connexion dans le fichier config de votre application, qui référence le raccourci `UseDevelopmentStorage=true`. Voici un exemple de chaîne de connexion à Azurite dans un fichier *app.config* : 

```xml
<appSettings>
  <add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
</appSettings>
```

#### <a name="connect-to-the-emulator-account-using-the-well-known-account-name-and-key"></a>Se connecter au compte de l’émulateur à l’aide d’un nom de compte connu et d’une clé
Pour créer une chaîne de connexion qui référence le nom et la clé du compte de l’émulateur, vous devez définir les points de terminaison associés aux services que vous souhaitez utiliser à partir de l’émulateur dans la chaîne de connexion. Cela est nécessaire pour que la chaîne de connexion puisse référencer les points de terminaison de l’émulateur, qui sont différents de ceux associés à un compte de stockage de production. Par exemple, la valeur de votre chaîne de connexion ressemblera à ceci :

```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
```

Cette valeur est identique au raccourci présenté plus haut, `UseDevelopmentStorage=true`.
