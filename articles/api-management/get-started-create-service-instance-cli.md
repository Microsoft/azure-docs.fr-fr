---
title: Démarrage rapide - Créer une instance Gestion des API Azure avec l’interface CLI (préversion)
description: Créez une instance Gestion des API Azure en utilisant l’interface de ligne de commande Azure.
author: dlepow
ms.service: api-management
ms.topic: quickstart
ms.custom: ''
ms.date: 09/10/2020
ms.author: apimpm
ms.openlocfilehash: 863cb6cd10664e14966898045cb2b1861ef75b9f
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94887448"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-by-using-the-azure-cli-preview"></a>Démarrage rapide : Créer une instance Gestion des API Azure avec l’interface de ligne de commande Azure (préversion)

Le service Gestion des API Azure (API Management, APIM) permet aux organisations de publier des API pour des développeurs externes, partenaires et internes afin de libérer le potentiel de leurs données et de leurs services. Il offre les compétences essentielles qui garantissent un programme d’API réussi au travers de l’engagement des développeurs, des perspectives commerciales, de l’analytique, de la sécurité et de la protection. APIM vous permet de créer et de gérer des passerelles d’API modernes pour des services back-end existants, où qu’ils soient hébergés. Pour plus d'informations, consultez la [Vue d’ensemble](api-management-key-concepts.md).

Ce guide de démarrage rapide décrit les étapes permettant de créer une instance Gestion des API avec les commandes [az apim](/cli/azure/apim) dans Azure CLI. Les commandes du groupe de commandes `az apim` sont disponibles en préversion et peuvent être changées ou supprimées dans une version ultérieure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.11.1 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Les instances du service Gestion des API Azure, comme toutes les ressources Azure, doivent être déployées dans un groupe de ressources. Les groupes de ressources vous permettent d’organiser et de gérer les ressources Azure connexes.

Commencez par créer un groupe de ressources nommé *myResourceGroup* dans la localisation USA Centre à l’aide de la commande [az group create](/cli/azure/group#az-group-create) suivante :

```azurecli-interactive
az group create --name myResourceGroup --location centralus
```

## <a name="create-a-new-service"></a>Créer un service

Maintenant que vous avez un groupe de ressources, vous pouvez créer une instance Gestion des API. Créez-en une à l’aide de la commande [az apim create](/cli/azure/apim#az-apim-create) et fournissez un nom de service et les détails de l’éditeur. Le nom de service doit être unique au sein d’Azure. 

Dans l’exemple suivant, *myapim* est utilisé comme nom de service. Mettez à jour le nom de façon à utiliser une valeur unique. Mettez également à jour le nom de l’organisation de l’éditeur de l’API et l’adresse e-mail pour la réception des notifications. 

```azurecli-interactive
az apim create --name myapim --resource-group myResourceGroup \
  --publisher-name Contoso --publisher-email admin@contoso.com \
  --no-wait
```

Par défaut, la commande crée l’instance au niveau Développeur, ce qui constitue une option économique pour évaluer la Gestion des API Azure. Ce niveau n’est pas destiné à la production. Pour en savoir plus sur la mise à l’échelle des niveaux du service Gestion des API, consultez [Mettre à niveau et mettre à l’échelle](upgrade-and-scale.md). 

> [!TIP]
> La création et l’activation d’un service Gestion des API à ce niveau peuvent prendre entre 30 et 40 minutes. La commande précédente utilise l’option `--no-wait` pour rendre la main immédiatement pendant la création du service.

Vérifiez l’état du déploiement en exécutant la commande [az apim show](/cli/azure/apim#az-apim-show) :

```azurecli-interactive
az apim show --name myapim --resource-group myResourceGroup --output table
```

Au départ, la sortie ressemble à ce qui suit, indiquant l’état `Activating` :

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR    PUBLIC IP    PRIVATE IP    STATUS      TIER       UNITS
-----------  ----------------  ----------  --------------  -----------  ------------  ----------  ---------  -------
myapim       myResourceGroup   Central US                                             Activating  Developer  1
```

Après l’activation, l’état est `Online` et l’instance de service a une adresse de passerelle et une adresse IP publique. Pour le moment, ces adresses n’exposent pas de contenu. Par exemple, .

```console
NAME         RESOURCE GROUP    LOCATION    GATEWAY ADDR                       PUBLIC IP     PRIVATE IP    STATUS    TIER       UNITS
-----------  ----------------  ----------  ---------------------------------  ------------  ------------  --------  ---------  -------
myapim       myResourceGroup   Central US  https://myapim.azure-api.net       203.0.113.1                 Online    Developer  1
```

Une fois votre instance Gestion des API en ligne, vous êtes prêt à l’utiliser. Commencez par le tutoriel [Importer et publier votre première API](import-and-publish.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources et l’instance Gestion des API.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Importer et publier votre première API](import-and-publish.md)
