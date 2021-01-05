---
title: Connecter des données de Pare-feu Windows Defender à Azure Sentinel | Microsoft Docs
description: Activez le connecteur du pare-feu Windows dans Azure Sentinel pour diffuser facilement les événements de pare-feu d’ordinateurs Windows sur lesquels des agents de Log Analytics sont installés.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0e41f896-8521-49b8-a244-71c78d469bc3
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/05/2020
ms.author: yelevin
ms.openlocfilehash: cf7e389fc4a8a8dfa88691dc034611cae3471731
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655338"
---
# <a name="connect-windows-defender-firewall-with-advanced-security-to-azure-sentinel"></a>Connecter Pare-feu Windows Defender avec sécurité avancée à Azure Sentinel

Le connecteur [Pare-feu Windows Defender avec sécurité avancée](/windows/security/threat-protection/windows-firewall/windows-firewall-with-advanced-security) permet à Azure Sentinel d’ingérer facilement les journaux Pare-feu Windows Defender avec sécurité avancée à partir de n’importe quelle machine Windows dans votre espace de travail. Cette connexion vous permet d’afficher et d’analyser les événements Pare-feu Windows dans vos classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer dans vos investigations de sécurité, ce qui vous donne plus d’informations sur le réseau de votre organisation et améliore vos capacités d’opérations de sécurité. 

La solution collecte les événements de pare-feu Windows à partir des machines Windows sur lesquelles un agent Log Analytics est installé. 

> [!NOTE]
> - Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.
>
> - Si les alertes Azure Defender d’Azure Security Center sont déjà collectées dans l’espace de travail Azure Sentinel, il n’est pas nécessaire d’activer la solution du Pare-feu Windows par le biais de ce connecteur. Toutefois, si vous l’avez activée, cela ne produira pas de données en double. 

## <a name="prerequisites"></a>Conditions préalables requises

- Vous devez disposer d’autorisations en lecture et en écriture sur l’espace de travail auquel sont connectées les machines que vous souhaitez analyser.

- Vous devez être affecté au rôle **Contributeur Log Analytics** sur la solution SecurityInsights de cet espace de travail, en plus de tout rôles **Azure Sentinel**. [En savoir plus](../role-based-access-control/built-in-roles.md#log-analytics-contributor)

## <a name="enable-the-connector"></a>Activer le connecteur 

1. Dans le portail Azure Sentinel, sélectionnez **Connecteurs de données** dans le menu de navigation.

1. Sélectionnez **Pare-feu Windows** dans la galerie des connecteurs et cliquez sur **Ouvrir la page du connecteur**.

### <a name="instructions-tab"></a>Onglet Instructions

- **Si vos machines Windows se trouvent dans Azure :**

    1. Sélectionnez **Installer l’agent sur une machine virtuelle Windows Azure**.

    1. Cliquez sur le lien **Télécharger et installer l’agent pour les machines virtuelles Windows Azure >** qui apparaît.

    1. Dans la liste **Machines virtuelles**, sélectionnez la machine Windows que vous souhaitez diffuser en continu dans Azure Sentinel. (Vous pouvez sélectionner **Windows** dans le filtre de colonne du système d’exploitation pour vous assurer que seules les machines virtuelles Windows sont affichées).

    1. Dans la fenêtre qui s’ouvre pour cette machine virtuelle, cliquez sur **Connecter**.

    1. Revenez au volet **Machines virtuelles** et répétez les deux étapes précédentes pour toutes les autres machines virtuelles que vous souhaitez connecter. Lorsque vous avez terminé, revenez au volet **Pare-feu Windows**.

- **Si votre machine Windows n’est pas une machine virtuelle Azure :**

    1. Sélectionnez **Installer l’agent sur une machine Windows non-Azure**.

    1. Cliquez sur le lien **Télécharger et installer l’agent pour les machines Windows non-Azure >** qui apparaît.

    1. Dans le volet **Gestion des agents**, sélectionnez **Télécharger l’agent Windows (64 bits)** ou **Télécharger l’agent Windows (32 bits)** , le cas échéant.

    1. Copiez les chaînes **ID d’espace de travail**, **Clé primaire** et **Clé secondaire** dans un fichier texte. Copiez ce fichier et le fichier d’installation téléchargé sur votre machine Windows. Exécutez le fichier d’installation et, lorsque vous y êtes invité, entrez les chaînes d’ID et de clé dans le fichier texte pendant l’installation.

    1. Revenez au volet **Pare-feu Windows**.

1. Cliquez sur **Installer la solution**.

### <a name="next-steps-tab"></a>Onglet Étapes suivantes

- Pour obtenir des informations sur les données du journal Pare-feu Windows, consultez les exemples de requêtes et les classeurs recommandés disponibles, fournis avec le connecteur de données **Pare-feu Windows**.

- Pour interroger les données du pare-feu Windows dans **Journaux**, saisissez **WindowsFirewall** dans la fenêtre de requête.

## <a name="validate-connectivity"></a>Valider la connectivité
 
Comme les journaux Pare-feu Windows sont envoyés à Azure Sentinel uniquement lorsque le fichier journal local atteint sa capacité maximale, laisser le journal à sa taille par défaut de 4096 ko entraînera très probablement une latence de collecte élevée. Vous pouvez réduire la latence en diminuant la taille du fichier journal. Consultez les instructions pour [configurer le journal Pare-feu Windows](/windows/security/threat-protection/windows-firewall/configure-the-windows-firewall-log). Notez que si la définition de la taille de journal la plus petite possible (1 ko) élimine pratiquement la latence de collecte, elle peut également avoir un impact négatif sur le niveau de performance de l’ordinateur local. 

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter le pare-feu Windows à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).