---
title: Topologies Apache Storm avec Visual Studio et C# - Azure HDInsight
description: Découvrez comment créer des topologies Storm en C#. Créez une topologie de comptage de mots dans Visual Studio à l’aide des outils Hadoop pour Visual Studio.
ROBOTS: NOINDEX
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/31/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 271f62625433a6651ba0e3230a62be51e5147f3e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89000190"
---
# <a name="develop-c-topologies-for-apache-storm-by-using-the-data-lake-tools-for-visual-studio"></a>Développer des topologies C# pour Apache Storm à l’aide de Data Lake Tools pour Visual Studio

Découvrez comment créer une topologie C# Apache Storm à l’aide des outils Azure Data Lake (Apache Hadoop) pour Visual Studio. Ce document vous guide dans le processus de création d’un projet Storm dans Visual Studio, de test local et de déploiement vers Apache Storm sur un cluster Azure HDInsight.

Vous apprendrez également à créer des topologies hybrides qui utilisent des composants Java et C#.

Les topologies C# utilisent .NET 4,5 et utilisent Mono pour s’exécuter sur le cluster HDInsight. Pour obtenir des informations sur les incompatibilités potentielles, consultez [Compatibilité Mono](https://www.mono-project.com/docs/about-mono/compatibility/). Pour utiliser une topologie C#, vous devez mettre à jour le package NuGet `Microsoft.SCP.Net.SDK` utilisé par votre projet vers la version 0.10.0.6 ou une version supérieure. La version du package doit également correspondre à la version principale de Storm installée sur HDInsight.

| Version de HDInsight | Version d’Apache Storm | Version de SCP.NET | Version Mono par défaut |
|:-----------------:|:-------------:|:---------------:|:--------------------:|
| 3.4 | 0.10.0.x | 0.10.0.x | 3.2.8 |
| 3,5 | 1.0.2.x | 1.0.0.x | 4.2.1 |
| 3.6 | 1.1.0.x | 1.0.0.x | 4.2.8 |

## <a name="prerequisite"></a>Configuration requise

