---
title: 'ML Studio (classique) : Codes d’erreur d’API REST - Azure'
description: Ces codes d’erreur peuvent être renvoyés par une opération effectuée sur un service web Azure Machine Learning.
keywords: ''
services: machine-learning
author: likebupt
ms.author: keli19
editor: cgronlun
ms.assetid: 0923074b-3728-439d-a1b8-8a7245e39be4
ms.service: machine-learning
ms.subservice: studio
ms.topic: reference
ms.date: 11/16/2016
ms.openlocfilehash: 24e0fa530b370782af730a2383df1240bbf5c859
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308661"
---
# <a name="azure-machine-learning-studio-classic-rest-api-error-codes"></a>Codes d’erreur de l’API REST Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Les codes d’erreur suivants peuvent être retournés par une opération effectuée sur un service web Azure Machine Learning Studio (classique).
 
## <a name="badargument-http-status-code-400"></a>BadArgument (code d’état HTTP :  400)
 
Argument non valide fourni.
 
Cette classe d’erreurs indique qu’un des arguments fournis est non valide. Il peut s’agir d’une information d’identification ou d’un emplacement de stockage Azure relatifs à du contenu transmis au service web. Examinez le champ du code d’erreur dans la section des détails afin d’identifier l’argument spécifique non valide.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| BadParameterValue | La valeur de paramètre fournie ne satisfait pas la règle de paramètre |
| BadSubscriptionId | L’ID d’abonnement utilisé pour la notation n’est pas celui de la ressource |
| BadVersionCall | Un paramètre de version non valide a été transmis pendant l’appel d’API : {0}. Consultez la page d’aide de l’API afin de transmettre la version appropriée, puis réessayez. |
| BatchJobInputsNotSpecified | Les entrées requises suivantes n’ont pas été spécifiées avec la requête : {0}. Vérifiez que l’ensemble des données d’entrées sont spécifiées, puis réessayez. |
| BatchJobInputsTooManySpecified | Le nombre d’entrées spécifiées dans la requête était supérieur à la valeur définie dans le service. Liste des entrées acceptées : {0}. Vérifiez que toutes les données d’entrée sont spécifiées correctement, puis réessayez. |
| BlobNameTooLong | Le chemin d’accès au stockage Blob Azure fourni pour les résultats de diagnostics est trop long : {0}. Raccourcissez le chemin d’accès, puis réessayez. |
| BlobNotFound | Impossible d’accéder au stockage Blob Azure fourni : {0}.  Message d’erreur Azure : {1}. |
| ContainerIsEmpty | Aucun nom de conteneur de stockage Azure n’a été fourni. Fournissez un nom de conteneur valide, puis réessayez. |
| ContainerSegmentInvalid | Nom du conteneur non valide. Fournissez un nom de conteneur valide, puis réessayez. |
| ContainerValidationFailed | La validation du conteneur d’objets blob a échoué et renvoyé une erreur : {0}. |
| DataTypeNotSupported | Le type de données fourni n’est pas pris en charge. Fournissez un type de données valide, puis réessayez. |
| DuplicateInputInBatchCall | La requête de lots n’est pas valide. Il n’est pas possible de spécifier simultanément des entrées simples et multiples. Supprimez l’un de ces éléments de la requête, puis réessayez. |
| ExpiryTimeInThePast | Le délai d’expiration fourni fait partie du passé : {0}. FOurnissez une heure d’expiration postérieure en heure UTC, puis réessayez. Pour rendre l’expiration non valide, définissez l’heure d’expiration sur NULL. |
| IncompleteSettings | Les paramètres de diagnostic sont incomplets. |
| InputBlobRelativeLocationInvalid | Aucun nom d’objet blob de stockage Azure fourni. Fournissez un nom d’objet blob valide, puis réessayez. |
| InvalidBlob | Spécification blob pour objet blob non valide : {0}. Assurez-vous du caractère approprié de cette spécification de chaîne de connexion/chemin d’accès relatif ou jeton SAS, puis réessayez. |
| InvalidBlobConnectionString | La chaîne de connexion indiquée pour l’un des objets blob d’entrée/de sortie n’est pas valide : {0}. Corrigez ce problème, puis réessayez. |
| InvalidBlobExtension | La référence d’objet blob {0} présente une extension de fichier manquante ou non valide. Les extensions de fichier prises en charge pour ce type de sortie sont les suivantes : « {1} ». |
| InvalidInputNames | Les noms d’entrée de service indiqués dans la requête ne sont pas valides : {0}. Mappez les données d’entrée sur les entrées appropriées de service, puis réessayez. |
| InvalidOutputOverrideName | Le nom de remplacement de sortie {0} n’est pas valide. Ce service ne présente aucun nœud de sortie portant ce nom. Communiquez un nom approprié de nœud de sortie de remplacement (contenu sensible à la casse). |
| InvalidQueryParameter | Le paramètre de requête « {0} » n’est pas valide. {1} |
| MissingInputBlobInformation | Informations manquantes sur l’objet blob de stockage Azure. Fournissez une chaîne de connexion valide et un chemin d’accès relatif ou un URI, puis réessayez. |
| MissingJobId | Aucun ID de travail fourni. Un ID de travail est renvoyé lors de la première soumission d’un travail. Vérifiez qu’il s’agit de l’ID de travail approprié, puis réessayez. |
| MissingKeys | Aucune clé fournie, ou les clés primaire ou secondaire ne sont pas fournies. |
| MissingModelPackage | Aucun ID de package de modèle ou package de modèle fourni. Fournissez un ID de package de modèle ou un package de modèle valide, puis réessayez. |
| MissingOutputOverrideSpecification | La requête ne contient pas la spécification d’objets blob pour le remplacement de sortie {0}. Spécifiez un emplacement valide d’objet blob avec la requête, ou supprimez la spécification de sortie si aucun remplacement d’emplacement n’est souhaité. |
| MissingRequestInput | Le service web attend une entrée, mais aucune entrée n’a été fournie. Vérifiez que les entrées valides sont fournies en fonction des ports d’entrées publiés dans le modèle, puis réessayez. |
| MissingRequiredGlobalParameters | Les paramètres requis de service web ne sont pas tous fournis. Vérifiez que les paramètres attendus pour les modules sont corrects, puis réessayez. |
| MissingRequiredOutputOverrides | Lors de l’appel d’un point de terminaison de service chiffré, il est obligatoire de transmettre les remplacements de l’ensemble des sorties de service. Les remplacements pour ces sorties et à cet horodatage sont manquants : {0} |
| MissingWebServiceGroupId | Aucun ID de groupe de service web fourni. Fournissez un ID valide de groupe de service web, puis réessayez. |
| MissingWebServiceId | Aucun ID de service web fourni. Fournissez un ID valide de service web, puis réessayez. |
| MissingWebServicePackage | Aucun package de service web fourni. Fournissez un package valide de service web, puis réessayez. |
| MissingWorkspaceId | Aucun ID d’espace de travail fourni. Fournissez un ID valide d’espace de travail, puis réessayez. |
| ModelConfigurationInvalid | Configuration de modèle non valide dans le package de modèle. Vérifiez que la configuration du modèle contient la définition des points de terminaison de sortie, le point de terminaison d’erreur standard et le point de terminaison de sortie standard, puis réessayez. |
| ModelPackageIdInvalid | ID non valide de package de modèle. Vérifiez que l’ID du package de modèle est correct, puis réessayez. |
| RequestBodyInvalid | Aucun corps de requête fourni ou erreur dans la désérialisation du corps de requête. |
| RequestIsEmpty | Aucune requête fournie. Fournissez une requête valide, puis réessayez. |
| UnexpectedParameter | Paramètres inattendus requis. Vérifiez que tous les noms de paramètres sont correctement orthographiés, que seuls les paramètres attendus sont transmis, puis réessayez. |
| UnknownError | Erreur inconnue. |
| UserParameterInvalid | {0} |
| WebServiceConcurrentRequestRequirementInvalid | Impossible de modifier la configuration requise des requêtes simultanées pour le service web {0}. |
| WebServiceIdInvalid | L’ID de service web fourni n’est pas valide. L’ID du service web doit être une valeur GUID valide. |
| WebServiceTooManyConcurrentRequestRequirement | Impossible de définir sur plus de {0} la configuration requise des requêtes simultanées. |
| WebServiceTypeInvalid | Le type du service web fourni n’est pas valide. Vérifiez que le type du service web fourni est correct, puis réessayez. Types valides de service web : {0}. |
 
