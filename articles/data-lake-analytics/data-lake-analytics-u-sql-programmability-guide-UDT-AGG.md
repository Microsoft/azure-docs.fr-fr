---
title: Guide de programmabilité U-SQL UDT et UDAGG pour Azure Data Lake
description: En savoir plus sur la programmabilité U-SQL UDT et UDAGG pour Azure Data Lake Analytics afin de créer un script USQL efficace.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 10fcce9a667d9a08318f5adab804f482387052ff
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606655"
---
# <a name="u-sql-programmability-guide---udt-and-udagg"></a>Guide de programmabilité U-SQL - UDT et UDAGG



## <a name="use-user-defined-types-udt"></a>Utiliser des types définis par l’utilisateur : UDT
Les types définis par l’utilisateur (ou UDT) sont une autre fonctionnalité de programmabilité de U-SQL. Un type défini par l’utilisateur U-SQL agit comme un type défini par l’utilisateur C# standard. C# est un langage fortement typé qui permet d’utiliser des types définis par l’utilisateur intégrés et personnalisés.

U-SQL ne peut pas implicitement sérialiser ou désérialiser des types définis par l’utilisateur arbitraires lorsque le type défini par l’utilisateur est transmis entre les sommets des jeux de lignes. Cela implique que l’utilisateur doit fournir un formateur explicite à l’aide de l’interface IFormatter. U-SQL connaît ainsi les méthodes de sérialisation et désérialisation pour le type défini par l’utilisateur.

> [!NOTE]
> Les extracteurs et générateurs de sortie intégrés à U-SQL ne peuvent pas actuellement sérialiser ou désérialiser les données du type défini par l’utilisateur vers ou à partir de fichiers, même avec le jeu IFormatter. Par conséquent, lors de l’écriture de données du type défini par l’utilisateur dans un fichier avec l’instruction OUTPUT, ou lors de sa lecture à l’aide d’un extracteur, vous devez les transmettre sous la forme d’une chaîne ou d’un tableau d’octets. Puis appelez explicitement le code de sérialisation et de désérialisation (c’est-à-dire, la méthode ToString() du type défini par l’utilisateur). Les extracteurs et les générateurs de sortie définis par l’utilisateur, quant à eux, peuvent lire et écrire les types définis par l’utilisateur.

Si nous essayons d’utiliser un type défini par l’utilisateur dans un EXTRACTEUR ou un GÉNÉRATEUR DE SORTIE (à partir de l’instruction SELECT précédente), comme indiqué ici :

```usql
@rs1 =
    SELECT 
        MyNameSpace.Myfunction_Returning_UDT(filed1) AS myfield
    FROM @rs0;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

Nous recevons l’erreur suivante :

```output
Error   1   E_CSC_USER_INVALIDTYPEINOUTPUTTER: Outputters.Text was used to output column myfield of type
MyNameSpace.Myfunction_Returning_UDT.

Description:

Outputters.Text only supports built-in types.

Resolution:

Implement a custom outputter that knows how to serialize this type, or call a serialization method on the type in
the preceding SELECT.   C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\
USQL-Programmability\Types.usql 52  1   USQL-Programmability
```

Pour utiliser un type défini par l’utilisateur dans un générateur de sortie, nous devons soit le sérialiser en une chaîne avec la méthode ToString(), soit créer un générateur de sortie personnalisé.

Il n’est actuellement pas possible d’utiliser des types définis par l’utilisateur dans une instruction GROUP BY. Si un type défini par l’utilisateur est utilisé dans une instruction GROUP BY, l’erreur suivante est levée :

```output
Error   1   E_CSC_USER_INVALIDTYPEINCLAUSE: GROUP BY doesn't support type MyNameSpace.Myfunction_Returning_UDT
for column myfield

Description:

GROUP BY doesn't support UDT or Complex types.

Resolution:

