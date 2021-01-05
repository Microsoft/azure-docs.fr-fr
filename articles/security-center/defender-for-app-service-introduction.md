---
title: Azure Defender pour App Service - avantages et fonctionnalités
description: Découvrez les avantages et les fonctionnalités d’Azure Defender pour App Service.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: bb0e073d5ccf73434d05c801b9a8727c1d19fa47
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96122238"
---
# <a name="introduction-to-azure-defender-for-app-service"></a>Présentation d’Azure Defender pour App Service

Azure App Service est une plateforme entièrement gérée pour la création et l’hébergement de vos applications Web et API sans vous soucier de la gestion de l’infrastructure. Elle fournit des informations sur la gestion, la surveillance et l’exploitation afin de répondre à des exigences en matière de performances, de sécurité et de conformité de classe entreprise. Pour en savoir plus, consultez [Azure App Service](https://azure.microsoft.com/services/app-service/).

**Azure Defender pour App Service** utilise l’échelle du cloud pour identifier les attaques ciblant les applications exécutées sur App Service. Les attaquants sondent les applications web pour trouver et exploiter les faiblesses. Avant d’être routées vers des environnements spécifiques, les demandes adressées aux applications s’exécutant dans Azure passent par plusieurs passerelles, où elles sont inspectées et journalisées. Ces données sont ensuite utilisées pour identifier les codes malveillants exploitant une faille de sécurité ainsi que les attaquants, et pour découvrir de nouveaux modèles qui seront utilisés ultérieurement.

En utilisant la visibilité dont Azure bénéficie en tant que fournisseur de cloud, Security Center analyse les journaux internes d’App Service pour identifier la méthodologie d’attaque sur plusieurs cibles. Les attaques distribuées et les attaques par analyse généralisée sont des exemples de méthodologie. En général, ce type d’attaque provient d’un petit sous-ensemble d’adresses IP, et présente des modèles de progression vers des points de terminaison similaires sur plusieurs hôtes. Les attaques recherchent une page ou un plug-in vulnérable et ne peuvent pas être identifiées du point de vue d’un seul hôte.


## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|[Azure Defender pour App Service](azure-defender.md) est facturé comme indiqué sur [la page de tarification](security-center-pricing.md)|
|Plans App Service pris en charge :|![Oui](./media/icons/yes-icon.png) De base, Standard, Premium, Isolé ou Linux<br>![Non](./media/icons/no-icon.png) Gratuit, Partagé ou Consommation<br>[En savoir plus sur les plans App Service](https://azure.microsoft.com/pricing/details/app-service/plans/)|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||

## <a name="what-does-azure-defender-for-app-service-protect"></a>Quels sont les éléments protégés par Azure Defender pour App Service ?

Une fois le plan App Service activé, Security Center évalue les ressources couvertes par votre plan App Service et génère des suggestions en matière de sécurité en fonction de ses résultats. Security Center protège l’instance de machine virtuelle dans laquelle votre App Service s’exécute et l’interface de gestion. Il surveille également les demandes et réponses échangées avec vos applications s’exécutant dans App Service.

Si vous exécutez un plan App Service Windows, Security Center a également accès aux machines virtuelles et aux bacs à sable sous-jacents. Couplée aux données de journal mentionnées ci-dessus, l’infrastructure peut raconter l’histoire, depuis une nouvelle attaque lâchée dans la nature à la compromission des machines des clients. Par conséquent, même si Security Center est déployé après qu’une application web a été attaquée, il est capable de détecter les attaques en cours.


## <a name="protect-your-azure-app-service-web-apps-and-apis"></a>Protéger vos applications Web et API Azure App Service
Pour protéger votre plan Azure App Service avec Azure Defender pour App Service :

- Vérifiez que vous disposez d’un plan App Service pris en charge et associé aux ordinateurs dédiés. Les plans pris en charge sont répertoriés ci-dessus dans la section [Disponibilité](#availability).

- Activez **Azure Defender** dans votre abonnement (vous pouvez éventuellement activer uniquement le plan **Azure Defender pour App Service**), comme décrit dans [Niveaux tarifaires d’Azure Security Center](security-center-pricing.md)

Security Center étant intégré en mode natif avec App Service, aucun déploiement ou intégration ne sont nécessaires. L’intégration est transparente.

>[!NOTE]
> La page de tarification et des paramètres répertorie un certain nombre d’instances pour votre **quantité de ressources**. Cela représente le nombre total d’instances de calcul, dans tous les plans App Service sur cet abonnement, en cours d’exécution lorsque vous avez ouvert la page du niveau tarifaire.
>
> Azure App Service offre une variété de plans. Votre plan App Service définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application web. Ils sont équivalents aux batteries de serveurs dans l’hébergement web conventionnel. Une ou plusieurs applications peuvent être configurées pour s’exécuter sur les mêmes ressources informatiques (ou dans le même plan App Service).
>
>Pour valider le nombre, reportez-vous à la section « Plans App Service » dans le portail Azure, où vous pouvez voir le nombre d’instances de calcul utilisées par chaque plan. 



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert Azure Defender pour App Service. 

Pour des informations connexes, consultez les articles suivants : 

- Qu’une alerte soit générée par Security Center ou reçue par Security Center à partir d’un autre produit de sécurité, vous pouvez l’exporter. Pour exporter vos alertes vers Azure Sentinel (ou un système SIEM tiers) ou tout autre outil externe, suivez les instructions fournies dans [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](export-to-siem.md).
- Pour obtenir la liste des alertes Azure App Service, consultez la [table de référence des alertes](alerts-reference.md#alerts-azureappserv).
- Pour plus d’informations sur les plans App Service, consultez [Plans App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).
- > [!div class="nextstepaction"]
    > [Activer Azure Defender](security-center-pricing.md)