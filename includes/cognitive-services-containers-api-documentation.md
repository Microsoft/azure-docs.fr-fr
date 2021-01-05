---
author: IEvangelist
ms.author: dapine
ms.date: 08/22/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 8b3d856d255968b4a6736db908ce3999cbd56193
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996412"
---
## <a name="validate-that-a-container-is-running"></a>Valider l’exécution d’un conteneur 

Il existe plusieurs façons de confirmer que le conteneur s’exécute. Recherchez l’*adresse IP* externe et le port exposé du conteneur en question, puis ouvrez le navigateur web de votre choix. Utilisez les différentes URL de requête ci-dessous pour vérifier que le conteneur est en cours d’exécution. Les exemples d’URL de requête listés ci-dessous sont `http://localhost:5000`, mais votre conteneur spécifique peut varier. N’oubliez pas que vous devez vous fier à l’adresse *IP externe* de votre conteneur et au port exposé.

| URL de la demande | Objectif |
|--|--|
| `http://localhost:5000/` | Le conteneur fournit une page d’accueil. |
| `http://localhost:5000/ready` | Cette demande, qui utilise GET, permet de vérifier que le conteneur est prêt à accepter une requête sur le modèle.  Cette requête peut être utilisée pour les [probes liveness et readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes. |
| `http://localhost:5000/status` | Cette demande, qui utilise également GET, permet de vérifier si la clé API servant à démarrer le conteneur est valide sans provoquer de requête de point de terminaison. Cette requête peut être utilisée pour les [probes liveness et readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes. |
| `http://localhost:5000/swagger` | Le conteneur fournit un ensemble complet de documentation pour les points de terminaison et une fonctionnalité **Essayer**. Avec cette fonctionnalité, vous pouvez entrer vos paramètres dans un formulaire HTML basé sur le web, et constituer la requête sans avoir à écrire du code. Une fois la requête retournée, un exemple de commande CURL est fourni pour illustrer les en-têtes HTTP, et le format du corps qui est nécessaire. |

![Page d’accueil du conteneur](./media/cognitive-services-containers-api-documentation/container-webpage.png)