Add a SELECT statement where you can project a scalar column that you want to use with GROUP BY.
C:\Users\sergeypu\Documents\Visual Studio 2013\Projects\USQL-Programmability\USQL-Programmability\Types.usql
62  5   USQL-Programmability
```

Pour définir un type défini par l’utilisateur, nous devons effectuer les opérations suivantes :

1. Ajoutez les espaces de noms suivants :

```csharp
using Microsoft.Analytics.Interfaces
using System.IO;
```

2. Ajouter `Microsoft.Analytics.Interfaces`, qui est requis pour les interfaces du type défini par l’utilisateur. En outre, `System.IO` peut être nécessaire pour définir l’interface IFormatter.

3. Définir un type défini par l’utilisateur avec l’attribut SqlUserDefinedType.

**SqlUserDefinedType** est utilisé pour marquer une définition de type dans un assembly en tant que type défini par l’utilisateur (UDT) dans U-SQL. Les propriétés de l’attribut reflètent les caractéristiques physiques du type défini par l’utilisateur. Cette classe ne peut pas être héritée.

SqlUserDefinedType est un attribut obligatoire pour la définition du type défini par l’utilisateur.

Le constructeur de la classe :  

* SqlUserDefinedTypeAttribute (formateur de type)

* Formateur de type : paramètre requis pour définir un formateur de type défini par l’utilisateur. Plus précisément, le type de l’interface `IFormatter` doit être transmis ici.

```csharp
[SqlUserDefinedType(typeof(MyTypeFormatter))]
public class MyType
{ … }
```

* Un type défini par l’utilisateur standard nécessite également une définition de l’interface IFormatter, comme illustré dans l’exemple suivant :

```csharp
public class MyTypeFormatter : IFormatter<MyType>
{
    public void Serialize(MyType instance, IColumnWriter writer, ISerializationContext context)
    { … }

    public MyType Deserialize(IColumnReader reader, ISerializationContext context)
    { … }
}
```

L’interface `IFormatter` sérialise et désérialise un graphique d’objets avec le type de racine \<typeparamref name="T">.

\<typeparam name="T">Type de racine pour le graphique d’objet à sérialiser et désérialiser.

* **Désérialiser** : désérialise les données sur le flux fourni et reconstitue le graphe d’objet.

* **Sérialiser** : sérialise un objet, ou graphe d’objet, avec la racine donnée dans le flux fourni.

Instance `MyType` : instance du type.  
Enregistreur `IColumnWriter` / Lecteur `IColumnReader` : flux de colonne sous-jacent.  
Contexte `ISerializationContext` : énumération définissant un jeu d’indicateurs qui spécifie le contexte source ou cible pour le flux pendant la sérialisation.

* **Intermediate** : spécifie que le contexte source ou cible n’est pas un magasin persistant.

* **Persistence** : spécifie que le contexte source ou cible est un magasin persistant.

En tant que type C# standard, une définition de type défini par l’utilisateur U-SQL peut inclure des remplacements pour des opérateurs tels que +/==/!=. Elle peut également inclure des méthodes statiques. Par exemple, si nous prévoyons d’utiliser ce type défini par l’utilisateur en tant que paramètre pour une fonction d’agrégation U-SQL MIN, nous devons définir un remplacement de l’opérateur <.

Précédemment dans ce guide, nous avons présenté un exemple d’identification de l’exercice fiscal en partant de la date spécifiée au format `Qn:Pn (Q1:P10)`. L’exemple suivant montre comment définir un type personnalisé pour les valeurs de période fiscale.

Vous trouverez ci-dessous un exemple de section code-behind avec un type défini par l’utilisateur et une interface personnalisée IFormatter :

```csharp
[SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
public struct FiscalPeriod
{
    public int Quarter { get; private set; }

    public int Month { get; private set; }

    public FiscalPeriod(int quarter, int month):this()
    {
    this.Quarter = quarter;
    this.Month = month;
    }

    public override bool Equals(object obj)
    {
    if (ReferenceEquals(null, obj))
    {
        return false;
    }

    return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
    }

    public bool Equals(FiscalPeriod other)
    {
return this.Quarter.Equals(other.Quarter) && this.Month.Equals(other.Month);
    }

    public bool GreaterThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
    }

    public bool LessThan(FiscalPeriod other)
    {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
    }

    public override int GetHashCode()
    {
    unchecked
    {
        return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
    }
    }

    public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
    {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
    }

    public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.Equals(c2);
    }

    public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
    {
    return !c1.Equals(c2);
    }
    public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.GreaterThan(c2);
    }
    public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
    {
    return c1.LessThan(c2);
    }
    public override string ToString()
    {
    return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
    }

}

