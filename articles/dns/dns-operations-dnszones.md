---
title: Gérer des zones DNS dans Azure DNS - PowerShell | Microsoft Docs
description: Vous pouvez gérer des zones DNS à l’aide d’Azure Powershell. Cet article décrit comment mettre à jour, supprimer et créer des zones DNS sur Azure DNS.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/19/2018
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6abcca9d9888dc8968d7233e7aee6cd76aa215f7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965746"
---
# <a name="how-to-manage-dns-zones-using-powershell"></a>Gestion des zones DNS à l'aide de PowerShell

> [!div class="op_single_selector"]
> * [Portail](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [Azure Classic CLI](./dns-operations-dnszones-cli.md)
> * [Azure CLI](dns-operations-dnszones-cli.md)

Cet article vous montre comment gérer vos zones DNS avec Azure PowerShell. Vous pouvez également gérer vos zones DNS à l’aide de [l’interface de ligne de commande Azure](dns-operations-dnszones-cli.md) multiplateforme ou du portail Azure.

Ce guide traite spécifiquement des zones DNS publiques. Pour plus d’informations sur l’utilisation d’Azure PowerShell pour gérer les Zones privées dans Azure DNS, consultez [Prise en main d’Azure DNS Private Zones à l’aide d’Azure PowerShell](private-dns-getstarted-powershell.md).

[!INCLUDE [dns-create-zone-about](../../includes/dns-create-zone-about-include.md)]

[!INCLUDE [dns-powershell-setup](../../includes/dns-powershell-setup-include.md)]


## <a name="create-a-dns-zone"></a>Création d’une zone DNS

Une zone DNS est créée à l’aide de l’applet de commande `New-AzureRmDnsZone` .

L’exemple ci-dessous crée une zone DNS appelée *contoso.com* dans le groupe de ressources *MyResourceGroup* :

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

L’exemple suivant montre comment créer une zone DNS avec deux [balises Azure Resource Manager](dns-zones-records.md#tags), *projet = demo* et *env = test* :

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

Azure DNS prend également en charge les zones DNS privées.  Pour en savoir plus sur les zones DNS privées, consultez la session relative à [l’utilisation d’Azure DNS pour les domaines privés](private-dns-overview.md). Vous trouverez un exemple de création d’une zone DNS privée sur la page [Bien démarrer avec les zones privées Azure DNS à l’aide de PowerShell](./private-dns-getstarted-powershell.md).

## <a name="get-a-dns-zone"></a>Obtention d’une zone DNS

Pour récupérer une zone DNS, utilisez l’applet de commande `Get-AzureRmDnsZone` : Cette opération retourne un objet de zone DNS correspondant à une zone existante dans Azure DNS. Cet objet contient des données sur la zone (par exemple le nombre de jeux d’enregistrements), mais ne contient pas les jeux d’enregistrement eux-mêmes (voir `Get-AzureRmDnsRecordSet`).

```powershell
Get-AzureRmDnsZone -Name contoso.com –ResourceGroupName MyAzureResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-8ec2-f4879750d201
Tags                  : {project, env}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org.,
                        ns4-01.azure-dns.info.}
NumberOfRecordSets    : 2
MaxNumberOfRecordSets : 5000
```

## <a name="list-dns-zones"></a>Création de la liste des zones DNS

En omettant le nom de la zone dans `Get-AzureRmDnsZone`, vous pouvez énumérer toutes les zones d’un groupe de ressources. Cette opération renvoie un tableau d’objets de la zone.

```powershell
$zoneList = Get-AzureRmDnsZone -ResourceGroupName MyAzureResourceGroup
```

En omettant le nom de zone et le nom du groupe de ressources de `Get-AzureRmDnsZone`, vous pouvez énumérer toutes les zones dans l’abonnement Azure.

```powershell
$zoneList = Get-AzureRmDnsZone
```

## <a name="update-a-dns-zone"></a>Mise à jour d’une zone DNS

Vous pouvez apporter des modifications à une ressource de zone DNS à l’aide de `Set-AzureRmDnsZone`. Cette applet de commande ne met pas à jour les jeux d’enregistrements DNS dans la zone (voir [Gestion des enregistrements DNS](dns-operations-recordsets.md)). Elle est utilisée seulement pour mettre à jour les propriétés de la ressource de zone elle-même. Les propriétés de zone accessibles en écriture sont actuellement limitées aux [« balises » Azure Resource Manager de la ressource de zone](dns-zones-records.md#tags).

Utilisez l’une des options suivantes pour mettre à jour la zone DNS :

### <a name="specify-the-zone-using-the-zone-name-and-resource-group"></a>Spécifier la zone en utilisant le nom de zone et le groupe de ressources

Cette approche remplace les balises de zone existantes par les valeurs spécifiées.

```powershell
Set-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup -Tag @{ project="demo"; env="test" }
```

### <a name="specify-the-zone-using-a-zone-object"></a>Spécifier la zone en utilisant un objet $zone

Cette approche récupère l’objet existant de la zone, modifie les balises, puis valide les modifications. De cette façon, les balises existantes peuvent être conservées.

```powershell
# Get the zone object
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup

# Remove an existing tag
$zone.Tags.Remove("project")

# Add a new tag
$zone.Tags.Add("status","approved")

# Commit changes
Set-AzureRmDnsZone -Zone $zone
```

Lors de l’utilisation de `Set-AzureRmDnsZone` avec un objet $zone, les [vérifications d’ETag](dns-zones-records.md#etags) sont utilisées pour éviter que les modifications simultanées soient remplacées. Le commutateur facultatif `-Overwrite` permet de supprimer ces vérifications.

## <a name="delete-a-dns-zone"></a>Suppression d’une zone DNS

Les zones DNS peuvent être supprimées à l’aide de l’applet de commande `Remove-AzureRmDnsZone`.

> [!NOTE]
> Supprimer une zone DNS supprime également tous les enregistrements DNS de la zone. Cette opération ne peut pas être annulée. Si la zone DNS est en cours d’utilisation, les services utilisant la zone échouent lors de la suppression de la zone.
>
>Pour vous protéger contre la suppression accidentelle de zones, consultez la page [Comment protéger des zones et enregistrements DNS](dns-protect-zones-recordsets.md).


Utilisez l’une des options suivantes pour supprimer une zone DNS :

### <a name="specify-the-zone-using-the-zone-name-and-resource-group-name"></a>Spécifier la zone en utilisant le nom de zone et le nom de groupe de ressources

```powershell
Remove-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
```

### <a name="specify-the-zone-using-a-zone-object"></a>Spécifier la zone en utilisant un objet $zone

Vous pouvez spécifier la zone à supprimer à l’aide d’un objet `$zone` retourné par `Get-AzureRmDnsZone`.

```powershell
$zone = Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup
Remove-AzureRmDnsZone -Zone $zone
```

L’objet de zone peut également être redirigé au lieu d’être transmis en tant que paramètre :

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyAzureResourceGroup | Remove-AzureRmDnsZone

```

Comme avec `Set-AzureRmDnsZone`, la spécification de la zone avec un objet `$zone` active les vérifications d’ETag et permet de s’assurer que les modifications simultanées ne sont pas supprimées. Utilisez le commutateur `-Overwrite` pour supprimer ces vérifications.

## <a name="confirmation-prompts"></a>Invites de confirmation

Les applets de commande `New-AzureRmDnsZone`, `Set-AzureRmDnsZone` et `Remove-AzureRmDnsZone` prennent en charge les invites de confirmation.

Les invites `New-AzureRmDnsZone` et `Set-AzureRmDnsZone` demandent une confirmation si la variable de préférence PowerShell `$ConfirmPreference` a la valeur `Medium` ou une valeur inférieure. En raison de l’impact potentiellement élevé de la suppression d’une zone DNS, l’applet de commande `Remove-AzureRmDnsZone` vous demande de confirmer si la variable PowerShell `$ConfirmPreference` a une valeur autre que `None`.

Étant donné que la valeur par défaut `$ConfirmPreference` est `High`, seule la commande `Remove-AzureRmDnsZone` demande une confirmation par défaut.

Vous pouvez remplacer le paramétrage actuel de `$ConfirmPreference` par le paramètre `-Confirm`. Si vous spécifiez les paramètres `-Confirm` ou `-Confirm:$True`, les applets de commande vous invitent à confirmer l’exécution. Si vous spécifiez le paramètre `-Confirm:$False`, l’applet de commande ne demande pas de confirmation.

Pour plus d’informations sur les paramètres `-Confirm` et `$ConfirmPreference`, voir [À propos des variables de préférence](/powershell/module/microsoft.powershell.core/about/about_preference_variables).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [gérer des jeux d’enregistrements et des enregistrements](dns-operations-recordsets.md) dans votre zone DNS.
<br>
Découvrez comment [déléguer votre domaine à Azure DNS](dns-domain-delegation.md).
<br>
Examinez la [documentation de référence d’Azure DNS PowerShell](/powershell/module/azurerm.dns).