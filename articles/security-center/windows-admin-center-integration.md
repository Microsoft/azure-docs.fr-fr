---
title: Comment protéger des serveurs Windows Admin Center grâce à Azure Security Center
description: Cet article explique comment intégrer Azure Security Center à Windows Admin Center
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: memildin
ms.openlocfilehash: ca3dd410474af0a42a3cd5b4217c3bba90b8ff1a
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511982"
---
# <a name="protect-windows-admin-center-resources-with-security-center"></a>Protéger des ressources Windows Admin Center grâce à Security Center

Windows Admin Center est un outil de gestion pour vos serveurs Windows. Il s’agit d’un emplacement unique permettant aux administrateurs système d’accéder à la plupart des outils d’administration les plus couramment utilisés. Dans Windows Admin Center, vous pouvez directement intégrer vos serveurs locaux dans Azure Security Center. Vous pouvez ensuite afficher un résumé de vos recommandations et alertes de sécurité directement dans l’expérience Windows Admin Center.

> [!NOTE]
> Azure Defender doit être activé à la fois sur votre abonnement Azure et sur l’espace de travail Log Analytics associé afin d’activer l’intégration de Windows Admin Center.
> Azure Defender est gratuit pendant les 30 premiers jours si vous ne l’avez pas déjà utilisé sur l’abonnement et l’espace de travail. Pour plus d’informations, consultez la [page d’informations de tarification](security-center-pricing.md).
>

Une fois que vous avez correctement intégré un serveur de Windows Admin Center dans Azure Security Center, vous pouvez :

* Afficher les alertes de sécurité et les recommandations à l’intérieur de l’extension Security Center dans Windows Admin Center
* Afficher la situation de sécurité et récupérer des informations détaillées supplémentaires sur vos serveurs gérés par Windows Admin Center dans Security Center au sein du portail Azure (ou via une API)

En combinant ces deux outils, Security Center devient votre fenêtre unique pour afficher toutes vos informations de sécurité, quelle que soit la ressource : protection de vos serveurs locaux gérés par Windows Admin Center, de vos machines virtuelles et de toutes les charges de travail PaaS supplémentaires.

## <a name="onboard-windows-admin-center-managed-servers-into-security-center"></a>Intégrer des serveurs gérés par Windows Admin Center à Security Center

1. Dans Windows Admin Center, sélectionnez l’un de vos serveurs, puis dans le volet **Outils**, sélectionnez l’extension Azure Security Center :

    ![Extension Azure Security Center dans Windows Admin Center](./media/windows-admin-center-integration/onboarding-from-wac.png)

    > [!NOTE]
    > Si le serveur est déjà intégré à Security Center, la fenêtre de configuration n’apparaît pas.

1. Cliquez sur **Se connecter à Azure et configurer**.
    ![Intégration de l’extension Windows Admin Center à Azure Security Center](./media/windows-admin-center-integration/onboarding-from-wac-welcome.png)

1. Suivez les instructions pour connecter votre serveur à Security Center. Une fois que vous avez entré et confirmé les informations nécessaires, Security Center effectue les modifications de configuration nécessaires pour s’assurer que toutes les conditions suivantes sont remplies :
    * Une passerelle Azure est inscrite.
    * Le serveur dispose d’un espace de travail pour les rapports et d’un abonnement associé.
    * La solution Log Analytics de Security Center est activée sur l’espace de travail. Cette solution fournit les fonctionnalités d’Azure Defender pour *tous* les serveurs et toutes les machines virtuelles qui produisent des rapports sur cet espace de travail.
    * Azure Defender pour serveurs est activé sur l’abonnement.
    * L’agent Log Analytics est installé sur le serveur et configuré pour rendre compte à l’espace de travail sélectionné. Si le serveur est déjà dans un autre espace de travail, il est configuré pour créer des rapports sur l’espace de travail nouvellement sélectionné.

    > [!NOTE]
    > L’intégration des recommandations peut prendre un certain temps. En fait, en fonction de l’activité de votre serveur, vous risquez de ne recevoir *aucune* alerte. Pour générer des alertes de test pour vérifier que vos alertes fonctionnent correctement, suivez les instructions de [la procédure de validation d’alerte](security-center-alert-validation.md).


## <a name="view-security-recommendations-and-alerts-in-windows-admin-center"></a>Afficher des recommandations et des alertes de sécurité dans Windows Admin Center

Une fois intégré, vous pouvez afficher vos alertes et vos recommandations directement dans la zone Azure Security Center de Windows Admin Center. Cliquez sur une recommandation ou une alerte pour l’afficher dans le portail Azure. Là, vous obtiendrez des informations supplémentaires et découvrirez comment corriger les problèmes.

[![Alertes et recommandations de Security Center comme elles sont affichées dans Windows Admin Center](media/windows-admin-center-integration/asc-recommendations-and-alerts-in-wac.png)](media/windows-admin-center-integration/asc-recommendations-and-alerts-in-wac.png#lightbox)

## <a name="view-security-recommendations-and-alerts-for-windows-admin-center-managed-servers-in-security-center"></a>Afficher des recommandations et des alertes de sécurité pour les serveurs gérés par Windows Admin Center dans Security Center
À partir d’Azure Security Center :

* Pour afficher les recommandations de sécurité concernant l’ensemble de vos serveurs Windows Admin Center, ouvrez l’[inventaire des actifs](asset-inventory.md) et appliquez un filtre sur le type de machine que vous souhaitez examiner. sélectionnez l’onglet **Machines virtuelles et ordinateurs**.

* Pour afficher les alertes de sécurité pour tous les serveurs Windows Admin Center, ouvrez **Alertes de sécurité**. Cliquez sur **Filtre** et assurez-vous que **seul** « Non Azure » est sélectionné :

    :::image type="content" source="./media/windows-admin-center-integration/filtering-alerts-by-environment.png" alt-text="Filtrer les alertes de sécurité pour les serveurs gérés par Windows Admin Center" lightbox="./media/windows-admin-center-integration/filtering-alerts-by-environment.png":::