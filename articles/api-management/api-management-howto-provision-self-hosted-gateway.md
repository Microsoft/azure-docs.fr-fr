---
title: Approvisionner une passerelle auto-hébergée dans Gestion des API | Microsoft Docs
description: Découvrez comment approvisionner une passerelle auto-hébergée dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/31/2020
ms.author: apimpm
ms.openlocfilehash: e79248e16ca21ae84022f8ac7f280d93f489c6a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87050349"
---
# <a name="provision-a-self-hosted-gateway-in-azure-api-management"></a>Approvisionner une passerelle auto-hébergée dans Gestion des API Azure

L’approvisionnement d’une ressource de passerelle dans votre instance Gestion des API Azure est un composant requis pour le déploiement d’une passerelle auto-hébergée. Cet article décrit les étapes d’approvisionnement d’une ressource de passerelle dans Gestion des API.

## <a name="prerequisites"></a>Conditions préalables requises

Suivez le guide de démarrage rapide suivant : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="provision-a-self-hosted-gateway"></a>Approvisionner une passerelle auto-hébergée

1. Sélectionnez **Passerelles** sous **Déploiement et infrastructure**.
2. Cliquez sur **+ Ajouter**.
3. Entrez le **Nom** et la **Région** de la passerelle.
> [!TIP]
> **Région** spécifie l’emplacement prévu des nœuds de passerelle qui seront associés à cette ressource de passerelle. Elle est sémantiquement équivalente à une propriété similaire associée à une ressource Azure, mais peut être affectée à une valeur de chaîne arbitraire.

4. Le cas échéant, entrez une **Description** de la ressource de passerelle.
5. Le cas échéant, sélectionnez **+** sous **API** pour associer une ou plusieurs API à la ressource de passerelle.
> [!IMPORTANT]
> Par défaut, aucune des API existantes n’est associée à la nouvelle ressource de passerelle. Par conséquent, les tentatives d’appels de ces API via la nouvelle passerelle produiront des erreurs de type `404 Resource Not Found`.

6. Cliquez sur **Ajouter**.

La ressource de passerelle a maintenant été approvisionnée dans votre instance Gestion des API. Vous pouvez procéder au déploiement de la passerelle.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la passerelle auto-hébergée, consultez [Vue d’ensemble de la passerelle auto-hébergée Gestion des API](self-hosted-gateway-overview.md)
* Apprenez-en davantage pour [Déployer une passerelle auto-hébergée sur Kubernetes](how-to-deploy-self-hosted-gateway-kubernetes.md)
* Apprenez-en davantage sur [Déployer une passerelle auto-hébergée sur Docker](how-to-deploy-self-hosted-gateway-docker.md)
