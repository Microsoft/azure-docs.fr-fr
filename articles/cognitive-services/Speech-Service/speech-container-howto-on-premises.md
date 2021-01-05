---
title: Utiliser les conteneurs du service Speech avec Kubernetes et Helm
titleSuffix: Azure Cognitive Services
description: À l’aide de Kubernetes et de Helm pour définir les images de conteneur de reconnaissance et de synthèse vocale, nous allons créer un package Kubernetes. Ce package sera déployé sur un cluster Kubernetes local.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/30/2020
ms.author: aahi
ms.openlocfilehash: c3791a9049a3eab3ddd6fc70073629c38830dbc7
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96002268"
---
# <a name="use-speech-service-containers-with-kubernetes-and-helm"></a>Utiliser les conteneurs du service Speech avec Kubernetes et Helm

L’une des options permettant de gérer vos conteneurs vocaux en local consiste à utiliser Kubernetes et Helm. À l’aide de Kubernetes et de Helm pour définir les images de conteneur de reconnaissance et de synthèse vocale, nous allons créer un package Kubernetes. Ce package sera déployé sur un cluster Kubernetes local. Enfin, nous allons découvrir comment tester les services déployés et diverses options de configuration. Pour plus d’informations sur l’exécution des conteneurs Docker sans orchestration Kubernetes, consultez [Installer et exécuter des conteneurs Speech](speech-container-howto.md).

## <a name="prerequisites"></a>Prérequis

L’utilisation locale des conteneurs Speech est soumise aux prérequis suivants :

| Obligatoire | Objectif |
|----------|---------|
| Compte Azure. | Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][free-azure-account] avant de commencer. |
| Accès à Container Registry | Pour que Kubernetes puisse extraire les images docker dans le cluster, il aura besoin d’accéder au registre de conteneurs. |
| Kubernetes CLI | L’interface [Kubernetes CLI][kubernetes-cli] est requise pour gérer les informations d’identification partagées à partir du registre de conteneurs. Kubernetes est également nécessaire avant Helm, qui est le gestionnaire de package de Kubernetes. |
| Helm CLI | Installez l’[interface de ligne de commande Helm][helm-install], utilisée pour installer un graphique Helm (définition de package de conteneur). |
|Ressource Speech |Pour pouvoir utiliser ces conteneurs, vous devez avoir :<br><br>Ressource Azure de _Speech_ permettant d’obtenir la clé de facturation et l’URI du point de terminaison de facturation associés. Les deux valeurs, disponibles dans les pages Vue d’ensemble de **Speech** et Clés du portail Azure, sont nécessaires au démarrage du conteneur.<br><br>**{API_KEY}** : clé de ressource<br><br>**{ENDPOINT_URI}**  : exemple d’URI de point de terminaison : `https://westus.api.cognitive.microsoft.com/sts/v1.0`|

## <a name="the-recommended-host-computer-configuration"></a>La configuration d’ordinateur hôte recommandée

Reportez-vous aux informations concernant l’[ordinateur hôte du conteneur du service Speech][speech-container-host-computer] comme référence. Ce *graphique Helm* calcule automatiquement les besoins en ressources processeur et mémoire en fonction du nombre de décodages (demandes simultanées) que l’utilisateur spécifie. En outre, il s’ajuste en fonction selon que les optimisations pour l’entrée audio/textuelles sont configurées sur `enabled` ou non. Par défaut, le graphique Helm considère deux demandes simultanées et l’optimisation désactivée.

| Service | Processeur/conteneur | Mémoire/conteneur |
|--|--|--|
| **Reconnaissance vocale** | un décodeur nécessite un minimum de 1 150 millicores. Si `optimizedForAudioFile` est activé, 1 950 millicores sont requis. (par défaut : deux décodeurs) | Requis : 2 Go<br>Limité :  4 Go |
| **Synthèse vocale** | une demande simultanée nécessite un minimum de 500 millicores. Si `optimizeForTurboMode` est activé, 1 000 millicores sont requis. (par défaut : deux demandes simultanées) | Requis : 1 Go<br> Limité : 2 Go |

## <a name="connect-to-the-kubernetes-cluster"></a>Se connecter au cluster Kubernetes

L’ordinateur hôte doit avoir un cluster Kubernetes disponible. Consultez ce didacticiel sur [le déploiement d’un cluster Kubernetes](../../aks/tutorial-kubernetes-deploy-cluster.md) pour des informations conceptuelles sur la façon de déployer un cluster Kubernetes sur un ordinateur hôte.

## <a name="configure-helm-chart-values-for-deployment"></a>Configurer les valeurs du graphique Helm pour le déploiement

Visitez le [hub Microsoft Helm][ms-helm-hub] pour accéder à tous les graphiques Helm publiquement offerts par Microsoft. Dans le hub Helm Microsoft, vous trouverez le **Graphique Cognitive Services Speech local**. Le graphique **Cognitive Services Speech local** est celui que nous allons installer, mais nous devons tout d’abord créer un fichier `config-values.yaml` avec des configurations explicites. Commençons par ajouter le référentiel Microsoft à notre instance Helm.

