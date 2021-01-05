---
title: Montée en charge d’Azure Analysis Services | Microsoft Docs
description: Répliquer les serveurs Azure Analysis Services avec Scale-out. Les requêtes client peuvent ensuite être distribuées entre plusieurs réplicas de requête dans un pool de requêtes Scale-out.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 24ee31b941d836d296c30927cfb9636f3023fa89
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019429"
---
# <a name="azure-analysis-services-scale-out"></a>Montée en charge d’Azure Analysis Services

La montée en charge permet de distribuer les requêtes des clients sur plusieurs *réplicas de requête* dans un *pool de requêtes*, ce qui diminue les temps de réponse quand les charges de travail de requêtes sont importantes. Vous pouvez aussi séparer le traitement du pool de requêtes, ce qui garantit que les requêtes des clients ne sont pas affectées par les opérations de traitement. La montée en charge peut être configurée dans le portail Azure ou avec l’API REST d’Analysis Services.

La montée en charge est disponible pour les serveurs du niveau tarifaire Standard. Chaque réplica de requête est facturé au même prix que votre serveur. Tous les réplicas de la requête sont créés dans la même région que votre serveur. Le nombre de réplicas de requête que vous pouvez configurer est limité par la région de votre serveur. Pour plus d’informations, consultez [Disponibilité par région](analysis-services-overview.md#availability-by-region). La montée en charge n’augmente pas la quantité de mémoire disponible pour votre serveur. Pour augmenter la mémoire, vous devez passer à un forfait supérieur. 

## <a name="why-scale-out"></a>Pourquoi effectuer un scale-out ?

Dans un déploiement de serveur classique, un même serveur est utilisé comme serveur de traitement et comme serveur de requêtes. Si le nombre de requêtes de clients sur des modèles présents sur votre serveur dépasse les unités de traitement des requêtes du plan de votre serveur, ou que le traitement du modèle est effectué en même temps que des charges de travail de requêtes importantes, les performances peuvent diminuer. 

La montée en charge permet de créer un pool de requêtes comportant jusqu’à sept ressources de réplicas de requête supplémentaires (huit au total en comptant le serveur *principal*). Il est possible d’augmenter le nombre de réplicas du pool de requêtes pour répondre aux demandes de QPU aux moments critiques, et de séparer à tout moment un serveur de traitement du pool de requêtes. 

Quel que soit le nombre de réplicas de requête dont vous disposez dans un pool de requêtes, le traitement des charges de travail n’est pas distribué entre les réplicas de requête. Le serveur principal sert de serveur de traitement. Les réplicas de requête ne traitent que les requêtes portant sur les bases de données model synchronisées entre le serveur principal et chacun des réplicas du pool de requêtes. 

Lors de la montée en charge, l’ajout incrémentiel des nouveaux réplicas de requête au pool de requêtes peut prendre jusqu’à cinq minutes. Dès qu’ils sont tous opérationnels, la charge des nouvelles connexions client est équilibrée entre les ressources du pool de requêtes. Les connexions client existantes ne sont pas modifiées à partir de la ressource à laquelle ils sont actuellement connectés. Lors d’une mise à l’échelle à la baisse, toutes les connexions client existantes à une ressource de pool de requêtes en cours de suppression du pool de requêtes sont arrêtées. Les clients peuvent se reconnecter à une ressource restante du pool de requêtes.

## <a name="how-it-works"></a>Fonctionnement

Lors de la première configuration de la montée en charge, les bases de données model du serveur principal sont *automatiquement* synchronisées avec les nouveaux réplicas d’un nouveau pool de requêtes. La synchronisation automatique ne se produit qu’une seule fois. À cette occasion, les fichiers de données du serveur principal (chiffrées au repos dans le Stockage Blob) sont copiés à un second emplacement, également chiffré au repos dans le Stockage Blob. Les réplicas du pool de requêtes sont ensuite *alimentés* avec des données issues du deuxième ensemble de fichiers. 

Si la synchronisation automatique ne se produit que lors de la première montée en charge du serveur, une synchronisation manuelle reste possible pour que les données des réplicas du pool de requêtes correspondent à celles du serveur principal. En cas de traitement (actualisation) des modèles sur le serveur principal, il est nécessaire d’effectuer une synchronisation *une fois* les opérations de traitement terminées. Elle copie dans le deuxième ensemble de fichiers les données mises à jour des fichiers du serveur principal dans le Stockage Blob. Les réplicas du pool de requêtes sont ensuite alimentés avec les données mises à jour issues du deuxième ensemble de fichiers dans le Stockage Blob. 

Lors des opérations ultérieures de montée en charge (par exemple, augmentation de deux à cinq du nombre de réplicas dans le pool de requêtes), les nouveaux réplicas sont alimentés avec des données issues du deuxième ensemble de fichiers dans le Stockage Blob. Aucune synchronisation n’est effectuée. Si vous effectuez ensuite une synchronisation après la montée en charge, les nouveaux réplicas dans le pool de requêtes seront alimentés deux fois, ce qui serait redondant. Lors des opérations ultérieures de montée en charge, il est important de garder à l’esprit plusieurs considérations :

* Effectuez une synchronisation *avant l’opération de montée en charge* afin d’éviter une alimentation redondante des réplicas ajoutés. Les opérations simultanées de montée en charge et de synchronisation ne sont pas autorisées.

* En cas d’automatisation d’opérations de traitement *et* de montée en charge, il est important de commencer par traiter les données sur le serveur principal, puis d’effectuer une synchronisation et enfin de passer à l’opération de montée en charge. Cette séquence réduit l’impact sur les ressources QPU et de mémoire.

* Pendant les opérations de Scale-out, tous les serveurs du pool de requêtes, y compris le serveur principal, sont temporairement hors connexion.

* La synchronisation reste autorisée même en l’absence de réplicas dans le pool de requêtes. Si vous passez de zéro à un ou plusieurs réplicas avec de nouvelles données issues d’une opération de traitement sur le serveur principal, effectuez tout d’abord la synchronisation sans aucun réplica dans le pool de requêtes, puis passez à la montée en charge. L’ordre synchronisation-montée en charge évite une alimentation redondante des réplicas ajoutés.

* La suppression d’une base de données model du serveur principal n’a pas pour effet de la supprimer automatiquement des réplicas du pool de requêtes. Il est nécessaire d’effectuer une opération de synchronisation avec la commande PowerShell [Sync-AzAnalysisServicesInstance](/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance), qui supprime de l’emplacement de Stockage Blob partagé du réplica le ou les fichiers de cette base de données, puis supprime la base de données model sur les réplicas du pool de requêtes. Pour déterminer si une base de données model existe sur les réplicas du pool de requêtes, mais pas sur le serveur principal, définissez le paramètre **Séparer le serveur de traitement du pool de requêtes** sur **Oui**. Ensuite, utilisez SSMS pour vous connecter au serveur principal avec le qualificateur `:rw` afin de voir si la base de données existe déjà. Connectez-vous aux réplicas du pool de requêtes sans le qualificateur `:rw` pour savoir si la même base de données s’y trouve également. Si la base de données existe sur les réplicas du pool de requêtes, mais pas sur le serveur principal, exécutez une opération de synchronisation.   

* En cas de renommage d’une base de données sur le serveur principal, une étape supplémentaire est nécessaire pour que la base de données soit correctement synchronisée avec tous les réplicas. Après le renommage, effectuez une synchronisation avec la commande [Sync-AzAnalysisServicesInstance](/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance) en spécifiant le paramètre `-Database` avec l’ancien nom de la base de données. Cette synchronisation supprime de tous les réplicas la base de données et les fichiers portant l’ancien nom. Ensuite, effectuez une autre synchronisation en indiquant le paramètre `-Database` avec le nouveau nom de la base de données. La deuxième synchronisation copie la base de données portant le nouveau nom dans le deuxième ensemble de fichiers et alimente tous les réplicas. Il n’est pas possible d’effectuer ces synchronisations avec la commande Synchroniser le modèle du portail.

### <a name="synchronization-mode"></a>Mode de synchronisation

Par défaut, les réplicas de requête sont réalimentés intégralement, et non pas de façon incrémentielle. La réalimentation se produit par étapes. Ils sont détachés et attachés deux à la fois (en supposant qu’il existe au moins trois réplicas) pour garantir qu’au moins un réplica est conservé en ligne pour les requêtes à tout moment. Dans certains cas, les clients peuvent avoir besoin de se reconnecter à l’un des réplicas en ligne pendant que ce processus a lieu. En utilisant le paramètre **ReplicaSyncMode**, vous pouvez désormais spécifier la synchronisation du réplica de requête en parallèle. La synchronisation parallèle offre les avantages suivants : 

- Réduction significative du temps de synchronisation. 
- Les données des réplicas sont plus susceptibles d’être cohérentes au cours du processus de synchronisation. 
- Étant donné que les bases de données sont conservées en ligne sur tous les réplicas tout au long du processus de synchronisation, les clients n’ont pas besoin de se reconnecter. 
- Le cache en mémoire est mis à jour de façon incrémentielle avec uniquement les données modifiées, ce qui peut être plus rapide que la réalimentation complète du modèle. 

#### <a name="setting-replicasyncmode"></a>Paramétrage de ReplicaSyncMode

Utilisez SSMS pour définir ReplicaSyncMode dans les Propriétés avancées. Les valeurs possibles sont les suivantes : 

- `1` (par défaut) : Réalimentation complète de la base de données des réplicas par étapes (incrémentielle). 
- `2`: Synchronisation optimisée en parallèle. 

![Paramètre RelicaSyncMode](media/analysis-services-scale-out/aas-scale-out-sync-mode.png)

Lorsque vous définissez **ReplicaSyncMode = 2**, en fonction de la quantité de cache qui doit être mise à jour, la mémoire supplémentaire peut être utilisée par les réplicas de requête. Pour que la base de données reste en ligne et disponible pour les requêtes, en fonction de la quantité de données modifiée, l’opération peut nécessiter jusqu’à *doubler la mémoire* sur le réplica, car les segments anciens et nouveaux sont conservés simultanément en mémoire. Les nœuds de réplica ont la même allocation de mémoire que le nœud principal, et il y a généralement de la mémoire supplémentaire sur le nœud principal pour les opérations d’actualisation. Il est donc peu probable que les réplicas manquent de mémoire. En outre, le scénario commun est que la base de données est mise à jour de façon incrémentielle sur le nœud principal, et donc la nécessité de doubler la mémoire devrait être peu commune. Si l’opération de synchronisation rencontre une erreur de mémoire insuffisante, elle réessaie à l’aide de la technique par défaut (attacher/détacher deux à la fois). 

### <a name="separate-processing-from-query-pool"></a>Traitement distinct à partir du pool de requêtes

Pour optimiser les performances des opérations de traitement et de requête, vous pouvez choisir de séparer votre serveur de traitement du pool de requêtes. Lorsqu’ils sont séparés, de nouvelles connexions client sont attribuées aux réplicas de requête dans le pool de requêtes uniquement. Si les opérations de traitement ne prennent qu’un court laps de temps, vous pouvez choisir de séparer votre serveur de traitement du pool de requêtes uniquement pendant le temps nécessaire à la réalisation des opérations de traitement et de synchronisation, puis de le réinclure dans le pool de requêtes. Les opérations de séparation du serveur de traitement du pool de requêtes et de rajout dans le pool de requêtes peuvent prendre jusqu'à cinq minutes.

## <a name="monitor-qpu-usage"></a>Surveiller l’utilisation des unités de traitement des requêtes

Afin de déterminer si un scale-out est nécessaire pour votre serveur, [supervisez votre serveur](analysis-services-monitor.md) dans le portail Azure en utilisant des métriques. Si vos unités de traitement des requêtes atteignent régulièrement le maximum, cela signifie que le nombre de requêtes sur vos modèles dépasse la limite d’unités de traitement des requêtes pour votre forfait. La métrique Longueur de la file d’attente des travaux du pool de requêtes augmente aussi quand le nombre de requêtes dans la file d’attente du pool de threads de requêtes dépasse les unités de traitement des requêtes disponibles. 

L’indicateur de QPU moyen par ServerResourceType est également intéressant. Cette mesure compare le nombre moyen de QPU pour le serveur principal et le pool de requêtes. 

![Indicateurs de scale-out des requêtes](media/analysis-services-scale-out/aas-scale-out-monitor.png)

**Pour configurer les QPU (unités de traitement des requêtes) par ServerResourceType**

1. Dans un graphique en courbes Mesures, cliquez sur **Ajouter une mesure**. 
2. Dans **RESSOURCE**, sélectionnez votre serveur, puis, dans **ESPACE DE NOMS DE LA MESURE**, sélectionnez **Mesures standard Analysis Services** ; sélectionnez **QPU** dans **MESURE**, puis **Moyenne** dans **AGRÉGATION**. 
3. Cliquez sur **Appliquer le fractionnement**. 
4. Dans **VALEURS**, sélectionnez **ServerResourceType**.  

### <a name="detailed-diagnostic-logging"></a>Journalisation des diagnostics détaillés

Utilisez les journaux Azure Monitor pour obtenir des diagnostics plus détaillés des ressources serveur ayant fait l’objet d’un scale-out. Avec les journaux, vous pouvez utiliser des requêtes Log Analytics pour répartir les QPU et la mémoire par serveur et par réplica. Pour plus d’informations, consultez des exemples de requêtes dans [Journalisation des diagnostics Analysis Services](analysis-services-logging.md#example-queries).


## <a name="configure-scale-out"></a>Configurer la montée en charge

### <a name="in-azure-portal"></a>Dans le portail Azure

1. Dans le portail, cliquez sur **Montée en charge**. Utilisez le curseur pour sélectionner le nombre de serveurs réplica de requête. Le nombre de réplicas que vous choisissez s’ajoute à votre serveur existant.  

2. Dans **Séparer le serveur de traitement du pool de requêtes**, sélectionnez Oui pour exclure votre serveur de traitement des serveurs de requêtes. Les [connexions](#connections) client utilisant la chaîne de connexion par défaut (sans `:rw`) sont redirigées vers les réplicas du pool de requêtes. 

   ![Curseur de montée en charge](media/analysis-services-scale-out/aas-scale-out-slider.png)

3. Cliquez sur **Enregistrer** pour provisionner vos nouveaux serveurs réplica de requête. 

Lors de la première configuration de la montée en charge d’un serveur, les bases de données model du serveur principal sont automatiquement synchronisées avec les réplicas du pool de requêtes. La synchronisation automatique ne se produit qu’une fois, lors de la première configuration de la montée en charge pour passer à un ou plusieurs réplicas. Les modifications apportées par la suite au nombre de réplicas sur le même serveur *ne déclenchent pas d’autre synchronisation automatique*, même en redéfinissant le serveur sur zéro réplica pour ensuite le repasser à un nombre supérieur de réplicas. 

## <a name="synchronize"></a>Synchroniser 

Les opérations de synchronisation doivent être effectuées manuellement ou avec l’API REST.

### <a name="in-azure-portal"></a>Dans le portail Azure

Dans **Vue d’ensemble** > Modèle > **Synchroniser le modèle**.

![Icône Synchroniser](media/analysis-services-scale-out/aas-scale-out-sync.png)

### <a name="rest-api"></a>API REST

Utilisez l’opération de **synchronisation**.

#### <a name="synchronize-a-model"></a>Synchroniser un modèle   

`POST https://<region>.asazure.windows.net/servers/<servername>:rw/models/<modelname>/sync`

#### <a name="get-sync-status"></a>Obtenir l’état de la synchronisation  

`GET https://<region>.asazure.windows.net/servers/<servername>/models/<modelname>/sync`

Codes de statut de retour :


|Code  |Description  |
|---------|---------|
|-1     |  Non valide       |
|0     | En cours de réplication        |
|1     |  En cours de réalimentation       |
|2     |   Completed       |
|3     |   Échec      |
|4     |    En cours de finalisation     |
|||


### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Avant d’utiliser PowerShell, [installez ou mettez à jour le dernier module Azure PowerShell](/powershell/azure/install-az-ps). 

Pour exécuter la synchronisation, utilisez [Sync-AzAnalysisServicesInstance](/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance).

Pour définir le nombre de réplicas de requête, utilisez [Set-AzAnalysisServicesServer](/powershell/module/az.analysisservices/set-azanalysisservicesserver). Spécifiez le paramètre `-ReadonlyReplicaCount` facultatif.

Pour séparer le serveur de traitement du pool de requêtes, utilisez [Set-AzAnalysisServicesServer](/powershell/module/az.analysisservices/set-azanalysisservicesserver). Spécifiez le paramètre `-DefaultConnectionMode` facultatif pour utiliser `Readonly`.

Pour plus d’informations, voir [Utiliser un principal de service avec le module Az.AnalysisServices](analysis-services-service-principal.md#azmodule).

## <a name="connections"></a>Connexions

Dans la page Vue d’ensemble de votre serveur, vous voyez deux noms de serveur. Si vous n’avez pas encore configuré la montée en charge pour un serveur, les deux noms de serveur fonctionnent de la même façon. Une fois le scale-out configuré pour un serveur, vous devez spécifier le nom du serveur approprié en fonction du type de connexion. 

Pour les connexions de clients d’utilisateur finaux, comme Power BI Desktop, Excel et des applications personnalisées, utilisez **Nom du serveur**. 

Pour SSMS, Visual Studio et les chaînes de connexion dans PowerShell, les applications de fonction Azure et AMO, utilisez **Nom du serveur d’administration**. Le nom du serveur d’administration inclut un qualificateur `:rw` (lecture-écriture) spécial. Toutes les opérations de traitement se produisent sur le serveur d’administration (principal).

![Noms de serveur](media/analysis-services-scale-out/aas-scale-out-name.png)

## <a name="scale-up-scale-down-vs-scale-out"></a>Montée en puissance, descente en puissance vs. Montée en charge

Vous pouvez modifier le niveau tarifaire sur un serveur avec plusieurs réplicas. Le même niveau tarifaire s’applique à tous les réplicas. Une opération de mise à l’échelle entraînera d’abord la mise à niveau de tous les réplicas à la fois, puis affichera tous les réplicas sur le nouveau niveau tarifaire.

## <a name="troubleshoot"></a>Dépanner

**Problème :** Les utilisateurs obtiennent l’erreur **Instance « \<Name of the server> » du serveur introuvable en mode « Lecture seule ».**

**Solution :** si l’option **Séparer le serveur de traitement du pool de requêtes** est sélectionnée, les connexions client utilisant la chaîne de connexion par défaut (sans `:rw`) sont redirigées vers les réplicas du pool de requêtes. Si les réplicas du pool de requêtes ne sont pas encore en ligne, car la synchronisation n’est pas terminée, les connexions client redirigées peuvent échouer. Pour empêcher l’échec des connexions, vous devez avoir au moins deux serveurs dans le pool de requêtes quand vous effectuez une synchronisation. Chaque serveur est synchronisé individuellement pendant que les autres serveurs restent en ligne. Si vous choisissez de ne pas mettre le serveur de traitement dans le pool de requêtes durant le traitement, vous pouvez l’enlever du pool avant le traitement, puis le réintégrer au pool une fois le traitement terminé, mais avant la synchronisation. Utilisez les métriques Mémoire et QPU pour superviser l’état de synchronisation.



## <a name="related-information"></a>Informations connexes

[Surveiller les métriques du serveur](analysis-services-monitor.md)   
[Gérer Azure Analysis Services](analysis-services-manage.md)