---
title: Contrat Standard pour le marketplace commercial de Microsoft
description: Contrat Standard pour la Place de marché Azure et AppSource dans l’Espace partenaires
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 05/20/2020
ms.openlocfilehash: 20a257bde6022249fd7b2ab875b94f356234b490
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94488874"
---
# <a name="standard-contract-for-microsoft-commercial-marketplace"></a>Contrat Standard pour le marketplace commercial de Microsoft

Microsoft offre un contrat Standard pour le marketplace commercial de Microsoft. Cela permet de simplifier le processus d’approvisionnement aux clients, de réduire la complexité juridique à laquelle sont confrontés les fournisseurs de logiciels et de faciliter les transactions sur la place de marché. Plutôt que d’élaborer des conditions générales personnalisées, en tant qu’éditeur de la place de marché commerciale, vous pouvez choisir de proposer votre logiciel dans le cadre du [contrat Standard](https://go.microsoft.com/fwlink/?linkid=2041178), que les clients ne doivent accepter qu’une seule fois.

Les conditions générales d’une offre sont définies lors de la création de l’offre dans l’Espace partenaires. Vous pouvez choisir d’utiliser le contrat Standard pour la place de marché commerciale de Microsoft au lieu de fournir vos propres conditions générales.

>[!Note]
>Après avoir publié une offre à l’aide du contrat Standard pour la place de marché commerciale de Microsoft, vous ne pouvez pas utiliser vos propres conditions générales. Vous proposez votre solution avec le contrat Standard *ou* avec vos propres conditions générales. Les conditions générales personnalisées sont définies au niveau de l’offre et s’appliquent à tous les plans ; écrivez vos conditions générales dans la page **Propriétés** de votre offre dans l’Espace partenaires. Si vous souhaitez modifier les conditions du contrat Standard, vous pouvez le faire par le biais des Modifications du contrat Standard.

## <a name="standard-contract-amendments"></a>Contrat Standard | Modifications

Les modifications du contrat Standard permettent aux éditeurs de sélectionner le contrat Standard par souci de simplicité, et avec des conditions personnalisées pour leur produit ou entreprise. Les clients sont invités à examiner les modifications apportées au contrat, puis à accepter le contrat Standard de Microsoft.

Deux types de modifications sont proposées aux éditeurs de la place de marché commerciale :

* Modifications universelles : Ces modifications sont appliquées de manière universelle au contrat Standard de tous les clients. Les modifications universelles s’affichent pour chaque client de l’offre dans le flux d’achat. Les clients doivent accepter les conditions du contrat Standard et la modification avant de pouvoir utiliser votre offre.

* Modifications personnalisées : Il s’agit de modifications spéciales apportées au contrat Standard qui ciblent certains clients uniquement via les ID de locataires Azure. Les éditeurs peuvent choisir le locataire qu'ils souhaitent cibler. Seuls les clients du locataire recevront les conditions personnalisées dans le flux d’achat de l’offre.  Les clients doivent accepter les conditions du contrat Standard et la ou les modifications avant de pouvoir utiliser votre offre.

>[!Note]
>Ces deux types de modifications s’ajoutent l’un à l’autre. Les clients ciblés par les modifications personnalisées reçoivent aussi la modification universelle apportée au contrat Standard lors de l’achat. Les modifications sont limitées à 4 000 caractères, espaces comprises.

Vous pouvez tirer parti du contrat Standard pour la place de marché commerciale de Microsoft pour les types d’offre suivants :  Applications Azure (modèles de solution et applications managées), machines virtuelles et SaaS.

## <a name="customer-experience"></a>Expérience client

Pendant l’expérience de découverte dans la Place de marché Azure ou AppSource, les clients pourront voir les conditions associées à l’offre sous la forme du contrat Standard pour la place de marché commerciale de Microsoft et toute modification universelle.

![Expérience de découverte du client sur le portail Azure.](media/marketplace-publishers-guide/azure-discovery-process.png)

Pendant le processus d’achat sur le portail Azure, les clients pourront voir les conditions associées à l’offre sous la forme du contrat Standard pour la place de marché commerciale de Microsoft et toute modification universelle et/ou spécifique au locataire.

![Expérience d’achat du client sur le portail Azure.](media/marketplace-publishers-guide/azure-purchase-process.png)

## <a name="api"></a>API

Les clients peuvent utiliser Get-AzureRmMarketplaceTerms pour récupérer les conditions d’une offre et l’accepter. Le contrat Standard et les modifications qui s'y rapportent sont renvoyés dans la sortie de la cmdlet.
