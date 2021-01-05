---
title: Créer un contrôleur de données Azure Arc dans le portail Azure
description: Créer un contrôleur de données Azure Arc dans le portail Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 297efa83fb1563e3a360f652a6ac1bc2b1679998
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930640"
---
# <a name="create-an-azure-arc-data-controller-in-the-azure-portal"></a>Créer un contrôleur de données Azure Arc dans le portail Azure

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="introduction"></a>Introduction

Vous pouvez utiliser le portail Azure pour créer un contrôleur de données Azure Arc.

La plupart des expériences de création pour Azure Arc commencent dans le portail Azure, même si la ressource à créer ou à gérer se trouve en dehors de l’infrastructure Azure. Le modèle d’expérience utilisateur dans ces cas, en particulier lorsqu’il n’existe aucune connexion directe entre Azure et votre environnement, consiste à utiliser le portail Azure pour générer un script qui peut ensuite être téléchargé et exécuté dans votre environnement afin d’établir une connexion sécurisée à Azure. Par exemple, les serveurs activés par Azure Arc suivent ce modèle pour [créer des serveurs activés par Arc](../servers/onboard-portal.md).

Actuellement, étant donné que préversion ne prend en charge que le mode connecté indirect des services de données activés par Azure Arc, vous pouvez utiliser le portail Azure pour générer un bloc-notes qui peut ensuite être téléchargé et exécuté dans Azure Data Studio sur votre cluster Kubernetes. À l’avenir, lorsque le mode de connexion directe sera disponible, vous pourrez approvisionner le contrôleur de données directement à partir du portail Azure. Pour en savoir plus, consultez [Modes de connectivité](connectivity.md).

## <a name="use-the-azure-portal-to-create-an-azure-arc-data-controller"></a>Utiliser le portail Azure pour créer un contrôleur de données Azure Arc

Pour créer un contrôleur de données Azure Arc à l’aide du portail Azure et d’Azure Data Studio, procédez comme suit.

1. Commencez par vous connecter à la [place de marché du portail Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/MarketplaceOffersBlade/selectedMenuItemId/home/searchQuery/azure%20arc%20data%20controller).  Les résultats de la recherche dans la place de marché sont filtrés pour afficher le « contrôleur de données Azure Arc ».
2. Si la première étape n’a pas entré les critères de recherche. Accédez aux résultats de la recherche, puis cliquez sur « Contrôleur de données Azure Arc ».
3. Sélectionnez la vignette du Contrôleur de données Azure de la place de marché.
4. Cliquez sur le bouton **Créer**.
5. Examinez la configuration requise pour créer un contrôleur de données Azure Arc et installez tous les logiciels prérequis manquants, tels qu’Azure Data Studio et kubectl.
6. Cliquez sur le bouton **Détails du contrôleur de données**.
7. Choisissez un abonnement, un groupe de ressources et un emplacement Azure comme vous le feriez pour toute autre ressource que vous créeriez dans le portail Azure. Dans ce cas, l’emplacement Azure que vous sélectionnez correspond à l’emplacement où les métadonnées relatives à la ressource seront stockées.  La ressource elle-même sera créée sur l’infrastructure de votre choix. Elle n’a pas besoin d’être sur l’infrastructure Azure.
8. Entrez un nom pour votre contrôleur de données.
9. Seul le mode de connexion indirecte est pris en charge.
10. Sélectionnez un profil de configuration de déploiement.
11. Cliquez sur le bouton **Ouvrir dans Azure Studio**.
12. L’écran suivant affiche un résumé de vos sélections et un bloc-notes généré.  Vous pouvez cliquer sur le bouton **Télécharger le bloc-notes d’approvisionnement** pour télécharger le bloc-notes.
13. Ouvrez le bloc-notes dans Azure Data Studio, puis cliquez sur le bouton **Exécuter tout** en haut.
14. Suivez les invites et les instructions du bloc-notes pour terminer la création du contrôleur de données.

## <a name="monitoring-the-creation-status"></a>Surveillance de l’état de la création

La création du contrôleur prend plusieurs minutes. Vous pouvez surveiller la progression dans une autre fenêtre du terminal à l’aide des commandes suivantes :

> [!NOTE]
>  Les exemples de commandes ci-dessous supposent que vous avez créé un contrôleur de données et un espace de noms Kubernetes avec le nom « arc ».  Si vous avez utilisé un nom de contrôleur de données ou d’espace de noms différent, vous pouvez remplacer « arc » par votre nom.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Vous pouvez également vérifier l’état de la création de n’importe quel pod en exécutant une commande comme celle qui figure ci-dessous.  C’est particulièrement utile pour résoudre les problèmes.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Résolution des problèmes de création

Si vous rencontrez des problèmes avec la création, consultez le [Guide de résolution des problèmes](troubleshoot-guide.md).
