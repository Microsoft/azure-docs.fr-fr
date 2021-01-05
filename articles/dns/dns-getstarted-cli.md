---
title: 'Démarrage rapide : Créer un enregistrement et une zone Azure DNS – Azure CLI'
titleSuffix: Azure DNS
description: 'Démarrage rapide : découvrez comment créer une zone et un enregistrement DNS dans Azure DNS. Il s’agit d’un guide pas à pas pour la création et la gestion de votre première zone et de votre premier enregistrement DNS à l’aide d’Azure CLI.'
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1929cd512d18d7fd234aff1f55814c423455e63b
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94561367"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-cli"></a>Démarrage rapide : Créer un enregistrement et une zone Azure DNS avec Azure CLI

Cet article vous guide tout au long de la procédure de création de votre première zone et de votre premier enregistrement DNS à l’aide de l’interface de ligne de commande Azure, qui est disponible pour Windows, Mac et Linux. Vous pouvez également effectuer ces étapes à l’aide du [portail Azure](dns-getstarted-portal.md) ou [d’Azure PowerShell](dns-getstarted-powershell.md).

Une zone DNS permet d’héberger les enregistrements DNS d’un domaine particulier. Pour commencer à héberger votre domaine dans le DNS Azure, vous devez créer une zone DNS pour ce nom de domaine. Chaque enregistrement DNS pour votre domaine est ensuite créé à l’intérieur de cette zone DNS. Enfin, pour publier votre zone DNS sur Internet, vous devez configurer les serveurs de noms du domaine. Chacune de ces étapes est décrite ci-dessous.

Azure DNS prend également en charge les zones DNS privées. Pour en savoir plus sur les zones DNS privées, consultez la session relative à [l’utilisation d’Azure DNS pour les domaines privés](private-dns-overview.md). Vous pouvez trouver un exemple de création d’une zone DNS privée sur la page [Créer une zone privée Azure DNS avec Azure CLI](./private-dns-getstarted-cli.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.4 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Avant de créer la zone DNS, créez un groupe de ressources pour contenir la zone DNS :

```azurecli
az group create --name MyResourceGroup --location "East US"
```

## <a name="create-a-dns-zone"></a>Création d’une zone DNS

Une zone DNS est créée à l'aide de la commande `az network dns zone create`. Pour consulter l’aide de cette commande, tapez `az network dns zone create -h`.

L’exemple ci-dessous crée une zone DNS appelée *contoso.xyz* dans le groupe de ressources *MyResourceGroup*. Utilisez l’exemple pour créer une zone DNS, en remplaçant les valeurs indiquées par vos propres valeurs.

```azurecli
az network dns zone create -g MyResourceGroup -n contoso.xyz
```

## <a name="create-a-dns-record"></a>Créer un enregistrement DNS

Pour créer un enregistrement DNS, utilisez la commande `az network dns record-set [record type] add-record`. Pour obtenir de l’aide sur les enregistrements A, consultez `azure network dns record-set A add-record -h`.

L’exemple suivant crée un enregistrement avec le nom relatif « www » dans la zone DNS « contoso.xyz », dans le groupe de ressources « MyResourceGroup ». Le nom complet du jeu d’enregistrements est « www.contoso.xzy ». Le type d’enregistrement est « A », avec l’adresse IP « 10.10.10.10 » et une durée de vie (TTL) par défaut de 3 600 secondes (1 heure).

```azurecli
az network dns record-set a add-record -g MyResourceGroup -z contoso.xyz -n www -a 10.10.10.10
```

## <a name="view-records"></a>Affichage des enregistrements

Pour répertorier les enregistrements DNS dans votre zone, utilisez :

```azurecli
az network dns record-set list -g MyResourceGroup -z contoso.xyz
```

## <a name="test-the-name-resolution"></a>Tester la résolution de nom

Maintenant que vous disposez d’une zone DNS test avec un enregistrement « A » test, vous pouvez tester la résolution de noms avec un outil appelé *nslookup*. 

**Pour tester la résolution de noms DNS**

1. Exécutez l’applet de commande suivante pour obtenir la liste des serveurs de noms pour votre zone :

   ```azurecli
   az network dns record-set ns show --resource-group MyResourceGroup --zone-name contoso.xyz --name @
   ```

1. Copiez un des noms de serveur de noms à partir de la sortie de l’étape précédente.

1. Ouvrez une invite de commandes et exécutez la commande suivante :

   ```
   nslookup www.contoso.xyz <name server name>
   ```

   Par exemple :

   ```
   nslookup www.contoso.xyz ns1-08.azure-dns.com.
   ```

   Un écran similaire à celui-ci doit s’afficher :

   ![Capture d’écran montrant une fenêtre d’invite de commandes avec une commande nslookup et des valeurs pour Serveur, Adresse, Nom et Adresse.](media/dns-getstarted-portal/nslookup.PNG)

Le nom d’hôte **www\.contoso.xyz** se résout en **10.10.10.10**, tel que vous l’avez configuré. Ce résultat confirme que la résolution de noms fonctionne correctement.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez supprimer toutes les ressources créées dans ce démarrage rapide en supprimant le groupe de ressources :

```azurecli
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé vos premiers enregistrement et zone DNS à l’aide d’Azure CLI, vous pouvez créer des enregistrements pour une application web dans un domaine personnalisé.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)
