---
title: Récupérer le nom d’utilisateur et le mot de passe pour se connecter au contrôleur de données arc
description: Récupérer le nom d’utilisateur et le mot de passe pour se connecter au contrôleur de données arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 8cabb48e3620f1a17d1bb9b87e1646ce2793143b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91761700"
---
# <a name="retrieve-the-user-name-and-password-to-connect-to-the-arc-data-controller"></a>Récupérer le nom d’utilisateur et le mot de passe pour se connecter au contrôleur de données arc

Vous pouvez être dans une situation où vous devez récupérer le nom d’utilisateur et le mot de passe pour le contrôleur de données. Voici les commandes dont vous avez besoin pour l’exécution. 

```console
azdata login
```

Si vous êtes l’administrateur Kubernetes du cluster. Par conséquent, vous avez le droit d’exécuter des commandes pour récupérer à partir du secret Kubernetes stocke les informations qui sont conservées par Azure Arc.

> [!NOTE]
>  Si vous avez utilisé un autre nom pour l’espace de noms dans lequel le contrôleur de données a été créé, veillez à modifier le paramètre `-n arc` dans les commandes ci-dessous afin d’utiliser le nom de l’espace de noms dans lequel vous avez créé le contrôleur de données.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="linux"></a>Linux

Exécutez la commande suivante pour récupérer le nom d’utilisateur :

```console
kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['username']}" | base64 -d
```

Exécutez la commande suivante pour récupérer le mot de passe :

```console
kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['password']}" | base64 -d
```

## <a name="powershell"></a>PowerShell

Exécutez la commande suivante pour récupérer le nom d’utilisateur :

```console
[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String((kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['username']}")))
```

Exécutez la commande suivante pour récupérer le mot de passe :

```console
[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String((kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['password']}")))
```

## <a name="next-steps"></a>Étapes suivantes

Essayer d’autres [scénarios](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory-domain-services/scenarios.md)
