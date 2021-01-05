---
title: Comment fonctionne l’exécution de votre code avec Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Décrit les processus d’exécution de votre code sur le service Azure Kubernetes avec Azure Dev Spaces
keywords: azds.yaml, Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: 1cace325f9415d46210636e5c04cc2d75589cc11
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014429"
---
# <a name="how-running-your-code-with-azure-dev-spaces-works"></a>Comment fonctionne l’exécution de votre code avec Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe sur un cluster Azure Kubernetes Service (AKS). Une fois que votre [projet est préparé pour s’exécuter dans un espace de développement][how-it-works-prep], vous pouvez utiliser Dev Spaces pour générer et exécuter votre projet dans votre cluster AKS.

Cet article décrit ce qui se produit lors de l’exécution de votre code dans AKS avec Dev Spaces.

## <a name="run-your-code"></a>Exécuter votre code

Pour exécuter votre code dans un espace de développement, lancez la commande `up` dans le même répertoire que votre fichier `azds.yaml` :

```cmd
azds up
```

La commande `up` charge les fichiers source de votre application et autres artefacts nécessaires pour générer et exécuter votre projet dans l'espace de développement. De là, le contrôleur dans votre espace de développement :

1. Crée les objets Kubernetes pour déployer votre application.
1. Génère le conteneur pour votre application.
1. Déploie votre application dans l'espace de développement.
1. Crée un nom DNS accessible au public pour votre point de terminaison d'application, s'il est configuré.
1. Utilise *port-forward* pour donner accès à votre point de terminaison d'application grâce à http://localhost.
1. Transfère stdout et stderr vers les outils côté client.


## <a name="starting-a-service"></a>Démarrage d'un service

Lorsque vous démarrez un service dans un espace de développement, les outils côté client et le contrôleur se coordonnent pour synchroniser vos fichiers source, créer votre conteneur et vos objets Kubernetes, puis exécuter votre application.

À un niveau plus granulaire, voici ce qui se passe lorsque vous exécutez `azds up` :

1. [Les fichiers sont synchronisés][sync-section] depuis l’ordinateur de l'utilisateur vers un stockage de fichiers Azure spécifique au cluster AKS de cet utilisateur. Le code source, le graphique Helm et les fichiers de configuration sont chargés.
1. Le contrôleur crée une demande de démarrage d'une nouvelle session. Cette requête contient plusieurs propriétés, notamment un ID unique, un nom d'espace, le chemin vers le code source et un indicateur de débogage.
1. Le contrôleur remplace le caractère générique *$(tag)* dans le graphique Helm par l'ID de session unique et installe ce graphique pour votre service. L'ajout d'une référence à l'ID de session unique au graphique Helm permet de relier le conteneur déployé dans le cluster AKS pour cette session spécifique à la demande de session et aux informations associées.
1. Lors de l'installation du graphique Helm Chart, le serveur d'admission webhook Kubernetes ajoute des conteneurs supplémentaires au pod de votre application pour l'instrumentation et l'accès au code source de votre projet. Les conteneurs devspaces-proxy et devspaces-proxy-init sont ajoutés pour permettre le traçage HTTP et le routage des espaces. Le conteneur devspaces-build est ajouté pour permettre au pod d'accéder à l'instance Docker et au code source du projet afin de générer le conteneur de votre application.
1. Lorsque le pod de l'application est lancé, le conteneur devspaces-build et le conteneur devspaces-proxy-init sont utilisés pour générer le conteneur de l'application. Le conteneur d'application et les conteneurs devspaces-proxy sont alors démarrés.
1. Après le démarrage du conteneur d'application, la fonctionnalité côté client utilise la fonctionnalité Kubernetes *port-forward* pour fournir un accès HTTP à votre application sur http://localhost. Cette redirection de port connecte votre ordinateur de développement au service dans votre espace de développement.
1. Lorsque tous les conteneurs du pod ont démarré, le service est actif. À ce stade, la fonctionnalité côté client commence à diffuser en continu les traces HTTP, stdout et stderr. Ces informations sont affichées par la fonctionnalité côté client pour le développeur.

## <a name="updating-a-running-service"></a>Mise à jour d’un service en cours d’exécution

Pendant l'exécution d'un service, Azure Dev Spaces peut mettre à jour ce service si l'un des fichiers source du projet change. Dev Spaces gère également la mise à jour du service différemment selon le type de fichier modifié. Il existe trois façons pour Dev Spaces de mettre à jour un service en cours d'exécution :

