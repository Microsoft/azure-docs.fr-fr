---
title: Gérer des modèles personnalisés
titleSuffix: Azure Digital Twins
description: Comprendre comment créer, modifier et supprimer un modèle dans Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: ca56c285baff9982ff465b0d4115d15eadedb8c9
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534753"
---
# <a name="manage-azure-digital-twins-models"></a>Gérer les modèles Azure Digital Twins

Vous pouvez gérer les [modèles](concepts-models.md) que votre instance Azure Digital Twins connait à l’aide des API [**DigitalTwinModels**](/rest/api/digital-twins/dataplane/models), du [SDK .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) ou de [l’interface CLI Azure Digital Twins](how-to-use-cli.md). 

Les opérations de gestion incluent le chargement, la validation, la récupération et la suppression des modèles. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

## <a name="ways-to-manage-models"></a>Modes de gestion des modèles

[!INCLUDE [digital-twins-ways-to-manage.md](../../includes/digital-twins-ways-to-manage.md)]

## <a name="create-models"></a>Créer des modèles

Les modèles pour Azure Digital Twins sont écrits en DTDL et enregistrés sous forme de fichiers *.JSON*. Il existe également une [extension DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) disponible pour [Visual Studio Code](https://code.visualstudio.com/), offrant une validation de la syntaxe et d’autres fonctionnalités facilitant l’écriture de documents DTDL.

Prenons l’exemple d’un hôpital souhaitant disposer d’une représentation numérique des pièces de son bâtiment. Chaque pièce contient un distributeur de savon intelligent permettant de contrôler le lavage des mains, et des capteurs pour suivre le trafic.

La première étape de la solution consiste à créer des modèles pour représenter chaque aspect de l’hôpital. Dans ce scénario, la chambre d’un patient peut être décrite comme suit :

```json
{
  "@id": "dtmi:com:contoso:PatientRoom;1",
  "@type": "Interface",
  "@context": "dtmi:dtdl:context;2",
  "displayName": "Patient Room",
  "contents": [
    {
      "@type": "Property",
      "name": "visitorCount",
      "schema": "double"
    },
    {
      "@type": "Property",
      "name": "handWashCount",
      "schema": "double"
    },
    {
      "@type": "Property",
      "name": "handWashPercentage",
      "schema": "double"
    },
    {
      "@type": "Relationship",
      "name": "hasDevices"
    }
  ]
}
```

> [!NOTE]
> Il s’agit d’un exemple de corps pour un fichier. JSON au sein duquel un modèle est défini et enregistré, et doit être téléchargé dans le cadre d’un projet client. En revanche, l’appel d’API REST prend un tableau de définitions de modèle comme celle ci-dessus (qui est mappée à un `IEnumerable<string>` dans le kit de développement logiciel .NET). Par conséquent, pour utiliser ce modèle dans l’API REST directement, entourez-le avec des crochets.

Ce modèle définit un nom et un ID unique pour la chambre du patient, et les propriétés pour représenter le nombre de visiteurs et l’état du lavage à la main (ces compteurs seront mis à jour à partir des capteurs de mouvement et des distributeurs de savon intelligents, et ils seront utilisés ensemble pour calculer une propriété de *pourcentage de lavage de main*). Le modèle définit également une relation *hasDevices*, qui sera utilisée pour connecter toute [représentation numérique](concepts-twins-graph.md) basée sur ce modèle *Room* aux périphériques réels.

En suivant cette méthode, vous pouvez définir des modèles pour l’hôpital entier, ou bien pour certaines zones.

### <a name="validate-syntax"></a>Valider la syntaxe

[!INCLUDE [Azure Digital Twins: validate models info](../../includes/digital-twins-validate.md)]

## <a name="upload-models"></a>Charger des modèles

Une fois les modèles créés, vous pouvez les charger vers l’instance Azure Digital Twins.

> [!TIP]
> Il est recommandé de valider les modèles hors connexion avant de les charger sur une instance Azure Digital Twins. Vous pouvez utiliser la [bibliothèque de l’analyseur côté client DTDL](https://nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser/) et [l’exemple de validateur DTDL](/samples/azure-samples/dtdl-validator/dtdl-validator) décrit dans [*Comment : Analyser et valider les modèles*](how-to-parse-models.md) pour vérifier vos modèles avant de les charger vers le service.

Lorsque vous êtes prêt à télécharger un modèle, vous pouvez utiliser l’extrait de code suivant :

```csharp
// 'client' is an instance of DigitalTwinsClient
// Read model file into string (not part of SDK)
StreamReader r = new StreamReader("MyModelFile.json");
string dtdl = r.ReadToEnd(); r.Close();
string[] dtdls = new string[] { dtdl };
client.CreateModels(dtdls);
```

Notez que la méthode `CreateModels` accepte plusieurs fichiers dans une seule transaction. Voici un exemple pour illustrer ce qui suit :

```csharp
var dtdlFiles = Directory.EnumerateFiles(sourceDirectory, "*.json");

List<string> dtdlStrings = new List<string>();
foreach (string fileName in dtdlFiles)
{
    // Read model file into string (not part of SDK)
    StreamReader r = new StreamReader(fileName);
    string dtdl = r.ReadToEnd(); r.Close();
    dtdlStrings.Add(dtdl);
}
client.CreateModels(dtdlStrings);
```

Les fichiers de modèle peuvent en contenir plusieurs. Dans un tel cas, les modèles doivent être placés dans un tableau JSON. Par exemple :

```json
[
  {
    "@id": "dtmi:com:contoso:Planet",
    "@type": "Interface",
    //...
  },
  {
    "@id": "dtmi:com:contoso:Moon",
    "@type": "Interface",
    //...
  }
]
```
 
Lors du chargement, les fichiers de modèle sont validés par le service.

## <a name="retrieve-models"></a>Récupérer des modèles

Vous pouvez répertorier et récupérer des modèles stockés sur votre instance Azure Digital Twins. 

Voici les options disponibles :
* Récupérer un seul modèle
* Récupérer tous les modèles
* Récupérer les métadonnées et les dépendances pour les modèles

Voici quelques exemples d’appels :

```csharp
// 'client' is a valid DigitalTwinsClient object

// Get a single model, metadata and data
DigitalTwinsModelData md1 = client.GetModel(id);

// Get a list of the metadata of all available models
Pageable<DigitalTwinsModelData> pmd2 = client.GetModels();

// Get models and metadata for a model ID, including all dependencies (models that it inherits from, components it references)
Pageable<DigitalTwinsModelData> pmd3 = client.GetModels(new GetModelsOptions { IncludeModelDefinition = true });
```

Les appels d’API pour récupérer les modèles retournent tous les objets `DigitalTwinsModelData`. `DigitalTwinsModelData` contient les métadonnées relatives au modèle stocké dans l’instance Azure Digital Twins, par exemple le nom, le DTMI et la date de création du modèle. L’objet `DigitalTwinsModelData` comprend également le modèle lui-même. En fonction des paramètres, vous pouvez utiliser les appels de récupération pour récupérer uniquement les métadonnées (utile lorsque vous souhaitez afficher une liste d’interfaces utilisateur des outils disponibles, par exemple) ou le modèle entier.

L’appel `RetrieveModelWithDependencies` retourne non seulement le modèle demandé, mais également tous les modèles dont il dépend.

Les modèles ne sont pas nécessairement retournés exactement sous la même forme que dans le document au sein duquel ils ont été chargés. Azure Digital Twins garantit uniquement que la forme de retour est sémantiquement équivalente. 

## <a name="update-models"></a>Mettre à jour les modèles

Une fois qu’un modèle est chargé dans votre instance Azure Digital Twins, l’interface de modèle entière est immuable. Cela signifie qu’il n’y a pas de « modification » traditionnelle des modèles. Azure Digital Twins n’autorise pas non plus le rechargement du même modèle.

Mais si vous voulez apporter des modifications à un modèle (par exemple, modifier `displayName` ou `description`), vous pouvez le faire en chargeant une **version plus récente** de ce modèle. 

### <a name="model-versioning"></a>Gestion des versions des modèles

Pour créer une nouvelle version d’un modèle existant, commencez par le DTDL du modèle d’origine. Mettez à jour, ajoutez ou supprimez les champs à modifier.

Ensuite, marquez ce modèle comme une version plus récente en mettant à jour son champ `id`. La dernière section de l’ID de modèle, après le point-virgule (`;`), représente le numéro de modèle. Pour indiquer qu’il s’agit maintenant d’une version plus à jour de ce modèle, incrémentez le nombre à la fin de la valeur `id` en un nombre supérieur au numéro de version actuel.

Par exemple, si votre ID de modèle précédent ressemble à ceci :

```json
"@id": "dtmi:com:contoso:PatientRoom;1",
```

la version 2 de ce modèle peut se présenter comme suit :

```json
"@id": "dtmi:com:contoso:PatientRoom;2",
```

Ensuite, chargez la nouvelle version du modèle dans votre instance. 

Cette version du modèle devient alors disponible dans votre instance pour les jumeaux numériques. Elle ne remplace **pas** les versions antérieures du modèle. Ainsi, plusieurs versions du modèle coexistent dans votre instance jusqu’à ce que vous [les supprimiez](#remove-models).

### <a name="impact-on-twins"></a>Impact sur les jumeaux

Quand vous créez un jumeau, étant donné que la nouvelle version du modèle et l’ancienne coexistent, le nouveau jumeau peut utiliser l’une ou l’autre des versions.

Cela signifie également que le chargement d’une nouvelle version d’un modèle n’affecte pas automatiquement les jumeaux existants. Les jumeaux existants vont simplement rester des instances de l’ancienne version du modèle.

Vous pouvez mettre à jour ces jumeaux existants vers la nouvelle version du modèle à l’aide d’un correctif, comme décrit dans la section [*Mettre à jour le modèle d’un jumeau numérique*](how-to-manage-twin.md#update-a-digital-twins-model) de la rubrique *Guide pratique : Gestion des jumeaux numériques*. Dans le même correctif, vous devez mettre à jour à la fois l’**ID du modèle** (vers la nouvelle version) et **tous les champs qui nécessitent une modification sur le jumeau pour le rendre conforme au nouveau modèle**.

## <a name="remove-models"></a>Supprimer des modèles

Les modèles peuvent également être supprimés du service, de l’une des deux manières suivantes :
* **Désaffectation** : Une fois qu’un modèle est désactivé, vous ne pouvez plus l’utiliser pour créer de nouvelles représentations numériques. Les représentations numériques existantes utilisant déjà ce modèle ne sont pas affectées. vous pouvez donc toujours les mettre à jour avec des éléments tels que les modifications de propriétés et l’ajout ou la suppression de relations.
* **Suppression** : Cette opération supprime complètement le modèle de la solution. Les représentations qui utilisaient ce modèle ne sont plus associées à un modèle valide. Elles sont donc traitées comme si elles n’en avaient pas. Vous pouvez toujours lire ces représentations, mais vous ne pourrez pas les mettre à jour tant qu’elles ne seront pas réaffectées à un autre modèle.

Il s’agit de fonctionnalités distinctes qui ne s’affectent pas réciproquement, bien qu’elles puissent être utilisées ensemble pour supprimer un modèle progressivement. 

### <a name="decommissioning"></a>Désaffectation

Voici le code permettant de désaffecter un modèle :

```csharp
// 'client' is a valid DigitalTwinsClient  
client.DecommissionModel(dtmiOfPlanetInterface);
// Write some code that deletes or transitions digital twins
//...
```

L’état de désaffectation d’un modèle est inclus dans les enregistrements `ModelData` retournés par les API de récupération de modèle.

### <a name="deletion"></a>Suppression

Vous pouvez supprimer tous les modèles de votre instance d’une seule traite, ou vous pouvez le faire sur une base individuelle.

Pour obtenir un exemple de suppression de tous les modèles, téléchargez l’exemple d’application utilisé dans le didacticiel [ *: Explorer les bases avec un exemple d’application cliente*](tutorial-command-line-app.md). Le fichier *CommandLoop.cs* le fait dans une fonction `CommandDeleteAllModels`.

Le reste de cette section décompose la suppression du modèle plus en détails et montre comment le faire pour un modèle individuel.

#### <a name="before-deletion-deletion-requirements"></a>Avant la suppression : Conditions requises pour la suppression

En règle générale, les modèles peuvent être supprimés à tout moment.

L’exception concerne les modèles dont dépendent d’autres modèles, qu’il s’agisse d’une relation `extends` ou en tant que composant. Par exemple, si un modèle *ConferenceRoom* étend un modèle *Room*, et qu’il possède un modèle *ACUnit* en tant que composant, vous ne pouvez pas supprimer *Room* ou *ACUnit* avant que *ConferenceRoom* ne supprime leurs références respectives. 

Pour ce faire, vous pouvez mettre à jour le modèle dépendant pour supprimer les dépendances ou bien supprimer complètement le modèle dépendant.

#### <a name="during-deletion-deletion-process"></a>Durant la suppression : Processus de suppression

Même si un modèle répond aux exigences pour une suppression immédiate, vous voudrez peut-être suivre quelques étapes préliminaires pour éviter des conséquences inattendues pour la représentation numérique. Voici quelques étapes qui peuvent vous aider à gérer le processus :
1. Tout d’abord, désaffectez le modèle
2. Attendez quelques minutes, pour vous assurer que le service a bien traité toutes les demandes de création de représentations de dernière minute envoyées avant la désaffectation
3. Interrogez les représentations par modèle pour voir toutes les représentations utilisant le modèle désormais désactivé
4. Supprimez les représentations si vous n’en avez plus besoin ou bien liez-les à un nouveau modèle si nécessaire. Vous pouvez également choisir de les conserver seules, auquel cas elles deviennent des représentations sans modèle une celui-ci supprimé. Consultez la section suivante pour connaître les implications de cet état.
5. Patientez quelques minutes pour vous assurer que les modifications ont été répercutées
6. Supprimer le modèle 

Pour supprimer un modèle, utilisez cet appel :
```csharp
// 'client' is a valid DigitalTwinsClient
await client.DeleteModelAsync(IDToDelete);
```

#### <a name="after-deletion-twins-without-models"></a>Après la suppression : Représentations sans modèle

Une fois qu’un modèle est supprimé, les représentations numériques qui l’utilisaient sont désormais considérées comme sans modèle. Notez qu’il n’existe aucune requête vous fournissant une liste de tous les représentations dans cet état, bien que vous *puissiez* toujours interroger les représentations du modèle supprimé pour connaître les représentations affectées.

Voici une vue d’ensemble de ce que vous pouvez et ne pouvez pas faire avec des représentations sans modèle.

Choses que vous **pouvez** faire :
* Interroger la représentation
* Lire les propriétés
* Lire les relations sortantes
* Ajouter et supprimer des relations entrantes (d’autres représentations peuvent toujours former des relations *vers* cette représentation)
  - La `target` dans la définition de la relation peut toujours refléter le DTMI du modèle supprimé. Une relation sans cible définie peut également fonctionner ici.
* Supprimer des relations
* Supprimer la représentation

Choses que vous **ne pouvez pas** faire :
* Modifier les relations sortantes (les relations *de* cette représentation vers d’autres représentations)
* Modifier les propriétés

#### <a name="after-deletion-re-uploading-a-model"></a>Après la suppression : Rechargement d’un modèle

Une fois qu’un modèle a été supprimé, vous pouvez décider de charger un nouveau modèle ayant un ID identique à celui que vous avez supprimé. Voici ce qui se passe dans ce cas.
* Du point de vue du magasin de solutions, cela revient à charger un modèle entièrement nouveau. Le service ne se rappelle pas du chargement de l’ancien modèle.   
* S’il existe des représentations restantes dans le graphique référençant le modèle supprimé, elles ne sont plus orphelines ; Cet ID de modèle est de nouveau valide avec la nouvelle définition. Cependant, si la nouvelle définition pour le modèle est différente de celle du modèle supprimée, ces représentations peuvent disposer de propriétés et de relations correspondant à la définition supprimée et qui ne conviennent pas à la nouvelle.

Azure Digital Twins n’empêche pas cet état, veillez donc à lier les représentations correctement afin de veiller à ce qu’elles restent valides après le changement de définition de modèle.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer et gérer des représentations numériques basées sur vos modèles :
* [*Guide pratique : Gérer des jumeaux numériques*](how-to-manage-twin.md)