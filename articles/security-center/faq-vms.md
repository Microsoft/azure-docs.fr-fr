---
title: FAQ Azure Security Center - Questions relatives aux machines virtuelles
description: Questions fréquentes relatives aux machines virtuelles dans Azure Security Center, produit qui vous aide à prévenir, détecter et répondre aux menaces
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 32023fca15de9d3f9258bc05166c3cff4184ea15
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499847"
---
# <a name="faq---questions-about-virtual-machines"></a>FAQ - Questions relatives aux machines virtuelles


## <a name="what-types-of-virtual-machines-are-supported"></a>Quels sont les types de machines virtuelles pris en charge ?

Le monitoring et des recommandations sont disponibles pour les machines virtuelles créées à l’aide des [modèles de déploiement Classic et Resource Manager](../azure-resource-manager/management/deployment-models.md).

Consultez [Plateformes prises en charge dans Azure Security Center](security-center-os-coverage.md) pour obtenir la liste des plateformes prises en charge.


## <a name="why-doesnt-azure-security-center-recognize-the-antimalware-solution-running-on-my-azure-vm"></a>Pourquoi Azure Security Center ne reconnaît-il pas la solution anti-programme malveillant de ma machine virtuelle Azure ?

Azure Security Center bénéficie d’une visibilité sur les logiciels anti-programme malveillant installés par le biais des extensions Azure. Par exemple, Security Center n’est pas en mesure de détecter les logiciels anti-programme malveillant préinstallés sur une image que vous avez fournie ou installés sur vos machines virtuelles à l’aide de votre propre processus (notamment les systèmes de gestion de la configuration).


## <a name="why-do-i-get-the-message-missing-scan-data-for-my-vm"></a>Pourquoi reçois-je le message « Données d’analyse manquantes » pour ma machine virtuelle ?

Ce message s’affiche lorsqu’il n’existe aucune donnée d’analyse pour une machine virtuelle. Le remplissage des données d’analyse peut prendre un certain temps (inférieur à une heure) une fois la collecte de données activée dans Azure Security Center. Après le remplissage initial des données d’analyse, vous pouvez recevoir ce message s’il n’existe aucune donnée d’analyse ou s’il n’existe pas de données d’analyse récentes. Le remplissage des analyses n’est pas effectué pour une machine virtuelle à l’état Arrêté. Ce message peut également s’afficher si les données d’analyse n’ont pas été renseignées récemment (conformément à la stratégie de rétention de l’agent Windows, dont la valeur par défaut est 30 jours).


## <a name="how-often-does-security-center-scan-for-operating-system-vulnerabilities-system-updates-and-endpoint-protection-issues"></a>À quelle fréquence Security Center recherche-t-il les vulnérabilités du système d’exploitation, les mises à jour du système et les problèmes de protection du point de terminaison ?

Les temps de latence des analyses de Security Center en lien avec les vulnérabilités, les mises à jour et les problèmes figurent ci-dessous :

- Configuration de la sécurité du système d’exploitation ; les données sont mises à jour dans les 48 heures
- Mises à jour du système ; les données sont mises à jour dans les 24 heures
- Problèmes de protection du point de terminaison ; les données sont mises à jour dans les 8 heures

Généralement, Security Center analyse les nouvelles données toutes les heures et actualise les suggestions en conséquence. 

> [!NOTE]
> Security Center utilise l'agent Log Analytics pour collecter et stocker les données. Pour en savoir plus, consultez [Migration de plateforme Azure Security Center](./security-center-enable-data-collection.md).


## <a name="why-do-i-get-the-message-vm-agent-is-missing"></a>Pourquoi reçois-je le message « Agent de machine virtuelle manquant » ?

L’agent de machine virtuelle doit être installé sur les machines virtuelles pour activer la collecte des données. L’agent de machine virtuelle est installé par défaut sur les machines virtuelles déployées depuis Azure Marketplace. Pour plus d’informations sur l’installation de l’agent de machine virtuelle sur d’autres machines virtuelles, consultez l’article de blog [Agent de machine virtuelle et extensions](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-2/).