---
title: Résolution des problèmes courants
description: Découvrez comment résoudre les problèmes courants quand vous déployez, exécutez ou gérez Azure Container Instances
ms.topic: article
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d8e7fb85e369f5f278436370944eafeb1fb6a50e
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779513"
---
# <a name="troubleshoot-common-issues-in-azure-container-instances"></a>Résoudre les problèmes courants dans Azure Container Instances

Cet article explique comment résoudre les problèmes courants de gestion ou de déploiement de conteneurs dans Azure Container Instances. Consultez également le [Forum aux questions](container-instances-faq.md).

Si vous avez besoin d’une assistance supplémentaire, consultez les options d’**Aide + support** sur le [portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="issues-during-container-group-deployment"></a>Problèmes lors du déploiement d’un groupe de conteneurs
### <a name="naming-conventions"></a>Conventions d’affectation de noms

Lorsque vous définissez la spécification du conteneur, certains paramètres requièrent le respect des restrictions en matière d’affectation de noms. Le tableau suivant indique les exigences spécifiques des propriétés du groupe de conteneurs. Pour plus d’informations, consultez [Conventions d’affectation de noms][azure-name-restrictions] dans le Centre des architectures Azure, et [Règles de nommage et restrictions pour les ressources Azure][naming-rules].

| Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemple |
| --- | --- | --- | --- | --- | --- |
| Nom du conteneur<sup>1</sup> | 1-63 |Minuscules | Caractères alphanumériques et traits d’union n’importe où sauf en première ou dernière position |`<name>-<role>-container<number>` |`web-batch-container1` |
| Ports du conteneur | Entre 1 et 65 535 |Integer |Entier compris entre 1 et 65 535 |`<port-number>` |`443` |
| Étiquette du nom DNS | 5 à 63 |Insensible à la casse |Caractères alphanumériques et traits d’union n’importe où sauf en première ou dernière position |`<name>` |`frontend-site1` |
| Variable d’environnement | 1-63 |Insensible à la casse |Caractères alphanumériques et trait de soulignement (_) n’importe où sauf en première ou dernière position |`<name>` |`MY_VARIABLE` |
| Nom du volume | 5 à 63 |Minuscules |Caractères alphanumériques et traits d’union n’importe où sauf en première ou dernière position. Ne peut pas contenir deux traits d’union consécutifs. |`<name>` |`batch-output-volume` |

<sup>1</sup>Restriction également pour les noms de groupes de conteneurs quand ils ne sont pas spécifiés indépendamment des instances de conteneur, par exemple, avec des déploiements de commande `az container create`.

### <a name="os-version-of-image-not-supported"></a>La version du système d’exploitation de l’image n’est pas prise en charge

Si l’image spécifiée n’est pas prise en charge par Azure Container Instances, une erreur `OsVersionNotSupported` est renvoyée. L’erreur est semblable à ce qui suit, où `{0}` est le nom de l’image que vous avez tentée de déployer :

```json
{
  "error": {
    "code": "OsVersionNotSupported",
    "message": "The OS version of image '{0}' is not supported."
  }
}
```

Cette erreur se produit généralement lors du déploiement d'images Windows basées sur les versions 1709 ou 1803 du canal semi-annuel, qui ne sont pas prises en charge. Pour les images Windows prises en charge dans Azure Container Instances, consultez le [Forum aux questions](container-instances-faq.md#what-windows-base-os-images-are-supported).

### <a name="unable-to-pull-image"></a>Impossible d’extraire l’image

Si Azure Container Instances ne parvient pas à extraire votre image initialement, il réessaie pendant une certaine période, sans succès. Si l’opération d’extraction image continue à échouer, le déploiement ACI risque d’échouer, et vous verrez une erreur `Failed to pull image`.

Pour résoudre ce problème, supprimez l’instance de conteneur et réessayez votre déploiement. Vérifiez que l’image existe dans le Registre, et que vous avez correctement tapé le nom de l’image.

Des événements tels que les suivants sont alors affichés dans la sortie de [az container show][az-container-show] :

```bash
"events": [
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:19+00:00",
    "lastTimestamp": "2017-12-21T22:57:00+00:00",
    "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-hellowrld\"",
    "name": "Pulling",
    "type": "Normal"
  },
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:19+00:00",
    "lastTimestamp": "2017-12-21T22:57:00+00:00",
    "message": "Failed to pull image \"mcr.microsoft.com/azuredocs/aci-hellowrld\": rpc error: code 2 desc Error: image t/aci-hellowrld:latest not found",
    "name": "Failed",
    "type": "Warning"
  },
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:20+00:00",
    "lastTimestamp": "2017-12-21T22:57:16+00:00",
    "message": "Back-off pulling image \"mcr.microsoft.com/azuredocs/aci-hellowrld\"",
    "name": "BackOff",
    "type": "Normal"
  }
],
```
### <a name="resource-not-available-error"></a>Erreur : ressource non disponible

En raison d’une charge de ressources régionales variable dans Azure, vous pouvez recevoir l’erreur suivante quand vous tentez de déployer une instance de conteneur :

`The requested resource with 'x' CPU and 'y.z' GB memory is not available in the location 'example region' at this moment. Please retry with a different resource request or in another location.`

Cette erreur indique qu’en raison d’une charge importante dans la région dans laquelle vous tentez de déployer, les ressources spécifiées pour votre conteneur ne peuvent pas être allouées à ce moment-là. Utilisez une ou plusieurs des étapes d’atténuation suivantes pour vous aider à résoudre votre problème.

* Vérifier que les paramètres de votre déploiement de conteneur correspondent à ceux définis dans [Disponibilité des régions pour Azure Container Instances](container-instances-region-availability.md)
* Spécifier des paramètres de processeur et de mémoire inférieurs pour le conteneur
* Déployer sur une autre région Azure
* Déployer plus tard

## <a name="issues-during-container-group-runtime"></a>Problèmes lors de l’exécution du runtime de groupe de conteneurs
### <a name="container-continually-exits-and-restarts-no-long-running-process"></a>Le conteneur s’arrête et redémarre en permanence (pas de processus au long cours)

Les groupes de conteneurs sont définis par défaut sur la [stratégie de redémarrage](container-instances-restart-policy.md)**Toujours**, de sorte que les conteneurs du groupe de conteneurs redémarrent toujours après avoir été exécutés. Vous devrez peut-être définir ce paramètre sur **OnFailure** ou **Jamais** si vous envisagez d’exécuter des conteneurs basés sur des tâches. Si vous spécifiez **OnFailure** et constatez encore des redémarrages continus, il peut y avoir un problème avec l’application ou le script exécutés dans votre conteneur.

Lors de l’exécution de groupes de conteneurs sans processus au long cours, vous pouvez observer de multiples arrêts et redémarrages avec des images telles que Ubuntu ou Alpine. La connexion via [EXEC](container-instances-exec.md) ne va pas fonctionner car le conteneur n’a aucun processus pour le maintenir actif. Pour résoudre ce problème, ajoutez une commande de démarrage (voir exemple ci-dessous) au déploiement de votre groupe de conteneurs pour que le conteneur continue à s'exécuter.

```azurecli-interactive
## Deploying a Linux container
az container create -g MyResourceGroup --name myapp --image ubuntu --command-line "tail -f /dev/null"
```

```azurecli-interactive 
## Deploying a Windows container
az container create -g myResourceGroup --name mywindowsapp --os-type Windows --image mcr.microsoft.com/windows/servercore:ltsc2019
 --command-line "ping -t localhost"
```

L’API Container Instances et le portail Azure incluent une propriété `restartCount`. Pour vérifier le nombre de redémarrages d’un conteneur, vous pouvez utiliser la commande [az container show][az-container-show] dans Azure CLI. Dans l’exemple de sortie suivant (qui a été tronqué par souci de concision), vous pouvez voir la propriété `restartCount` à la fin de la sortie.

```json
...
 "events": [
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:06+00:00",
     "lastTimestamp": "2017-11-13T21:20:06+00:00",
     "message": "Pulling: pulling image \"myregistry.azurecr.io/aci-tutorial-app:v1\"",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Pulled: Successfully pulled image \"myregistry.azurecr.io/aci-tutorial-app:v1\"",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Created: Created container with id bf25a6ac73a925687cafcec792c9e3723b0776f683d8d1402b20cc9fb5f66a10",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Started: Started container with id bf25a6ac73a925687cafcec792c9e3723b0776f683d8d1402b20cc9fb5f66a10",
     "type": "Normal"
   }
 ],
 "previousState": null,
 "restartCount": 0
...
}
```

> [!NOTE]
> La plupart des images conteneur pour les distributions Linux définissent un interpréteur de commandes, tel que bash, comme commande par défaut. Un interpréteur de commandes n’étant pas en soi un service de longue durée, ces conteneurs quittent la procédure immédiatement et entrent dans une boucle de redémarrage lorsqu’ils sont configurés avec la stratégie de redémarrage par défaut (**Always**).

### <a name="container-takes-a-long-time-to-start"></a>Le démarrage du conteneur prend beaucoup de temps

Les trois principaux facteurs qui contribuent à l’heure de démarrage dans Azure Container Instances sont :

* [Taille de l'image](#image-size)
* [Emplacement de l’image](#image-location)
* [Images mises en cache](#cached-images)

Les images Windows ont des [considérations supplémentaires](#cached-images).

#### <a name="image-size"></a>Taille de l'image

Si le démarrage de votre conteneur prend beaucoup de temps, commencez par examiner la taille de votre image conteneur. Étant donné qu’Azure Container Instances extrait l’image conteneur à la demande, le temps de démarrage est directement lié à la taille de cette image.

Vous pouvez afficher la taille de l’image de votre conteneur à l’aide de la commande `docker images` dans l’interface CLI Docker :

```console
$ docker images
REPOSITORY                                    TAG       IMAGE ID        CREATED          SIZE
mcr.microsoft.com/azuredocs/aci-helloworld    latest    7367f3256b41    15 months ago    67.6MB
```

Pour que l’image conserve une petite taille, faites en sorte que l’image finale ne contienne aucun élément qui soit superflu au moment de l’exécution. Pour ce faire, vous pouvez utiliser des [builds à plusieurs étapes][docker-multi-stage-builds]. Grâce aux builds à plusieurs étapes, vous pouvez facilement faire en sorte que l’image finale ne contienne que les artefacts nécessaires à votre application, à l’exclusion de tout contenu supplémentaire qui était requis au moment de la génération.

#### <a name="image-location"></a>Emplacement de l’image

Il existe une autre façon de réduire l’impact de l’extraction de l’image sur le temps de démarrage de votre conteneur. Elle consiste à héberger l’image du conteneur dans [Azure Container Registry](../container-registry/index.yml) dans la région où vous envisagez de déployer Azure Container Instances. Cette opération raccourcit le chemin réseau que l’image conteneur doit parcourir, réduisant considérablement le temps de téléchargement.

#### <a name="cached-images"></a>Images mises en cache

Azure Container Instances utilise un mécanisme de mise en cache afin d'accélérer le démarrage des conteneurs pour les images générées à partir d'[images Windows](container-instances-faq.md#what-windows-base-os-images-are-supported) courantes, notamment `nanoserver:1809`, `servercore:ltsc2019` et `servercore:1809`. Les images Linux couramment utilisées, telles que `ubuntu:1604` et `alpine:3.6`, sont également mises en cache. Pour les images Windows et Linux, évitez d’utiliser la balise `latest`. Pour obtenir de l’aide, consultez les [meilleures pratiques relatives aux balises d’image](../container-registry/container-registry-image-tag-version.md) de Container Registry. Pour obtenir une liste à jour des images et balises mises en cache, utilisez l'API [Liste des images mises en cache][list-cached-images].

> [!NOTE]
> L’utilisation d’images basées sur Windows Server 2019 dans Azure Container Instances est disponible en version préliminaire.

#### <a name="windows-containers-slow-network-readiness"></a>Disponibilité lente du réseau des conteneurs Windows

Lors de la création initiale, les conteneurs Windows peuvent n’avoir aucune connectivité entrante ou sortante pendant 30 secondes (ou plus longtemps dans de rares cas). Si votre application conteneur requiert une connexion Internet, ajoutez un retard et réessayez la logique afin de laisser 30 secondes pour établir la connectivité Interne. Après l’installation initiale, la mise en réseau de conteneur devrait reprendre correctement.

### <a name="cannot-connect-to-underlying-docker-api-or-run-privileged-containers"></a>Ne peut pas se connecter à l’API Docker sous-jacent ou exécuter des conteneurs privilégiés

Azure Container Instances n’expose pas un accès direct à l’infrastructure sous-jacente qui héberge les groupes de conteneurs. Cela inclut l’accès à l’API Docker en cours d’exécution sur l’hôte du conteneur et les conteneurs privilégiés en cours d’exécution. Si vous avez besoin d’interaction avec le Docker, vérifiez la [Documentation de référence REST](/rest/api/container-instances/) pour voir ce que l’API ACI prend en charge. S’il manque des informations, envoyez une requête sur le [Forum Internet de commentaires ACI](https://aka.ms/aci/feedback).

### <a name="container-group-ip-address-may-not-be-accessible-due-to-mismatched-ports"></a>Il se peut que l’adresse IP d’un groupe de conteneurs soit inaccessible en raison d’une discordance de ports

Azure Container Instances ne prend pas encore en charge le mappage de ports, contrairement à une configuration Docker classique. Si vous constatez que l’adresses IP d’un groupe de conteneurs n’est pas accessible alors que vous pensez qu’elle devrait l’être, vérifiez que vous avez configuré votre image conteneur pour écouter les mêmes ports que ceux que vous exposez dans votre groupe de conteneurs avec la propriété `ports`.

Si vous souhaitez vérifier qu’Azure Container Instances peut écouter le port que vous avez configuré dans votre image conteneur, testez un déploiement de l’image `aci-helloworld` qui expose le port. Exécutez également l’application `aci-helloworld` afin qu’elle écoute le port. `aci-helloworld` accepte une variable d’environnement facultative `PORT` pour remplacer le port 80 écouté par défaut. Par exemple, pour tester le port 9000, définissez la [variable d’environnement](container-instances-environment-variables.md) lors de la création du groupe de conteneurs :

1. Configurez le groupe de conteneurs pour exposer le port 9000 et transmettez le numéro de port comme valeur de la variable d’environnement. L’exemple est mis en forme pour l’interpréteur de commandes Bash. Si vous préférez un autre interpréteur de commandes, PowerShell ou l’invite de commande, vous devrez ajuster les variables en conséquence.
    ```azurecli
    az container create --resource-group myResourceGroup \
    --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ip-address Public --ports 9000 \
    --environment-variables 'PORT'='9000'
    ```
1. Recherchez l’adresse IP du groupe de conteneurs dans la sortie de commande de `az container create`. Recherchez la valeur de **ip**. 
1. Une fois le conteneur correctement approvisionné, accédez à l’adresse IP et au port de l’application conteneur dans votre navigateur, par exemple : `192.0.2.0:9000`. 

    Vous devriez voir le message « Bienvenue dans Azure Container Instances ! » s’afficher dans l’application web.
1. Lorsque vous avez fini d’utiliser le conteneur, vous pouvez le supprimer à l’aide de la commande `az container delete` :

    ```azurecli
    az container delete --resource-group myResourceGroup --name mycontainer
    ```

## <a name="next-steps"></a>Étapes suivantes

Apprenez à [récupérer les journaux d'activité et les événements de conteneur](container-instances-get-logs.md) pour vous aider à déboguer vos conteneurs.

<!-- LINKS - External -->
[azure-name-restrictions]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources
[naming-rules]: ../azure-resource-manager/management/resource-name-rules.md
[windows-sac-overview]: /windows-server/get-started/semi-annual-channel-overview
[docker-multi-stage-builds]: https://docs.docker.com/engine/userguide/eng-image/multistage-build/
[docker-hub-windows-core]: https://hub.docker.com/_/microsoft-windows-servercore
[docker-hub-windows-nano]: https://hub.docker.com/_/microsoft-windows-nanoserver

<!-- LINKS - Internal -->
[az-container-show]: /cli/azure/container#az-container-show
[list-cached-images]: /rest/api/container-instances/location/listcachedimages