## <a name="baduserargument-http-status-code-400"></a>BadUserArgument (code d’état HTTP : 400)
 
L’argument utilisateur fourni n’est pas valide.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| InputMismatchError | Les données d’entrée ne correspondent pas au schéma du port d’entrée. |
| InputParseError | L’analyse du vecteur d’entrée a été mise en échec.  Vérifiez que le vecteur d’entrée présente le nombre approprié de colonnes et de types de données.  Informations supplémentaires : {0}. |
| MissingRequiredGlobalParameters | Les paramètres attendus par le service web sont manquants. Vérifiez que les paramètres requis attendus par le service web sont corrects, puis réessayez. |
| UnexpectedParameter | Vérifiez que seuls les paramètres requis attendus par le service web sont transmis, puis réessayez. |
| UserParameterInvalid | {0} |
 
## <a name="invalidoperation-http-status-code-400"></a>InvalidOperation (code d’état HTTP : 400)
 
La requête n’est pas valide dans le contexte actuel.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| CannotStartJob | Le travail ne peut pas être démarré, car il présente l’état {0}. |
| IncompatibleModel | Le modèle n’est pas compatible avec la version de la requête. La version de la requête prend uniquement en charge les modèles de sortie de tables de données uniques. |
| MultipleInputsNotAllowed | Le modèle n’autorise pas plusieurs sorties. |
 
