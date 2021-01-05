---
title: Comment corriger manuellement les problèmes de synchronisation de ServiceNow
description: Réinitialiser la connexion à ServiceNow afin que les alertes dans Microsoft Azure puissent à nouveau appeler ServiceNow
ms.subservice: alerts
ms.topic: conceptual
author: nolavime
ms.author: nolavime
ms.date: 04/12/2020
ms.openlocfilehash: 3e836873219bde3836f2863e328b0b6f5b89addc
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507283"
---
# <a name="troubleshooting-problems-in-itsm-connector"></a>Résolution des problèmes liés au connecteur ITSM

Cet article décrit des problèmes courants dans le connecteur ITSM et la manière de les résoudre.

Azure Monitor vous avertit de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Elles permettent d’identifier et de résoudre les problèmes avant que les utilisateurs de votre système ne les remarquent. Pour plus d’informations sur les alertes, consultez Vue d’ensemble des alertes dans Microsoft Azure.
Le client peut choisir la manière dont il souhaite être informé de l’alerte, que ce soit par e-mail, SMS, webhook, ou d’automatiser une solution. Une autre option de notification consiste à utiliser ITSM.
ITSM vous donne la possibilité d’envoyer les alertes à un système de tickets externe tel que ServiceNow.

## <a name="visualize-and-analyze-the-incident-and-change-request-data"></a>Visualiser et analyser les données d’incident et de demande de modification

