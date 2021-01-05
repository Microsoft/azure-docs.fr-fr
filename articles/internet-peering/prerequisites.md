---
title: Prérequis pour configurer le peering avec Microsoft
titleSuffix: Azure
description: Prérequis pour configurer le peering avec Microsoft
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: bc45bc8809eabe75902b602835ea595b56ff3cf8
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586838"
---
# <a name="prerequisites-to-set-up-peering-with-microsoft"></a>Prérequis pour configurer le peering avec Microsoft

Vérifiez que les conditions préalables ci-dessous sont remplies avant d’effectuer une nouvelle demande de peering ou de convertir un peering hérité en ressource Azure.

## <a name="azure-related-prerequisites"></a>Conditions préalables relatives à Azure
* **Compte Microsoft Azure :** Si vous ne possédez pas de compte Microsoft Azure, créez un [compte Microsoft Azure](https://azure.microsoft.com/free). Un abonnement Microsoft Azure valide et actif est requis pour configurer le peering, car les peerings sont modélisées en tant que ressources dans les abonnements Azure. Il est important de noter ce qui suit :
    * Les types de ressources Azure utilisés pour configurer le peering sont toujours des produits Azure gratuits, c’est-à-dire que vous n’êtes pas facturé pour la création d’un compte Azure ou d’un abonnement ou l’accès aux ressources Azure **PeerAsn** et **Peering** pour configurer le peering. À ne pas confondre avec l’accord de peering pour le peering direct conclu entre vous et Microsoft et dont les conditions sont explicitement définies avec notre équipe de peering. Contactez le [Peering Microsoft](mailto:peering@microsoft.com) si vous avez des questions à ce sujet.
    * Vous pouvez utiliser le même abonnement Azure pour accéder à d’autres produits ou services cloud Azure qui peuvent être gratuits ou payants. Si vous accédez à un produit payant, des frais vous seront facturés.
    * Si vous créez un nouveau compte et/ou un nouvel abonnement Azure, vous pouvez éventuellement bénéficier d'un crédit Azure gratuit pendant une période d'essai que vous pouvez utiliser pour essayer les services cloud Azure. Si vous souhaitez obtenir plus d’informations, consultez le [compte Microsoft Azure](https://azure.microsoft.com/free).

* **Associer un ASN de pair :** Avant d’effectuer une demande de peering, associez d’abord votre ASN et vos coordonnées à votre abonnement. Suivez les instructions indiquées dans [Associer un ASN de pair à un abonnement Azure](howto-subscription-association-powershell.md).

## <a name="other-prerequisites"></a>Autres conditions préalables
* **Profil PeeringDB :** Les pairs doivent avoir un profil complet et à jour sur [PeeringDB](https://www.peeringdb.com). Nous utilisons ces informations dans notre système d’inscription pour valider les détails du pair, tels que les informations NOC, les informations de contact technique ou encore leur présence dans les installations de Peering, etc.

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un peering direct à l’aide du portail](howto-direct-portal.md)
* [Convertir un peering direct hérité en ressource Azure à l’aide du portail](howto-legacy-direct-portal.md)
* [Créer ou modifier un Peering Exchange à l’aide du portail](howto-exchange-portal.md)
* [Convertir un peering Exchange hérité en ressource Azure à l’aide du portail](howto-legacy-exchange-portal.md)