---
title: Exceptions Service Bus dans Azure Resource Manager | Microsoft Docs
description: Liste des exceptions de Service Bus signalées par Azure Resource Manager et actions suggérées.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: a0b0338da0f002c7b667748ffd2bf5a40c91c580
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85336966"
---
# <a name="service-bus-resource-manager-exceptions"></a>Exceptions Service Bus dans Azure Resource Manager

Cet article répertorie les exceptions générées lors de l’interaction avec Azure Service Bus à l’aide d’Azure Resource Manager via des modèles ou des appels directs.

> [!IMPORTANT]
> Ce document est fréquemment mis à jour. Revenez plus tard pour suivre l’évolution de la situation.

Vous trouverez ci-dessous les différentes exceptions/erreurs signalées par Azure Resource Manager.

## <a name="error-bad-request"></a>Erreur : Demande incorrecte

« Demande incorrecte » signifie que la validation de la demande reçue par Resource Manager a échoué.

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Demande incorrecte | 40000 | SubCode=40000. La propriété *'nom de propriété'* ne peut pas être définie lors de la création d’une file d’attente car l’espace de noms *'nom de l’espace de noms'* utilise le niveau 'De base'. Cette opération est prise en charge uniquement aux niveaux ’Standard’ ou ’Premium’. | Au niveau De base d’Azure Service Bus, il n’est pas possible de définir ou de mettre à jour les propriétés suivantes : <ul> <li> RequiresDuplicateDetection </li> <li> AutoDeleteOnIdle </li> <li>RequiresSession</li> <li>DefaultMessageTimeToLive </li> <li> DuplicateDetectionHistoryTimeWindow </li> <li> EnableExpress </li> <li> ForwardTo </li> <li> Rubriques </li> </ul> | Pour utiliser cette fonctionnalité, envisagez de passer du niveau De base au niveau Standard ou Premium. |
| Demande incorrecte | 40000 | SubCode=40000. La valeur de la propriété ’requiresDuplicateDetection’ d’une file d’attente (ou d’une rubrique) existante ne peut pas être modifiée. | La détection des doublons doit être activée/désactivée au moment de la création de l’entité. Il n’est pas possible de modifier le paramètre de configuration de la détection des doublons après la création. | Pour activer la détection des doublons dans une file d’attente ou rubrique créées précédemment, vous pouvez créer une nouvelle file d’attente ou rubrique avec détection des doublons, puis la transférer de la file d’attente d’origine vers la nouvelle file d’attente ou rubrique. |
| Demande incorrecte | 40000 | SubCode=40000. La valeur spécifiée 16384 n’est pas valide. La propriété ’MaxSizeInMegabytes’ doit avoir l’une des valeurs suivantes : 1024;2048;3072;4096;5120. | La valeur MaxSizeInMegabytes n’est pas valide. | Assurez-vous que MaxSizeInMegabytes a l’une des valeurs suivantes : 1024, 2048, 3072, 4096, 5120. |
| Demande incorrecte | 40000 | SubCode=40000. Impossible de modifier le partitionnement pour la file d’attente ou rubrique. | Impossible de modifier le partitionnement pour l’entité. | Créez une entité (file d’attente ou rubrique) et activez les partitions. | 
| Demande incorrecte | Aucun | L’espace de noms *'nom de l’espace de noms'* n’existe pas. | L’espace de noms n’existe pas dans votre abonnement Azure. | Pour résoudre cette erreur, essayez ce qui suit <ul> <li> Assurez-vous que l’abonnement Azure est correct. </li> <li> Assurez-vous que l’espace de noms existe. </li> <li> Vérifiez que le nom de l’espace de noms est correct (exempt de faute d’orthographe ou de chaîne Null). </li> </ul> | 
| Demande incorrecte | 40400 | SubCode=40400. L’entité de destination de transfert automatique n’existe pas. | La destination de l’entité de destination de transfert automatique n’existe pas. | L’entité de destination (file d’attente ou rubrique) doit exister avant la création de la source. Réessayez après avoir créé l’entité de destination. |
| Demande incorrecte | 40000 | SubCode=40000. Le temps de verrouillage spécifié dépasse le maximum autorisé de « 5 » minutes. | La durée pendant laquelle un message peut être verrouillé doit être comprise entre 1 (minimum) et 5 (maximum) minutes. | Assurez-vous que le temps de verrouillage spécifié est compris entre 1 et 5 minutes. |
| Demande incorrecte | 40000 | SubCode=40000. Les propriétés DelayedPersistence et RequiresDuplicateDetection ne peuvent pas être activées ensemble. | Les entités pour lesquelles la détection des doublons est activée doivent être persistantes ; la persistance ne peut donc pas être retardée. | En savoir plus sur la [Détection des doublons](duplicate-detection.md) |
| Demande incorrecte | 40000 | SubCode=40000. La valeur de la propriété RequiresSession d’une file d’attente existante ne peut pas être modifiée. | La prise en charge des sessions doit être activée au moment de la création de l’entité. Une fois les sessions créées, vous ne pas les activer/désactiver sur une entité existante (file d’attente ou abonnement) | Supprimez et recréez une file d’attente (ou un abonnement) avec la propriété « RequiresSession » activée. |
| Demande incorrecte | 40000 | SubCode=40000. 'URI_PATH' contient des caractères que Service Bus n’autorise pas. Les segments d’entité ne peuvent contenir que des lettres, des chiffres, des points (.), des tirets (-) et des traits de soulignement (_). | Les segments d’entité ne peuvent contenir que des lettres, des chiffres, des points (.), des tirets (-) et des traits de soulignement (_). Tout autre caractère entraîne l’échec de la requête. | Assurez-vous qu’il n’existe pas de caractères non valides dans le chemin d’accès à l’URI. |


