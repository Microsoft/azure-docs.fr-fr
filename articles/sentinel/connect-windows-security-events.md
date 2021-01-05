---
title: Connecter les données d’événement de sécurité Windows à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur Événements de sécurité pour transmettre en continu tous les événements de sécurité de vos systèmes Windows à votre espace de travail Azure Sentinel. 
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d51d2e09-a073-41c8-b396-91d60b057e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: a16afcafa03ef2ab8642316db560e30a473a526b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90883720"
---
# <a name="connect-windows-security-events"></a>Connecter les événements de sécurité Windows 

Le connecteur Événements de sécurité vous permet de transmettre en continu tous les événements de sécurité de vos systèmes Windows (serveurs et stations de travail, physiques et virtuels) à votre espace de travail Azure Sentinel. Cela vous permet d’afficher les événements de sécurité Windows dans vos tableaux de bord, de les utiliser pour créer des alertes personnalisées et de les utiliser pour améliorer vos investigations, ce qui vous donne plus d’informations sur le réseau de votre organisation et développe vos capacités d’opérations de sécurité. Vous pouvez sélectionner les événements à transmettre en continu à partir des ensembles suivants : <a name="event-sets"></a>

- **Tous les événements** : tous les événements AppLocker et de sécurité Windows.
- **Courant** : un ensemble d’événements à des fins d’audit. Une piste d’audit utilisateur complète est fournie dans cet ensemble. Par exemple, il contient les événements de connexion et de déconnexion de l’utilisateur (ID d’événement 4624, 4634). Il existe également des actions d’audit, telles que les modifications de groupe de sécurité, les opérations Kerberos du contrôleur de domaine clé et les autres événements alignés avec les meilleures pratiques acceptées.

    L’ensemble d’événements **Common** peut contenir certains types d’événements qui ne sont pas si courants.  Cela est dû au fait que le point principal de l’ensemble **Common** est de réduire le volume d’événements à un niveau plus gérable, tout en conservant la fonctionnalité de piste d’audit complète.

- **Minimal** : un petit ensemble d’événements qui peuvent indiquer des menaces potentielles. Cet ensemble ne contient pas de piste d’audit complète. Il couvre uniquement les événements qui peuvent indiquer une violation avérée et d’autres événements importants qui ont un taux d’occurrence très faible. Par exemple, il contient des connexions utilisateur ayant réussi et ayant échoué (ID d’événement 4624, 4625), mais il ne contient pas d’informations de déconnexion (4634), qui sont importantes pour l’audit, mais pas pour la détection de violation, et dont le volume est relativement élevé. Le plus gros du volume de données de cet ensemble est constitué d’événements de connexion et d’événements de création de processus (ID d’événement 4688).

- **Aucun** : aucun événement AppLocker ni de sécurité. (Ce paramètre est utilisé pour désactiver le connecteur.)

    La liste suivante fournit le détail complet des ID d’événement App Locker et de sécurité pour chaque ensemble :

    | Ensemble d’événements | ID d’événements collectés |
    | --- | --- |
    | **Minimal** | 1102, 4624, 4625, 4657, 4663, 4688, 4700, 4702, 4719, 4720, 4722, 4723, 4724, 4727, 4728, 4732, 4735, 4737, 4739, 4740, 4754, 4755, 4756, 4767, 4799, 4825, 4946, 4948, 4956, 5024, 5033, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222 |
    | **Commun** | 1, 299, 300, 324, 340, 403, 404, 410, 411, 412, 413, 431, 500, 501, 1100, 1102, 1107, 1108, 4608, 4610, 4611, 4614, 4622, 4624, 4625, 4634, 4647, 4648, 4649, 4657, 4661, 4662, 4663, 4665, 4666, 4667, 4688, 4670, 4672, 4673, 4674, 4675, 4689, 4697, 4700, 4702, 4704, 4705, 4716, 4717, 4718, 4719, 4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4733, 4732, 4735, 4737, 4738, 4739, 4740, 4742, 4744, 4745, 4746, 4750, 4751, 4752, 4754, 4755, 4756, 4757, 4760, 4761, 4762, 4764, 4767, 4768, 4771, 4774, 4778, 4779, 4781, 4793, 4797, 4798, 4799, 4800, 4801, 4802, 4803, 4825, 4826, 4870, 4886, 4887, 4888, 4893, 4898, 4902, 4904, 4905, 4907, 4931, 4932, 4933, 4946, 4948, 4956, 4985, 5024, 5033, 5059, 5136, 5137, 5140, 5145, 5632, 6144, 6145, 6272, 6273, 6278, 6416, 6423, 6424, 8001, 8002, 8003, 8004, 8005, 8006, 8007, 8222, 26401, 30004 |

> [!NOTE]
> La collecte des événements de sécurité dans le contexte d’un seul espace de travail peut être configurée à partir d’Azure Security Center ou d’Azure Sentinel, mais pas les deux à la fois. Si vous intégrez Azure Sentinel dans un espace de travail qui reçoit déjà des alertes Azure Defender et qui est défini pour collecter des événements de sécurité, vous disposez de deux options :
> - Laissez la collecte d’événements de sécurité dans Azure Security Center telle quelle. Vous êtes alors en mesure d’interroger et d’analyser ces événements dans Azure Sentinel ainsi que dans Azure Defender. Toutefois, vous n’êtes pas en mesure de surveiller l’état de connectivité du connecteur ou de modifier sa configuration dans Azure Sentinel. Si cela est important pour vous, envisagez la deuxième option.
>
> - [Désactivez la collecte des événements de sécurité](../security-center/security-center-enable-data-collection.md) dans Azure Security Center, puis ajoutez le connecteur Événements de sécurité dans Azure Sentinel. Comme pour la première option, vous êtes en mesure d’interroger et d’analyser les événements à la fois dans Azure Sentinel et Azure Defender/ASC. Cependant, vous pouvez maintenant surveiller l’état de connectivité du connecteur ou modifier sa configuration dans, et uniquement dans, Azure Sentinel.