public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
{
    public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
    {
    using (var binaryWriter = new BinaryWriter(writer.BaseStream))
    {
        binaryWriter.Write(instance.Quarter);
        binaryWriter.Write(instance.Month);
        binaryWriter.Flush();
    }
    }

    public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
    {
    using (var binaryReader = new BinaryReader(reader.BaseStream))
    {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
        return result;
    }
    }
}
```

Le type défini inclut deux numéros, respectivement pour le trimestre et le mois. Les opérateurs `==/!=/>/<` et la méthode statique `ToString()` sont définis ici.

Comme mentionné précédemment, le type défini par l’utilisateur peut être utilisé dans des expressions SELECT, mais pas dans un GÉNÉRATEUR DE SORTIE/EXTRACTEUR sans sérialisation personnalisée. Il doit être sérialisé en tant que chaîne avec `ToString()`, ou utilisé avec un OUTPUTTER/EXTRACTOR personnalisé.

Voyons à présent l’usage d’un type défini par l’utilisateur. Dans une section code-behind, nous avons modifié notre fonction GetFiscalPeriod comme suit :

```csharp
public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
{
    int FiscalMonth = 0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter = 0;
    if (FiscalMonth >= 1 && FiscalMonth <= 3)
    {
    FiscalQuarter = 1;
    }
    if (FiscalMonth >= 4 && FiscalMonth <= 6)
    {
    FiscalQuarter = 2;
    }
    if (FiscalMonth >= 7 && FiscalMonth <= 9)
    {
    FiscalQuarter = 3;
    }
    if (FiscalMonth >= 10 && FiscalMonth <= 12)
    {
    FiscalQuarter = 4;
    }

    return new FiscalPeriod(FiscalQuarter, FiscalMonth);
}
```

Comme vous pouvez le voir, elle retourne la valeur de notre type FiscalPeriod.

Voici un exemple montrant comment l’utiliser dans un script U-SQL de base. Cet exemple illustre différentes façons d’appeler un type défini par l’utilisateur à partir d’un script U-SQL.

```usql
DECLARE @input_file string = @"c:\work\cosmos\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"c:\work\cosmos\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        guid string,
        dt DateTime,
        user String,
        des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT 
        guid AS start_id,
        dt,
        DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Quarter AS fiscalquarter,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).Month AS fiscalmonth,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt) + new USQL_Programmability.CustomFunctions.FiscalPeriod(1,7) AS fiscalperiod_adjusted,
        user,
        des
    FROM @rs0;

@rs2 =
    SELECT 
        start_id,
        dt,
        DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
        fiscalquarter,
        fiscalmonth,
        USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS fiscalperiod,

       // This user-defined type was created in the prior SELECT.  Passing the UDT to this subsequent SELECT would have failed if the UDT was not annotated with an IFormatter.
           fiscalperiod_adjusted.ToString() AS fiscalperiod_adjusted,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```

Voici un exemple d’une section code-behind complète :

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        static public DateTime? ToDateTime(string dt)
        {
            DateTime dtValue;

            if (!DateTime.TryParse(dt, out dtValue))
                return Convert.ToDateTime(dt);
            else
                return null;
        }

        public static FiscalPeriod GetFiscalPeriodWithCustomType(DateTime dt)
        {
            int FiscalMonth = 0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter = 0;
            if (FiscalMonth >= 1 && FiscalMonth <= 3)
            {
                FiscalQuarter = 1;
            }
            if (FiscalMonth >= 4 && FiscalMonth <= 6)
            {
                FiscalQuarter = 2;
            }
            if (FiscalMonth >= 7 && FiscalMonth <= 9)
            {
                FiscalQuarter = 3;
            }
            if (FiscalMonth >= 10 && FiscalMonth <= 12)
            {
                FiscalQuarter = 4;
            }

            return new FiscalPeriod(FiscalQuarter, FiscalMonth);
        }        [SqlUserDefinedType(typeof(FiscalPeriodFormatter))]
        public struct FiscalPeriod
        {
            public int Quarter { get; private set; }

            public int Month { get; private set; }

            public FiscalPeriod(int quarter, int month):this()
            {
                this.Quarter = quarter;
                this.Month = month;
            }

            public override bool Equals(object obj)
            {
                if (ReferenceEquals(null, obj))
                {
                    return false;
                }

                return obj is FiscalPeriod && Equals((FiscalPeriod)obj);
            }

            public bool Equals(FiscalPeriod other)
            {
return this.Quarter.Equals(other.Quarter) &&    this.Month.Equals(other.Month);
            }

            public bool GreaterThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) > 0 || this.Month.CompareTo(other.Month) > 0;
            }

            public bool LessThan(FiscalPeriod other)
            {
return this.Quarter.CompareTo(other.Quarter) < 0 || this.Month.CompareTo(other.Month) < 0;
            }

            public override int GetHashCode()
            {
                unchecked
                {
                    return (this.Quarter.GetHashCode() * 397) ^ this.Month.GetHashCode();
                }
            }

            public static FiscalPeriod operator +(FiscalPeriod c1, FiscalPeriod c2)
            {
return new FiscalPeriod((c1.Quarter + c2.Quarter) > 4 ? (c1.Quarter + c2.Quarter)-4 : (c1.Quarter + c2.Quarter), (c1.Month + c2.Month) > 12 ? (c1.Month + c2.Month) - 12 : (c1.Month + c2.Month));
            }

            public static bool operator ==(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.Equals(c2);
            }

            public static bool operator !=(FiscalPeriod c1, FiscalPeriod c2)
            {
                return !c1.Equals(c2);
            }
            public static bool operator >(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.GreaterThan(c2);
            }
            public static bool operator <(FiscalPeriod c1, FiscalPeriod c2)
            {
                return c1.LessThan(c2);
            }
            public override string ToString()
            {
                return (String.Format("Q{0}:P{1}", this.Quarter, this.Month));
            }

        }

        public class FiscalPeriodFormatter : IFormatter<FiscalPeriod>
        {
public void Serialize(FiscalPeriod instance, IColumnWriter writer, ISerializationContext context)
            {
                using (var binaryWriter = new BinaryWriter(writer.BaseStream))
                {
                    binaryWriter.Write(instance.Quarter);
                    binaryWriter.Write(instance.Month);
                    binaryWriter.Flush();
                }
            }

public FiscalPeriod Deserialize(IColumnReader reader, ISerializationContext context)
            {
                using (var binaryReader = new BinaryReader(reader.BaseStream))
                {
var result = new FiscalPeriod(binaryReader.ReadInt16(), binaryReader.ReadInt16());
                    return result;
                }
            }
        }
    }
}
```

