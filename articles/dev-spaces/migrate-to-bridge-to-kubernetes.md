---
title: Migration vers la solution Bridge to Kubernetes
services: azure-dev-spaces
ms.date: 10/21/2020
ms.topic: conceptual
description: Décrit le processus de migration d’Azure Dev Spaces vers Bridge to Kubernetes
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Bridge to Kubernetes
ms.openlocfilehash: d48814df30c17f9b51d8642efa0960a26bbd24f4
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888519"
---
# <a name="migrating-to-bridge-to-kubernetes"></a>Migration vers la solution Bridge to Kubernetes

> [!IMPORTANT]
> Azure Dev Spaces sera mis hors service le 31 octobre 2023. Les clients doivent passer à l’utilisation de Bridge to Kubernetes, un outil de développement client.
>
> L’objectif d’Azure Dev Spaces était d’amener les utilisateurs à développer sur Kubernetes. Un inconvénient considérable dans l’approche d’Azure Dev Spaces a été de compliquer la tâche aux utilisateurs pour comprendre les configurations de Kubernetes et de Docker, ainsi que des concepts de déploiement Kubernetes. Au fil du temps, il est également devenu évident que l’approche d’Azure Dev Spaces n’a pas réduit efficacement la vitesse de développement de la boucle interne sur Kubernetes. Bridge to Kubernetes réduit efficacement la vitesse de développement de la boucle interne et évite toute charge inutile pour les utilisateurs.
>
> La mission principale reste inchangée : développer des expériences optimales pour développer, tester et déboguer du code de microservice dans le contexte d’une application plus large.

La solution Bridge to Kubernetes fournit une alternative plus légère à de nombreux scénarios de développement qui fonctionnent avec Azure Dev Spaces. Il s’agit d’une expérience exclusivement côté client utilisant des extensions dans [Visual Studio][vs] et [Visual Studio Code][vsc].  

La solution Bridge to Kubernetes facilite votre expérience de développement en permettant à une application Kubernetes établie d’inclure un service s’exécutant sur votre station de travail de développement locale. À la différence du service Dev Spaces, la solution Bridge to Kubernetes réduit les complexités de la boucle interne en évitant la nécessité de créer des configurations Docker et Kubernetes, ce qui permet aux développeurs de se concentrer uniquement sur la logique métier du code de leur microservice.

La solution Bridge to Kubernetes vous permet d’effectuer une itération sur le code qui s’exécute sur votre ordinateur de développement, tout en consommant des dépendances et une configuration existante de votre environnement Kubernetes. En revanche, le service Azure Dev Spaces déploie votre microservice dans l’environnement Kubernetes avant que vous puissiez déboguer à distance votre service et effectuer une itération sur votre code.

Cet article fournit une comparaison entre le service Azure Dev Spaces et la solution Bridge to Kubernetes, ainsi que des instructions pour la migration d’Azure Dev Spaces vers Bridge to Kubernetes.

## <a name="development-approaches"></a>Approches de développement

![Approches de développement](media/migrate-to-btk/development-approaches.svg)

Le service Azure Dev Spaces a aidé les développeurs de Kubernetes à travailler avec du code s’exécutant directement dans leur cluster AKS, évitant ainsi la nécessité de disposer d’un environnement local qui ne ressemble pas à l’environnement déployé. Cette approche a amélioré certains aspects du développement, mais a également introduit une condition préalable d’apprentissage et de maintien de concepts supplémentaires tels que Docker, Kubernetes et Helm avant de commencer à utiliser le service Azure Dev Spaces.

La solution Bridge to Kubernetes permet aux développeurs de travailler directement sur leurs ordinateurs de développement tout en interagissant avec le reste de leur cluster. Cette approche tire parti de la connaissance et de la vitesse d’exécution du code directement sur leurs ordinateurs de développement, tout en partageant les dépendances et l’environnement fournis par leur cluster. Cette approche tire également parti de la fidélité et de la mise à l’échelle résultant de l’exécution dans Kubernetes.

## <a name="feature-comparison"></a>Comparaison des fonctionnalités

