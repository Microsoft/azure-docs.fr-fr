---
title: Configurer l’accès au registre public
description: Configurez des règles IP pour activer l’accès à un registre de conteneurs Azure à partir de certaines IP publiques ou plages d’adresses.
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: 4e4ee817e2534bcca09cba89daafd379ff3f03f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89488760"
---
# <a name="configure-public-ip-network-rules"></a>Configurer des règles de réseau IP public

Par défaut, un registre de conteneurs Azure accepte les connexions via Internet à partir d’hôtes sur n’importe quel réseau. Cet article explique comment configurer votre registre de conteneurs pour autoriser l’accès uniquement à partir d’IP publiques ou de plages d’adresses spécifiques. Les étapes équivalentes à l’aide de l’interface de ligne de commande Azure et du portail Azure sont fournies.

Les règles de réseau IP sont configurées sur le point de terminaison du registre public. Les règles de réseau IP ne s’appliquent pas aux points de terminaison privés configurés avec [Azure Private Link](container-registry-private-link.md).

La configuration des règles d’accès IP est disponible uniquement au niveau de service **Premium** de registre de conteneurs. Pour plus d’informations sur les niveaux de service et les limites de registre, consultez [Niveaux de service Azure Container Registry](container-registry-skus.md).

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="access-from-selected-public-network---cli"></a>Accès à partir du réseau public sélectionné (CLI)

### <a name="change-default-network-access-to-registry"></a>Changer l’accès réseau par défaut au registre

Pour limiter l’accès à un réseau public sélectionné, commencez par changer l’action par défaut de manière à refuser l’accès. Substituez le nom de votre registre dans la commande [az acr update][az-acr-update] suivante :

```azurecli
az acr update --name myContainerRegistry --default-action Deny
```

### <a name="add-network-rule-to-registry"></a>Ajouter une règle de réseau au registre

Utilisez la commande [az acr network-rule add][az-acr-network-rule-add] pour ajouter à votre registre une règle de réseau qui autorise l’accès à partir d’une IP publique ou d’une plage. Par exemple, substituez le nom du registre de conteneurs et l’IP publique d’une machine virtuelle dans un réseau virtuel.

```azurecli
az acr network-rule add \
  --name mycontainerregistry \
  --ip-address <public-IP-address>
```

> [!NOTE]
> Après l’ajout d’une règle, il faut quelques minutes pour que la règle prenne effet.

## <a name="access-from-selected-public-network---portal"></a>Accès à partir du réseau public sélectionné (Portail)

1. Dans le portail, accédez à votre registre de conteneurs.
1. Sous **Paramètres**, sélectionnez **Mise en réseau**.
1. Sous l’onglet **Accès public**, sélectionnez l’option permettant l’accès public à partir des **réseaux sélectionnés**.
1. Sous **Pare-feu**, entrez une IP publique, telle que l’IP publique d’une machine virtuelle dans un réseau virtuel. Ou bien, entrez en notation CIDR une plage d’adresses qui contient l’adresse IP de la machine virtuelle.
1. Sélectionnez **Enregistrer**.

![Configurer la règle de pare-feu pour le registre de conteneurs][acr-access-selected-networks]

> [!NOTE]
> Après l’ajout d’une règle, il faut quelques minutes pour que la règle prenne effet.

> [!TIP]
> Activez éventuellement l’accès au registre à partir d’un ordinateur client local ou d’une plage d’adresses IP. Pour autoriser cet accès, vous avez besoin de l’adresse IPv4 publique de l’ordinateur. Vous pouvez trouver cette adresse en effectuant une recherche sur « quelle est mon adresse IP » dans un navigateur Internet. L’adresse IPv4 actuelle du client apparaît également de façon automatique lorsque vous configurez les paramètres de pare-feu sur la page **Mise en réseau** du portail.

## <a name="disable-public-network-access"></a>Désactiver l’accès au réseau public

Éventuellement, désactivez le point de terminaison public sur le registre. La désactivation du point de terminaison public écrase toutes les configurations de pare-feu. Par exemple, vous souhaiterez peut-être désactiver l’accès public à un registre sécurisé dans un réseau virtuel à l’aide d’[Azure Private Link](container-registry-private-link.md).

