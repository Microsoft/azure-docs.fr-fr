---
title: Surveiller l’intégrité des ressources CDN Azure | Microsoft Docs
description: Découvrez comment surveiller l’intégrité de vos ressources CDN Azure à l’aide d’Azure Resource Health.
services: cdn
documentationcenter: .net
author: zhangmanling
manager: zhangmanling
editor: ''
ms.assetid: bf23bd89-35b2-4aca-ac7f-68ee02953f31
ms.service: azure-cdn
ms.devlang: multiple
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 81c92b6ef5bdf89391364bc238f55200eb3679f6
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778170"
---
# <a name="monitor-the-health-of-azure-cdn-resources"></a>Surveiller l’intégrité des ressources CDN Azure
  
Azure CDN Resource Health est un sous-ensemble [d’Azure Resource Health](../service-health/resource-health-overview.md).  Vous pouvez utiliser Azure Resource Health pour surveiller l’intégrité des ressources CDN et recevoir des conseils pour résoudre les problèmes.

>[!IMPORTANT] 
>Pour le moment, Azure CDN Resource Health gère uniquement l’intégrité de la remise CDN générale et des fonctionnalités des API.  Azure CDN Resource Health ne vérifie pas les points de terminaison CDN individuels.
>
>Les signaux qui alimentent Azure CDN Resource Health peuvent être retardés de 15 minutes au plus.

## <a name="how-to-find-azure-cdn-resource-health"></a>Comment trouver Azure CDN Resource Health

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre profil CDN.

2. Cliquez sur le bouton **Settings** .

    ![Bouton Paramètres](./media/cdn-resource-health/cdn-profile-settings.png)

3. Sous *Support + dépannage* , cliquez sur **Intégrité des ressources** .

    ![Intégrité des ressources CDN](./media/cdn-resource-health/cdn-resource-health3.png)

>[!TIP] 
>Vous pouvez également rechercher les ressources CDN répertoriées dans la vignette *Intégrité des ressources* du panneau *Aide + support* .  Pour accéder rapidement à *Aide + support* , cliquez sur le point d’interrogation **?** entouré d’un cercle dans l’angle supérieur droit du portail.
>
> ![Aide + Support](./media/cdn-resource-health/cdn-help-support.png)

## <a name="azure-cdn-specific-messages"></a>Messages propres à CDN Azure

États liés à CDN Azure Resource Health :

|Message | Action recommandée |
|---|---|
|Vous avez peut-être arrêté, supprimé ou incorrectement configuré un ou plusieurs points de terminaison CDN | Vous avez peut-être arrêté, supprimé ou incorrectement configuré un ou plusieurs points de terminaison CDN.|
|Nous sommes désolés, le service de gestion CDN n’est pas disponible pour le moment | Revenez ici pour vérifier si la situation a évolué. Si votre problème persiste passé le délai prévu de résolution, contactez le support.|
|Désolé. Vos points de terminaison CDN peuvent être affectés par des problèmes courants liés à certains de nos fournisseurs CDN | Revenez ici pour vérifier si la situation a évolué. Utilisez l’outil de dépannage pour découvrir comment tester votre point de terminaison d’origine et CDN. Si votre problème persiste passé le délai prévu de résolution, contactez le support. |
|Nous sommes désolés, les modifications de la configuration des points de terminaison CDN subissent des retards de propagation | Revenez ici pour vérifier si la situation a évolué. Si vos modifications de la configuration ne sont pas entièrement propagées dans le délai imparti, contactez le support technique.|
|Nous sommes désolés, nous rencontrons des problèmes pour charger le portail supplémentaire | Revenez ici pour vérifier si la situation a évolué. Si votre problème persiste passé le délai prévu de résolution, contactez le support.|
Nous sommes désolés, nous rencontrons des problèmes avec certains de nos fournisseurs CDN | Revenez ici pour vérifier si la situation a évolué. Si votre problème persiste passé le délai prévu de résolution, contactez le support. |

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Resource Health](../service-health/resource-health-overview.md)
- [Résolution des problèmes de compression des fichiers CDN](./cdn-troubleshoot-compression.md)
- [Dépannage des points de terminaison de CDN renvoyant des états 404](./cdn-troubleshoot-endpoint.md)