* Mise à jour directe d’un fichier
* Régénération et redémarrage du processus de l'application à l'intérieur du conteneur de l'application en cours d'exécution
* Régénération et redéploiement du conteneur de l'application

![Synchronisation des fichiers Azure Dev Spaces](media/how-dev-spaces-works/file-sync.svg)

Certains fichiers du projet représentant des ressources statiques, notamment les fichiers html, css et cshtml, peuvent être mis à jour directement dans le conteneur de l'application, sans aucun redémarrage. Si une ressource statique change, le nouveau fichier est synchronisé avec l'espace de développement puis utilisé par le conteneur courant.

Les modifications apportées aux fichiers tels que le code source ou les fichiers de configuration de l'application peuvent être appliquées en redémarrant le processus de l'application dans le conteneur actif. Une fois ces fichiers synchronisés, le processus de l'application est relancé dans le conteneur en cours d'exécution à l’aide du processus *devhostagent*. Lors de la création initiale du conteneur de l'application, le contrôleur remplace la commande de démarrage de l'application par un processus différent appelé *devhostagent*. Le processus réel de l'application est alors exécuté en tant que processus enfant sous *devhostagent*, et sa sortie est extraite à l’aide de *devhostagent*. Le processus *devhostagent* fait également partie de Dev Spaces et peut exécuter des commandes dans le conteneur courant pour le compte de Dev Spaces. Lors d'un redémarrage, *devhostagent* :

* Arrête le ou les processus actuels associés à l’application
* Régénère l'application
* Redémarre le ou les processus associés à l'application

La façon dont *devhostagent* exécute les étapes précédentes est [définie dans le fichier de configuration dans `azds.yaml`][azds-yaml-section].

Les mises à jour des fichiers du projet tels que les fichiers Dockerfile, les fichiers csproj ou toute partie du graphique Helm nécessitent la régénération et le redéploiement du conteneur de l’application. Si un de ces fichiers est synchronisé avec l’espace de développement, le contrôleur exécute la commande [helm upgrade][helm-upgrade], puis un conteneur de l’application est régénéré et redéployé.

## <a name="file-synchronization"></a>Synchronisation de fichiers

La première fois qu’une application démarre dans un espace de développement, tous les fichiers de l’application source sont chargés. Pendant l’exécution de l’application et lors des redémarrages ultérieurs, seuls les fichiers modifiés sont chargés. Deux fichiers sont utilisés pour coordonner ce processus : un fichier côté client et un autre côté contrôleur.

Stocké dans un répertoire temporaire, le fichier côté client est nommé en fonction du hachage du répertoire du projet en cours d’exécution dans les espaces de développement. Par exemple, sur Windows, le fichier peut être *Users\USERNAME\AppData\Local\Temp\1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef.synclog* pour votre projet. Sur Linux, le fichier côté client est stocké dans le répertoire */tmp*. Vous pouvez trouver le répertoire sur macOS en exécutant la commande `echo $TMPDIR`.

Ce fichier au format JSON contient :

* Une entrée pour chaque fichier de projet synchronisé avec l’espace de développement
* Un ID de synchronisation
* L’horodatage de la dernière opération de synchronisation

Chaque entrée du fichier du projet contient un chemin d’accès au fichier et son horodatage.

Le fichier côté contrôleur est stocké sur le cluster AKS. Il contient l’ID de synchronisation et l’horodatage de la dernière synchronisation.

Une synchronisation se produit lorsque les horodatages de synchronisation ne correspondent pas entre les fichiers côté client et côté contrôleur. Au cours d'une synchronisation, les outils côté client effectuent une itération sur les entrées du fichier côté client. Si l'horodatage du fichier est postérieur à l'horodatage de synchronisation, ce fichier est synchronisé avec l'espace de développement. Une fois la synchronisation terminée, les horodatages de synchronisation sont mis à jour sur les fichiers côté client et côté contrôleur.

Tous les fichiers du projet sont synchronisés si le fichier côté client n'est pas présent. Ce comportement vous permet de forcer une synchronisation complète en supprimant le fichier côté client.

## <a name="how-running-your-code-is-configured"></a>Configuration de l'exécution de votre code

Azure Dev Spaces utilise le fichier `azds.yaml` pour installer et configurer votre service. Le contrôleur utilise la propriété `install` du fichier `azds.yaml` pour installer le graphique Helm et créer les objets Kubernetes :

