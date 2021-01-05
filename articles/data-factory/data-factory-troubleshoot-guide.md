---
title: Résoudre les problèmes rencontrés dans Azure Data Factory | Microsoft Docs
description: Découvrez comment résoudre les problèmes liés aux activités de contrôle externes dans Azure Data Factory.
services: data-factory
author: nabhishek
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 11/16/2020
ms.author: abnarain
ms.reviewer: craigg
ms.openlocfilehash: c9dd39ffa68d8261f5c5d301d4c351c52b3f27c1
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654590"
---
# <a name="troubleshoot-azure-data-factory"></a>Résoudre les problèmes dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article présente des méthodes couramment employées pour résoudre les problèmes liés aux activités de contrôle externes dans Azure Data Factory.

## <a name="connector-and-copy-activity"></a>Activité de copie et connecteur

Pour des problèmes de connecteur, tels qu’une erreur lors de l’utilisation d’une activité de copie, consultez [Résoudre les problèmes liés aux connecteurs dans Azure Data Factory](connector-troubleshoot-guide.md).

## <a name="azure-databricks"></a>Azure Databricks

### <a name="error-code-3200"></a>Code d’erreur : 3200

- **Message** : Erreur 403.

- **Cause** : `The Databricks access token has expired.`

- **Recommandation** : Par défaut, le jeton d’accès Azure Databricks est valide pendant 90 jours. Créez un autre jeton et mettez à jour le service lié.

### <a name="error-code-3201"></a>Code d’erreur : 3201

- **Message** : `Missing required field: settings.task.notebook_task.notebook_path.`

- **Cause** : `Bad authoring: Notebook path not specified correctly.`

- **Recommandation** : Spécifiez le chemin du notebook dans l’activité Databricks.

<br/> 

- **Message** : `Cluster... does not exist.`

- **Cause** : `Authoring error: Databricks cluster does not exist or has been deleted.`

- **Recommandation** : Vérifiez que le cluster Databricks existe.

<br/> 

- **Message** : `Invalid Python file URI... Please visit Databricks user guide for supported URI schemes.`

- **Cause** : `Bad authoring.`

- **Recommandation** : Spécifiez les chemins absolus des schémas d’adressage d’espace de travail ou `dbfs:/folder/subfolder/foo.py` pour les fichiers stockés dans Databricks File System (DFS).

<br/> 

- **Message** : `{0} LinkedService should have domain and accessToken as required properties.`

- **Cause** : `Bad authoring.`

