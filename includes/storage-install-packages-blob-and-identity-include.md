---
title: Fichier Include
description: Fichier Include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 11/26/2019
ms.author: tamram
ms.custom: include
ms.openlocfilehash: de79ea50d12ab322d1e28d0ad650df30ecc0c222
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74806578"
---
## <a name="install-client-library-packages"></a>Installer des packages de bibliothèque de client

> [!NOTE]
> Les exemples illustrés ici utilisent la version 12 de la bibliothèque de client du stockage Azure. La bibliothèque de client version 12 fait partie du SDK Azure. Pour plus d’informations sur le SDK Azure, consultez le dépôt du SDK Azure sur [GitHub](https://github.com/Azure/azure-sdk).

Pour installer le package du stockage d’objets blob, exécutez la commande suivante dans la console du gestionnaire de package NuGet :

```powershell
Install-Package Azure.Storage.Blobs
```

Les exemples de cet article utilisent également la dernière version de la [bibliothèque de client Azure Identity pour .NET](https://www.nuget.org/packages/Azure.Identity/) pour s’authentifier avec des informations d’identification Azure AD. Pour installer le package, exécutez la commande suivante dans la console du gestionnaire de package NuGet :

```powershell
Install-Package Azure.Identity
```
