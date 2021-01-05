---
title: Guide de programmation SCP.NET pour Storm dans Azure HDInsight
description: Découvrez comment utiliser SCP.NET pour créer des topologies Storm basées sur .NET en vue d’une utilisation avec Storm dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/13/2020
ms.openlocfilehash: d54a06c457451fc5323ae37b34b53411cdd6abda
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89000139"
---
# <a name="scp-programming-guide-for-apache-storm-in-azure-hdinsight"></a>Guide de programmation SCP pour Apache Storm dans Azure HDInsight

SCP est une plateforme permettant de développer des applications de traitement de données en temps réel, fiables, cohérentes et aux performances élevées. Elle est basée sur [Apache Storm](https://storm.incubator.apache.org/), un système de traitement par flux conçu par des communautés de logiciels open source. Storm a été créé par Nathan Marz. Il a été publié en open source par Twitter. Storm exploite [Apache ZooKeeper](https://zookeeper.apache.org/), un autre projet Apache qui permet une coordination distribuée et une gestion d’état très fiables.

Le projet SCP a non seulement migré Storm sur Windows, mais également des extensions et une personnalisation ajoutées au projet pour l’environnement Windows. Les extensions incluent l’expérience de développement .NET et les bibliothèques .NET. La personnalisation comprend un déploiement basé sur Windows.

Grâce aux extensions et à la personnalisation, vous n’avez pas besoin de dupliquer (fork) les projets de logiciels open source. Vous pouvez utiliser des environnements dérivés basés sur Storm.

## <a name="processing-model"></a>Modèle de traitement

Les données de SCP sont modélisées en tant que flux de tuples continus. En règle générale, les tuples :

1. se déversent dans une file d’attente ;
1. sont récupérés et transformés par la logique métier hébergée dans une topologie Storm ;
1. ont leur sortie redirigée en tant que tuples vers un autre système SCP ou sont validés dans des magasins tels que des systèmes de fichiers distribués et des bases de données comme SQL Server.

![Diagramme d’une file d’attente fournissant des données à traiter, puis destinées à un magasin de données](./media/apache-storm-scp-programming-guide/queue-feeding-data-to-processing-to-data-store.png)

Dans Storm, une topologie d’application définit un graphique de calcul. Chaque nœud d’une topologie contient une logique de traitement. Les liens entre les nœuds indiquent le flux de données.

Les nœuds qui injectent des données d’entrée dans la topologie sont appelés _Spouts_. Vous pouvez les utiliser pour séquencer les données. Les données d’entrée peuvent provenir d’une source telle que des fichiers journaux, une base de données transactionnelle ou un compteur de performances système.

Les nœuds qui ont à la fois des flux de données d’entrée et de sortie sont appelés _Bolts_. Ils effectuent le filtrage, la sélection et l’agrégation des données.

SCP prend en charge les types de traitement de données Meilleur effort, Une fois au minimum et Exactement une fois.

Dans une application de traitement de flux distribués, des erreurs peuvent se produire pendant le traitement des données. Il peut s’agir d’une panne du réseau, d’une défaillance de l’ordinateur ou d’une erreur dans votre code. Le type de traitement Une fois au minimum garantit que toutes les données sont traitées au moins une fois en relisant automatiquement les mêmes données lorsqu’une erreur se produit.

Le type de traitement Une fois au minimum est simple et fiable, et il convient à de nombreuses applications. Cependant, lorsqu’une application requiert un comptage exact, le traitement Une fois au minimum n’est pas suffisant, car les mêmes données peuvent être relues dans la topologie de l’application. Dans ce cas, le type de traitement Exactement une fois garantit que le résultat est correct, même quand les données sont relues et traitées plusieurs fois.

SCP permet aux développeurs .NET de créer des applications de traitement des données en temps réel tout en utilisant une Machine virtuelle Java (JVM) avec Storm. Une JVM et .NET communiquent via des sockets locaux TCP. Chaque Spout/Bolt constitue une paire de processus .NET/Java, où la logique utilisateur s’exécute dans un processus .NET en tant que plug-in.

Pour générer une application de traitement de données sur SCP, procédez comme suit :

1. Concevez et implémentez des Spouts pour extraire des données de files d’attente.
1. Concevez et implémentez des Bolts pour traiter les données d’entrée, puis enregistrez-les dans des magasins externes tels qu’une base de données.
1. Concevez la topologie, puis validez-la et exécutez-la.

La topologie définit les vertex et les flux de données entre ces vertex. SCP récupère une spécification de la topologie et la déploie dans un cluster Storm, où chaque vertex est exécuté sur un nœud logique. Le planificateur de tâches Storm prend en charge le basculement et la mise à l’échelle.

Cet article présente quelques exemples simples pour illustrer la création d’applications de traitement des données avec SCP.

## <a name="scp-plug-in-interface"></a>Interface de plug-in SCP

Les plug-ins SCP sont des applications autonomes. Ils peuvent s’exécuter au sein de Visual Studio pendant le développement et être connectées au pipeline Storm après le déploiement de production.

L’écriture d’un plug-in SCP est identique à celle de n’importe quelle autre application console Windows. La plateforme SCP.NET déclare des interfaces pour le Spout/Bolt. Votre code de plug-in implémente ces interfaces. L’objectif principal de cette conception est de vous permettre de vous concentrer sur votre logique métier tout en permettant à la plateforme SCP.NET de gérer d’autres choses.

Votre code de plug-in implémente l’une des interfaces suivantes. L’interface varie selon que la topologie est transactionnelle ou non transactionnelle et selon que le composant est un Spout ou un Bolt.

* **ISCPSpout**
* **ISCPBolt**
* **ISCPTxSpout**
* **ISCPBatchBolt**

### <a name="iscpplugin"></a>ISCPPlugin

**ISCPPlugin** est l’interface commune pour de nombreux plug-ins. Actuellement, c’est une interface factice.

```csharp
public interface ISCPPlugin
{
}
```

### <a name="iscpspout"></a>ISCPSpout

**ISCPSpout** est l’interface d’un Spout non transactionnel.

```csharp
public interface ISCPSpout : ISCPPlugin
{
    void NextTuple(Dictionary<string, Object> parms);
    void Ack(long seqId, Dictionary<string, Object> parms); 
    void Fail(long seqId, Dictionary<string, Object> parms);
}
```

Lorsque **NextTuple** est appelé, votre code C# peut émettre un ou plusieurs tuples. S’il n’y a rien à émettre, cette méthode doit retourner sans émettre quoi que ce soit.

Les méthodes **NextTuple**, **Ack** et **Fail** sont toutes appelées dans une boucle courte dans le thread unique d’un processus C#. Lorsqu’il n’y a aucun tuple à émettre, **NextTuple** se met en veille pendant un court laps de temps, par exemple 10 millisecondes. Cette mise en veille permet d’éviter de gaspiller la disponibilité de l’UC.

Les méthodes **Ack** et **Fail** sont appelées uniquement lorsqu’un fichier de spécification active le mécanisme d’accusé de réception. Le paramètre *seqId* identifie le tuple qui est reconnu ou qui a échoué. Si l’accusé de réception est activé dans une topologie non transactionnelle, la fonction **Emit** suivante doit être utilisée dans un Spout :

```csharp
public abstract void Emit(string streamId, List<object> values, long seqId);
```

Si une topologie non transactionnelle ne prend pas en charge l’accusé de réception, **Ack** et **Fail** peuvent être laissées comme fonctions vides.

Le paramètre d’entrée *parms* de ces fonctions spécifie un dictionnaire vide et est réservé à un usage ultérieur.

### <a name="iscpbolt"></a>ISCPBolt

**ISCPBolt** est l’interface d’un Bolt non transactionnel.

```csharp
public interface ISCPBolt : ISCPPlugin
{
void Execute(SCPTuple tuple);
}
```

Quand un nouveau tuple est disponible, la fonction **Execute** est appelée pour le traiter.

### <a name="iscptxspout"></a>ISCPTxSpout

**ISCPTxSpout** est l’interface d’un Spout transactionnel.

```csharp
public interface ISCPTxSpout : ISCPPlugin
{
    void NextTx(out long seqId, Dictionary<string, Object> parms);  
    void Ack(long seqId, Dictionary<string, Object> parms);         
    void Fail(long seqId, Dictionary<string, Object> parms);        
}
```

Tout comme leurs équivalents non transactionnels, les fonctions **NextTx**, **Ack** et **Fail** sont toutes appelées dans une boucle courte dans le thread unique d’un processus C#. Lorsqu’il n’y a aucun tuple à émettre, **NextTx** se met en veille pendant un court laps de temps, par exemple 10 millisecondes. Cette mise en veille permet d’éviter de gaspiller la disponibilité de l’UC.

Lorsque **NextTx** est appelée pour démarrer une nouvelle transaction, le paramètre de sortie *seqId* identifie la transaction. La transaction est également utilisée dans **Ack** et **Fail**. Votre méthode **NextTx** peut émettre des données du côté Java. Les données sont stockées dans ZooKeeper pour prendre en charge la relecture. Étant donné que ZooKeeper a une capacité limitée, votre code doit émettre uniquement des métadonnées, et non des données en bloc dans un Spout transactionnel.

Étant donné que Storm relit automatiquement une transaction non réussie, **Fail** n’est généralement pas appelée. Mais si SCP peut vérifier les métadonnées émises par un Spout transactionnel, il peut appeler la fonction **Fail** quand les métadonnées ne sont pas valides.

Le paramètre d’entrée *parms* de ces fonctions spécifie un dictionnaire vide et est réservé à un usage ultérieur.

### <a name="iscpbatchbolt"></a>ISCPBatchBolt

**ISCPBatchBolt** est l’interface d’un Bolt transactionnel.

```csharp
public interface ISCPBatchBolt : ISCPPlugin
{
    void Execute(SCPTuple tuple);
    void FinishBatch(Dictionary<string, Object> parms);  
}
```

La méthode **Execute** est appelée lorsqu’un nouveau tuple arrive au Bolt. La méthode **FinishBatch** est appelée lorsque cette transaction se termine. Le paramètre d’entrée *parms* est réservé à un usage ultérieur.

Pour une topologie transactionnelle, **StormTxAttempt** est une classe importante. Elle a deux membres : **TxId** et **AttemptId**. Le membre **TxId** identifie une transaction spécifique. Une transaction peut être tentée plusieurs fois si elle échoue et est relue.

SCP.NET crée un objet **ISCPBatchBolt** pour traiter chaque objet **StormTxAttempt**, tout comme Storm le fait dans Java. L’objectif de cette conception est de prendre en charge le traitement des transactions parallèles. Une fois qu’une tentative de transaction est terminée, l’objet **ISCPBatchBolt** correspondant est détruit et nettoyé de la mémoire.

## <a name="object-model"></a>Modèle objet

SCP.NET fournit également un ensemble d'objets clés pour les développeurs. Il s’agit des objets **Context**, **StateStore** et **SCPRuntime**. Ils sont présentés dans cette section.

### <a name="context"></a>Context

L’objet **Context** fournit un environnement d’exécution à une application. Chaque instance **ISCPPlugin** de **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** ou **ISCPBatchBolt** dispose d’une instance **Context** correspondante. La fonctionnalité fournie par **Context** peut être divisée en deux parties :

* Partie statique, qui est disponible dans l’ensemble du processus C#
* Partie dynamique, qui est disponible uniquement pour l’instance **Context** spécifique

### <a name="static-part"></a>Partie statique

```csharp
public static ILogger Logger = null;
public static SCPPluginType pluginType;
public static Config Config { get; set; }
public static TopologyContext TopologyContext { get; set; }  
```

L’objet **Logger** est fourni à des fins de journalisation.

L’objet **pluginType** indique le type de plug-in du processus C#. Si le processus est exécuté dans un mode de test local sans Java, le type de plug-in est **SCP_NET_LOCAL**.

```csharp
public enum SCPPluginType 
{
    SCP_NET_LOCAL = 0,
    SCP_NET_SPOUT = 1,
    SCP_NET_BOLT = 2,
    SCP_NET_TX_SPOUT = 3,
    SCP_NET_BATCH_BOLT = 4  
    }
```

La propriété **Config** obtient des paramètres de configuration provenant du côté Java, qui les transmet lorsqu’un plug-in C# est initialisé. Les paramètres **Config** sont divisés en deux parties : **stormConf** et **pluginConf**.

```csharp
public Dictionary<string, Object> stormConf { get; set; }  
public Dictionary<string, Object> pluginConf { get; set; }  
```

La partie **stormConf** correspond à des paramètres définis par Storm, et la partie **pluginConf** correspond à des paramètres définis par SCP. Voici un exemple :

```csharp
public class Constants
{
    … …

    // constant string for pluginConf
    public static readonly String NONTRANSACTIONAL_ENABLE_ACK = "nontransactional.ack.enabled";  

    // constant string for stormConf
    public static readonly String STORM_ZOOKEEPER_SERVERS = "storm.zookeeper.servers";
    public static readonly String STORM_ZOOKEEPER_PORT = "storm.zookeeper.port";
}
```

Le type **TopologyContext** obtient le contexte de la topologie. Il est surtout utile pour les composants parallèles multiples. Voici un exemple :

```csharp
//demo how to get TopologyContext info
if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
{
    Context.Logger.Info("TopologyContext info:");
    TopologyContext topologyContext = Context.TopologyContext;
    Context.Logger.Info("taskId: {0}", topologyContext.GetThisTaskId());
    taskIndex = topologyContext.GetThisTaskIndex();
    Context.Logger.Info("taskIndex: {0}", taskIndex);
    string componentId = topologyContext.GetThisComponentId();
    Context.Logger.Info("componentId: {0}", componentId);
    List<int> componentTasks = topologyContext.GetComponentTasks(componentId);  
    Context.Logger.Info("taskNum: {0}", componentTasks.Count);
}
```

### <a name="dynamic-part"></a>Partie dynamique

Les interfaces suivantes sont pertinentes pour une certaine instance **Context**, qui est créée par la plateforme SCP.NET et transmise à votre code :

```csharp
// Declare the Output and Input Stream Schemas

public void DeclareComponentSchema(ComponentStreamSchema schema);

// Emit tuple to default stream.
public abstract void Emit(List<object> values);

// Emit tuple to the specific stream.
public abstract void Emit(string streamId, List<object> values);  
```

Pour un Spout non transactionnel qui prend en charge l’accusé de réception, la méthode suivante est fournie :

```csharp
// for nontransactional spout that supports ack
public abstract void Emit(string streamId, List<object> values, long seqId);  
```

Un Bolt non transactionnel qui prend en charge l’accusé de réception doit appeler explicitement **Ack** ou **Fail** avec le tuple qu’il a reçu. Lors de l’émission d’un nouveau tuple, le Bolt doit également spécifier les ancres du tuple. Les méthodes suivantes sont fournies :

```csharp
public abstract void Emit(string streamId, IEnumerable<SCPTuple> anchors, List<object> values);
public abstract void Ack(SCPTuple tuple);
public abstract void Fail(SCPTuple tuple);
```

### <a name="statestore"></a>StateStore

L’objet **StateStore** fournit des services de métadonnées, une génération de séquence unitone et une coordination sans attente. Vous pouvez générer des abstractions de concurrence distribuées de niveau supérieur sur **StateStore**. Ces abstractions incluent les verrous distribués, les files d’attente distribuées, les barrières et les services de transaction.

Les applications SCP peuvent utiliser l’objet **State** pour sérialiser des informations dans [Apache ZooKeeper](https://zookeeper.apache.org/). Cette fonctionnalité est particulièrement utile pour une topologie transactionnelle. Si un Spout transactionnel ne répond plus et redémarre, **State** peut récupérer les informations nécessaires à partir de ZooKeeper, puis redémarrer le pipeline.

L’objet **StateStore** utilisent les méthodes principales suivantes :

```csharp
/// <summary>
/// Static method to retrieve a state store of the given path and connStr 
/// </summary>
/// <param name="storePath">StateStore path</param>
/// <param name="connStr">StateStore address</param>
/// <returns>Instance of StateStore</returns>
public static StateStore Get(string storePath, string connStr);

/// <summary>
/// Create a new state object in this state store instance
/// </summary>
/// <returns>State from StateStore</returns>
public State Create();

/// <summary>
/// Retrieve all states that were previously uncommitted, excluding all exited states
/// </summary>
/// <returns>Uncommitted states</returns>
public IEnumerable<State> GetUnCommitted();

/// <summary>
/// Get all the states in the StateStore
/// </summary>
/// <returns>All the states</returns>
public IEnumerable<State> States();

/// <summary>
/// Get state or registry object
/// </summary>
/// <param name="info">Registry name (registry only)</param>
/// <typeparam name="T">Type, registry or state</typeparam>
/// <returns>Return registry or state</returns>
public T Get<T>(string info = null);

/// <summary>
/// List all the committed states
/// </summary>
/// <returns>Registries containing the committed state </returns>
public IEnumerable<Registry> Committed();

/// <summary>
/// List all the exited states in the StateStore
/// </summary>
/// <returns>Registries containing the exited states</returns>
public IEnumerable<Registry> Aborted();

/// <summary>
/// Retrieve an existing state object from this state store instance 
/// </summary>
/// <returns>State from StateStore</returns>
/// <typeparam name="T">stateId, id of the State</typeparam>
public State GetState(long stateId)
```

L’objet **State** utilisent les méthodes principales suivantes :

```csharp
/// <summary>
/// Set the status of the state object to commit
/// </summary>
public void Commit(bool simpleMode = true);

/// <summary>
/// Set the status of the state object to exit
/// </summary>
public void Abort();

/// <summary>
/// Put an attribute value under the given key
/// </summary>
/// <param name="key">Key</param>
/// <param name="attribute">State attribute</param>
    public void PutAttribute<T>(string key, T attribute);

/// <summary>
/// Get the attribute value associated with the given key
/// </summary>
/// <param name="key">Key</param>
/// <returns>State attribute</returns>
    public T GetAttribute<T>(string key);
```

Quand **simpleMode** est défini sur **true**, la méthode **Commit** supprime le ZNode correspondant dans ZooKeeper. Dans le cas contraire, la méthode supprime le ZNode actuel et ajoute un nouveau nœud dans COMMITTED\_PATH.

### <a name="scpruntime"></a>SCPRuntime

La classe **SCPRuntime** fournit les deux méthodes suivantes :

```csharp
public static void Initialize();

public static void LaunchPlugin(newSCPPlugin createDelegate);  
```

La méthode **Initialize** initialise l’environnement du runtime SCP. Dans cette méthode, le processus C# se connecte au côté Java pour recevoir les paramètres de configuration ainsi que le contexte de la topologie.

La méthode **LaunchPlugin** démarre la boucle de traitement du message. Dans cette boucle, le plug-in C# reçoit des messages provenant du côté Java. Ces messages incluent des tuples et des signaux de contrôle. Le plug-in traite ensuite les messages, par exemple en appelant la méthode d’interface fournie par votre code.

Le paramètre d’entrée pour **LaunchPlugin** est un délégué. La méthode peut retourner un objet qui implémente l’interface **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** ou **ISCPBatchBolt**.

```csharp
public delegate ISCPPlugin newSCPPlugin(Context ctx, Dictionary<string, Object> parms);
```

Pour **ISCPBatchBolt**, vous pouvez obtenir un objet **StormTxAttempt** à partir du paramètre *parms* et l’utiliser pour juger si la tentative est une tentative relue. La vérification d’une tentative de relecture est souvent effectuée sur le Bolt de validation. L’exemple HelloWorldTx, plus loin dans cet article, illustre cette vérification.

Les plug-ins SCP peuvent généralement s’exécuter selon deux modes : le mode de test local et le mode normal.

#### <a name="local-test-mode"></a>Mode de test local

Dans ce mode, les plug-ins SCP de votre code C# s’exécutent dans Visual Studio durant la phase de développement. Vous pouvez utiliser l’interface **ILocalContext** dans ce mode. L’interface fournit des méthodes pour sérialiser les tuples émis vers des fichiers locaux, puis les relire dans la RAM.

```csharp
public interface ILocalContext
{
    List<SCPTuple> RecvFromMsgQueue();
    void WriteMsgQueueToFile(string filepath, bool append = false);  
    void ReadFromFileToMsgQueue(string filepath);
}
```

#### <a name="regular-mode"></a>Mode normal

Dans ce mode, le processus Java Storm exécute les plug-ins SCP. Voici un exemple :

```csharp
namespace Scp.App.HelloWorld
{
public class Generator : ISCPSpout
{
    … …
    public static Generator Get(Context ctx, Dictionary<string, Object> parms)
    {
    return new Generator(ctx);
    }
}

class HelloWorld
{
    static void Main(string[] args)
    {
    /* Setting the environment variable here can change the log file name */
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "HelloWorld");

    SCPRuntime.Initialize();
    SCPRuntime.LaunchPlugin(new newSCPPlugin(Generator.Get));
    }
}
}
```

## <a name="topology-specification-language"></a>Langage de spécification de topologie

La spécification de topologie SCP est un langage spécifique à un domaine (DSL) pour décrire et configurer les topologies SCP. Il est basé sur [Clojure DSL de Storm](https://storm.incubator.apache.org/documentation/Clojure-DSL.html) et est étendu par SCP.

Vous pouvez envoyer des spécifications de topologie directement vers un cluster Storm pour les exécuter via la commande **runSpec**.

SCP.NET a ajouté les fonctions suivantes pour définir les topologies transactionnelles :

| Nouvelle fonction | Paramètres | Description |
| --- | --- | --- |
| **tx-topolopy** |*topology-name*<br />*spout-map*<br />*bolt-map* |Définit une topologie transactionnelle avec un nom de topologie, une carte de définition de Spouts et une carte de définition de Bolts. |
| **scp-tx-spout** |*exec-name*<br />*args*<br />*fields* |Définit un Spout transactionnel. La fonction exécute l’application spécifiée par *exec-name* et utilise *args*.<br /><br />Le paramètre *fields* spécifie les champs de sortie pour le Spout. |
| **scp-tx-batch-bolt** |*exec-name*<br />*args*<br />*fields* |Définit un Bolt de traitement transactionnel. La fonction exécute l’application spécifiée par *exec-name* et utilise *args*.<br /><br />Le paramètre *fields* spécifie les champs de sortie pour le Bolt. |
| **scp-tx-commit-bolt** |*exec-name*<br />*args*<br />*fields* |Définit un Bolt de validation transactionnel. La fonction exécute l’application spécifiée par *exec-name* et utilise *args*.<br /><br />Le paramètre *fields* spécifie les champs de sortie pour le Bolt. |
| **nontx-topology** |*topology-name*<br />*spout-map*<br />*bolt-map* |Définit une topologie non transactionnelle avec un nom de topologie, une carte de définition de Spouts et une carte de définition de Bolts. |
| **scp-spout** |*exec-name*<br />*args*<br />*fields*<br />*parameters* |Définit un Spout non transactionnel. La fonction exécute l’application spécifiée par *exec-name* et utilise *args*.<br /><br />Le paramètre *fields* spécifie les champs de sortie pour le Spout.<br /><br />Le paramètre *parameters* est facultatif. Utilisez-le pour spécifier des paramètres tels que « nontransactional.ack.enabled ». |
| **scp-bolt** |*exec-name*<br />*args*<br />*fields*<br />*parameters* |Définit un Bolt non transactionnel. La fonction exécute l’application spécifiée par *exec-name* et utilise *args*.<br /><br />Le paramètre *fields* spécifie les champs de sortie pour le Bolt.<br /><br />Le paramètre *parameters* est facultatif. Utilisez-le pour spécifier des paramètres tels que « nontransactional.ack.enabled ». |

SCP.NET définit les mots clés suivants :

| Mot clé | Description |
| --- | --- |
| **:name** |Nom de la topologie |
| **:topology** |Topologie utilisant les fonctions du tableau précédent et des fonctions intégrées |
| **:p** |Indicateur de parallélisme pour chaque Spout ou Bolt |
| **:config** |Indique s’il faut configurer les paramètres ou les mettre à jour |
| **:schema** |Schéma du flux |

SCP.NET définit également ces paramètres fréquemment utilisés :

| Paramètre | Description |
| --- | --- |
| « plugin.name » |Nom de fichier .exe du plug-in C# |
| « plugin.args » |Arguments du plug-in |
| « output.schema » |Schéma de sortie |
| « nontransactional.ack.enabled » |Indique si l’accusé de réception est activé pour la topologie non transactionnelle |

La commande **runSpec** est déployée avec les bits. Voici l’utilisation de la commande :

```csharp
.\bin\runSpec.cmd
usage: runSpec [spec-file target-dir [resource-dir] [-cp classpath]]
ex: runSpec examples\HelloWorld\HelloWorld.spec specs examples\HelloWorld\Target
```

Le paramètre *resource-dir* est facultatif. Spécifiez-le lorsque vous souhaitez intégrer une application C#. Le répertoire spécifié contient l’application, les dépendances et les configurations.

Le paramètre *classpath* est également facultatif. Il permet de spécifier le classpath Java si le fichier de spécification contient un Spout ou un Bolt Java.

## <a name="miscellaneous-features"></a>Fonctionnalités diverses

### <a name="input-and-output-schema-declarations"></a>Déclaration de schéma d’entrée et de sortie

Vos processus C# peuvent émettre des tuples. Pour ce faire, la plateforme sérialise les tuples dans des objets **byte[]** et transfère les objets du côté Java. Storm transfère ensuite ces tuples aux cibles.

Dans les composants en aval, les processus C# reçoivent des tuples en retour du côté Java et les convertissent aux types d’origine de la plateforme. Toutes ces opérations sont masquées par la plateforme.

Pour prendre en charge la sérialisation et la désérialisation, votre code doit déclarer le schéma d’entrée et de sortie. Le schéma est défini en tant que dictionnaire. L’ID du flux est la clé du dictionnaire. La valeur de clé correspond aux types des colonnes. Un composant peut déclarer plusieurs flux.

```csharp
public class ComponentStreamSchema
{
    public Dictionary<string, List<Type>> InputStreamSchema { get; set; }
    public Dictionary<string, List<Type>> OutputStreamSchema { get; set; }
    public ComponentStreamSchema(Dictionary<string, List<Type>> input, Dictionary<string, List<Type>> output)
    {
        InputStreamSchema = input;
        OutputStreamSchema = output;
    }
}
```

La fonction suivante est ajoutée à un objet **Context** :

```csharp
public void DeclareComponentSchema(ComponentStreamSchema schema)
```

Les développeurs doivent s’assurer que les tuples émis obéissent au schéma défini pour un flux. Dans le cas contraire, le système lèvera une exception de runtime.

### <a name="multistream-support"></a>Prise en charge du multiflux

SCP permet à votre code d’émettre ou de recevoir plusieurs flux distincts en même temps. L’objet **Context** reflète cette prise en charge par le paramètre ID de flux facultatif de la méthode **Emit**.

Deux méthodes ont été ajoutées dans l’objet **Context** de SCP.NET. Elles émettent un ou plusieurs tuples vers des flux spécifiques. Le paramètre *streamId* est une chaîne. Sa valeur doit être la même dans le code C# et la spécification de définition de la topologie.

```csharp
/* Emit tuple to the specific stream. */
public abstract void Emit(string streamId, List<object> values);

/* for nontransactional spout only */
public abstract void Emit(string streamId, List<object> values, long seqId);
```

L’émission vers un flux inexistant entraîne des exceptions de runtime.

### <a name="fields-grouping"></a>Regroupement de champs

Le regroupement de champs intégré à Storm ne fonctionne pas correctement dans SCP.NET. Du côté du proxy Java, le type de données de tous les champs est en réalité **byte[]** . Le regroupement de champs utilise le code de hachage de l’objet **byte[]** pour effectuer le regroupement. Le code de hachage correspond à l’adresse de cet objet dans la RAM. Le regroupement sera donc incorrect pour les objets multioctets partageant le même contenu, mais pas la même adresse.

SCP.NET ajoute une méthode de regroupement personnalisée et utilise le contenu de l’objet **byte[]** pour procéder au regroupement. Dans un fichier de spécification, la syntaxe ressemble à l’exemple suivant :

```csharp
(bolt-spec
    {
        "spout_test" (scp-field-group :non-tx [0,1])
    }
    …
)
```

Dans le fichier de spécification précédent :

* `scp-field-group` spécifie que le regroupement est un regroupement de champs personnalisé implémenté par SCP.
* `:tx` ou `:non-tx` spécifie si la topologie est transactionnelle. Vous avez besoin de ces informations, car l’index de départ est différent selon qu’il s’agit de topologies transactionnelles ou non transactionnelles.
* `[0,1]` spécifie un ensemble haché d’ID de champ qui commencent par zéro.

### <a name="hybrid-topology"></a>Topologie hybride

Le code Storm natif est écrit en Java. SCP.NET a amélioré Storm pour vous permettre d’écrire du code C# afin de gérer votre logique métier. Toutefois, SCP.NET prend également en charge les topologies hybrides, qui contiennent non seulement des Spouts et des Bolts C#, mais également des Spouts et des Bolts Java.

### <a name="specify-java-spoutbolt-in-a-specification-file"></a>Spécifier le Spout/Bolt Java dans un fichier de spécification

Vous pouvez utiliser **scp-spout** et **scp-bolt** dans un fichier de spécification pour spécifier les Spouts et les Bolts Java. Voici un exemple :

```csharp
(spout-spec 
  (microsoft.scp.example.HybridTopology.Generator.)
  :p 1)
```

Ici, `microsoft.scp.example.HybridTopology.Generator` est le nom de la classe Spout Java.

### <a name="specify-the-java-classpath-in-a-runspec-command"></a>Spécifier le classpath Java dans une commande runSpec

Si vous souhaitez envoyer la topologie qui contient des Spouts ou des Bolts Java, commencez par les compiler pour produire des fichiers JAR. Spécifiez ensuite le classpath Java qui contient les fichiers JAR au moment de l’envoi de la topologie. Voici un exemple :

```csharp
bin\runSpec.cmd examples\HybridTopology\HybridTopology.spec specs examples\HybridTopology\net\Target -cp examples\HybridTopology\java\target\*
```

Ici, `examples\HybridTopology\java\target\` est le dossier contenant le fichier JAR des Spouts/Bolts Java.

### <a name="serialization-and-deserialization-between-java-and-c"></a>Sérialisation et désérialisation entre Java et C#

Un composant SCP comprend le côté Java et le côté C#. Pour interagir avec des Spouts/Bolts Java natifs, la sérialisation et la désérialisation doivent intervenir entre le côté Java et le côté C#, comme l’illustre le graphique suivant :

![Diagramme d’un composant Java qui envoie au composant SCP, qui envoie ensuite à un autre composant Java](./media/apache-storm-scp-programming-guide/java-compent-sending-to-scp-component-sending-to-java-component.png)

#### <a name="serialization-in-the-java-side-and-deserialization-in-the-c-side"></a>Sérialisation côté Java et désérialisation côté C#

Tout d’abord, fournissez l’implémentation par défaut pour la sérialisation côté Java et la désérialisation côté C#.

Spécifiez la méthode de sérialisation du côté Java dans un fichier de spécification.

```csharp
(scp-bolt
    {
        "plugin.name" "HybridTopology.exe"
        "plugin.args" ["displayer"]
        "output.schema" {}
        "customized.java.serializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer"]
    })
```

Spécifiez la méthode de désérialisation du côté C# dans votre code C#.

```csharp
Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
inputSchema.Add("default", new List<Type>() { typeof(Person) });
this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());
```  

Si le type de données n’est pas trop complexe, cette implémentation par défaut peut gérer la plupart des cas. Voici les cas où vous pouvez intégrer votre propre implémentation :

* Votre type de données est trop complexe pour l’implémentation par défaut.
* Les performances de votre implémentation par défaut ne répondent pas à vos besoins.

L’interface de sérialisation côté Java est définie comme suit :

```csharp
public interface ICustomizedInteropJavaSerializer {
    public void prepare(String[] args);
    public List<ByteBuffer> serialize(List<Object> objectList);
}
```

L’interface de désérialisation côté C# est définie comme suit :

```csharp
public interface ICustomizedInteropCSharpDeserializer
{
    List<Object> Deserialize(List<byte[]> dataList, List<Type> targetTypes);
}
```

#### <a name="serialization-in-the-c-side-and-deserialization-in-the-java-side"></a>Sérialisation côté C# et désérialisation côté Java

Spécifiez la méthode de sérialisation du côté C# dans votre code C#.

```csharp
this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer()); 
```

Spécifiez la méthode de désérialisation du côté Java dans un fichier de spécification.

```csharp
(scp-spout
   {
     "plugin.name" "HybridTopology.exe"
     "plugin.args" ["generator"]
     "output.schema" {"default" ["person"]}
     "customized.java.deserializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer" "microsoft.scp.example.HybridTopology.Person"]
   }
)
```

Ici, `"microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer"` est le nom du désérialiseur, et `"microsoft.scp.example.HybridTopology.Person"` est la classe cible vers laquelle les données sont désérialisées.

Vous pouvez également intégrer votre propre implémentation d’un sérialiseur C# et d’un désérialiseur Java.

Voici le code de l’interface pour le sérialiseur C# :

```csharp
public interface ICustomizedInteropCSharpSerializer
{
    List<byte[]> Serialize(List<object> dataList);
}
```

Voici le code de l’interface pour le désérialiseur Java :

```csharp
public interface ICustomizedInteropJavaDeserializer {
    public void prepare(String[] targetClassNames);
    public List<Object> Deserialize(List<ByteBuffer> dataList);
}
```

## <a name="scp-host-mode"></a>Mode d’hébergement SCP

Dans ce mode, vous pouvez compiler votre code en DLL, puis utiliser le fichier SCPHost.exe comme fourni par SCP pour envoyer une topologie. Un fichier de spécification ressemble à ce code :

```csharp
(scp-spout
  {
    "plugin.name" "SCPHost.exe"
    "plugin.args" ["HelloWorld.dll" "Scp.App.HelloWorld.Generator" "Get"]
    "output.schema" {"default" ["sentence"]}
  })
