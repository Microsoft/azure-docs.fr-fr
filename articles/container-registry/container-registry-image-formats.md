---
title: Formats de contenu pris en charge
description: Découvrez les formats de contenu pris en charge par Azure Container Registry, y compris les images conteneur compatibles avec Docker, les charts Helm, les images OCI et les artefacts OCI.
ms.topic: article
ms.date: 08/30/2019
ms.openlocfilehash: ab915385f46f83c7b655acd1a48d66df84b50653
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84695264"
---
# <a name="content-formats-supported-in-azure-container-registry"></a>Formats de contenu pris en charge dans Azure Container Registry

Utilisez un référentiel privé dans Azure Container Registry pour gérer l’un des formats de contenu suivants. 

## <a name="docker-compatible-container-images"></a>Images de conteneur compatibles avec Docker

Les formats d’image conteneur Docker suivants sont pris en charge :

* [Docker Image manifeste V2, schéma 1](https://docs.docker.com/registry/spec/manifest-v2-1/)

* [Docker Image manifeste V2, schéma 2](https://docs.docker.com/registry/spec/manifest-v2-2/) - inclut des listes de manifeste qui permettent aux registres de stocker des images multiplateformes sous une seule référence « image:tag »

## <a name="oci-images"></a>Images OCI

Azure Container Registry prend en charge les images qui répondent à la [spécification de format d’image OCI (Open Container Initiative)](https://github.com/opencontainers/image-spec/blob/master/spec.md). Les formats d’empaquetage incluent [SIF (Singularity Image Format)](https://github.com/sylabs/sif).

## <a name="oci-artifacts"></a>Artefacts OCI

Azure Container Registry prend en charge la [spécification de distribution OCI](https://github.com/opencontainers/distribution-spec), une spécification indépendante du Cloud et du fournisseur pour stocker, partager, sécuriser et déployer des images conteneur et d’autres types de contenu (artefacts). La spécification permet à un registre de stocker un large éventail d’artefacts, en plus des images conteneur. Vous utilisez les outils appropriés à l’artefact pour envoyer (push) et tirer (pull) des artefacts. Pour obtenir un exemple, consultez [Envoyer (push) et tirer (pull) un artefact OCI à l’aide d’un registre de conteneurs Azure](container-registry-oci-artifacts.md).

Pour en savoir plus sur les artefacts OCI, consultez le dépôt [OCI Registry as Storage (ORAS)](https://github.com/deislabs/oras) et le dépôt [OCI Artifacts](https://github.com/opencontainers/artifacts) sur GitHub.

## <a name="helm-charts"></a>Graphiques Helm

Azure Container Registry peut héberger des référentiels pour [graphiques Helm](https://helm.sh/), un format d’empaquetage utilisé pour gérer et déployer rapidement des applications pour Kubernetes. Le [client Helm](https://docs.helm.sh/using_helm/#installing-helm) version 2 (2.11.0 ou ultérieure) est recommandé.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [envoyer et récupérer](container-registry-get-started-docker-cli.md) des images avec Azure Container Registry.

* Utilisez les [tâches ACR](container-registry-tasks-overview.md) pour générer et tester des images de conteneur. 

* Utilisez le [Moby BuildKit](https://github.com/moby/buildkit) pour générer et empaqueter des conteneurs dans le format OCI.

* Configurez un [référentiel Helm](container-registry-helm-repos.md) hébergé dans Azure Container Registry. 