Si le service Azure Dev Spaces et la solution Bridge to Kubernetes offrent des fonctionnalités similaires, ils diffèrent également dans plusieurs domaines :

| Condition requise  | Azure Dev Spaces  | Bridge to Kubernetes  |
|---------------|-------------------|--------------------------------|
| Azure Kubernetes Service | Dans 15 régions Azure | Toute région de service AKS    |
| **Sécurité** |
| Accès de sécurité nécessaire sur votre cluster  | Contributeur de cluster AKS  | RBAC Kubernetes – Mise à jour du déploiement   |
| Accès de sécurité nécessaire sur votre ordinateur de développement  | N/A  | Administrateur local/sudo   |
| **Usage** |
| Indépendant des artefacts Kubernetes et Docker  | Non  | Oui   |
| Restauration automatique des modifications, après débogage  | Non  | Oui   |
| **Outils clients pris en charge** |
| Fonctionne avec Visual Studio 2019  | Oui  | Oui   |
| Fonctionne avec Visual Studio Code  | Oui  | Oui   |
| Fonctionne avec une interface CLI  | Oui  | Non   |
| **Compatibilité du système d'exploitation** |
| Fonctionne sur Windows 10  | Oui  | Oui  |
| Fonctionne sur Linux  | Oui  | Oui  |
| Fonctionne sur macOS  | Oui  | Oui  |
| **Capabilities** |
| Isolation du développeur ou développement en équipe  | Oui  | Oui  |
| Remplacer de manière sélective les variables d’environnement  | Non  | Oui  |
| Création de graphique Dockerfile et Helm  | Oui  | Non  |
| Déploiement persistant du code sur Kubernetes  | Oui  | Non  |
| Débogage à distance dans un pod Kubernetes  | Oui  | Non  |
| Débogage local, connecté à Kubernetes  | Non  | Oui  |
| Débogage de plusieurs services en même temps, sur la même station de travail  | Oui  | Oui  |

## <a name="kubernetes-inner-loop-development"></a>Développement de la boucle interne Kubernetes

La plus grande différence entre le service Azure Dev Spaces et la solution Bridge to Kubernetes réside dans l’emplacement d’exécution de votre code. Le service Azure Dev Spaces permet de développer et déboguer votre code de microservice, mais exige que vous exécutiez ce code dans votre cluster. La solution Bridge to Kubernetes vous permet de développer et déboguer votre code de microservice directement sur votre ordinateur de développement, tout en restant dans le contexte de l’application plus volumineuse s’exécutant dans Kubernetes. La solution Bridge to Kubernetes étend le périmètre du cluster Kubernetes, et permet aux processus locaux d’hériter de la configuration de Kubernetes.

![Développement de boucle interne](media/migrate-to-btk/btk-graphic-non-isolated.gif)

Lors de l’utilisation de la solution Bridge to Kubernetes, une connexion réseau entre votre ordinateur de développement et votre cluster est établie. Pour la durée de vie de cette connexion, un proxy est ajouté à votre cluster à la place de votre déploiement Kubernetes, qui redirige les requêtes adressées au service vers votre ordinateur de développement. Lorsque vous vous déconnectez, le déploiement de l’application revient à utiliser la version d’origine du déploiement s’exécutant sur le cluster. Cette approche diffère de la façon dont fonctionne le service Azure Dev Spaces, où le code est synchronisé avec le cluster, généré, puis exécuté. Azure Dev Spaces ne restaure pas non plus votre code.

La solution Bridge to Kubernetes offre la flexibilité nécessaire pour travailler avec les applications s’exécutant dans Kubernetes, quelle que soit leur méthode de déploiement. Si vous utilisez CI/CD pour générer et exécuter votre application, que vous vous serviez d’outils existants ou de scripts personnalisés, vous pouvez toujours utiliser la solution Bridge to Kubernetes pour développer et déboguer votre code.

> [!TIP]
> L’ [extension Microsoft Kubernetes][kubernetes-extension] vous permet de développer rapidement des manifestes Kubernetes avec IntelliSense, et vous aide à générer des graphiques Helm.  

