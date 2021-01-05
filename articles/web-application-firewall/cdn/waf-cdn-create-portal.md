---
title: 'Tutoriel : Créer une stratégie WAF pour Azure CDN - Portail Azure'
description: Dans ce tutoriel, vous allez apprendre à créer une stratégie de pare-feu d’applications web (WAF) sur Azure CDN à l’aide du portail Azure.
author: vhorne
ms.service: web-application-firewall
services: web-application-firewall
ms.topic: tutorial
ms.date: 09/16/2020
ms.author: victorh
ms.openlocfilehash: 9579d0da3347bdd4ecc627662cee42f909cbfaf7
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132769"
---
# <a name="tutorial-create-a-waf-policy-on-azure-cdn-using-the-azure-portal"></a>Tutoriel : Créer une stratégie WAF sur Azure CDN à l’aide du portail Azure

Ce tutoriel explique comment créer une stratégie de pare-feu d’applications web (WAF) Azure de base et comment l’appliquer à un point de terminaison sur Azure Content Delivery Network (CDN).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une stratégie de pare-feu d’applications web (WAF).
> * L’associer à un point de terminaison CDN. Vous pouvez associer une stratégie WAF uniquement à des points de terminaison hébergés sur la référence SKU **Azure CDN Standard de Microsoft** .
> * Configurer des règles WAF

## <a name="prerequisites"></a>Prérequis

Créez un profil et un point de terminaison Azure CDN en suivant les instructions fournies dans [Démarrage rapide : Créer un point de terminaison et un profil de réseau de distribution de contenu Azure](../../cdn/cdn-create-new-endpoint.md). 

## <a name="create-a-web-application-firewall-policy"></a>Créer une stratégie de pare-feu d’applications web

Commencez par créer une stratégie WAF de base avec un ensemble de règles par défaut managé via le portail.

1. Dans l’angle supérieur gauche de l’écran, sélectionnez **Créer une ressource** > recherchez **WAF** > sélectionnez **Pare-feu d’applications web** > sélectionnez **Créer** .
2. Sous l’onglet **De base** dans la page **Créer une stratégie WAF** , entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis sélectionnez **Vérifier + créer**  :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Stratégie pour            |Sélectionnez Azure CDN (préversion).|
    | Abonnement            |Sélectionnez le nom de votre abonnement Front Door.|
    | Resource group          |Sélectionnez le nom de votre groupe de ressources Front Door.|
    | Nom de stratégie             |Entrez un nom unique pour votre stratégie WAF.|

   :::image type="content" source="../media/waf-cdn-create-portal/basic.png" alt-text="Capture d’écran de la page Créer une stratégie WAF, avec le bouton Vérifier + créer et des valeurs entrées pour différents paramètres." border="false":::

3. Sous l’onglet **Association** de la page **Créer une stratégie WAF** , sélectionnez **Ajouter un point de terminaison CDN** , entrez les paramètres suivants, puis sélectionnez **Ajouter**  :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Profil CDN              | Sélectionnez le nom de votre profil CDN.|
    | Point de terminaison           | Sélectionnez le nom de votre point de terminaison, puis sélectionnez **Ajouter** .|
    
    > [!NOTE]
    > Si le point de terminaison est associé à une stratégie WAF, il apparaît grisé. Vous devez d’abord supprimer le point de terminaison de la stratégie associée, puis le réassocier à une nouvelle stratégie WAF.
1. Sélectionnez **Vérifier + créer** , puis sélectionnez **Créer** .

## <a name="configure-web-application-firewall-policy-optional"></a>Configurer une stratégie de pare-feu d’applications web (facultatif)

### <a name="change-mode"></a>Changer de mode

Quand vous créez une stratégie WAF, celle-ci est par défaut en mode *Détection* . En mode *Détection* , WAF ne bloque aucune demande. Au lieu de cela, les demandes correspondant aux règles de WAF sont journalisées dans les journaux WAF.

Pour voir le fonctionnement de WAF, passez du mode *Détection* au mode *Prévention* . En mode *Prévention* , les requêtes qui correspondent à des règles définies dans un ensemble de règles par défaut sont bloquées et enregistrées dans les journaux WAF.

 :::image type="content" source="../media/waf-cdn-create-portal/policy.png" alt-text="Capture d’écran de la page Créer une stratégie WAF, avec le bouton Vérifier + créer et des valeurs entrées pour différents paramètres." border="false":::

### <a name="custom-rules"></a>Règles personnalisées

Pour créer une règle personnalisée, sélectionnez **Ajouter une règle personnalisée** sous la section **Règles personnalisées** . Cette action ouvre la page de configuration de règle personnalisée. Il existe deux types de règles personnalisées : la **règle de correspondance** et la **règle de limite de débit** .

La capture d’écran suivante montre une règle de correspondance personnalisée permettant de bloquer une demande si la chaîne de requête contient la valeur **blockme** .

:::image type="content" source="../media/waf-cdn-create-portal/custommatch.png" alt-text="Capture d’écran de la page Créer une stratégie WAF, avec le bouton Vérifier + créer et des valeurs entrées pour différents paramètres." border="false":::

Les règles de limite de débit nécessitent deux champs supplémentaires : **Durée de la limite de débit** et **Seuil limite du débit (demandes)** comme indiqué dans l’exemple suivant :

:::image type="content" source="../media/waf-cdn-create-portal/customrate.png" alt-text="Capture d’écran de la page Créer une stratégie WAF, avec le bouton Vérifier + créer et des valeurs entrées pour différents paramètres." border="false":::

### <a name="default-rule-set-drs"></a>Ensemble de règles par défaut

L’ensemble de règles par défaut managées par Azure est activé par défaut. Pour désactiver une règle spécifique dans un groupe de règles, développez les règles de celui-ci, cochez la case devant le numéro de la règle, puis sélectionnez **Désactiver** sous l’onglet situé au-dessus. Pour changer le type d’action associée à une règle dans l’ensemble de règles, sélectionnez la case à cocher devant le numéro de la règle, puis sélectionnez l’onglet **Changer l’action** ci-dessus.

 :::image type="content" source="../media/waf-cdn-create-portal/managed2.png" alt-text="Capture d’écran de la page Créer une stratégie WAF, avec le bouton Vérifier + créer et des valeurs entrées pour différents paramètres." border="false":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources et toutes les ressources associées.


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez en plus sur le pare-feu d’application web Azure](../overview.md).
