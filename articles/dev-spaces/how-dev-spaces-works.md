---
title: Fonctionnement d’Azure Dev Spaces
services: azure-dev-spaces
ms.date: 06/02/2020
ms.topic: conceptual
description: Décrit les processus sur lesquels repose Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: 8de2c27ce03c871e60b6437656ad630fc8de8408
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963701"
---
# <a name="how-azure-dev-spaces-works"></a>Fonctionnement d’Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Le développement d’une application Kubernetes peut s'avérer difficile. Vous avez besoin des fichiers de configuration Docker et Kubernetes. Vous devez trouver comment tester votre application localement et interagir avec d'autres services dépendants. Vous devrez éventuellement gérer le développement et le test sur plusieurs services à la fois et avec une équipe de développeurs.

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe. Cet article décrit ce que Azure Dev Spaces peut faire et son fonctionnement.

## <a name="rapidly-iterate-and-debug-your-kubernetes-application"></a>Itérer et déboguer rapidement votre application Kubernetes

Azure Dev Spaces réduit l’effort de développement, de test et d’itération de votre application Kubernetes dans le contexte de votre cluster AKS. Cette réduction des efforts permet aux développeurs de se concentrer sur la logique métier de leurs applications plutôt que sur la configuration de leurs services pour qu’ils s’exécutent dans Kubernetes.

### <a name="bridge-to-kubernetes"></a>Bridge to Kubernetes

Avec Bridge to Kubernetes, vous pouvez connecter votre ordinateur de développement à votre cluster Kubernetes et ainsi exécuter et déboguer du code sur votre ordinateur de développement comme s’il s’exécutait sur le cluster. Bridge to Kubernetes redirige le trafic de votre cluster connecté en exécutant un pod sur votre cluster, qui joue le rôle d’agent distant pour rediriger le trafic entre votre ordinateur de développement et le cluster. Cette redirection du trafic permet au code sur votre ordinateur de développement et aux services en cours d’exécution dans votre cluster de communiquer comme s’ils se trouvaient dans le même cluster. Pour plus d’informations sur la connexion de votre ordinateur de développement à un cluster Kubernetes, consultez [Fonctionnement de Bridge to Kubernetes][how-it-works-bridge-to-kubernetes].

### <a name="run-your-code-in-aks"></a>Exécuter votre code dans AKS

En plus de rediriger le trafic entre votre ordinateur de développement et votre cluster AKS, Azure Dev Spaces vous permet de configurer et d’exécuter rapidement votre code directement dans AKS. Avec Visual Studio, Visual Studio Code ou l’interface de ligne de commande Azure Dev Spaces, Azure Dev Spaces charge votre code dans le cluster, puis le génère et l’exécute. Azure Dev Spaces peut également synchroniser intelligemment les modifications du code et redémarrer votre service pour refléter les modifications si nécessaire. Lors de l’exécution de votre code, les journaux de génération et les suivis HTTP sont transmis vers votre client, ce qui vous permet de surveiller la progression et de diagnostiquer les problèmes. Vous pouvez également utiliser Azure Dev Spaces pour attacher le débogueur dans Visual Studio et Visual Studio Code aux services Java, Node.js et .NET Core. Pour plus d’informations, consultez [Fonctionnement de la préparation d’un projet pour Azure Dev Spaces][how-it-works-prep], [Comment fonctionne l’exécution de votre code avec Azure Dev Spaces][how-it-works-up] et [Fonctionnement du débogage à distance de votre code avec Azure Dev Spaces][how-it-works-remote-debugging].

## <a name="team-development"></a>Développement en équipe

Azure Dev Spaces permet aux équipes de travailler de manière productive sur leur application sur le même cluster AKS sans interruption.

### <a name="intelligent-routing-between-dev-spaces"></a>Routage intelligent entre espaces de développement

Avec Azure Dev Spaces, une équipe peut partager un cluster AKS unique exécutant une application cloud native et créer des espaces de développement isolés dans lesquels l’équipe peut développer, tester et déboguer sans interférer avec les autres espaces de développement. Une version de référence de l’application s’exécute dans un espace de développement racine. Les membres de l’équipe créent ensuite des espaces de développement enfants indépendants basés sur l’espace racine pour le développement, le test et le débogage des modifications apportées à l’application. Grâce aux fonctionnalités de routage de Dev Spaces, les espaces de développement enfants peuvent acheminer les requêtes entre les services s’exécutant dans l’espace de développement enfant et dans l’espace de développement parent. Ce routage permet aux développeurs d’exécuter leur propre version d’un service tout en réutilisant les services dépendants de l’espace parent. Chaque espace enfant possède sa propre URL unique, qui peut être partagée et accessible par d’autres personnes à des fins de collaboration. Pour plus d’informations sur le fonctionnement du routage dans Azure Dev Spaces, consultez [Fonctionnement du routage avec Azure Dev Spaces][how-it-works-routing].

### <a name="live-testing-an-open-pull-request"></a>Test direct d’une demande de tirage (pull request) ouverte

Vous pouvez également utiliser des actions GitHub avec Azure Dev Spaces pour tester les modifications apportées à votre application dans une demande de tirage (pull request) directement dans votre cluster avant la fusion. Azure Dev Spaces peut déployer automatiquement une version de révision de l’application sur votre cluster, ce qui permet à l’auteur et à d’autres membres de l’équipe d’examiner les modifications dans le contexte de l’application entière. À l’aide des fonctionnalités de routage d’Azure Dev Spaces, cette version de révision de l’application est également déployée sur votre cluster sans impact sur les autres espaces de développement. Toutes ces fonctionnalités vous permettent d’approuver et de fusionner en toute confiance des demandes de tirage (pull requests). Pour voir un exemple d’actions GitHub et d’Azure Dev Spaces , consultez [Actions GitHub & Azure Kubernetes Service][pr-flow].

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à connecter votre ordinateur de développement local à votre cluster AKS, consultez [Connecter votre ordinateur de développement à un cluster AKS][connect].


[connect]: https://code.visualstudio.com/docs/containers/bridge-to-kubernetes
[how-it-works-bridge-to-kubernetes]: /visualstudio/containers/overview-bridge-to-kubernetes
[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[how-it-works-up]: how-dev-spaces-works-up.md
[pr-flow]: how-to/github-actions.md
[routing]: #team-development