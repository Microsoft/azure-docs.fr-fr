---
title: Gérer les enregistrements DNS dans Azure DNS avec Azure PowerShell | Microsoft Docs
description: Gestion des jeux d'enregistrements DNS et des enregistrements dans Azure DNS lorsque votre domaine est hébergé dans Azure DNS. Toutes les commandes PowerShell pour les opérations sur les jeux d'enregistrements et les enregistrements.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.custom: H1Hack27Feb2017
ms.workload: infrastructure-services
ms.date: 12/21/2016
ms.author: rohink
ms.openlocfilehash: f4e713f54ab4702b21763dc9fc6c7b606f94a945
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011589"
---
# <a name="manage-dns-records-and-recordsets-in-azure-dns-using-azure-powershell"></a>Gérer les enregistrements et jeux d’enregistrements DNS dans Azure DNS à l’aide d’Azure PowerShell

> [!div class="op_single_selector"]
> * [Portail Azure](dns-operations-recordsets-portal.md)
> * [Azure Classic CLI](./dns-operations-recordsets-cli.md)
> * [Azure CLI](dns-operations-recordsets-cli.md)
> * [PowerShell](dns-operations-recordsets.md)

Cet article explique comment gérer les enregistrements DNS pour votre zone DNS avec Azure PowerShell. Vous pouvez également gérer les enregistrements DNS à l’aide de l’[interface de ligne de commande Azure (Azure CLI)](dns-operations-recordsets-cli.md) multiplateforme ou via le [portail Azure](dns-operations-recordsets-portal.md).

