---
title: Interroger les données Avro à l’aide d’Azure Data Lake Analytics | Microsoft Docs
description: Utilisez les propriétés du corps de message pour acheminer les données de télémétrie de l’appareil vers le stockage Blob et interrogez les données au format Avro enregistrées dans le stockage Blob.
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: asrastog
ms.openlocfilehash: 3cfe75edcf338f5248baf396147a5b77803fbfb3
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655932"
---
# <a name="query-avro-data-by-using-azure-data-lake-analytics"></a>Interroger les données Avro à l’aide d’Azure Data Lake Analytics

Cet article vous explique comment interroger les données Avro afin d’acheminer efficacement les messages entre Azure IoT Hub et les services Azure. Le [Routage des messages](iot-hub-devguide-messages-d2c.md) vous permet de filtrer les données au moyen de requêtes élaborées, basées sur les propriétés de message, les corps de message, et les étiquettes et propriétés de jumeau d’appareil. Pour en savoir plus sur les fonctionnalités d’interrogation dans le routage des messages, consultez l’article sur la [syntaxe des requêtes du routage des messages](iot-hub-devguide-routing-query-syntax.md).

La principale difficulté survient lorsque Azure IoT Hub achemine les messages vers le stockage Blob Azure et qu’IoT Hub écrit par défaut le contenu au format Avro, qui présente à la fois une propriété de corps de message et une propriété de message. Le format Avro n’est pas utilisé pour d’autres points de terminaison. Bien que le format Avro soit exceptionnellement adapté pour la conservation des messages et des données, il est moins adapté pour l’interrogation des données. En comparaison, le format JSON ou CSV est beaucoup plus pratique pour interroger des données. IoT Hub prend désormais en charge l’écriture de données dans le stockage d’objets Blob aux formats JSON et AVRO.

Pour plus d’informations, consultez [Using Azure Storage as a routing endpoint (Utilisation du stockage Azure comme point de terminaison de routage)](iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint).

Pour résoudre ce problème, vous pouvez utiliser la plupart des modèles de Big Data pour transformer et mettre à l’échelle les données afin de répondre aux besoins et aux formats en matière de Big Data non relationnelles. Un de ces modèles, le « paiement à la requête », est Azure Data Lake Analytics, qui constitue l’objet principal de cet article. Bien que vous puissiez facilement exécuter la requête dans Hadoop ou dans d’autres solutions, Data Lake Analytics se révèle souvent mieux adapté pour cette approche de « paiement à la requête ».

