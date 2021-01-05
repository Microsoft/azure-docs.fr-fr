---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/26/2019
ms.author: glenga
ms.openlocfilehash: e10de8093bf152b75cc6f262a142ff07c3d5b0d3
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "72329603"
---
Vous avez déjà créé une application de fonction dans Azure ainsi que le compte de stockage nécessaire. La chaîne de connexion pour ce compte est stockée de manière sécurisée dans les paramètres d’application au sein d’Azure. Dans cet article, vous allez écrire des messages dans une file d’attente de stockage au sein du même compte. Pour vous connecter à votre compte de stockage lors de l’exécution de la fonction localement, vous devez télécharger les paramètres de l’application dans le fichier local.settings.json. 

À partir de la racine du projet, exécutez la commande Azure Functions Core Tools suivante pour télécharger les paramètres dans le fichier local.settings.json, en remplaçant `<APP_NAME>` par le nom de votre application de fonction décrite dans l’article précédent :

```bash
func azure functionapp fetch-app-settings <APP_NAME>
```

Il se peut que vous deviez vous connecter à votre compte Azure.

> [!IMPORTANT]  
> Cette commande remplace tous les paramètres existants par des valeurs de votre application de fonction dans Azure.  
>
> Comme il contient des secrets, le fichier local.settings.json n’est jamais publié et doit être exclu du contrôle de code source.  
> 

Vous avez besoin de la valeur `AzureWebJobsStorage`, qui est la chaîne de connexion de compte de stockage. Cette connexion vous permet de vérifier que la liaison de sortie fonctionne comme prévu.
