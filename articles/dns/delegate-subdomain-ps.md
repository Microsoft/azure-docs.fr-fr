---
title: Déléguer un sous-domaine - Azure PowerShell - Azure DNS
description: Avec ce parcours d’apprentissage, commencez à déléguer un sous-domaine Azure DNS à l'aide d'Azure PowerShell.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 2/7/2019
ms.author: rohink
ms.openlocfilehash: 9b37d313aa5d8c2255b4e3be69831dfcb50238ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84712545"
---
# <a name="delegate-an-azure-dns-subdomain-using-azure-powershell"></a>Déléguer un sous-domaine d’Azure DNS à l’aide d’Azure PowerShell

Vous pouvez utiliser Azure PowerShell pour déléguer un sous-domaine de DNS. Par exemple, si vous êtes propriétaire du domaine contoso.com, vous pouvez déléguer un sous-domaine nommé *engineering* à une autre zone que vous pouvez administrer séparément de la zone contoso.com.

Si vous préférez, vous pouvez déléguer un sous-domaine à l’aide du [portail Azure](delegate-subdomain.md).

> [!NOTE]
> Contoso.com est utilisé comme exemple tout au long de cet article. Changez votre nom de votre domaine par contoso.com.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Pour déléguer un sous-domaine Azure DNS, vous devez tout d’abord déléguer votre domaine public à Azure DNS. Pour obtenir des instructions sur la façon de configurer vos serveurs de noms pour la délégation, consultez [Déléguer un domaine à Azure DNS](./dns-delegate-domain-azure-dns.md). Une fois votre domaine délégué à votre zone Azure DNS, vous pouvez déléguer votre sous-domaine.

## <a name="create-a-zone-for-your-subdomain"></a>Créer une zone pour votre sous-domaine

Tout d’abord, créez la zone pour le sous-domaine **engineering**.

`New-AzDnsZone -ResourceGroupName <resource group name> -Name engineering.contoso.com`

## <a name="note-the-name-servers"></a>Noter les serveurs de noms

Ensuite, notez les quatre serveurs de noms pour le sous-domaine engineering.

`Get-AzDnsRecordSet -ZoneName engineering.contoso.com -ResourceGroupName <resource group name> -RecordType NS`

## <a name="create-a-test-record"></a>Créer un enregistrement test

Dans la zone engineering, créez un enregistrement **A** à utiliser à des fins de test.

   `New-AzDnsRecordSet -ZoneName engineering.contoso.com -ResourceGroupName <resource group name> -Name www -RecordType A -ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address 10.10.10.10)`.

## <a name="create-an-ns-record"></a>Créer un enregistrement NS

Ensuite, créez un enregistrement de serveur de noms (NS) pour la zone **engineering** dans la zone contoso.com.

```azurepowershell
$Records = @()
$Records += New-AzDnsRecordConfig -Nsdname <name server 1 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 2 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 3 noted previously>
$Records += New-AzDnsRecordConfig -Nsdname <name server 4 noted previously>
$RecordSet = New-AzDnsRecordSet -Name engineering -RecordType NS -ResourceGroupName <resource group name> -TTL 3600 -ZoneName contoso.com -DnsRecords $Records
```

## <a name="test-the-delegation"></a>Tester la délégation

Utilisez nslookup pour tester la délégation.

1. Ouvrez une fenêtre PowerShell.
2. À l’invite de commandes, tapez `nslookup www.engineering.contoso.com.`.
3. Vous devez recevoir une réponse ne faisant pas autorité montrant l’adresse **10.10.10.10**.

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [Configurer des DNS inversés dans les services hébergés par Azure](dns-reverse-dns-for-azure-services.md).