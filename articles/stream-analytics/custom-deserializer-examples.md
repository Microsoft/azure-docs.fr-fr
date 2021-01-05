---
title: Lire les entrées dans n’importe quel format avec des désérialiseurs .NET personnalisés dans Azure Stream Analytics
description: Cet article explique le format de sérialisation et les interfaces qui définissent les désérialiseurs .NET personnalisés pour les tâches Azure Stream Analytics cloud et Edge.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 1/28/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 4adf0b8a7c12abed9689b9ac0cc9c5d5c8c3f980
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94488432"
---
# <a name="read-input-in-any-format-using-net-custom-deserializers"></a>Lire les entrées dans n’importe quel format avec des désérialiseurs .NET personnalisés

Les désérialiseurs .NET personnalisés permettent à votre travail Azure Stream Analytics de lire des données à partir de formats en dehors des trois [formats de données intégrés](stream-analytics-parsing-json.md). Cet article explique le format de sérialisation et les interfaces qui définissent les désérialiseurs .NET personnalisés pour les tâches Azure Stream Analytics cloud et Edge. Il existe également des exemples de désérialiseurs pour la mémoire tampon de protocole et le format CSV.

## <a name="net-custom-deserializer"></a>Désérialiseur .NET personnalisé

Les exemples de code suivants sont les interfaces qui définissent le désérialiseur personnalisé et implémentent `StreamDeserializer<T>`.

`UserDefinedOperator` est la classe de base pour tous les opérateurs de diffusion personnalisés. Elle initialise `StreamingContext`, qui fournit le contexte qui inclut le mécanisme de publication des diagnostics dont vous aurez besoin pour déboguer les problèmes liés à votre désérialiseur.

```csharp
    public abstract class UserDefinedOperator
    {
        public abstract void Initialize(StreamingContext streamingContext);
    }
```

L’extrait de code suivant est la désérialisation pour les données de diffusion en continu. 

Les erreurs ignorables doivent être émises avec `IStreamingDiagnostics` et transmises par le biais de la méthode Initialize de `UserDefinedOperator`. Toutes les exceptions seront traitées comme des erreurs, et le désérialiseur sera recréé. Après un certain nombre d’erreurs, le travail passe à l’état d’échec.

`StreamDeserializer<T>` désérialise un flux en un objet de type `T`. Les conditions suivantes doivent être remplies :

1. T est une classe ou un struct.
1. Tous les champs publics dans T sont au choix
    1. Un type parmi [sbyte, byte, short, ushort, int, uint, long, DateTime, string, float, double] ou leur équivalent pouvant accepter la valeur Null.
    1. Une autre struct ou classe qui suit les mêmes règles.
    1. Un tableau de type `T2` qui suit les mêmes règles.
    1. Une IList`T2` où T2 suit les mêmes règles.
    1. N’a pas de types récursifs.

Le paramètre `stream` est le flux contenant l’objet sérialisé. `Deserialize` renvoie une collection d’instances de `T`.

```csharp
    public abstract class StreamDeserializer<T> : UserDefinedOperator
    {
        public abstract IEnumerable<T> Deserialize(Stream stream);
    }
```

`StreamingContext` fournit un contexte qui inclut le mécanisme de publication des diagnostics pour l’opérateur de l’utilisateur.

```csharp
    public abstract class StreamingContext
    {
        public abstract StreamingDiagnostics Diagnostics { get; }
    }
```

`StreamingDiagnostics` représente les diagnostics pour les opérateurs définis par l’utilisateur, notamment le sérialiseur, le désérialiseur et les fonctions définies par l’utilisateur.

`WriteError` écrit un message d’erreur dans les journaux de ressources et envoie l’erreur aux diagnostics.

`briefMessage` est un bref message d’erreur. Ce message s’affiche dans les diagnostics et est utilisé par l’équipe produit à des fins de débogage. N’incluez pas d’informations sensibles et maintenez une taille de message inférieure à 200 caractères

`detailedMessage` est un message d’erreur détaillé qui est uniquement ajouté aux journaux de ressources dans votre stockage. Ce message doit comprendre moins de 2000 caractères.

```csharp
    public abstract class StreamingDiagnostics
    {
        public abstract void WriteError(string briefMessage, string detailedMessage);
    }
```

## <a name="deserializer-examples"></a>Exemples de désérialiseur

