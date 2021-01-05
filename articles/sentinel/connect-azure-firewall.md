---
title: Connecter les données de Pare-feu Azure à Azure Sentinel
description: Découvrez comment connecter des données de Pare-feu Azure à Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 08/30/2020
ms.author: yelevin
ms.openlocfilehash: 4dffaac329f1581d9082fd8ab2c314f52b1730ab
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656018"
---
# <a name="connect-data-from-azure-firewall"></a>Connecter des données à partir de Pare-feu Azure

> [!IMPORTANT]
> Le connecteur de données Pare-feu Azure dans Azure Sentinel est disponible en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Pare-feu Azure est un service de sécurité réseau informatique géré qui protège vos ressources Réseau virtuel Azure. Il s’agit d’un service de pare-feu entièrement avec état, pourvu d’une haute disponibilité intégrée et de l’extensibilité sans limites du cloud. 

Vous pouvez connecter des journaux Pare-feu Azure à Azure Sentinel, ce qui vous permet d’afficher les données des journaux dans des classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer pour améliorer votre investigation.

En savoir plus sur la [supervision des journaux Pare-feu Azure](../firewall/firewall-diagnostics.md).

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel.

## <a name="connect-to-azure-firewall"></a>Se connecter à Pare-feu Azure
    
1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**.

1. Sélectionnez **Pare-feu Azure** dans la galerie de connecteurs de données, puis sélectionnez **Ouvrir la page du connecteur** dans le volet de visualisation.

1. Activez **Journaux de diagnostic** sur tous les pare-feu dont vous souhaitez connecter les journaux :

    1. Sélectionnez le lien **Ouvrir la ressource Pare-feu Azure >**.

    1. Dans le menu de navigation **Pare-feu**, sélectionnez **Paramètres de diagnostic**.

    1. Sélectionnez **+ Ajouter un paramètre de diagnostic** en bas de la liste.

    1. Dans l’écran **Paramètres de diagnostic**, entrez un nom dans le champ **Nom des paramètres de diagnostic**.
    
    1. Activez la case à cocher **Envoyer à Log Analytics**. Deux nouveaux champs s’affichent sous celle-ci. Choisissez l’**Abonnement** approprié et l’**Espace de travail Log Analytics** (où se trouve Azure Sentinel).

    1. Activez les cases à cocher des types de règles dont vous souhaitez ingérer les journaux. Nous vous recommandons d’utiliser **AzureFirewallApplicationRule** et **AzureFirewallNetworkRule**.

    1. En haut de l’écran, sélectionnez **Enregistrer**.

1. Pour utiliser le schéma pertinent dans Log Analytics pour les alertes de Pare-feu Azure, recherchez **AzureDiagnostics**.

> [!NOTE]
>
> Avec ce connecteur de données particulier, les indicateurs d’état de connectivité (une bande de couleur dans la galerie de connecteurs de données et des icônes de connexion en regard des noms de types de données) s’affichent comme *Connectés* (en vert) uniquement si les données ont été ingérées au cours des deux dernières semaines. Après deux semaines passées sans ingestion de données, le connecteur déconnecté. Quand plus de données arrivent, l’état *connecté* est renvoyé.

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a appris à connecter les journaux de Pare-feu Azure à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).