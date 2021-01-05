---
title: Azure Front Door - Prise en charge des domaines génériques
description: Cet article vous aide à comprendre la prise en charge par Azure Front Door du mappage et de la gestion des domaines génériques dans la liste des domaines personnalisés.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/29/2020
ms.author: duau
ms.openlocfilehash: 18504f1ed4200889b20c9608c9c0ad2c13c9aaa5
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425760"
---
# <a name="wildcard-domains"></a>Domaines génériques

En dehors des domaines et sous-domaines apex, vous pouvez également mapper un domaine générique aux hôtes front-end ou aux domaines personnalisés de votre profil Azure Front Door. Le fait d’avoir des domaines génériques dans votre configuration Azure Front Door simplifie le comportement du routage du trafic de plusieurs sous-domaines pour une API, une application ou un site web à partir d’une même règle d’acheminement. Vous n’avez pas à modifier la configuration pour ajouter et/ou spécifier chaque sous-domaine séparément. Par exemple, vous pouvez définir le routage pour `customer1.contoso.com`, `customer2.contoso.com` et `customerN.contoso.com` à l’aide de la même règle d’acheminement en ajoutant le domaine générique `*.contoso.com`.

Les scénarios améliorés avec la prise en charge des domaines génériques comprennent les suivants :

- Vous n’avez pas besoin d’intégrer chaque sous-domaine dans votre profil Azure Front Door, puis d’activer HTTPS pour lier un certificat pour chaque sous-domaine.
- Vous n’avez plus besoin de changer votre configuration Front Door de production si une application ajoute un nouveau sous-domaine. Auparavant, vous deviez ajouter le sous-domaine, lui lier un certificat, attacher une stratégie de pare-feu d’applications web (WAF), puis ajouter le domaine à différentes règles de routage.

> [!NOTE]
> Actuellement, l’ajout de domaines de caractères génériques dans Azure DNS n’est pris en charge que via l’API, PowerShell et Azure CLI. La prise en charge de l’ajout et de la gestion des domaines génériques dans le portail Azure n’est pas disponible.

## <a name="adding-wildcard-domains"></a>Ajout de domaines génériques

Vous pouvez ajouter un domaine générique sous la section pour les hôtes frontaux ou les domaines. Comme pour les sous-domaines, Azure Front Door vérifie qu’il existe un mappage d’enregistrement CNAME pour votre domaine générique. Ce mappage DNS peut être un mappage d’enregistrement CNAME direct, comme `*.contoso.com` mappé à `contoso.azurefd.net`. Ou vous pouvez utiliser le mappage temporaire afdverify. Par exemple, `afdverify.contoso.com` mappé à `afdverify.contoso.azurefd.net` valide le mappage d’enregistrement CNAME pour le domaine générique.

> [!NOTE]
> Azure DNS prend en charge les enregistrements génériques.

Vous pouvez ajouter autant de sous-domaines à un seul niveau que le domaine générique dans les hôtes frontaux, jusqu’à la limite des hôtes frontaux. Cette fonctionnalité peut être nécessaire pour :

- Définir d’un itinéraire différent pour un sous-domaine par rapport au reste des domaines (comparé au domaine générique).

- Avoir une stratégie de pare-feu d'applications web différente pour un sous-domaine spécifique. Par exemple, `*.contoso.com` permet d’ajouter `foo.contoso.com` sans devoir à nouveau prouver la propriété du domaine. Mais cela n’autorise pas `foo.bar.contoso.com`, car il ne s’agit pas d’un sous-domaine de niveau unique de `*.contoso.com`. Pour ajouter `foo.bar.contoso.com` sans validation supplémentaire de la propriété du domaine, vous devez ajouter `*.bar.contosonews.com`.

Vous pouvez ajouter des domaines génériques et leurs sous-domaines avec certaines limitations :

