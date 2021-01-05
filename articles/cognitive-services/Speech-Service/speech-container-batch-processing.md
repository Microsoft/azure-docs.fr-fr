---
title: Kit de traitement par lots pour des conteneurs Speech
titleSuffix: Azure Cognitive Services
description: Utilisez le kit de traitement par lots pour mettre à l’échelle les requêtes de conteneur Speech.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/22/2020
ms.author: aahi
ms.openlocfilehash: 80e0de73bbeae2ee1a79199fde34a3c430959ac8
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93356703"
---
# <a name="batch-processing-kit-for-speech-containers"></a>Kit de traitement par lots pour des conteneurs Speech

Utilisez le kit de traitement par lots pour compléter et effectuer un scale-out des charges de travail sur des conteneurs Speech. Disponible en tant que conteneur, cet utilitaire open source permet de faciliter la transcription par lot pour un grand nombre de fichiers audio, sur des points de terminaison de conteneur Speech locaux et basés sur le cloud. 

:::image type="content" source="media/containers/general-diagram.png" alt-text="Diagramme montrant un exemple de flux de travail de conteneurs de kits par lots.":::

Le conteneur de kits par lots est disponible gratuitement sur [GitHub](https://github.com/microsoft/batch-processing-kit) et [hub Docker](https://hub.docker.com/r/batchkit/speech-batch-kit/tags). Vous n’êtes [facturé](speech-container-howto.md#billing) que pour les conteneurs Speech que vous utilisez.

| Fonctionnalité  | Description  |
|---------|---------|
| Distribution de fichiers audio par lot     | Distribuer automatiquement un grand nombre de fichiers vers des points de terminaison de conteneur Speech locaux ou basés sur le cloud. Les fichiers peuvent se trouver sur n’importe quel volume compatible POSIX, y compris les systèmes de fichiers réseau.       |
| Intégration du Kit de développement logiciel (SDK) Speech | Transmettez les indicateurs courants au kit de développement logiciel (SDK) Speech, notamment : les n meilleures hypothèses, la diarisation, le langage, le masquage des grossièretés.  |
|Exécuter des modes     | Exécutez le client Batch une seule fois, en continu en arrière-plan ou créez des points de terminaison HTTP pour les fichiers audio.         |
| Tolérance de panne | Procédez automatiquement à une nouvelle tentative et poursuivez la transcription sans perdre la progression et différenciez les erreurs qui peuvent être retentées et celles qui ne le peuvent pas. |
| Détection de la disponibilité du point de terminaison | Si un point de terminaison devient indisponible, le client par lot continue la transcription à l’aide d’autres points de terminaison du conteneur. Une fois que le client est de nouveau disponible, il commence automatiquement à utiliser le point de terminaison.   |
| Échange à chaud du point de terminaison | Ajoutez, supprimez ou modifiez des points de terminaison de conteneur Speech au cours de l’exécution sans interrompre l’avancement du traitement. Les mises à jour sont immédiates. |
| Journalisation en temps réel | Journalisation en temps réel des requêtes tentées, des timestamps et des motifs d’échec, avec les fichiers journaux du kit de développement logiciel (SDK) Speech pour chaque fichier audio. |

## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger le dernier conteneur du kit par lot.

```bash
docker pull docker.io/batchkit/speech-batch-kit:latest
```

## <a name="endpoint-configuration"></a>Configuration du point de terminaison

Le client par lot utilise un fichier de configuration YAML qui spécifie les points de terminaison du conteneur local. L’exemple suivant peut être écrit dans `/mnt/my_nfs/config.yaml`, qui est utilisé dans les exemples ci-dessous. 

```yaml
MyContainer1: 
  concurrency: 5 
  host: 192.168.0.100 
  port: 5000 
  rtf: 3 
MyContainer2: 
  concurrency: 5 
  host: BatchVM0.corp.redmond.microsoft.com 
  port: 5000 
  rtf: 2 
MyContainer3: 
  concurrency: 10 
  host: localhost 
  port: 6001 
  rtf: 4 
```

Cet exemple de YAML spécifie trois conteneurs Speech sur trois hôtes. Le premier hôte est spécifié par une adresse IPv4, le second s’exécute sur la même machine virtuelle que le client par lot et le troisième conteneur est spécifié par le nom d’hôte DNS d’une autre machine virtuelle. La valeur `concurrency` spécifie le nombre maximal de transcriptions de fichiers simultanées pouvant s’exécuter sur le même conteneur. La valeur `rtf` (facteur en temps réel) est facultative et peut être utilisée pour régler les performances.
 
Le client par lot peut détecter de manière dynamique si un point de terminaison devient indisponible (par exemple en raison d’un redémarrage du conteneur ou d’un problème réseau) et lorsqu’il redevient disponible. Les requêtes de transcription ne sont pas envoyées aux conteneurs qui ne sont pas disponibles et le client continue à utiliser d’autres conteneurs disponibles. Vous pouvez ajouter, supprimer ou modifier des points de terminaison à tout moment sans interrompre la progression de votre lot.


## <a name="run-the-batch-processing-container"></a>Exécuter le conteneur de traitement par lots
  
> [!NOTE] 
> * Cet exemple utilise le même répertoire (`/my_nfs`) pour le fichier de configuration et les répertoires d’entrées, de sorties et de journaux. Vous pouvez utiliser des répertoires hébergés ou montés sur NFS pour ces dossiers.
> * L’exécution du client avec `–h` répertorie les paramètres de ligne de commande disponibles et leurs valeurs par défaut. 
> * Le conteneur de traitement par lots est uniquement pris en charge sur Linux.

Utilisez la commande `run` de Docker pour exécuter le conteneur. Cela démarre un interpréteur de commandes interactif à l’intérieur du conteneur.

```Docker
docker run --rm -ti -v  /mnt/my_nfs:/my_nfs --entrypoint /bin/bash /mnt/my_nfs:/my_nfs docker.io/batchkit/speech-batch-kit:latest
```

Pour exécuter le client par lot :  

```Docker
run-batch-client -config /my_nfs/config.yaml -input_folder /my_nfs/audio_files -output_folder /my_nfs/transcriptions -log_folder  /my_nfs/logs -log_level DEBUG -nbest 1 -m ONESHOT -diarization  None -language en-US -strict_config   
```

Pour exécuter le client et le conteneur par lot dans une commande unique :

```Docker
docker run --rm -ti -v  /mnt/my_nfs:/my_nfs docker.io/batchkit/speech-batch-kit:latest  -config /my_nfs/config.yaml -input_folder /my_nfs/audio_files -output_folder /my_nfs/transcriptions -log_folder  /my_nfs/logs -log_level DEBUG -nbest 1 -m ONESHOT -diarization  None -language en-US -strict_config   
```


Le client commence à être exécuté. Si un fichier audio a déjà été transcrit dans une exécution précédente, le client ignore automatiquement le fichier. Les fichiers sont envoyés avec une nouvelle tentative automatique si des erreurs temporaires se produisent et vous pouvez différencier les erreurs pour lesquelles le client peut procéder à une nouvelle tentative et les autres. Dans le cas d’une erreur de transcription, le client continue la transcription et peut procéder à une nouvelle tentative sans perdre la progression.  

## <a name="run-modes"></a>Exécuter des modes 

Le kit de traitement par lots propose trois modes, à l’aide du paramètre `--run-mode`.

#### <a name="oneshot"></a>[Oneshot](#tab/oneshot)

Le mode `ONESHOT` retranscrit un seul lot de fichiers audio (à partir d’un répertoire d’entrée et d’une liste de fichiers facultative) dans un dossier de sortie.

:::image type="content" source="media/containers/batch-oneshot-mode.png" alt-text="Diagramme montrant les fichiers de traitement de conteneur de kit par lot en mode OneShot.":::

1. Définissez les points de terminaison de conteneur Speech que le client par lot utilisera dans le fichier `config.yaml`. 
2. Placez les fichiers audio pour transcription dans un répertoire d’entrée.  
3. Appelez le conteneur sur le répertoire, qui commencera le traitement des fichiers. Si le fichier audio a déjà été retranscrit dans une exécution précédente avec le même répertoire de sortie (même nom de fichier et somme de contrôle), le client ignore le fichier. 
4. Les fichiers sont distribués aux points de terminaison de conteneur à partir de l’étape 1.
5. Les journaux et la sortie du conteneur Speech sont retournés au répertoire de sortie spécifié. 

#### <a name="daemon"></a>[Démon](#tab/daemon)

> [!TIP]
> Si plusieurs fichiers sont ajoutés au répertoire d’entrée en même temps, vous pouvez améliorer les performances en les ajoutant à un intervalle régulier.

Le mode `DAEMON` configure les fichiers existants dans un dossier donné et retranscrit en permanence les nouveaux fichiers audio à mesure qu’ils sont ajoutés.          

:::image type="content" source="media/containers/batch-daemon-mode.png" alt-text="Diagramme montrant les fichiers de traitement de conteneur de kit par lot en mode Démon.":::

1. Définissez les points de terminaison de conteneur Speech que le client par lot utilisera dans le fichier `config.yaml`. 
2. Appelez le conteneur sur un répertoire d’entrée. Le client par lot commence à analyser le répertoire pour les fichiers entrants. 
3. Configurez la livraison continue des fichiers audio dans le répertoire d’entrée. Si le fichier audio a déjà été retranscrit dans une exécution précédente avec le même répertoire de sortie (même nom de fichier et somme de contrôle), le client ignore le fichier. 
4. Une fois qu’un fichier d’écriture ou de signal POSIX est détecté, le conteneur est déclenché pour répondre.
5. Les fichiers sont distribués aux points de terminaison de conteneur à partir de l’étape 1.
6. Les journaux et la sortie du conteneur Speech sont retournés au répertoire de sortie spécifié. 

#### <a name="rest"></a>[REST](#tab/rest)

Le mode `REST` est un mode de serveur d’API qui fournit un ensemble de base de points de terminaison HTTP pour l’envoi par lots de fichiers audio, la vérification de l’état et l’interrogation longue. Active également la consommation de programmatique à l’aide d’une extension de module Python ou l’importation en tant que sous-module.

:::image type="content" source="media/containers/batch-rest-api-mode.png" alt-text="Diagramme montrant les fichiers de traitement de conteneur de kit par lot en mode REST.":::

1. Définissez les points de terminaison de conteneur Speech que le client par lot utilisera dans le fichier `config.yaml`. 
2. Envoyer une requête de requête HTTP à l’un des points de terminaison du serveur d’API. 
        
    |Point de terminaison  |Description  |
    |---------|---------|
    |`/submit`     | Point de terminaison pour la création de nouvelles requêtes de lots.        |
    |`/status`     | Point de terminaison pour vérifier l’état d’une requête de lot. La connexion reste ouverte jusqu’à ce que le traitement soit terminé.       |
    |`/watch`     | Point de terminaison pour l’utilisation de l’interrogation HTTP longue jusqu’à la fin du traitement.        |

3. Les fichiers audio sont téléchargés à partir du répertoire d’entrée. Si le fichier audio a déjà été retranscrit dans une exécution précédente avec le même répertoire de sortie (même nom de fichier et somme de contrôle), le client ignore le fichier. 
4. Les fichiers sont distribués aux points de terminaison de conteneur à partir de l’étape 1 si une requête est envoyée au point de terminaison `/submit`.
5. Les journaux et la sortie du conteneur Speech sont retournés au répertoire de sortie spécifié. 

---

## <a name="logging"></a>Journalisation

> [!NOTE]
> Le client par lot peut remplacer périodiquement le fichier *run.log* s’il est trop volumineux.

Le client crée un fichier *run.log* dans le répertoire spécifié par l’argument `-log_folder` dans la commande `run` du Docker. Les journaux sont capturés au niveau DEBUG par défaut. Les mêmes journaux sont envoyés au `stdout/stderr` et filtrés en fonction de l’argument `-log_level`. Ce journal est uniquement nécessaire pour le débogage ou si vous devez envoyer une trace pour le support. Le dossier de journalisation contient également les journaux du kit de développement logiciel (SDK) Speech pour chaque fichier audio.

Le répertoire de sortie spécifié par `-output_folder` contient un fichier *run_summary.json* , régulièrement réécrit toutes les 30 secondes ou chaque fois que de nouvelles transcriptions sont terminées. Vous pouvez utiliser ce fichier pour vérifier la progression de l’exécution du lot. Elle contient également les statistiques d’exécution finale et l’état final de chaque fichier lorsque le lot est terminé. Le traitement est terminé lorsque le processus a une sortie normale. 

## <a name="next-steps"></a>Étapes suivantes

* [Guide pratique pour installer et exécuter des conteneurs](speech-container-howto.md)
