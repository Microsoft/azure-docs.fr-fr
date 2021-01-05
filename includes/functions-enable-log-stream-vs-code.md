---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 06/25/2019
ms.author: glenga
ms.openlocfilehash: 437b4ab62cc8c4903af88ca2f9632e89b953c798
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010500"
---
Pour activer les journaux de diffusion en continu pour votre Function App dans Azure :

1. Sélectionnez F1 pour ouvrir la palette de commandes, puis recherchez et exécutez la commande **Azure Functions : Start Streaming Logs** (Démarrer les journaux de diffusion en continu).

1. Sélectionnez votre Function App dans Azure, puis sélectionnez **Oui** pour activer la journalisation d’application de votre Function App.

1. Déclenchez vos fonctions dans Azure. Notez que les données de journal sont affichées dans la fenêtre de résultats de Visual Studio Code.

1. Lorsque vous avez terminé, n’oubliez pas d’exécuter la commande **Azure Functions : Stop Streaming Logs** (Arrêter les journaux de diffusion en continu) pour désactiver la journalisation de la Function App.