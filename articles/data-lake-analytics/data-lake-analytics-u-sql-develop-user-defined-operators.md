---
title: Développer des opérateurs U-SQL définis par l’utilisateur - Azure Data Lake Analytics
description: Découvrez comment développer des opérateurs définis par l’utilisateur pour les utiliser et les réutiliser dans des travaux Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 12/05/2016
ms.openlocfilehash: 11efdb727bacadb674fb49374ef1c70fcc788ecc
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219987"
---
# <a name="develop-u-sql-user-defined-operators-udos"></a>Développer des opérateurs U-SQL définis par l’utilisateur
Cet article explique comment développer des opérateurs définis par l’utilisateur pour traiter les données d’un travail U-SQL.

## <a name="define-and-use-a-user-defined-operator-in-u-sql"></a>Définir et utiliser un opérateur défini par l’utilisateur dans U-SQL

### <a name="to-create-and-submit-a-u-sql-job"></a>Pour créer et soumettre un travail U-SQL

1. Dans Visual Studio, sélectionnez **Fichier > Nouveau > Projet > Projet U-SQL**.
2. Cliquez sur **OK**. Visual Studio crée une solution avec un fichier Script.usql.
3. À partir de **l’Explorateur de solutions**, développez Script.usql, puis double-cliquez sur **Script.usql.cs**.
4. Collez le code suivant dans le fichier :

   ```usql
   using Microsoft.Analytics.Interfaces;
   using System.Collections.Generic;
   namespace USQL_UDO
   {
       public class CountryName : IProcessor
       {
           private static IDictionary<string, string> CountryTranslation = new Dictionary<string, string>
           {
               {
                   "Deutschland", "Germany"
               },
               {
                   "Suisse", "Switzerland"
               },
               {
                   "UK", "United Kingdom"
               },
               {
                   "USA", "United States of America"
               },
               {
                   "中国", "PR China"
               }
           };
           public override IRow Process(IRow input, IUpdatableRow output)
           {
               string UserID = input.Get<string>("UserID");
               string Name = input.Get<string>("Name");
               string Address = input.Get<string>("Address");
               string City = input.Get<string>("City");
               string State = input.Get<string>("State");
               string PostalCode = input.Get<string>("PostalCode");
               string Country = input.Get<string>("Country");
               string Phone = input.Get<string>("Phone");
               if (CountryTranslation.Keys.Contains(Country))
               {
                   Country = CountryTranslation[Country];
               }
               output.Set<string>(0, UserID);
               output.Set<string>(1, Name);
               output.Set<string>(2, Address);
               output.Set<string>(3, City);
               output.Set<string>(4, State);
               output.Set<string>(5, PostalCode);
               output.Set<string>(6, Country);
               output.Set<string>(7, Phone);
               return output.AsReadOnly();
           }
       }
   }
   ```

5. Ouvrez **Script.usql** et collez le script U-SQL suivant :

   ```usql
   @drivers =
       EXTRACT UserID      string,
               Name        string,
               Address     string,
               City        string,
               State       string,
               PostalCode  string,
               Country     string,
               Phone       string
       FROM "/Samples/Data/AmbulanceData/Drivers.txt"
       USING Extractors.Tsv(Encoding.Unicode);

   @drivers_CountryName =
       PROCESS @drivers
       PRODUCE UserID string,
               Name string,
               Address string,
               City string,
               State string,
               PostalCode string,
               Country string,
               Phone string
       USING new USQL_UDO.CountryName();

   OUTPUT @drivers_CountryName
       TO "/Samples/Outputs/Drivers.csv"
       USING Outputters.Csv(Encoding.Unicode);
   ```

6. Spécifiez le compte Data Lake Analytics, la base de données et le schéma.
7. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Script.usql**, puis cliquez sur **Générer le script**.
8. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Script.usql**, puis cliquez sur **Soumettre le script**.
9. Si vous ne vous êtes pas connecté à votre abonnement Azure, vous serez invité à entrer vos informations d’identification de compte Azure.
10. Cliquez sur **Envoyer**. Les résultats de l’envoi et le lien vers le travail sont disponibles dans la fenêtre Résultats quand l’envoi est terminé.
11. Cliquez sur le bouton **Actualiser** pour afficher le dernier état du travail et actualiser l’écran.

### <a name="to-see-the-output"></a>Pour afficher la sortie

1. Dans **l’Explorateur de serveurs**, développez successivement **Azure**, **Data Lake Analytics**, votre compte Data Lake Analytics, **Comptes de stockage**, puis cliquez avec le bouton droit sur le stockage par défaut et sur **Explorer**.

2. Développez des exemples, des sorties, puis double-cliquez sur **Drivers.csv**.

## <a name="next-steps"></a>Étapes suivantes

* [Extension des expressions U-SQL avec le code utilisateur](/u-sql/concepts/extending-u-sql-expressions-with-user-code)
* [Utiliser les outils Data Lake pour Visual Studio pour le développement d’applications U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
