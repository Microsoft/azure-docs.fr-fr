---
author: baanders
description: Fichier include pour Azure Digital Twins – Configurer la dernière extension IoT
ms.service: digital-twins
ms.topic: include
ms.date: 7/31/2020
ms.author: baanders
ms.openlocfilehash: 7e43ab6d419d6a08973864838c66aef0941d74f5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90606235"
---
Tout d’abord, exécutez cette commande pour voir la liste de toutes les extensions que vous avez déjà installées.

```azurecli
az extension list
```

La sortie est un tableau de toutes les extensions dont vous disposez actuellement. Recherchez le champ `"name"` pour chaque entrée de liste afin de voir les noms des extensions.

Utilisez la sortie pour déterminer lesquelles des commandes suivantes doivent être exécutées pour la configuration de l’extension (vous pouvez en exécuter plusieurs).
* Si la liste contient `azure-iot` : vous disposez déjà de l’extension. Exécutez cette commande pour vérifier que vous disposez de la dernière mise à jour et qu’il n’y a plus de mises à jour disponibles :

   ```azurecli
   az extension update --name azure-iot
   ```

* Si la liste ne contient **pas** `azure-iot` : Vous devez installer l’extension. Utilisez la commande suivante :

    ```azurecli
    az extension add --name azure-iot
    ```

* Si la liste contient `azure-iot-cli-ext` : il s’agit de la version héritée de l’extension. Une seule version de l’extension doit être installée à la fois. Vous devez donc désinstaller l’extension héritée. Utilisez la commande suivante :

   ```azurecli
   az extension remove --name azure-cli-iot-ext
   ```