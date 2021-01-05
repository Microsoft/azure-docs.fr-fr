---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/26/2020
ms.author: larryfr
ms.openlocfilehash: 6147dff71ff8c93c8c46b5921b37db459ea73e4a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027953"
---
> [!TIP]
> Après vous être connecté, vous voyez une liste des abonnements associés à votre compte Azure. Les informations d’abonnement avec `isDefault: true` correspond à l’abonnement actuellement activé pour les commandes Azure CLI. Cet abonnement doit être le même que celui qui contient votre espace de travail Azure Machine Learning. Vous pouvez trouver l’ID d’abonnement à partir du [portail Azure](https://portal.azure.com) en visitant la page Vue d’ensemble de votre espace de travail. Vous pouvez également utiliser le kit de développement logiciel (SDK) pour récupérer l’ID d’abonnement de l’objet d’espace de travail. Par exemple : `Workspace.from_config().subscription_id`.
> 
> Pour sélectionner un autre abonnement, utilisez la commande `az account set -s <subscription name or ID>` et spécifiez le nom ou l’ID de l’abonnement auquel vous voulez passer. Pour plus d’informations sur la sélection d’abonnements, consultez [Utiliser plusieurs abonnements Azure](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest).