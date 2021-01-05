---
title: Développer des fonctions .NET Standard pour des travaux Azure Stream Analytics (préversion)
description: Découvrez comment écrire des fonctions C# définies par l’utilisateur pour des travaux Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/10/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 9cf929a3a6f5b3752b030f449b3b24b2bdc941a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90907304"
---
# <a name="develop-net-standard-user-defined-functions-for-azure-stream-analytics-jobs-preview"></a>Développer des fonctions .NET Standard définies par l’utilisateur pour des travaux Azure Stream Analytics (préversion)

Azure Stream Analytics offre un langage de requête de type SQL pour effectuer des transformations et des calculs sur des flux de données d’événement. De nombreuses fonctions sont intégrées, mais certains scénarios complexes nécessitent davantage de flexibilité. Avec les fonctions .NET Standard définies par l’utilisateur, vous pouvez appeler vos propres fonctions écrites dans n’importe quel langage .NET Standard (C#, F#, etc.) pour étendre le langage de requête Stream Analytics. Les fonctions définies par l’utilisateur vous permettent d’effectuer des calculs mathématiques complexes, d’importer des modèles Machine Learning personnalisés à l’aide de ML.NET et d’utiliser une logique personnalisée d’imputation pour les données manquantes. La fonctionnalité de fonction définie par l’utilisateur pour les travaux Stream Analytics est actuellement disponible en préversion et ne doit pas être utilisée dans des charges de travail de production.

La fonction .NET définie par l’utilisateur pour les travaux cloud est disponible dans les régions suivantes :
* Centre-USA Ouest
* Europe Nord
* USA Est
* USA Ouest
* USA Est 2
* Europe Ouest

Si vous êtes intéressé par l’utilisation de cette fonctionnalité dans une autre région, vous pouvez [demander l’accès](https://aka.ms/ccodereqregion).

## <a name="package-path"></a>Chemin du package

Le format d’un package de fonction définie par l’utilisateur a le chemin `/UserCustomCode/CLR/*`. Les bibliothèques de liens dynamiques (DLL) et les ressources sont copiées sous le dossier `/UserCustomCode/CLR/*`, qui permet d’isoler les DLL utilisateur du système et les DLL Azure Stream Analytics. Ce chemin du package est utilisé pour toutes les fonctions, quelle que soit la méthode d’utilisation choisie.

## <a name="supported-types-and-mapping"></a>Types pris en charge et mappage

Pour que les valeurs Azure Stream Analytics puissent être utilisées en C#, elles doivent être marshalées d’un environnement à l’autre. Le marshaling se produit pour tous les paramètres en entrée d’une fonction définie par l’utilisateur. Chaque type Azure Stream Analytics a un type correspondant en C#, indiqué dans le tableau ci-dessous :

|**Type Azure Stream Analytics** |**Type C#** |
|---------|---------|
|bigint | long |
|float | double |
|nvarchar(max) | string |
|DATETIME | DateTime |
|Enregistrement | Dictionnaire\<string, object> |
|Array | Object[] |

Il en va de même lorsque les données doivent être marshalées de C# vers Azure Stream Analytics, ce qui se produit sur la valeur de sortie d’une fonction définie par l’utilisateur. Le tableau ci-dessous montre les types pris en charge :

|**Type C#**  |**Type Azure Stream Analytics**  |
|---------|---------|
|long  |  bigint   |
|double  |  float   |
|string  |  nvarchar(max)   |
|DateTime  |  dateTime   |
|struct  |  Enregistrement   |
|object  |  Enregistrement   |
|Object[]  |  Array   |
|Dictionnaire\<string, object>  |  Enregistrement   |

## <a name="develop-a-udf-in-visual-studio-code"></a>Développez une UDF dans Visual Studio Code

[Les outils Visual Studio Code pour Azure Stream Analytics](quick-create-visual-studio-code.md) vous facilite l’écriture de fonctions définies par l’utilisateur, le test de vos travaux localement (même en mode hors connexion) et la publication de votre travail Stream Analytics dans Azure.

Il existe deux façons d’implémenter des fonctions définies par l’utilisateur .NET Standard dans des outils Visual Studio Code.

* UDF des bibliothèques de liens dynamiques locales
* Fonction définie par l’utilisateur à partir d’un projet local

### <a name="local-project"></a>Projet local

Les fonctions définies par l’utilisateur peuvent être écrites dans un assembly référencé plus tard dans une requête Azure Stream Analytics. Il s’agit de l’option recommandée pour les fonctions complexes qui nécessitent toute la puissance d’un langage .NET Standard au-delà de son langage d’expression, comme la logique procédurale ou la récursivité. Les fonctions définies par l’utilisateur à partir d’un projet local peuvent également être utilisées si vous avez besoin de partager la logique de la fonction entre plusieurs requêtes Azure Stream Analytics. L’ajout de fonctions définies par l’utilisateur à votre projet local vous donne la possibilité de déboguer et de tester vos fonctions localement.

Pour référencer un projet local :

1. Créez une nouvelle bibliothèque de classes .NET Standard sur votre ordinateur local.
2. Écrivez le code dans votre classe. Toutes les classes doivent être définies comme étant *publiques* et tous les objets comme étant *publics statiques*.
3. Ajoutez un nouveau fichier de configuration de fonction CSharp dans votre projet Azure Stream Analytics, et référencez le projet de bibliothèque de classes CSharp.
4. Configurez le chemin de l’assembly dans le fichier de configuration du travail `JobConfig.json`, section **CustomCodeStorage**. Cette étape n’est pas nécessaire pour les tests locaux.

### <a name="local-dlls"></a>Bibliothèques de liens dynamiques locales

Vous pouvez également référencer des bibliothèques de liens dynamiques locales qui incluent les fonctions définies par l’utilisateur.

### <a name="example"></a>Exemple

Dans cet exemple, **CSharpUDFProject** est un projet de bibliothèque de classes C# et **ASAUDFDemo** est le projet Azure Stream Analytics qui référence **CSharpUDFProject**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-demo.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

La fonction définie par l’utilisateur suivante a une fonction qui multiplie un entier par lui-même pour produire le carré de l’entier. Les classes doivent être définies en tant que classes *publiques* et les objets doivent être définis en tant qu’objets *publics statiques*.

```csharp
using System;

namespace CSharpUDFProject
{
    // 
    public class Class1
    {
        public static Int64 SquareFunction(Int64 a)
        {
            return a * a;
        }
    }
}
```

Les étapes suivantes vous montrent comment ajouter la fonction UDF C# à votre projet Stream Analytics.

1. Cliquez avec le bouton droit sur le dossier **Fonctions** et choisissez **Ajouter un élément**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

2. Ajoutez une fonction C# **SquareFunction** à votre projet Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-2.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-add-function-name.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

3. Dans la configuration de la fonction C#, sélectionnez **Sélectionner le chemin d’accès au projet de bibliothèque** pour choisir votre projet C# dans la liste déroulante et sélectionnez **Projet build** pour générer votre projet. Choisissez ensuite **Sélectionner une classe** et **Sélectionner la méthode** pour sélectionner le nom de la classe et de la méthode associée dans la liste déroulante. Pour référencer les méthodes, types et fonctions dans la requête Stream Analytics, les classes doivent être définies comme étant *publiques* et les objets comme étant *publics statiques*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-project.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

    Si vous souhaitez utiliser C# UDF à partir d’une DLL, sélectionnez **Choisir le chemin d’accès à la DLL de la bibliothèque** pour choisir la DLL. Choisissez ensuite **Sélectionner une classe** et **Sélectionner la méthode** pour sélectionner le nom de la classe et de la méthode associée dans la liste déroulante.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-choose-dll.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

4. Appelez l’UDF dans votre requête Azure Stream Analytics.

   ```sql
    SELECT price, udf.SquareFunction(price)
    INTO Output
    FROM Input 
   ```

5. Avant de soumettre la tâche à Azure, configurez le chemin d’accès au package dans le fichier de configuration du travail, `JobConfig.json`, section **CustomCodeStorage**. Utilisez **Sélectionner votre abonnement** dans CodeLens pour choisir votre abonnement et choisissez le compte de stockage et le nom du conteneur dans la liste déroulante. Laissez **Chemin d’accès** par défaut. Cette étape n’est pas nécessaire pour les tests locaux.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/visual-studio-code-csharp-udf-configure-storage-account.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

## <a name="develop-a-udf-in-visual-studio"></a>Développez une UDF dans Visual Studio Code

Il existe trois façons d’implémenter dans des outils Visual Studio.

* Fichiers code-behind dans un projet ASA
* Fonction définie par l’utilisateur à partir d’un projet local
* Package existant à partir d’un compte de stockage Azure

### <a name="codebehind"></a>CodeBehind

Vous pouvez écrire des fonctions définies par l’utilisateur dans le fichier code-behind **Script.asql**. Les outils Visual Studio compilent automatiquement le fichier code-behind dans un fichier d’assembly. Les assemblys sont packagés dans un fichier zip et chargés dans votre compte de stockage quand vous envoyez votre travail à Azure. Vous pouvez apprendre à écrire une fonction C# définie par l’utilisateur à l’aide de d’un fichier code-behind en suivant le tutoriel [Fonction C# définie par l’utilisateur pour les travaux de périphérie Stream Analytics](stream-analytics-edge-csharp-udf.md). 

### <a name="local-project"></a>Projet local

Pour référencer un projet local dans Visual Studio :

1. Créez une nouvelle bibliothèque de classes .NET Standard dans votre solution
2. Écrivez le code dans votre classe. Toutes les classes doivent être définies comme étant *publiques* et tous les objets comme étant *publics statiques*. 
3. Générez votre projet. Les outils créent un fichier zip contenant tous les artefacts du dossier bin, puis chargent ce fichier zip dans le compte de stockage. Pour les références externes, utilisez la référence d’assembly au lieu du package NuGet.
4. Référencez la nouvelle classe dans votre projet Azure Stream Analytics.
5. Ajoutez une nouvelle fonction dans votre projet Azure Stream Analytics.
6. Configurez le chemin d’assembly dans le fichier de configuration du travail, `JobConfig.json`. Définissez le chemin d’assembly sur **Référence de projet locale ou code-behind**.
7. Regénérez le projet de fonction et le projet Azure Stream Analytics.  

### <a name="example"></a>Exemple

Dans cet exemple, **UDFTest** est un projet de bibliothèque de classes C# et **ASAUDFDemo** est le projet Azure Stream Analytics qui référence **UDFTest**.

:::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-demo.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

1. Générez votre projet C# pour pouvoir ajouter une référence à votre fonction C# définie par l’utilisateur à partir de la requête Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-build-project.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

2. Ajoutez la référence au projet C# dans le projet ASA. Cliquez avec le bouton droit sur le nœud Références et choisissez Ajouter une référence.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-reference.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

3. Choisissez le nom du projet C# dans la liste.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-choose-project-name.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

4. Vous devez voir **UDFTest** listé sous **Références** dans **l’Explorateur de solutions**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-added-reference.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

5. Cliquez avec le bouton droit sur le dossier **Fonctions** et choisissez **Nouvel élément**.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

6. Ajoutez une fonction C# **SquareFunction.json** à votre projet Azure Stream Analytics.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-add-csharp-function-2.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

7. Double-cliquez sur la fonction dans **l’Explorateur de solutions** pour ouvrir la boîte de dialogue de configuration.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-csharp-function-config.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

8. Dans la configuration de fonction C#, choisissez **Charger à partir d’une référence de projet ASA** ainsi que les noms des assemblys, classes et méthodes associés dans la liste déroulante. Pour référencer les méthodes, types et fonctions dans la requête Stream Analytics, les classes doivent être définies comme étant *publiques* et les objets comme étant *publics statiques*.

   :::image type="content" source="media/stream-analytics-edge-csharp-udf-methods/stream-analytics-edge-udf-asa-csharp-function-config.png" alt-text="Projet Azure Stream Analytics dans Visual Studio Code":::

## <a name="existing-packages"></a>Packages existants

Vous pouvez créer des fonctions définies par l’utilisateur .NET Standard dans l’IDE de votre choix et les appeler à partir de votre requête Azure Stream Analytics. Commencez par compiler votre code et packager toutes les DLL. Le format du package a le chemin `/UserCustomCode/CLR/*`. Ensuite, chargez `UserCustomCode.zip` à la racine du conteneur dans votre compte de stockage Azure.

Une fois les packages zip d’assemblys chargés dans votre compte de stockage Azure, vous pouvez utiliser les fonctions dans des requêtes Azure Stream Analytics. Il vous suffit d’inclure les informations de stockage dans la configuration du travail Stream Analytics. Vous ne pouvez pas tester la fonction localement à l’aide de cette option, car les outils Visual Studio ne téléchargent pas votre package. Le chemin du package est analysé directement par le service. 

Pour configurer le chemin d’assembly dans le fichier de configuration du travail, `JobConfig.json` :

Développez la section **Configuration du code défini par l’utilisateur** et remplissez la configuration avec les valeurs suggérées suivantes :

   |**Paramètre**|**Valeur suggérée**|
   |-------|---------------|
   |Ressource des paramètres de stockage généraux|Choisir une source de données du compte actuel|
   |Abonnement des paramètres de stockage généraux| < votre abonnement >|
   |Compte de stockage des paramètres de stockage généraux| < votre compte de stockage >|
   |Ressource des paramètres de stockage de code personnalisé|Choisir une source de données du compte actuel|
   |Compte de stockage des paramètres de stockage de code personnalisé|< votre compte de stockage >|
   |Conteneur des paramètres de stockage de code personnalisé|< votre conteneur de stockage >|
   |Source de l’assembly de code personnalisé|Packages d’assembly existants du cloud|
   |Source de l’assembly de code personnalisé|UserCustomCode.zip|

## <a name="user-logging"></a>Journalisation utilisateur

Le mécanisme de journalisation vous permet de capturer des informations personnalisées pendant l’exécution d’un travail. Vous pouvez utiliser les données de journal pour déboguer ou évaluer l’exactitude du code personnalisé en temps réel.

La classe `StreamingContext` vous permet de publier des informations de diagnostic à l’aide de la fonction `StreamingDiagnostics.WriteError`. Le code ci-dessous montre l’interface exposée par Azure Stream Analytics.

```csharp
public abstract class StreamingContext
{
    public abstract StreamingDiagnostics Diagnostics { get; }
}

public abstract class StreamingDiagnostics
{
    public abstract void WriteError(string briefMessage, string detailedMessage);
}
```

`StreamingContext` est transmis à la méthode UDF en tant que paramètre d’entrée et peut être utilisé dans l’UDF pour publier des informations de journal personnalisées. Dans l’exemple ci-dessous, `MyUdfMethod` définit une entrée de **données**, fournie par la requête, et un **contexte** d’entrée en tant que `StreamingContext`, fourni par le moteur d’exécution. 

```csharp
public static long MyUdfMethod(long data, StreamingContext context)
{
    // write log
    context.Diagnostics.WriteError("User Log", "This is a log message");
    
    return data;
}
```

La valeur `StreamingContext` n’a pas besoin d’être transmise par la requête SQL. Azure Stream Analytics fournit automatiquement un objet de contexte si un paramètre d’entrée est présent. L’utilisation de `MyUdfMethod` ne change pas, comme illustré dans la requête suivante :

```sql
SELECT udf.MyUdfMethod(input.value) as udfValue FROM input
```

Vous pouvez accéder aux messages du journal par le biais des [journaux de diagnostic](data-errors.md).

## <a name="limitations"></a>Limites

La préversion des fonctions définies par l’utilisateur a les limitations suivantes :

* Les fonctions définies par l’utilisateur .NET Standard peuvent uniquement être créées dans Visual Studio Code ou Visual Studio et publiées sur Azure. Les versions en lecture seule des fonctions définies par l’utilisateur .NET Standard peuvent être consultées sous **Fonctions** dans le portail Azure. La création de fonctions .NET Standard n’est pas prise en charge dans le portail Azure.

* L’éditeur de requêtes du portail Azure affiche une erreur quand vous utilisez des fonctions définies par l’utilisateur .NET Standard dans le portail. 

* Comme le code personnalisé partage le contexte du moteur Azure Stream Analytics, il ne peut pas référencer un élément qui a un espace de noms/nom_dll en conflit avec le code Azure Stream Analytics. Par exemple, vous ne pouvez pas référencer *Newtonsoft Json*.

* Les fichiers de support inclus dans le projet sont copiés dans le fichier zip UserCustomCode utilisé lors de la publication du travail dans le cloud. Tous les fichiers des sous-dossiers sont copiés directement à la racine du dossier UserCustomCode dans le cloud lorsqu’ils sont décompressés. Le fichier zip est « aplati » lors de la décompression.

* UserCustomCode ne gère pas les dossiers vides. N’ajoutez pas de dossiers vides aux fichiers de support dans le projet.

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Écrire une fonction C# définie par l’utilisateur pour un travail Azure Stream Analytics (préversion)](stream-analytics-edge-csharp-udf.md)
* [Tutoriel : Fonctions JavaScript définies par l’utilisateur Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)