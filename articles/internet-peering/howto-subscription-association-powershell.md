---
title: Associer un NSA d’homologue à un abonnement Azure à l’aide de PowerShell
titleSuffix: Azure
description: Associer un NSA d’homologue à un abonnement Azure à l’aide de PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 12/15/2020
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3b463293899dc2586404d68145943caff3105e89
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590186"
---
# <a name="associate-peer-asn-to-azure-subscription-using-powershell"></a>Associer un NSA d’homologue à un abonnement Azure à l’aide de PowerShell

Avant de soumettre une demande de Peering, vous devez d’abord associer votre NSA à un abonnement Azure en effectuant les étapes ci-dessous.

Si vous préférez, vous pouvez suivre ce guide à l’aide du [portail](howto-subscription-association-portal.md).

### <a name="working-with-azure-powershell"></a>Utilisation d’Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-peerasn-to-associate-your-asn-with-azure-subscription"></a>Créer un PeerASN pour associer votre NSA à un abonnement Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Vous connecter à votre compte Azure et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="register-for-peering-resource-provider"></a>S’inscrire auprès du fournisseur de ressources de Peering
Inscrivez-vous auprès du fournisseur de ressources de Peering dans votre abonnement à l’aide de la commande ci-dessous. Si vous n’exécutez pas cette commande, les ressources Azure nécessaires pour configurer le Peering ne sont pas accessibles.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

Vous pouvez vérifier l’état de l’inscription à l’aide des commandes ci-dessous :
```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

> [!IMPORTANT]
> Attendez que *RegistrationState* passe à « Registered » avant de continuer. Vous devrez peut-être patienter de cinq à 30 minutes après l’exécution de la commande.

### <a name="update-the-peer-information-associated-with-this-subscription"></a>Mettre à jour les informations d’homologue associées à cet abonnement

Voici un exemple de mise à jour des informations d’homologue.

```powershell
New-AzPeerAsn `
    -Name "Contoso_1234" `
    -PeerName "Contoso" `
    -PeerAsn 1234 `
    -Email noc@contoso.com, support@contoso.com `
    -Phone "+1 (555) 555-5555"
```

> [!NOTE]
> -Name correspond au nom de la ressource, et vous pouvez le choisir librement. -peerName correspond au nom de votre société, et doit ressembler le plus possible à votre profil PeeringDB. Notez que la valeur de -peerName prend en charge uniquement les caractères a-z, A-Z et l’espace.

Un abonnement peut avoir plusieurs NSA. Mettez à jour les informations de Peering pour chaque NSA. Vérifiez que le nom est unique pour chaque NSA.

Les homologues doivent avoir un profil complet et à jour sur [PeeringDB](https://www.peeringdb.com). Nous utilisons ces informations pendant l’inscription pour valider les détails de l’homologue, tels que les informations NOC, les informations de contact technique, leur présence dans les installations de Peering, et ainsi de suite.

Notez que, à la place de **{subscriptionId}** dans la sortie ci-dessus, l’ID d’abonnement réel sera affiché.

## <a name="view-status-of-a-peerasn"></a>Afficher l’état d’un PeerASN

Vérifiez l’état de la validation de NSA à l’aide de la commande ci-dessous :

```powershell
Get-AzPeerAsn
```

Voici un exemple de réponse :
```powershell
PeerContactInfo : Microsoft.Azure.PowerShell.Cmdlets.Peering.Models.PSContactInfo
PeerName        : Contoso
ValidationState : Approved
PeerAsnProperty : 1234
Name            : Contoso_1234
Id              : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/Contoso_1234
Type            : Microsoft.Peering/peerAsns
```

> [!IMPORTANT]
> Attendez que l’état de validation passe à « Approved » avant de soumettre une demande de Peering. Cette approbation peut prendre jusqu’à 12 heures.

## <a name="modify-peerasn"></a>Modifier un PeerAsn
Vous pouvez modifier les informations de contact NOC à tout moment.

Voici un exemple :

```powershell
Set-PeerAsn -Name Contoso_1234 -Email "newemail@test.com" -Phone "1800-000-0000"
```

## <a name="delete-peerasn"></a>Supprimer un PeerAsn
La suppression d’un PeerASN n’est pas prise en charge actuellement. Si vous avez besoin de supprimer un PeerASN, contactez l’assistance [Peering Microsoft](mailto:peering@microsoft.com).

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering direct](howto-direct-powershell.md)
* [Convertir un Peering direct hérité en ressource Azure](howto-legacy-direct-powershell.md)
* [Créer ou modifier un Peering Exchange](howto-exchange-powershell.md)
* [Convertir un Peering Exchange hérité en ressource Azure](howto-legacy-exchange-powershell.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez [FAQ sur le peering Internet](faqs.md).
