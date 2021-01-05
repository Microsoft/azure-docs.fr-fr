---
title: Gestion des enregistrements DNS dans Azure DNS à l’aide de l’interface de ligne de commande Azure
description: Gestion des jeux d'enregistrements DNS et des enregistrements dans Azure DNS lorsque votre domaine est hébergé dans Azure DNS.
author: rohinkoul
ms.assetid: 5356a3a5-8dec-44ac-9709-0c2b707f6cb5
ms.service: dns
ms.devlang: azurecli
ms.topic: how-to
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.workload: infrastructure-services
ms.date: 05/15/2018
ms.author: rohink
ms.openlocfilehash: 2d3989b3c477a35d602f1ccf3e45d6f597f5d78d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011561"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-the-azure-cli"></a>Gérer les enregistrements DNS et les jeux d’enregistrement dans Azure DNS à l’aide d’Azure CLI

> [!div class="op_single_selector"]
> * [Portail Azure](dns-operations-recordsets-portal.md)
> * [Azure CLI](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

Cet article explique comment gérer des enregistrements DNS pour votre zone DNS à l’aide de l’interface Azure CLI multiplateforme, disponible sur Windows, Mac et Linux. Vous pouvez également gérer vos enregistrements DNS à l’aide [d’Azure PowerShell](dns-operations-recordsets.md) ou du [portail Azure](dns-operations-recordsets-portal.md).

Les exemples de cet article supposent que vous avez déjà [installé l’interface de ligne de commande Azure, ouvert une session et créé une zone DNS](dns-operations-dnszones-cli.md).

## <a name="introduction"></a>Introduction

Avant de créer des enregistrements DNS dans Azure DNS, vous devez comprendre comment Azure DNS organise les enregistrements DNS en jeux d’enregistrements DNS.

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

Pour plus d’informations sur les enregistrements DNS dans Azure DNS, voir [Enregistrements et zones DNS](dns-zones-records.md).

## <a name="create-a-dns-record"></a>Créer un enregistrement DNS

Pour créer un enregistrement DNS, utilisez la commande `az network dns record-set <record-type> add-record` (où `<record-type>` correspond au type d’enregistrement, c’est-à-dire a, srv, txt, etc.) Pour obtenir de l’aide, consultez l’article `az network dns record-set --help`.

Lors de la création d’un enregistrement, vous devez spécifier le nom du groupe de ressources, le nom de la zone, le type d’enregistrement et les détails de l’enregistrement créé. Le nom du jeu d’enregistrements doit être un nom *relatif*, c’est-à-dire qu’il ne doit pas contenir le nom de la zone.

Si le jeu d’enregistrements n’existe pas, cette commande le crée pour vous. Si le jeu d’enregistrements existe déjà, cette commande ajoute l’enregistrement spécifié au jeu d’enregistrements existant.

Si un jeu d’enregistrements est créé, une durée de vie (TTL) de 3600 est utilisée par défaut. Pour plus d’instructions sur l’utilisation de différents TTL, consultez [Création d’un jeu d’enregistrements DNS](#create-a-dns-record-set).

L’exemple suivant crée un enregistrement A appelé *www* dans la zone *contoso.com* du groupe de ressources *MyResourceGroup*. L’adresse IP de l’enregistrement A est *1.2.3.4*.

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

Pour créer un jeu d’enregistrements à l’extrémité de la zone (dans cet exemple, « contoso.com »), utilisez le nom d’enregistrement "\@" (guillemets compris) :

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --ipv4-address 1.2.3.4
```

## <a name="create-a-dns-record-set"></a>Créer un jeu d’enregistrements DNS

Dans les exemples ci-dessus, l’enregistrement DNS a été ajouté à un jeu d’enregistrements existant, ou le jeu d’enregistrements a été créé *implicitement*. Vous pouvez également créer le jeu d’enregistrements *explicitement* avant d’ajouter des enregistrements à celui-ci. Azure DNS prend en charge les jeux d’enregistrements « vides », qui peuvent servir d’espaces réservés pour réserver un nom DNS avant de créer des enregistrements DNS. Les jeux d’enregistrements vides sont visibles dans le volet de contrôle d’Azure DNS, mais n’apparaissent pas sur les serveurs de noms Azure DNS.

Des jeux d’enregistrements sont créés à l’aide de la commande `az network dns record-set <record-type> create`. Pour obtenir de l’aide, consultez l’article `az network dns record-set <record-type> create --help`.

La création explicite du jeu d’enregistrements permet de spécifier les propriétés de jeu d’enregistrements, comme la [Durée de vie (TTL)](dns-zones-records.md#time-to-live) et les métadonnées. Vous pouvez utiliser des [métadonnées de jeu d’enregistrements](dns-zones-records.md#tags-and-metadata) pour associer les données spécifiques de l’application à chaque jeu d’enregistrements, comme paires clé-valeur.

L’exemple suivant crée un jeu d’enregistrements vide de type « A » avec une durée de vie de 60 secondes, à l’aide du paramètre `--ttl` (forme abrégée `-l`) :

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --ttl 60
```

L’exemple suivant crée un jeu d’enregistrements avec deux entrées de métadonnées, dept=finance et environment=production, par le biais du paramètre `--metadata` :

```azurecli
az network dns record-set a create --resource-group myresourcegroup --zone-name contoso.com --name www --metadata "dept=finance" "environment=production"
```

Après avoir créé un jeu d’enregistrements vide, les enregistrements peuvent être ajoutés à l’aide de `azure network dns record-set <record-type> add-record`, comme décrit dans [Création d’un enregistrement DNS](#create-a-dns-record).

## <a name="create-records-of-other-types"></a>Créer des enregistrements d’autres types

À présent que nous avons vu en détail comment créer des enregistrements de type « A », les exemples suivants montrent comment créer des enregistrements d’autres types pris en charge par Azure DNS.

Les paramètres utilisés pour spécifier les données de l’enregistrement varient selon le type de l’enregistrement. Par exemple, pour un enregistrement de type « A », vous spécifiez l’adresse IPv4 avec le paramètre `--ipv4-address <IPv4 address>`. Les paramètres pour chaque type d’enregistrement peuvent être spécifiés à l’aide de `az network dns record-set <record-type> add-record --help`.

Dans chaque cas, nous montrons comment créer un seul enregistrement. L’enregistrement est ajouté au jeu d’enregistrements existant, ou à un jeu d’enregistrements créé implicitement. Pour plus d’informations sur la création de jeux d’enregistrements et la définition explicite des paramètres de jeu d’enregistrements, consultez [Création d’un jeu d’enregistrements DNS](#create-a-dns-record-set).

Nous ne donnons pas d’exemple de création de jeu d’enregistrements SOA (Architecture orientée services), car les enregistrements de ce type sont créés et supprimés avec chaque zone DNS, et ne peuvent pas l’être séparément. En revanche, vous pouvez [modifier les enregistrements SOA en procédant de la manière décrite dans un exemple plus loin](#to-modify-an-soa-record).

### <a name="create-an-aaaa-record"></a>Créer un enregistrement AAAA

```azurecli
az network dns record-set aaaa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-aaaa --ipv6-address 2607:f8b0:4009:1803::1005
```

### <a name="create-an-caa-record"></a>Créer un enregistrement CAA

```azurecli
az network dns record-set caa add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-caa --flags 0 --tag "issue" --value "ca1.contoso.com"
```

### <a name="create-a-cname-record"></a>Créer un enregistrement CNAME

> [!NOTE]
> Les normes DNS n’autorisent pas la présence d’enregistrements CNAME ou de jeux d’enregistrements contenant plusieurs enregistrements à l’apex (sommet) d’une zone (`--Name "@"`).
> 
> Pour plus d’informations, voir [Enregistrements CNAME](dns-zones-records.md#cname-records).

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.contoso.com
```

### <a name="create-an-mx-record"></a>Créer un enregistrement MX

Dans cet exemple, nous utilisons le nom de jeu d’enregistrements « \@ » pour créer l’enregistrement MX à l’apex de la zone (dans ce cas, « contoso.com »).

```azurecli
az network dns record-set mx add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --exchange mail.contoso.com --preference 5
```

### <a name="create-an-ns-record"></a>Créer un enregistrement NS

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-ns --nsdname ns1.contoso.com
```

### <a name="create-a-ptr-record"></a>Création d’un enregistrement PTR

Dans ce cas, « my-arpa-zone.com » indique la zone ARPA représentant votre plage d’adresses IP. Chaque enregistrement PTR défini dans cette zone correspond à une adresse IP figurant dans cette plage d’adresses IP.  Le nom d’enregistrement « 10 » est le dernier octet de l’adresse IP dans cette plage d’IP représentée par cet enregistrement.

```azurecli
az network dns record-set ptr add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name my-arpa.zone.com --ptrdname myservice.contoso.com
```

### <a name="create-an-srv-record"></a>Création d’un enregistrement SRV

Lorsque vous créez un [jeu d’enregistrements SRV](dns-zones-records.md#srv-records), spécifiez le *\_service* et le *\_protocole* dans le nom du jeu d’enregistrements. Il est inutile d’inclure « \@ » dans le nom du jeu d’enregistrements lors de la création d’un enregistrement SRV défini à l’extrémité de la zone.

```azurecli
az network dns record-set srv add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name _sip._tls --priority 10 --weight 5 --port 8080 --target sip.contoso.com
```

### <a name="create-a-txt-record"></a>Création d’un enregistrement TXT

L’exemple suivant montre comment créer un enregistrement TXT. Pour plus d’informations sur la longueur maximale de chaîne prise en charge dans les enregistrements TXT, voir [Enregistrements TXT](dns-zones-records.md#txt-records).

```azurecli
az network dns record-set txt add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-txt --value "This is a TXT record"
```

## <a name="get-a-record-set"></a>Obtention d’un jeu d'enregistrements

Pour récupérer un jeu d’enregistrements existant, utilisez `az network dns record-set <record-type> show`. Pour obtenir de l’aide, consultez l’article `az network dns record-set <record-type> show --help`.

Comme lors de la création d’un enregistrement ou jeu d’enregistrements, le nom du jeu d’enregistrements doit être un nom *relatif*, c’est-à-dire qu’il ne doit pas contenir le nom de la zone. Vous devez également spécifier le type d’enregistrement, la zone contenant le jeu d’enregistrements et le groupe de ressources contenant la zone.

L’exemple suivant retrouve l’enregistrement *www* de type A dans la zone *contoso.com* du groupe de ressources *MyResourceGroup* :

```azurecli
az network dns record-set a show --resource-group myresourcegroup --zone-name contoso.com --name www
```

## <a name="list-record-sets"></a>Liste des jeux d'enregistrements

Vous pouvez répertorier tous les enregistrements d’une zone DNS à l’aide de la commande `az network dns record-set list` . Pour obtenir de l’aide, consultez l’article `az network dns record-set list --help`.

Cet exemple renvoie tous les jeux d’enregistrements dans la zone *contoso.com*, dans le groupe de ressources *MyResourceGroup*, quel que soit le nom ou le type d’enregistrement :

```azurecli
az network dns record-set list --resource-group myresourcegroup --zone-name contoso.com
```

Cet exemple retourne tous les jeux d’enregistrements correspondant au type d’enregistrement donné (dans ce cas, les enregistrements « A ») :

```azurecli
az network dns record-set a list --resource-group myresourcegroup --zone-name contoso.com 
```

## <a name="add-a-record-to-an-existing-record-set"></a>Ajouter un enregistrement à un jeu d’enregistrements existant

Vous pouvez utiliser `az network dns record-set <record-type> add-record` à la fois pour créer un enregistrement dans un nouveau jeu d’enregistrements ou pour ajouter un enregistrement à un jeu d’enregistrements existant.

Pour plus d’informations, consultez [Création d’un enregistrement DNS](#create-a-dns-record) et [Création d’enregistrements d’autres types](#create-records-of-other-types) ci-dessus.

## <a name="remove-a-record-from-an-existing-record-set"></a>Suppression d’un enregistrement d’un jeu d'enregistrements existant.

Pour supprimer un enregistrement DNS d’un jeu d'enregistrements existant, utilisez `az network dns record-set <record-type> remove-record`. Pour obtenir de l’aide, consultez l’article `az network dns record-set <record-type> remove-record -h`.

Cette commande supprime un enregistrement DNS d’un jeu d’enregistrements. Si le dernier enregistrement d’un jeu d’enregistrements est supprimé, le jeu d’enregistrements lui-même est également supprimé. Pour conserver le jeu d’enregistrements vide, utilisez l’option `--keep-empty-record-set`.

Vous devez spécifier l’enregistrement à supprimer et la zone de laquelle il doit être supprimé, en utilisant les mêmes paramètres que lors de la création d’un enregistrement avec `az network dns record-set <record-type> add-record`. Ces paramètres sont décrits dans [Création d’un enregistrement DNS](#create-a-dns-record) et [Création d’enregistrements d’autres types](#create-records-of-other-types) ci-dessus.

L’exemple suivant supprime l’enregistrement A avec la valeur « 1.2.3.4 » du jeu d’enregistrements *www* dans la zone *contoso.com* du groupe de ressources *MyResourceGroup*.

```azurecli
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "www" --ipv4-address 1.2.3.4
```

## <a name="modify-an-existing-record-set"></a>Modifier un jeu d’enregistrements

Chaque jeu d’enregistrements contient une [durée de vie (TTL)](dns-zones-records.md#time-to-live), des [métadonnées](dns-zones-records.md#tags-and-metadata) et des enregistrements DNS. Les sections suivantes expliquent comment modifier chacune de ces propriétés.

### <a name="to-modify-an-a-aaaa-caa-mx-ns-ptr-srv-or-txt-record"></a>Pour modifier un enregistrement A, AAAA, CAA, MX, NS, PTR, SRV ou TXT

Pour modifier un enregistrement existant de type A, AAAA, CAA, MX, NS, PTR, SRV ou TXT, vous devez d’abord ajouter un nouvel enregistrement, puis supprimer l’enregistrement existant. Pour obtenir des instructions détaillées sur la façon de supprimer et ajouter des enregistrements, consultez les sections précédentes de cet article.

L’exemple suivant montre comment modifier un enregistrement « A », de l’adresse IP 1.2.3.4 à l’adresse IP 5.6.7.8 :

```azurecli
az network dns record-set a add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 5.6.7.8
az network dns record-set a remove-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name www --ipv4-address 1.2.3.4
```

Vous ne pouvez pas ajouter, supprimer ou modifier les enregistrements dans le jeu d’enregistrements NS créé automatiquement à l’apex de la zone (`--Name "@"`, guillemets inclus). Pour ce jeu d’enregistrements, les seules modifications autorisées sont celles de la durée de vie (TTL) et des métadonnées du jeu d’enregistrements.

### <a name="to-modify-a-cname-record"></a>Pour modifier un enregistrement CNAME

Contrairement à la plupart des autres types d’enregistrements, un jeu d’enregistrements CNAME ne peut contenir qu’un seul enregistrement.  Par conséquent, vous ne pouvez pas remplacer la valeur actuelle en ajoutant un nouvel enregistrement et en supprimant l’enregistrement existant, comme pour d’autres types d’enregistrements.

À la place, pour modifier un enregistrement CNAME, utilisez `az network dns record-set cname set-record`. Pour obtenir de l’aide, consultez `az network dns record-set cname set-record --help`

Cet exemple modifie le jeu d’enregistrements CNAME *www* dans la zone *contoso.com*, dans le groupe de ressources *MyResourceGroup*, pour pointer vers « www.fabrikam.net » au lieu de sa valeur existante :

```azurecli
az network dns record-set cname set-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name test-cname --cname www.fabrikam.net
``` 

### <a name="to-modify-an-soa-record"></a>Pour modifier un enregistrement SOA

Contrairement à la plupart des autres types d’enregistrements, un jeu d’enregistrements CNAME ne peut contenir qu’un seul enregistrement.  Par conséquent, vous ne pouvez pas remplacer la valeur actuelle en ajoutant un nouvel enregistrement et en supprimant l’enregistrement existant, comme pour d’autres types d’enregistrements.

À la place, pour modifier l’enregistrement SOA, utilisez `az network dns record-set soa update`. Pour obtenir de l’aide, consultez l’article `az network dns record-set soa update --help`.

L’exemple suivant montre comment définir la propriété « email » de l’enregistrement SOA pour la zone *contoso.com* dans le groupe de ressources *MyResourceGroup* :

```azurecli
az network dns record-set soa update --resource-group myresourcegroup --zone-name contoso.com --email admin.contoso.com
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>Pour modifier des enregistrements NS à l’apex de la zone

Le jeu d’enregistrements NS à l’apex de la zone est créé automatiquement avec chaque zone DNS. Il contient les noms des serveurs de noms Azure DNS attribués à la zone.

Vous pouvez ajouter des serveurs de noms supplémentaires à ce jeu d’enregistrements NS, pour prendre en charge le co-hébergement de domaines avec plusieurs fournisseurs DNS. Vous pouvez également modifier la durée de vie et les métadonnées pour ce jeu d’enregistrements. Toutefois, vous ne pouvez pas supprimer ni modifier les serveurs de noms Azure DNS préremplis.

Notez que cela s’applique uniquement au jeu d’enregistrements NS défini à l’apex de la zone. Les autres jeux d’enregistrements NS dans votre zone (tels que ceux utilisés pour déléguer des zones enfants) peuvent être modifiés sans contrainte.

L’exemple suivant montre comment ajouter un serveur de noms supplémentaire au jeu d’enregistrements NS défini à l’apex de la zone :

```azurecli
az network dns record-set ns add-record --resource-group myresourcegroup --zone-name contoso.com --record-set-name "@" --nsdname ns1.myotherdnsprovider.com 
```

### <a name="to-modify-the-ttl-of-an-existing-record-set"></a>Pour modifier la durée de vie (TTL) d’un jeu d’enregistrements

Pour modifier la durée de vie (TTL) d’un jeu d’enregistrements, utilisez `azure network dns record-set <record-type> update`. Pour obtenir de l’aide, consultez l’article `azure network dns record-set <record-type> update --help`.

L’exemple suivant montre comment modifier la durée de vie d’un jeu d’enregistrements, dans ce cas sur 60 secondes :

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set ttl=60
```

### <a name="to-modify-the-metadata-of-an-existing-record-set"></a>Pour modifier les métadonnées d’un jeu d’enregistrements

Vous pouvez utiliser des [métadonnées de jeu d’enregistrements](dns-zones-records.md#tags-and-metadata) pour associer les données spécifiques de l’application à chaque jeu d’enregistrements, comme paires clé-valeur. Pour modifier les métadonnées d’un jeu d’enregistrements, utilisez `az network dns record-set <record-type> update`. Pour obtenir de l’aide, consultez l’article `az network dns record-set <record-type> update --help`.

L’exemple suivant montre comment modifier un jeu d’enregistrements avec deux entrées de métadonnées, « dept=finance » et « environment=production ». Notez que toutes les métadonnées existantes sont *remplacées* par les valeurs fournies.

```azurecli
az network dns record-set a update --resource-group myresourcegroup --zone-name contoso.com --name www --set metadata.dept=finance metadata.environment=production
```

## <a name="delete-a-record-set"></a>Supprimer un jeu d’enregistrements

Les jeux d’enregistrements peuvent être supprimés à l’aide de la commande `az network dns record-set <record-type> delete`. Pour obtenir de l’aide, consultez l’article `azure network dns record-set <record-type> delete --help`. La suppression d’un jeu d’enregistrements a pour effet de supprimer également tous les enregistrements qu’il contient.

> [!NOTE]
> Vous ne pouvez pas supprimer de jeux d’enregistrements SOA et NS au niveau de l’apex de la zone (`--name "@"`).  Ceux-ci sont créés automatiquement lors de la création de la zone, et automatiquement supprimés lors de la suppression de celle-ci.

L’exemple suivant supprime le jeu d’enregistrements *www* de type A dans la zone *contoso.com* du groupe de ressources *MyResourceGroup* :

```azurecli
az network dns record-set a delete --resource-group myresourcegroup --zone-name contoso.com --name www
```

Vous êtes invité à confirmer l’opération de suppression. Pour supprimer cette invite, utilisez le switch `--yes`.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les [zones et enregistrements dans Azure DNS](dns-zones-records.md).
<br>
Découvrez comment [protéger vos zones et enregistrements](dns-protect-zones-recordsets.md) lors de l’utilisation d’Azure DNS.
