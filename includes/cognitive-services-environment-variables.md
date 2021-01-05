---
author: aahill
ms.service: cognitive-services
ms.topic: include
ms.date: 06/24/2019
ms.author: aahi
ms.openlocfilehash: 95bd83575809f6ecda716ff751b47b7bb499cae3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85073310"
---
## <a name="configure-an-environment-variable-for-authentication"></a>Configurer une variable d’environnement pour l’authentification

Les applications ont besoin d’authentifier l’accès à Cognitive Services qu’elles utilisent. Pour l’authentification, nous vous recommandons de créer une variable d’environnement destinée à stocker les clés pour vos ressources Azure. 

Après avoir obtenu votre clé, écrivez-la dans une variable d’environnement sur la machine locale exécutant l’application. Pour définir la variable d’environnement, ouvrez une fenêtre de console et suivez les instructions pour votre système d’exploitation. Remplacez `your-key` par l’une des clés de votre ressource.

#### <a name="windows"></a>[Windows](#tab/windows)

```console
setx COGNITIVE_SERVICE_KEY "your-key"
```

Après avoir ajouté la variable d’environnement, vous devrez peut-être redémarrer tous les programmes en cours d’exécution qui devront la lire, y compris la fenêtre de console. Par exemple, si vous utilisez Visual Studio comme éditeur, redémarrez Visual Studio avant d’exécuter l’exemple.

#### <a name="linux"></a>[Linux](#tab/linux)

```bash
export COGNITIVE_SERVICE_KEY=your-key
```

Après avoir ajouté la variable d’environnement, exécutez `source ~/.bashrc` depuis la fenêtre de console pour appliquer les changements.

#### <a name="macos"></a>[macOS](#tab/unix)

Modifiez votre profil bash_profile, et ajoutez la variable d’environnement :

```bash
export COGNITIVE_SERVICE_KEY=your-key
```

Après avoir ajouté la variable d’environnement, exécutez `source ~/.bash_profile` depuis la fenêtre de console pour appliquer les changements.

***
