---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 5cb345ef2d20f75066e90f9e6478be27f925b1b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80673363"
---
### <a name="retrieve-the-azure-storage-connection-string"></a>Récupérer la chaîne de connexion de Stockage Azure

Vous avez créé auparavant un compte de stockage Azure que l’application de fonction va utiliser. La chaîne de connexion pour ce compte est stockée de manière sécurisée dans les paramètres d’application au sein d’Azure. En téléchargeant le paramètre dans le fichier *local.settings.json*, vous pouvez utiliser cette connexion pour écrire dans une file d’attente de stockage du même compte au moment de l’exécution locale de la fonction. 

1. À partir de la racine du projet, exécutez la commande suivante, en remplaçant `<app_name>` par le nom de votre application de fonction créée dans le guide de démarrage rapide précédent. Cette commande remplace toutes les valeurs existantes dans le fichier.

    ```
    func azure functionapp fetch-app-settings <app_name>
    ```
    
1. Ouvrez *local.settings.json* et recherchez la valeur nommée `AzureWebJobsStorage`, qui est la chaîne de connexion du compte de stockage. Vous utilisez le nom `AzureWebJobsStorage` et la chaîne de connexion dans d’autres sections de cet article.

> [!IMPORTANT]
> Dans la mesure où *local.settings.json* contient des secrets téléchargés à partir d’Azure, excluez toujours ce fichier du contrôle de code source. Le fichier *.gitignore* créé avec un projet Functions local exclut le fichier par défaut.