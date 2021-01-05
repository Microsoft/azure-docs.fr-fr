---
title: Validation des alertes dans Azure Security Center | Microsoft Docs
description: Découvrir comment vérifier que vos alertes de sécurité sont correctement configurées dans Azure Security Center
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: f8f17a55-e672-4d86-8ba9-6c3ce2e71a57
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2020
ms.author: memildin
ms.openlocfilehash: 598c13b0434a364e73471b53c82663b94fb42f4e
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560099"
---
# <a name="alert-validation-in-azure-security-center"></a>Validation des alertes dans Azure Security Center
Ce document est conçu pour vous apprendre à vérifier si votre système est correctement configuré pour les alertes dans Azure Security Center.

## <a name="what-are-security-alerts"></a>Que sont les alertes de sécurité ?
Les alertes sont les notifications que Security Center génère lorsqu'il détecte des menaces sur vos ressources. Il hiérarchise et répertorie les alertes ainsi que les informations vous permettant d’analyser rapidement le problème. Security Center fournit également des suggestions sur la manière dont vous pouvez corriger les problèmes liés à une attaque.
Pour plus d’informations, consultez [Alertes de sécurité dans Security Center](security-center-alerts-overview.md) et [Gestion et résolution des alertes de sécurité](security-center-managing-and-responding-alerts.md)


## <a name="generate-sample-azure-defender-alerts"></a>Générer des exemples d’alertes Azure Defender

Si vous utilisez la nouvelle expérience d’alertes en préversion comme décrit dans [Gérer et résoudre les alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md), vous pouvez créer des exemples d’alertes en quelques clics dans la page des alertes de sécurité du portail Azure.

Utilisez des exemples d’alertes pour :

- évaluer la valeur et les fonctionnalités d’Azure Defender ;
- valider les configurations que vous avez effectuées pour vos alertes de sécurité (telles que les intégrations SIEM, l’automatisation des flux de travail et les notifications par e-mail).

> [!NOTE]
> Cette procédure requiert l’utilisation de la nouvelle expérience d’alertes (en préversion) disponible dans la bannière en haut de la page **Alertes de sécurité**.
>
> :::image type="content" source="media/security-center-managing-and-responding-alerts/preview-alerts-experience-banner.png" alt-text="Bannière avec un lien vers la nouvelle expérience d’alertes en préversion":::

Pour créer des exemples d’alertes :

1. Dans la barre d’outils de la page des alertes, sélectionnez **Créer des exemples d’alertes**. 
1. Sélectionnez l’abonnement.
1. Sélectionnez le ou les plans Azure Defender appropriés pour lesquels vous souhaitez afficher les alertes. 
1. Sélectionnez **Créer des exemples d’alertes**.

    :::image type="content" source="media/security-center-alert-validation/create-sample-alerts-procedures.png" alt-text="Étapes de création d’exemples d’alertes dans Azure Security Center":::
    
    Une notification s’affiche pour vous informer que les exemples d’alertes sont en cours de création :

    :::image type="content" source="media/security-center-alert-validation/notification-sample-alerts-creation.png" alt-text="Notification indiquant que les exemples d’alertes sont générés.":::

    Après quelques minutes, les alertes s’affichent dans la page des alertes de sécurité. Ils apparaîtront également dans tous les autres emplacements que vous avez configurés pour recevoir les alertes de sécurité de votre Azure Security Center (SIEM connectés, notifications par e-mail, etc.).

    :::image type="content" source="media/security-center-alert-validation/sample-alerts.png" alt-text="Exemples d’alertes dans la liste des alertes de sécurité":::

    > [!TIP]
    > Les alertes concernent ont trait à des ressources simulées.

## <a name="simulate-alerts-on-your-azure-vms-windows"></a>Simuler des alertes sur vos machines virtuelles Azure (Windows) <a name="validate-windows"></a>

Après avoir installé l’agent de Security Center sur votre ordinateur, suivez les étapes ci-dessous depuis l’ordinateur sur lequel vous voulez configurer l’alerte de la ressource attaquée :

1. Copiez un exécutable (par exemple, **calc.exe**) sur le bureau de votre ordinateur ou dans le répertoire de votre choix et renommez-le **ASC_AlertTest_662jfi039N.exe**.
1. Ouvrez l’invite de commandes et exécutez ce fichier avec un argument (un faux nom d’argument), tel que : ```ASC_AlertTest_662jfi039N.exe -foo```
1. Patientez 5 à 10 minutes puis ouvrez les alertes dans Security Center. Une alerte doit s’afficher.

> [!NOTE]
> En examinant cette alerte de test, veillez à ce que la valeur du champ **Arguments Auditing Enabled** (Audit pour arguments activé) soit **true**. S’il est **false**, vous devez activer la ligne de commande d’audit pour arguments. Pour ce faire, utilisez la commande suivante :
>
>```reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system\Audit" /f /v "ProcessCreationIncludeCmdLine_Enabled"```

## <a name="simulate-alerts-on-your-azure-vms-linux"></a>Simuler des alertes sur vos machines virtuelles Azure (Linux) <a name="validate-linux"></a>

Après avoir installé l’agent de Security Center sur votre ordinateur, suivez les étapes ci-dessous depuis l’ordinateur sur lequel vous voulez configurer l’alerte de la ressource attaquée :
1. Copiez un fichier exécutable dans un emplacement approprié et renommez-le **. / asc_alerttest_662jfi039n**, par exemple :

    ```cp /bin/echo ./asc_alerttest_662jfi039n```

1. Ouvrez l’invite de commandes et exécutez ce fichier :

    ```./asc_alerttest_662jfi039n testing eicar pipe```

1. Patientez 5 à 10 minutes puis ouvrez les alertes dans Security Center. Une alerte doit s’afficher.


## <a name="simulate-alerts-on-kubernetes"></a>Simuler des alertes sur Kubernetes <a name="validate-kubernetes"></a>

Si vous avez intégré Azure Kubernetes Service au Security Center, vous pouvez tester le fonctionnement de vos alertes avec la commande kubectl suivante :

```kubectl get pods --namespace=asc-alerttest-662jfi039n```

Pour en savoir plus sur la protection de vos nœuds et clusters Kubernetes, consultez [Présentation d’Azure Defender pour Kubernetes](defender-for-kubernetes-introduction.md)

## <a name="next-steps"></a>Étapes suivantes
Cet article vous a présenté le processus de validation des alertes. Maintenant que vous êtes familiarisé avec la validation, vous pouvez consulter les articles suivants :

* [Validation de la détection des menaces d’Azure Key Vault dans Azure Security Center](https://techcommunity.microsoft.com/t5/azure-security-center/validating-azure-key-vault-threat-detection-in-azure-security/ba-p/1220336)
* [Gestion et résolution des alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md) : découvrez comment gérer des alertes et résoudre les incidents de sécurité dans Security Center.
* [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md) : découvrez comment surveiller l’intégrité de vos ressources Azure.
* [Présentation des alertes de sécurité dans Azure Security Center](./security-center-alerts-overview.md) : en savoir plus sur les différents types d’alertes de sécurité.