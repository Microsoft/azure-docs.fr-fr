---
title: Surveillance et alertes Azure Key Vault | Microsoft Docs
description: Créez un tableau de bord pour surveiller l'intégrité de votre coffre de clés et configurer des alertes.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/06/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to learn the options available to monitor the health of my vaults
ms.openlocfilehash: 9195bb59264731914740e1cca902707603e3502d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018118"
---
# <a name="monitoring-and-alerting-for-azure-key-vault"></a>Surveillance et alertes Azure Key Vault

## <a name="overview"></a>Vue d’ensemble

Lorsque vous commencez à utiliser un coffre de clés pour stocker vos secrets de production, vous devez impérativement surveiller son intégrité pour veiller au bon fonctionnement de votre service. Dès que vous entamerez la mise à l'échelle votre service, le nombre de requêtes envoyées à votre coffre de clés augmentera. Cela risque d'augmenter la latence de vos requêtes et, dans les cas les plus extrêmes, de les limiter, ce qui aura un impact sur les performances de votre service. Vous devez également être alerté si votre coffre de clés envoie un nombre inhabituel de codes d'erreur afin d'être rapidement informé de tout problème de stratégie d'accès ou de configuration du pare-feu. Ce document couvre les sujets suivants :

+ Métriques Key Vault de base à surveiller
+ Configurer les métriques et créer un tableau de bord 
+ Créer des alertes lorsque les seuils spécifiés sont atteints 

## <a name="basic-key-vault-metrics-to-monitor"></a>Métriques Key Vault de base à surveiller

+ Disponibilité du coffre  
+ Saturation du coffre 
+ Latence de l'API de service 
+ Nombre total d'accès à l'API de service (filtrer par type d'activité) 
+ Codes d'erreur (filtrer par code d'état) 

**Disponibilité du coffre** : cette métrique doit toujours être de 100 %. Elle doit impérativement être surveillée car elle peut rapidement vous indiquer si votre coffre de clés a subi une interruption. 

**Saturation du coffre** : le nombre de requêtes par seconde qu'un coffre de clés peut traiter repose sur le type d'opération effectuée. Pour certaines opérations relatives au coffre, le seuil de requêtes par seconde est plus bas. Cette métrique agrège l'utilisation totale de votre coffre de clés, tous types d'opérations confondus, pour fournir une valeur en pourcentage qui indique votre utilisation actuelle du coffre. Pour obtenir la liste complète des limites du service Key Vault, consultez le document suivant. [Limites du service Azure Key Vault](service-limits.md)

**Latence de l'API de service** : cette métrique indique la latence moyenne d'un appel au coffre de clés. Même si votre coffre de clés se trouve dans les limites de service, une utilisation intensive de celui-ci peut entraîner une latence qui fait échouer les applications situées en aval. 

**Nombre total d'accès à l'API** : cette métrique affiche tous les appels passés à votre coffre de clés. Vous pouvez ainsi identifier les applications qui appellent votre coffre de clés. 

**Codes d'erreur** : cette métrique indique si votre coffre de clés rencontre un nombre inhabituel d'erreurs. Pour obtenir la liste complète des codes d'erreur et des conseils de dépannage, consultez le document suivant. [Codes d'erreur de l'API REST Azure Key Vault](rest-error-codes.md)

## <a name="how-to-configure-metrics-and-create-a-dashboard"></a>Configurer les métriques et créer un tableau de bord

1. Connectez-vous au portail Azure.
2. Accédez à votre coffre de clés.
3. Sous **Supervision**, sélectionnez **Métriques**. 

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence l'option Métriques dans la section Surveillance.](../media/alert-1.png)

4. Modifiez le titre du graphique en fonction de ce que vous souhaitez voir sur votre tableau de bord. 
5. Sélectionnez l'étendue. Dans cet exemple, nous allons sélectionner un seul coffre de clés. 
6. Sélectionnez la métrique **Disponibilité globale du coffre** et l'agrégation **Moy.** 
7. Remplacez l'intervalle de temps par Dernières 24 heures et la Granularité temporelle par 1 minute. 

> [!div class="mx-imgBorder"]
> ![Capture d'écran illustrant la métrique Disponibilité globale du coffre.](../media/alert-2.png)

8. Répétez les étapes ci-dessus pour les métriques Saturation du coffre et Latence de l'API de service. Sélectionnez **Épingler au tableau de bord** pour enregistrer vos métriques dans un tableau de bord. 

> [!IMPORTANT]
> Sélectionnez « Épingler au tableau de bord » et enregistrez toutes les métriques que vous configurez. Si vous quittez la page et que vous y revenez sans l'enregistrer, vos modifications de configuration seront perdues. 

9. Pour surveiller tous les types d'opérations sur le coffre de clés, utilisez la métrique **Nombre total d'accès à l'API de service** et sélectionnez **Appliquer le fractionnement par type d'activité**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran représentant le bouton Appliquer le fractionnement.](../media/alert-3.png)

