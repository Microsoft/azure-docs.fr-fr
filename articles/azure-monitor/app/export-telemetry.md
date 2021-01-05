---
title: Exportation continue des données de télémétrie d’Application Insights | Microsoft Docs
description: Exportez les données de diagnostic et les données d’utilisation dans le stockage Microsoft Azure et téléchargez-les à partir de là.
ms.topic: conceptual
ms.date: 05/26/2020
ms.openlocfilehash: f67a5c555c438298cee701ca065aaf8c01c6406e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87324333"
---
# <a name="export-telemetry-from-application-insights"></a>Exporter la télémétrie depuis Application Insights
Vous souhaitez conserver votre télémétrie plus longtemps que la période de rétention standard ? Ou la traiter d’une façon spécialisée ? L’exportation continue est idéale dans ce cas. Les événements que vous voyez dans le portail Application Insights peuvent être exportés vers le stockage Microsoft Azure au format JSON. À partir de là, vous pouvez télécharger vos données et écrire le code dont vous avez besoin pour les traiter.  

> [!NOTE]
> L’exportation continue est uniquement prise en charge pour les ressources Application Insights classiques. [Les ressources Application Insights basées sur un espace de travail](./create-workspace-resource.md) doivent utiliser des [paramètres de diagnostic](./create-workspace-resource.md#export-telemetry).
>

Avant de configurer l’exportation continue, d’autres options doivent être prises en considération :

* Le bouton Exporter en haut d’un onglet de métriques ou de recherche permet de transférer des tables et des graphiques dans une feuille de calcul Excel.

* [Analytics](../log-query/log-query-overview.md) fournit un puissant langage de requête pour la télémétrie et peut également en exporter les résultats.
* Si vous cherchez à [explorer vos données dans Power BI](./export-power-bi.md), vous pouvez le faire sans utiliser l’exportation continue.
* [L’API REST d’accès aux données](https://dev.applicationinsights.io/) vous permet d’accéder à vos données de télémétrie par programme.
* Vous pouvez également configurer [l’exportation continue par le biais de Powershell](/powershell/module/az.applicationinsights/new-azapplicationinsightscontinuousexport).

Une fois que l’exportation continue a copié vos données vers l’espace de stockage (où elles peuvent rester aussi longtemps que vous le souhaitez), elles restent disponibles dans Application Insights pendant la [période de rétention](./data-retention-privacy.md) habituelle.

## <a name="continuous-export-advanced-storage-configuration"></a>Configuration de stockage avancée de l’exportation continue

L’exportation continue **ne prend pas en charge** les fonctionnalités/configurations de stockage Azure suivantes :

* [Pare-feu de réseau virtuel/Stockage Azure](../../storage/common/storage-network-security.md) utilisés conjointement avec le Stockage Blob Azure.

* [Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-introduction.md).

## <a name="create-a-continuous-export"></a><a name="setup"></a> Créez une exportation continue.

1. Dans la ressource Application Insights de votre application, sous Configurer, à gauche, ouvrez Exportation continue et choisissez **Ajouter** :

2. Choisissez les types de données de télémétrie que vous souhaitez exporter.

3. Créez ou sélectionnez le [compte de stockage Azure](../../storage/common/storage-introduction.md) sur lequel vous voulez stocker les données. Pour plus d’informations sur les options de tarification de stockage, consultez la [page officielle sur la tarification](https://azure.microsoft.com/pricing/details/storage/).

     Cliquez sur Ajouter, Destination de l’exportation, Compte de stockage, puis créez un magasin ou choisissez un magasin existant.

    > [!Warning]
    > Par défaut, l’emplacement de stockage est défini dans la même région géographique que votre ressource Application Insights. Si vous utilisez une autre région de stockage, vous risquez de subir des frais de transfert.

4. Créez ou sélectionnez un conteneur dans votre stockage.

> [!NOTE]
> Une fois votre exportation créée, les données nouvellement ingérées commencent à circuler vers Stockage Blob Azure. L’exportation continue transmet uniquement les nouvelles données de télémétrie créées/ingérées après activation de l’exportation continue. Les données présentes avant l’activation de l’exportation continue ne sont pas exportées, et il n’existe aucun moyen permettant d’exporter rétroactivement des données créées précédemment à l’aide de l’exportation continue.

Il peut y avoir un délai d'environ une heure avant que les données n’apparaissent dans le stockage.

Une fois la première exportation terminée, vous trouverez une structure similaire à ce qui suit dans votre conteneur de stockage d’objets blob Azure : (Cela varie en fonction des données que vous collectez.)

|Nom | Description |
|:----|:------|
| [Disponibilité](export-data-model.md#availability) | Consigne les [tests web de disponibilité](./monitor-web-app-availability.md).  |
| [Event](export-data-model.md#events) | Événements personnalisés générés par [TrackEvent()](./api-custom-events-metrics.md#trackevent). 
| [Exceptions](export-data-model.md#exceptions) |Signale des [exceptions](./asp-net-exceptions.md) sur le serveur et dans le navigateur.
| [Messages](export-data-model.md#trace-messages) | Envoyé par [TrackTrace](./api-custom-events-metrics.md#tracktrace) et par les [adaptateurs de journalisation](./asp-net-trace-logs.md).
| [Métriques](export-data-model.md#metrics) | Généré par les appels d’API des métriques.
| [PerformanceCounters](export-data-model.md) | Compteurs de performances collectés par Application Insights.
| [Demandes](export-data-model.md#requests)| Envoyées par [TrackRequest](./api-custom-events-metrics.md#trackrequest). Les modules standard les utilisent pour consigner le temps de réponse du serveur, mesuré sur le serveur.| 

### <a name="to-edit-continuous-export"></a>Pour modifier une exportation continue

Cliquez sur Exportation continue et sélectionnez le compte de stockage à modifier.

### <a name="to-stop-continuous-export"></a>Pour suspendre une exportation continue

Pour arrêter l’exportation, cliquez sur Désactiver. Lorsque vous cliquez de nouveau sur Activer, l’exportation redémarre avec de nouvelles données. Vous n’obtiendrez pas les données qui sont arrivées sur le portail alors que l’exportation était désactivée.

Pour arrêter définitivement l’exportation, supprimez-la simplement. Cette opération ne supprime pas vos données du stockage.

### <a name="cant-add-or-change-an-export"></a>Impossible d’ajouter ou de modifier une exportation ?
* Pour ajouter ou modifier des exportations, vous devez disposer de droits d’accès de propriétaire, de contributeur ou de contributeur Application Insights. [En savoir plus sur les rôles][roles].

## <a name="what-events-do-you-get"></a><a name="analyze"></a> Quels sont les événements que vous obtenez ?
Les données exportées sont les données de télémétrie brutes que nous recevons de votre application. Toutefois, nous ajoutons les données d’emplacement que nous calculons à partir de l’adresse IP du client.

Les données qui ont été ignorées par l’ [échantillonnage](./sampling.md) ne sont pas incluses dans les données exportées.

Les autres mesures calculées ne sont pas incluses. Par exemple, nous n’exportons pas l’utilisation moyenne du processeur, mais nous exportons la télémétrie brute à partir de laquelle la moyenne est calculée.

Les données incluent également les résultats de n’importe quel [test web de disponibilité](./monitor-web-app-availability.md) que vous avez configuré.

> [!NOTE]
> **Échantillonnage.** Si votre application envoie beaucoup de données, la fonctionnalité d’échantillonnage peut fonctionner et envoyer seulement une partie des données de télémétrie générées. [En savoir plus sur l'échantillonnage.](./sampling.md)
>
>

## <a name="inspect-the-data"></a><a name="get"></a> Inspection des données
Vous pouvez inspecter le stockage directement sur le portail. Cliquez sur Accueil dans le menu de gauche. En haut, sous « Services Azure », sélectionnez **Comptes de stockage**, puis sélectionnez le nom du compte de stockage. Dans la page de présentation, sous les services, sélectionnez **Objets blob**. Enfin, sélectionnez le nom du conteneur.

Pour examiner le stockage Azure dans Visual Studio, ouvrez **Afficher**, **Cloud Explorer**. (Si vous n’avez pas cette commande de menu, vous devez installer le SDK Azure : ouvrez la boîte de dialogue **Nouveau projet**, développez Visual C#/Cloud et choisissez **Obtenir Microsoft Azure SDK pour .NET**.)

Lorsque vous ouvrez votre magasin d’objets blob, vous voyez un conteneur avec un ensemble de fichiers blob. L'URI de chaque fichier est dérivé du nom de votre ressource Application Insights, sa clé d'instrumentation, le type/la date/l'heure de télémétrie. (Le nom de la ressource est tout en minuscules et la clé d'instrumentation omet les tirets.)

![Inspectez le magasin d’objets blob avec un outil adapté.](./media/export-telemetry/04-data.png)

La date et l’heure sont au format UTC et correspondent au moment où la télémétrie a été placée dans le magasin, et pas au moment où elle a été générée. Par conséquent, si vous écrivez du code pour télécharger les données, il peut parcourir les données de façon linéaire.

Voici le format du chemin d’accès :

```console
$"{applicationName}_{instrumentationKey}/{type}/{blobDeliveryTimeUtc:yyyy-MM-dd}/{ blobDeliveryTimeUtc:HH}/{blobId}_{blobCreationTimeUtc:yyyyMMdd_HHmmss}.blob"
```

Where

* `blobCreationTimeUtc` correspond à l'heure de création de l’objet blob dans le stockage intermédiaire interne
* `blobDeliveryTimeUtc` est l’heure de copie de l’objet blob vers le stockage de destination d’exportation

## <a name="data-format"></a><a name="format"></a> Format de données
* Chaque objet blob est un fichier texte qui contient plusieurs lignes séparées par des \n. Il contient les données de télémétrie traitées sur une période de trente secondes environ.
* Chaque ligne représente un point de données de télémétrie, par exemple une demande ou un affichage de page.
* Chaque ligne est un document JSON sans mise en forme. Si vous souhaitez afficher les lignes, ouvrez le blob dans Visual Studio et choisissez **Modifier** > **le fichier de format** > **avancé** :

   ![Consultez la télémétrie avec un outil approprié.](./media/export-telemetry/06-json.png)

Les durées sont exprimées en nombre de cycles, où 10 000 cycles = 1 ms. Par exemple, ces valeurs indiquent une durée de 1 ms pour envoyer une demande à partir du navigateur, 3 ms pour la recevoir et 1,8 s pour traiter la page dans le navigateur :

```json
"sendRequest": {"value": 10000.0},
"receiveRequest": {"value": 30000.0},
"clientProcess": {"value": 17970000.0}
```

[Référence de modèle de données détaillé pour les valeurs et types de propriétés.](export-data-model.md)

## <a name="processing-the-data"></a>Traitement des données
À petite échelle, vous pouvez écrire du code pour décomposer vos données, les lire dans une feuille de calcul et ainsi de suite. Par exemple :

```csharp
private IEnumerable<T> DeserializeMany<T>(string folderName)
{
   var files = Directory.EnumerateFiles(folderName, "*.blob", SearchOption.AllDirectories);
   foreach (var file in files)
   {
      using (var fileReader = File.OpenText(file))
      {
         string fileContent = fileReader.ReadToEnd();
         IEnumerable<string> entities = fileContent.Split('\n').Where(s => !string.IsNullOrWhiteSpace(s));
         foreach (var entity in entities)
         {
            yield return JsonConvert.DeserializeObject<T>(entity);
         }
      }
   }
}
```

Pour obtenir un exemple de code plus long, consultez [Utilisation d’un rôle de travail][exportasa].

## <a name="delete-your-old-data"></a><a name="delete"></a>Supprimer les anciennes données
C’est à vous de gérer votre capacité de stockage et de supprimer les anciennes données si nécessaire.

## <a name="if-you-regenerate-your-storage-key"></a>Si vous régénérez votre clé de stockage...
Si vous modifiez la clé de votre stockage, l’exportation continue cesse de fonctionner. Vous voyez alors une notification dans votre compte Azure.

Ouvrez l’onglet Exportation continue et modifiez votre exportation. Modifiez la destination de l’exportation, mais laissez le même stockage sélectionné. Cliquez sur OK pour confirmer.

L’exportation continue redémarre.

## <a name="export-samples"></a>Exemples d’exportation

* [Exporter vers SQL à l’aide de Stream Analytics][exportasa]
* [Stream Analytics - Exemple 2](export-stream-analytics.md)

À plus grande échelle, envisagez d’utiliser des clusters [HDInsight](https://azure.microsoft.com/services/hdinsight/) - Hadoop dans le cloud. HDInsight propose de nombreuses technologies pour gérer et analyser Big Data, et vous pouvez l’utiliser pour traiter les données qui ont été exportées depuis Application Insights.

## <a name="q--a"></a>Questions et réponses
* *Je veux simplement télécharger un graphique.*  

    Oui, vous pouvez le faire. En haut de l’onglet, cliquez sur **Exporter les données**.
* *J’ai configuré une exportation, mais il n’y a pas de données dans mon magasin.*

    Application Insights a-t-il reçu de la télémétrie de votre application depuis que vous avez configuré l’exportation ? Vous recevrez uniquement les nouvelles données.
* *J’ai essayé de configurer une exportation, mais l’accès lui a été refusé.*

    Si le compte appartient à votre organisation, vous devez être membre du groupe des propriétaires ou des collaborateurs.
* *Puis-je exporter directement vers mon propre magasin local ?*

    Non. Pour le moment, notre moteur d’exportation fonctionne uniquement avec le stockage Azure.  
* *Existe-t-il une limite à la quantité de données qu’il est possible de placer dans mon magasin ?*

    Non. Nous transmettons les données jusqu’à ce que vous supprimiez l’exportation. Nous arrêtons si nous atteignons les limites extérieures du stockage d’objets blob, mais ceci représente un volume très important. C’est à vous de contrôler la quantité de stockage vous utilisez.  
* *Combien d’objets blob devrais-je voir dans le stockage ?*

  * Pour chaque type de données que vous avez choisi d'exporter un objet blob est créé toutes les minutes (si les données sont disponibles).
  * En outre, pour les applications avec un trafic élevé, des unités de partition supplémentaires sont allouées. Dans ce cas, chaque unité crée un objet blob par minute.
* *J’ai régénéré la clé de mon espace de stockage ou modifié le nom du conteneur et l’exportation ne fonctionne plus.*

    Modifiez l’exportation et ouvrez l’onglet de destination d’exportation. Conservez le même stockage que celui sélectionné auparavant, puis cliquez sur OK pour confirmer. L’exportation redémarre. Si la modification a eu lieu dans les derniers jours, vous ne perdrez pas de données.
* *Est-il possible de suspendre l’exportation ?*

    Oui. Cliquez sur Désactiver.

## <a name="code-samples"></a>Exemples de code

* [Stream Analytics - Exemple](export-stream-analytics.md)
* [Exporter vers SQL à l’aide de Stream Analytics][exportasa]
* [Référence de modèle de données détaillé pour les valeurs et types de propriétés.](export-data-model.md)

<!--Link references-->

[exportasa]: ./code-sample-export-sql-stream-analytics.md
[roles]: ./resources-roles-access-control.md

