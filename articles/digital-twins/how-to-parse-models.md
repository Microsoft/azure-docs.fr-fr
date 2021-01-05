---
title: Analyser et valider les modèles
titleSuffix: Azure Digital Twins
description: Découvrez comment utiliser la bibliothèque de l’analyseur pour analyser des modèles DTDL.
author: baanders
ms.author: baanders
ms.date: 4/10/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 2e13762698efd8d5df42ab6315c990d4096168cf
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145529"
---
# <a name="parse-and-validate-models-with-the-dtdl-parser-library"></a>Analyser et valider des modèles avec la bibliothèque de l’analyseur DTDL

Les [modèles](concepts-models.md) dans Azure Digital Twins sont définis à l’aide du langage DTDL (Digital Twins Definition Language) basé sur JSON-LD. **Il est recommandé de valider les modèles hors connexion avant de les charger sur une instance Azure Digital Twins.**

Pour vous aider à effectuer cette opération, une bibliothèque d’analyse DTDL côté client .NET est fournie sur NuGet : [**Microsoft.Azure.DigitalTwins.Parser**](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/). 

Vous pouvez utiliser la bibliothèque de l’analyseur directement dans votre code C#, ou utiliser l’exemple de projet de code indépendant du langage basé sur la bibliothèque de l’analyseur : [**Exemple de validateur DTDL**](/samples/azure-samples/dtdl-validator/dtdl-validator).

## <a name="use-the-dtdl-validator-sample"></a>Utiliser l’exemple de validateur DTDL

Le [**Validateur DTDL**](/samples/azure-samples/dtdl-validator/dtdl-validator) est un exemple de projet pouvant valider des documents de modèle afin de s’assurer que le DTDL est valide. Il est basé sur la bibliothèque de l’analyseur .NET et est indépendant du langage. Vous pouvez l’utiliser avec le bouton *Télécharger le fichier ZIP* sur le lien de l’exemple.

Le code source montre des exemples d’utilisation de la bibliothèque de l’analyseur. Vous pouvez utiliser l’exemple de validateur peut être utilisé comme utilitaire de ligne de commande pour valider une arborescence de répertoires de fichiers DTDL. Il fournit également un mode interactif.

Dans le dossier de l’exemple de validateur DTDL, consultez le fichier *readme.md* pour obtenir des instructions sur la façon d’empaqueter l’exemple dans un exécutable autonome.

Après avoir créé un package autonome et ajouté l’exécutable à votre chemin d’accès, vous pouvez exécuter le validateur avec cette commande dans une console sur votre ordinateur :

```cmd/sh
DTDLValidator
```

Avec les options par défaut, l’exemple recherche les fichiers `*.json` dans le répertoire actif et tous les sous-répertoires. Vous pouvez également ajouter l’option suivante pour que l’exemple recherche dans le répertoire indiqué et dans tous les sous-répertoires des fichiers portant l’extension *.dtdl*  :

```cmd/sh
DTDLValidator -d C:\Work\DTDL -e dtdl 
```

Vous pouvez ajouter l’option `-i` pour que l’exemple passe en mode interactif :

```cmd/sh
DTDLValidator -i
```

Pour plus d’informations sur cet exemple, consultez le code source ou exécutez `DTDLValidator --help`.

## <a name="use-the-net-parser-library"></a>Utiliser la bibliothèque d’analyseur .NET 

La bibliothèque [**Microsoft.Azure.DigitalTwins.Parser**](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/) fournit un accès par modèle aux définitions DTDL agissant essentiellement comme équivalent de C# pour DTDL. Vous pouvez l’utiliser indépendamment du [Kit de développement logiciel (SDK) Azure Digital Twins](how-to-use-apis-sdks.md), en particulier pour la validation de DTDL dans un éditeur visuel ou de texte. Elle est utile pour s’assurer que vos fichiers de définition de modèle sont valides avant d’essayer de les charger vers le service.

Pour utiliser la bibliothèque de l’analyseur, vous devez lui fournir un ensemble de documents DTDL. En règle générale, vous récupérez ces documents de modèle à partir du service, mais vous pouvez également les rendre disponibles localement si votre client était initialement responsable de les charger sur le service. 

Voici le flux de travail général pour l’utilisation de l’analyseur :
1. Récupérez tout ou partie des documents DTDL du service.
2. Transmettez les documents DTDL en mémoire retournés à l’analyseur.
3. L’analyseur valide le jeu de documents qui lui est transmis et retourne des informations détaillées sur l’erreur. Cette fonctionnalité est utile dans des scénarios d’éditeur.
4. Utilisez les API d’analyseurs pour poursuivre l’analyse des modèles inclus dans l’ensemble de documents. 

Les fonctionnalités de l’analyseur sont les suivantes :
* Obtenir toutes les interfaces de modèle implémentées (contenu de la section `extends` de l’interface).
* Obtenir l’ensemble des propriétés, de la télémétrie, des commandes, des composants et des relations déclarés dans le modèle. Cette commande obtient également toutes les métadonnées incluses dans ces définitions et prend en compte l’héritage (sections`extends`).
* Récupérer toutes les définitions de modèle complexes.
* Déterminer si un modèle peut être assigné à partir d’un autre.

