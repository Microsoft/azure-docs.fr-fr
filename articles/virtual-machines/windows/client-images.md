---
title: Utiliser des images du client Windows dans Azure
description: Comment utiliser les avantages de l’abonnement Visual Studio pour déployer Windows 7, Windows 8 ou Windows 10 dans Azure pour des scénarios de développement et/ou de test
author: cynthn
ms.subservice: imaging
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: 26bfd9a65cd3e6d36b8190e87d382543cc7f0f60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87292051"
---
# <a name="use-windows-client-in-azure-for-devtest-scenarios"></a>Utilisation d’un client Windows dans Azure pour les scénarios de développement et/ou test
Vous pouvez utiliser Windows 7, Windows 8 ou Windows 10 Entreprise (x64) dans Azure pour des scénarios de développement / de test à condition de disposer d’un abonnement Visual Studio (anciennement MSDN) approprié. Cet article décrit les conditions d’éligibilité pour les clients Windows 7, Windows 8.1 et Windows 10 Entreprise en cours d’exécution dans Azure et l’utilisation des images de galerie Azure suivantes.

![Détails de l’image dans le portail Azure](./media/client-images/windows-client-msdn-images.png) 

> [!NOTE]
> Pour l’image Windows 10 Pro et Windows 10 Pro N dans la galerie Azure, consultez la page [Guide pratique pour déployer Windows 10 sur Azure avec des droits d’hébergement multilocataire](windows-desktop-multitenant-hosting-deployment.md)
>![Détails de l’image Pro du portail Azure](./media/client-images/windows-client-pro-images.png) 
>

## <a name="subscription-eligibility"></a>Admissibilité à un abonnement
Les abonnés Visual Studio actifs (les personnes qui ont acquis une licence d’abonnement Visual Studio) peuvent utiliser un client Windows à des fins de développement et de tests. Un client Windows peut être utilisé sur votre propre matériel et vos machines virtuelles Azure en cours d’exécution dans n’importe quel type d’abonnement Azure. Il ne peut pas être déployé ou utilisé dans Azure pour la production normale ou utilisé par des personnes qui ne sont pas des abonnés Visual Studio actifs.

Pour vous faciliter la tâche, certaines images Windows 10 sont disponibles dans Azure Gallery, sous [eligible dev/test offers](#eligible-offers). Les abonnés Visual Studio dans n’importe quel type d’offre peuvent également [préparer et créer correctement](prepare-for-upload-vhd-image.md) une image 64 bits de Windows 7, Windows 8 ou Windows 10, puis la [charger dans Azure](upload-generalized-managed.md). L’utilisation reste limitée au développement/test par les abonnés Visual Studio actifs.

## <a name="eligible-offers"></a>Offres admissibles
Le tableau suivant décrit en détail les ID d’offres admissibles pour le déploiement de Windows 10 par le biais de la galerie Azure. Les images Windows 10 sont uniquement visibles par les offres suivantes. Les abonnés Visual Studio qui doivent exécuter le client Windows dans un autre type d’offre doivent [préparer et créer correctement](prepare-for-upload-vhd-image.md) une image 64 bits de Windows 7, Windows 8 ou Windows 10, [puis la charger dans Azure](upload-generalized-managed.md).

| Nom de l’offre | Numéro de l’offre | Images de client disponibles |
|:--- |:---:|:---:|
| [Dev/Test – Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/) |0023P |Windows 10 |
| [Abonnés Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/) |0029P |Windows 10 |
| [Abonnés Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/) |0059P |Windows 10 |
| [Abonnés Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/) |0060P |Windows 10 |
| [Visual Studio Premium avec MSDN (avantage)](https://azure.microsoft.com/offers/ms-azr-0061p/) |0061P |Windows 10 |
| [Abonnés Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) |0063P |Windows 10 |
| [Abonnés Visual Studio Enterprise (BizSpark)](https://azure.microsoft.com/offers/ms-azr-0064p/) |0064P |Windows 10 |
| [Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/) |0148P |Windows 10 |

## <a name="check-your-azure-subscription"></a>Vérification de votre abonnement Azure
Si vous ne connaissez pas votre ID d’offre, vous pouvez l’obtenir via le portail Azure de l’une des manières suivantes :  

- Sur la fenêtre *Abonnements* :

  ![Détails de l’ID de l’offre dans le portail Azure](./media/client-images/offer-id-azure-portal.png) 

- Vous pouvez également cliquer sur **Facturation**, puis sur votre ID d’abonnement. L’ID de l’offre apparaît sur la fenêtre *Facturation*.

Vous pouvez également voir l’ID d’offre dans [l’onglet « Abonnements »](https://account.windowsazure.com/Subscriptions) du portail de compte Azure :

![Détails de l’ID de l’offre dans le portail de compte Azure](./media/client-images/offer-id-azure-account-portal.png) 

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez désormais déployer vos machines virtuelles à l’aide de [PowerShell](quick-create-powershell.md), de [modèles Resource Manager](ps-template.md) ou de [Visual Studio](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
