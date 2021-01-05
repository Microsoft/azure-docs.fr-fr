---
title: Intégrer la Gestion des API Azure avec Azure Application Insights
titleSuffix: Azure API Management
description: Découvrez comment consigner et afficher des événements de la Gestion des API Azure dans Azure Application Insights.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/20/2018
ms.author: apimpm
ms.openlocfilehash: adb221c12af436135b1e740fdef7c5c0a0a7f0cb
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096034"
---
# <a name="how-to-integrate-azure-api-management-with-azure-application-insights"></a>Guide pratique pour intégrer la Gestion des API Azure avec Azure Application Insights

La Gestion des API Azure est parfaitement compatible avec Azure Application Insights, un service extensible permettant aux développeurs web de créer et de gérer des applications sur de multiples plateformes. Ce guide décrit la marche à suivre pour effectuer cette intégration, ainsi que des stratégies visant à réduire l’impact sur le fonctionnement de l’instance de service Gestion des API.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide, vous devrez disposer d’une instance de Gestion des API Azure. Si vous n’en avez pas, suivez d’abord le [tutoriel](get-started-create-service-instance.md).

## <a name="create-an-azure-application-insights-instance"></a>Créer une instance Azure Application Insights

Pour pouvoir utiliser Azure Application Insights, il faut commencer par créer une instance du service.

1. Ouvrez le **Portail Azure** et accédez à **Application Insights**.  
    ![Capture d’écran montrant comment accéder à Application Insights.](media/api-management-howto-app-insights/apim-app-insights-instance-1.png)  
2. Cliquez sur **+ Ajouter**.  
    ![Création App Insights](media/api-management-howto-app-insights/apim-app-insights-instance-2.png)  
3. Remplissez le formulaire. Sélectionnez le **Type d’application** **Général**.
4. Cliquez sur **Créer**.

## <a name="create-a-connection-between-azure-application-insights-and-azure-api-management-service-instance"></a>Créer une connexion entre Azure Application Insights et l’instance du service Gestion des API

1. Accédez à votre **instance de service Gestion des API Azure** sur le **Portail Azure**.
2. Sélectionnez **Application Insights** dans le menu de gauche.
3. Cliquez sur **+ Ajouter**.  
    ![Capture d’écran montrant où ajouter une nouvelle connexion.](media/api-management-howto-app-insights/apim-app-insights-logger-1.png)  
4. Sélectionnez l’instance **Application Insights** créée et indiquez une courte description.
5. Cliquez sur **Créer**.
6. Vous venez de créer un enregistreur d’événements Azure Application Insights avec une clé d’instrumentation. Il devrait maintenant apparaître dans la liste.  
    ![Capture d’écran montrant où afficher l’enregistreur d’événements Azure Application Insights nouvellement créé avec la clé d’instrumentation.](media/api-management-howto-app-insights/apim-app-insights-logger-2.png)  

> [!NOTE]
> En arrière-plan, une entité [Enregistreur d’événements](/rest/api/apimanagement/2019-12-01/logger/createorupdate) est créée dans votre instance de gestion des API, qui contient la clé d’instrumentation de l’instance Application Insights.

## <a name="enable-application-insights-logging-for-your-api"></a>Activer la journalisation Application Insights pour l’API