```console
helm repo add microsoft https://microsoft.github.io/charts/repo
```

Ensuite, nous allons configurer nos valeurs de graphique Helm. Copiez et collez la configuration YAML suivante dans un fichier nommé `config-values.yaml`. Pour plus d’informations sur la personnalisation du **Graphique Helm pour Cognitive Services Speech local**, consultez [Personnaliser des graphiques Helm](#customize-helm-charts). Remplacez les commentaires `# {ENDPOINT_URI}` et `# {API_KEY}` par vos propres valeurs.

```yaml
# These settings are deployment specific and users can provide customizations
# speech-to-text configurations
speechToText:
  enabled: true
  numberOfConcurrentRequest: 3
  optimizeForAudioFile: true
  image:
    registry: mcr.microsoft.com
    repository: azure-cognitive-services/speechservices/speech-to-text
    tag: latest
    pullSecrets:
      - mcr # Or an existing secret
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}

# text-to-speech configurations
textToSpeech:
  enabled: true
  numberOfConcurrentRequest: 3
  optimizeForTurboMode: true
  image:
    registry: mcr.microsoft.com
    repository: azure-cognitive-services/speechservices/speech-to-text
    tag: latest
    pullSecrets:
      - mcr # Or an existing secret
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}
```

> [!IMPORTANT]
> Si les valeurs `billing` et `apikey` ne sont pas fournies, les services expireront après 15 minutes. De même, la vérification échouera, car les services ne seront pas disponibles.

### <a name="the-kubernetes-package-helm-chart"></a>Le package Kubernetes (graphique Helm)

Le *graphique Helm* contient la configuration de la ou des images docker à extraire du registre de conteneurs `mcr.microsoft.com`.

> Un [graphique Helm][helm-charts] est une collection de fichiers qui décrivent un ensemble de ressources Kubernetes. Un graphique unique peut être utilisé pour déployer quelque chose de simple comme un pod mis en cache, ou quelque chose de complexe, comme une pile d’application web complète avec des serveurs HTTP, des bases de données, des caches et ainsi de suite.

Les *graphiques Helm* fournis tirent (pull) les images Docker du service Speech (à la fois les services de synthèse et de reconnaissance vocale) à partir du registre de conteneurs `mcr.microsoft.com`.

## <a name="install-the-helm-chart-on-the-kubernetes-cluster"></a>Installer le graphique Helm sur le cluster Kubernetes

Pour installer le *graphique helm*, nous allons devoir exécuter la commande [`helm install`][helm-install-cmd], en remplaçant `<config-values.yaml>` par l’argument de nom de chemin d’accès et de fichier approprié. Le graphique Helm `microsoft/cognitive-services-speech-onpremise` référencé ci-dessous est disponible sur le [hub Microsoft Helm][ms-helm-hub-speech-chart].

```console
helm install onprem-speech microsoft/cognitive-services-speech-onpremise \
    --version 0.1.1 \
    --values <config-values.yaml> 
```

Voici un exemple de sortie que vous pouvez vous attendre à voir pour une exécution d’installation réussie :

```console
NAME:   onprem-speech
LAST DEPLOYED: Tue Jul  2 12:51:42 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                             READY  STATUS             RESTARTS  AGE
speech-to-text-7664f5f465-87w2d  0/1    Pending            0         0s
speech-to-text-7664f5f465-klbr8  0/1    ContainerCreating  0         0s
text-to-speech-56f8fb685b-4jtzh  0/1    ContainerCreating  0         0s
text-to-speech-56f8fb685b-frwxf  0/1    Pending            0         0s

==> v1/Service
NAME            TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)       AGE
speech-to-text  LoadBalancer  10.0.252.106  <pending>    80:31811/TCP  1s
text-to-speech  LoadBalancer  10.0.125.187  <pending>    80:31247/TCP  0s

==> v1beta1/PodDisruptionBudget
NAME                                MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
speech-to-text-poddisruptionbudget  N/A            20%              0                    1s
text-to-speech-poddisruptionbudget  N/A            20%              0                    1s

==> v1beta2/Deployment
NAME            READY  UP-TO-DATE  AVAILABLE  AGE
speech-to-text  0/2    2           0          0s
text-to-speech  0/2    2           0          0s

==> v2beta2/HorizontalPodAutoscaler
NAME                       REFERENCE                  TARGETS        MINPODS  MAXPODS  REPLICAS  AGE
speech-to-text-autoscaler  Deployment/speech-to-text  <unknown>/50%  2        10       0         0s
text-to-speech-autoscaler  Deployment/text-to-speech  <unknown>/50%  2        10       0         0s


NOTES:
cognitive-services-speech-onpremise has been installed!
Release is named onprem-speech
```

Le déploiement de Kubernetes peut prendre plusieurs minutes. Pour vérifier que les pods et les services sont correctement déployés et disponibles, exécutez la commande suivante :

```console
kubectl get all
```

Vous devriez obtenir un graphique similaire à la sortie suivante :

```console
NAME                                  READY     STATUS    RESTARTS   AGE
pod/speech-to-text-7664f5f465-87w2d   1/1       Running   0          34m
pod/speech-to-text-7664f5f465-klbr8   1/1       Running   0          34m
pod/text-to-speech-56f8fb685b-4jtzh   1/1       Running   0          34m
pod/text-to-speech-56f8fb685b-frwxf   1/1       Running   0          34m

NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
service/kubernetes       ClusterIP      10.0.0.1       <none>           443/TCP        3h
service/speech-to-text   LoadBalancer   10.0.252.106   52.162.123.151   80:31811/TCP   34m
service/text-to-speech   LoadBalancer   10.0.125.187   65.52.233.162    80:31247/TCP   34m

NAME                             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/speech-to-text   2         2         2            2           34m
deployment.apps/text-to-speech   2         2         2            2           34m

NAME                                        DESIRED   CURRENT   READY     AGE
replicaset.apps/speech-to-text-7664f5f465   2         2         2         34m
replicaset.apps/text-to-speech-56f8fb685b   2         2         2         34m

NAME                                                            REFERENCE                   TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/speech-to-text-autoscaler   Deployment/speech-to-text   1%/50%    2         10        2          34m
horizontalpodautoscaler.autoscaling/text-to-speech-autoscaler   Deployment/text-to-speech   0%/50%    2         10        2          34m
```

### <a name="verify-helm-deployment-with-helm-tests"></a>Vérifier le déploiement de Helm avec des tests Helm

Les graphiques Helm installés définissent les *tests Help*, qui sont proposés à des fins pratiques pour la vérification. Ces tests valident la disponibilité du service. Pour vérifier à la fois les services de **reconnaissance vocale** et de **synthèse vocale**, nous allons exécuter la commande [Help test][helm-test].

```console
helm test onprem-speech
```

> [!IMPORTANT]
> Ces tests échouent si l’état du pod n’est pas `Running` ou si le déploiement n’est pas répertorié sous la colonne `AVAILABLE`. Soyez patient, car cela peut prendre plus de dix minutes.

Ces tests génèrent différents résultats d’état :

```console
RUNNING: speech-to-text-readiness-test
PASSED: speech-to-text-readiness-test
RUNNING: text-to-speech-readiness-test
PASSED: text-to-speech-readiness-test
```

Comme alternative à l’exécution des *tests Help*, vous pouvez recueillir les *adresses IP externes* et ports correspondants avec la commande `kubectl get all`. En utilisant l’adresse IP et le port, ouvrez un navigateur web et accédez à `http://<external-ip>:<port>:/swagger/index.html` pour voir la ou les pages Swagger de l’API.

## <a name="customize-helm-charts"></a>Personnaliser des graphiques Helm

Les graphiques Helm sont hiérarchiques. Cette hiérarchie permet l’héritage de graphiques ainsi que le concept de spécificité, où les paramètres plus spécifiques remplacent les règles héritées.

[!INCLUDE [Speech umbrella-helm-chart-config](includes/speech-umbrella-helm-chart-config.md)]

[!INCLUDE [Speech-to-Text Helm Chart Config](includes/speech-to-text-chart-config.md)]

[!INCLUDE [Text-to-Speech Helm Chart Config](includes/text-to-speech-chart-config.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’installation d’applications avec Helm dans Azure Kubernetes Service (AKS), [consultez ceci][installing-helm-apps-in-aks].

> [!div class="nextstepaction"]
> [Conteneurs Cognitive Services][cog-svcs-containers]

<!-- LINKS - external -->
[free-azure-account]: https://azure.microsoft.com/free
[git-download]: https://git-scm.com/downloads
[azure-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[docker-engine]: https://www.docker.com/products/docker-engine
[kubernetes-cli]: https://kubernetes.io/docs/tasks/tools/install-kubectl
[helm-install]: https://helm.sh/docs/intro/install/
[helm-install-cmd]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[tiller-install]: https://helm.sh/docs/install/#installing-tiller
[helm-charts]: https://helm.sh/docs/topics/charts/
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-test]: https://v2.helm.sh/docs/helm/#helm-test
[ms-helm-hub]: https://hub.helm.sh/charts/microsoft
[ms-helm-hub-speech-chart]: https://hub.helm.sh/charts/microsoft/cognitive-services-speech-onpremise

<!-- LINKS - internal -->
[speech-container-host-computer]: speech-container-howto.md#the-host-computer
[installing-helm-apps-in-aks]: ../../aks/kubernetes-helm.md
[cog-svcs-containers]: ../cognitive-services-container-support.md