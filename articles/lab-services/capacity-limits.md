---
title: Limites de capacité dans Azure Lab Services
description: Découvrez les limites de capacité (limites de machines virtuelles) dans Azure Lab Services.
ms.topic: conceptual
ms.date: 06/26/2020
ms.openlocfilehash: 9866628cd11ec8df67e6fe16ae8806f0f30ae9a1
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491016"
---
# <a name="capacity-limits-in-azure-lab-services"></a>Limites de capacité dans Azure Lab Services
Azure Lab Services a des limites de capacité par défaut sur les abonnements Azure pour respecter les limitations de quota d’Azure Compute ainsi que pour atténuer la fraude. Tous les abonnements Azure ont une limite de capacité initiale, qui peut varier en fonction du type d’abonnement, du nombre de cœurs de calcul standard et des cœurs de GPU disponibles dans Azure Lab Services. Ainsi, il existe une limite au nombre de machines virtuelles que vous pouvez créer dans vos labos avant toute demande d’augmentation de cette limite.  

Si vous êtes proche ou si vous avez atteint la limite du nombre de cœurs de machines virtuelles de votre abonnement, vous voyez s’afficher des messages d’Azure Lab Services quand vous essayez d’effectuer des actions visant à créer des machines virtuelles supplémentaires. Par exemple : 

- Créer un labo
- Publier un labo
- Ajuster la capacité d’un labo pour ajouter des machines virtuelles à un labo existant

Ces actions peuvent également être désactivées si vous avez déjà atteint la limite du nombre de cœurs. 

![Limites du nombre de cœurs - Message d’avertissement](./media/capacity-limits/warning-message.png)

## <a name="subscriptions-with-default-limit-of-zero-cores"></a>Abonnements avec une limite par défaut de zéro cœur
Certains types d’abonnement rares qui sont plus couramment utilisés pour la fraude peuvent avoir une limite par défaut de 0 cœur standard et de 0 cœur GPU. Si vous utilisez l’un de ces types d’abonnement, l’administrateur qui crée votre compte labo doit demander une augmentation de limite pour que vous puissiez utiliser Azure Lab Services. 

L’administrateur peut suivre les étapes ci-après pour demander une augmentation de limite :  

1.  Dans votre abonnement, [créez un compte labo](tutorial-setup-lab-account.md).
2.  Dans la page **Vue d’ensemble** du compte labo, cliquez sur le bouton **Demander une augmentation de limite** situé en haut. 
3.  Suivez les étapes du formulaire pour envoyer une demande de support afin d’augmenter la limite.

## <a name="request-a-limit-increase"></a>Demander une augmentation de limite
Si vous atteignez la limite du nombre de cœurs, vous pouvez demander une augmentation de limite pour continuer à utiliser Azure Lab Services. Le processus de demande est un point de contrôle qui permet de vérifier si votre abonnement n’est pas impliqué dans des cas de fraude ou de déploiements involontaires et soudains à grande échelle.

Les messages relatifs à la limite du nombre de cœurs de machines virtuelles dans le portail Azure Lab Services incluent un lien permettant de demander une augmentation de limite. Le lien ouvre un nouvel onglet de navigateur dans lequel vous pouvez créer une demande de support. Les informations relatives au type de problème, à l’abonnement et au type de quota sont automatiquement renseignées, comme illustré dans l’image suivante : 

![Nouvelle demande de support](./media/capacity-limits/new-support-request.png)


Vous êtes ensuite invité à fournir des informations supplémentaires sur l’augmentation de limite. Dans le champ **Description**, indiquez les informations suivantes :

- Ce que vous essayez de faire (par exemple créer un labo pour donner un cours d’informatique, exécuter un hackathon, etc.)
- Taille de machine virtuelle que vous utilisez pour ce labo
- Nombre de machines virtuelles nécessaires

Une fois que vous enverrez la demande de support, nous la passerons en revue. Si nécessaire, nous vous contacterons pour obtenir des détails supplémentaires. 

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article suivant :
- [Guide de l’administrateur : dimensionnement des machines virtuelles](administrator-guide.md#vm-sizing).
- [Forum aux questions](classroom-labs-faq.md).