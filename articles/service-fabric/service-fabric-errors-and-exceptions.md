---
title: Exceptions FabricClient fréquemment levées
description: Décrit les exceptions et les erreurs courantes qui peuvent être générées par les API FabricClient lors des opérations de gestion des applications et des clusters.
author: georgewallace
ms.topic: conceptual
ms.date: 06/20/2018
ms.author: gwallace
ms.openlocfilehash: 7f3c3e072a3a2e4f7723f84b2c70ba0d0ddb9d03
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258842"
---
# <a name="common-exceptions-and-errors-when-working-with-the-fabricclient-apis"></a>Exceptions et erreurs courantes lorsque vous travaillez avec les API FabricClient
Les API [FabricClient](/dotnet/api/system.fabric.fabricclient) permettent aux administrateurs de clusters et d’applications d’effectuer des tâches administratives au niveau d’une application, d’un service ou d’un cluster Service Fabric. Par exemple : déploiement d’une application, mise à niveau, suppression, vérification de l'intégrité d’un cluster, ou test d’un service. Les développeurs d'applications et les administrateurs de clusters peuvent utiliser les API FabricClient pour développer des outils permettant de gérer le cluster et les applications Service Fabric.

Il existe de nombreux types d'opérations qui peuvent être effectuées à l'aide de FabricClient.  Chaque méthode peut générer des exceptions pour les erreurs dues à une saisie incorrecte, des erreurs d'exécution ou des problèmes d'infrastructure temporaires.  Consultez la documentation de référence sur les API pour identifier les exceptions générées par une méthode spécifique. Quelques exceptions peuvent cependant être levées par de nombreuses autres API [FabricClient](/dotnet/api/system.fabric.fabricclient). Le tableau suivant répertorie les exceptions communes aux API FabricClient.

| Exception | Générée lorsque |
| --- |:--- |
| [System.Fabric.FabricObjectClosedException](/dotnet/api/system.fabric.fabricobjectclosedexception) |L’objet [FabricClient](/dotnet/api/system.fabric.fabricclient) est à l’état fermé. Supprimez l’objet [FabricClient](/dotnet/api/system.fabric.fabricclient) que vous utilisez et instanciez un nouvel objet [FabricClient](/dotnet/api/system.fabric.fabricclient). |
| [System.TimeoutException](/dotnet/core/api/system.timeoutexception) |L'opération a expiré. [OperationTimedOut](/dotnet/api/system.fabric.fabricerrorcode) est renvoyé lorsque la durée de l’opération dépasse la valeur MaxOperationTimeout. |
| [System.UnauthorizedAccessException](/dotnet/core/api/system.unauthorizedaccessexception) |La vérification d'accès pour l'opération a échoué. E_ACCESSDENIED est renvoyée. |
| [System.Fabric.FabricException](/dotnet/api/system.fabric.fabricexception) |Une erreur d'exécution s'est produite lors de l'opération. N’importe quelle méthode FabricClient peut potentiellement générer une exception [FabricException](/dotnet/api/system.fabric.fabricexception). La propriété [ErrorCode](/dotnet/api/system.fabric.fabricexception.errorcode) indique la cause exacte de l’exception. Les codes d’erreur sont définis dans l’énumération [FabricErrorCode](/dotnet/api/system.fabric.fabricerrorcode). |
| [System.Fabric.FabricTransientException](/dotnet/api/system.fabric.fabrictransientexception) |L'opération a échoué en raison d'une condition d'erreur transitoire quelconque. Par exemple, une opération peut échouer si un quorum de réplicas est temporairement inaccessible. Les exceptions temporaires correspondent à des opérations ayant échoué et qui peuvent être relancées. |

Certaines erreurs [FabricErrorCode](/dotnet/api/system.fabric.fabricerrorcode) courantes peuvent être retournées dans une exception [FabricException](/dotnet/api/system.fabric.fabricexception) :

| Error | Condition |
| --- |:--- |
| CommunicationError |Une erreur de communication a provoqué l’échec de l'opération, retentez l'opération. |
| InvalidCredentialType |Le type d'informations d'identification n'est pas valide. |
| InvalidX509FindType |La valeur X509FindType n'est pas valide. |
| InvalidX509StoreLocation |L’emplacement du magasin X509 n'est pas valide. |
| InvalidX509StoreName |Le nom du magasin X509 n'est pas valide. |
| InvalidX509Thumbprint |La chaîne de l’empreinte de certificat X509 n'est pas valide. |
| InvalidProtectionLevel |Le niveau de protection n'est pas valide. |
| InvalidX509Store |Le magasin de certificats X509 ne peut pas être ouvert. |
| InvalidSubjectName |Le nom d'objet n'est pas valide. |
| InvalidAllowedCommonNameList |Le format de la chaîne de la liste de noms courants n'est pas valide. Ce doit être une liste séparée par des virgules. |
