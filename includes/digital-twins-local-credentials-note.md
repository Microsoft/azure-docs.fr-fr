---
author: baanders
description: Fichier include pour DefaultAzureCredential dans les exemples Azure Digital Twins - Remarque
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: 35996595158994308a78f4d1197dcdc8f00af618
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92494472"
---
>[!NOTE]
> Cet exemple utilise [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?preserve-view=true&view=azure-dotnet) (qui fait partie de la bibliothèque de `Azure.Identity`) pour authentifier les utilisateurs avec l’instance Azure Digital Twins lorsque vous l’exécutez sur votre ordinateur local. Avec ce type d’authentification, l’exemple recherche les informations d’identification Azure dans votre environnement local, comme une connexion à partir d’une instance [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) locale ou dans Visual Studio/Visual Studio Code.
>
> Pour plus d’informations sur l’utilisation de `DefaultAzureCredential` et d’autres options d’authentification, consultez [*Guide pratique : Écrire le code d’authentification d’une application*](../articles/digital-twins/how-to-authenticate-client.md).