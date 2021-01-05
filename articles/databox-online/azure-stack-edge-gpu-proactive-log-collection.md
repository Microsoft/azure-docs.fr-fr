---
title: Présentation de la collecte de journaux proactive sur un appareil Azure Stack Edge Pro
description: Décrit comment la collecte de journaux proactive est effectuée sur un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: alkohli
ms.openlocfilehash: f79de47ec0ffad11f650054b581dbbaae030edbf
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465273"
---
# <a name="proactive-log-collection-on-your-azure-stack-edge-device"></a>Collecte de journaux proactive sur votre appareil Azure Stack Edge

Vous pouvez activer la collecte de journaux proactive sur votre appareil Azure Stack Edge en fonction des indicateurs d’intégrité du système afin résoudre efficacement tout problème lié à l’appareil. Cet article décrit ce qui est la collecte de journaux proactive, comment l’activer et la manière dont les données sont traitées lorsque la collecte de journaux proactive est activée.
   
Les informations de cet article s’appliquent aux appareils Azure Stack Edge Pro GPU, Azure Stack Edge Pro R et Azure Stack Edge Mini R.

## <a name="about-proactive-log-collection"></a>À propos de la collecte de journaux proactive

Les équipes d’ingénierie et de support technique de Microsoft utilisent les journaux système de votre appareil Azure Stack Edge pour identifier et résoudre efficacement les problèmes susceptibles de survenir lors de l’utilisation de celui-ci. La collecte de journaux proactive est une méthode qui avertit Microsoft qu’un problème/événement (pour les événements suivis, voir la section sur les indicateurs de collecte de journaux proactive) a été détecté par l’appliance Azure Stack Edge du client. Les journaux de support relatifs au problème sont chargés automatiquement sur un compte de stockage Azure géré et contrôlé par Microsoft. Le support et les ingénieurs de Microsoft examinent ces journaux de support afin de déterminer la meilleure solution pour résoudre le problème avec le client.    

> [!NOTE]
> Ces journaux sont utilisés uniquement à des fins de débogage et fournissent un support aux clients en cas de problème. 


## <a name="enabling-proactive-log-collection"></a>Activation de la collecte de journaux proactive

Vous pouvez activer la collecte de journaux proactive lorsque vous tentez d’activer l’appareil via l’interface utilisateur locale. 

1. Dans l’interface utilisateur Web locale de l’appareil, accédez à la page **Bien démarrer**.
2. Dans la vignette **Activation**, sélectionnez **Activer**. 

    ![Page « Détails du cloud » 1 de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-activate/activate-1.png)
    
3. Dans le volet **Activer** :
    1. Entrez la **clé d’activation** que vous avez obtenue à l’étape [Obtenir la clé d’activation pour Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

    1. Vous pouvez activer une collecte de journaux proactive pour permettre à Microsoft de collecter les journaux en fonction de l’état d’intégrité de l’appareil. Les journaux collectés de cette façon sont chargés sur un compte de stockage Azure.
    
    1. Sélectionnez **Appliquer**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 2](./media/azure-stack-edge-pro-r-deploy-activate/activate-2.png)



## <a name="proactive-log-collection-indicators"></a>Indicateurs de collecte de journaux proactive

Une fois la collecte de journaux proactive activée, les journaux sont chargés automatiquement quand un des événements suivants est détecté sur l’appareil :  


|Alerte/Erreur/condition  |Description  |
|---------|---------|
|AcsUnhealthyCondition     |Les services cohérents Azure ne sont pas sains.         |
|IOTEdgeAgentNotRunningCondition      |L’agent IoT Edge n’est pas en cours d’exécution.         |
|UpdateInstallFailedEvent | Impossible d’installer la mise à jour.        |

 
Microsoft va continuer à ajouter de nouveaux événements à la liste précédente. Aucun consentement supplémentaire n’est nécessaire pour les nouveaux événements. Pour en savoir plus sur les indicateurs de collecte de journaux proactive les plus récents, consultez cette page.    
 

## <a name="other-log-collection-methods"></a>Autres méthodes de collecte de journaux

Outre la collecte des journaux proactive qui collecte des journaux spécifiques relatifs à un problème spécifique détecté, d’autres collectes de journaux peuvent apporter une compréhension plus approfondie de l’intégrité et du comportement du système. En général, ces autres collectes de journaux peuvent être exécutées lors d’une demande de support ou déclenchées par Microsoft en fonction des données de télémétrie que l’appareil fournit.  

## <a name="handling-data"></a>Gestion des données

Si un client active la collecte des journaux proactive, il consent à ce que Microsoft collecte les journaux de l’appareil Azure Stack Edge de la manière décrite dans le présent document. Le client consent également expressément au chargement et à la rétention de ces journaux dans un compte de stockage Azure managé et contrôlé par Microsoft.

Microsoft utilise les données uniquement pour résoudre des problèmes liés à l’intégrité du système. Les données ne sont pas utilisées à des fins de marketing, publicitaires ou commerciales sans le consentement du client. 

Microsoft traite les données collectées conformément à ses pratiques de confidentialité standard. Si un client décide de révoquer son consentement à la collecte des journaux proactive, cela n’affecte les données collectées avec son consentement avant la révocation de celui-ci.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Collecter un package de support](azure-stack-edge-gpu-troubleshoot.md#collect-support-package).