- Si un domaine générique est ajouté à un profil Azure Front Door :
  - Le domaine générique ne peut pas être ajouté à un autre profil Azure Front Door.
  - Les sous-domaines de premier niveau du domaine générique ne peuvent pas être ajoutés à un autre profil Azure Front Door ou un profil Azure Content Delivery Network.
- Si un sous-domaine d’un domaine générique est déjà ajouté à un profil Azure Front Door ou un profil Azure Content Delivery Network, le domaine générique ne peut pas être utilisé pour un autre profil Azure Front Door.
- Si deux profils (Azure Front Door ou Azure Content Delivery Network de Microsoft) possèdent plusieurs sous-domaines d’un domaine racine, les domaines génériques ne peuvent être ajoutés à aucun des deux profils.

## <a name="certificate-binding"></a>Liaison de certificats

Pour accepter le trafic HTTPS sur votre domaine générique, vous devez activer HTTPS sur le domaine générique. La liaison de certificat pour un domaine générique requiert un certificat générique. Autrement dit, le nom du sujet du certificat doit également comporter le nom de domaine générique.

> [!NOTE]
> Actuellement, seule l’option consistant à utiliser votre propre certificat SSL personnalisé est disponible pour activer HTTPS pour les domaines génériques. Les certificats Azure Front Door gérés ne peuvent pas être utilisés pour les domaines génériques.

Vous pouvez choisir d’utiliser le même certificat générique à partir d’Azure Key Vault ou des certificats gérés Azure Front Door pour les sous-domaines.

Si un sous-domaine est ajouté pour un domaine générique auquel un certificat est déjà associé, vous ne pouvez pas désactivé le protocole HTTPS pour le sous-domaine. Le sous-domaine utilise la liaison de certificat pour le domaine générique, à moins qu’une autre clé Key Vault ou un autre certificat géré par Azure Front Door ne le remplace.

## <a name="waf-policies"></a>Stratégies WAF

Les stratégies de pare-feu d’applications web peuvent être attachées à des domaines génériques similaires aux autres domaines. Une stratégie de pare-feu d’applications web peut être appliquée au sous-domaine d’un domaine générique. Pour les sous-domaines, vous devez spécifier la stratégie de pare-feu d’applications web à utiliser, même s’il s’agit de la même stratégie que celle du domaine générique. Les sous-domaines n’héritent *pas* automatiquement de la stratégie de pare-feu d’applications web du domaine générique.

Si vous ne souhaitez pas qu’une stratégie de pare-feu d'applications web s’exécute pour un sous-domaine, vous pouvez créer une stratégie vide sans ensemble de règles géré ou personnalisé.

## <a name="routing-rules"></a>Règles de routage

Lors de la configuration d’une règle d’acheminement, vous pouvez sélectionner un domaine générique comme hôte frontend. Vous pouvez également avoir un comportement de routage différent pour le domaine générique et pour les sous-domaines. Comme le décrit l’article [Fonctionnement des correspondances d’itinéraires Azure Front Door](front-door-route-matching.md), la correspondance la plus spécifique pour le domaine entre différentes règles d’acheminement est choisie au moment de l’exécution.

> [!IMPORTANT]
> Vous devez avoir des modèles de chemin d’accès correspondants à travers vos règles de routage, sans quoi vos clients verront des échecs. Par exemple, supposons que vous avez deux règles de routage, l’itinéraire 1 (`*.foo.com/*` mappé au pool principal A) et l’itinéraire 2 (`/bar.foo.com/somePath/*` mappé au pool principal B). Une requête arrive alors pour `bar.foo.com/anotherPath/*`. Azure Front Door sélectionne l’itinéraire 2 au motif d’une correspondance de domaine plus spécifique, mais ne trouve aucun modèle de chemin d’accès correspondant dans les itinéraires.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un profil Azure Front Door](quickstart-create-front-door.md).
- Découvrez comment [ajouter un domaine personnalisé à une instance Azure Front Door](front-door-custom-domain.md).
- Découvrez [comment activer le protocole HTTPS pour un domaine personnalisé](front-door-custom-domain-https.md).