Un cluster Apache Storm sur HDInsight. Consultez la section [Création de clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Storm** dans le champ **Type de cluster**.

## <a name="install-visual-studio"></a>Installation de Visual Studio

Vous pouvez développer des topologies C# avec SCP.NET en utilisant [Visual Studio](https://visualstudio.microsoft.com/downloads/). Les instructions fournies ici utilisent Visual Studio 2019, mais vous pouvez également utiliser des versions antérieures de Visual Studio.

## <a name="install-data-lake-tools-for-visual-studio"></a>Installer Data Lake Tools pour Visual Studio

Pour installer Data Lake Tools pour Visual Studio, procédez de la manière décrite dans [Prise en main de Data Lake Tools pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#install-data-lake-tools-for-visual-studio).

## <a name="install-java"></a>Installer Java

Lorsque vous soumettez une topologie Storm à partir de Visual Studio, SCP.NET génère un fichier zip contenant la topologie et les dépendances. Java est utilisé pour créer ces fichiers zip, car il utilise un format qui est plus compatible avec les clusters basés sur Linux.

1. Installez Java Development Kit (JDK) 7 ou version ultérieure dans votre environnement de développement. Vous pouvez obtenir le JDK Oracle à partir de la page [Oracle](https://openjdk.java.net/). Vous pouvez également utiliser d’[autres distributions Java](/java/azure/jdk/).

2. Définissez la variable d’environnement `JAVA_HOME` sur le répertoire contenant Java.

3. Définissez la variable d’environnement `PATH` afin d’inclure le répertoire `%JAVA_HOME%\bin`.

Pour vérifier que Java et le JDK sont correctement installés, vous pouvez générer et utiliser l’application de console C# suivante :

```csharp
using System;
using System.IO;
namespace ConsoleApplication2
{
   class Program
   {
       static void Main(string[] args)
       {
           string javaHome = Environment.GetEnvironmentVariable("JAVA_HOME");
           if (!string.IsNullOrEmpty(javaHome))
           {
               string jarExe = Path.Combine(javaHome + @"\bin", "jar.exe");
               if (File.Exists(jarExe))
               {
                   Console.WriteLine("JAVA Is Installed properly");
                    return;
               }
               else
               {
                   Console.WriteLine("A valid JAVA JDK is not found. Looks like JRE is installed instead of JDK.");
               }
           }
           else
           {
             Console.WriteLine("A valid JAVA JDK is not found. JAVA_HOME environment variable is not set.");
           }
       }  
   }
}
```

## <a name="apache-storm-templates"></a>Modèles Apache Storm

Data Lake Tools pour Visual Studio fournit les modèles suivants :

| Type de projet | Illustre le |
| --- | --- |
| Application Storm |Projet de topologie Storm vide |
| Exemple d’enregistreur SQL Azure Storm |Écriture dans Azure SQL Database. |
| Exemple de lecteur Azure Cosmos DB Storm |Lecture à partir d’Azure Cosmos DB |
| Exemple d’enregistreur Azure Cosmos DB Storm |Écriture dans Azure Cosmos DB |
| Exemple de lecteur EventHub Storm |Lecture à partir d’Azure Event Hubs |
| Exemple d’enregistreur EventHub Storm |Écriture dans Azure Event Hubs |
| Exemple de lecteur HBase Storm |Lecture à partir de HBase sur des clusters HDInsight |
| Exemple d’enregistreur HBase Storm |Écriture dans HBase sur des clusters HDInsight |
| Exemple Storm hybride |Utilisation d’un composant Java |
| Exemple Storm |Topologie de comptage de mots de base |

> [!WARNING]  
> Certains modèles ne fonctionnent pas avec HDInsight basé sur Linux. Les packages NuGet utilisés par les modèles peuvent ne pas être compatibles avec Mono. Pour identifier les problèmes potentiels, consultez le document sur la [compatibilité Mono](https://www.mono-project.com/docs/about-mono/compatibility/) et utilisez l’[Analyseur de portabilité .NET](../hdinsight-hadoop-migrate-dotnet-to-linux.md#automated-portability-analysis).

Dans les étapes de ce document, vous allez utiliser le type de projet Application Storm de base pour créer une topologie.

### <a name="apache-hbase-templates"></a>Modèles Apache HBase

Les modèles de lecteur et d’enregistreur HBase utilisent l’API REST HBase au lieu de l’API Java HBase pour communiquer avec un cluster HBase sur HDInsight.

### <a name="eventhub-templates"></a>Modèles EventHub

> [!IMPORTANT]  
> Il se peut que le composant spout EventHub basé sur Java inclus avec le modèle EventHub ne fonctionne pas avec Storm sur HDInsight version 3.5 ou ultérieure. Une version mise à jour de ce composant est disponible dans la page [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/HDI3.5/lib).

Pour un exemple de topologie utilisant ce composant et fonctionnant avec Storm sur HDInsight 3.5, voir [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

## <a name="create-a-c-topology"></a>Création d’une topologie C#

Pour créer un projet de topologie C# dans Visual Studio :

1. Ouvrez Visual Studio.

1. Dans la fenêtre **Démarrer**, sélectionnez **Créer un projet**.

1. Dans la fenêtre **Créer un nouveau projet**, faites défiler la liste jusqu’à **Application Storm**, puis sélectionnez **Suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, entrez le **nom de projet** *WordCount*, accédez au répertoire **Emplacement** ou créez-le pour le projet, puis sélectionnez **Créer**.

    ![Application Storm, boîte de dialogue Configurer votre nouveau projet, Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/apache-storm-new-project.png)

Une fois le projet créé, vous devez avoir les fichiers suivants :

* *Program.cs* : Définition de la topologie de votre projet. Par défaut, une topologie consistant en un seul spout et un seul bolt est créée.

* *Spout.cs* : exemple de spout émettant des nombres aléatoires.

* *Bolt.cs* : exemple de bolt qui effectue un suivi du décompte des nombres émis par le spout.

Lorsque vous créez le projet, NuGet télécharge le dernier [package SCP.NET](https://www.nuget.org/packages/Microsoft.SCP.Net.SDK/).

### <a name="implement-the-spout"></a>Implémenter le spout

Ensuite, ajoutez le code pour le spout, qui est utilisé pour lire les données d’une topologie à partir d’une source externe. Ce spout émet une phrase au hasard dans la topologie.

1. Ouvrez *Spout.cs*. Les principaux composants d’un spout sont :

   * `NextTuple`: appelé par Storm quand le spout est autorisé à émettre de nouveaux tuples.

   * `Ack` (topologie transactionnelle uniquement) : gère les accusés de réception démarrés par d’autres composants dans la topologie, pour les tuples envoyés à partir du spout. Un accusé de réception de tuple permet au spout de savoir qu’il a été correctement traité par les composants en aval.

   * `Fail` (topologie transactionnelle uniquement) : gère les tuples qui ne parviennent pas à traiter d’autres composants de la topologie. L’implémentation d’une méthode `Fail` permet d’émettre le tuple à nouveau pour qu’il puisse être traité une nouvelle fois.

2. Remplacez le contenu de la classe `Spout` par le texte suivant :

    ```csharp
    private Context ctx;
    private Random r = new Random();
    string[] sentences = new string[] {
        "the cow jumped over the moon",
        "an apple a day keeps the doctor away",
        "four score and seven years ago",
        "snow white and the seven dwarfs",
        "i am at two with nature"
    };

    public Spout(Context ctx)
    {
        // Set the instance context
        this.ctx = ctx;

        Context.Logger.Info("Generator constructor called");

        // Declare Output schema
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // The schema for the default output stream is
        // a tuple that contains a string field
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));
    }

    // Get an instance of the spout
    public static Spout Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Spout(ctx);
    }

    public void NextTuple(Dictionary<string, Object> parms)
    {
        Context.Logger.Info("NextTuple enter");
        // The sentence to be emitted
        string sentence;

        // Get a random sentence
        sentence = sentences[r.Next(0, sentences.Length - 1)];
        Context.Logger.Info("Emit: {0}", sentence);
        // Emit it
        this.ctx.Emit(new Values(sentence));

        Context.Logger.Info("NextTuple exit");
    }

    public void Ack(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }

    public void Fail(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }
    ```

### <a name="implement-the-bolts"></a>Implémentation des bolts

À présent, créez deux bolts Storm dans cet exemple :

1. Supprimez le fichier *Bolt.cs* du projet.

2. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Ajouter** > **Nouvel élément**. Dans la liste, sélectionnez **Bolt Storm**, entrez *Splitter.cs* comme nom. Dans le code du nouveau fichier, remplacez le nom de l’espace de noms par `WordCount`. Ensuite, répétez ce processus pour créer un deuxième bolt nommé *Counter.cs*.

   * *Splitter.cs* : implémente un bolt qui fractionne les phrases en mots et émet un nouveau flux de mots.

   * *Counter.cs* : implémente un bolt qui compte chaque mot et émet un nouveau flux de mots ainsi que le décompte de chaque mot.

     > [!NOTE]  
     > Bien que ces bolts lisent et écrivent dans des flux, vous pouvez également utiliser un bolt pour communiquer avec une base de données, un service, etc.

3. Ouvrez *Splitter.cs*. Il n’a qu’une seule méthode par défaut : `Execute`. La méthode `Execute` est appelée lorsque le bolt reçoit un tuple à traiter. Ici, vous pouvez lire et traiter des tuples entrants et émettre des tuples sortants.

4. Remplacez le contenu de la classe `Splitter` par le code suivant :

    ```csharp
    private Context ctx;

    // Constructor
    public Splitter(Context ctx)
    {
        Context.Logger.Info("Splitter constructor called");
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // Input contains a tuple with a string field (the sentence)
        inputSchema.Add("default", new List<Type>() { typeof(string) });
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // Outbound contains a tuple with a string field (the word)
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance of the bolt
    public static Splitter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Splitter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the sentence from the tuple
        string sentence = tuple.GetString(0);
        // Split at space characters
        foreach (string word in sentence.Split(' '))
        {
            Context.Logger.Info("Emit: {0}", word);
            //Emit each word
            this.ctx.Emit(new Values(word));
        }

        Context.Logger.Info("Execute exit");
    }
    ```

5. Ouvrez *Counter.cs*, puis remplacez le contenu de la classe par le code suivant :

    ```csharp
    private Context ctx;

    // Dictionary for holding words and counts
    private Dictionary<string, int> counts = new Dictionary<string, int>();

    // Constructor
    public Counter(Context ctx)
    {
        Context.Logger.Info("Counter constructor called");
        // Set instance context
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string field - the word
        inputSchema.Add("default", new List<Type>() { typeof(string) });

        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string and integer field - the word and the word count
        outputSchema.Add("default", new List<Type>() { typeof(string), typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance
    public static Counter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Counter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the word from the tuple
        string word = tuple.GetString(0);
        // Do we already have an entry for the word in the dictionary?
        // If no, create one with a count of 0
        int count = counts.ContainsKey(word) ? counts[word] : 0;
        // Increment the count
        count++;
        // Update the count in the dictionary
        counts[word] = count;

        Context.Logger.Info("Emit: {0}, count: {1}", word, count);
        // Emit the word and count information
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new List<SCPTuple> { tuple }, new Values(word, count));
        Context.Logger.Info("Execute exit");
    }
    ```

### <a name="define-the-topology"></a>Définition de la topologie

Les spouts et les bolts sont organisés dans un graphique, qui définit la circulation des données entre les composants. Dans cette topologie, le graphique est le suivant :

![Diagramme de disposition des composants spout et bolt, topologie Storm](./media/apache-storm-develop-csharp-visual-studio-topology/word-count-topology1.png)

Des phrases sont émises par le spout, puis distribuées à des instances du bolt Splitter. Le bolt de fractionnement fractionne les phrases en mots, qui sont distribués au bolt de décompte.

Comme l’instance Counter contient localement le nombre de mots, il convient de vérifier que des mots donnés sont transmis à la même instance de bolt Counter, afin de garantir que chaque instance suit un mot donné. Comme le bolt de fractionnement ne gère aucun état, il n’est pas important de savoir quelle instance du fractionnement reçoit quelle phrase.

Ouvrez *Program.cs*. La méthode importante ici est `GetTopologyBuilder`, qui est utilisée pour définir la topologie qui est envoyée à Storm. Remplacez le contenu de `GetTopologyBuilder` par le code suivant pour implémenter la topologie décrite précédemment :

```csharp
// Create a new topology named 'WordCount'
TopologyBuilder topologyBuilder = new TopologyBuilder(
    "WordCount" + DateTime.Now.ToString("yyyyMMddHHmmss"));

// Add the spout to the topology.
// Name the component 'sentences'
// Name the field that is emitted as 'sentence'
topologyBuilder.SetSpout(
    "sentences",
    Spout.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"sentence"}}
    },
    1);
// Add the splitter bolt to the topology.
// Name the component 'splitter'
// Name the field that is emitted 'word'
// Use suffleGrouping to distribute incoming tuples
//   from the 'sentences' spout across instances
//   of the splitter
topologyBuilder.SetBolt(
    "splitter",
    Splitter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word"}}
    },
    1).shuffleGrouping("sentences");

// Add the counter bolt to the topology.
// Name the component 'counter'
// Name the fields that are emitted 'word' and 'count'
// Use fieldsGrouping to ensure that tuples are routed
//   to counter instances based on the contents of field
//   position 0 (the word). This could also have been
//   List<string>(){"word"}.
//   This ensures that the word 'jumped', for example, will always
//   go to the same instance
topologyBuilder.SetBolt(
    "counter",
    Counter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word", "count"}}
    },
    1).fieldsGrouping("splitter", new List<int>() { 0 });

// Add topology config
topologyBuilder.SetTopologyConfig(new Dictionary<string, string>()
{
    {"topology.kryo.register","[\"[B\"]"}
});

return topologyBuilder;
```

## <a name="submit-the-topology"></a>Envoi de la topologie

Vous êtes maintenant prêt à envoyer la topologie à votre cluster HDInsight.

1. Accédez à **Affichage** > **Explorateur de serveurs**.

1. Cliquez avec le bouton droit sur **Azure**, sélectionnez **Se connecter à un abonnement Microsoft Azure...** , puis terminez le processus de connexion.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet et choisissez **Envoyer à Storm sur HDInsight**.

1. Dans la boîte de dialogue **Envoyer la topologie**, dans la liste déroulante **Cluster Storm**, choisissez votre cluster Storm sur HDInsight, puis sélectionnez **Envoyer**. Vous pouvez vérifier si l’envoi a été effectué correctement en examinant le volet **Sortie**.

    Une fois la topologie envoyée avec succès, la fenêtre **Vue des topologies Storm** du cluster doit apparaître. Choisissez la topologie **WordCount** dans la liste pour afficher des informations sur la topologie en cours d’exécution.

    ![Fenêtre Vue des topologies Storm, cluster HDInsight, Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/storm-topology-view.png)

    > [!NOTE]  
    > Vous pouvez également afficher les **Topologies Storm** dans l’**Explorateur de serveurs**. Développez **Azure** > **HDInsight**, cliquez avec le bouton droit sur un cluster Storm sur HDInsight, puis choisissez **Afficher les topologies Storm**.

    Pour afficher des informations sur les composants figurant dans la topologie, sélectionnez un composant dans le diagramme.

1. Dans la section **Résumé de la topologie**, sélectionnez **Supprimer** pour arrêter la topologie.

    > [!NOTE]  
    > Les topologies Storm poursuivent leur exécution jusqu'à ce qu'elles soient désactivées ou que le cluster soit supprimé.

## <a name="transactional-topology"></a>Topologie transactionnelle

La précédente ci-dessus est non transactionnelle. Les composants de la topologie n’implémentent aucune fonctionnalité de relecture des messages. Pour un exemple de topologie transactionnelle, créez un projet, puis sélectionnez **Exemple Storm** comme type de projet.

Les topologies transactionnelles implémentent les opérations suivantes pour prendre en charge la relecture des données :

* **Mise en cache des métadonnées** : le spout doit stocker les métadonnées relatives aux données émises, afin que les données puissent être récupérées et émises à nouveau en cas de défaillance. Dans la mesure où les données émises par l’exemple sont petites, les données brutes de chaque tuple sont stockées dans un dictionnaire pour la relecture.

* **Ack** : chaque bolt de la topologie peut appeler `this.ctx.Ack(tuple)` afin de signaler qu’il a traité un tuple avec succès. Quand tous les bolts ont accusé réception du tuple, la méthode `Ack` du spout est appelée. La méthode `Ack` permet au spout de supprimer les données mises en cache pour la relecture.

* **Fail** : chaque bolt peut appeler `this.ctx.Fail(tuple)` pour indiquer que le traitement d’un tuple a échoué. L’échec se propage à la méthode `Fail` du spout, où le tuple peut être relu à l’aide des métadonnées mises en cache.

* **ID de séquence** : lors de l’émission d’un tuple, un ID de séquence unique peut être spécifié. Cette valeur identifie le tuple pour le traitement de la relecture (`Ack` et `Fail`). Par exemple, le spout dans le projet **Exemple Storm** utilise l’appel de méthode suivant lors de l’émission de données :

  `this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), lastSeqId);`

  Ce code émet un tuple contenant une phrase dans le flux par défaut, avec la valeur d’ID de séquence contenue dans `lastSeqId`. Dans cet exemple, `lastSeqId` est incrémenté pour chaque tuple émis.

Comme illustré dans le projet **Exemple Storm** , un composant peut être défini comme transactionnel ou non lors de l’exécution, en fonction de la configuration.

## <a name="hybrid-topology-with-c-and-java"></a>Topologie hybride avec C# et Java

Vous pouvez également utiliser Data Lake Tools pour Visual Studio pour créer des topologies hybrides, où certains composants sont en C# et d’autres en Java.

Pour un exemple de topologie hybride, créez un projet, puis sélectionnez **Exemple Storm hybride**. Cet exemple de type illustre les concepts suivants :

* **Spout Java** et **bolt C#**  : Défini dans la classe `HybridTopology_javaSpout_csharpBolt`.

  Une version transactionnelle est définie dans la classe `HybridTopologyTx_javaSpout_csharpBolt`.

* **Spout C#** et **bolt Java** : Défini dans la classe `HybridTopology_csharpSpout_javaBolt`.

  Une version transactionnelle est définie dans la classe `HybridTopologyTx_csharpSpout_javaBolt`.

  > [!NOTE]  
  > Cette version montre également comment utiliser le code Clojure à partir d’un fichier texte en tant que composant Java.

Pour changer la topologie utilisée au moment d’envoyer le projet, déplacez l’instruction `[Active(true)]` vers la topologie que vous souhaitez utiliser avant de l’envoyer au cluster.

> [!NOTE]  
> Tous les fichiers Java requis sont fournis, dans le cadre de ce projet dans le dossier *JavaDependency* .

Lors de la création et de la soumission d’une topologie hybride, considérez les aspects suivants :

* Utilisez `JavaComponentConstructor` pour créer une instance de la classe Java pour un spout ou un bolt.

* Utilisez `microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer` pour sérialiser les données à l’intérieur ou à l’extérieur des composants Java, des objets Java vers JSON.

* Lors de l’envoi de la topologie au serveur, vous devez utiliser l’option **Configurations supplémentaires** pour spécifier les **chemins d’accès des fichiers Java**. Le chemin spécifié doit correspondre au répertoire qui détient les fichiers JAR contenant vos classes Java.

### <a name="azure-event-hubs"></a>Hubs d'événements Azure

SCP.NET 0.9.4.203 introduit une classe et une méthode inédites propres à l’utilisation du spout Event Hubs (spout Java qui lit à partir d’Event Hubs). Lorsque vous créez une topologie qui utilise un spout Event Hub (par exemple, à l’aide du modèle **Exemple de lecteur EventHub Storm**), utilisez les API suivantes :

* Classe `EventHubSpoutConfig` : crée un objet qui contient la configuration du composant de spout.

* Méthode `TopologyBuilder.SetEventHubSpout` : ajoute le composant de spout Event Hub à la topologie.

> [!NOTE]  
> Vous devez toujours utiliser `CustomizedInteropJSONSerializer` pour sérialiser les données produites par le spout.

## <a name="use-configurationmanager"></a>Utiliser ConfigurationManager

N’utilisez pas **ConfigurationManager** pour récupérer des valeurs de configuration de composants bolt et spout. Cela est susceptible d’entraîner une exception de pointeur null. À la place, transmettez la configuration de votre projet dans la topologie Storm en tant que paire clé/valeur dans le contexte de la topologie. Chaque composant qui repose sur des valeurs de configuration doit les récupérer à partir du contexte lors de l’initialisation.

Le code suivant montre comment récupérer ces valeurs :

```csharp
public class MyComponent : ISCPBolt
{
    // To hold configuration information loaded from context
    Configuration configuration;
    ...
    public MyComponent(Context ctx, Dictionary<string, Object> parms)
    {
        // Save a copy of the context for this component instance
        this.ctx = ctx;
        // If it exists, load the configuration for the component
        if(parms.ContainsKey(Constants.USER_CONFIG))
        {
            this.configuration = parms[Constants.USER_CONFIG] 
                as System.Configuration.Configuration;
        }
        // Retrieve the value of "Foo" from configuration
        var foo = this.configuration.AppSettings.Settings["Foo"].Value;
    }
    ...
}
```

Si vous utilisez une méthode `Get` pour retourner une instance de votre composant, vous devez vous assurer qu’il transmet les paramètres `Context` et `Dictionary<string, Object>` du constructeur. L’exemple suivant est une méthode `Get` de base qui transmet correctement ces valeurs :

```csharp
public static MyComponent Get(Context ctx, Dictionary<string, Object> parms)
{
    return new MyComponent(ctx, parms);
}
```

## <a name="how-to-update-scpnet"></a>Mise à jour de SCP.NET

Les dernières versions de SCP.NET prennent en charge la mise à niveau du package via NuGet. Vous recevez une notification de mise à niveau dès qu’une nouvelle mise à jour est disponible. Pour rechercher manuellement une mise à niveau, procédez comme suit :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Gérer les packages NuGet**.

2. Dans le Gestionnaire de package, sélectionnez **Mises à jour**. Si une mise à jour du package de support SCP.NET est disponible, elle est répertoriée. Sélectionnez **Mettre à jour** pour le package, puis, dans la boîte de dialogue **Aperçu des modifications**, sélectionnez **OK** pour l’installer.

> [!IMPORTANT]  
> Si votre projet a été créé avec une version précédente de SCP.NET qui n’utilisait pas NuGet, vous devez effectuer les étapes suivantes pour procéder à la mise à jour vers une version plus récente :
>
> 1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Gérer les packages NuGet**.
> 2. Dans le champ **Rechercher**, recherchez `Microsoft.SCP.Net.SDK`, puis ajoutez-le au projet.

## <a name="troubleshoot-common-issues-with-topologies"></a>Dépanner des problèmes courants avec les topologies

### <a name="null-pointer-exceptions"></a>Exceptions de pointeur null

Lorsque vous utilisez une topologie C# avec un cluster HDInsight basé sur Linux, les composants bolt et spout qui utilisent **ConfigurationManager** pour lire les paramètres de configuration au moment de l’exécution peuvent retourner des exceptions de pointeur null.

La configuration de votre projet est transmise dans la topologie Storm en tant que paire clé/valeur dans le contexte de la topologie. Elle peut être récupérée à partir de l’objet dictionnaire qui est transmis à vos composants quand ils sont initialisés.

Pour plus d’informations, consultez la section [Utiliser ConfigurationManager](#use-configurationmanager) de ce document.

### <a name="systemtypeloadexception"></a>System.TypeLoadException

Lorsque vous utilisez une topologie C# avec un cluster HDInsight basé sur Linux, vous pouvez rencontrer l’erreur suivante :

`System.TypeLoadException: Failure has occurred while loading a type.`

Cette erreur se produit quand vous utilisez un fichier binaire non compatible avec la version de .NET prise en charge par Mono.

Pour les clusters HDInsight basés sur Linux, vous devez vous assurer que votre projet utilise des fichiers binaires compilés pour .NET 4.5.

### <a name="test-a-topology-locally"></a>Test local d’une topologie

Il est facile de déployer une topologie sur un cluster, mais il est parfois nécessaire de tester localement une topologie. Utilisez les étapes suivantes pour exécuter et tester localement l’exemple de topologie de cet article localement dans votre environnement de développement.

> [!WARNING]  
> Le test local fonctionne uniquement pour les topologies exclusivement C# de base. Vous ne pouvez pas employer le test local pour les topologies hybrides ou celles qui utilisent plusieurs flux de données.

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet et sélectionnez **Propriétés**. Dans les propriétés du projet, définissez alors le **type de sortie** sur **Application Console**.

   ![Application Storm HDInsight, propriétés du projet, Type de sortie](./media/apache-storm-develop-csharp-visual-studio-topology/hdi-output-type-window.png)

   > [!NOTE]
   > N’oubliez pas de modifier le **Type de sortie** sur **Bibliothèque de classes** avant de déployer la topologie sur un cluster.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Ajouter** > **Nouvel élément**. Sélectionnez **Classe**, puis entrez *LocalTest.cs* en tant que nom de classe. Enfin, sélectionnez **Ajouter**.

1. Ouvrez *LocalTest.cs* et ajoutez l’instruction `using` suivante au début du fichier :

    ```csharp
    using Microsoft.SCP;
    ```

1. Utilisez le code suivant comme contenu de la classe `LocalTest` :

    ```csharp
    // Drives the topology components
    public void RunTestCase()
    {
        // An empty dictionary for use when creating components
        Dictionary<string, Object> emptyDictionary = new Dictionary<string, object>();

        #region Test the spout
        {
            Console.WriteLine("Starting spout");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext spoutCtx = LocalContext.Get();
            // Get a new instance of the spout, using the local context
            Spout sentences = Spout.Get(spoutCtx, emptyDictionary);

            // Emit 10 tuples
            for (int i = 0; i < 10; i++)
            {
                sentences.NextTuple(emptyDictionary);
            }
            // Use LocalContext to persist the data stream to file
            spoutCtx.WriteMsgQueueToFile("sentences.txt");
            Console.WriteLine("Spout finished");
        }
        #endregion

        #region Test the splitter bolt
        {
            Console.WriteLine("Starting splitter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext splitterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Splitter splitter = Splitter.Get(splitterCtx, emptyDictionary);

            // Set the data stream to the data created by the spout
            splitterCtx.ReadFromFileToMsgQueue("sentences.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = splitterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                splitter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            splitterCtx.WriteMsgQueueToFile("splitter.txt");
            Console.WriteLine("Splitter bolt finished");
        }
        #endregion

        #region Test the counter bolt
        {
            Console.WriteLine("Starting counter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext counterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Counter counter = Counter.Get(counterCtx, emptyDictionary);

            // Set the data stream to the data created by splitter bolt
            counterCtx.ReadFromFileToMsgQueue("splitter.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = counterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                counter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            counterCtx.WriteMsgQueueToFile("counter.txt");
            Console.WriteLine("Counter bolt finished");
        }
        #endregion
    }
    ```

    Prenez un moment pour lire les commentaires du code. Ce code utilise `LocalContext` pour exécuter les composants dans l’environnement de développement. Il conserve le flux de données entre les composants dans des fichiers texte sur le lecteur local.

1. Ouvrez *Program.cs* et ajoutez le code suivant à la méthode `Main` :

    ```csharp
    Console.WriteLine("Starting tests");
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "WordCount-LocalTest");
    // Initialize the runtime
    SCPRuntime.Initialize();

    //If we are not running under the local context, throw an error
    if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
    {
        throw new Exception(string.Format("unexpected pluginType: {0}", Context.pluginType));
    }
    // Create test instance
    LocalTest tests = new LocalTest();
    // Run tests
    tests.RunTestCase();
    Console.WriteLine("Tests finished");
    Console.ReadKey();
    ```

1. Enregistrez les modifications, puis sélectionnez **F5** ou choisissez **Débogage** > **Démarrer le débogage** pour démarrer le projet. Une fenêtre de console doit apparaître et enregistrer les statuts avec la progression des tests. Lorsque `Tests finished` apparaît, sélectionnez une touche quelconque pour fermer la fenêtre.

1. Utilisez l’**Explorateur Windows** pour localiser le répertoire contenant votre projet. (Par exemple : *C:\\Users\\\<your_user_name>\\source\\repos\\WordCount\\WordCount*.) Ensuite, dans ce répertoire, ouvrez *Bin*, puis sélectionnez *Débogage*. Vous devez voir les fichiers texte qui ont été produits lors de l’exécution de tests : *sentences.txt*, *counter.txt* et *splitter.txt*. Ouvrez chaque fichier texte et inspectez les données.

   > [!NOTE]  
   > Les chaînes de données sont conservées sous forme de tableau de valeurs décimales dans ces fichiers. Par exemple, `[[97,103,111]]` dans le fichier **splitter.txt** correspond au mot *ago*.

> [!NOTE]  
> Veillez à définir **Type de projet** sur **Bibliothèque de classes** dans les propriétés du projet avant le déploiement vers un cluster Storm sur HDInsight.

### <a name="log-information"></a>Enregistrement d’informations

Vous pouvez facilement enregistrer des informations à partir de vos composants de topologie à l’aide de `Context.Logger`. Par exemple, la commande suivante crée une entrée de journal d’information :

`Context.Logger.Info("Component started");`

Les informations enregistrées peuvent être affichées dans le **journal du service Hadoop**, qui se trouve dans **l’Explorateur de serveurs**. Développez l’entrée pour votre Storm sur le cluster HDInsight , puis le **journal du service Hadoop**. Enfin, sélectionnez le fichier journal à afficher.

> [!NOTE]  
> Les journaux d’activité sont stockés dans le compte de stockage Azure utilisé par votre cluster. Pour afficher les journaux d’activité dans Visual Studio, vous devez vous connecter à l’abonnement Azure contenant le compte de stockage.

### <a name="view-error-information"></a>Afficher les informations relatives aux erreurs

Pour afficher les erreurs qui se sont produites dans une topologie en cours d’exécution, procédez comme suit :

1. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur le cluster Storm sur HDInsight et sélectionnez **Afficher les topologies Storm**.

   Pour les **Spouts** et **Bolts**, la colonne **Dernière erreur** contient des informations sur la dernière erreur.

2. Sélectionnez **l’Id de spout** ou **l’Id de bolt** pour le composant ayant une erreur répertoriée. La page de détails affiche d’autres informations d’erreur dans la section **Erreurs**, en bas de page.

3. Pour plus d’informations, sélectionnez un **Port** dans la section **Exécuteurs** de la page afin d’afficher le journal de travail Storm relatif aux dernières minutes.

### <a name="errors-submitting-topologies"></a>Erreurs lors de la soumission des topologies

Si vous rencontrez des erreurs lors de la soumission d’une topologie à HDInsight, vous trouverez des journaux pour les composants côté serveur qui gèrent la soumission des topologies sur votre cluster HDInsight. Pour télécharger ces journaux, utilisez la commande suivante à partir d’une ligne de commande :

```cmd
scp sshuser@clustername-ssh.azurehdinsight.net:/var/log/hdinsight-scpwebapi/hdinsight-scpwebapi.out .
```

Remplacez *sshuser* par le compte d’utilisateur SSH du cluster. Remplacez *clustername* par le nom du cluster HDInsight. Pour en savoir plus sur l’utilisation de `scp` et de `ssh` avec HDInsight, voir [Utilisation de SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

Les envois peuvent échouer pour plusieurs raisons :

* Le JDK n’est pas installé ou ne figure pas sur ce chemin.
* Les dépendances Java requises ne sont pas incluses dans l’envoi.
* Les dépendances sont incompatibles.
* Les noms de topologie sont dupliqués.

Si le fichier journal *hdinsight-scpwebapi.out* contient un `FileNotFoundException`, l’exception peut être provoquée par les conditions suivantes :

* Le JDK ne figure pas sur le chemin dans l’environnement de développement. Vérifiez que le JDK est installé dans l’environnement de développement et que `%JAVA_HOME%/bin`figure dans le chemin d’accès.
* Il vous manque une dépendance Java. Veillez à inclure tous les fichiers .jar requis dans l’envoi.

## <a name="next-steps"></a>Étapes suivantes

Pour un exemple de traitement des données à partir des Event Hubs, voir [Traiter des événements d’Azure Event Hubs avec Storm sur HDInsight](apache-storm-develop-csharp-event-hub-topology.md).

Pour obtenir un exemple de topologie C# qui fractionne les données de flux en plusieurs flux de données, consultez la rubrique [Exemple c# Storm](https://github.com/Blackmist/csharp-storm-example).

Pour plus d’informations sur la création de topologies de C#, voir la page [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/SCPNet-GettingStarted.md).

Pour plus d’informations sur l’utilisation de HDInsight et pour obtenir davantage d’exemples Storm sur HDinsight, consultez les documents suivants :

**Microsoft SCP.NET**

* [Guide de programmation SCP pour Apache Storm dans Azure HDInsight](apache-storm-scp-programming-guide.md)

**Apache Storm sur HDInsight**

* [Déploiement et gestion des topologies Apache Storm sur Azure HDInsight](apache-storm-deploy-monitor-topology-linux.md)
* [Exemples de topologies Apache Storm dans Azure HDInsight](apache-storm-example-topology.md)

**Apache Hadoop sur HDInsight**

* [Présentation d’Apache Hive et HiveQL sur Azure HDInsight](../hadoop/hdinsight-use-hive.md)
* [Utiliser MapReduce dans Apache Hadoop sur HDInsight](../hadoop/hdinsight-use-mapreduce.md)

**Apache HBase sur HDInsight**

* [Utiliser Apache HBase dans Azure HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md)
