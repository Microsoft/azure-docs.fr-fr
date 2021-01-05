---
title: Obtenir les journaux d’activité et les événements d’instance de conteneur
description: Découvrez comment récupérer les journaux d'activité du conteneur et les événements dans Azure Container Instances pour résoudre les problèmes liés aux conteneurs.
ms.topic: article
ms.date: 12/30/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 20b6e1cfe6bb8f6ac721a401c3d0831d4f447edb
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746969"
---
# <a name="retrieve-container-logs-and-events-in-azure-container-instances"></a>Récupérer les journaux d’activité du conteneur et les événements dans Azure Container Instances

Dans Azure Container Instances, si un conteneur ne fonctionne pas correctement, commencez par afficher les journaux qui s'y rapportent avec [az container logs][az-container-logs] et diffusez en continu sa sortie standard et son erreur standard avec [az container attach][az-container-attach]. Vous pouvez également afficher les journaux et les événements pour les instances de conteneur dans le portail Azure ou envoyer des données de journal et d’événement pour les groupes de conteneurs aux [Journaux Azure Monitor](container-instances-log-analytics.md).

## <a name="view-logs"></a>Afficher les journaux d’activité

Pour afficher les journaux d’activité à partir de votre code d’application dans un conteneur, vous pouvez utiliser la commande [az container logs][az-container-logs].

Voici une sortie de journal extraite de l'exemple de conteneur basé sur des tâches dans [Définir la ligne de commande dans une instance de conteneur](container-instances-start-command.md#azure-cli-example), après avoir fourni une URL non valide à l’aide d’un remplacement de ligne de commande :

```azurecli
az container logs --resource-group myResourceGroup --name mycontainer
```

```output
Traceback (most recent call last):
  File "wordcount.py", line 11, in <module>
    urllib.request.urlretrieve (sys.argv[1], "foo.txt")
  File "/usr/local/lib/python3.6/urllib/request.py", line 248, in urlretrieve
    with contextlib.closing(urlopen(url, data)) as fp:
  File "/usr/local/lib/python3.6/urllib/request.py", line 223, in urlopen
    return opener.open(url, data, timeout)
  File "/usr/local/lib/python3.6/urllib/request.py", line 532, in open
    response = meth(req, response)
  File "/usr/local/lib/python3.6/urllib/request.py", line 642, in http_response
    'http', request, response, code, msg, hdrs)
  File "/usr/local/lib/python3.6/urllib/request.py", line 570, in error
    return self._call_chain(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 504, in _call_chain
    result = func(*args)
  File "/usr/local/lib/python3.6/urllib/request.py", line 650, in http_error_default
    raise HTTPError(req.full_url, code, msg, hdrs, fp)
urllib.error.HTTPError: HTTP Error 404: Not Found
```

## <a name="attach-output-streams"></a>Joindre des flux de sortie

La commande [liaison de conteneur az][az-container-attach] fournit des informations de diagnostic pendant le démarrage du conteneur. Une fois le conteneur démarré, il diffuse en continu STDOUT et STDERR sur votre console locale.

Par exemple, voici la sortie du conteneur basé sur des tâches dans [Définir la ligne de commande dans une instance de conteneur](container-instances-start-command.md#azure-cli-example), après avoir fourni une URL valide de fichier texte volumineux à traiter :

```azurecli
az container attach --resource-group myResourceGroup --name mycontainer
```

```output
Container 'mycontainer' is in state 'Unknown'...
Container 'mycontainer' is in state 'Waiting'...
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
Container 'mycontainer1' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 19:42:39+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:52+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-wordcount:latest"
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 19:42:55+00:00) Started container

Start streaming logs:
[('the', 22979),
 ('I', 20003),
 ('and', 18373),
 ('to', 15651),
 ('of', 15558),
 ('a', 12500),
 ('you', 11818),
 ('my', 10651),
 ('in', 9707),
 ('is', 8195)]
```

## <a name="get-diagnostic-events"></a>Récupérer des événements de diagnostic

Toutefois, si votre conteneur ne se déploie pas correctement, examinez les informations de diagnostic fournies par le fournisseur de ressources Azure Container Instances. Pour afficher les événements liés à votre conteneur, exécutez la commande [az container show][az-container-show] :

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer
```

La sortie inclut les propriétés principales de votre conteneur, ainsi que les événements de déploiement (qui apparaissent tronqués ici) :

```JSON
{
  "containers": [
    {
      "command": null,
      "environmentVariables": [],
      "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
      ...
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:22+00:00",
            "lastTimestamp": "2019-03-21T19:46:22+00:00",
            "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:28+00:00",
            "lastTimestamp": "2019-03-21T19:46:28+00:00",
            "message": "Successfully pulled image \"mcr.microsoft.com/azuredocs/aci-helloworld\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Created container",
            "name": "Created",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2019-03-21T19:46:31+00:00",
            "lastTimestamp": "2019-03-21T19:46:31+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "name": "mycontainer",
      "ports": [
        {
          "port": 80,
          "protocol": null
        }
      ],
      ...
    }
  ],
  ...
}
```
## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [résoudre les problèmes courants de conteneur et de déploiement](container-instances-troubleshooting.md) pour Azure Container Instances.

Découvrez comment envoyer des données de journaux et d’événements pour les groupes de conteneurs aux [Journaux Azure Monitor](container-instances-log-analytics.md).

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
