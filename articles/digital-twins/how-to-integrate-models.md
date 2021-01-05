---
title: Intégrer des modèles standard du secteur
titleSuffix: Azure Digital Twins
description: Découvrez comment intégrer des modèles standard du secteur dans DTDL pour Azure Digital Twins, soit en utilisant des ontologies DTDL spéciales, soit en convertissant des ontologies existantes.
author: baanders
ms.author: baanders
ms.date: 11/04/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: f5bfe128ddc04e8048bb89a8e39035434dfd2b92
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352879"
---
# <a name="integrate-industry-standard-models-with-dtdl-for-azure-digital-twins"></a>Intégrer des modèles standard du secteur à DTDL pour Azure Digital Twins

L’utilisation de modèles basés sur des normes du secteur ou d’une représentation d’ontologie standard, comme RDF ou OWL, constitue un point de départ intéressant quand vous concevez vos modèles Azure Digital Twins. L’utilisation de modèles sectoriels facilite également la normalisation et le partage d’informations.

Pour être utilisé avec Azure Digital Twins, un modèle doit être représenté à l’aide du [**langage DTDL (Digital Twins Definition Language)**](concepts-models.md) basé sur JSON-LD. Par conséquent, cet article décrit comment représenter vos modèles standard du secteur en DTDL, en intégrant les concepts existants à la sémantique DTDL, de sorte qu’Azure Digital Twins puissent les utiliser. Le modèle DTDL sert alors de source de vérité pour le modèle dans Azure Digital Twins.

Il existe trois parcours possibles pour intégrer des modèles standard du secteur à DTDL :
* **Adoption** : Vous pouvez démarrer votre solution avec une ontologie DTDL open source qui a été conçue sur des normes du secteur d’activité largement adoptées. 
* **Conversion** : Si vous avez déjà des modèles existants, vous avez besoin de les convertir en DTDL.
* **Création** : Vous pouvez toujours développer vos propres modèles DTDL personnalisés ex nihilo, comme décrit dans le [*guide pratique Gérer des modèles personnalisés*](how-to-manage-model.md).

## <a name="adopt-an-open-source-dtdl-ontology"></a>Adopter une ontologie DTDL open source

Il est souvent plus facile de commencer avec une ontologie DTDL open source que de démarrer à partir d’une page vierge. 