En fonction de la configuration choisie lors de l’établissement d’une connexion, ITSMC peut synchroniser jusqu’à 120 jours de données d’incidents et de demande de changement. Le schéma d’enregistrement de journal pour ces données est fourni dans la [section d’informations supplémentaires](https://docs.microsoft.com/azure/azure-monitor/platform/itsmc-overview#additional-information) de cet article.

Vous pouvez visualiser les données d’incident et de demande de changement à l’aide du tableau de bord ITSMC :

![Capture d’écran montrant le tableau de bord ITSMC.](media/itsmc-overview/itsmc-overview-sample-log-analytics.png)

Le tableau de bord fournit également des informations sur l’état du connecteur. Vous pouvez les utiliser comme point de départ pour analyser les problèmes liés aux connexions.

Vous pouvez également visualiser les incidents synchronisés avec les ordinateurs concernés dans Service Map.

La solution Carte de service détecte automatiquement les composants d’application sur les systèmes Windows et Linux et mappe la communication entre les services. Elle vous permet d’afficher les serveurs comme vous vous les représentez, c’est-à-dire comme des systèmes interconnectés qui fournissent des services critiques. Service Map affiche les connexions entre les serveurs, les processus et les ports sur n’importe quelle architecture connectée par TCP. Aucune configuration autre que l’installation d’un agent n’est exigée. Pour plus d’informations, consultez [Utilisation de Service Map](../insights/service-map.md).

Si vous utilisez Service Map, vous pouvez afficher les éléments de service d’assistance créés dans les solutions ITSM, comme indiqué ici :

![Capture d’écran montrant l’écran Log Analytics.](media/itsmc-overview/itsmc-overview-integrated-solutions.png)

## <a name="how-to-manually-fix-servicenow-sync-problems"></a>Comment corriger manuellement les problèmes de synchronisation de ServiceNow

Azure Monitor peut se connecter à des fournisseurs de gestion des services informatiques (ITSM) tiers. ServiceNow est un de ces fournisseurs.

Pour des raisons de sécurité, il peut être nécessaire d’actualiser le jeton d’authentification utilisé pour votre connexion à ServiceNow.
Utilisez le processus de synchronisation suivant pour réactiver la connexion et actualiser le jeton :


1. Recherchez la solution dans la bannière de recherche du haut, puis sélectionnez les solutions appropriées.

    ![Capture d’écran montrant la bannière de recherche supérieure et où sélectionner les solutions appropriées.](media/itsmc-resync-servicenow/solution-search-8bit.png)

1. Dans l’écran des solutions, choisissez « Sélectionner tout » dans le filtre d’abonnement, puis filtrez par « ServiceDesk ».

    ![Capture d’écran montrant où choisir Sélectionner tout et où filtrer par ServiceDesk.](media/itsmc-resync-servicenow/solutions-list-8bit.png)

1. Sélectionnez la solution de votre connexion ITSM.
1. Sélectionnez la connexion ITSM dans la bannière gauche.

    ![Capture d’écran montrant où sélectionner Connexions ITSM.](media/itsmc-resync-servicenow/itsm-connector-8bit.png)

1. Sélectionnez chaque connecteur dans la liste. 
    1. Cliquez sur le nom du connecteur pour le configurer
    1. Supprimez tous les connecteurs qui ne sont plus utilisés

    1. Mettez à jour les champs en fonction de [ces définitions](./itsmc-connections.md) selon votre type de partenaire

    1. Cliquer sur Synchroniser.

       ![Capture d’écran mettant en évidence le bouton Synchroniser.](media/itsmc-resync-servicenow/resync-8bit2.png)

    1. Cliquez sur Enregistrer.

        ![Nouvelle connexion](media/itsmc-resync-servicenow/save-8bit.png)

f.    Passez en revue les notifications pour voir si le processus s’est terminé avec succès.

## <a name="troubleshoot-itsm-connections"></a>Dépanner les connexions ITSM

- Si une connexion échoue à partir de l’interface utilisateur de la source connectée et que le message **Erreur lors de l’enregistrement de la connexion** s’affiche, effectuez les étapes suivantes :
   - Pour les connexions ServiceNow, Cherwell et Provance :  
     - Vérifiez que vous avez correctement entré le nom d’utilisateur, le mot de passe, l’ID client et le secret client pour chacune des connexions.  
     - Veillez à disposer de privilèges suffisants dans le produit ITSM correspondant afin d’établir la connexion.  
   - Pour les connexions Service Manager :  
     - Vérifiez que l’application web est correctement déployée et que la connexion hybride est créée. Pour vérifier que la connexion est établie avec l’ordinateur Service Manager local, accédez à l’URL de l’application web, comme décrit dans la documentation concernant l’établissement d’une [connexion hybride](./itsmc-connections.md#configure-the-hybrid-connection).  

- Si les données de ServiceNow ne sont pas synchronisées dans Log Analytics, vérifiez que l’instance ServiceNow n’est pas en état de veille. Parfois, les instances de développement ServiceNow entrent en veille quand elles restent longtemps inactives. Si ce n’est pas ce qui se passe, signalez le problème.
- Si des alertes Log Analytics se déclenchent mais qu’aucun élément de travail n’est créé dans le produit ITSM, si aucun élément de configuration n’est créé/lié à des éléments de travail ou pour obtenir d’autres informations, consultez ces ressources :
   -  ITSMC : la solution montre un récapitulatif des connexions, éléments de travail, ordinateurs, etc. Sélectionnez la vignette qui a l’étiquette **État du connecteur**. Cela vous permet d’accéder à **Recherche dans les journaux** avec la requête appropriée. Pour plus d’informations, examinez les enregistrements de journal dont `LogType_S` a la valeur `ERROR`.
   - Page **Recherche dans les journaux** : Consultez les erreurs et les informations associées directement à l’aide de la requête `*ServiceDeskLog_CL*`.

## <a name="troubleshoot-service-manager-web-app-deployment"></a>Résoudre les problèmes de déploiement de l’application web Service Manager

-   Si vous rencontrez des problèmes lors du déploiement d’application web, vérifiez que vous disposez des autorisations nécessaires pour créer/déployer des ressources dans l’abonnement.
-   Si l’erreur **Référence d’objet non définie sur une instance d’un objet** s’affiche pendant l’exécution du [script](itsmc-service-manager-script.md), vérifiez que vous avez entré des valeurs valides dans la section **Configuration utilisateur**.
-   Si vous ne parvenez pas à créer l’espace de noms de Service Bus Relay, vérifiez que le fournisseur de ressources nécessaire est inscrit dans l’abonnement. S’il n’est pas inscrit, créez manuellement l’espace de noms Service Bus Relay à partir du portail Azure. Vous pouvez également le créer quand vous [créez la connexion hybride](./itsmc-connections.md#configure-the-hybrid-connection) dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Découvrir plus d’informations sur les [Connexions de gestion des services informatiques](itsmc-connections.md)