10. Pour surveiller les codes d'erreur sur le coffre de clés, utilisez la métrique **Résultats totaux de l'API de service**, puis sélectionnez **Appliquer le fractionnement par type d'activité**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran dans laquelle la métrique Résultats totaux de l'API de service est sélectionnée.](../media/alert-4.png)

Vous disposez à présent d'un tableau de bord semblable à celui-ci. Vous pouvez cliquer sur les 3 points situés en haut à droite de chaque vignette, et vous pouvez réorganiser et redimensionner les vignettes selon vos besoins. 

Une fois le tableau de bord enregistré et publié, une nouvelle ressource est créée dans votre abonnement Azure. Pour y accéder à tout moment, recherchez « tableau de bord partagé ». 

> [!div class="mx-imgBorder"]
> ![Capture d'écran représentant le tableau de bord publié.](../media/alert-5.png)

## <a name="how-to-configure-alerts-on-your-key-vault"></a>Configurer des alertes sur votre coffre de clés 

Cette section vous explique comment configurer des alertes sur votre coffre de clés afin de pouvoir alerter votre équipe de la nécessité de prendre des mesures immédiates si l'état de votre coffre de clés n'est pas sain. Vous pouvez configurer des alertes qui envoient un e-mail, de préférence à un DL d'équipe, déclenchent une notification Event Grid, ou appellent ou envoient un SMS à un numéro de téléphone. Vous pouvez également choisir des alertes statiques basées sur une valeur fixe, ou une alerte dynamique qui vous alertera si une métrique surveillée dépasse la limite moyenne de votre coffre de clés un certain nombre de fois dans un intervalle de temps défini. 

> [!IMPORTANT]
> Veuillez noter qu'il peut s'écouler jusqu'à 10 minutes avant que les alertes nouvellement configurées ne commencent à envoyer des notifications. 

### <a name="configure-an-action-group"></a>Configurer un groupe d'actions 

Un groupe d'actions est une liste configurable de notifications et de propriétés.

1. Connectez-vous au portail Azure.
2. Recherchez **Alertes** dans la zone de recherche.
3. Sélectionnez **Gérer les actions**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence le bouton Gérer les actions.](../media/alert-6.png)

4. Sélectionnez **+ Ajouter un groupe d'actions**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence le bouton + Ajouter un groupe d'actions.](../media/alert-7.png)

5. Choisissez le **Type d'action** à associer à votre groupe d'actions. Pour les besoins de cet exemple, nous allons créer une alerte par e-mail.

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence les champs nécessaires à l'ajout d'un groupe d'actions.](../media/alert-8.png)

> [!div class="mx-imgBorder"]
> ![Capture d'écran illustrant ce qui est nécessaire pour ajouter un message d'alerte par e-mail ou par SMS.](../media/alert-9.png)

6. Cliquez sur **OK** au bas de la page. Vous venez de créer un groupe d'actions. 

Maintenant que vous avez configuré un groupe d'actions, vous allez configurer les seuils d'alerte du coffre de clés. 

### <a name="configure-alert-thresholds"></a>Configurer les seuils d'alerte 

1. Sélectionnez votre coffre de clés sur le portail Azure, puis choisissez **Alertes** sous **Surveillance**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran illustrant l'option de menu Alertes dans la section Surveillance.](../media/alert-10.png)

2. Sélectionnez **Nouvelle règle d'alerte**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran illustrant le bouton + Nouvelle règle d'alerte.](../media/alert-11.png)

3. Sélectionnez l'étendue de votre règle d'alerte. Vous pouvez sélectionner un ou plusieurs coffres. 

> [!IMPORTANT]
> Veuillez noter que lorsque vous sélectionnez plusieurs coffres pour l'étendue de vos alertes, tous les coffres sélectionnés doivent se trouver dans la même région. Vous devez configurer des règles d'alerte distinctes pour les coffres situés dans des régions différentes. 

