---
title: Supprimer des ressources d’image
description: Explique en détail comment gérer efficacement la taille du registre en supprimant les données d’image conteneur à l’aide de commandes d’interfaces de lignes de commande Azure.
ms.topic: article
ms.date: 07/31/2019
ms.openlocfilehash: 449a1c09bf88e3e0e0aeca4d3b687371d2a6b91a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78403343"
---
# <a name="delete-container-images-in-azure-container-registry-using-the-azure-cli"></a>Supprimer des images conteneur dans Azure Container Registry à l’aide de l’interface de ligne de commande Azure

Pour limiter la taille de votre registre de conteneurs Azure, vous devez supprimer régulièrement les données d’image obsolètes. Certaines images conteneur déployées dans l’environnement de production peuvent nécessiter un stockage à long terme, mais d’autres peuvent être supprimées plus rapidement. Par exemple, dans un scénario de génération et de tests automatisés, votre registre peut se remplir rapidement d’images qui peuvent ne jamais être déployées, et peuvent donc être vidées peu après la génération et la réussite des tests.

Il existe plusieurs façons de supprimer les données d’image, et il est important de comprendre comment chaque opération de suppression affecte l’utilisation du stockage. Cet article décrit plusieurs méthodes de suppression de données d’image :

* En supprimant un [référentiel](#delete-repository) : de cette façon, vous supprimez toutes les images et tous les calques contenus dans le référentiel.
* En supprimant en fonction de [l’étiquette](#delete-by-tag) : de cette façon, vous supprimez une image, son étiquette, tous les calques référencés par l’image et toutes les autres étiquettes associées à l’image.
* En supprimant en fonction du [code de hachage du manifeste](#delete-by-manifest-digest) : de cette façon, vous supprimez une image, tous les calques référencés par l’image et toutes les autres étiquettes associées à l’image.

Des exemples de scripts sont fournis pour automatiser les opérations de suppression.

Pour une présentation de ces concepts, consultez [À propos des registres, des dépôts et des images](container-registry-concepts.md).

## <a name="delete-repository"></a>Supprimer le référentiel

La suppression d’un référentiel supprime toutes les images qu’il contient, ainsi que leurs balises, calques propres et manifestes. Lorsque vous supprimez un référentiel, vous récupérez l’espace de stockage utilisé par les images qui font référence à des calques uniques qui s’y trouvaient.

La commande suivante d’Azure CLI supprime le référentiel « acr-helloworld », ainsi que toutes les étiquettes et tous les manifestes. Si les calques référencés par les manifestes supprimés ne sont pas référencés par d’autres images du registre, les données de calques sont elles aussi supprimées, permettant ainsi de récupérer de l’espace de stockage.

```azurecli
 az acr repository delete --name myregistry --repository acr-helloworld
```

## <a name="delete-by-tag"></a>Supprimer en fonction de l’étiquette

Vous pouvez supprimer des images d’un référentiel en spécifiant le nom du référentiel et l’étiquette dans l’opération de suppression. Lorsque vous supprimez une étiquette, vous récupérez l’espace de stockage utilisé par les calques propres à l’image (c’est-à-dire les calques qui ne sont partagés par aucune autre image du registre).

Pour supprimer l’étiquette, utilisez [az acr repository delete][az-acr-repository-delete] et spécifiez le nom de l’image dans le paramètre `--image`. Tous les calques propres à l’image, et toutes les autres étiquettes associées à l’image, sont supprimés.

Voici un exemple de suppression de l’image « acr-helloworld:latest » dans le registre « myregistry » :

```azurecli
az acr repository delete --name myregistry --image acr-helloworld:latest
```

```output
This operation will delete the manifest 'sha256:0a2e01852872580b2c2fea9380ff8d7b637d3928783c55beb3f21a6e58d5d108' and all the following images: 'acr-helloworld:latest', 'acr-helloworld:v3'.
Are you sure you want to continue? (y/n):
```

> [!TIP]
> La suppression *en fonction de l’étiquette* ne doit pas être confondue avec la suppression d’une étiquette. Vous pouvez supprimer une étiquette avec la commande [az acr repository untag][az-acr-repository-untag] d’Azure CLI. Aucun espace n'est libéré lorsque vous supprimez l'étiquette d'une image, car son [manifeste](container-registry-concepts.md#manifest) et ses données de calque sont conservés dans le registre. Seule la référence de l’étiquette est supprimée.

## <a name="delete-by-manifest-digest"></a>Supprimer en fonction du code de hachage du manifeste

Un [code de hachage de manifeste](container-registry-concepts.md#manifest-digest) peut être associé à une ou plusieurs étiquettes, ou à aucune. Lorsque vous supprimez en fonction du code de hachage, toutes les étiquettes référencées par le manifeste sont supprimées, comme le sont les données des calques propres à l’image. Les données de calques partagées ne sont pas supprimées.

Pour supprimer en fonction du code de hachage, commencez par répertorier les codes de hachage de manifeste du référentiel qui contient les images que vous souhaitez supprimer. Par exemple :

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
  }
]
```

Ensuite, spécifiez le code de hachage que vous souhaitez supprimer dans la commande [az acr repository delete][az-acr-repository-delete]. Le format de la commande est le suivant :

```azurecli
az acr repository delete --name <acrName> --image <repositoryName>@<digest>
```

Par exemple, pour supprimer le dernier manifeste répertorié dans la sortie précédente (avec l’étiquette « v2 ») :

```azurecli
az acr repository delete --name myregistry --image acr-helloworld@sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57
```

```output
This operation will delete the manifest 'sha256:3168a21b98836dda7eb7a846b3d735286e09a32b0aa2401773da518e7eba3b57' and all the following images: 'acr-helloworld:v2'.
Are you sure you want to continue? (y/n): 
```

L'image `acr-helloworld:v2` est supprimée du registre, comme le sont les données de calques propres à cette image. Si un manifeste est associé à plusieurs étiquettes, toutes les étiquettes associées sont également supprimées.

## <a name="delete-digests-by-timestamp"></a>Supprimer des codes de hachage par timestamp

Pour limiter la taille d'un référentiel ou d'un registre, vous devrez peut-être périodiquement supprimer les codes de hachage de manifeste antérieurs à une certaine date.

La commande Azure CLI suivante répertorie tous les codes de hachage de manifeste d'un référentiel antérieurs à un timestamp spécifié, par ordre croissant. Remplacez `<acrName>` et `<repositoryName>` par les valeurs adaptées à votre environnement. Le timestamp peut être une expression date-heure complète ou une date, comme dans cet exemple.

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName> \
--orderby time_asc -o tsv --query "[?timestamp < '2019-04-05'].[digest, timestamp]"
```

Après avoir identifié les codes de hachage de manifeste obsolètes, vous pouvez exécuter le script Bash suivant pour supprimer les codes de hachage de manifeste antérieurs à un timestamp spécifié. Ce script nécessite Azure CLI et **xargs**. Par défaut, le script n’effectue aucune suppression. Remplacez la valeur `ENABLE_DELETE` par `true` pour activer la suppression des images.

> [!WARNING]
> Utilisez l'exemple de script suivant avec précaution : les données d'image supprimées ne sont PAS RÉCUPÉRABLES. Si vous disposez de systèmes qui tirent (pull) les images en fonction du code de hachage du manifeste (et non en fonction du nom de l'image), vous ne devez pas exécuter ces scripts. La suppression des codes de hachage de manifeste empêchera ces systèmes de tirer (pull) les images à partir du registre. Au lieu de tirer en fonction du manifeste, envisagez d’adopter un schéma d’*étiquetage unique* (il s’agit là d’une [bonne pratique](container-registry-image-tag-version.md)). 

```bash
#!/bin/bash

# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to 'true' to enable image delete
ENABLE_DELETE=false

# Modify for your environment
# TIMESTAMP can be a date-time string such as 2019-03-15T17:55:00.
REGISTRY=myregistry
REPOSITORY=myrepository
TIMESTAMP=2019-04-05  

# Delete all images older than specified timestamp.

if [ "$ENABLE_DELETE" = true ]
then
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY \
    --orderby time_asc --query "[?timestamp < '$TIMESTAMP'].digest" -o tsv \
    | xargs -I% az acr repository delete --name $REGISTRY --image $REPOSITORY@% --yes
else
    echo "No data deleted."
    echo "Set ENABLE_DELETE=true to enable deletion of these images in $REPOSITORY:"
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY \
   --orderby time_asc --query "[?timestamp < '$TIMESTAMP'].[digest, timestamp]" -o tsv
fi
```

## <a name="delete-untagged-images"></a>Supprimer les images sans étiquette

Comme mentionné dans la section [Code de hachage de manifeste](container-registry-concepts.md#manifest-digest), l’envoi (push) d’une image modifiée à l’aide d’une étiquette existante entraîne la **suppression de l’étiquette** de l’image précédemment envoyée, ce qui a pour résultat une image orpheline (ou « non résolue »). L’image du manifeste précédemment envoyée (et ses données de calque) est conservée dans le registre. Regardez la séquence d’événements suivante :

1. Envoyer (push) l’image *acr-helloworld* avec l’étiquette **latest** : `docker push myregistry.azurecr.io/acr-helloworld:latest`
1. Vérifier les manifestes du référentiel *acr-helloworld* :

   ```azurecli
   az acr repository show-manifests --name myregistry --repository acr-helloworld
   
   ```
   
   ```output
   [
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

1. Modifier le Dockerfile de *acr-helloworld*
1. Envoyer (push) l’image *acr-helloworld* avec l’étiquette **latest** : `docker push myregistry.azurecr.io/acr-helloworld:latest`
1. Vérifier les manifestes du référentiel *acr-helloworld* :

   ```azurecli
   az acr repository show-manifests --name myregistry --repository acr-helloworld
   ```
   
   ```output
   [
     {
       "digest": "sha256:7ca0e0ae50c95155dbb0e380f37d7471e98d2232ed9e31eece9f9fb9078f2728",
       "tags": [
         "latest"
       ],
       "timestamp": "2018-07-11T21:38:35.9170967Z"
     },
     {
       "digest": "sha256:d2bdc0c22d78cde155f53b4092111d7e13fe28ebf87a945f94b19c248000ceec",
       "tags": [],
       "timestamp": "2018-07-11T21:32:21.1400513Z"
     }
   ]
   ```

Comme vous pouvez le voir dans la sortie de la dernière étape de la séquence, il existe maintenant un manifeste orphelin dont la propriété `"tags"` est une liste vide. Ce manifeste se trouve toujours dans le registre, ainsi que toutes les données des calques propres qu’il référence. **Pour supprimer ces images orphelines et leurs données de calque, vous devez supprimer en fonction du code de hachage du manifeste**.

## <a name="delete-all-untagged-images"></a>Supprimer toutes les images sans étiquette

Vous pouvez répertorier toutes les images sans étiquette de votre référentiel à l’aide de la commande Azure CLI suivante. Remplacez `<acrName>` et `<repositoryName>` par les valeurs adaptées à votre environnement.

```azurecli
az acr repository show-manifests --name <acrName> --repository <repositoryName> --query "[?tags[0]==null].digest"
```

L’utilisation de cette commande dans un script vous permet de supprimer toutes les images sans étiquette d’un référentiel.

> [!WARNING]
> Utilisez les exemples de scripts suivants avec prudence : la suppression des données d’image est irrécupérable. Si vous disposez de systèmes qui tirent (pull) les images en fonction du code de hachage du manifeste (et non en fonction du nom de l'image), vous ne devez pas exécuter ces scripts. La suppression des images sans étiquette va empêcher ces systèmes de tirer les images à partir du registre. Au lieu de tirer en fonction du manifeste, envisagez d’adopter un schéma d’*étiquetage unique* (il s’agit là d’une [bonne pratique](container-registry-image-tag-version.md)).

**Azure CLI dans Bash**

Le script Bash suivant supprime toutes les images sans étiquette d’un référentiel. Ce script nécessite Azure CLI et **xargs**. Par défaut, le script n’effectue aucune suppression. Remplacez la valeur `ENABLE_DELETE` par `true` pour activer la suppression des images.

```bash
#!/bin/bash

# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to 'true' to enable image delete
ENABLE_DELETE=false

# Modify for your environment
REGISTRY=myregistry
REPOSITORY=myrepository

# Delete all untagged (orphaned) images
if [ "$ENABLE_DELETE" = true ]
then
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY  --query "[?tags[0]==null].digest" -o tsv \
    | xargs -I% az acr repository delete --name $REGISTRY --image $REPOSITORY@% --yes
else
    echo "No data deleted."
    echo "Set ENABLE_DELETE=true to enable image deletion of these images in $REPOSITORY:"
    az acr repository show-manifests --name $REGISTRY --repository $REPOSITORY --query "[?tags[0]==null]" -o tsv
fi
```

**Azure CLI dans PowerShell**

Le script PowerShell suivant supprime toutes les images sans étiquette d’un référentiel. Il nécessite PowerShell et Azure CLI. Par défaut, le script n’effectue aucune suppression. Remplacez la valeur `$enableDelete` par `$TRUE` pour activer la suppression des images.

```powershell
# WARNING! This script deletes data!
# Run only if you do not have systems
# that pull images via manifest digest.

# Change to '$TRUE' to enable image delete
$enableDelete = $FALSE

# Modify for your environment
$registry = "myregistry"
$repository = "myrepository"

if ($enableDelete) {
    az acr repository show-manifests --name $registry --repository $repository --query "[?tags[0]==null].digest" -o tsv `
    | %{ az acr repository delete --name $registry --image $repository@$_ --yes }
} else {
    Write-Host "No data deleted."
    Write-Host "Set `$enableDelete = `$TRUE to enable image deletion."
    az acr repository show-manifests --name $registry --repository $repository --query "[?tags[0]==null]" -o tsv
}
```


## <a name="automatically-purge-tags-and-manifests-preview"></a>Vider automatiquement les balises et les manifestes (préversion)

Au lieu de scripts d’interfaces de lignes de commande Azure, exécutez une tâche ACR à la demande ou planifiée pour supprimer toutes les balises qui sont antérieures à une certaine durée ou correspondent à un filtre de nom spécifié. Pour plus d’informations, consultez [Purger automatiquement les images d’un registre de conteneurs Azure](container-registry-auto-purge.md).

Vous pouvez éventuellement définir une [stratégie de rétention](container-registry-retention-policy.md) pour chaque registre, afin de gérer les manifestes sans balise. Lorsque vous activez une stratégie de rétention, les fichiers manifestes d’image du registre qui n’ont aucune balise associée, ainsi que les données sous-jacentes, sont automatiquement supprimés après une période définie.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le stockage des images dans Azure Container Registry, consultez [Stockage des images conteneur dans Azure Container Registry](container-registry-storage.md).

<!-- IMAGES -->
[manifest-digest]: ./media/container-registry-delete/01-manifest-digest.png

<!-- LINKS - External -->
[docker-manifest-inspect]: https://docs.docker.com/edge/engine/reference/commandline/manifest/#manifest-inspect
[portal]: https://portal.azure.com

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[az-acr-repository-untag]: /cli/azure/acr/repository#az-acr-repository-untag
