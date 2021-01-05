---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/20/2020
ms.author: glenga
ms.openlocfilehash: 574756aa9d9bac4aa42febf6a4fca4c62014db3f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010458"
---
1. Dans Visual Studio Code, sélectionnez F1 pour ouvrir la palette de commandes. Dans la palette de commandes, recherchez et sélectionnez **Azure Functions: Deploy to function app**.

1. Si vous n’êtes pas connecté, vous êtes invité à le faire avec **Se connecter à Azure**. Une fois la connexion établie à partir du navigateur, revenez à Visual Studio Code. Si vous avez plusieurs abonnements, **sélectionnez un abonnement** qui contient votre application de fonction.

1. Sélectionnez votre application de fonction existante dans Azure. Dans la fenêtre d’avertissement sur le remplacement de tous les fichiers dans l’application de fonction, sélectionnez **Déployer** pour accepter l’avertissement et continuer.

Le projet est régénéré, repackagé et chargé vers Azure. Le projet existant est remplacé par le nouveau package, et l’application de fonction redémarre.