Par exemple, les solutions Bâtiments intelligents peuvent tirer parti de l’[**ontologie RealEstateCore basée sur DTDL**](https://github.com/Azure/opendigitaltwins-building), qui fournit une base commune pour la modélisation des bâtiments intelligents tout en exploitant des normes du secteur visant à prévenir la réinvention. 

Ces ontologies DTDL open source proposent aussi des bonnes pratiques de consommation et d’extension appropriée des modèles. 

## <a name="convert-existing-models-to-dtdl"></a>Convertir des modèles existants en DTDL

La plupart des modèles sectoriels (également appelés **ontologies**) sont basés sur des normes web sémantiques comme [OWL](https://www.w3.org/OWL/), [RDF](https://www.w3.org/2001/sw/wiki/RDF) et [RDFS](https://www.w3.org/2001/sw/wiki/RDFS). 

Pour utiliser un modèle avec Azure Digital Twins, il doit être au format DTDL. Cette section décrit des conseils de conception généraux sous la forme d’un **modèle de conversion** permettant de convertir des modèles RDF en DTDL pour qu’ils puissent être utilisés avec Azure Digital Twins. 

Elle contient aussi un [**exemple de code de convertisseur** pour un convertisseur RDF](#sample-converter-application), qui a été validé pour le schéma [Brick](https://brickschema.org/ontology/) et qui peut être étendu pour d’autres schémas du secteur du bâtiment.

### <a name="conversion-pattern"></a>Modèle de conversion

Il existe plusieurs bibliothèques tierces qui peuvent être utilisées lors de la conversion de modèles RDF en DTDL. Certaines de ces bibliothèques vous permettent de charger votre fichier de modèle dans un graphique. Vous pouvez parcourir le graphique en recherchant des constructions RDFS et OWL spécifiques, et les convertir en DTDL.   

Le tableau suivant est un exemple de la façon dont les constructions RDFS et OWL peuvent être mappées à DTDL. 

| Concept RDFS/OWL | Construction RDFS/OWL | Concept DTDL | Construction DTDL |
| --- | --- | --- | --- |
| Classes | `owl:Class`<br>Suffixe IRI<br>``rdfs:label``<br>``rdfs:comment`` | Interface | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment` 
| Sous-classes | `owl:Class`<br>Suffixe IRI<br>`rdfs:label`<br>`rdfs:comment`<br>`rdfs:subClassOf` | Interface | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment`<br>`extends` 
| Propriétés DataType | `owl:DatatypeProperty`<br>`rdfs:label` ou `INode`<br>`rdfs:label`<br>`rdfs:range` | Propriétés de l’interface | `@type:Property`<br>`name`<br>`displayName`<br>`schema` 
| Propriétés des objets | `owl:ObjectProperty`<br>`rdfs:label` ou `INode`<br>`rdfs:range`<br>`rdfs:comment`<br>`rdfs:label` | Relationship | `type:Relationship`<br>`name`<br>`target` (ou omis si aucune `rdfs:range`)<br>`comment`<br>`displayName`<br>

L’extrait de code C# suivant montre comment un fichier de modèle RDF est chargé dans un graphique et converti en DTDL à l’aide de la bibliothèque [**dotNetRDF**](https://www.dotnetrdf.org/). 

```csharp
using VDS.RDF.Ontology; 
using VDS.RDF.Parsing; 
using Microsoft.Azure.DigitalTwins.Parser; 

//...

Console.WriteLine("Reading file..."); 

FileLoader.Load(_ontologyGraph, rdfFile.FullName); 

// Start looping through for each owl:Class 
foreach (OntologyClass owlClass in _ontologyGraph.OwlClasses) 
{ 

    // Generate a DTMI for the owl:Class 
    string Id = GenerateDTMI(owlClass); 

    if (!String.IsNullOrEmpty(Id)) 
    { 

        Console.WriteLine($"{owlClass.ToString()} -> {Id}"); 

        // Create Interface
        DtdlInterface dtdlInterface = new DtdlInterface 
        { 
            Id = Id, 
            Type = "Interface", 
            DisplayName = GetInterfaceDisplayName(owlClass), 
            Comment = GetInterfaceComment(owlClass), 
            Contents = new List<DtdlContents>() 
        }; 

        // Use DTDL 'extends' for super classes 
        IEnumerable<OntologyClass> foundSuperClasses = owlClass.DirectSuperClasses; 

        //... 
     }

     // Add interface to the list of interfaces 
     _interfaceList.Add(dtdlInterface); 
} 

// Serialize to JSON 
var json = JsonConvert.SerializeObject(_interfaceList); 

//...
``` 

### <a name="sample-converter-application"></a>Exemple d’application de convertisseur 

Un exemple d’application disponible convertit un fichier de modèle RDF en [DTDL (version 2)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) pour une utilisation par le service Azure Digital Twins. Il a été validé pour le schéma [Brick](https://brickschema.org/ontology/) et peut être étendu pour d’autres schémas du secteur du bâtiment (par exemple, [Building Topology Ontology (BOT)](https://w3c-lbd-cg.github.io/bot/), [Semantic Sensor Network](https://www.w3.org/TR/vocab-ssn/) ou [buildingSmart Industry Foundation Classes (IFC)](https://technical.buildingsmart.org/standards/ifc/ifc-schema-specifications/)).

L’exemple est une application en ligne de commande .NET Core appelée **RdfToDtdlConverter**.

Vous pouvez vous procurer l’exemple ici : [**RdfToDtdlConverter**](/samples/azure-samples/rdftodtdlconverter/digital-twins-model-conversion-samples/). 

Pour télécharger le code sur votre ordinateur, cliquez sur le bouton *Télécharger ZIP* sous le titre de la page d’accueil de l’exemple. Cela permet de télécharger un fichier *ZIP* sous le nom *RdfToDtdlConverter_sample_application_to_convert_RDF_to_DTDL.zip*, que vous pouvez ensuite décompresser et explorer.

Vous pouvez utiliser cet exemple pour voir les modèles de conversion en contexte et comme bloc de construction pour vos propres applications effectuant des conversions de modèle en fonction de vos besoins spécifiques.

## <a name="validate-and-upload-dtdl-models"></a>Valider et charger des modèles DTDL

[!INCLUDE [Azure Digital Twins: validate models info](../../includes/digital-twins-validate.md)]

Une fois qu’un modèle est converti et validé, vous pouvez **Charger le modèle dans l’instance Azure Digital Twins**. Pour plus d’informations sur ce processus, consultez la section [*Charger des modèles*](how-to-manage-model.md#upload-models) de *Guide pratique : Gérer les modèles personnalisés*.

## <a name="next-steps"></a>Étapes suivantes 

Découvrez plus d’informations sur la conception et la gestion de modèles de jumeaux numériques :
 
* [*Concepts : Modèles personnalisés*](concepts-models.md)
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)
* [*Guide pratique : Analyser et valider les modèles*](how-to-parse-models.md)