Les exemples de cet article supposent que vous avez déjà [installé Azure PowerShell, ouvert une session et créé une zone DNS](dns-operations-dnszones.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="introduction"></a>Introduction

Avant de créer des enregistrements DNS dans Azure DNS, vous devez comprendre comment Azure DNS organise les enregistrements DNS en jeux d’enregistrements DNS.

[!INCLUDE [dns-about-records-include](../../includes/dns-about-records-include.md)]

Pour plus d’informations sur les enregistrements DNS dans Azure DNS, voir [Enregistrements et zones DNS](dns-zones-records.md).


## <a name="create-a-new-dns-record"></a>Créer un enregistrement DNS

Si votre nouvel enregistrement porte le même nom et est du même type qu’un enregistrement existant, vous devez l’[ajouter au jeu d’enregistrements existant](#add-a-record-to-an-existing-record-set). En revanche, si le nom et le type de votre nouvel enregistrement diffèrent de ceux de tous les enregistrements existants, vous devez créer un jeu d’enregistrements. 

### <a name="create-a-records-in-a-new-record-set"></a>Créer des enregistrements « A » dans un nouveau jeu d’enregistrements

Vous pouvez utiliser l’applet de commande `New-AzDnsRecordSet` pour créer des jeux d’enregistrements. Lors de la création d’un jeu d’enregistrements, vous devez en spécifier le nom, la zone et la durée de vie (TTL), ainsi que les enregistrements à créer et leur type.

Les paramètres pour ajouter des enregistrements à un jeu d'enregistrements varient selon le type de jeu d'enregistrements. Par exemple, si vous utilisez un jeu d’enregistrements de type « A », vous devez spécifier l’adresse IP à l’aide du paramètre `-IPv4Address`. D’autres paramètres sont utilisés pour d’autres types d’enregistrements. Pour plus d’informations, voir les autres exemples de types d’enregistrements.

L’exemple suivant crée un jeu d’enregistrements avec le nom relatif « www » dans la zone DNS « contoso.com ». Le nom complet du jeu d’enregistrements est « www.contoso.com ». Le type d’enregistrement est « A » et la durée de vie est de 3 600 secondes. Le jeu d’enregistrements ne contient qu’un seul enregistrement, dont l’adresse IP est « 1.2.3.4 ».

```powershell
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") 
```

Pour créer un jeu d’enregistrements à l’apex (au sommet) d’une zone (en l’occurrence, « contoso.com »), utilisez le nom de jeu d’enregistrements « \@ » (guillemets non compris) :

```powershell
New-AzDnsRecordSet -Name "@" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") 
```

Si vous devez créer un jeu d’enregistrements contenant plusieurs enregistrements, commencez par créer un groupe local, ajoutez les enregistrements, puis transmettez le groupe à `New-AzDnsRecordSet` comme suit :

```powershell
$aRecords = @()
$aRecords += New-AzDnsRecordConfig -IPv4Address "1.2.3.4"
$aRecords += New-AzDnsRecordConfig -IPv4Address "2.3.4.5"
New-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName MyResourceGroup -Ttl 3600 -RecordType A -DnsRecords $aRecords
```

Vous pouvez utiliser des [métadonnées de jeu d’enregistrements](dns-zones-records.md#tags-and-metadata) pour associer les données spécifiques de l’application à chaque jeu d’enregistrements, comme paires clé-valeur. L’exemple suivant montre comment créer un jeu d’enregistrements avec deux entrées de métadonnées, « dept=finance » et « environment=production ».

```powershell
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "1.2.3.4") -Metadata @{ dept="finance"; environment="production" } 
```

Azure DNS prend également en charge les jeux d’enregistrements « vides », qui peuvent servir d’espaces réservés pour réserver un nom DNS avant de créer des enregistrements DNS. Les jeux d’enregistrements vides sont visibles dans le volet de contrôle d’Azure DNS, mais n’apparaissent pas sur les serveurs de noms Azure DNS. L’exemple suivant crée un jeu d’enregistrements vide :

```powershell
New-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords @()
```

## <a name="create-records-of-other-types"></a>Créer des enregistrements d’autres types

À présent que nous avons vu en détail comment créer des enregistrements de type « A », les exemples suivants montrent comment créer des enregistrements d’autres types pris en charge par Azure DNS.

Dans chaque cas, nous montrons comment créer un jeu d’enregistrements contenant un seul enregistrement. Vous pouvez adapter les exemples précédents pour les enregistrements de type « A » afin de créer des jeux d’enregistrements d’autres types contenant plusieurs enregistrements avec des métadonnées, ou des jeux d’enregistrements vides.

Nous ne donnons pas d’exemple de création de jeu d’enregistrements SOA (Architecture orientée services), car les enregistrements de ce type sont créés et supprimés avec chaque zone DNS, et ne peuvent pas l’être séparément. En revanche, vous pouvez [modifier les enregistrements SOA en procédant de la manière décrite dans un exemple plus loin](#to-modify-an-soa-record).

### <a name="create-an-aaaa-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements AAAA avec un seul enregistrement

```powershell
New-AzDnsRecordSet -Name "test-aaaa" -RecordType AAAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ipv6Address "2607:f8b0:4009:1803::1005") 
```

### <a name="create-a-caa-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements CAA avec un seul enregistrement

```powershell
New-AzDnsRecordSet -Name "test-caa" -RecordType CAA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Caaflags 0 -CaaTag "issue" -CaaValue "ca1.contoso.com") 
```

### <a name="create-a-cname-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements CNAME avec un seul enregistrement

> [!NOTE]
> Les normes DNS n’autorisent pas la présence d’enregistrements CNAME ou de jeux d’enregistrements contenant plusieurs enregistrements à l’apex (sommet) d’une zone (`-Name '@'`).
> 
> Pour plus d’informations, voir [Enregistrements CNAME](dns-zones-records.md#cname-records).


```powershell
New-AzDnsRecordSet -Name "test-cname" -RecordType CNAME -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Cname "www.contoso.com") 
```

### <a name="create-an-mx-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements MX avec un seul enregistrement

Dans cet exemple, nous utilisons le nom de jeu d’enregistrements « \@ » pour créer un enregistrement MX à l’apex de la zone (dans ce cas, « contoso.com »).


```powershell
New-AzDnsRecordSet -Name "@" -RecordType MX -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Exchange "mail.contoso.com" -Preference 5) 
```

### <a name="create-an-ns-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements NS avec un seul enregistrement

```powershell
New-AzDnsRecordSet -Name "test-ns" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Nsdname "ns1.contoso.com") 
```

### <a name="create-a-ptr-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements PTR avec un seul enregistrement

Dans ce cas, « my-arpa-zone.com » indique la zone ARPA de recherche inversée représentant votre plage d’adresses IP. Chaque enregistrement PTR défini dans cette zone correspond à une adresse IP figurant dans cette plage d’adresses IP. Le nom d’enregistrement « 10 » est le dernier octet de l’adresse IP dans cette plage d’IP représentée par cet enregistrement.

```powershell
New-AzDnsRecordSet -Name 10 -RecordType PTR -ZoneName "my-arpa-zone.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Ptrdname "myservice.contoso.com") 
```

### <a name="create-an-srv-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements SRV avec un seul enregistrement

Lorsque vous créez un [jeu d’enregistrements SRV](dns-zones-records.md#srv-records), spécifiez le *\_service* et le *\_protocole* dans le nom du jeu d’enregistrements. Il est inutile d’inclure « \@ » dans le nom du jeu d’enregistrements lors de la création d’un enregistrement SRV défini à l’extrémité de la zone.

```powershell
New-AzDnsRecordSet -Name "_sip._tls" -RecordType SRV -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Priority 0 -Weight 5 -Port 8080 -Target "sip.contoso.com") 
```


### <a name="create-a-txt-record-set-with-a-single-record"></a>Créer un jeu d’enregistrements TXT avec un seul enregistrement

L’exemple suivant montre comment créer un enregistrement TXT. Pour plus d’informations sur la longueur maximale de chaîne prise en charge dans les enregistrements TXT, voir [Enregistrements TXT](dns-zones-records.md#txt-records).

```powershell
New-AzDnsRecordSet -Name "test-txt" -RecordType TXT -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -Value "This is a TXT record") 
```


## <a name="get-a-record-set"></a>Obtention d’un jeu d'enregistrements

Pour récupérer un jeu d’enregistrements existant, utilisez `Get-AzDnsRecordSet`. Cette applet de commande renvoie un objet local représentant le jeu d’enregistrements dans Azure DNS.

Comme avec l’applet de commande `New-AzDnsRecordSet`, le nom du jeu d’enregistrements doit être un nom *relatif*, c’est-à-dire qu’il ne doit pas contenir le nom de la zone. Vous devez également spécifier le type d’enregistrement et la zone contenant le jeu d’enregistrements.

L’exemple suivant montre comment récupérer un jeu d’enregistrements. Dans cet exemple, la zone est spécifiée à l’aide des paramètres `-ZoneName` et `-ResourceGroupName`.

```powershell
$rs = Get-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

Vous pouvez également spécifier la zone à l’aide d’un objet zone, transmis en utilisant le `-Zone` paramètre.

```powershell
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs = Get-AzDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

## <a name="list-record-sets"></a>Liste des jeux d'enregistrements

Vous pouvez également utiliser l’applet de commande `Get-AzDnsZone` pour répertorier les jeux d’enregistrements présents dans une zone, en omettant les paramètres `-Name` et/ou `-RecordType`.

L’exemple suivant retourne tous les jeux d’enregistrements présents dans la zone :

```powershell
$recordsets = Get-AzDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

L’exemple suivant montre comment récupérer tous les jeux d’enregistrements d’un type donné en spécifiant le type d’enregistrement, mais en omettant le nom du jeu d’enregistrements :

```powershell
$recordsets = Get-AzDnsRecordSet -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

Pour récupérer parmi les types d’enregistrements tous les jeux d’enregistrements portant un nom spécifique, vous devez récupérer tous les jeux d’enregistrements, puis filtrer les résultats :

```powershell
$recordsets = Get-AzDnsRecordSet -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | where {$_.Name.Equals("www")}
```

Dans tous les exemples ci-dessus, vous pouvez spécifier la zone à l’aide des paramètres `-ZoneName` et `-ResourceGroupName` (comme indiqué), ou en spécifiant un objet zone :

```powershell
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
$recordsets = Get-AzDnsRecordSet -Zone $zone
```

## <a name="add-a-record-to-an-existing-record-set"></a>Ajouter un enregistrement à un jeu d’enregistrements existant

Pour ajouter un enregistrement à un jeu d’enregistrements existant, suivez les trois étapes suivantes :

1. Obtenez le jeu d’enregistrements existant

    ```powershell
    $rs = Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

2. Ajoutez le nouvel enregistrement au jeu d’enregistrements local. Cette opération se fait hors connexion.

    ```powershell
    Add-AzDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. Validez la modification sur le service Azure DNS. 

    ```powershell
    Set-AzDnsRecordSet -RecordSet $rs
    ```

La cmdlet `Set-AzDnsRecordSet` *remplace* le jeu d’enregistrements existant dans Azure DNS (et tous les enregistrements qu’il contient) par le jeu d’enregistrements spécifié. Des [vérifications ETag](dns-zones-records.md#etags) permettent d’éviter les conflits de modifications simultanées. Le commutateur facultatif `-Overwrite` permet de supprimer ces vérifications.

Vous pouvez également *canaliser* cette séquence d’opérations, c’est-à-dire transmettre l’objet jeu d’enregistrements en utilisant le canal plutôt qu’en le transmettant en tant que paramètre :

```powershell
Get-AzDnsRecordSet -Name "www" –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Add-AzDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzDnsRecordSet
```

Les exemples ci-dessus montrent comment ajouter un enregistrement « A » à un jeu existant d’enregistrements de type « A ». Une séquence similaire d’opérations est utilisée pour ajouter des enregistrements à des jeux d’enregistrements d’autres types, en remplaçant le paramètre `-Ipv4Address` de l’applet de commande `Add-AzDnsRecordConfig` par d’autres paramètres spécifiques de chaque type d’enregistrement. Les paramètres pour chaque type d’enregistrement sont identiques à ceux de l’applet de commande `New-AzDnsRecordConfig`, comme indiqué dans les autres exemples de types d’enregistrements ci-dessus.

Les jeux d’enregistrements de type « CNAME » ou « SOA » ne peuvent pas contenir plusieurs enregistrements. Cette contrainte résulte des normes DNS. Il ne s’agit pas d’une limitation d’Azure DNS.

## <a name="remove-a-record-from-an-existing-record-set"></a>Suppression d’un enregistrement d’un jeu d'enregistrements existant

Le processus de suppression d’un enregistrement d’un jeu d’enregistrements est similaire au processus d’ajout d’un enregistrement à un jeu d’enregistrements existant :

1. Obtenez le jeu d’enregistrements existant

    ```powershell
    $rs = Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A
    ```

2. Supprimez l’enregistrement de l’objet jeu d’enregistrements local. Cette opération se fait hors connexion. L’enregistrement à supprimer doit correspondre exactement à un enregistrement existant relativement à tous les paramètres.

    ```powershell
    Remove-AzDnsRecordConfig -RecordSet $rs -Ipv4Address "5.6.7.8"
    ```

3. Validez la modification sur le service Azure DNS. Utilisez le commutateur facultatif `-Overwrite` pour désactiver les [vérifications ETag](dns-zones-records.md#etags) des modifications simultanées.

    ```powershell
    Set-AzDnsRecordSet -RecordSet $Rs
    ```

La séquence utilisée ci-dessus pour supprimer le dernier enregistrement d’un jeu d’enregistrements n’a pas pour effet de supprimer celui-ci, mais de le laisser vide. Pour supprimer entièrement un jeu d’enregistrements, voir [Supprimer un jeu d’enregistrements](#delete-a-record-set).

Comme pour l’ajout d’enregistrements à un jeu d’enregistrements, vous pouvez canaliser la séquence des opérations de suppression d’un jeu d’enregistrements comme suit :

```powershell
Get-AzDnsRecordSet -Name www –ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" -RecordType A | Remove-AzDnsRecordConfig -Ipv4Address "5.6.7.8" | Set-AzDnsRecordSet
```

Différents types d’enregistrements sont pris en charge en transmettant les paramètres spécifiques du type approprié à `Remove-AzDnsRecordSet`. Les paramètres pour chaque type d’enregistrement sont identiques à ceux de l’applet de commande `New-AzDnsRecordConfig`, comme indiqué dans les autres exemples de types d’enregistrements ci-dessus.


## <a name="modify-an-existing-record-set"></a>Modifier un jeu d’enregistrements

Pour modifier un jeu d’enregistrements, procédez de la même manière que pour y ajouter ou en supprimer des enregistrements :

1. Récupérez le jeu d’enregistrements existant en utilisant `Get-AzDnsRecordSet`.
2. Modifiez l’objet jeu d’enregistrements local à l’aide des opérations suivantes :
    * ajout ou suppression d’enregistrements ;
    * modification des paramètres d’enregistrements existants ;
    * modification des métadonnées et de la durée de vie (TTL) du jeu d’enregistrements.
3. Validez vos modifications en utilisant l’applet de commande `Set-AzDnsRecordSet` . Celle-ci *remplace* le jeu d’enregistrement existant dans Azure DNS par le jeu d’enregistrements spécifié.

Lorsque vous utilisez l’applet de commande `Set-AzDnsRecordSet`, des [vérifications ETag](dns-zones-records.md#etags) permettent d’éviter les conflits de modifications simultanées. Le commutateur facultatif `-Overwrite` permet de supprimer ces vérifications.

### <a name="to-update-a-record-in-an-existing-record-set"></a>Pour mettre à jour un enregistrement dans un jeu d’enregistrements existant

Dans cet exemple, nous modifions l’adresse IP d’un enregistrement « A » existant :

```powershell
$rs = Get-AzDnsRecordSet -name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Ipv4Address = "9.8.7.6"
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-an-soa-record"></a>Pour modifier un enregistrement SOA

Vous ne pouvez pas ajouter ou supprimer d’enregistrements dans le jeu d’enregistrements SOA créé automatiquement à l’apex de la zone (`-Name "@"`, guillemets compris). Vous pouvez cependant modifier les paramètres dans l’enregistrement SOA (à l’exception de « l’hôte ») et pendant la durée de vie du jeu d’enregistrements.

L’exemple suivant montre comment modifier la propriété *Email* de l’enregistrement SOA :

```powershell
$rs = Get-AzDnsRecordSet -Name "@" -RecordType SOA -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
$rs.Records[0].Email = "admin.contoso.com"
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-ns-records-at-the-zone-apex"></a>Pour modifier des enregistrements NS à l’apex de la zone

Le jeu d’enregistrements NS à l’apex de la zone est créé automatiquement avec chaque zone DNS. Il contient les noms des serveurs de noms Azure DNS attribués à la zone.

Vous pouvez ajouter des serveurs de noms supplémentaires à ce jeu d’enregistrements NS, pour prendre en charge le co-hébergement de domaines avec plusieurs fournisseurs DNS. Vous pouvez également modifier la durée de vie et les métadonnées pour ce jeu d’enregistrements. Toutefois, vous ne pouvez pas supprimer ni modifier les serveurs de noms Azure DNS préremplis.

Notez que cela s’applique uniquement au jeu d’enregistrements NS défini à l’apex de la zone. Les autres jeux d’enregistrements NS dans votre zone (tels que ceux utilisés pour déléguer des zones enfants) peuvent être modifiés sans contrainte.

L’exemple suivant montre comment ajouter un serveur de noms supplémentaire au jeu d’enregistrements NS défini à l’apex de la zone :

```powershell
$rs = Get-AzDnsRecordSet -Name "@" -RecordType NS -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Add-AzDnsRecordConfig -RecordSet $rs -Nsdname ns1.myotherdnsprovider.com
Set-AzDnsRecordSet -RecordSet $rs
```

### <a name="to-modify-record-set-metadata"></a>Pour modifier les métadonnées du jeu d’enregistrements

Vous pouvez utiliser des [métadonnées de jeu d’enregistrements](dns-zones-records.md#tags-and-metadata) pour associer les données spécifiques de l’application à chaque jeu d’enregistrements, comme paires clé-valeur.

L’exemple suivant montre comment modifier les métadonnées d’un jeu d’enregistrements :

```powershell
# Get the record set
$rs = Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"

# Add 'dept=finance' name-value pair
$rs.Metadata.Add('dept', 'finance') 

# Remove metadata item named 'environment'
$rs.Metadata.Remove('environment')  

# Commit changes
Set-AzDnsRecordSet -RecordSet $rs
```


## <a name="delete-a-record-set"></a>Supprimer un jeu d’enregistrements

Les jeux d’enregistrements peuvent être supprimés à l’aide de l’applet de commande `Remove-AzDnsRecordSet` . La suppression d’un jeu d’enregistrements a pour effet de supprimer également tous les enregistrements qu’il contient.

> [!NOTE]
> Vous ne pouvez pas supprimer de jeux d’enregistrements SOA et NS au niveau de l’apex de la zone (`-Name '@'`).  Azure DNS les crée automatiquement lors de la création de la zone, et les supprime automatiquement lors de la suppression de celle-ci.

L’exemple suivant montre comment supprimer un jeu d’enregistrements. Dans cet exemple, le nom, le type, la zone et le groupe de ressources du jeu d’enregistrements sont spécifiés explicitement.

```powershell
Remove-AzDnsRecordSet -Name "www" -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
```

Vous pouvez également spécifier le jeu d’enregistrements par un nom et un type, et la zone à l’aide d’un objet :

```powershell
$zone = Get-AzDnsZone -Name "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzDnsRecordSet -Name "www" -RecordType A -Zone $zone
```

Une troisième option consiste à spécifier le jeu d’enregistrements en utilisant un objet jeu d’enregistrements :

```powershell
$rs = Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup"
Remove-AzDnsRecordSet -RecordSet $rs
```

Lorsque vous spécifiez le jeu d’enregistrements à supprimer à l’aide d’un objet jeu d’enregistrements, des [vérifications ETag](dns-zones-records.md#etags) veillent à ce que des modifications simultanées ne soient pas supprimées. Le commutateur facultatif `-Overwrite` permet de supprimer ces vérifications.

L'objet du jeu d'enregistrements peut également être envoyé au lieu d’être transmis en tant que paramètre :

```powershell
Get-AzDnsRecordSet -Name www -RecordType A -ZoneName "contoso.com" -ResourceGroupName "MyResourceGroup" | Remove-AzDnsRecordSet
```

## <a name="confirmation-prompts"></a>Invites de confirmation

Les applets de commande `New-AzDnsRecordSet`, `Set-AzDnsRecordSet` et `Remove-AzDnsRecordSet` prennent en charge les invites de confirmation.

Chaque applet de commande demande une confirmation si la variable de préférence PowerShell `$ConfirmPreference` a la valeur `Medium` ou une valeur inférieure. La valeur par défaut de la variable `$ConfirmPreference` étant `High`, ces invites ne s’affichent pas en cas d’utilisation des paramètres PowerShell par défaut.

Vous pouvez remplacer le paramétrage actuel de `$ConfirmPreference` par le paramètre `-Confirm`. Si vous spécifiez les paramètres `-Confirm` ou `-Confirm:$True`, les applets de commande vous invitent à confirmer l’exécution. Si vous spécifiez le paramètre `-Confirm:$False`, l’applet de commande ne demande pas de confirmation. 

Pour plus d’informations sur les paramètres `-Confirm` et `$ConfirmPreference`, voir [À propos des variables de préférence](/powershell/module/microsoft.powershell.core/about/about_preference_variables).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les [zones et enregistrements dans Azure DNS](dns-zones-records.md).
<br>
Découvrez comment [protéger vos zones et enregistrements](dns-protect-zones-recordsets.md) lors de l’utilisation d’Azure DNS.
<br>
Examinez la [documentation de référence d’Azure DNS PowerShell](/powershell/module/az.dns).