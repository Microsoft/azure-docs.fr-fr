---
title: Région ou séries de références SKU non disponibles
description: Certaines séries de références SKU ne sont pas disponibles pour l’abonnement sélectionné de cette région, ce qui peut nécessiter une demande de support pour la gestion des abonnements.
author: stevendotwang
ms.topic: troubleshooting
ms.date: 01/27/2020
ms.author: xingwan
ms.openlocfilehash: 2bbe1dff6bbf8fed481300db95482c46b0e8a7b7
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745279"
---
# <a name="region-or-sku-unavailable"></a>Région ou référence (SKU) non disponible

Cet article décrit comment résoudre le problème quand un abonnement Azure n’a pas accès à une région ou à un SKU de machine virtuelle.

## <a name="symptoms"></a>Symptômes

Quand vous déployez une machine virtuelle, vous recevez l’un des messages d’erreur suivants :

```
Code: SkuNotAvailable
Message: The requested size for resource '<resource>' is currently not available in location 
'<location>' zones '<zone>' for subscription '<subscriptionID>'. Please try another size or 
deploy to a different location or zones. See https://aka.ms/azureskunotavailable for details.
```

```
Message: Your subscription doesn’t support virtual machine creation in <location>. Choose a 
different location. Supported locations are <list of locations>
```

```
Code: NotAvailableForSubscription
Message: This size is currently unavailable in this location for this subscription
```

Quand vous achetez des instances de machine virtuelle réservées, vous recevez l’un des messages d’erreur suivants :

```
Message: Your subscription doesn’t support virtual machine reservation in <location>. Choose a 
different location. Supported locations are: <list of locations>  
```

```
Message: This size is currently unavailable in this location for this subscription
```

Quand vous créez une demande de support pour augmenter le quota de cœurs de calcul, une région ou une famille de références SKU n’est pas disponible pour la sélection.

## <a name="solution"></a>Solution

Tout d’abord, nous vous recommandons d’utiliser une autre région ou référence qui répond aux besoins de votre entreprise.

Si vous ne parvenez pas à trouver une région ou une référence appropriée, créez une **demande de support** [Gestion des abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en suivant ces étapes :

1. Dans le menu [Portail Azure](https://portal.azure.com), sélectionnez **Aide + support**. Puis sélectionnez **Nouvelle demande de support**.

1. Dans **Fonctions de base**, comme **Type de problème**, sélectionnez **Gestion des abonnements**.

1. Sélectionnez un **Abonnement** et entrez une brève description dans **Résumé**.

   ![Onglet Fonctions de base de Nouvelle demande de support](./media/SKU-series-unavailable/support-request-basics.png)

1. Comme **Type de problème**, choisissez **Sélectionner un type de problème**.

1. Pour **Sélectionner un type de problème**, choisissez une option, par exemple **Impossible d’accéder à mon abonnement ou à la ressource** > **Mon problème n’est pas répertorié ci-dessus**. Sélectionnez **Enregistrer**.

   ![Spécifier un problème pour la demande](./media/SKU-series-unavailable/support-request-select-problem-type.png)

1. Sélectionnez **Suivant : Solutions** pour explorer les solutions possibles. Si nécessaire, sélectionnez **Suivant : Détails** pour continuer.

1. Entrez les informations complémentaires que vous pouvez fournir, ainsi que vos coordonnées.

1. Sélectionnez **Revoir + créer**. Après avoir vérifié vos informations, sélectionnez **Créer** pour créer la demande.

## <a name="send-us-your-suggestions"></a>Envoyez-nous vos suggestions.

Nous sommes ouverts aux commentaires et suggestions ! Envoyez-nous vos [suggestions](https://feedback.azure.com/forums/266794-support-feedback). Vous pouvez aussi nous contacter sur [Twitter](https://twitter.com/azuresupport) ou par le biais de la [page de questions Microsoft Q&A](/answers/products/azure).

## <a name="learn-more"></a>En savoir plus

[FAQ du support Azure](https://azure.microsoft.com/support/faq)