Il existe un « extracteur » pour Avro dans U-SQL. Pour plus d’informations, consultez [Exemple Avro U-SQL](https://github.com/Azure/usql/tree/master/Examples/AvroExamples).

## <a name="query-and-export-avro-data-to-a-csv-file"></a>Interroger et exporter les données Avro vers un fichier CSV

Dans cette section, vous interrogez des données Avro et les exportez vers un fichier CSV dans le stockage Blob Azure, bien qu’il soit facile de placer les données dans d’autres référentiels ou magasins de données.

1. Configurez Azure IoT Hub pour acheminer les données vers un point de terminaison de stockage Blob Azure à l’aide d’une propriété dans le corps du message pour sélectionner les messages.

   ![Section « Points de terminaison personnalisés »](./media/iot-hub-query-avro-data/query-avro-data-1a.png)

   ![Les règles de routage](./media/iot-hub-query-avro-data/query-avro-data-1b.png)

   Pour plus d’informations sur la configuration des itinéraires et des points de terminaison personnalisés, consultez [Routage des messages pour un hub IoT](iot-hub-create-through-portal.md#message-routing-for-an-iot-hub).

2. Assurez-vous que votre appareil dispose du codage, du type de contenu et des données nécessaires dans les propriétés ou le corps du message, comme indiqué dans la documentation du produit. Lorsque vous affichez ces attributs dans Device Explorer, comme illustré ici, vous pouvez vérifier qu’ils sont correctement définis.

   ![Volet de données Event Hub](./media/iot-hub-query-avro-data/query-avro-data-2.png)

3. Configurez une instance Azure Data Lake Store et une instance Data Lake Analytics. Azure IoT Hub n’achemine pas vers une instance Data Lake Store, mais une instance Data Lake Analytics en requiert une.

   ![Instances Data Lake Store et Data Lake Analytics](./media/iot-hub-query-avro-data/query-avro-data-3.png)

4. Dans Data Lake Analytics, configurez le stockage Blob Azure comme un magasin supplémentaire, le même stockage Blob que celui vers lequel Azure IoT Hub achemine les données.

   ![Volet « Sources de données »](./media/iot-hub-query-avro-data/query-avro-data-4.png)

5. Comme indiqué dans l’[exemple Avro U-SQL](https://github.com/Azure/usql/tree/master/Examples/AvroExamples), vous avez besoin de quatre fichiers DLL. Chargez ces fichiers vers un emplacement dans votre instance Data Lake Store.

   ![Quatre fichiers DLL chargés](./media/iot-hub-query-avro-data/query-avro-data-5.png)

6. Dans Visual Studio, créez un projet U-SQL.

   ![Créer un projet U-SQL](./media/iot-hub-query-avro-data/query-avro-data-6.png)

7. Collez le contenu du script suivant dans le fichier qui vient d’être créé. Modifiez les trois sections en surbrillance : votre compte Data Lake Analytics, les chemins d’accès aux DLL associés et le chemin d’accès correct à votre compte de stockage.

   ![Les trois sections à modifier](./media/iot-hub-query-avro-data/query-avro-data-7a.png)

   Le script U-SQL réel pour une sortie simple dans un fichier CSV :

    ```sql
        DROP ASSEMBLY IF EXISTS [Avro];
        CREATE ASSEMBLY [Avro] FROM @"/Assemblies/Avro/Avro.dll";
        DROP ASSEMBLY IF EXISTS [Microsoft.Analytics.Samples.Formats];
        CREATE ASSEMBLY [Microsoft.Analytics.Samples.Formats] FROM @"/Assemblies/Avro/Microsoft.Analytics.Samples.Formats.dll";
        DROP ASSEMBLY IF EXISTS [Newtonsoft.Json];
        CREATE ASSEMBLY [Newtonsoft.Json] FROM @"/Assemblies/Avro/Newtonsoft.Json.dll";
        DROP ASSEMBLY IF EXISTS [log4net];
        CREATE ASSEMBLY [log4net] FROM @"/Assemblies/Avro/log4net.dll";

        REFERENCE ASSEMBLY [Newtonsoft.Json];
        REFERENCE ASSEMBLY [log4net];
        REFERENCE ASSEMBLY [Avro];
        REFERENCE ASSEMBLY [Microsoft.Analytics.Samples.Formats];

        // Blob container storage account filenames, with any path
        DECLARE @input_file string = @"wasb://hottubrawdata@kevinsayazstorage/kevinsayIoT/{*}/{*}/{*}/{*}/{*}/{*}";
        DECLARE @output_file string = @"/output/output.csv";

        @rs =
        EXTRACT
        EnqueuedTimeUtc string,
        Body byte[]
        FROM @input_file

        USING new Microsoft.Analytics.Samples.Formats.ApacheAvro.AvroExtractor(@"
        {
            ""type"":""record"",
            ""name"":""Message"",
            ""namespace"":""Microsoft.Azure.Devices"",
            ""fields"":
           [{
                ""name"":""EnqueuedTimeUtc"",
                ""type"":""string""
            },
            {
                ""name"":""Properties"",
                ""type"":
                {
                    ""type"":""map"",
                    ""values"":""string""
                }
            },
            {
                ""name"":""SystemProperties"",
                ""type"":
                {
                    ""type"":""map"",
                    ""values"":""string""
                }
            },
            {
                ""name"":""Body"",
                ""type"":[""null"",""bytes""]
            }]
        }"
        );

        @cnt =
        SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
        FROM @rs;

        OUTPUT @cnt TO @output_file USING Outputters.Text(); 
    ```

    Data Lake Analytics a eu besoin de cinq minutes pour exécuter le script suivant, limité à 10 unités d’analytique et 177 fichiers traités. Le résultat est affiché dans la sortie du fichier CSV affiché dans l’image suivante :

    ![Résultats de la sortie dans un fichier CSV](./media/iot-hub-query-avro-data/query-avro-data-7b.png)

    ![Sortie convertie en fichier CSV](./media/iot-hub-query-avro-data/query-avro-data-7c.png)

    Pour analyser le JSON, passez à l’étape 8.

8. La plupart des messages IoT sont au format de fichier JSON. En ajoutant les lignes suivantes, vous pouvez analyser le message dans un fichier JSON, ce qui vous permet d’ajouter les clauses WHERE et d’obtenir uniquement les données nécessaires.

    ```sql
       @jsonify =
         SELECT Microsoft.Analytics.Samples.Formats.Json.JsonFunctions.JsonTuple(Encoding.UTF8.GetString(Body))
           AS message FROM @rs;
    
        /*
        @cnt =
            SELECT EnqueuedTimeUtc AS time, Encoding.UTF8.GetString(Body) AS jsonmessage
            FROM @rs;
        
        OUTPUT @cnt TO @output_file USING Outputters.Text();
        */
        
        @cnt =
            SELECT message["message"] AS iotmessage,
                message["event"] AS msgevent,
                message["object"] AS msgobject,
                message["status"] AS msgstatus,
                message["host"] AS msghost
            FROM @jsonify;
            
        OUTPUT @cnt TO @output_file USING Outputters.Text();
    ```

    La sortie affiche une colonne pour chaque élément dans la commande `SELECT`.

    ![Sortie affichant une colonne pour chaque élément](./media/iot-hub-query-avro-data/query-avro-data-8.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment interroger les données Avro afin d’acheminer efficacement les messages entre Azure IoT Hub et les services Azure.

Pour obtenir des exemples de solutions de bout en bout complètes, qui utilisent IoT Hub, consultez la [documentation sur les accélérateurs de solution Azure IoT](../iot-accelerators/index.yml).

Pour en savoir plus sur le développement de solutions avec IoT Hub, consultez le [Guide du développeur d’IoT Hub](iot-hub-devguide.md).

Pour en savoir plus sur le routage des messages dans IoT Hub, consultez [Envoyer et recevoir des messages avec IoT Hub](iot-hub-devguide-messaging.md).
