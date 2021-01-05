---
title: Préparer vos ressources techniques de conteneur Azure
description: Ressources et conseils techniques pour vous aider à configurer une offre de conteneur sur la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 11/30/2020
ms.openlocfilehash: 014bcd6fc519c267cdf17e9e98b850425c25ead6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459327"
---
# <a name="prepare-your-azure-container-technical-assets"></a>Préparer vos ressources techniques de conteneur Azure

Cet article fournit des ressources et recommandations techniques pour vous aider à créer une offre de conteneur sur la Place de marché Azure.

## <a name="before-you-begin"></a>Avant de commencer

Pour accéder à des guides de démarrage rapide, didacticiels et autres exemples, consultez la [Documentation sur Azure Container Instances](../container-instances/index.yml).

## <a name="fundamental-technical-knowledge"></a>Connaissances techniques fondamentales

Concevoir, créer et tester ces ressources demande du temps et des connaissances techniques sur la plateforme Azure et les technologies utilisées pour créer l’offre.

En plus du domaine de votre solution, votre équipe d'ingénierie doit connaître les technologies Microsoft suivantes :

- Connaissances de base des [services Azure](https://azure.microsoft.com/services/)
- Comment [concevoir et créer des applications Azure](https://azure.microsoft.com/solutions/architecture/)
- Expérience de travail avec les [machines virtuelles Azure](https://azure.microsoft.com/services/virtual-machines/), [Stockage Azure](https://azure.microsoft.com/services/?filter=storage) et [Mise en réseau Azure](https://azure.microsoft.com/services/?filter=networking)
- Expérience de travail avec [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/)
- Expérience de travail avec [JSON](https://www.json.org/).

## <a name="suggested-tools"></a>Outils suggérés

Choisissez l'un des environnements de script suivants, ou les deux, pour vous aider à gérer votre image conteneur :

- [Azure PowerShell](/powershell/azure/)
- [Azure CLI](/cli/azure/).

Nous vous recommandons d'ajouter les outils suivants à votre environnement de développement :

- [Azure Storage Explorer](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows)
- [Visual Studio Code](https://code.visualstudio.com/)
  - Extension : [Outils Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
  - Extension : [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
  - Extension : [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json).

Examinez les outils disponibles sur la page [Outils de développement Azure](https://azure.microsoft.com/). Si vous utilisez Visual Studio, passez en revue les outils disponibles sur [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="create-the-container-image"></a>Créer l’image conteneur

Vous ne pouvez pas déployer une image sur Azure Container Instances à partir d’un registre local.

- Si vous disposez déjà d’un conteneur fonctionnel dans votre registre local, créez un registre Azure, puis chargez votre image conteneur sur Azure Container Registry. Pour en savoir plus, consultez [Tutoriel : Générer et déployer des images conteneurs dans le cloud avec Azure Container Registry Tasks](../container-registry/container-registry-tutorial-quick-task.md).

- Si vous n’avez pas encore d’image conteneur et que vous devez conteneuriser votre application existante ou créer une application basée sur un conteneur, clonez le code source de l’application à partir de GitHub, créez une image conteneur à partir de la source de l’application, puis testez l’image dans un environnement Docker local. Pour en savoir plus, consultez [Tutoriel : Créer une image conteneur à déployer sur Azure Container Instances](../container-instances/container-instances-tutorial-prepare-app.md).

## <a name="next-steps"></a>Étapes suivantes

- [Créer votre offre de conteneur](create-azure-container-offer.md)