## <a name="libraryexecutionerror-http-status-code-400"></a>LibraryExecutionError (code d’état HTTP : 400)
 
L’exécution du module a rencontré une erreur de bibliothèque interne.
 
 
## <a name="moduleexecutionerror-http-status-code-400"></a>ModuleExecutionError (code d’état HTTP : 400)
 
L’exécution du module a rencontré une erreur.
 
 
## <a name="webservicepackageerror-http-status-code-400"></a>WebServicePackageError (code d’état HTTP : 400)
 
Le package du service web n’est pas valide. Vérifiez que le package du service web est correct, plus réessayez.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| FormatError | Le package du service web est mal formé. Détails : {0} |
| RuntimesError | Le graphique du package de service web n’est pas valide. Détails : {0} |
| ValidationError | Le graphique du package de service web n’est pas valide. Détails : {0} |
 
## <a name="unauthorized-http-status-code-401"></a>Unauthorized (code d'état HTTP : 401)
 
La requête n’est pas autorisée à accéder à la ressource.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| AdminRequestUnauthorized | Non autorisé |
| ManagementRequestUnauthorized | Non autorisé |
| ScoreRequestUnauthorized | Informations d’identification non valides fournies. |
 
## <a name="notfound-http-status-code-404"></a>NotFound (code d’état HTTP : 404)
 
La ressource est introuvable.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| ModelPackageNotFound | Le package de modèle est introuvable. Vérifiez que l’ID du package de modèle est correct, puis réessayez. |
| WebServiceIdNotFoundInWorkspace | Le service web de cet espace de travail est introuvable. Les ID du service Web et de l’espace de travail ne correspondent pas. Vérifiez que le service web fourni fait partie de l’espace de travail, puis réessayez. |
| WebServiceNotFound | Le service web est introuvable. Vérifiez que l’ID du service web est correct, puis réessayez. |
| WorkspaceNotFound | L’espace de travail est introuvable. Vérifiez que l’ID de l’espace de travail est correct, puis réessayez. |
 
## <a name="requesttimeout-http-status-code-408"></a>RequestTimeout (code d’état HTTP : 408)
 
Impossible d'exécuter l'opération dans le temps imparti.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| RequestCanceled | La requête a été annulée par le client. |
| ScoreRequestTimeout | La requête d’exécution a expiré. |
 
## <a name="conflict-http-status-code-409"></a>Conflit (code d’état HTTP : 409)
 
La ressource existe déjà.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| ModelOutputMetadataMismatch | Le nom du paramètre de sortie n’est pas valide. Essayez d’utiliser le module de Éditeur de métadonnées pour renommer les colonnes, puis réessayez. |
 
## <a name="memoryquotaviolation-http-status-code-413"></a>MemoryQuotaViolation (code d’état HTTP : 413)
 
Le modèle a dépassé le quota de mémoire qui lui a été affecté.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| OutOfMemoryLimit | Le modèle a consommé plus de mémoire que son attribution. La capacité maximale de mémoire autorisée pour le modèle est de {0} Mo. Vérifiez que votre modèle ne présente pas de problèmes. |
 
## <a name="internalerror-http-status-code-500"></a>InternalError (code d’état HTTP : 500)
 