```yaml
...
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
...
```

Par défaut, la commande `prep` génère le graphique Helm. Elle définit également la propriété *install.chart* sur le répertoire du graphique Helm. Si vous voulez utiliser un graphique Helm à un autre emplacement, vous pouvez mettre à jour cette propriété en fonction de cet emplacement.

Lors de l'installation des graphiques Helm, Azure Dev Spaces fournit un moyen de remplacer les valeurs du graphique Helm. Les valeurs par défaut du graphique Helm sont affichées dans `charts/APP_NAME/values.yaml`.

En utilisant la propriété *install.values*, vous pouvez répertorier un ou plusieurs fichiers qui définissent les valeurs que vous voulez remplacer dans le graphique Helm. Par exemple, vous pouvez utiliser cette fonctionnalité de remplacement pour obtenir une configuration de nom d'hôte ou de base de données spécifique lors de l'exécution de votre application dans un espace de développement. Vous pouvez également ajouter un *?* . à la fin de n'importe quel nom de fichier pour le définir comme facultatif.

La propriété *install.set* vous permet de configurer une ou plusieurs valeurs à remplacer dans le graphique Helm. Toutes les valeurs configurées dans *install.set* remplaceront les valeurs configurées dans les fichiers répertoriés dans *install.values*. Les propriétés sous *install.set* dépendent des valeurs du graphique Helm et peuvent être différentes selon le graphique Helm généré.

Dans l'exemple ci-dessus, la propriété *install.set.replicaCount* indique au contrôleur combien d'instances de votre application doivent être exécutées dans votre espace de développement. Selon votre scénario, vous pouvez augmenter cette valeur, mais cela aura un impact sur la connexion d'un débogueur au pod de votre application. Pour plus d’informations, consultez l’[article de dépannage][troubleshooting].

Dans le graphique Helm généré, l'image du conteneur est définie sur *{{ .Values.image.repository }}:{{ .Values.image.tag }}* . Le ficher `azds.yaml` définit la propriété *install.set.image.tag* sur *$(tag)* par défaut, utilisée comme valeur pour *{{ .Values.image.tag }}* . En définissant la propriété *install.set.image.tag* de cette façon, l’image du conteneur de votre application sera étiquetée d'une manière distincte lors de l'exécution des espaces de développement Azure. Dans ce cas spécifique, l’image est marquée *\<value from image.repository>:$(tag)* . Vous devez utiliser la variable *$(tag)* comme valeur de *install.set.image.tag* pour que Dev Spaces reconnaisse et situe le conteneur dans le cluster AKS.

Dans l'exemple ci-dessus, `azds.yaml` définit *install.set.ingress.hosts*. La propriété *install.set.ingress.hosts* définit un format de nom d'hôte pour les points de terminaison publics. Cette propriété utilise également *$$(spacePrefix)* , *$(rootSpacePrefix)* , et *$(hostSuffix)* , valeurs fournies par le contrôleur.

*$$(spacePrefix)* est le nom de l'espace de développement enfant, qui prend la forme *SPACENAME.s*. *$$(rootSpacePrefix)* est le nom de l'espace parent. Par exemple, si *azureuser* est un espace enfant de *default*, la valeur pour *$(rootSpacePrefix)* est *default*, et la valeur de *$(spacePrefix)* est *azureuser.s*. Si l'espace n'est pas un espace enfant, *$(spacePrefix)* est vide. Par exemple, si l'espace *default* n'a pas d'espace parent, la valeur pour *$(rootSpacePrefix)* est *default* et la valeur de *$(spacePrefix)* est vide. *$$(hostSuffix)* est un suffixe DNS qui pointe vers le contrôleur d'entrée Azure Dev Spaces exécuté dans votre cluster AKS. Ce suffixe DNS correspond à une entrée DNS générique, par exemple *\*.RANDOM_VALUE.eus.azds.io*, créée lorsque le contrôleur Azure Dev Spaces a été ajouté à votre cluster AKS.

Dans le fichier `azds.yaml` ci-dessus, vous pouvez également mettre à jour *install.set.ingress.hosts* pour changer le nom d'hôte de votre application. Par exemple, si vous voulez simplifier le nom d'hôte de votre application de *$$(spacePrefix)$(rootSpacePrefix)$(hostSuffix)* à *$(spacePrefix)$(rootSpacePrefix)web$(hostSuffix)* .