Cette section vous montre comment écrire des désérialiseurs personnalisés pour Protobuf et CSV. Pour d’autres exemples, tels que le format AVRO pour la capture Event Hub, consultez [Azure Stream Analytics sur GitHub](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="protocol-buffer-protobuf-format"></a>Format de tampon de protocole (Protobuf)

Il s’agit d’un exemple utilisant le format de tampon de protocole.

Supposons la définition de tampon de protocole suivante.

```proto
syntax = "proto3";
// protoc.exe from nuget "Google.Protobuf.Tools" is used to generate .cs file from this schema definition.
// Run below command to generate the csharp class
// protoc.exe --csharp_out=. MessageBodyProto.proto

package SimulatedTemperatureSensor;
message MessageBodyProto {
    message Ambient {
      double temperature = 1;
      int64 humidity = 2;
    }

    message Machine {
      double temperature = 1;
      double pressure = 2;
    }

    Machine machine = 1;
    Ambient ambient = 2;
    string timeCreated = 3;
}
```

L’exécution de `protoc.exe` à partir du NuGet **Google.Protobuf.Tools** génère un fichier. cs avec la définition. Le fichier généré n’est pas montré ici. Vous devez vous vérifier que la version de Protobuf NuGet utilisée dans votre projet Stream Analytics correspond à la version de Protobuf qui a servi à générer l’entrée. 

L’extrait de code suivant est l’implémentation du désérialiseur en supposant que le fichier généré est inclus dans le projet. Cette implémentation est simplement un wrapper léger sur le fichier généré.

```csharp
    public class MessageBodyDeserializer : StreamDeserializer<SimulatedTemperatureSensor.MessageBodyProto>
    {
        public override IEnumerable<SimulatedTemperatureSensor.MessageBodyProto> Deserialize(Stream stream)
        {
            while (stream.Position < stream.Length)
            {
                yield return SimulatedTemperatureSensor.MessageBodyProto.Parser.ParseDelimitedFrom(stream);
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
        }
    }
```

### <a name="csv"></a>CSV

L’extrait de code suivant est un désérialiseur CSV simple qui illustre également la propagation des erreurs.

```csharp
using System.Collections.Generic;
using System.IO;

using Microsoft.Azure.StreamAnalytics;
using Microsoft.Azure.StreamAnalytics.Serialization;

namespace ExampleCustomCode.Serialization
{
    public class CustomCsvDeserializer : StreamDeserializer<CustomEvent>
    {
        private StreamingDiagnostics streamingDiagnostics;

        public override IEnumerable<CustomEvent> Deserialize(Stream stream)
        {
            using (var sr = new StreamReader(stream))
            {
                string line = sr.ReadLine();
                while (line != null)
                {
                    if (line.Length > 0 && !string.IsNullOrWhiteSpace(line))
                    {
                        string[] parts = line.Split(',');
                        if (parts.Length != 3)
                        {
                            streamingDiagnostics.WriteError("Did not get expected number of columns", $"Invalid line: {line}");
                        }
                        else
                        {
                            yield return new CustomEvent()
                            {
                                Column1 = parts[0],
                                Column2 = parts[1],
                                Column3 = parts[2]
                            };
                        }
                    }

                    line = sr.ReadLine();
                }
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
            this.streamingDiagnostics = streamingContext.Diagnostics;
        }
    }

    public class CustomEvent
    {
        public string Column1 { get; set; }

        public string Column2 { get; set; }

        public string Column3 { get; set; }
    }
}

```

## <a name="serialization-format-for-rest-apis"></a>Format de sérialisation pour les API REST

Chaque entrée de Stream Analytics a un **format de sérialisation**. Pour plus d’informations sur les options d’entrée, consultez la documentation relative à [l’API REST d’entrée](/rest/api/streamanalytics/2016-03-01/inputs).

Le code JavaScript suivant est un exemple de format de sérialisation du désérialiseur .NET lors de l’utilisation de l’API REST :

```javascript
{    
   "properties":{    
      "type":"stream",  
      "serialization":{    
         "type":"CustomCLR",  
         "properties":{    
            "serializationDllPath":"<path to the dll inside UserCustomCode\CLR\ folder>", 
            "serializationClassName":"<Full name of the deserializer class name>" 
         }  
      }
   }  
}  
```

La classe `serializationClassName` doit implémenter `StreamDeserializer<T>`. Cela est décrit dans les sections suivantes.

## <a name="region-support"></a>Prise en charge de la région

Cette fonctionnalité est disponible dans les régions suivantes :

* Centre-USA Ouest
* Europe Nord
* USA Est
* USA Ouest
* USA Est 2
* Europe Ouest

Vous pouvez faire une [demande de prise en charge](https://aka.ms/ccodereqregion) pour des régions supplémentaires.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="when-will-this-feature-be-available-in-all-azure-regions"></a>Quand cette fonctionnalité sera-t-elle disponible dans toutes les régions Azure ?

Cette fonctionnalité est disponible dans [6 régions](#region-support). Si vous êtes intéressé par l’utilisation de cette fonctionnalité dans une autre région, vous pouvez [envoyer une demande](https://aka.ms/ccodereqregion). La prise en charge de toutes les régions Azure fait partie de la feuille de route.

### <a name="can-i-access-metadatapropertyvalue-from-my-inputs-similar-to-getmetadatapropertyvalue-function"></a>Puis-je accéder à MetadataPropertyValue à partir de mes entrées similaires à la fonction GetMetadataPropertyValue ?

Cette fonctionnalité n'est pas prise en charge. Si vous avez besoin de cette fonctionnalité, vous pouvez voter pour cette demande sur [UserVoice](https://feedback.azure.com/forums/270577-stream-analytics/suggestions/38779801-accessing-input-metadata-properties-in-custom-dese).

### <a name="can-i-share-my-deserializer-implementation-with-the-community-so-that-others-can-benefit"></a>Puis-je partager mon implémentation de désérialiseur avec la communauté afin que d’autres puissent en bénéficier ?

Une fois que vous avez implémenté votre désérialiseur, vous pouvez aider d’autres personnes en le partageant avec la communauté. Envoyez votre code au [référentiel GitHub d’Azure Stream Analytics](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="what-are-the-other-limitations-of-using-custom-deserializers-in-stream-analytics"></a>Quelles sont les autres limites de l’utilisation de désérialiseurs personnalisés dans Stream Analytics ?

Si votre entrée est au format Protobuf avec un schéma contenant le type `MapField`, vous ne pourrez pas implémenter de désérialiseur personnalisé. En outre, les désérialiseurs personnalisés ne prennent pas en charge les exemples de données ou les données d’aperçu. 

## <a name="next-steps"></a>Étapes suivantes

* [Désérialiseurs .NET personnalisés pour les travaux cloud Azure Stream Analytics](custom-deserializer.md)
