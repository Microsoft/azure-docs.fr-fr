---
title: Connecter les données du pare-feu d’applications web (WAF) à Azure Sentinel
description: Découvrez comment connecter les données du WAF Azure à Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 05/07/2020
ms.author: yelevin
ms.openlocfilehash: c554f3582e67622a5a1739c9e410328c902d491b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655899"
---
# <a name="connect-data-from-azure-web-application-firewall-waf"></a>Connecter les données du pare-feu d’application web (WAF) Azure

Les applications web sont de plus en plus visées par des attaques malveillantes qui exploitent des vulnérabilités connues. Le pare-feu d’applications web (WAF) Azure offre une protection centralisée de vos applications web contre les exploitations de faille de sécurité et des menaces telles que des injections de code et des scripts intersites. Le WAF Azure peut être déployé sur le service [Azure Application Gateway](../web-application-firewall/ag/ag-overview.md), le service [Azure Front Door](../web-application-firewall/afds/afds-overview.md), et via une stratégie de WAF [Azure Content Delivery Network (CDN)](../web-application-firewall/cdn/cdn-overview.md) (en préversion).
Vous pouvez connecter des journaux de WAF Azure à Azure Sentinel, ce qui vous permet d’afficher les données des journaux dans des classeurs, de les utiliser pour créer des alertes personnalisées et de les incorporer pour améliorer votre investigation.

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel.

## <a name="connect-to-azure-waf"></a>Connexion au WAF Azure

### <a name="instructions-tab"></a>Onglet Instructions

1. Dans le menu de navigation d’Azure Sentinel, sélectionnez **Connecteurs de données**.

1. Sélectionnez **Pare-feu d’applications web (WAF) Azure** dans la galerie de connecteurs de données, puis sélectionnez **Ouvrir la page du connecteur** dans le volet de visualisation.

1. Sélectionnez le lien pour le type de ressource WAF dont vous souhaitez connecter les journaux (**Ouvrir la ressource Application Gateway >** , **Ouvrir la ressource Front Door >** ou **Ouvrir la stratégie WAF CDN (Content Delivery Network)** ), puis, une fois dans l’écran de la liste des ressources, sélectionnez une ressource WAF dans la liste.

    1. Dans le menu de navigation de la ressource WAF, sélectionnez **Paramètres de diagnostic**.

    1. Sélectionnez **+ Ajouter un paramètre de diagnostic** en bas de la liste.

    1. Dans l’écran **Paramètres de diagnostic**, tapez un nom dans le champ **Nom des paramètres de diagnostic**.

    1. Activez la case à cocher **Envoyer à Log Analytics**. Deux nouveaux champs s’affichent sous celle-ci. Choisissez l’**Abonnement** approprié et l’**Espace de travail Log Analytics** (où se trouve Azure Sentinel).

    1. Activez les cases à cocher des types de règles dont vous souhaitez ingérer les journaux. Nos recommandations pour chaque type de ressource sont les suivantes :

        | Ressource | Journaux à sélectionner pour l’ingestion |
        |----------|------------------------------|
        | Application Gateway | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog |
        | Front Door          | FrontdoorAccessLog<br>FrontdoorWebApplicationFirewallLog |
        | Stratégie WAF de CDN      | WebApplicationFirewallLogs |
        |

    1. Sélectionnez **Enregistrer**.

### <a name="next-steps-tab"></a>Onglet Étapes suivantes

- Consultez les classeurs, les exemples de requêtes et les modèles de règle d’analytique recommandés qui sont fournis avec le connecteur de données **Pare-feu d’applications web Azure** pour obtenir des insights sur les données de votre journal WAF Azure.

- Pour interroger les données de WAF Azure dans **Journaux**, tapez **AzureDiagnostics** dans la fenêtre de requête.

> [!NOTE]
>
> Avec ce connecteur de données particulier, les indicateurs d’état de connectivité (une bande de couleur dans la galerie de connecteurs de données et des icônes de connexion en regard des noms de types de données) s’affichent comme *Connectés* (en vert) uniquement si les données ont été ingérées au cours des deux dernières semaines. Après deux semaines passées sans ingestion de données, le connecteur déconnecté. Quand plus de données arrivent, l’état *connecté* est renvoyé.

## <a name="next-steps"></a>Étapes suivantes
Ce document vous a appris à connecter les journaux du WAF Azure à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).