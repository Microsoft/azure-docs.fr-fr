---
title: Connecter les données Azure Defender à Azure Sentinel
description: Découvrez comment connecter les alertes Azure Defender à partir d’Azure Security Center et les diffuser vers Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: d28c2264-2dce-42e1-b096-b5a234ff858a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 09/07/2020
ms.author: yelevin
ms.openlocfilehash: b37675d46d501fc43446eafc83d6c0a02f238177
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655950"
---
# <a name="connect-azure-defender-alert-data-from-azure-security-center"></a>Connecter les données d’alertes Azure Defender à partir d’Azure Security Center

Utilisez le connecteur d’alerte Azure Defender pour ingérer des alertes Azure Defender à partir d’[Azure Security Center](../security-center/security-center-introduction.md) et les diffuser vers Azure Sentinel. 

## <a name="prerequisites"></a>Prérequis

- Votre utilisateur doit avoir le rôle Lecteur Sécurité dans l’abonnement des journaux que vous diffusez.

- Vous devez activer Azure Defender dans Azure Security Center (le niveau standard n’existe plus et n’est plus une exigence de licence).

## <a name="connect-to-azure-defender"></a>Connexion à Azure Defender

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données** dans le menu de navigation.

1. Dans la Galerie connecteurs de données, sélectionnez **Alertes Azure Defender issues d’ASC** (peut être appelé Azure Security Center), puis cliquez sur le bouton **Ouvrir la page du connecteur**.

1. Sous **Configuration**, cliquez sur **Se connecter** à côté de chaque abonnement dont vous souhaitez diffuser les alertes dans Azure Sentinel. Le bouton Connecter n’est disponible que si vous disposez des autorisations requises.

1. Vous pouvez indiquer si vous voulez que les alertes d’Azure Defender génèrent automatiquement des incidents dans Azure Sentinel. Sous **Créer des incidents**, sélectionnez **Activé** pour activer la règle analytique par défaut qui crée automatiquement des incidents à partir d’alertes. Vous pouvez ensuite modifier cette règle sous **Analytique** dans l’onglet **Règles actives**.

1. Pour utiliser le schéma pertinent dans Log Analytics pour les alertes d’Azure Defender, recherchez **SecurityAlert**.

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a montré comment connecter Azure Defender à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).