### <a name="transition-to-bridge-to-kubernetes-from-azure-dev-spaces"></a>Transition vers Bridge to Kubernetes à partir d’Azure Dev Spaces

1. Si vous utilisez Visual Studio, mettez à jour votre environnement de développement intégré (IDE) Visual Studio vers la version 16.7 ou une version ultérieure, et installez l’extension Bridge to Kubernetes à partir de [Visual Studio Marketplace][vs-marketplace]. Si vous utilisez Visual Studio Code, installez l’extension [Bridge to Kubernetes][vsc-marketplace].
1. Désactivez le contrôleur Azure Dev Spaces à l’aide du portail Azure ou de l’[interface de ligne de commande Azure Dev Spaces][azds-delete].
1. Utilisez [Azure Cloud Shell](https://shell.azure.com). Ou sur Mac, Linux ou Windows avec bash installé, ouvrez une invite d’interpréteur de commandes bash. Assurez-vous que les outils suivants sont disponibles dans votre environnement de ligne de commande : Azure CLI, Docker, kubectl, curl, tar et gunzip.
1. Créez un registre de conteneurs ou utilisez-en un existant. Vous pouvez créer un registre de conteneurs dans Azure à l’aide d’[Azure Container Registry](https://azure.microsoft.com/services/container-registry/) ou en utilisant [Docker Hub](https://hub.docker.com/). Lorsque vous utilisez Azure Cloud Shell, seul Azure Container Registry est disponible pour héberger les images Docker.
1. Exécutez le script de migration pour convertir les ressources Azure Dev Spaces en ressources Bridge to Kubernetes. Le script génère une nouvelle image compatible avec Bridge to Kubernetes, la charge dans le registre désigné, puis utilise [Helm](https://helm.sh) pour mettre à jour le cluster avec l’image. Vous devez indiquer le groupe de ressources, le nom du cluster AKS et un registre de conteneurs. Il existe d’autres options de ligne de commande, comme illustré ici :

   ```azure-cli
   curl -sL https://aka.ms/migrate-tool | bash -s -- -g ResourceGroupName -n AKSName -h ContainerRegistryName -r PathOfTheProject -y
   ```

   Ce script prend en charge les indicateurs suivants :

   ```cmd  
    -g Name of resource group of AKS Cluster [required]
    -n Name of AKS Cluster [required]
    -h Container registry name. Examples: ACR, Docker [required]
    -k Kubernetes namespace to deploy resources (uses 'default' otherwise)
    -r Path to root of the project that needs to be migrated (default = current working directory)
    -t Image name & tag in format 'name:tag' (default is 'projectName:stable')
    -i Enable a public endpoint to access your service over internet. (default is false)
    -c Docker build context path. (default = project root path passed to '-r' option)
    -y Doesn't prompt for non-tty terminals
    -d Helm Debug switch
   ```

1. Migrez manuellement toutes les personnalisations, telles que les paramètres de variable d’environnement, qui se trouvent dans *azds.yaml* dans le fichier *values.yml* de votre projet.
1. (Facultatif) Supprimez le fichier `azds.yaml` de votre projet.
1. Configurez la solution Bridge to Kubernetes sur votre application déployée. Pour plus d’informations sur l’utilisation de l’extension Bridge to Kubernetes dans Visual Studio, consultez [Utiliser Bridge to Kubernetes dans Visual Studio][use-btk-vs]. Pour VS Code, consultez [Utiliser Bridge to Kubernetes dans VS Code][use-btk-vsc].
1. Démarrez le débogage en utilisant le profil de lancement/débogage Bridge to Kubernetes nouvellement créé.
1. Si nécessaire, vous pouvez réexécuter le script pour le redéployer sur votre cluster.

## <a name="team-development-in-a-shared-cluster"></a>Développement en équipe dans un cluster partagé

Vous pouvez également utiliser un routage spécifique du développeur avec la solution Bridge to Kubernetes. Le scénario de développement en équipe du service Azure Dev Spaces utilise plusieurs espaces de noms Kubernetes pour isoler un service du reste de l’application en utilisant le concept d’espaces de noms parent et enfant. La solution Bridge to Kubernetes offre la même capacité, mais avec des caractéristiques de performances améliorées et au sein du même espace de noms d’application.

La solution Bridge to Kubernetes et le service Azure Dev Spaces requièrent que des en-têtes HTTP soient présents et propagés dans l’ensemble de l’application. Si vous avez déjà configuré votre application pour gérer la propagation d’en-tête pour le service Azure Dev Spaces, l’en-tête doit être mis à jour. Pour effectuer la transition du service Azure Dev Spaces vers la solution Bridge to Kubernetes, mettez à jour l’en-tête configuré de *azds-route-as* vers *kubernetes-route-as*.

## <a name="evaluate-bridge-to-kubernetes"></a>Évaluer la solution Bridge to Kubernetes

Si vous souhaitez expérimenter la solution Bridge to Kubernetes avant de désactiver le service Azure Dev Spaces dans votre cluster, le moyen le plus simple consiste à utiliser un nouveau cluster. Si vous essayez d’utiliser le service Azure Dev Spaces et la solution Bridge to Kubernetes en même temps sur le même cluster, vous allez rencontrer des problèmes lors de l’utilisation des fonctionnalités de routage sur les deux.

### <a name="use-visual-studio-to-evaluate-bridge-to-kubernetes"></a>Utiliser Visual Studio pour évaluer la solution Bridge to Kubernetes

1. Mettez à jour votre environnement de développement intégré (IDE) Visual Studio vers la version 16.7 ou une version ultérieure, et installez l’extension Bridge to Kubernetes à partir de [Visual Studio Marketplace][vs-marketplace].
1. Créez un cluster AKS et déployez votre application. Vous pouvez également utiliser un [exemple d’application][btk-sample-app].
1. Configurez la solution Bridge to Kubernetes sur votre application déployée. Pour plus d’informations sur l’utilisation de la solution Bridge to Kubernetes dans Visual Studio, consultez [Utiliser Bridge to Kubernetes][use-btk-vs].
1. Démarrez le débogage dans Visual Studio en utilisant le profil de débogage nouvellement créé de la solution Bridge to Kubernetes.

### <a name="use-visual-studio-code-to-evaluate-bridge-to-kubernetes"></a>Utiliser Visual Studio Code pour évaluer la solution Bridge to Kubernetes

1. Installez l’[extension Bridge to Kubernetes][vsc-marketplace].
1. Créez un cluster AKS et déployez votre application. Vous pouvez également utiliser un [exemple d’application][btk-sample-app].
1. Configurez la solution Bridge to Kubernetes sur votre application déployée. Pour plus d’informations sur l’utilisation de la solution Bridge to Kubernetes dans Visual Studio Code, consultez [Utiliser Bridge to Kubernetes][use-btk-vsc].
1. Démarrez le débogage dans Visual Studio en utilisant le profil de lancement nouvellement créé de la solution Bridge to Kubernetes.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le fonctionnement de la solution Bridge to Kubernetes.

> [!div class="nextstepaction"]
> [Fonctionnement de la solution Bridge to Kubernetes][how-it-works-bridge-to-kubernetes]


[azds-delete]: how-to/install-dev-spaces.md#remove-azure-dev-spaces-using-the-cli
[kubernetes-extension]: https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools
[btk-sample-app]: /visualstudio/containers/bridge-to-kubernetes#install-the-sample-application
[how-it-works-bridge-to-kubernetes]: /visualstudio/containers/overview-bridge-to-kubernetes
[use-btk-vs]: /visualstudio/containers/bridge-to-kubernetes#connect-to-your-cluster-and-debug-a-service
[use-btk-vsc]: https://code.visualstudio.com/docs/containers/bridge-to-kubernetes
[vs]: https://visualstudio.microsoft.com/
[vsc-marketplace]: https://marketplace.visualstudio.com/items?itemName=mindaro.mindaro
[vs-marketplace]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.mindaro
[vsc]: https://code.visualstudio.com/