## <a name="error-code-429"></a>Code d’erreur : 429

Comme en HTTP, le « code d’erreur 429 » signifie « Trop de demandes ». Cela implique que la ressource spécifique (espace de noms) est limitée en raison d’un trop grand nombre de demandes (ou d’opérations conflictuelles) sur cette ressource.

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| 429 | 50004 | SubCode=50004. La demande a été arrêtée parce que l’espace de noms *votre espace de noms* est limité. | Cette condition d’erreur est atteinte lorsque le nombre de demandes entrantes dépasse la limitation de la ressource. | Patientez quelques secondes, puis réessayez. <br/> <br/> En savoir plus sur les [quotas](service-bus-quotas.md) et les [limites de demandes d’Azure Resource Manager](../azure-resource-manager/management/request-limits-and-throttling.md).|
| 429 | 40901 | SubCode=40901. Une autre opération conflictuelle est en cours. | Une autre opération conflictuelle est en cours sur la même ressource/entité | Attendez la fin de l’opération en cours avant de réessayer. |
| 429 | 40900 | SubCode=40900. Conflit. Vous demandez une opération qui n’est pas autorisée dans l’état actuel de la ressource. | Cette condition peut être atteinte lorsque plusieurs demandes sont faites pour effectuer les opérations sur la même entité (file d’attente, rubrique, abonnement ou règle) en même temps. | Patientez quelques secondes, puis réessayez |
| 429 | 40901 | Une demande sur l’entité *'nom de l’entité'* est en conflit avec une autre demande | Une autre opération conflictuelle est en cours sur la même ressource/entité | Attendez la fin de l’opération précédente avant de réessayer |
| 429 | 40901 | Une autre demande de mise à jour est en cours pour l’entité *'nom de l’entité'* . | Une autre opération conflictuelle est en cours sur la même ressource/entité | Attendez la fin de l’opération précédente avant de réessayer |
| 429 | Aucun | Un conflit de ressources s’est produit. Une autre opération conflictuelle est peut-être en cours. S’il s’agit d’une nouvelle tentative d’exécution d’une opération qui a échoué, cela signifie qu’un nettoyage en arrière-plan est en attente. Réessayez plus tard. | Cette condition peut être atteinte quand il existe une opération en attente sur la même entité. | Attendez la fin de l’opération précédente avant de réessayer. |


## <a name="error-code-not-found"></a>Code d’erreur : Introuvable

Cette classe d’erreurs indique que la ressource est introuvable.

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Introuvable | Aucun | L’entité *'nom de l’entité'* est introuvable. | L’entité sur laquelle porte l’opération est introuvable. | Vérifiez si l’entité existe, puis réessayez d’effectuer l’opération. |
| Introuvable | Aucun | Introuvable. L’opération n’existe pas. | L’opération que vous essayez d’effectuer n’existe pas. | Vérifiez l’opération, puis réessayez. |
| Introuvable | Aucun | La demande entrante n’est pas reconnue en tant que demande Put de stratégie d’espace de noms. | Le corps de la demande entrante est Null et ne peut donc pas être exécuté en tant que demande Put. | Vérifiez le corps de la demande pour vous assurer qu’il n’est pas Null. | 
| Introuvable | Aucun | L’entité de messagerie *'nom de l’entité'* est introuvable. | L’entité sur laquelle vous tentez d’exécuter l’opération est introuvable. | Vérifiez que l’entité existe, puis réessayez l’opération. |

## <a name="error-code-internal-server-error"></a>Code d’erreur : Erreur interne du serveur

Cette classe d’erreurs indique qu’une erreur de serveur interne s’est produite.

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Erreur interne du serveur | 50000 | SubCode=50000. Erreur interne du serveur| Il existe diverses raisons possibles. Voici quelques-uns des symptômes : <ul> <li> Le corps/la demande du client est endommagé et génère une erreur. </li> <li> La demande du client a expiré en raison de problèmes de traitement sur le service. </li> </ul> | Pour résoudre ce problème : <ul> <li> Vérifiez que les paramètres de la demande ne sont pas Null ou incorrects. </li> <li> Relancez la requête. </li> </ul> |

## <a name="error-code-unauthorized"></a>Code d’erreur : Non autorisé

Cette classe d’erreurs indique une absence d’autorisation d’exécution de la commande.

| Code d'erreur | Sous-code d’erreur | Message d’erreur | Description | Recommandation |
| ---------- | ------------- | ------------- | ----------- | -------------- |
| Non autorisé | Aucun | Opération non valide sur l’espace de noms secondaire. Celui-ci est en lecture seule. | L’opération a été effectuée sur l’espace de noms secondaire, qui est configuré en lecture seule. | Réessayez la commande sur l’espace de noms principal. [En savoir plus sur l’espace de noms secondaire](service-bus-geo-dr.md). |
| Non autorisé | Aucun | MissingToken : Aucun en-tête d’autorisation n’a été trouvé. | Cette erreur se produit lorsque l’autorisation a des valeurs Null ou incorrectes. | Vérifiez que la valeur du jeton mentionnée dans l’en-tête d’autorisation est correcte et non Null. |