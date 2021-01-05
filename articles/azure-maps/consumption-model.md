---
title: Modèles de consommation du véhicule pour le routage | Microsoft Azure Maps
description: 'En savoir plus sur les modèles de consommation pris en charge par Azure Maps : combustion et électricité. Découvrez les paramètres utilisés par chaque modèle et les contraintes des paramètres.'
author: subbarayudukamma
ms.author: skamma
ms.date: 05/08/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: b44186d783a249192a8c13ee97063034ee319df7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013137"
---
# <a name="consumption-model"></a>Modèle de consommation

Le service Route fournit un ensemble de paramètres permettant de décrire en détail le modèle de consommation spécifique à un véhicule.
Selon la valeur de **vehicleEngineType**, deux grands modèles de consommation sont pris en charge : _Combustion_ et _Electric_. Le système considère comme incorrect le fait de spécifier des paramètres appartenant à des modèles différents dans la même requête. De plus, les paramètres du modèle de consommation ne peuvent pas être utilisés avec les valeurs suivantes pour **travelMode** : _bicycle_ (vélo) et _pedestrian_ (piéton).

## <a name="parameter-constraints-for-consumption-model"></a>Contraintes des paramètres pour le modèle de consommation

Dans les deux modèles de consommation, il existe des dépendances lors de la spécification des paramètres. Cela signifie que la spécification explicite de certains paramètres nécessite la spécification de certains autres paramètres. Voici les dépendances qu’il faut connaître :

* Tous les paramètres nécessitent que l’utilisateur spécifie **constantSpeedConsumption**. Il n’est pas possible de spécifier un autre paramètre de modèle de consommation si **constantSpeedConsumption** n’est pas spécifié. Le paramètre **vehicleWeight** est une exception pour cette exigence.
* **accelerationEfficiency** et **decelerationEfficiency** doivent toujours être spécifiés par paire (c’est-à-dire tous les deux ou aucun).
* Si **accelerationEfficiency** et **decelerationEfficiency** sont spécifiés, le produit de leurs valeurs ne doit pas être supérieur à 1 (pour empêcher tout mouvement perpétuel).
* **uphillEfficiency** et **downhillEfficiency** doivent toujours être spécifiés par paire (c’est-à-dire tous les deux ou aucun).
* Si **uphillEfficiency** et **downhillEfficiency** sont spécifiés, le produit de leurs valeurs ne doit pas être supérieur à 1 (pour empêcher tout mouvement perpétuel).
* Si les paramètres \*__Efficiency__ sont spécifiés par l’utilisateur, **vehicleWeight** doit également être spécifié. Lorsque **vehicleEngineType** a pour valeur _combustion_, **fuelEnergyDensityInMJoulesPerLiter** doit également être spécifié.
* **maxChargeInkWh** et **currentChargeInkWh** doivent toujours être spécifiés par paire (c’est-à-dire tous les deux ou aucun).

> [!NOTE]
> Si seul **constantSpeedConsumption** est spécifié, aucun autre aspect de la consommation, comme la déclivité ou l’accélération du véhicule, n’est pris en compte dans les calculs de consommation.

## <a name="combustion-consumption-model"></a>Modèle de consommation Combustion

Le modèle de consommation Combustion est utilisé lorsque **vehicleEngineType** a la valeur _combustion_.
La liste des paramètres appartenant à ce modèle figure ci-dessous. Reportez-vous à la section Paramètres pour obtenir une description détaillée.

* constantSpeedConsumptionInLitersPerHundredkm
* vehicleWeight
* currentFuelInLiters
* auxiliaryPowerInLitersPerHour
* fuelEnergyDensityInMJoulesPerLiter
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="electric-consumption-model"></a>Modèle de consommation Electric

Le modèle de consommation Electric est utilisé lorsque **vehicleEngineType** a la valeur _electric_.
La liste des paramètres appartenant à ce modèle figure ci-dessous. Reportez-vous à la section Paramètres pour obtenir une description détaillée.

* constantSpeedConsumptionInkWhPerHundredkm
* vehicleWeight
* currentChargeInkWh
* maxChargeInkWh
* auxiliaryPowerInkW
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="sensible-values-of-consumption-parameters"></a>Valeurs sensibles des paramètres de consommation

Un ensemble spécifique de paramètres de consommation peut être rejeté, même s’il répond à toutes les exigences explicites. Cela se produit quand la valeur d’un paramètre spécifique ou une combinaison de valeurs de plusieurs paramètres est considérée comme engendrant des mesures déraisonnables pour les valeurs de consommation. Cela indique probablement une erreur d’entrée, car la prise en charge de toutes les valeurs sensibles des paramètres de consommation se fait de manière appropriée. En cas de rejet d’un ensemble spécifique de paramètres de consommation, le message d’erreur associé contient une description textuelle de la ou des raisons de ce rejet.
Les descriptions détaillées des paramètres présentent des exemples de valeurs sensibles pour les deux modèles.
