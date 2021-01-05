---
title: Connecter des données Symantec ICDx à Azure Sentinel | Microsoft Docs
description: Découvrez comment utiliser le connecteur Symantec ICDx pour connecter facilement tous vos journaux de la solution de sécurité Symantec à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d068223f-395e-46d6-bb94-7ca1afd3503c
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 67fc80b5f34cf3a98fd39ddc352cb2dd9a5e7151
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85564915"
---
# <a name="connect-your-symantec-icdx-appliance"></a>Connecter votre appliance Symantec ICDx 



Le connecteur Symantec ICDx vous permet de connecter facilement les journaux de votre solution de sécurité Symantec à Azure Sentinel, de consulter des tableaux de bord, de créer des alertes personnalisées et d’améliorer les enquêtes. Cela vous donne plus d’informations sur le réseau de votre organisation et améliore vos capacités d’opération de sécurité. L'intégration entre Symantec ICDx et Azure Sentinel utilise l’API REST.


> [!NOTE]
> Les données seront stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

## <a name="configure-and-connect-symantec-icdx"></a>Configurer et connecter Symantec ICDx 

Symantec ICDx peut intégrer et exporter des journaux d'activité directement dans Azure Sentinel.

1. Ouvrez la Console de gestion ICDx pour ajouter des redirecteurs Microsoft Azure Sentinel (Log Analytics).
2. Dans la barre de navigation ICDX, cliquez sur **Configuration**. 
3. En haut de l'écran **Configuration**, cliquez sur **Redirecteurs**.
4. Sous **Redirecteurs**, en regard de Microsoft Azure Sentinel (Log Analytics), cliquez sur **Ajouter**. 
4. Dans la fenêtre **Microsoft Azure Sentinel (Log Analytics)** , cliquez sur **Afficher les options avancées**. 
5. En haut de la fenêtre développée Microsoft Azure Sentinel (Log Analytics), procédez comme suit :
    -   **Name** : Entrez un nom pour le redirecteur ne dépassant pas 30 caractères. Choisissez un nom unique et explicite. Ce nom apparaît dans la liste des redirecteurs de l'écran **Configuration**, ainsi que dans les tableaux de bord de l'écran **Tableau de bord**. Par exemple : Microsoft Azure Log Analytics Est. Ce champ est obligatoire.
    -   **Description** : Entrez une description pour le redirecteur. Cette description apparaît également dans la liste des redirecteurs de l'écran **Configuration**. Incluez des détails tels que le type d’événement transféré et le groupe chargé d'inspecter les données.
    -   **Type de démarrage** : Sélectionnez la méthode de démarrage pour la configuration du redirecteur. Les options dont vous disposez sont Manuel et Automatique.<br>L'option Automatique est sélectionnée par défaut. 
6. Sous **Événements**, procédez comme suit : 
    - **Source** : Sélectionnez une ou plusieurs archives à partir desquelles transférer des événements. Vous pouvez sélectionner des archives de collecteur actives (y compris Archives courantes), des archives de collecteur orphelines (archives correspondant aux collecteurs que vous avez supprimés), des archives de récepteur ICDx ou des archives système. <br>Les archives courantes sont sélectionnées par défaut.
      > [!NOTE]
      > Les archives de récepteur ICDx sont répertoriées séparément, par nom. 
 
    - **Filtre** : Ajoutez un filtre qui spécifie le sous-ensemble d’événements à transférer. Effectuez l’une des actions suivantes :
        - Pour sélectionner une condition de filtre, cliquez sur un Type, un Attribut, un Opérateur et une Valeur. 
        - Dans le champ Filtre, examinez la condition du filtre. Vous pouvez la modifier directement dans le champ, voire la supprimer, si besoin.
        - Cliquez sur AND ou OR pour ajouter votre condition de filtre.
        - Vous pouvez également cliquer sur Requêtes enregistrées pour appliquer une requête enregistrée.
    - **Attributs inclus** : Entrez la liste des attributs séparés par des virgules à inclure dans les données transférées. Les attributs inclus sont prioritaires sur les attributs exclus.
    - **Attributs exclus** : Entrez la liste des attributs séparés par des virgules à exclure des données transférées.
    - **Taille du lot** : Sélectionnez le nombre d’événements à envoyer par lot. Les options disponibles sont 10, 50, 100, 500 et 1 000.<br>La valeur par défaut est 100. 
    - **Débit maximal** : Sélectionnez le débit de transfert des événements, exprimé en événements par seconde. Les options disponibles sont Illimité, 500 et 1000, 5 000 et 10 000. <br> L'option par défaut est 5 000. 
7. Sous **Destination Azure**, procédez comme suit : 
    - **ID de l'espace de travail** : Collez l’ID de l’espace de travail ci-dessous. Ce champ est obligatoire.
    - **Clé primaire** : Collez la clé primaire ci-dessous. Ce champ est obligatoire.
    - **Nom de journal personnalisé** : Entrez le nom de journal personnalisé dans l’espace de travail Log Analytics du portail Microsoft Azure vers lequel transférer les événements. L'option par défaut est SymantecICDx. Ce champ est obligatoire.
8. Cliquez sur *Enregistrer* pour terminer la configuration du redirecteur. 
9. Pour démarrer le redirecteur, sous **Options**, cliquez sur **Plus**, puis sur **Démarrer**.
10. Pour utiliser le schéma pertinent dans Log Analytics pour les événements de Symantec ICDx, recherchez **SymantecICDx_CL**.


## <a name="validate-connectivity"></a>Valider la connectivité

Plus de 20 minutes peuvent être nécessaires avant que vos journaux ne commencent à apparaître dans Log Analytics. 



## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Symantec ICDx à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Utilisez des classeurs](tutorial-monitor-your-data.md) pour superviser vos données.


