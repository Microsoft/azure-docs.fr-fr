---
title: "Résoudre l'erreur : Le runtime d'Azure Functions est inaccessible"
description: Découvrez comment résoudre les problèmes relatifs à un compte de stockage non valide.
ms.topic: article
ms.date: 09/05/2018
ms.openlocfilehash: 0b6778a08bf04367f2a0ef10f7cd4fe29a52dd61
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579009"
---
# <a name="troubleshoot-error-azure-functions-runtime-is-unreachable"></a>Résoudre l'erreur : « Le runtime d'Azure Functions est inaccessible »

Cet article vous aide à résoudre la chaîne d'erreur suivante lorsqu'elle apparaît sur le portail Azure :

> « Erreur : Le runtime d'Azure Functions est inaccessible. Cliquez ici pour plus d'informations sur la configuration du stockage. »

Ce problème se produit lorsque le runtime d'Azure Functions ne démarre pas. En général, ce problème survient lorsque l'application de fonction n'a plus accès à son compte de stockage. Pour plus d'informations, consultez [Exigences relatives au compte de stockage](./functions-create-function-app-portal.md#storage-account-requirements).

Le reste de cet article vous aide à détecter le problème pour les causes suivantes de cette erreur, notamment la façon d’identifier et de résoudre chaque cas.

## <a name="storage-account-was-deleted"></a>Le compte de stockage a été supprimé

Chaque application de fonction nécessite un compte de stockage afin de fonctionner. Si ce compte est supprimé, votre fonction ne s'exécutera pas.

Commencez par rechercher le nom de votre compte de stockage dans les paramètres de votre application. `AzureWebJobsStorage` ou `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` contient le nom de votre compte de stockage, encapsulé dans une chaîne de connexion. Pour plus d'informations, consultez [Informations de référence sur les paramètres d'application d'Azure Functions](./functions-app-settings.md#azurewebjobsstorage).

Recherchez votre compte de stockage sur le portail Azure pour voir s'il existe toujours. S'il a été supprimé, recréez-le et remplacez vos chaînes de connexion de stockage. Votre code de fonction est perdu et vous devez le redéployer.

## <a name="storage-account-application-settings-were-deleted"></a>Les paramètres d'application du compte de stockage ont été supprimés

À l'étape précédente, si vous ne trouvez pas de chaîne de connexion de compte de stockage, cela signifie probablement qu'elle a été supprimée ou écrasée. La suppression des paramètres d'application survient généralement lorsque vous utilisez des emplacements de déploiement ou des scripts Azure Resource Manager pour définir les paramètres d'application.

### <a name="required-application-settings"></a>Paramètres de l’application obligatoires

* Requis :
    * [`AzureWebJobsStorage`](./functions-app-settings.md#azurewebjobsstorage)
* Obligatoire pour les fonctions des plans Consommation et Premium :
    * [`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`](./functions-app-settings.md)
    * [`WEBSITE_CONTENTSHARE`](./functions-app-settings.md)

Pour plus d'informations, consultez [Informations de référence sur les paramètres d'application d'Azure Functions](./functions-app-settings.md).

### <a name="guidance"></a>Assistance

* N’activez pas le « paramètre d’emplacement » pour ces paramètres. Si vous échangez les emplacements de déploiement, l'application de fonction s'arrête.
* Ne modifiez pas ces paramètres dans le cadre de déploiements automatisés.
* Ces paramètres doivent être fournis et valides au moment de la création. Un déploiement automatisé sans ces paramètres produit une application de fonction qui ne s’exécute pas et ce, même si les paramètres sont ajoutés ultérieurement.

## <a name="storage-account-credentials-are-invalid"></a>Les informations d'identification du compte de stockage ne sont pas valides

Les chaînes de connexion de compte de stockage mentionnées précédemment doivent être mises à jour si vous régénérez les clés de stockage. Pour plus d'informations sur la gestion des clés de stockage, consultez [Créer un compte de stockage Azure](../storage/common/storage-account-create.md).

## <a name="storage-account-is-inaccessible"></a>Le compte de stockage est inaccessible

Votre application de fonction doit être en mesure d’accéder au compte de stockage. Les principaux problèmes qui empêchent une application de fonction d'accéder à un compte de stockage sont les suivants :

* L'application de fonction est déployée sur votre environnement ASE (App Service Environment) sans les règles de réseau appropriées pour autoriser le trafic vers et depuis le compte de stockage.

* Le pare-feu du compte de stockage est activé et non configuré pour autoriser le trafic vers et depuis des fonctions. Pour plus d’informations, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../storage/common/storage-network-security.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="daily-execution-quota-is-full"></a>Le quota d'exécution quotidien a été atteint

Si vous avez configuré un quota d’exécution quotidien, votre application de fonction sera temporairement désactivée et la plupart des commandes du portail seront indisponibles. 

Pour vérifier le quota sur le [portail Azure](https://portal.azure.com), sélectionnez **Fonctionnalités de la plateforme** > **Paramètres de l'application de fonction** dans votre application de fonction. Si vous avez dépassé le **quota d'utilisation quotidienne** que vous avez défini, le message suivant s'affiche :

  > « L'application de fonction a atteint le quota d'utilisation quotidienne et a été arrêtée pour les prochaines 24 heures. »

Pour résoudre ce problème, supprimez ou augmentez le quota d'utilisation quotidienne et redémarrez votre application. Sinon, l'exécution de votre application sera bloquée jusqu'au lendemain.

## <a name="app-is-behind-a-firewall"></a>Application située derrière un pare-feu

Le runtime de votre fonction peut être inaccessible pour l'une des raisons suivantes :

* Votre application de fonction est hébergée dans un [environnement ASE (App Service Environment) à équilibrage de charge interne](../app-service/environment/create-ilb-ase.md) et elle est configurée pour bloquer le trafic Internet entrant.

* Votre application de fonction présente des [restrictions sur les adresses IP entrantes](functions-networking-options.md#inbound-access-restrictions) qui sont configurées pour bloquer l'accès à Internet. 

Le portail Azure appelle directement l'application en cours d'exécution pour extraire la liste des fonctions et passe des appels HTTP vers le point de terminaison Kudu. Les paramètres au niveau de la plateforme, sous l'onglet **Fonctionnalités de la plateforme**, sont toujours disponibles.

Pour vérifier la configuration de votre environnement ASE (App Service Environment) :
1. Accédez au groupe de sécurité réseau du sous-réseau où réside l'environnement ASE.
1. Validez les règles de trafic entrant pour autoriser le trafic provenant de l'adresse IP publique de l'ordinateur sur lequel vous accédez à l'application. 
   
Vous pouvez également utiliser le portail à partir d'un ordinateur connecté au réseau virtuel où est exécutée votre application, ou à partir d'une machine virtuelle exécutée sur votre réseau virtuel. 

Pour plus d'informations sur la configuration des règles de trafic entrant, consultez la section « Groupes de sécurité réseau » de l'article [Considérations relatives à la mise en réseau pour un environnement ASE (App Service Environment)](../app-service/environment/network-info.md#network-security-groups).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la supervision de vos applications de fonction :

> [!div class="nextstepaction"]
> [Superviser Azure Functions](functions-monitoring.md)