1. Accédez à votre **instance de service Gestion des API Azure** sur le **Portail Azure**.
2. Sélectionnez **API** dans le menu de gauche.
3. Cliquez sur votre API, dans ce cas **Demo Conference API**.
4. Accédez à l’onglet **Paramètres** de la barre supérieure.
5. Descendez jusqu’à la section **Journaux de diagnostic**.  
    ![Enregistreur d'événements App Insights](media/api-management-howto-app-insights/apim-app-insights-api-1.png)  
6. Cochez la case **Activer**.
7. Sélectionnez l’enregistreur d'événements joint dans la liste déroulante **Destination**.
8. Entrez **100** pour **Échantillonnage (%)** et cochez la case **Toujours consigner les erreurs**.
9. Cliquez sur **Enregistrer**.

> [!WARNING]
> Le remplacement de la valeur par défaut **0** dans le champ **Premiers octets du corps** peut considérablement réduire les performances de vos API.

> [!NOTE]
> En arrière-plan, une entité [Diagnostic](/rest/api/apimanagement/2019-12-01/diagnostic/createorupdate) nommée « applicationinsights » est créée au niveau de l’API.

| Nom du paramètre                        | Type de valeur                        | Description                                                                                                                                                                                                                                                                                                                                      |
|-------------------------------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Activer                              | boolean                           | Indique si la journalisation de cette API est activée.                                                                                                                                                                                                                                                                                                |
| Destination                         | Enregistreur d'événements Azure Application Insights | Indique l’enregistreur d'événements Azure Application Insights utilisé.                                                                                                                                                                                                                                                                                           |
| Échantillonnage (%)                        | Décimal                           | Valeurs comprises entre 0 et 100 (%). <br/> Indique le pourcentage de demandes qui seront consignées dans Azure Application Insights. Un échantillonnage à 0 % signifie qu’aucune demande ne sera consignée, tandis qu’un échantillonnage à 100 % indique qu’elles le seront toutes. <br/> Ce paramètre permet de réduire les conséquences des demandes de journalisation sur les performances d’Azure Application Insights (voir la section ci-dessous). |
| Toujours consigner les erreurs                   | boolean                           | Si ce paramètre est sélectionné, toutes les défaillances seront consignées dans Azure Application Insights, quel que soit le paramètre **Échantillonnage**.                                                                                                                                                                                                                  |
| Options de base : headers              | list                              | Précise quels en-têtes seront consignés dans Azure Application Insights pour les demandes et les réponses.  Par défaut, aucun en-tête n’est consigné.                                                                                                                                                                                                             |
| Options de base : Premiers octets du corps  | entier                           | Indique combien de premiers octets du corps seront consignés dans Azure Application Insights pour les demandes et les réponses.  Par défaut, le corps n’est pas consigné.                                                                                                                                                                                                    |
| Options avancées : Commentaires         |                                   | Spécifie le niveau de verbosité. Seules les traces personnalisées avec un niveau de gravité supérieur sont journalisées. Valeur par défaut : Information.                                                                                                                                                                                                                               |
| Options avancées : Demande du serveur frontal  |                                   | Précise si des *demandes frontales* seront consignées dans Azure Application Insights et, si oui, comment. Une *demande frontale* est une demande entrante dans le service Gestion des API Azure.                                                                                                                                                                        |
| Options avancées : Réponse du serveur frontal |                                   | Précise si des *réponses frontales* seront consignées dans Azure Application Insights et, si oui, comment. Une *réponse frontale* est une réponse sortante du service Gestion des API Azure.                                                                                                                                                                   |
| Options avancées : Demande principale   |                                   | Précise si des *demandes principales* seront consignées dans Azure Application Insights et, si oui, comment. Une *demande principale* est une demande sortante du service Gestion des API Azure.                                                                                                                                                                        |
| Options avancées : Réponse principale  |                                   | Précise si des *réponses principales* seront consignées dans Azure Application Insights et, si oui, comment. Une *réponse principale* est une réponse entrante dans le service Gestion des API Azure.                                                                                                                                                                       |

> [!NOTE]
> Il est possible de spécifier des enregistreurs d'événements à différents niveaux : pour une unique API ou pour toutes les API.
>  
> Si vous spécifiez les deux :
> + S’ils sont différents, les deux seront utilisés (journaux d’activité de multiplexage).
> + Si ce sont les mêmes avec des paramètres différents, celui de l’API unique (niveau de détail supérieur) aura la priorité sur celui de l’ensemble des API.

## <a name="what-data-is-added-to-azure-application-insights"></a>Données ajoutées à Azure Application Insights

Azure Application Insights reçoit :

+ un élément de télémétrie *Demande* pour chaque demande entrante ( *demande frontale* , *réponse frontale* ) ;
+ un élément de télémétrie *Dépendance* pour chaque demande transmise à un service principal ( *demande principale* , *réponse principale* ) ;
+ un élément de télémétrie *Exception* pour chaque demande ayant échoué.

Une demande ayant échoué peut être une demande qui :

+ a échoué en raison d’une interruption de la connexion cliente ;
+ a déclenché une section *on-error* des stratégies d’API ;
+ a un code de statut HTTP de réponse de type 4xx ou 5xx.

## <a name="performance-implications-and-log-sampling"></a>Conséquences sur les performances et échantillonnage des journaux

> [!WARNING]
> La journalisation de l’ensemble des événements peut avoir de grosses répercussions sur les performances en fonction du rythme des demandes entrantes.

Lors de tests de charge internes, cette fonctionnalité a provoqué une baisse du débit de 40 %-50 % lorsque la fréquence des demandes a dépassé 1 000 par seconde. Azure Application Insights est conçu pour utiliser l’analyse statistique dans le but d’évaluer les performances des applications. Il n’est ni destiné à servir de système d’audit ni adapté à la journalisation de la totalité des demandes dans le cas d’API générant de gros volumes.

Pour modifier le nombre de demandes consignées, réglez le paramètre **Échantillonnage** (voir les étapes ci-dessus). Une valeur de 100 % signifie que toutes les demandes sont consignées, tandis que le 0 % indique une absence totale de journalisation. **L’échantillonnage** permet de réduire le volume de télémétrie, ce qui se révèle efficace en prévention d’une dégradation significative des performances tout en conservant les avantages de la journalisation.

Il est également utile d’ignorer la journalisation des en-têtes et du corps des demandes et des réponses pour limiter les problèmes de performances.

## <a name="video"></a>Vidéo

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2pkXv]
>
>

## <a name="next-steps"></a>Étapes suivantes

+ Découvrez plus en détail [Azure Application Insights](/azure/application-insights/).
+ Étudiez la possibilité de [journalisation avec Azure Event Hubs](api-management-howto-log-event-hubs.md).