## <a name="use-user-defined-aggregates-udagg"></a>Utiliser des agrégats définis par l’utilisateur : UDAGG
Les agrégats définis par l’utilisateur sont toutes les fonctions d’agrégation qui ne sont pas fournies prêtes à l’emploi avec U-SQL. L’exemple peut être un agrégat effectuant des calculs mathématiques personnalisés, des concaténations de chaînes, des manipulations de chaînes, etc.

La définition de classe de base d’agrégat défini par l’utilisateur est la suivante :

```csharp
    [SqlUserDefinedAggregate]
    public abstract class IAggregate<T1, T2, TResult> : IAggregate
    {
        protected IAggregate();

        public abstract void Accumulate(T1 t1, T2 t2);
        public abstract void Init();
        public abstract TResult Terminate();
    }
```

**SqlUserDefinedAggregate** indique que le type doit être inscrit en tant qu’agrégat défini par l’utilisateur. Cette classe ne peut pas être héritée.

L’attribut SqlUserDefinedType est **facultatif** pour la définition d’UDAGG.


La classe de base permet de transmettre trois paramètres abstraits : deux paramètres d’entrée et un de résultat. Les types de données sont variables et doivent être définis lors de l’héritage de classe.

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    { … }

    public override void Accumulate(string guid, string user)
    { … }

    public override string Terminate()
    { … }
}
```

* **Init** appelle une seule fois chaque groupe pendant le calcul. Elle fournit une routine d’initialisation pour chaque groupe d’agrégation.  
* **Accumulate** est exécutée une fois pour chaque valeur. Elle fournit la fonctionnalité principale pour l’algorithme d’agrégation. Elle peut être utilisée pour agréger des valeurs de différents types de données qui sont définis lors de l’héritage de classe. Elle peut accepter deux paramètres de types de données variables.
* **Terminate** est exécutée une fois par groupe d’agrégation à la fin du traitement pour générer le résultat de chaque groupe.

Pour déclarer des types de données d’entrée et de sortie corrects, utilisez la définition de classe suivante :

```csharp
public abstract class IAggregate<T1, T2, TResult> : IAggregate
```

* T1 : premier paramètre pour la routine accumulate
* T2 : deuxième paramètre pour la routine accumulate
* TResult : type de retour de la routine terminate

Par exemple :

```csharp
public class GuidAggregate : IAggregate<string, int, int>
```

or

```csharp
public class GuidAggregate : IAggregate<string, string, string>
```

### <a name="use-udagg-in-u-sql"></a>Utiliser UDAGG dans U-SQL
Pour utiliser UDAGG, commencez par le définir dans code-behind ou par le référencer à partir de la DLL de programmabilité existante, comme décrit précédemment.

Utilisez ensuite la syntaxe suivante :

```csharp
AGG<UDAGG_functionname>(param1,param2)
```

Voici l’exemple d’UDAGG :

```csharp
public class GuidAggregate : IAggregate<string, string, string>
{
    string guid_agg;

    public override void Init()
    {
        guid_agg = "";
    }

    public override void Accumulate(string guid, string user)
    {
        if (user.ToUpper()== "USER1")
        {
        guid_agg += "{" + guid + "}";
        }
    }

    public override string Terminate()
    {
        return guid_agg;
    }

}
```

Et le script U-SQL de base :

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @" \usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    SELECT
        user,
        AGG<USQL_Programmability.GuidAggregate>(guid,user) AS guid_list
    FROM @rs0
    GROUP BY user;

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

Dans ce scénario d’utilisation, nous concaténons des GUID de classe pour les utilisateurs spécifiés.

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDO](data-lake-analytics-u-sql-programmability-guide-UDO.md)
