---
title: Diagnostiquer et résoudre les problèmes – Azure Time Series Insights
description: Cet article explique comment diagnostiquer, dépanner et résoudre les problèmes courants dans votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 15f2ff5aaa1d731c13125d0a3ab4ac32acb9276c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023273"
---
# <a name="diagnose-and-solve-issues-in-your-azure-time-series-insights-gen1-environment"></a>Diagnostiquer et résoudre les problèmes dans votre environnement Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article décrit les problèmes que vous pouvez rencontrer dans votre environnement Azure Time Series Insights. Il présente les causes éventuelles et les solutions pour les résoudre.

## <a name="video"></a>Vidéo

### <a name="learn-about-common-azure-time-series-insights-challenges-and-mitigationsbr"></a>En savoir plus sur les problèmes et les atténuations d’Azure Time Series Insights</br>

> [!VIDEO https://www.youtube.com/embed/7U0SwxAVSKw]

## <a name="problem-no-data-is-shown"></a>Problème : Aucune donnée n’est affichée

Si aucune donnée n’apparaît dans [l’Explorateur Azure Time Series Insights](https://insights.timeseries.azure.com), examinez ces causes courantes.

### <a name="cause-a-event-source-data-isnt-in-json-format"></a>Cause A : Les données sources de l’événement ne sont pas au format JSON

Azure Time Series Insights ne prend en charge que les données JSON. Pour obtenir des exemples de données JSON, consultez [Structures JSON prises en charge](./concepts-json-flattening-escaping-rules.md).

### <a name="cause-b-the-event-source-key-is-missing-a-required-permission"></a>Cause B : Il manque une autorisation requise pour la clé de source de l’événement

* Pour un hub IoT dans Azure IoT Hub, vous devez fournir la clé qui dispose des autorisations de connexion de service. Sélectionnez la stratégie **iothubowner** ou **service**. Les deux disposent d’autorisations de connexion de service.

   [![Autorisations de connexion de service IoT Hub](media/diagnose-and-solve-problems/iothub-serviceconnect-permissions.png)](media/diagnose-and-solve-problems/iothub-serviceconnect-permissions.png#lightbox)

* Pour un Event Hub dans Azure Event Hubs, vous devez fournir la clé disposant des autorisations d’écoute. Les stratégies **read** et **manage** fonctionnent toutes les deux, car elles disposent toutes deux d’autorisations d’écoute.

   [![Autorisations d’écoute Event Hub](media/diagnose-and-solve-problems/eventhub-listen-permissions.png)](media/diagnose-and-solve-problems/eventhub-listen-permissions.png#lightbox)

### <a name="cause-c-the-provided-consumer-group-isnt-exclusive-to-azure-time-series-insights"></a>Cause C : Le groupe de consommateurs fourni n’est pas exclusif à Azure Time Series Insights

Lorsque vous inscrivez un hub IoT ou un Event Hub, il est important de définir le groupe de consommateurs que vous souhaitez utiliser pour lire les données. Ce groupe de consommateurs *ne peut pas être partagé*. Si le groupe de consommateurs est partagé, le hub IoT ou l’Event Hub sous-jacent déconnecte de manière automatique et aléatoire l’un des lecteurs. Fournissez un groupe de consommateurs unique auprès duquel Azure Time Series Insights lira les informations.

### <a name="cause-d-the-environment-has-just-been-provisioned"></a>Cause D : L’environnement vient d’être approvisionné

Les données s’afficheront dans votre explorateur Azure Time Series Insights quelques minutes après la création initiale de l’environnement et de ses données.

## <a name="problem-some-data-is-shown-but-data-is-missing"></a>Problème : Certaines données sont affichées, mais d’autres manquent

Lorsque les données n’apparaissent que partiellement et qu’elles semblent avoir du retard, examinez ces possibles problèmes.

### <a name="cause-a-your-environment-is-being-throttled"></a>Cause A : Votre environnement est sujet à des limitations

La [limitation](time-series-insights-environment-mitigate-latency.md) est un problème courant lors du provisionnement des environnements après la création d’une source d’événement qui comporte des données. Azure IoT Hub et Azure Events Hubs stockent les données jusqu’à sept jours. Azure Time Series Insights commence toujours par l’événement le plus ancien dans la source de l’événement (premier entré, premier sorti ou *FIFO*).

Par exemple, si 5 millions d’événements se trouvent dans une source de l’événement lorsque vous vous connectez à un S1 (environnement Azure Time Series Insights à une unité), Azure Time Series Insights lit environ 1 million d’événements par jour. Azure Time Series Insights peut donc subir cinq jours de latence. En réalité, l’environnement est limité.

Si votre source de l’événement comporte des événements antérieurs, vous pouvez aborder la limitation de l’une des deux façons suivantes :

* Modifiez les limites de rétention de votre source de l’événement pour vous débarrasser des anciens événements que vous ne souhaitez pas voir apparaître dans Azure Time Series Insights.
* Configurez une plus grande taille d’environnement (nombre d’unités) pour augmenter le débit des anciens événements. Dans l’exemple précédent, si vous augmentez l’environnement S1 à cinq unités pendant un jour, l’environnement doit rattraper le retard en un seul jour. Si votre production d’événement stable compte un million d’événements par jour maximum, vous pouvez réduire la capacité d’événements à une unité après qu’Azure Time Series Insights ait rattrapé son retard.

Les limitations sont appliquées en fonction de la capacité et du type de référence SKU de l’environnement. Cette capacité est répartie entre les différentes sources d’événements de l’environnement. Si la source d’événements pour votre hub IoT/Event Hub envoie des données au-delà des limites définies, vous allez constater un ralentissement et un décalage.

La figure suivante montre un environnement Azure Time Series Insights ayant une référence SKU S1 et une capacité de 3 unités. Cet environnement peut recevoir 3 millions d’événements par jour.

[![Environment capacity](media/diagnose-and-solve-problems/environment-sku-current-capacity.png)](media/diagnose-and-solve-problems/environment-sku-current-capacity.png#lightbox)

Supposons qu’un environnement ingère les messages à partir d’un Event Hub. Il a un taux d’entrée quotidien d’environ 67 000 messages. Cela représente environ 46 messages par minute.

* Si chaque message de l’Event Hub est aplati dans un seul événement Azure Time Series Insights, aucune limitation ne se produit.
* Si chaque message de l’Event Hub est aplati dans 100 événements Azure Time Series Insights, 4 600 événements devraient être ingérés toutes les minutes.

Un environnement de référence SKU S1 qui a une capacité de 3 unités peut seulement ingérer 2 100 événements toutes les minutes (1 million d’événements par jour = 700 événements par minute à 3 unités = 2 100 événements par minute).

Pour en savoir plus sur le fonctionnement de la logique de mise à plat, consultez [Structures JSON prises en charge](./concepts-json-flattening-escaping-rules.md).

#### <a name="recommended-resolutions-for-excessive-throttling"></a>Résolutions recommandées en cas de limitation excessive

Pour éviter tout décalage, augmentez la capacité de votre environnement. Pour plus d’informations, consultez [Mettre à l’échelle votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).

### <a name="cause-b-initial-ingestion-of-historical-data-slows-ingress"></a>Cause B : L’ingestion initiale de données historiques ralentit l’entrée

Si vous vous connectez à une source de l’événement existante, il est probable que votre hub IoT ou votre Event Hub comporte déjà des données. L’environnement démarre l’extraction des données depuis le début de la période de rétention des messages de la source d’événement. Ce traitement par défaut ne peut pas être remplacé. Vous pouvez appliquer des limitations. La limitation peut prendre un certain temps en ce qui concerne le rattrapage, car elle ingère les données d’historique.

#### <a name="recommended-resolutions-for-large-initial-ingestion"></a>Résolutions recommandées en cas d’ingestion initiale volumineuse

Pour corriger le décalage :

1. Définissez la capacité de référence SKU sur la valeur maximale autorisée (10 unités dans ce cas). Après augmentation de la capacité, le processus d’entrée commence à rattraper le retard beaucoup plus rapidement. L’augmentation de capacité occasionne des frais supplémentaires. Pour suivre sa progression rapide, consultez le graphique de disponibilité dans l’[explorateur Azure Time Series Insights](https://insights.timeseries.azure.com).

2. Une fois le retard rattrapé, rétablissez la capacité de référence SKU sur votre taux d’entrée normal.

## <a name="problem-data-was-showing-previously-but-is-no-longer-showing"></a>Problème : Les données étaient affichées précédemment, mais ne sont plus affichées

Si Azure Time Series Insights n’ingère plus les données, mais que les événements sont toujours diffusés en continu dans IoT Hub ou Event Hub, considérez cette cause potentielle.

### <a name="cause-a-your-hub-access-key-was-regenerated-and-your-environment-needs-to-be-updated"></a>Cause A : La clé d’accès de votre hub a été régénérée et votre environnement doit être mis à jour

Ce problème se produit lorsque la clé fournie lors de la création de votre source d’événements n’est plus valide. Vous pouvez voir les données de télémétrie dans votre hub, mais aucun message n’a été reçu en entrée dans Azure Time Series Insights. Si vous ne savez pas si la clé a été régénérée, vous pouvez rechercher dans le journal d’activité de votre Event Hub « Création ou mise à jour de règles d’autorisation d’espace de noms ». Pour IoT Hub, recherchez « Création ou mise à jour d’une ressource IoT Hub ».

Pour mettre à jour votre environnement Azure Time Series Insights avec la nouvelle clé, ouvrez votre ressource de hub dans le Portail Azure et copiez la nouvelle clé. Accédez à votre ressource Azure Time Series Insights et sélectionnez **Sources d’événements** :

   [![Select Event Sources](media/diagnose-and-solve-problems/update-hub-key-step-1.png)](media/diagnose-and-solve-problems/update-hub-key-step-1.png#lightbox)

Sélectionnez la ou les sources d’événements à partir desquelles l’ingestion s’est arrêtée, collez la nouvelle clé, puis sélectionnez **Enregistrer** :

   [![Paste in the new key](media/diagnose-and-solve-problems/update-hub-key-step-2.png)](media/diagnose-and-solve-problems/update-hub-key-step-2.png#lightbox)

## <a name="problem-the-event-sources-timestamp-property-name-setting-doesnt-work"></a>Problème : Le paramètre Nom de la propriété timestamp de la source d’événement ne fonctionne pas

Vérifiez que la valeur de la propriété timestamp provenant de votre source d’événement, comme une chaîne JSON, est au format _aaaa-MM-jjTHH:mm:ss.FFFFFFFK_. Voici un exemple : **2008-04-12T12:53Z**.

N’oubliez pas que le nom de la propriété timestamp est sensible à la casse.

La méthode la plus simple pour vous assurer que votre nom de propriété timestamp est capturé et fonctionne correctement consiste à utiliser l’explorateur Azure Time Series Insights. Dans l’explorateur Azure Time Series Insights, à l’aide du graphique, sélectionnez une période de temps spécifique après avoir indiqué le nom de la propriété timestamp. Cliquez avec le bouton droit sur la sélection, puis sélectionnez **Explorer les événements**.

Le premier en-tête de colonne doit correspondre au nom de votre propriété timestamp. À côté du mot **Horodatage** est affiché **($ts)** .

Les valeurs suivantes ne seront pas affichées :

* *(abc)* : indique qu’Azure Time Series Insights lit les valeurs de données sous forme de chaînes.
* *Icône de calendrier* : indique qu’Azure Time Series Insights lit les valeurs de données en tant que valeurs DateHeure.
* *#*  : indique qu’Azure Time Series Insights lit les valeurs de données sous forme d’entiers.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [la façon de limiter la latence dans Azure Time Series Insights](time-series-insights-environment-mitigate-latency.md).

* Apprenez [comment mettre à l’échelle votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).