## <a name="set-up-the-windows-security-events-connector"></a>Configurer le connecteur d’événements de sécurité Windows

Pour collecter vos événements de sécurité Windows dans Azure Sentinel :

1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**. Dans la liste des connecteurs, cliquez sur **Événements de sécurité**, puis sur le bouton **Ouvrir la page du connecteur** dans le coin inférieur droit. Suivez ensuite les instructions à l’écran sous l’onglet **Instructions**, comme décrit dans le reste de cette section.

1. Vérifiez que vous disposez des autorisations appropriées, comme décrit dans la section **Conditions préalables**.

1. Téléchargez et installez l’[agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) (également appelé Microsoft Monitoring Agent ou MMA) sur les machines pour lesquelles vous souhaitez diffuser des événements de sécurité dans Azure Sentinel.

    Pour les machines virtuelles Azure :
    
    1. Cliquez sur **Installer l’agent sur la machine virtuelle Windows Azure**, puis sur le lien qui apparaît dessous.
    1. Pour chaque machine virtuelle que vous souhaitez connecter, cliquez sur son nom dans la liste qui s’affiche à droite, puis cliquez sur **Connecter**.

    Pour les machines Windows non-Azure (physiques, virtuelles locales ou virtuelles dans un autre cloud) :

    1. Cliquez sur **Installer l’agent sur une machine virtuelle Windows non-Azure**, puis sur le lien qui apparaît dessous.
    1. Cliquez sur les liens de téléchargement appropriés qui s’affichent à droite, sous **Ordinateurs Windows**.
    1. À l’aide du fichier exécutable téléchargé, installez l’agent sur les systèmes Windows de votre choix et configurez-le à l’aide de l’**ID et des clés de l’espace de travail** qui s’affichent sous les liens de téléchargement mentionnés ci-dessus.

    > [!NOTE]
    >
    > Pour permettre aux systèmes Windows qui n’ont pas la connexion Internet nécessaire de diffuser en continu des événements vers Azure Sentinel, téléchargez et installez la passerelle **OMS** sur un ordinateur distinct, en utilisant le lien situé en bas à droite pour l’utiliser en tant que proxy.  Vous devrez toujours installer l’agent Log Analytics sur chaque système Windows dont vous souhaitez collecter les événements.
    >
    > Pour plus d’informations sur ce scénario, consultez la documentation relative à la [**passerelle Log Analytics**](../azure-monitor/platform/gateway.md).

    Pour obtenir des options d’installation supplémentaires et d’autres informations, consultez la documentation sur l’[**agent Log Analytics**](../azure-monitor/platform/agent-windows.md).

1. Sélectionnez l’ensemble d’événements ([All, Common ou Minimal](#event-sets)) que vous souhaitez diffuser.

1. Cliquez sur **Update**.

1. Pour utiliser le schéma approprié dans Log Analytics pour les événements de sécurité Windows, saisissez `SecurityEvent` dans la fenêtre de requête.

## <a name="validate-connectivity"></a>Valider la connectivité

Environ 20 minutes peuvent être nécessaires avant que vos journaux commencent à apparaître dans Log Analytics. 

### <a name="configure-the-security-events-connector-for-anomalous-rdp-login-detection"></a>Configurer le connecteur Événements de sécurité pour la détection de connexion RDP anormale

> [!IMPORTANT]
> La détection de connexion Remote Desktop Protocol (RDP) anormale est actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel peut appliquer du Machine Learning (ML) aux données Événements de sécurité pour identifier une activité de connexion RDP anormale. Il s’agit entre autres des scénarios suivants :

- **IP inhabituelle** : l’adresse IP a rarement ou n’a jamais été observée au cours des 30 derniers jours

- **Emplacement géographique inhabituel** : l’adresse IP, la ville, le pays et l’ASN ont rarement ou n’ont jamais été observés au cours des 30 derniers jours

- **Nouvel utilisateur** : un nouvel utilisateur se connecte à partir d’une adresse IP et d’un emplacement géographique, dont les deux ou l’un des deux n’étaient pas censés être vus sur la base des données des 30 jours précédents.

**Instructions de configuration**

1. Vous devez collecter les données de connexion RDP (ID d’événement 4624) par le biais du connecteur de données **Événements de sécurité**. Assurez-vous d’avoir sélectionné un [jeu d’événements](#event-sets) en plus de « Aucun » pour transmettre en continu dans Azure Sentinel.

1. Dans le portail Azure Sentinel, cliquez sur **Analytics**, puis sur l’onglet **Modèles de règles**. Choisissez la règle **Détection de connexion RDP anormale (préversion)** et déplacez le curseur **État** sur **Activé**.

    > [!NOTE]
    > Comme l’algorithme de Machine Learning nécessite 30 jours de données pour établir un profil de base du comportement de l’utilisateur, vous devez autoriser la collecte de 30 jours de données Événements de sécurité avant que tout incident puisse être détecté.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter les événements de sécurité Windows à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Commencez à détecter les menaces avec Azure Sentinel à l’aide de règles [intégrées](tutorial-detect-threats-built-in.md) ou [personnalisées](tutorial-detect-threats-custom.md).