> [!NOTE]
> Les appareils [IoT Plug-and-Play (PnP)](../iot-pnp/overview-iot-plug-and-play.md) utilisent une petite variante de syntaxe pour décrire leur fonctionnalité. Cette variante de syntaxe est un sous-ensemble sémantiquement compatible du langage DTDL utilisé dans Azure Digital Twins. Lorsque vous utilisez la bibliothèque de l’analyseur, vous n’avez pas besoin de savoir quelle variante de syntaxe a été utilisée pour créer le DTDL pour votre jumeau numérique. Par défaut, l’analyseur retourne toujours le même modèle pour la syntaxe PnP et Azure Digital Twins.

### <a name="code-with-the-parser-library"></a>Coder avec la bibliothèque de l’analyseur

Vous pouvez utiliser la bibliothèque de l’analyseur directement, par exemple, pour valider des modèles dans votre propre application ou pour générer une interface utilisateur, des tableaux de bord et des rapports dynamiques basés sur des modèles.

Pour prendre en charge l’exemple de code de l’analyseur ci-dessous, considérez plusieurs modèles définis dans une instance Azure Digital Twins :

```json
[
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:contoso:coffeeMaker;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Component",
        "name": "coffeeMaker",
        "schema": "dtmi:com:contoso:coffeeMakerInterface;1"
      }
    ]
  },
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:contoso:coffeeMakerInterface;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Property",
        "name": "waterTemp",
        "schema": "double"
      }
    ]
  },
  {
    "@context": "dtmi:dtdl:context;2",
    "@id": "dtmi:com:contoso:coffeeBar;1",
    "@type": "Interface",
    "contents": [
      {
        "@type": "Relationship",
        "name": "foo",
        "target": "dtmi:com:contoso:coffeeMaker;1"
      },
      {
        "@type": "Property",
        "name": "capacity",
        "schema": "integer"
      }
    ]
  }
]
```

Le code suivant montre un exemple d’utilisation de la bibliothèque de l’analyseur pour refléter ces définitions en C# :

```csharp
async void ParseDemo(DigitalTwinsClient client)
{
    try
    {
        AsyncPageable<DigitalTwinsModelData> mdata = client.GetModelsAsync(new GetModelsOptions { IncludeModelDefinition = true });
        List<string> models = new List<string>();
        await foreach (DigitalTwinsModelData md in mdata)
            models.Add(md.DtdlModel);
        ModelParser parser = new ModelParser();
        IReadOnlyDictionary<Dtmi, DTEntityInfo> dtdlOM = await parser.ParseAsync(models);

        List<DTInterfaceInfo> interfaces = new List<DTInterfaceInfo>();
        IEnumerable<DTInterfaceInfo> ifenum = 
            from entity in dtdlOM.Values
            where entity.EntityKind == DTEntityKind.Interface
            select entity as DTInterfaceInfo;
        interfaces.AddRange(ifenum);
        foreach (DTInterfaceInfo dtif in interfaces)
        {
            PrintInterfaceContent(dtif, dtdlOM);
        }

    } catch (RequestFailedException rex)
    {

    }
}

void PrintInterfaceContent(DTInterfaceInfo dtif, IReadOnlyDictionary<Dtmi, DTEntityInfo> dtdlOM, int indent=0)
{
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < indent; i++) sb.Append("  ");
    Console.WriteLine($"{sb}Interface: {dtif.Id} | {dtif.DisplayName}");
    SortedDictionary<string, DTContentInfo> contents = dtif.Contents;
    foreach (DTContentInfo item in contents.Values)
    {
        switch (item.EntityKind)
        {
            case DTEntityKind.Property:
                DTPropertyInfo pi = item as DTPropertyInfo;
                Console.WriteLine($"{sb}--Property: {pi.Name} with schema {pi.Schema}");
                break;
            case DTEntityKind.Relationship:
                DTRelationshipInfo ri = item as DTRelationshipInfo;
                Console.WriteLine($"{sb}--Relationship: {ri.Name} with target {ri.Target}");
                break;
            case DTEntityKind.Telemetry:
                DTTelemetryInfo ti = item as DTTelemetryInfo;
                Console.WriteLine($"{sb}--Telemetry: {ti.Name} with schema {ti.Schema}");
                break;
            case DTEntityKind.Component:
                DTComponentInfo ci = item as DTComponentInfo;
                Console.WriteLine($"{sb}--Component: {ci.Id} | {ci.Name}");
                dtdlOM.TryGetValue(ci.Id, out DTEntityInfo value);
                DTInterfaceInfo component = value as DTInterfaceInfo;
                PrintInterfaceContent(component, dtdlOM, indent + 1);
                break;
            default:
                break;
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez fini d’écrire vos modèles, découvrez comment les télécharger (et effectuer d’autres opérations de gestion) avec les API DigitalTwinsModels :
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)