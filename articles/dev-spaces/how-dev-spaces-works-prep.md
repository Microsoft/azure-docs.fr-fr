---
title: Préparation d’un projet pour Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Décrit la préparation de votre projet avec Azure Dev Spaces
keywords: azds.yaml, Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: d2da69dd8a8c2683ff584dfd0ffc61cb023f2ece
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91968155"
---
# <a name="how-preparing-a-project-for-azure-dev-spaces-works"></a>Préparation d’un projet pour Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe sur un cluster Azure Kubernetes Service (AKS). Dev Spaces peut générer des fichiers Dockerfiles et des graphiques Helm pour votre projet. Dev Spaces crée et utilise également un fichier de configuration pour déployer, exécuter et déboguer vos applications Kubernetes dans AKS. Tous ces fichiers contiennent le code de votre application et peuvent être ajoutés à votre système de contrôle de version.

Cet article décrit ce qui se passe lorsque vous préparez votre projet pour une exécution dans AKS avec Dev Spaces.

## <a name="prepare-your-code"></a>Préparer votre code

Pour exécuter votre application dans un espace de développement, elle doit être conteneurisée, et vous devez définir la façon dont elle doit être déployée sur Kubernetes. Pour conteneuriser votre application, vous avez besoin d'un Dockerfile. Pour définir la façon dont votre application est déployée sur Kubernetes, vous avez besoin d'un [graphique Helm](https://docs.helm.sh/). Afin vous aider à créer à la fois le Dockerfile et le graphique Helm pour votre application, les outils côté client fournissent la commande `prep` :

```cmd
azds prep --enable-ingress
```

La commande `prep` examinera les fichiers de votre projet et tentera de créer le Dockerfile et le graphique Helm pour exécuter votre application dans Kubernetes. Actuellement, la commande `prep` génère un Dockerfile et un diagramme Helm avec les langages suivants :

* Java
* Node.js
* .NET Core

Vous *devez* exécuter la commande `prep` depuis un répertoire contenant le code source. Exécuter la commande `prep` depuis le répertoire adéquat permet aux outils côté client d'identifier le langage et de créer un Dockerfile approprié pour conteneuriser votre application. Vous pouvez également exécuter la commande `prep` à partir d'un répertoire contenant un fichier *pom.xml* pour les projets Java.

Si vous exécutez la commande `prep` à partir d'un répertoire qui ne contient aucun code source, les outils côté client ne généreront aucun Dockerfile. Ils afficheront également le message d'erreur suivant : *Impossible de générer Dockerfile en raison d’un langage non pris en charge*. Cette erreur se produit également si les outils côté client ne reconnaissent pas le type de projet.

Lorsque vous exécutez la commande `prep`, vous avez la possibilité de spécifier l'indicateur `--enable-ingress`. Cet indicateur demande au contrôleur de créer un point d'accès Internet pour ce service. Si vous ne spécifiez pas cet indicateur, le service n'est accessible que depuis le cluster ou via le tunnel localhost créé par les outils côté client. Vous pouvez activer ou désactiver ce comportement après avoir exécuté la commande `prep` en mettant à jour le graphique Helm généré.

La commande `prep` ne remplacera pas les Dockerfiles ou les graphiques Helm existants que contient votre projet. Si un Dockerfile ou un graphique Helm existant utilise la même convention d'appellation que les fichiers générés par la commande `prep`, celle-ci `prep` ignore la génération de ces fichiers. Sinon, la commande `prep` générera son propre Dockerfile ou graphique Helm à côté des fichiers existants.

> [!IMPORTANT]
> Azure Dev Spaces utilise le fichier Dockerfile et le graphique Helm de votre projet pour générer et exécuter votre code, mais vous pouvez modifier ces fichiers si vous souhaitez changer la façon dont le projet est généré et exécuté.

La commande `prep` génère également un fichier `azds.yaml` à la racine de votre projet. Azure Dev Spaces utilise ce fichier pour générer, installer, configurer et exécuter votre application. Ce fichier de configuration indique l'emplacement de votre Dockerfile et de votre graphique Helm et également une configuration supplémentaire en plus de ces artefacts.

Voici un exemple de fichier azds.yaml créé avec un [exemple d'application .NET Core](https://github.com/Azure/dev-spaces/tree/master/samples/dotnetcore/getting-started/webfrontend) :

```yaml
kind: helm-release
apiVersion: 1.1
build:
  context: .
  dockerfile: Dockerfile
install:
  chart: charts/webfrontend
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
  set:
    replicaCount: 1
    image:
      repository: webfrontend
      tag: $(tag)
      pullPolicy: Never
    ingress:
      annotations:
        kubernetes.io/ingress.class: traefik-azds
      hosts:
        # This expands to [space.s.][rootSpace.]webfrontend.<random suffix>.<region>.azds.io
        # Customize the public URL by changing the 'webfrontend' text between the $(rootSpacePrefix) and $(hostSuffix) tokens
        # For more information see https://aka.ms/devspaces/routing
        - $(spacePrefix)$(rootSpacePrefix)webfrontend$(hostSuffix)
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
    container:
      sync:
      - "**/Pages/**"
      - "**/Views/**"
      - "**/wwwroot/**"
      - "!**/*.{sln,csproj}"
      command: [dotnet, run, --no-restore, --no-build, --no-launch-profile, -c, "${BUILD_CONFIGURATION:-Debug}"]
      iterate:
        processesToKill: [dotnet, vsdbg]
        buildCommands:
        - [dotnet, build, --no-restore, -c, "${BUILD_CONFIGURATION:-Debug}"]
```

Le fichier `azds.yaml` généré par la commande `prep` est conçu pour fonctionner dans des scénarios de développement de projets simples et uniques. Si votre projet spécifique est plus complexe, vous devrez peut-être mettre à jour ce fichier après avoir exécuté la commande `prep`. Par exemple, il se peut que les processus de génération ou de lancement de votre projet doivent être modifiés en fonction de vos besoins en termes de développement ou de débogage. Votre projet contient peut-être plusieurs applications nécessitant plusieurs processus de génération ou un contenu de génération différent.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’exécution de code dans votre espace de développement, consultez [Comment fonctionne l’exécution de votre code avec Azure Dev Spaces][how-it-works-up].

[how-it-works-up]: how-dev-spaces-works-up.md