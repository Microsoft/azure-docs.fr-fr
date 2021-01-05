---
title: Personnaliser la solution Usine connectée - Azure | Microsoft Docs
description: Découvrez comment personnaliser le comportement de l’accélérateur de solution Usine connectée.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.devlang: csharp
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: dobett
ms.openlocfilehash: a092859517ee6ac41e62e55fe95f72b8bfdf87d5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011198"
---
# <a name="customize-how-the-connected-factory-solution-displays-data-from-your-opc-ua-servers"></a>Personnaliser la façon dont l’accélérateur de solution Usine connectée présente les données à partir de vos serveurs OPC UA

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

La solution Usine connectée agrège et affiche les données des serveurs OPC UA qui y sont connectés. Vous pouvez parcourir les serveurs OPC UA et leur envoyer des commandes dans votre solution. Pour plus d’informations sur l’UA OPC, consultez les [questions fréquentes (FAQ) relatives à Usine connectée](iot-accelerators-faq-cf.md).

Des exemples de données agrégées dans la solution incluent l’efficacité globale des équipements (OEE) et les indicateurs de performance clés (KPI), que vous pouvez afficher dans le tableau de bord au niveau d’une usine, d’une ligne de production et d’un poste. La capture d’écran suivante illustre les valeurs d’OEE et de KPI pour le poste d’assemblage **Assembly** de la ligne de production **Production line 1** dans l’usine de **Munich** :

![Exemple de valeurs d’OEE et de KPI dans la solution][img-oee-kpi]

La solution permet d’afficher des informations détaillées pour des éléments de données spécifiques des serveurs OPC UA appelés *postes*. La capture d’écran suivante illustre des graphiques du nombre d’articles fabriqués à partir d’un poste spécifique :

![Graphiques du nombre d’éléments fabriqués][img-manufactured-items]

Si vous cliquez sur l’un des graphiques, vous pouvez explorer les données plus en détail à l’aide de Time Series Insights (TSI) :

![Explorer les données à l’aide de Time Series Insights][img-tsi]

Cet article aborde les points suivants :

- Comment rendre disponibles les données dans les différentes vues de la solution.
- Comment personnaliser le mode d’affichage des données par la solution.

## <a name="data-sources"></a>Sources de données

La solution Usine connectée affiche les données des serveurs OPC UA qui y sont connectés. L’installation par défaut inclut plusieurs serveurs OPC UA exécutant une simulation d’usine. Vous pouvez ajouter vos propres serveurs OPC UA qui [se connectent via une passerelle][lnk-connect-cf] à votre solution.

Vous pouvez parcourir les éléments de données qu’un serveur OPC UA peut envoyer à votre solution dans le tableau de bord :

1. Choisissez **Navigateur** pour accéder à la vue **Sélectionner un serveur OPC UA** :

    ![Accéder à la vue Select an OPC UA server (Sélectionner un serveur OPC UA)][img-select-server]

1. Sélectionnez un serveur et cliquez sur **Connect** (Connexion). Lorsque l’avertissement de sécurité s’affiche, cliquez sur **Proceed** (Continuer).

    > [!NOTE]
    > Cet avertissement s’affiche une seule fois pour chaque serveur et établit une relation d’approbation entre le tableau de bord de la solution et le serveur.

1. Vous pouvez maintenant parcourir les éléments de données que le serveur peut envoyer à la solution. Les éléments qui sont envoyés à la solution présentent une coche :

    ![Éléments publiés][img-published]

1. Si vous êtes *administrateur* de la solution, vous pouvez choisir de publier un élément de données pour le rendre disponible dans la solution Usine connectée. En tant qu’administrateur, vous pouvez également modifier la valeur des éléments de données et appeler des méthodes sur le serveur OPC UA.

## <a name="map-the-data"></a>Mapper les données

La solution Usine connectée mappe et agrège les éléments de données publiés à partir du serveur OPC UA dans les différentes vues de la solution. La solution Usine connectée est déployée dans votre compte Azure quand vous la provisionnez. Un fichier JSON de la solution Visual Studio Usine connectée stocke ces informations de mappage. Vous pouvez afficher et modifier ce fichier de configuration JSON dans la solution Visual Studio Usine connectée. Vous pouvez redéployer la solution une fois que vous apportez une modification.

Vous pouvez utiliser le fichier de configuration pour :

- Modifier les usines, les lignes de production existantes et les postes simulés existants.
- Mapper les données des serveurs OPC UA réels que vous connectez à la solution.

Pour plus d’informations sur le mappage et l’agrégation des données en fonction de vos besoins, consultez [Guide pratique pour configurer l’accélérateur de solution Usine connectée](iot-accelerators-connected-factory-configure.md).

## <a name="deploy-the-changes"></a>Déployer les modifications

Une fois que vous avez apporté toutes les modifications nécessaires au fichier **ContosoTopologyDescription.json**, vous devez redéployer la solution Usine connectée dans votre compte Azure.

Le référentiel **azure-iot-connected-factory** inclut un script PowerShell **build.ps1** que vous pouvez utiliser pour régénérer et déployer la solution.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’accélérateur de solution Usine connectée, consultez les articles suivants :

* [Autorisations sur le site azureiotsolutions.com][lnk-permissions]
* [Questions fréquentes (FAQ) sur l’Usine connectée](iot-accelerators-faq-cf.md)
* [FORUM AUX QUESTIONS][lnk-faq]


[img-oee-kpi]: ./media/iot-accelerators-connected-factory-customize/oeenadkpi.png
[img-manufactured-items]: ./media/iot-accelerators-connected-factory-customize/manufactured.png
[img-tsi]: ./media/iot-accelerators-connected-factory-customize/tsi.png
[img-select-server]: ./media/iot-accelerators-connected-factory-customize/selectserver.png
[img-published]: ./media/iot-accelerators-connected-factory-customize/published.png


[lnk-permissions]: iot-accelerators-permissions.md
[lnk-faq]: iot-accelerators-faq.md