> [!NOTE]
> Si le registre est configuré dans un réseau virtuel avec un [point de terminaison de service](container-registry-vnet.md), la désactivation de l'accès au point de terminaison public du registre désactive également l'accès au registre au sein du réseau virtuel.

### <a name="disable-public-access---cli"></a>Désactiver l’accès public (CLI)

Pour désactiver l’accès public à l’aide d’Azure CLI, exécutez [az acr update][az-acr-update] et définissez `--public-network-enabled` avec `false`. L’argument `public-network-enabled` nécessite Azure CLI 2.6.0 ou ultérieur. 

```azurecli
az acr update --name myContainerRegistry --public-network-enabled false
```

### <a name="disable-public-access---portal"></a>Désactiver l’accès public (portail)

1. Dans le portail, accédez à votre registre de conteneurs et sélectionnez **Paramètres > Mise en réseau**.
1. Sous l’onglet **Accès public**, dans **Autoriser l’accès au réseau public**, sélectionnez **Désactivé**. Ensuite, sélectionnez **Enregistrer**.

![Désactiver l’accès public][acr-access-disabled]


## <a name="restore-public-network-access"></a>Restaurer l’accès au réseau public

Pour réactiver le point de terminaison public, mettez à jour les paramètres de mise en réseau pour autoriser l’accès public. L’activation du point de terminaison public écrase toutes les configurations de pare-feu. 

### <a name="restore-public-access---cli"></a>Restaurer l’accès public (CLI)

Exécutez [az acr update][az-acr-update] et définissez `--public-network-enabled` sur `true`. 

> [!NOTE]
> L’argument `public-network-enabled` requiert Azure CLI 2.6.0 ou une version ultérieure. 

```azurecli
az acr update --name myContainerRegistry --public-network-enabled true
```

### <a name="restore-public-access---portal"></a>Restaurer l’accès public (Portail)

1. Dans le portail, accédez à votre registre de conteneurs et sélectionnez **Paramètres > Mise en réseau**.
1. Sous l’onglet **Accès public**, dans **Autoriser l’accès au réseau public**, sélectionnez **Tous les réseaux**. Ensuite, sélectionnez **Enregistrer**.

![Accès public à partir de tous les réseaux][acr-access-all-networks]

## <a name="troubleshoot"></a>Dépanner

Si une règle de réseau public est définie ou si l’accès public au registre est refusé, les tentatives de connexion au registre à partir d’un réseau public non autorisé échouent. L’accès client depuis derrière un proxy HTTPS échoue également si aucune règle d’accès n’est définie pour le proxy. Un message d'erreur semblable à `Error response from daemon: login attempt failed with status: 403 Forbidden` ou `Looks like you don't have access to registry` s'affiche.

Ces erreurs peuvent également se produire si vous utilisez un proxy HTTPS qui est autorisé par une règle d’accès réseau, mais que le proxy n’est pas correctement configuré dans l’environnement client. Vérifiez que votre client Docker et le démon Docker sont configurés pour le comportement du proxy. Pour plus d’informations, consultez [proxy HTTP/HTTPS](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) dans la documentation Docker.


## <a name="next-steps"></a>Étapes suivantes

* Pour restreindre l’accès à un registre à l’aide d’un point de terminaison privé dans un réseau virtuel, consultez [Configurer Azure Private Link pour un registre de conteneurs Azure](container-registry-private-link.md).
* Si vous devez configurer des règles d’accès au registre derrière un pare-feu client, consultez [Configurer des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu](container-registry-firewall-access-rules.md).

[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-network-rule-add]: /cli/azure/acr/network-rule/#az-acr-network-rule-add
[az-acr-network-rule-remove]: /cli/azure/acr/network-rule/#az-acr-network-rule-remove
[az-acr-network-rule-list]: /cli/azure/acr/network-rule/#az-acr-network-rule-list
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-update]: /cli/azure/acr#az-acr-update
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com

[acr-access-selected-networks]: ./media/container-registry-access-selected-networks/acr-access-selected-networks.png
[acr-access-disabled]: ./media/container-registry-access-selected-networks/acr-access-disabled.png
[acr-access-all-networks]: ./media/container-registry-access-selected-networks/acr-access-all-networks.png