```

Ici, `"plugin.name"` est spécifié en tant que `"SCPHost.exe"`, qui est fourni par le Kit de développement logiciel (SDK) SCP. SCPHost.exe accepte trois paramètres dans l’ordre suivant :

1. Le nom de la DLL, `"HelloWorld.dll"` dans cet exemple.
1. Le nom de la classe, `"Scp.App.HelloWorld.Generator"` dans cet exemple.
1. Le nom de la méthode statique publique, qui peut être appelée pour obtenir une instance **ISCPPlugin**.

En mode d’hébergement, compilez votre code sous forme de DLL pour l’appel par la plateforme SCP. Étant donné que la plateforme peut ensuite obtenir le contrôle total de l’ensemble de la logique de traitement, nous vous recommandons d’envoyer la topologie en mode d’hébergement SCP. Cela simplifie l’expérience de développement. Cela offre également une plus grande flexibilité et une meilleure compatibilité descendante pour les versions ultérieures.

## <a name="scp-programming-examples"></a>Exemples de programmation SCP

### <a name="helloworld"></a>HelloWorld

L’exemple simple de HelloWorld qui suit montre un avant-goût de SCP.NET. Il emploie une topologie non transactionnelle, avec un Spout appelé **generator**, et deux Bolts appelés **splitter** et **counter**. Le Spout **generator** génère des phrases aléatoirement, avant de les émettre vers **splitter**. Le Bolt **splitter** divise les phrases en mots et les émet vers le Bolt **counter**. Le bolt **counter** utilise un dictionnaire pour enregistrer les occurrences de chaque mot.

Cet exemple contient deux fichiers de spécification : HelloWorld.spec et HelloWorld\_EnableAck.spec. Le code C# peut déterminer si l’accusé de réception est activé en récupérant l’objet `pluginConf` du côté Java.

```csharp
/* demo how to get pluginConf info */
if (Context.Config.pluginConf.ContainsKey(Constants.NONTRANSACTIONAL_ENABLE_ACK))
{
    enableAck = (bool)(Context.Config.pluginConf[Constants.NONTRANSACTIONAL_ENABLE_ACK]);
}
Context.Logger.Info("enableAck: {0}", enableAck);
```

Si l’accusé de réception est activé dans le Spout, un dictionnaire met en cache les tuples qui n’ont pas été reçus. Si `Fail` est appelée, les tuples ayant échoué sont relus.

```csharp
public void Fail(long seqId, Dictionary<string, Object> parms)
{
    Context.Logger.Info("Fail, seqId: {0}", seqId);
    if (cachedTuples.ContainsKey(seqId))
    {
        /* get the cached tuple */
        string sentence = cachedTuples[seqId];

        /* replay the failed tuple */
        Context.Logger.Info("Re-Emit: {0}, seqId: {1}", sentence, seqId);
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), seqId);
    }
    else
    {
        Context.Logger.Warn("Fail(), can't find cached tuple for seqId {0}!", seqId);
    }
}
```

### <a name="helloworldtx"></a>HelloWorldTx

L’exemple HelloWorldTx suivant illustre l’implémentation d’une topologie transactionnelle. L’exemple contient un Spout nommé **generator**, un Bolt de traitement nommé **partial-count** et un Bolt de validation nommé **count-sum**. L’exemple contient également trois fichiers texte existants : DataSource0.txt, DataSource1.txt et DataSource2.txt.

Dans chaque transaction, le Spout **generator** sélectionne deux des trois fichiers de façon aléatoire, avant d’émettre le nom des deux fichiers vers le Bolt **partial-count**. Le Bolt **partial-count** :

1. récupère un nom de fichier dans le tuple reçu ;
1. ouvre le fichier correspondant ;
1. compte le nombre de mots dans le fichier ;
1. émet le nombre de mots dans le Bolt **count-sum**.

Le bolt **count-sum** résume le résultat total.

Pour mener à bien la sémantique Exactement une fois, le Bolt de validation **count-sum** doit décider s’il s’agit d’une transaction relue. Dans cet exemple, il contient la variable de membre statique suivante :

```csharp
public static long lastCommittedTxId = -1; 
```

Au moment de la création d’une instance **ISCPBatchBolt**, la valeur de l’objet `txAttempt` est récupéré à partir des paramètres d’entrée.

```csharp
public static CountSum Get(Context ctx, Dictionary<string, Object> parms)
{
    /* for transactional topology, we can get txAttempt from the input parms */
    if (parms.ContainsKey(Constants.STORM_TX_ATTEMPT))
    {
        StormTxAttempt txAttempt = (StormTxAttempt)parms[Constants.STORM_TX_ATTEMPT];
        return new CountSum(ctx, txAttempt);
    }
    else
    {
        throw new Exception("null txAttempt");
    }
}
```

Quand `FinishBatch` est appelé, `lastCommittedTxId` est mis à jour s’il ne s’agit pas d’une transaction relue.

```csharp
public void FinishBatch(Dictionary<string, Object> parms)
{
    /* judge whether it is a replayed transaction */
    bool replay = (this.txAttempt.TxId <= lastCommittedTxId);

    if (!replay)
    {
        /* If it is not replayed, update the totalCount and lastCommittedTxId value */
        totalCount = totalCount + this.count;
        lastCommittedTxId = this.txAttempt.TxId;
    }
    … …
}
```

### <a name="hybridtopology"></a>HybridTopology

Cette topologie contient un Spout Java et un Bolt C#. Elle utilise l’implémentation de sérialisation et de désérialisation par défaut fournie par la plateforme SCP. Pour plus d’informations sur le fichier de spécification, consultez le fichier HybridTopology.spec dans le dossier examples\\HybridTopology. Consultez également SubmitTopology.bat pour savoir comment spécifier le classpath Java.

### <a name="scphostdemo"></a>SCPHostDemo

Cet exemple est essentiellement identique à celui de HelloWorld. Les seules différences sont que votre code est compilé en tant que DLL et que la topologie est envoyée en utilisant SCPHost.exe. Pour obtenir une explication plus détaillée, consultez la section relative au mode d’hébergement SCP.

## <a name="next-steps"></a>Étapes suivantes

Pour des exemples de topologies Apache Storm créées avec SCP, consultez les articles suivants :

* [Développement de topologies C# pour Apache Storm dans HDInsight à l'aide de Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Traiter des événements issus d’Azure Event Hubs avec Apache Storm sur HDInsight](apache-storm-develop-csharp-event-hub-topology.md)
* [Traiter des données de capteurs de véhicules issues d’Event Hubs avec Apache Storm sur HDInsight](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/IotExample)
* [Effectuer des opérations ETL (extraction, transformation et chargement) d’Azure Event Hubs vers Apache HBase](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample)
