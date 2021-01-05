---
title: Créer des actions et des ressources personnalisées
description: Ce tutoriel explique comment créer des actions et des ressources personnalisées dans Azure Resource Manager. Il montre également comment les workflows personnalisés interagissent avec les modèles Azure Resource Manager, Azure CLI, Azure Policy et les journaux d’activité Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: 45f18727b53b802ba746da41b47fe955543ed9d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75648546"
---
# <a name="create-custom-actions-and-resources-in-azure"></a>Créer des actions et des ressources personnalisées dans Azure

Un fournisseur personnalisé implique qu’un contrat soit passé entre Azure et un point de terminaison. Avec les fournisseurs personnalisés, vous pouvez modifier les workflows dans Azure en ajoutant de nouvelles API à Azure Resource Manager. Avec ces API personnalisées, Resource Manager peut utiliser de nouvelles fonctionnalités de déploiement et de gestion.

Ce tutoriel fait appel à un exemple simple pour vous montrer comment ajouter de nouvelles actions et ressources à Azure, et comment les intégrer.

## <a name="set-up-azure-functions-for-azure-custom-providers"></a>Configurer Azure Functions pour les fournisseurs personnalisés Azure

La première partie de ce tutoriel montre comment configurer une application de fonction Azure pour qu’elle fonctionne avec les fournisseurs personnalisés :

- [Configurer Azure Functions pour les fournisseurs personnalisés Azure](./tutorial-custom-providers-function-setup.md)

Les fournisseurs personnalisés peuvent utiliser n’importe quelle URL publique.

## <a name="author-a-restful-endpoint-for-custom-providers"></a>Créer un point de terminaison RESTful pour des fournisseurs personnalisés

La deuxième partie de ce tutoriel explique comment créer un point de terminaison RESTful pour les fournisseurs personnalisés :

- [Créer un point de terminaison RESTful pour des fournisseurs personnalisés](./tutorial-custom-providers-function-authoring.md)

## <a name="create-and-use-a-custom-provider"></a>Créer et utiliser un fournisseur personnalisé

La troisième partie de ce tutoriel explique comment créer un fournisseur personnalisé et utiliser ses actions et ses ressources personnalisées :

- [Créer et utiliser un fournisseur personnalisé](./tutorial-custom-providers-create.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris ce qu’est un fournisseur personnalisé et comment en créer un. Pour passer au tutoriel suivant, consultez [Tutoriel : Configurer Azure Functions pour les fournisseurs personnalisés Azure](./tutorial-custom-providers-function-setup.md).

Si vous cherchez des références ou un guide de démarrage rapide, voici quelques liens utiles :

- [Démarrage rapide : Créer un fournisseur de ressources personnalisé Azure et déployer des ressources personnalisées](./create-custom-provider.md)
- [Procédure : ajouter des actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Procédure : ajouter des ressources personnalisées à l’API REST Azure](./custom-providers-resources-endpoint-how-to.md)
