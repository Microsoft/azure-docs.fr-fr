---
title: Fichier include
description: Fichier include
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 08/24/2020
ms.openlocfilehash: aa8aea66c5481454f0d7d4d118934f5fbf34a911
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89147223"
---
L’**identité managée par défaut** est l’identité managée qui est attribuée par le système ou la première identité managée qui est attribuée par l’utilisateur.

Pendant une exécution, il y a deux applications d’une identité :

1. Le système utilise une identité pour configurer les montages de stockage, le registre de conteneurs et les magasins de données associés à l’utilisateur.

    * Dans ce cas, le système utilise l’identité managée par défaut.

1. L’utilisateur applique une identité pour accéder à des ressources à partir du code pour une exécution envoyée

    * Dans ce cas, fournissez la valeur *client_id* correspondant à l’identité managée à utiliser pour récupérer des informations d’identification.
    * Vous pouvez également récupérer l’ID client de l’identité attribuée par l’utilisateur au moyen de la variable d’environnement *DEFAULT_IDENTITY_CLIENT_ID*.

    Par exemple, si vous voulez récupérer un jeton pour un magasin de données avec l’identité managée par défaut :

    ```python
    client_id = os.environ.get('DEFAULT_IDENTITY_CLIENT_ID')
    credential = ManagedIdentityCredential(client_id=client_id)
    token = credential.get_token('https://storage.azure.com/')
    ```