> [!div class="mx-imgBorder"]
> ![Capture d'écran montrant comment sélectionner un coffre.](../media/alert-12.png)

4. Sélectionnez les conditions relatives à vos alertes. Vous pouvez choisir l'un des signaux suivants et définir votre logique d'alerte. L'équipe Key Vault recommande de configurer les seuils d'alerte suivants. 

    + La disponibilité du coffre de clés est inférieure à 100 % (seuil statique)
    + La latence du coffre de clés est supérieure à 500 ms (seuil statique) 
    + La saturation globale du coffre est supérieure à 75 % (seuil statique) 
    + La saturation globale du coffre est supérieure à la moyenne (seuil dynamique)
    + Le nombre total de codes d'erreur est supérieur à la moyenne (seuil dynamique) 

> [!div class="mx-imgBorder"]
> ![Capture d'écran montrant où sélectionner les conditions des alertes.](../media/alert-13.png)

### <a name="example-1-configuring-a-static-alert-threshold-for-latency"></a>Exemple 1 : Configuration d'un seuil d'alerte statique pour la latence

Sélectionnez **Latence globale de l'API de service** comme nom de signal.
> [!div class="mx-imgBorder"]
> ![Capture d'écran du nom de signal Latence globale de l'API de service.](../media/alert-14.png)

Veuillez consulter les paramètres de configuration suivants.

+ Définissez le seuil sur **Statique**. 
+ Définissez l'opérateur sur **Supérieur à**.
+ Définissez le Type d'agrégation sur **Moyenne**.
+ Définissez la Valeur de seuil sur **500**.
+ Définissez la Période d'agrégation sur **5 minutes**.
+ Définissez la Fréquence d'évaluation sur **1 minute**.
+ Sélectionnez **Terminé**  

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence la logique d'alerte configurée.](../media/alert-15.png)

### <a name="example-2-configuring-a-dynamic-alert-threshold-for-vault-saturation"></a>Exemple 2 : Configuration d'un seuil d'alerte dynamique pour la saturation du coffre 

Lorsque vous utilisez une alerte dynamique, vous pouvez consulter les données historiques du coffre de clés que vous avez sélectionné. La zone bleue représente l'utilisation moyenne de votre coffre de clés. La zone rouge indique les pics susceptibles de déclencher une alerte si d'autres critères de la configuration d'alerte sont remplis. Les points rouges indiquent les cas de violations dans lesquels les critères d'alerte ont été remplis pendant la fenêtre temporelle agrégée. Vous pouvez définir une alerte pour qu'elle se déclenche après un certain nombre de violations sur une période déterminée. Si vous ne souhaitez pas inclure de données passées, une option vous permet d'exclure les anciennes données ci-dessous dans les paramètres avancés. 

> [!div class="mx-imgBorder"]
> ![Capture d'écran d'un graphique représentant la saturation globale du coffre.](../media/alert-16.png)

Veuillez consulter les paramètres de configuration suivants.

+ Définissez le seuil sur **Dynamique**. 
+ Définissez l'opérateur sur **Supérieur à**.
+ Définissez le Type d'agrégation sur **Moyenne**.
+ Définissez la Sensibilité du seuil sur **Moyenne**.
+ Définissez la Période d'agrégation sur **5 minutes**.
+ Définissez la Fréquence d'évaluation sur **1 minute**.
+ **Facultatif** Configurez les paramètres avancés. 
+ Sélectionnez **Terminé**

> [!div class="mx-imgBorder"]
> ![Capture d'écran du portail Azure](../media/alert-17.png)

5. Ajoutez le groupe d'actions que vous avez configuré.

> [!div class="mx-imgBorder"]
> ![Capture d'écran montrant comment ajouter un groupe d'actions.](../media/alert-18.png)

6. Activez l'alerte et attribuez une gravité.

> [!div class="mx-imgBorder"]
> ![Capture d'écran montrant où activer l'alerte et attribuer une gravité.](../media/alert-19.png)

7. Créez l’alerte 


## <a name="next-steps"></a>Étapes suivantes

Félicitations, vous avez créé un tableau de bord de surveillance et configuré des alertes pour votre coffre de clés ! Lorsque vous aurez suivi toutes les étapes décrites ci-dessus, vous recevrez des alertes par e-mail chaque fois que votre coffre de clés répondra aux critères d'alerte que vous avez configurés. Voici un exemple. Utilisez les outils que vous avez configurés dans cet article pour surveiller activement l'intégrité de votre coffre de clés. 

### <a name="example-email-alert"></a>Exemple d'alerte par e-mail 

> [!div class="mx-imgBorder"]
> ![Capture d'écran mettant en évidence les informations nécessaires à la configuration d'une alerte par e-mail.](../media/alert-20.png)
