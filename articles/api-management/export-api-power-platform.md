---
title: Exporter des API depuis Gestion des API Azure vers Power Platform | Microsoft Docs
description: Découvrez comment exporter des API depuis Gestion des API vers Power Platform.
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 05/01/2020
ms.author: apimpm
ms.openlocfilehash: 7c4d32dd63120a52fd7351977943574455e5cfad
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146652"
---
# <a name="export-apis-from-azure-api-management-to-the-power-platform"></a>Exporter des API depuis Gestion des API Azure vers Power Platform 

Les développeurs non professionnels qui utilisent Microsoft [Power Platform](https://powerplatform.microsoft.com) doivent souvent faire appel à des fonctionnalités métier développées par des développeurs professionnels et déployées dans Azure. [Gestion des API Azure](https://aka.ms/apimrocks) permet aux développeurs professionnels de publier leur service back-end en tant qu’API, et d’exporter facilement ces API vers Power Platform (Power Apps et Power Automate) en tant que connecteurs personnalisés destinés à être consommés par des développeurs non professionnels. 

Cet article décrit les étapes à suivre pour exporter des API depuis Gestion des API vers Power Platform. 

## <a name="prerequisites"></a>Prérequis

+ Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)
+ Vérifiez qu’il existe une API dans votre instance de Gestion des API que vous souhaitez exporter vers Power Platform
+ Veillez à disposer d’un [environnement](/powerapps/powerapps-overview#power-apps-for-admins) Power Apps ou Power Automate 

## <a name="export-an-api"></a>Exporter une API

1. Dans le portail Azure, accédez à votre service Gestion des API, puis sélectionnez **API** dans le menu.
2. Cliquez sur les trois points en regard de l’API que vous voulez exporter. 
3. Sélectionnez **Exporter**.
4. Sélectionnez **Power Apps et Power Automate**.
5. Choisissez un environnement vers lequel exporter l’API. 
6. Spécifiez un nom d’affichage, qui sera utilisé comme nom du connecteur personnalisé.  
7. (Facultatif) Si l’API est protégée par un serveur OAuth 2.0, vous devez également fournir des détails supplémentaires, notamment `Client ID`, `Client secret`, `Authorization URL`, `Token URL` et `Refresh URL`.  
8. Sélectionnez **Exporter**. 

Une fois l’exportation terminée, accédez à votre environnement Power Apps ou Power Automate. Vous verrez l’API en tant que connecteur personnalisé.

## <a name="next-steps"></a>Étapes suivantes

* [Découvrir plus d’informations sur Power Platform](https://powerplatform.microsoft.com/)
* [Découvrir les tâches courantes dans Gestion des API en suivant les tutoriels](./import-and-publish.md)
