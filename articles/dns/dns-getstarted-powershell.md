---
title: 'Démarrage rapide : Créer un enregistrement et une zone Azure DNS - Azure PowerShell'
titleSuffix: Azure DNS
description: Découvrez comment créer une zone et un enregistrement DNS dans Azure DNS. Il s’agit d’un démarrage rapide pas à pas pour la création et la gestion de votre première zone et de votre premier enregistrement DNS à l’aide d’Azure PowerShell.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: rohink
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f2563c33d02490732f73fcf9d1a78f548ec2d3e2
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92282217"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-azure-powershell"></a>Démarrage rapide : Créer un enregistrement et une zone Azure DNS à l’aide d’Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Dans ce démarrage rapide, vous allez créer vos premiers enregistrement et zone DNS à l’aide d’Azure PowerShell. Vous pouvez également effectuer ces étapes à l’aide du [portail Azure](dns-getstarted-portal.md) ou [d’Azure CLI](dns-getstarted-cli.md). 

Une zone DNS permet d’héberger les enregistrements DNS d’un domaine particulier. Pour commencer à héberger votre domaine dans le DNS Azure, vous devez créer une zone DNS pour ce nom de domaine. Chaque enregistrement DNS pour votre domaine est ensuite créé à l’intérieur de cette zone DNS. Enfin, pour publier votre zone DNS sur Internet, vous devez configurer les serveurs de noms du domaine. Chacune de ces étapes est décrite ci-dessous.

Azure DNS prend également en charge la création de domaines privés. Pour obtenir des instructions détaillées sur la création de votre première zone DNS privée et enregistrement, consultez [Bien démarrer avec les zones privées Azure DNS à l’aide de PowerShell](private-dns-getstarted-powershell.md).

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell installé localement ou Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Avant de créer la zone DNS, créez un groupe de ressources pour contenir la zone DNS :

```powershell
New-AzResourceGroup -name MyResourceGroup -location "eastus"
```

## <a name="create-a-dns-zone"></a>Création d’une zone DNS

Une zone DNS est créée à l’aide de l’applet de commande `New-AzDnsZone` . L’exemple suivant crée une zone DNS appelée *contoso.xyz* dans le groupe de ressources appelé *MyResourceGroup*. Utilisez l’exemple pour créer une zone DNS, en remplaçant les valeurs indiquées par vos propres valeurs.

```powershell
New-AzDnsZone -Name contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="create-a-dns-record"></a>Créer un enregistrement DNS

Vous pouvez utiliser l’applet de commande `New-AzDnsRecordSet` pour créer des jeux d’enregistrements. L’exemple suivant crée un enregistrement avec le nom relatif « www » dans la zone DNS « contoso.xzy », dans le groupe de ressources « MyResourceGroup ». Le nom complet du jeu d’enregistrements est « www.contoso.xzy ». Le type d’enregistrement est « A », avec l’adresse IP « 10.10.10.10 » et la durée de vie (TTL) est de 3600 secondes.

```powershell
New-AzDnsRecordSet -Name www -RecordType A -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "10.10.10.10")
```

## <a name="view-records"></a>Affichage des enregistrements

Pour répertorier les enregistrements DNS dans votre zone, utilisez :

```powershell
Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup
```

## <a name="test-the-name-resolution"></a>Tester la résolution de nom

Maintenant que vous disposez d’une zone DNS test avec un enregistrement « A » test, vous pouvez tester la résolution de noms avec un outil appelé *nslookup*. 

**Pour tester la résolution de noms DNS**

1. Exécutez l’applet de commande suivante pour obtenir la liste des serveurs de noms pour votre zone :

   ```azurepowershell
   Get-AzDnsRecordSet -ZoneName contoso.xyz -ResourceGroupName MyResourceGroup -RecordType ns
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

```powershell
Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez créé vos premiers enregistrement et zone DNS à l’aide d’Azure PowerShell, vous pouvez créer des enregistrements pour une application web dans un domaine personnalisé.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)