- **Recommandation** : Vérifiez la [définition du service lié](compute-linked-services.md#azure-databricks-linked-service).

<br/> 

- **Message** : `{0} LinkedService should specify either existing cluster ID or new cluster information for creation.`

- **Cause** : `Bad authoring.`

- **Recommandation** : Vérifiez la [définition du service lié](compute-linked-services.md#azure-databricks-linked-service).

<br/> 

- **Message** : `Node type Standard_D16S_v3 is not supported. Supported node types: Standard_DS3_v2, Standard_DS4_v2, Standard_DS5_v2, Standard_D8s_v3, Standard_D16s_v3, Standard_D32s_v3, Standard_D64s_v3, Standard_D3_v2, Standard_D8_v3, Standard_D16_v3, Standard_D32_v3, Standard_D64_v3, Standard_D12_v2, Standard_D13_v2, Standard_D14_v2, Standard_D15_v2, Standard_DS12_v2, Standard_DS13_v2, Standard_DS14_v2, Standard_DS15_v2, Standard_E8s_v3, Standard_E16s_v3, Standard_E32s_v3, Standard_E64s_v3, Standard_L4s, Standard_L8s, Standard_L16s, Standard_L32s, Standard_F4s, Standard_F8s, Standard_F16s, Standard_H16, Standard_F4s_v2, Standard_F8s_v2, Standard_F16s_v2, Standard_F32s_v2, Standard_F64s_v2, Standard_F72s_v2, Standard_NC12, Standard_NC24, Standard_NC6s_v3, Standard_NC12s_v3, Standard_NC24s_v3, Standard_L8s_v2, Standard_L16s_v2, Standard_L32s_v2, Standard_L64s_v2, Standard_L80s_v2.`

- **Cause** : `Bad authoring.`

- **Recommandation** : Voir le message d’erreur.

<br/>

### <a name="error-code-3202"></a>Code d’erreur : 3202

- **Message** : `There were already 1000 jobs created in past 3600 seconds, exceeding rate limit: 1000 job creations per 3600 seconds.`

- **Cause** : `Too many Databricks runs in an hour.`

- **Recommandation** : Vérifiez le taux de création de travaux de tous les pipelines qui utilisent cet espace de travail Databricks. Si les pipelines ont lancé trop de travaux Databricks dans un agrégat, migrez des pipelines dans un nouvel espace de travail.

<br/> 

- **Message** : `Could not parse request object: Expected 'key' and 'value' to be set for JSON map field base_parameters, got 'key: "..."' instead.`

- **Cause** : `Authoring error: No value provided for the parameter.`

- **Recommandation** : Inspectez le code JSON de pipeline et vérifiez que tous les paramètres du notebook baseParameters spécifient une valeur non vide.

<br/> 

- **Message** : `User: `SimpleUserContext{userId=..., name=user@company.com, orgId=...}` is not authorized to access cluster.`

- **Cause** : L’utilisateur qui a généré le jeton d’accès n’est pas autorisé à accéder au cluster Databricks spécifié dans le service lié.

- **Recommandation** : Vérifiez que l’utilisateur dispose des autorisations nécessaires dans l’espace de travail.

### <a name="error-code-3203"></a>Code d’erreur : 3203

- **Message** : `The cluster is in Terminated state, not available to receive jobs. Please fix the cluster or retry later.`

- **Cause** : Le cluster a été arrêté. Pour les clusters interactifs, il peut s’agir d’une condition de concurrence.

- **Recommandation** : Pour éviter cette erreur, utilisez des clusters de travail.

### <a name="error-code-3204"></a>Code d’erreur : 3204

- **Message** : `Job execution failed.`

- **Cause** : Les messages d’erreur indiquent différents problèmes, tels qu’un état de cluster inattendu ou une activité inattendue. Souvent, aucun message d'erreur ne s'affiche.

- **Recommandation** : N/A

### <a name="error-code-3208"></a>Code d’erreur : 3208

- **Message** : `An error occurred while sending the request.`

- **Cause** : La connexion réseau au service Databricks a été interrompue.

- **Recommandation** : Si vous utilisez un runtime d’intégration auto-hébergé, assurez-vous que la connexion réseau est fiable à partir des nœuds du runtime d’intégration. Si vous utilisez le runtime d’intégration Azure, une nouvelle tentative fonctionne généralement.
 
## <a name="azure-data-lake-analytics"></a>Service Analytique Azure Data Lake

Le tableau suivant s’applique à U-SQL.
 
### <a name="error-code-2709"></a>Code d’erreur : 2709

- **Message** : `The access token is from the wrong tenant.`

- **Cause** : Mauvais locataire Azure Active Directory (Azure AD).

- **Recommandation** : Mauvais locataire Azure Active Directory (Azure AD).

<br/>

- **Message** : `We cannot accept your job at this moment. The maximum number of queued jobs for your account is 200. `

- **Cause** : Cette erreur est provoquée par une limitation dans Data Lake Analytics.

- **Recommandation** : Réduisez le nombre de travaux soumis à Data Lake Analytics. Vous pouvez modifier les déclencheurs Data Factory et les paramètres de concurrence sur les activités ou augmenter les limites sur Data Lake Analytics.

<br/> 

- **Message** : `This job was rejected because it requires 24 AUs. This account's administrator-defined policy prevents a job from using more than 5 AUs.`

- **Cause** : Cette erreur est provoquée par une limitation dans Data Lake Analytics.

- **Recommandation** : Réduisez le nombre de travaux soumis à Data Lake Analytics. Vous pouvez modifier les déclencheurs Data Factory et les paramètres de concurrence sur les activités ou augmenter les limites sur Data Lake Analytics.

### <a name="error-code-2705"></a>Code d’erreur : 2705

- **Message** : `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Cause** : Le principal de service ou le certificat fourni n’a pas accès au fichier situé dans le stockage.

- **Recommandation** : Vérifiez que le principal de service ou le certificat fourni par l’utilisateur pour les travaux Data Lake Analytics a accès au compte Data Lake Analytics et à l’instance Data Lake Storage par défaut à partir du dossier racine.

### <a name="error-code-2711"></a>Code d’erreur : 2711

- **Message** : `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Cause** : Le principal de service ou le certificat fourni n’a pas accès au fichier situé dans le stockage.

- **Recommandation** : Vérifiez que le principal de service ou le certificat fourni par l’utilisateur pour les travaux Data Lake Analytics a accès au compte Data Lake Analytics et à l’instance Data Lake Storage par défaut à partir du dossier racine.

<br/> 

- **Message** : `Cannot find the 'Azure Data Lake Store' file or folder.`

- **Cause** : Le chemin vers le fichier U-SQL est erroné ou les informations d'identification du service lié n’autorisent pas l’accès.

- **Recommandation** : Vérifiez le chemin et les informations d’identification fournies dans le service lié.

### <a name="error-code-2704"></a>Code d’erreur : 2704

- **Message** : `Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.<br/> <br/> User is not able to access Data Lake Store. <br/> <br/> User is not authorized to use Data Lake Analytics.`

- **Cause** : Le principal de service ou le certificat fourni n’a pas accès au fichier situé dans le stockage.

- **Recommandation** : Vérifiez que le principal de service ou le certificat fourni par l’utilisateur pour les travaux Data Lake Analytics a accès au compte Data Lake Analytics et à l’instance Data Lake Storage par défaut à partir du dossier racine.

### <a name="error-code-2707"></a>Code d’erreur : 2707

- **Message** : `Cannot resolve the account of AzureDataLakeAnalytics. Please check 'AccountName' and 'DataLakeAnalyticsUri'.`

- **Cause** : Le compte Data Lake Analytics du service lié est incorrect.

- **Recommandation** : Vérifiez que le bon compte a été fourni.

### <a name="error-code-2703"></a>Code d’erreur : 2703

- **Message** : `Error Id: E_CQO_SYSTEM_INTERNAL_ERROR (or any error that starts with "Error Id:").`

- **Cause** : L’erreur provient de Data Lake Analytics.

- **Recommandation** : Le travail envoyé à Data Lake Analytics et le script ont échoués. Examinez le problème dans Data Lake Analytics. Dans le portail, accédez au compte Data Lake Analytics et recherchez le travail à l’aide de l’ID d’exécution de l’activité Data Factory (et non de l’ID d’exécution du pipeline). Le travail fournit davantage d’informations sur l’erreur et vous permettra de détecter un problème.

   Si vous n’êtes pas sûr d’avoir résolu votre problème, contactez l’équipe du support technique Data Lake Analytics pour lui donner l’URL (Universal Resource Locator) du travail, qui comprend le nom de votre compte et l’ID du travail.
 
## <a name="azure-functions"></a>Fonctions Azure

### <a name="error-code-3602"></a>Code d’erreur : 3602

- **Message** : `Invalid HttpMethod: '%method;'.`

- **Cause** : La HttpMethod spécifiée dans la charge utile d’activité n’est pas prise en charge par l’activité de fonction Azure.

- **Recommandation** : Les Httpmethods prises en charge sont les suivantes : PUT, POST, GET, DELETE, OPTIONS, HEAD et TRACE.

### <a name="error-code-3603"></a>Code d’erreur : 3603

- **Message** : `Response Content is not a valid JObject.`

- **Cause** : La fonction Azure qui a été appelée n’a pas retourné de charge utile JSON dans sa réponse. L’activité de fonction Azure Data Factory (ADF) Azure prend uniquement en charge le contenu de la réponse JSON.

- **Recommandation** : Mettez à jour la fonction Azure pour retourner une charge utile JSON valide car une fonction C# doit retourner `(ActionResult)new OkObjectResult("{\"Id\":\"123\"}");`

### <a name="error-code-3606"></a>Code d’erreur : 3606

- **Message** : Clé de fonction manquante dans l’activité de fonction Azure.

- **Cause** : La définition de l’activité de fonction Azure n’est pas terminée.

- **Recommandation** : Vérifiez que la définition JSON de l’activité de fonction Azure dispose d'une propriété nommée `functionKey`.

### <a name="error-code-3607"></a>Code d’erreur : 3607

- **Message** : `Azure function activity missing function name.`

- **Cause** : La définition de l’activité de fonction Azure n’est pas terminée.

- **Recommandation** : Vérifiez que la définition JSON de l’activité de fonction Azure dispose d'une propriété nommée `functionName`.

### <a name="error-code-3608"></a>Code d’erreur : 3608

- **Message** : `Call to provided Azure function '%FunctionName;' failed with status-'%statusCode;' and message - '%message;'.`

- **Cause** : Les détails de la fonction Azure dans la définition de l’activité sont peut-être incorrects.

- **Recommandation** : Corrigez les détails de la fonction Azure, puis réessayez.

### <a name="error-code-3609"></a>Code d’erreur : 3609

- **Message** : `Azure function activity missing functionAppUrl.`

- **Cause** : La définition de l’activité de fonction Azure n’est pas terminée.

- **Recommandation** : Vérifiez que la définition JSON de l’activité de fonction Azure dispose d'une propriété nommée `functionAppUrl`.

### <a name="error-code-3610"></a>Code d’erreur : 3610

- **Message** : `There was an error while calling endpoint.`

- **Cause** : L’URL de la fonction est peut-être incorrecte.

- **Recommandation** : Vérifiez que la valeur de `functionAppUrl` dans le JSON de l’activité est correcte, puis réessayez.

### <a name="error-code-3611"></a>Code d’erreur : 3611

- **Message** : `Azure function activity missing Method in JSON.`

- **Cause** : La définition de l’activité de fonction Azure n’est pas terminée.

- **Recommandation** : Vérifiez que la définition JSON de l’activité de fonction Azure dispose d'une propriété nommée `method`.

### <a name="error-code-3612"></a>Code d’erreur : 3612

- **Message** : `Azure function activity missing LinkedService definition in JSON.`

- **Cause** : La définition de l’activité de fonction Azure n’est pas terminée.

- **Recommandation** : Vérifiez que la définition JSON de l’activité de la fonction Azure dispose des détails sur le service lié.

## <a name="azure-machine-learning"></a>Azure Machine Learning

### <a name="error-code-4101"></a>Code d’erreur : 4101

- **Message** : `AzureMLExecutePipeline activity '%activityName;' has invalid value for property '%propertyName;'.`

- **Cause** : Format incorrect ou définition manquante de la propriété `%propertyName;`.

- **Recommandation** : Vérifiez si l’activité `%activityName;` a la propriété `%propertyName;` définie avec les données qui conviennent.

### <a name="error-code-4110"></a>Code d’erreur : 4110

- **Message** : `AzureMLExecutePipeline activity missing LinkedService definition in JSON.`

- **Cause** : La définition de l’activité AzureMLExecutePipeline n’est pas terminée.

- **Recommandation** : Vérifiez si la définition JSON de l’activité AzureMLExecutePipeline dispose des détails corrects sur le service lié.

### <a name="error-code-4111"></a>Code d’erreur : 4111

- **Message** : `AzureMLExecutePipeline activity has wrong LinkedService type in JSON. Expected LinkedService type: '%expectedLinkedServiceType;', current LinkedService type: Expected LinkedService type: '%currentLinkedServiceType;'.`

- **Cause** : Définition d'activité incorrecte.

- **Recommandation** : Vérifiez si la définition JSON de l’activité AzureMLExecutePipeline dispose des détails corrects sur le service lié.

### <a name="error-code-4112"></a>Code d’erreur : 4112

- **Message** : `AzureMLService linked service has invalid value for property '%propertyName;'.`

- **Cause** : Format incorrect ou définition manquante de la propriété « %propertyName; ».

- **Recommandation** : Vérifiez la propriété `%propertyName;` du service lié est définie avec les données qui conviennent.

### <a name="error-code-4121"></a>Code d’erreur : 4121

- **Message** : `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Cause** : Les informations d’identification utilisées pour accéder à Azure Machine Learning ont expiré.

- **Recommandation** : Vérifiez que les informations d’identification sont valides et réessayez.

### <a name="error-code-4122"></a>Code d’erreur : 4122

- **Message** : `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Cause** : Les informations d’identification fournies dans le service lié du service Azure Machine Learning ne sont pas valides ni autorisées pour l’opération.

- **Recommandation** : Vérifiez que les informations d’identification du service lié sont valides et autorisées pour accéder à Azure Machine Learning.

### <a name="error-code-4123"></a>Code d’erreur : 4123

- **Message** : `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Cause** : Les propriétés de l’activité, comme `pipelineParameters`, ne sont pas valides pour le pipeline Azure Machine Learning (ML).

- **Recommandation** : Vérifiez que la valeur des propriétés de l’activité correspond à la charge utile attendue du pipeline Azure ML publié spécifié dans le service lié.

### <a name="error-code-4124"></a>Code d’erreur : 4124

- **Message** : `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Cause** : Le point de terminaison de pipeline Azure ML publié n’existe pas.

- **Recommandation** : Vérifiez que le point de terminaison du pipeline Azure Machine Learning publié et spécifié dans le service lié existe dans Azure Machine Learning.

### <a name="error-code-4125"></a>Code d’erreur : 4125

- **Message** : `Request sent to Azure Machine Learning for operation '%operation;' failed with http status code '%statusCode;'. Error message from Azure Machine Learning: '%externalMessage;'.`

- **Cause** : Il existe une erreur de serveur sur Azure Machine Learning.

- **Recommandation** : Réessayez ultérieurement. Pour obtenir de l’aide, contactez l’équipe Azure Machine Learning si le problème persiste.

### <a name="error-code-4126"></a>Code d’erreur : 4126

- **Message** : `Azure ML pipeline run failed with status: '%amlPipelineRunStatus;'. Azure ML pipeline run Id: '%amlPipelineRunId;'. Please check in Azure Machine Learning for more error logs.`

- **Cause** : L’exécution du pipeline Azure ML a échoué.

- **Recommandation** : Pour obtenir des journaux d’erreurs supplémentaires et corriger le pipeline ML, consultez Azure Machine Learning.

## <a name="common"></a>Courant

### <a name="error-code-2103"></a>Code d’erreur : 2103

- **Message** : `Please provide value for the required property '%propertyName;'.`

- **Cause** : La valeur requise pour la propriété n’a pas été fournie.

- **Recommandation** : Spécifiez la valeur demandée dans le message, puis réessayez.

### <a name="error-code-2104"></a>Code d’erreur : 2104

- **Message** : `The type of the property '%propertyName;' is incorrect.`

- **Cause** : Le type de propriété fourni n’est pas correct.

- **Recommandation** : Corrigez le type de la propriété et réessayez.

### <a name="error-code-2105"></a>Code d’erreur : 2105

- **Message** : `An invalid json is provided for property '%propertyName;'. Encountered an error while trying to parse: '%message;'.`

- **Cause** : La valeur de la propriété n’est pas valide ou n’est pas au format attendu.

- **Recommandation** : Reportez-vous à la documentation de la propriété et vérifiez que la valeur fournie comprend le format et le type corrects.

### <a name="error-code-2106"></a>Code d’erreur : 2106

- **Message** : `The storage connection string is invalid. %errorMessage;`

- **Cause** : La chaîne de connexion pour le stockage n’est pas valide ou son format est incorrect.

- **Recommandation** : Accédez au Portail Azure, recherchez votre stockage, copiez et collez la chaîne de connexion dans votre service lié, puis réessayez.

### <a name="error-code-2108"></a>Code d’erreur : 2108

- **Message** : `Error calling the endpoint '%url;'. Response status code: '%code;'`

- **Cause** : La requête a échoué en raison d’un problème sous-jacent, par exemple un problème de connectivité réseau, un échec de DNS, un problème de validation de certificat de serveur ou l’expiration d’un délai.

- **Recommandation** : Utilisez Fiddler/Postman pour valider la requête.

### <a name="error-code-2110"></a>Code d’erreur : 2110

- **Message** : `The linked service type '%linkedServiceType;' is not supported for '%executorType;' activities.`

- **Cause** : Le service lié spécifié dans l’activité est incorrect.

- **Recommandation** : Assurez-vous que le type de service lié fait partie des types pris en charge pour l’activité. Par exemple, pour les activités HDI, le type de service lié peut être HDInsight ou HDInsightOnDemand.

### <a name="error-code-2111"></a>Code d’erreur : 2111

- **Message** : `The type of the property '%propertyName;' is incorrect. The expected type is %expectedType;.`

- **Cause** : Le type de la propriété fournie est incorrect.

- **Recommandation** : Corrigez le type de la propriété et réessayez.

### <a name="error-code-2112"></a>Code d’erreur : 2112

- **Message** : `The cloud type is unsupported or could not be determined for storage from the EndpointSuffix '%endpointSuffix;'.`

- **Cause** : Le type de cloud n’est pas pris en charge ou n’a pas pu être déterminé pour le stockage à partir du EndpointSuffix.

- **Recommandation** : Utilisez un stockage dans un autre cloud et réessayez.

### <a name="error-code-2128"></a>Code d’erreur : 2128

- **Message** : `No response from the endpoint. Possible causes: network connectivity, DNS failure, server certificate validation or timeout.`

- **Cause** : Connectivité réseau, échec de DNS, validation de certificat de serveur ou délai d’expiration.

- **Recommandation** : Vérifiez que le point de terminaison que vous essayez d’atteindre répond aux requêtes. Vous pouvez utiliser des outils tels que Fiddler/Postman.

## <a name="custom"></a>Custom

Le tableau suivant s’applique à Azure Batch.
 
### <a name="error-code-2500"></a>Code d’erreur : 2 500

- **Message** : `Hit unexpected exception and execution failed.`

- **Cause** : `Can't launch command, or the program returned an error code.`

- **Recommandation** : Vérifiez que le fichier exécutable existe. Si le programme a démarré, vérifiez que les fichiers *stdout.txt* et *stderr.txt* ont été chargés dans le compte de stockage. Il est conseillé d’inclure des journaux dans votre code à des fins de débogage.

### <a name="error-code-2501"></a>Code d’erreur : 2501

- **Message** : `Cannot access user batch account; please check batch account settings.`

- **Cause** : Le nom de pool ou la clé d’accès Batch fournis ne sont pas corrects.

- **Recommandation** : Vérifiez le nom du pool et la clé d’accès Batch dans le service lié.

### <a name="error-code-2502"></a>Code d’erreur : 2502

- **Message** : `Cannot access user storage account; please check storage account settings.`

- **Cause** : Le nom du compte de stockage ou la clé d’accès fournis ne sont pas corrects.

- **Recommandation** : Vérifiez le nom du compte de stockage et la clé d’accès dans le service lié.

### <a name="error-code-2504"></a>Code d’erreur : 2504

- **Message** : `Operation returned an invalid status code 'BadRequest'.`

- **Cause** : Trop de fichiers dans le `folderPath` de l’activité personnalisée. La taille totale de `resourceFiles` ne peut pas être supérieure à 32 768 caractères.

- **Recommandation** : Supprimez les fichiers inutiles ou placez-les dans un fichier zip et ajouter une commande de décompression pour les extraire.
   
   Par exemple, utilisez `powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::ExtractToDirectory($zipFile, $folder); }" ; $folder\yourProgram.exe`

### <a name="error-code-2505"></a>Code d’erreur : 2505

- **Message** : `Cannot create Shared Access Signature unless Account Key credentials are used.`

- **Cause** : Les activités personnalisées prennent uniquement en charge les comptes de stockage qui utilisent une clé d’accès.

- **Recommandation** : Voir la description de l’erreur.

### <a name="error-code-2507"></a>Code d’erreur : 2507

- **Message** : `The folder path does not exist or is empty: ...`

- **Cause** : Aucun fichier ne se trouve dans le compte de stockage situé au chemin spécifié.

- **Recommandation** : Le chemin du dossier doit contenir les fichiers exécutables que vous souhaitez exécuter.

### <a name="error-code-2508"></a>Code d’erreur : 2508

- **Message** : `There are duplicate files in the resource folder.`

- **Cause** : Il existe plusieurs fichiers du même nom dans différents sous-dossiers de folderPath.

- **Recommandation** : Les activités personnalisées aplatissent la structure de dossiers sous folderPath. Si vous devez conserver la structure de dossiers, compressez les fichiers dans un fichier zip, puis extrayez-les dans Azure Batch à l’aide d’une commande de décompression.
   
   Par exemple, utilisez `powershell.exe -nologo -noprofile -command "& { Add-Type -A 'System.IO.Compression.FileSystem'; [IO.Compression.ZipFile]::ExtractToDirectory($zipFile, $folder); }" ; $folder\yourProgram.exe`

### <a name="error-code-2509"></a>Code d’erreur : 2509

- **Message** : `Batch url ... is invalid; it must be in Uri format.`

- **Cause** : L’URL batch doit être semblable à `https://mybatchaccount.eastus.batch.azure.com`.

- **Recommandation** : Voir la description de l’erreur.

### <a name="error-code-2510"></a>Code d’erreur : 2510

- **Message** : `An error occurred while sending the request.`

- **Cause** : L’URL Batch n’est pas valide.

- **Recommandation** : Vérifiez l’URL Batch.
 
## <a name="hdinsight"></a>HDInsight

### <a name="error-code-206"></a>Code d’erreur : 206

- **Message** : `The batch ID for Spark job is invalid. Please retry your job.`

- **Cause** : Un problème interne s’est produit au niveau du service.

- **Recommandation** : Ce problème peut être temporaire. Réessayez ultérieurement.

### <a name="error-code-207"></a>Code d’erreur : 207

- **Message** : `Could not determine the region from the provided storage account. Please try using another primary storage account for the on demand HDI.`

- **Cause** : Une erreur interne s’est produite lors de la tentative de détermination de la région à partir du compte de stockage principal.

- **Recommandation** : Essayez un autre stockage. 

### <a name="error-code-208"></a>Code d’erreur : 208

- **Message** : `Service Principal or the MSI authenticator are not instantiated. Please consider providing a Service Principal in the HDI on demand linked service which has permissions to create an HDInsight cluster in the provided subscription and try again.`

- **Cause** : Une erreur interne s’est produite lors de la tentative de lecture du principal de service ou de l’instanciation de l’authentification MSI.

- **Recommandation** : Spécifiez un principal de service disposant des autorisations nécessaires pour créer un cluster HDInsight dans l’abonnement indiqué, puis réessayez. Vérifiez que les [identités managées sont correctement configurées](../hdinsight/hdinsight-managed-identities.md).


### <a name="error-code-2300"></a>Code d’erreur : 2300

- **Message** : `Failed to submit the job '%jobId;' to the cluster '%cluster;'. Error: %errorMessage;.`

- **Cause** : Le message d’erreur contient un message similaire à `The remote name could not be resolved.`. L’URI de cluster fourni n’est pas valide.

- **Recommandation** : Vérifiez que le cluster n’a pas été supprimé et que l’URI fourni est correct. Quand vous ouvrez l’URI dans un navigateur, vous devez voir l’interface utilisateur d’Ambari. Si le cluster se trouve dans un réseau virtuel, l’URI doit être l’URI privé. Pour l’ouvrir, utilisez une machine virtuelle qui fait partie du même réseau virtuel.

   Pour plus d’informations, consultez [Se connecter directement aux services Apache Hadoop](../hdinsight/hdinsight-plan-virtual-network-deployment.md#directly-connect-to-apache-hadoop-services).
 
 </br>

- **Cause** : Si le message d’erreur contient un message similaire à `A task was canceled.`, la soumission du travail expire.

- **Recommandation** : Il peut s’agir d’un problème de connectivité générale HDInsight ou d’un problème de connectivité réseau. Tout d’abord, vérifiez que l’interface utilisateur Ambari HDInsight est accessible à partir de n’importe quel navigateur. Vérifiez alors que vos informations d’identification sont toujours valides.
   
   Si vous utilisez le runtime d’intégration (IR) auto-hébergé, veillez à utiliser la machine virtuelle ou l’ordinateur où celui-ci est installé. Ensuite, renvoyez le travail à partir de Data Factory.

   Pour plus d’informations, lisez [Interface utilisateur web d’Ambari](../hdinsight/hdinsight-hadoop-manage-ambari.md#ambari-web-ui).

 </br>

- **Cause** : Lorsque le message d’erreur contient un message similaire à `User admin is locked out in Ambari` ou `Unauthorized: Ambari user name or password is incorrect`, les informations d’identification pour HDInsight ne sont pas correctes ou ont expiré.

- **Recommandation** : Corrigez les informations d’identification et redéployez le service lié. Vérifiez avant tout que les informations d’identification fonctionnent sur HDInsight en ouvrant l’URI du cluster sur n’importe quel navigateur et essayez de vous connecter. Si elles ne fonctionnent pas, vous pouvez les réinitialiser à partir du portail Azure.

   Pour le cluster ESP, vous pouvez [réinitialiser le mot de passe via la réinitialisation de mot de passe en libre-service](../active-directory/user-help/active-directory-passwords-update-your-own-password.md).

 </br>

- **Cause** : Lorsque le message d’erreur contient un message similaire à `502 - Web server received an invalid response while acting as a gateway or proxy server`, cette erreur est retournée par le service HDInsight.

- **Recommandation** : Une erreur 502 est survenue lors de l’interruption du processus du Serveur Ambari. Vous pouvez redémarrer les services Ambari en redémarrant le nœud principal.

    1. Connectez-vous à l’un de vos nœuds sur HDInsight à l’aide de SSH.
    1. Identifiez votre hôte de nœud principal actif en exécutant `ping headnodehost`.
    1. Connectez-vous à votre nœud principal actif, sur lequel se trouve le serveur Ambari, à l’aide de SSH. 
    1. Redémarrez le nœud principal actif.

       Pour plus d’informations, consultez la documentation de résolution des problèmes d’Azure HDInsight. Par exemple :

       * [Erreur 502 de l’interface utilisateur Ambari](https://hdinsight.github.io/ambari/ambari-ui-502-error.html)
       * [RpcTimeoutException pour le serveur Apache Spark Thrift](../hdinsight/spark/apache-spark-troubleshoot-rpctimeoutexception.md)
       * [Résolution des erreurs de passerelle incorrecte dans Application Gateway](../application-gateway/application-gateway-troubleshooting-502.md).

 </br>

- **Cause** : Lorsque le message d’erreur contient un message similaire à `Unable to service the submit job request as templeton service is busy with too many submit job requests` ou `Queue root.joblauncher already has 500 applications, cannot accept submission of application`, un trop grand nombre de travaux sont envoyées à HDInsight en même temps.

- **Recommandation** : Limitez le nombre de travaux concurrents soumis à HDInsight. Renseignez-vous sur la simultanéité des activités Data Factory si les travaux sont envoyés par la même activité. Modifiez les déclencheurs pour que les exécutions de pipeline simultanées soient réparties au fil du temps.

   Reportez-vous à la [documentation HDInsight](../hdinsight/hdinsight-hadoop-templeton-webhcat-debug-errors.md) pour ajuster `templeton.parallellism.job.submit` comme le suggère l’erreur.

### <a name="error-code-2301"></a>Code d’erreur : 2301

- **Message** : `Could not get the status of the application '%physicalJobId;' from the HDInsight service. Received the following error: %message;. Please refer to HDInsight troubleshooting documentation or contact their support for further assistance.`

- **Cause** : Le service ou le cluster HDInsight présente des problèmes.

- **Recommandation** : Cette erreur survient lorsque ADF ne reçoit pas de réponse de la part du cluster HDInsight lorsqu’il tente de demander le statut du travail en cour d’exécution. Le problème doit se situé sur le cluster lui-même ou le service HDInsight doit être en panne.

   Consultez la documentation Résolution des problèmes HDInsight sur https://docs.microsoft.com/azure/hdinsight/hdinsight-troubleshoot-guide ou contactez le support client pour obtenir de l’aide.

### <a name="error-code-2302"></a>Code d’erreur : 2302

- **Message** : `Hadoop job failed with exit code '%exitCode;'. See '%logPath;/stderr' for more details. Alternatively, open the Ambari UI on the HDI cluster and find the logs for the job '%jobId;'. Contact HDInsight team for further support.`

- **Cause** : Le travail a été envoyé au cluster HDI et a échoué à ce stade.

- **Recommandation** : 

 1. Vérifiez l’interface utilisateur Ambari :
    1. Vérifiez que tous les services sont en cours d’exécution.
    1. Dans l’interface utilisateur Ambari, consultez la section Alerte de votre tableau de bord.
       1. Pour plus d’informations sur les alertes et les résolutions des alertes, consultez [Gestion et surveillance d’un cluster](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
    1. Passez en revue la mémoire YARN. Si la mémoire YARN est élevée, le traitement de vos tâches peut être retardé. Si vous ne disposez pas de ressources suffisantes pour prendre en charge votre application/travail Spark, mettez à l’échelle le cluster pour vous assurer qu’il dispose de suffisamment de mémoire et de cœurs. 
 1. Exécutez un exemple de travail.
    1. Si vous exécutez le même travail sur le serveur principal HDInsight, vérifiez qu’il a réussi. Pour obtenir des exemples d’exécutions, consultez [Exécuter les exemples MapReduce inclus dans HDInsight](../hdinsight/hadoop/apache-hadoop-run-samples-linux.md) 
 1. Si le travail échoue encore sur HDInsight, vérifiez les journaux des applications et les informations, fournies pour apporter de l’aide :
    1. Vérifiez que le travail a été envoyé sur YARN. Si le travail n’a pas été envoyé sur YARN, consultez `--master yarn`.
    1. Si l’application a terminé l’exécution, collectez l’heure de début et l’heure de fin de l’application YARN. Si l’application n’a pas terminé l’exécution, collectez l’heure de début/heure de lancement.
    1. Vérifiez et collectez le journal des applications avec `yarn logs -applicationId <Insert_Your_Application_ID>`.
    1. Vérifiez et recueillez les journaux Gestionnaire des ressources journaux dans le répertoire `/var/log/hadoop-yarn/yarn`.
    1. Si ces étapes ne suffisent pas à résoudre le problème, contactez l’équipe Azure HDInsight pour obtenir de l’aide et fournissez les journaux et les horodateurs ci-dessus.

### <a name="error-code-2303"></a>Code d’erreur : 2303

- **Message** : `Hadoop job failed with transient exit code '%exitCode;'. See '%logPath;/stderr' for more details. Alternatively, open the Ambari UI on the HDI cluster and find the logs for the job '%jobId;'. Try again or contact HDInsight team for further support.`

- **Cause** : Le travail a été envoyé au cluster HDI et a échoué à ce stade.

- **Recommandation** : 

 1. Vérifiez l’interface utilisateur Ambari :
    1. Vérifiez que tous les services sont en cours d’exécution.
    1. Dans l’interface utilisateur Ambari, consultez la section Alerte de votre tableau de bord.
       1. Pour plus d’informations sur les alertes et les résolutions des alertes, consultez [Gestion et surveillance d’un cluster](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
    1. Passez en revue la mémoire YARN. Si la mémoire YARN est élevée, le traitement de vos tâches peut être retardé. Si vous ne disposez pas de ressources suffisantes pour prendre en charge votre application/travail Spark, mettez à l’échelle le cluster pour vous assurer qu’il dispose de suffisamment de mémoire et de cœurs. 
 1. Exécutez un exemple de travail.
    1. Si vous exécutez le même travail sur le serveur principal HDInsight, vérifiez qu’il a réussi. Pour obtenir des exemples d’exécutions, consultez [Exécuter les exemples MapReduce inclus dans HDInsight](../hdinsight/hadoop/apache-hadoop-run-samples-linux.md) 
 1. Si le travail échoue encore sur HDInsight, vérifiez les journaux des applications et les informations, fournies pour apporter de l’aide :
    1. Vérifiez que le travail a été envoyé sur YARN. Si le travail n’a pas été envoyé sur YARN, consultez `--master yarn`.
    1. Si l’application a terminé l’exécution, collectez l’heure de début et l’heure de fin de l’application YARN. Si l’application n’a pas terminé l’exécution, collectez l’heure de début/heure de lancement.
    1. Vérifiez et collectez le journal des applications avec `yarn logs -applicationId <Insert_Your_Application_ID>`.
    1. Vérifiez et recueillez les journaux Gestionnaire des ressources journaux dans le répertoire `/var/log/hadoop-yarn/yarn`.
    1. Si ces étapes ne suffisent pas à résoudre le problème, contactez l’équipe Azure HDInsight pour obtenir de l’aide et fournissez les journaux et les horodateurs ci-dessus.

### <a name="error-code-2304"></a>Code d’erreur : 2304

- **Message** : `MSI authentication is not supported on storages for HDI activities.`

- **Cause** : Le ou les services liés de stockage utilisés dans le service lié HDInsight (HDI) ou l’activité HDI sont configurés avec l’authentification MSI, ce qui n’est pas pris en charge.

- **Recommandation** : Spécifiez des chaînes de connexion complètes pour les comptes de stockage utilisés dans le service lié HDI ou l’activité HDI.

### <a name="error-code-2305"></a>Code d’erreur : 2305

- **Message** : `Failed to initialize the HDInsight client for the cluster '%cluster;'. Error: '%message;'`

- **Cause** : Les informations de connexion pour le cluster HDI sont incorrectes. L’utilisateur spécifié n’est pas autorisé à effectuer l’action requise ou le service HDInsight a rencontré des problèmes en répondant aux requêtes d’ADF.

- **Recommandation** : Vérifiez que les informations de l’utilisateur sont correctes et que l’interface utilisateur Ambari pour le cluster HDI peut être ouvert dans un navigateur depuis la machine virtuelle ou l’IR est installé (pour un IR auto-hébergé) ou qu’elle peut être ouverte à partir de n’importe quel ordinateur (s’il s’agit d’un IR Azure).

### <a name="error-code-2306"></a>Code d’erreur : 2306

- **Message** : `An invalid json is provided for script action '%scriptActionName;'. Error: '%message;'`

- **Cause** : Le JSON fourni pour l’action de script n’est pas valide.

- **Recommandation** : Le message d’erreur doit vous aider à identifier le problème. Corrigez la configuration json, puis réessayez.

   Pour plus d’informations, vérifiez [Service lié HDInsight à la demande](./compute-linked-services.md#azure-hdinsight-on-demand-linked-service).

### <a name="error-code-2310"></a>Code d’erreur : 2310

- **Message** : `Failed to submit Spark job. Error: '%message;'`

- **Cause** : ADF a tenté de créer un lot sur un cluster Spark à l’aide de l’API Livy (livy/batch), mais a reçu une erreur.

- **Recommandation** : Suivez le message d'erreur pour résoudre le problème. Si vous ne disposez pas d’informations suffisantes pour résoudre le problème, contactez l’équipe HDI et indiquez-lui les ID de lot et de travail qui se trouvent dans la sortie de l’exécution de l’activité, dans la page de supervision d’ADF. Pour poursuivre la résolution des problèmes, collectez le journal complet du traitement par lots.

   Pour plus d’informations sur la collecte de l’intégralité du journal, consultez [Obtenir le journal complet d’un programme de traitement par lots](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job).

### <a name="error-code-2312"></a>Code d’erreur : 2312

- **Message** : `Spark job failed, batch id:%batchId;. Please follow the links in the activity run Output from ADF Monitoring page to troubleshoot the run on HDInsight Spark cluster. Please contact HDInsight support team for further assistance.`

- **Cause** : Le travail a échoué sur le cluster HDInsight Spark.

- **Recommandation** : Suivez les liens figurant dans la sortie de l’exécution de l’activité (dans la page de supervision d’ADF) pour résoudre le problème d’exécution sur le cluster HDInsight Spark. Contactez l’équipe du support technique HDInsight pour obtenir une aide supplémentaire.

   Pour plus d’informations sur la collecte de l’intégralité du journal, consultez [Obtenir le journal complet d’un programme de traitement par lots](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job).

### <a name="error-code-2313"></a>Code d’erreur : 2313

- **Message** : `The batch with ID '%batchId;' was not found on Spark cluster. Open the Spark History UI and try to find it there. Contact HDInsight support for further assistance.`

- **Cause** : Le lot a été supprimé du cluster HDInsight Spark.

- **Recommandation** : Résolvez les problèmes de lots sur le cluster HDInsight Spark. Contactez le support HDInsight pour obtenir une aide supplémentaire. 

   Pour plus d’informations sur la collecte de l’intégralité du journal, consultez [Obtenir le journal complet d’un programme de traitement par lots](/rest/api/hdinsightspark/hdinsight-spark-batch-job#get-the-full-log-of-a-batch-job) et partager le journal complet avec la prise en charge de HDInsight pour obtenir de l’aide.

### <a name="error-code-2328"></a>Code d’erreur : 2328

- **Message** : `Failed to create the on demand HDI cluster. Cluster or linked service name: '%clusterName;', error: '%message;'`

- **Cause** : Le message d’erreur doit fournir des informations détaillées sur la cause du problème.

- **Recommandation** : Le message d’erreur doit vous aider à résoudre le problème.

### <a name="error-code-2329"></a>Code d’erreur : 2329

- **Message** : `Failed to delete the on demand HDI cluster. Cluster or linked service name: '%clusterName;', error: '%message;'`

- **Cause** : Le message d’erreur doit fournir des informations détaillées sur la cause du problème.

- **Recommandation** : Le message d’erreur doit vous aider à résoudre le problème.

### <a name="error-code-2331"></a>Code d’erreur : 2331

- **Message** : `The file path should not be null or empty.`

- **Cause** : Le chemin de fichier spécifié est vide.

- **Recommandation** : Spécifiez un chemin de fichier existant.

### <a name="error-code-2340"></a>Code d’erreur : 2340

- **Message** : `HDInsightOnDemand linked service does not support execution via SelfHosted IR. Your IR name is '%IRName;'. Please select an Azure IR instead.`

- **Cause** : Le service lié HDInsightOnDemand ne prend pas en charge l’exécution par le biais d’un runtime d’intégration auto-hébergé.

- **Recommandation** : Veuillez sélectionner un IR Azure et réessayez.

### <a name="error-code-2341"></a>Code d’erreur : 2341

- **Message** : `HDInsight cluster URL '%clusterUrl;' is incorrect, it must be in URI format and the scheme must be 'https'.`

- **Cause** : L’URL spécifiée n’est pas au bon format.

- **Recommandation** : Corrigez l’URL de cluster et réessayez.

### <a name="error-code-2342"></a>Code d’erreur : 2342

- **Message** : `Failed to connect to HDInsight cluster: '%errorMessage;'.`

- **Cause** : Les informations d’identification spécifiées sont incorrectes pour le cluster, un problème de connexion ou de configuration réseau s’est produit ou le runtime d’intégration rencontre des problèmes de connexion au cluster.

- **Recommandation** : 
    1. Vérifiez que les informations d’identification sont correctes en ouvrant l’interface utilisateur Ambari du cluster HDInsight dans un navigateur.
    1. Si le cluster se trouve dans un réseau virtuel (VNet) et que le runtime d’intégration auto-hébergé est en cours d’utilisation, l’URL HDI doit être l’URL privée dans les réseaux virtuels, ce qui signifie que le nom du cluster doit être suivi de « -int ».
    
       Par exemple, remplacez `https://mycluster.azurehdinsight.net/` par `https://mycluster-int.azurehdinsight.net/`. Notez le `-int` après `mycluster`, mais avant `.azurehdinsight.net`
    1. Si le cluster se trouve dans un réseau virtuel, que le runtime d’intégration auto-hébergé est en cours d’utilisation, que l’URL privée a été utilisée et que la connexion a encore échoué, cela signifie que la machine virtuelle sur laquelle l’IR est installé a rencontré des problèmes de connexion à HDI. 
    
       Connectez-vous à la machine virtuelle où l’IR est installé et ouvrez l’interface utilisateur Ambari dans un navigateur. Utilisez l’URL privée pour le cluster. Cette connexion doit fonctionner à partir du navigateur. Dans le cas contraire, contactez l’équipe du support technique HDInsight pour obtenir une aide supplémentaire.
    1. Si l’IR auto-hébergée n’est pas utilisée, le cluster HDI doit être accessible publiquement. Ouvrez l’interface utilisateur Ambari dans un navigateur et vérifiez qu’elle est ouverte. En cas de problème avec le cluster ou les services correspondants, contactez l’équipe du support technique HDInsight pour obtenir de l’aide.

       L’URL du cluster HDI utilisé dans le service lié ADF doit être accessible à l’IR ADF (auto-hébergée ou Azure) afin que la connexion test passe et que les exécutions fonctionnent. Cet état peut être vérifié en ouvrant l’URL à partir du navigateur soit à partir de la machine virtuelle, soit à partir d’un ordinateur public.

### <a name="error-code-2343"></a>Code d’erreur : 2343

- **Message** : `User name and password cannot be null or empty to connect to the HDInsight cluster.`

- **Cause** : Le nom d’utilisateur ou le mot de passe est vide.

- **Recommandation** : Fournissez les informations d’identification correctes pour vous connecter à HDI et réessayez.

### <a name="error-code-2345"></a>Code d’erreur : 2345

- **Message** : `Failed to read the content of the hive script. Error: '%message;'`

- **Cause** : Le fichier script n’existe pas ou ADF ne peut pas se connecter à l’emplacement du script.

- **Recommandation** : Vérifiez que le script existe et que le service lié associé dispose des informations d’identification correctes pour la connexion.

### <a name="error-code-2346"></a>Code d’erreur : 2346

- **Message** : `Failed to create ODBC connection to the HDI cluster with error message '%message;'.`

- **Cause** : ADF a essayé d’établir une connexion ODBC (Open Database Connectivity) au cluster HDI et a échoué avec une erreur.

- **Recommandation** : 

   1. Vérifiez que vous avez correctement configuré votre connexion ODBC/Java Database Connectivity (JDBC).
      1. Pour JDBC, si vous utilisez le même réseau virtuel, vous pouvez obtenir cette connexion à partir de :<br>
        `Hive -> Summary -> HIVESERVER2 JDBC URL`
      1. Pour vous assurer que vous disposez de la configuration JDBC appropriée, consultez [Interroger Apache Hive via le pilote JDBC dans HDInsight](../hdinsight/hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
      1. Pour Open Database (ODB), consultez [Didacticiel : Interroger Apache Hive avec ODBC et PowerShell](../hdinsight/interactive-query/apache-hive-query-odbc-driver-powershell.md) pour vous assurer que vous disposez de la configuration correcte. 
   1. Vérifiez que Hiveserver2, Hive Metastore et Hiveserver2 interactive sont actifs et opérationnels. 
   1. Vérifiez l’interface utilisateur Ambari :
      1. Vérifiez que tous les services sont en cours d’exécution.
      1. Dans l’interface utilisateur Ambari, consultez la section Alerte de votre tableau de bord.
         1. Pour plus d’informations sur les alertes et les résolutions des alertes, consultez [Gestion et surveillance d’un cluster](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
   1. Si ces étapes ne suffisent pas pour résoudre le problème, contactez l’équipe Azure HDInsight.

### <a name="error-code-2347"></a>Code d’erreur : 2347

- **Message** : `Hive execution through ODBC failed with error message '%message;'.`

- **Cause** : ADF a envoyé le script Hive pour exécution au cluster HDI par le biais d’une connexion ODBC et le script a échoué sur HDI.

- **Recommandation** : 

   1. Vérifiez que vous avez correctement configuré votre connexion ODBC/Java Database Connectivity (JDBC).
      1. Pour JDBC, si vous utilisez le même réseau virtuel, vous pouvez obtenir cette connexion à partir de :<br>
        `Hive -> Summary -> HIVESERVER2 JDBC URL`
      1. Pour vous assurer que vous disposez de la configuration JDBC appropriée, consultez [Interroger Apache Hive via le pilote JDBC dans HDInsight](../hdinsight/hadoop/apache-hadoop-connect-hive-jdbc-driver.md).
      1. Pour Open Database (ODB), consultez [Didacticiel : Interroger Apache Hive avec ODBC et PowerShell](../hdinsight/interactive-query/apache-hive-query-odbc-driver-powershell.md) pour vous assurer que vous disposez de la configuration correcte. 
   1. Vérifiez que Hiveserver2, Hive Metastore et Hiveserver2 interactive sont actifs et opérationnels. 
   1. Vérifiez l’interface utilisateur Ambari :
      1. Vérifiez que tous les services sont en cours d’exécution.
      1. Dans l’interface utilisateur Ambari, consultez la section Alerte de votre tableau de bord.
         1. Pour plus d’informations sur les alertes et les résolutions des alertes, consultez [Gestion et surveillance d’un cluster](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.5.0/managing-and-monitoring-ambari/content/amb_predefined_alerts.html).
   1. Si ces étapes ne suffisent pas pour résoudre le problème, contactez l’équipe Azure HDInsight.

### <a name="error-code-2348"></a>Code d’erreur : 2348

- **Message** : `The main storage has not been initialized. Please check the properties of the storage linked service in the HDI linked service.`

- **Cause** : Les propriétés du service lié de stockage ne sont pas définies correctement.

- **Recommandation** : Seules les chaînes de connexion complètes sont prises en charge dans le service lié de stockage principal pour les activités HDI. Vérifiez que vous n’utilisez pas d’autorisations ni d’applications MSI.

### <a name="error-code-2350"></a>Code d’erreur : 2350

- **Message** : `Failed to prepare the files for the run '%jobId;'. HDI cluster: '%cluster;', Error: '%errorMessage;'`

- **Cause** : Les informations d’identification spécifiées pour la connexion au stockage qui doit héberger les fichiers sont incorrectes, ou les fichiers n’existent pas à cet emplacement.

- **Recommandation** : Cette erreur se produit lorsque ADF prépare les activités HDI et tente de copier des fichiers vers le stockage principal avant de soumettre le travail à HDI. Vérifiez que les fichiers existent à l’emplacement spécifié et que la connexion au stockage est correcte. Les activités HDI d’ADF ne prennent pas en charge l’authentification MSI sur les comptes de stockage associés aux activités HDI. Assurez-vous donc que ces services liés disposent de clés complètes ou utilisent Azure Key Vault.

### <a name="error-code-2351"></a>Code d’erreur : 2351

- **Message** : `Could not open the file '%filePath;' in container/fileSystem '%container;'.`

- **Cause** : Le fichier n’existe pas au chemin spécifié.

- **Recommandation** : Vérifiez que le fichier existe réellement et que le service lié dont les informations de connexion pointent vers ce fichier dispose des informations d’identification correctes.

### <a name="error-code-2352"></a>Code d’erreur : 2352

- **Message** : `The file storage has not been initialized. Please check the properties of the file storage linked service in the HDI activity.`

- **Cause** : Les propriétés du service lié de stockage de fichiers ne sont pas définies correctement.

- **Recommandation** : Vérifiez que les propriétés du service lié de stockage de fichiers sont correctement configurées.

### <a name="error-code-2353"></a>Code d’erreur : 2353

- **Message** : `The script storage has not been initialized. Please check the properties of the script storage linked service in the HDI activity.`

- **Cause** : Les propriétés du service lié de stockage de scripts ne sont pas définies correctement.

- **Recommandation** : Vérifiez que les propriétés du service lié de stockage de scripts sont correctement configurées.

### <a name="error-code-2354"></a>Code d’erreur : 2354

- **Message** : `The storage linked service type '%linkedServiceType;' is not supported for '%executorType;' activities for property '%linkedServicePropertyName;'.`

- **Cause** : Le type de service lié de stockage n’est pas pris en charge par l’activité.

- **Recommandation** : Vérifiez que le service lié sélectionné fait partie des types pris en charge pour l’activité. Les activités HDI prennent en charge les services liés AzureBlobStorage et AzureBlobFSStorage.

   Pour plus d’informations, lisez [Comparer les options de stockage à utiliser avec les clusters Azure HDInsight](../hdinsight/hdinsight-hadoop-compare-storage-options.md).

### <a name="error-code-2355"></a>Code d’erreur : 2355

- **Message** : `The '%value' provided for commandEnvironment is incorrect. The expected value should be an array of strings where each string has the format CmdEnvVarName=CmdEnvVarValue.`

- **Cause** : La valeur fournie pour `commandEnvironment` est incorrecte.

- **Recommandation** : Vérifiez que la valeur fournie est semblable à celle-ci :
 
    ``` \"commandEnvironment\": [
    \"variableName=variableValue\"
    ]
    ```

    Vérifiez également que chaque variable s’affiche une seule fois dans la liste.

### <a name="error-code-2356"></a>Code d’erreur : 2356

- **Message** : `The commandEnvironment already contains a variable named '%variableName;'.`

- **Cause** : La valeur fournie pour `commandEnvironment` est incorrecte.

- **Recommandation** : Vérifiez que la valeur fournie est semblable à celle-ci :
 
    ``` \"commandEnvironment\": [
    \"variableName=variableValue\"
    ]
    ```

    Vérifiez également que chaque variable s’affiche une seule fois dans la liste.

### <a name="error-code-2357"></a>Code d’erreur : 2357

- **Message** : `The certificate or password is wrong for ADLS Gen 1 storage.`

- **Cause** : Les informations d’identification spécifiées sont incorrectes.

- **Recommandation** : Vérifiez les informations de connexion dans ADLS Gen1 lié au service et vérifiez que le test de connexion réussit.

### <a name="error-code-2358"></a>Code d’erreur : 2358

- **Message** : `The value '%value;' for the required property 'TimeToLive' in the on demand HDInsight linked service '%linkedServiceName;' has invalid format. It should be a timespan between '00:05:00' and '24:00:00'.`

- **Cause** : La valeur spécifiée pour la propriété requise `TimeToLive` n’est pas valide. 

- **Recommandation** : Mettez à jour la valeur pour qu’elle soit incluse dans la plage suggérée, puis réessayez.

### <a name="error-code-2359"></a>Code d’erreur : 2359

- **Message** : `The value '%value;' for the property 'roles' is invalid. Expected types are 'zookeeper', 'headnode', and 'workernode'.`

- **Cause** : La valeur spécifiée pour la propriété `roles` n’est pas valide.

- **Recommandation** : Mettez à jour la valeur pour qu’elle corresponde à l’une des valeurs suggérées, puis réessayez.

### <a name="error-code-2360"></a>Code d’erreur : 2360

- **Message** : `The connection string in HCatalogLinkedService is invalid. Encountered an error while trying to parse: '%message;'.`

- **Cause** : La chaîne de connexion spécifiée pour `HCatalogLinkedService` n’est pas valide.

- **Recommandation** : Mettez à jour la valeur pour qu’elle corresponde à une chaîne de connexion Azure SQL correcte, puis réessayez.

### <a name="error-code-2361"></a>Code d’erreur : 2361

- **Message** : `Failed to create on demand HDI cluster. Cluster name is '%clusterName;'.`

- **Cause** : La création du cluster a échoué, et ADF n’a pas reçu d’erreur du service HDInsight.

- **Recommandation** : Ouvrez le Portail Azure, recherchez la ressource HDI avec le nom spécifié et vérifiez l’état du provisionnement. Contactez l’équipe du support technique HDInsight pour obtenir une aide supplémentaire.

### <a name="error-code-2362"></a>Code d’erreur : 2362

- **Message** : `Only Azure Blob storage accounts are supported as additional storages for HDInsight on demand linked service.`

- **Cause** : Le stockage supplémentaire spécifié n’était pas un stockage d’objets blob Azure.

- **Recommandation** : Spécifiez un compte de stockage blob Azure en tant que stockage supplémentaire pour le service lié HDInsight à la demande.

## <a name="web-activity"></a>Activité web

### <a name="error-code-2128"></a>Code d’erreur : 2128

- **Message** : `No response from the endpoint. Possible causes: network connectivity, DNS failure, server certificate validation or timeout.`

- **Cause** : Ce problème est dû à une connectivité réseau, à une défaillance DNS, à une validation de certificat de serveur ou à un délai d’attente.

- **Recommandation** : Vérifiez que le point de terminaison que vous essayez d’atteindre répond aux requêtes. Vous pouvez utiliser des outils tels que **Fiddler/Postman**.

### <a name="error-code-2108"></a>Code d’erreur : 2108

- **Message** : `Error calling the endpoint '%url;'. Response status code: '%code;'`

- **Cause** : La requête a échoué en raison d’un problème sous-jacent, par exemple un problème de connectivité réseau, un échec de DNS, un problème de validation de certificat de serveur ou l’expiration d’un délai.

- **Recommandation** : Utilisez Fiddler/Postman pour valider la requête.

#### <a name="more-details"></a>Détails supplémentaires
Pour utiliser **Fiddler** en vue de créer une session HTTP de l’application web supervisée :

1. Téléchargez, installez puis ouvrez [Fiddler](https://www.telerik.com/download/fiddler).

1. Si votre application web utilise le protocole HTTPS, accédez à **Tools** > **Fiddler Options** > **HTTPS** (Outils > Options Fiddler > HTTPS).

   1. Dans l’onglet HTTPS, sélectionnez **Capturer les HTTPS CONNECT** et **Déchiffrer le trafic HTTPS**.

      ![Options Fiddler](media/data-factory-troubleshoot-guide/fiddler-options.png)

1. Si votre application utilise des certificats TLS/SSL, ajoutez le certificat Fiddler à votre appareil.

   Accédez à : **Outils** > **Options Fiddler** > **HTTPS** > **Actions** > **Exporter le certificat racine vers le Bureau**.

1. Désactivez la capture en accédant à **File** > **Capture Traffic** (Fichier > Capturer le trafic). Vous pouvez aussi appuyer sur **F12**.

1. Videz le cache de votre navigateur afin que tous les éléments présents soient supprimés et doivent être téléchargés à nouveau.

1. Créez une requête :

1. Sélectionnez l’onglet **Composer** (Éditeur).

   1. Définissez la méthode HTTP et l’URL.
 
   1. Si nécessaire, ajoutez des en-têtes et un corps de requête.

   1. Sélectionnez **Exécuter**.

1. Réactivez la capture du trafic, puis terminez la transaction problématique dans votre page.

1. Accédez à : **Fichier** > **Enregistrer** > **Toutes les sessions**.

Pour plus d’informations, consultez [Bien démarrer avec Fiddler](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/ConfigureFiddler).

## <a name="general"></a>Général

### <a name="activity-stuck-issue"></a>Problème d’activité bloquée

Si vous remarquez que l’exécution de l’activité dure beaucoup plus longtemps que vos exécutions normales et qu’elle ne progresse pratiquement pas, elle peut être bloquée. Vous pouvez essayer de l’annuler et de faire une nouvelle tentative pour voir si cela est utile. S’il s’agit d’une activité de copie, vous pouvez en savoir plus sur l’analyse des performances et la résolution des problèmes en consultant la section [Résoudre les problèmes de performances de l’activité de copie](copy-activity-performance-troubleshooting.md) ; s’il s’agit d’un flux de données, consultez la section [Performances des flux de données de mappage](concepts-data-flow-performance.md) et le guide de réglage.

### <a name="payload-is-too-large"></a>La charge utile est trop grande

**Message d’erreur :** `The payload including configurations on activity/dataSet/linked service is too large. Please check if you have settings with very large value and try to reduce its size.`

**Cause :** La charge utile pour chaque exécution d’activité comprend la configuration de l’activité, le ou les jeux de données associés et les configurations du ou des services liés le cas échéant, et une petite partie des propriétés système générées par type d’activité. La limite de cette taille de charge utile est de 896 ko, comme indiqué dans [Limites de Data Factory](../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits).

**Recommandation :** Vous atteignez cette limite probablement parce que vous transmettez une ou plusieurs grandes valeurs de paramètres à partir de la sortie d’une activité en amont ou d’un élément externe, surtout si vous transmettez des données réelles entre les activités dans le flux de contrôle. Vérifiez si vous pouvez réduire la taille des valeurs de paramètres élevées, ou paramétrez votre logique de pipeline pour éviter de transmettre de telles valeurs entre les activités et pour les gérer plutôt à l’intérieur de l’activité.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire, essayez les ressources suivantes :

* [Blog Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
* [Demandes de fonctionnalités Data Factory](https://feedback.azure.com/forums/270578-data-factory)
* [Forum Stack Overflow pour Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
* [Informations Twitter sur Data Factory](https://twitter.com/hashtag/DataFactory)
* [Vidéos Azure](https://azure.microsoft.com/resources/videos/index/)
* [Page de questions Microsoft Q&R](/answers/topics/azure-data-factory.html)
