---
title: À propos des dépôts et des images
description: Introduction aux concepts clés des registres de conteneurs Azure, des référentiels et des images conteneur.
ms.topic: article
ms.date: 06/16/2020
ms.openlocfilehash: cd2f93c119817c722401f7290064894f3d39dac9
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335892"
---
# <a name="about-registries-repositories-and-images"></a>À propos des registres, des dépôts et des images

Cet article présente les concepts clés des registres de conteneurs, des référentiels, des images conteneur et les artefacts associés. 

## <a name="registry"></a>Registre

Un *registre* de conteneurs est un service qui stocke et distribue des images conteneur. Docker Hub est un registre de conteneur public qui prend en charge de la communauté open source et sert de catalogue général d’images. Azure Container Registry fournit aux utilisateurs un contrôle direct de leurs images, avec l’authentification intégrée, la[géo-réplication](container-registry-geo-replication.md) prenant en charge la distribution mondiale et la fiabilité pour les déploiements proches du réseau, la [configuration du pare-feu et de réseau virtuel](container-registry-vnet.md), le [verrouillage des étiquettes](container-registry-image-lock.md), et de nombreuses autres fonctionnalités améliorées. 

En plus des images conteneur Docker, Azure Container Registry prend en charger les[artefacts de contenu](container-registry-image-formats.md) connexes, y compris les formats d’image Open Container Initiative (OCI).

## <a name="content-addressable-elements-of-an-artifact"></a>Éléments adressables de contenu d’un artefact

L’adressage d’un artefact dans un registre de conteneurs Azure inclut les éléments suivants. 

`[loginUrl]/[repository:][tag]`

* **loginUrl** - nom complet de l’hôte de registre. L’hôte de registre dans un registre de conteneurs Azure est au format *myregistry*. azurecr.io (tout en minuscules). Vous devez spécifier le loginUrl lors de l’utilisation de Docker ou d’autres outils de client pour les artefacts de tirage (pull) ou d’envoi (push) à un registre de conteneurs Azure. 
* **référentiel**  : nom d’un regroupement logique d’une ou de plusieurs images ou artefacts associés, par exemple les images d’une application ou d’un système d’exploitation de base. Peut inclure un chemin d’accès d’ *espace de noms*. 
* **étiquette**  : identificateur d’une version spécifique d’une image ou d’un artefact stocké dans un référentiel.

Par exemple, le nom complet d’une image dans un registre de conteneurs Azure peut se présenter comme :

*myregistry.azurecr.io/marketing/campaign10-18/email-sender:v2*

Pour plus d’informations sur ces éléments, consultez les sections suivantes.

## <a name="repository-name"></a>Nom du dépôt

Un *référentiel* est une collection d’images conteneur ou d’autres artefacts ayant le même nom, mais des étiquettes différentes. Par exemple, les trois images suivantes se trouvent dans le référentiel « acr-helloworld » :


- *acr-helloworld:latest*
- *acr-helloworld:v1*
- *acr-helloworld:v2*

