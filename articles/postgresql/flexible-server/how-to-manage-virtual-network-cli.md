---
title: Gérer des réseaux virtuels - Azure CLI - Azure Database pour PostgreSQL - Serveur flexible
description: Créer et gérer des réseaux virtuels pour le serveur flexible Azure Database pour PostgreSQL à l'aide du portail d’Azure CLI
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 830a97db562820853efcd88b1ab8c0b729a5dc9a
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490133"
---
# <a name="create-and-manage-virtual-networks-for-azure-database-for-postgresql---flexible-server-using-the-azure-cli"></a>Créer et gérer des réseaux virtuels pour le serveur flexible Azure Database pour PostgreSQL à l'aide du portail d’Azure CLI

> [!IMPORTANT]
> Le serveur flexible Azure Database pour PostgreSQL est en préversion

Azure Database pour PostgreSQL - Serveur flexible prend en charge deux types de méthodes de connectivité réseau mutuellement exclusives pour se connecter à votre serveur flexible. Les deux options sont les suivantes :

* Accès public (adresses IP autorisées)
* Accès privé (intégration au réseau virtuel)

Dans cet article, nous allons nous concentrer sur la création d’un serveur PostgreSQL avec **accès privé (intégration au réseau virtuel)** avec Azure CLI. Avec l’option *Accès privé (intégration au réseau virtuel)* , vous pouvez déployer votre serveur flexible sur votre propre [réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md). Les réseaux virtuels Azure offrent des communications réseau privées et sécurisées. Dans un accès privé, les connexions au serveur PostgreSQL sont limitées uniquement à votre réseau virtuel. Pour en savoir plus à ce sujet, reportez-vous à [Accès privé (intégration au réseau virtuel)](./concepts-networking.md#private-access-vnet-integration).

Dans le serveur flexible Azure Database pour PostgreSQL, vous pouvez uniquement déployer le serveur sur un réseau virtuel et un sous-réseau lors de la création du serveur. Une fois le serveur flexible déployé sur un réseau virtuel et un sous-réseau, vous ne pouvez pas le déplacer vers un autre réseau virtuel ou sous-réseau, ni vers l’ *accès public (adresses IP autorisées)* .

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également ouvrir Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell et sélectionnez **Entrée** pour les exécuter.

Si vous préférez installer et utiliser l’interface de ligne de commande en local, ce guide de démarrage rapide nécessite au minimum la version 2.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Prérequis

Vous devrez vous connecter à votre compte avec la commande [az login](/cli/azure/reference-index#az-login). Notez la propriété **ID** , qui fait référence à l’ **ID d’abonnement** de votre compte Azure.

```azurecli-interactive
az login
```

Sélectionnez l’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account#az-account-set). Notez la valeur **ID** issue de la commande **az login**  ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour accéder à l’ensemble de votre abonnement, utilisez [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-azure-database-for-postgresql---flexible-server-using-cli"></a>Créer Azure Database pour PostgreSQL - Serveur flexible à l’aide de l’interface de ligne de commande
Vous pouvez utiliser la commande `az postgres flexible-server` pour créer le serveur flexible avec l’option *Accès privé (intégration au réseau virtuel)* . Cette commande utilise l’accès privé (intégration au réseau virtuel) comme méthode de connectivité par défaut. Un réseau virtuel et un sous-réseau sont automatiquement créés si aucun n’est fourni. Vous pouvez également fournir le réseau virtuel et le sous-réseau déjà existants à l’aide de l’ID de sous-réseau. <!-- You can provide the **vnet**,**subnet**,**vnet-address-prefix** or**subnet-address-prefix** to customize the virtual network and subnet.--> Il existe différentes options pour créer un serveur flexible à l’aide de l’interface CLI, comme indiqué dans les exemples ci-dessous.

>[!Important]
> L’utilisation de cette commande permet de déléguer le sous-réseau à **Microsoft.DBforPostgreSQL/flexibleServers** . Cette délégation spécifie que seuls les serveurs flexibles Azure Database pour PostgreSQL peuvent utiliser ce sous-réseau. Aucun autre type de ressource Azure ne peut se trouver dans le sous-réseau délégué.
>
Reportez-vous à la documentation de référence sur l’interface de ligne de commande Azure <!--FIXME --> pour obtenir la liste complète des paramètres d’interface de ligne de commande configurables. Par exemple, dans les commandes ci-dessous, vous pouvez éventuellement spécifier le groupe de ressources.

- Créer un serveur flexible à l’aide d’un réseau virtuel et d’un sous-réseau par défaut avec le préfixe d’adresse par défaut
    ```azurecli-interactive
    az postgres flexible-server create
    ```
<!--- Create a flexible server using already existing virtual network and subnet
    ```azurecli-interactive
    az postgres flexible-server create --vnet myVnet --subnet mySubnet
    ```-->
- Créer un serveur flexible à l’aide d’un réseau virtuel, d’un sous-réseau et de l’ID de sous-réseau déjà existants. Le sous-réseau fourni ne doit pas contenir d’autres ressources déjà déployées. Il est délégué à **Microsoft.DBforPostgreSQL/flexibleServers** , en l’absence de délégation existante.
    ```azurecli-interactive
    az postgres flexible-server create --subnet /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNetName}/subnets/{SubnetName}
    ```
    > [!Note]
    > Le réseau virtuel et le sous-réseau doivent se trouver dans la même région et le même abonnement que votre serveur flexible.
<!--  
- Create a flexible server using new virtual network, subnet with non-default address prefix
    ```azurecli-interactive
    az postgres flexible-server create --vnet myVnet --vnet-address-prefix 10.0.0.0/24 --subnet mySubnet --subnet-address-prefix 10.0.0.0/24
    ```-->
Reportez-vous à la documentation de référence sur l’interface de ligne de commande Azure <!--FIXME --> pour obtenir la liste complète de ses paramètres configurables.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez-en plus sur la [mise en réseau dans le serveur flexible Azure Database pour PostgreSQL](./concepts-networking.md).
- [Créer et gérer un réseau virtuel de serveur flexible Azure Database pour PostgreSQL à l’aide du portail Azure](./how-to-manage-virtual-network-portal.md).
- Apprenez-en plus sur le [réseau virtuel de serveur flexible Azure Database pour PostgreSQL](./concepts-networking.md#private-access-vnet-integration).