Afin de générer le conteneur pour votre application, le contrôleur utilise les sections ci-dessous du fichier de configuration `azds.yaml` :

```yaml
build:
  context: .
  dockerfile: Dockerfile
...
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
...
```

Le contrôleur utilise un Dockerfile pour générer et exécuter votre application.

La propriété *build.context* indique le répertoire où figurent les Dockerfiles. La propriété *build.dockerfile* définit le nom du Dockerfile pour générer la version de production de l'application. La propriété *configurations.develop.build.build.dockerfile* configure le nom du Dockerfile pour la version de développement de l'application.

Avoir différents Dockerfiles pour le développement et la production vous permet d'activer certains éléments pendant le développement et de désactiver ces éléments pour les déploiements en production. Par exemple, vous pouvez activer le débogage ou la journalisation détaillée pendant le développement et le désactiver dans un environnement de production. Vous pouvez également mettre à jour ces propriétés si vos Dockerfiles sont nommés différemment ou se trouvent dans un autre emplacement.

Pour accélérer l’itération pendant le développement, Azure Dev Spaces synchronisera les changements de votre projet local et mettra à jour progressivement votre application. La section ci-dessous du fichier de configuration `azds.yaml` sert à configurer la synchronisation et la mise à jour :

```yaml
...
configurations:
  develop:
    ...
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
...
```

Les fichiers et répertoires qui synchroniseront les changements sont répertoriés dans la propriété *configurations.develop.container.sync*. Ces répertoires sont synchronisés initialement lorsque vous exécutez la commande `up` et lorsque des changements sont détectés. Si vous souhaitez synchroniser des répertoires supplémentaires ou différents avec votre espace de développement, vous pouvez modifier cette propriété.

La propriété *configurations.develop.container.iterate.buildCommands* indique comment générer l'application dans un scénario de développement. La propriété *configurations.develop.container.command* fournit la commande pour exécuter l'application dans un scénario de développement. Vous pouvez mettre à jour l'une ou l'autre de ces propriétés si vous souhaitez utiliser d'autres indicateurs ou paramètres de génération ou d'exécution pendant le développement.

La propriété *configurations.develop.container.iterate.processesToKill* répertorie les processus à tuer pour arrêter l'application. Vous pouvez mettre à jour cette propriété pour changer le comportement de redémarrage de votre application pendant le développement. Par exemple, si vous avez mis à jour les propriétés *configurations.develop.container.iterate.buildCommands* ou *configurations.develop.container.command* pour modifier la génération ou le lancement de l'application, vous devrez peut-être changer quels processus sont arrêtés.

Lorsque vous préparez votre code à l'aide de la commande `azds prep`, vous avez la possibilité d'ajouter l'indicateur `--enable-ingress`. L'ajout de l'indicateur `--enable-ingress` crée une URL accessible au public pour votre application. Si vous omettez cet indicateur, l'application n'est accessible que dans le cluster ou en utilisant le tunnel localhost. Après avoir exécuté la commande `azds prep`, vous pouvez modifier ce paramètre en changeant la propriété *ingress.enabled* dans `charts/APPNAME/values.yaml` :

```yaml
ingress:
  enabled: true
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la mise en réseau et la façon dont les requêtes sont routées dans Azure Dev Spaces consultez [Fonctionnement du routage avec Azure Dev Spaces][how-it-works-routing].

Pour en savoir plus sur l’itération et le développement rapides avec Kubernetes, consultez [Fonctionnement de la solution Bridge to Kubernetes][how-it-works-bridge-to-kubernetes] et [Comment fonctionne le débogage à distance de votre code avec Azure Dev Spaces][how-it-works-remote-debugging].


[azds-yaml-section]: #how-running-your-code-is-configured
[helm-upgrade]: https://helm.sh/docs/intro/using_helm/#helm-upgrade-and-helm-rollback-upgrading-a-release-and-recovering-on-failure
[how-it-works-bridge-to-kubernetes]: /visualstudio/containers/overview-bridge-to-kubernetes
[how-it-works-prep]: how-dev-spaces-works-prep.md
[how-it-works-remote-debugging]: how-dev-spaces-works-remote-debugging.md
[how-it-works-routing]: how-dev-spaces-works-routing.md
[sync-section]: #file-synchronization
[troubleshooting]: troubleshooting.md