L’exécution a rencontré une erreur interne.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| AdminAuthenticationFailed |  |
| BackendArgumentError |  |
| BackendBadRequest |  |
| ClusterConfigBlobMisconfigured |  |
| ContainerProcessTerminatedWithSystemError | Le processus du conteneur s’est arrêté avec une erreur système. |
| ContainerProcessTerminatedWithUnknownError | Le processus du conteneur s’est arrêté avec une erreur inconnue. |
| ContainerValidationFailed | La validation du conteneur d’objets blob a échoué et renvoyé une erreur : {0}. |
| DeleteWebServiceResourceFailed |  |
| ExceptionDeserializationError |  |
| FailedGettingApiDocument |  |
| FailedStoringWebService |  |
| InvalidMemoryConfiguration | InvalidMemoryConfiguration, ConfigValue : {0} |
| InvalidResourceCacheConfiguration |  |
| InvalidResourceDownloadConfiguration |  |
| InvalidWebServiceResources |  |
| MissingTaskInstance | Aucun argument fourni. Vérifiez que les arguments valides sont transmis, puis réessayez. |
| ModelPackageInvalid |  |
| ModuleExecutionFailed |  |
| ModuleLoadFailed |  |
| ModuleObjectCloneFailed |  |
| OutputConversionFailed |  |
| PortDataTypeNotSupported | L’ID de port ={0} présente un type de données non pris en charge : {1}. |
| ResourceDownload |  |
| ResourceLoadFailed |  |
| ServiceUrisNotFound |  |
| SwaggerGeneration | La génération de l’interface utilisateur Swagger a échoué. Détails : {0} |
| UnexpectedScoreStatus |  |
| UnknownBackendErrorResponse |  |
| UnknownError |  |
| UnknownJobStatusCode | Code inconnu d’état du travail {0}. |
| UnknownModuleError |  |
| UpdateWebServiceResourceFailed |  |
| WebServiceGroupNotFound |  |
| WebServicePackageInvalid | InvalidWebServicePackage, Détails : {0} |
| WorkerAuthorizationFailed |  |
| WorkerUnreachable |  |
 
## <a name="internalerrorsystemlowonmemory-http-status-code-500"></a>InternalErrorSystemLowOnMemory (code d’état HTTP : 500)
 
L’exécution a rencontré une erreur interne. La capacité de mémoire du système est faible. Réessayez.
 
 
## <a name="modelpackageformaterror-http-status-code-500"></a>ModelPackageFormatError (code d’état HTTP : 500)
 
Le package du modèle n’est pas valide. Vérifiez que le package du modèle fourni est correct, puis réessayez.
 
 
## <a name="webservicepackageinternalerror-http-status-code-500"></a>WebServicePackageInternalError (code d’état HTTP : 500)
 
Le package du service web n’est pas valide. Vérifiez que le package web fourni est correct, puis réessayez.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| ModuleError | Le graphique du package de service web n’est pas valide. Détails : {0} |
 
## <a name="initializingcontainers-http-status-code-503"></a>InitializingContainers (code d’état HTTP : 503)
 
La requête ne peut pas s’exécuter, car les conteneurs sont en cours d’initialisation.
 
 
## <a name="serviceunavailable-http-status-code-503"></a>ServiceUnavailable (code d’état HTTP : 503)
 
Le service est temporairement indisponible.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| NoMoreResources | Aucune ressource n’est disponible pour la requête. |
| RequestThrottled | La requête a été limitée pour le point de terminaison {0}. L’accès concurrentiel maximal pour le point de terminaison est {1}. |
| TooManyConcurrentRequests | Un nombre trop important de requêtes simultanées ont été envoyées. |
| TooManyHostsBeingInitialized | Un nombre trop important d’hôtes sont initialisés au même moment. Envisagez de limiter/de réessayer. |
| TooManyHostsBeingInitializedPerModel | Un nombre trop important d’hôtes sont initialisés au même moment. Envisagez de limiter/de réessayer. |
 
## <a name="gatewaytimeout-http-status-code-504"></a>GatewayTimeout (code d’état HTTP : 504)
 
Impossible d'exécuter l'opération dans le temps imparti.
 
| Code d'erreur | Message utilisateur |
| ---------- |--------------|
| BackendInitializationTimeout | L’initialisation du service web n’a pas pu s’exécuter dans le temps imparti. |
| BackendScoreTimeout | L’exécution de la requête du service web n’a pas pu s’exécuter dans le temps imparti. |