Les noms des référentiels peuvent également comprendre des [espaces de noms](container-registry-best-practices.md#repository-namespaces). Les espaces de noms vous permettent d’identifier des référentiels associés et la propriété d’artefact dans votre organisation en utilisant des noms de délimités par des barres obliques. Toutefois, le registre gère tous les référentiels indépendamment, et non en tant que hiérarchie. Par exemple :

- *marketing/campaign10-18/web:v2*
- *marketing/campaign10-18/api:v3*
- *marketing/campaign10-18/email-sender:v2*
- *product-returns/web-submission:20180604*
- *product-returns/legacy-integrator:20180715*

Les noms de référentiel ne peuvent inclure que des caractères alphanumériques en minuscules, des points, des tirets, des tirets du bas et des barres obliques. 

Pour connaître les règles complètes de nommage de référentiel, consultez la [spécification de distribution Open Container Initiative](https://github.com/docker/distribution/blob/master/docs/spec/api.md#overview).

## <a name="image"></a>Image

Une image conteneur (ou un autre artefact qui se trouve dans le registre) est associée à une ou plusieurs étiquettes, est constituée d’un ou plusieurs calques, et est identifiable par son manifeste. Comprendre comment ces composants sont liés entre eux peut vous aider à gérer de manière efficace votre registre.

### <a name="tag"></a>Tag

L’ *étiquette* pour une image ou autre artefact spécifie sa version. Dans un référentiel, un artefact peut être associé à une ou plusieurs étiquettes, mais il peut aussi n’être associée à aucune étiquette. Autrement dit, vous pouvez supprimer toutes les étiquettes d'une image tout en conservant les données de cette image (ses calques) dans le registre.

Le nom d’une image est défini par le référentiel (ou le référentiel et l’espace de noms) et l’étiquette. Vous pouvez envoyer (push) et tirer (pull) une image en spécifiant son nom dans l’opération push ou pull. L’étiquette `latest` est utilisée par défaut si vous n’en fournissez pas une dans vos commandes Docker.

La façon dont vous étiquettez les images conteneur est guidée par vos scénarios pour les développer ou les déployer. Par exemple, les étiquettes stables sont recommandées pour gérer vos images de base et les étiquettes uniques pour le déploiement d’images. Pour plus d’informations, consultez les [Recommandations pour les images conteneur de balisage et le contrôle de version](container-registry-image-tag-version.md).

Pour connaître les règles de nommage d’étiquette, consultez la [documentation Docker](https://docs.docker.com/engine/reference/commandline/tag/).

### <a name="layer"></a>Couche

Les images conteneur sont constituées d’un ou plusieurs *calques* , correspondant chacun à une ligne dans le fichier Dockerfile qui définit l’image. Les images d’un registre ont des calques en commun, dans le but d’accroître l’efficacité du stockage. Par exemple, des images situées dans des référentiels différents peuvent partager le même calque de base Alpine Linux, qui n’est stocké qu’une seule fois dans le registre.

Le partage des calques optimise également la distribution des calques vers les nœuds qui contiennent plusieurs images ayant des calques en commun. Par exemple, si une image déjà présente dans un nœud a comme base un calque Alpine Linux, le prochain tirage (pull) d’une autre image référençant le même calque ne va pas transférer le calque vers le nœud. Au lieu de cela, il va référencer le calque déjà présent dans le nœud.

Pour fournir une isolation sécurisée et une protection contre la manipulation de calques potentiels, les calques ne sont pas partagés entre les registres.

### <a name="manifest"></a>Manifeste

Chaque image conteneur (ou artefact) envoyée (push) à un registre de conteneurs est associée à un *manifeste*. Le manifeste, qui est généré par le registre lorsque l’image est envoyée, identifie l’image de façon unique et spécifie ses calques. Vous pouvez répertorier les manifestes d’un référentiel avec la commande Azure CLI [az acr repository show-manifests][az-acr-repository-show-manifests] :

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName>
```

Par exemple, voici la liste des manifestes pour le référentiel « acr-helloworld » :

```azurecli
az acr repository show-manifests --name myregistry --repository acr-helloworld
```

```output
[
  {
    "digest": "sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108",
    "tags": [
      "latest",
      "v3"
    ],
    "timestamp": "2018-07-12T15:52:00.2075864Z"
  },
  {
    "digest": "sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57",
    "tags": [
      "v2"
    ],
    "timestamp": "2018-07-12T15:50:53.5372468Z"
  },
  {
    "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
    "tags": [
      "v1"
    ],
    "timestamp": "2018-07-11T21:38:35.9170967Z"
  }
]
```

### <a name="manifest-digest"></a>Code de hachage de manifeste

Les manifestes sont identifiables par un code de hachage SHA-256 unique, appelé *code de hachage de manifeste*. Chaque image (ou artefact), qu’elle soit ou non associée à une étiquette, est identifiable par son code de hachage. La valeur du code de hachage est unique, même si les données de calque de l’image sont identiques à celles d’une autre image. Ce mécanisme permet d’envoyer (push) à plusieurs reprises des images ayant la même étiquette vers le registre. Par exemple, vous pouvez envoyer `myimage:latest` à plusieurs reprises vers votre registre sans vous tromper, car chaque image est identifiable par son propre code de hachage.

Vous pouvez tirer (pull) une image à partir du registre en spécifiant son code de hachage dans l’opération de tirage. Certains systèmes peuvent être configurés pour tirer en fonction du code de hachage, car celui-ci garantit la version de l’image tirée, même si une image avec la même étiquette est ensuite envoyée (push) vers le registre.

Voici un exemple de tirage d’une image à partir du référentiel « acr-helloworld » en fonction du code de hachage du manifeste :

`docker pull myregistry.azurecr.io/acr-helloworld@sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108`

> [!IMPORTANT]
> Si vous envoyez plusieurs fois des images modifiées ayant une même étiquette, vous risquez de créer des images orphelines (c’est-à-dire sans étiquette), mais qui occupent toujours de l’espace dans votre registre. Lorsque vous répertoriez les images par étiquette, les images sans étiquette ne s’affichent pas dans Azure CLI ni dans le portail Azure. Toutefois, leurs calques se trouvent toujours dans le registre et consomment de l’espace de stockage. La suppression d’une image sans balise libère de l’espace dans le Registre lorsque le manifeste est le seul ou le dernier à pointer vers une couche particulière. Pour plus d’informations sur la libération d’espace utilisé par les images non balisées, consultez [Supprimer des images conteneur dans Azure Container Registry](container-registry-delete.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [stockage d’images](container-registry-storage.md) et la [prise en charge des formats de contenu](container-registry-image-formats.md) dans Azure Container Registry.

<!-- LINKS - Internal -->
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests


