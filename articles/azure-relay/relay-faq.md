---
title: FAQ sur Azure Relay | Documents Microsoft
description: Cet article fournit des réponses aux questions fréquemment posées sur le service Azure Relay.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 9d967d926c6ab59e027fe4d4cf98e8418a8ff9bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89299284"
---
# <a name="azure-relay-faqs"></a>FAQ sur Azure Relay

Cet article contient les réponses à certaines questions fréquemment posées sur [Azure Relay](https://azure.microsoft.com/services/service-bus/). Pour des informations générales concernant la tarification et le support technique d’Azure, consultez le [Forum aux questions sur le support technique Azure](https://azure.microsoft.com/support/faq/).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="general-questions"></a>Questions générales
### <a name="what-is-azure-relay"></a>Qu’est-ce qu’Azure Relay ?
Le [service Azure Relay](relay-what-is-it.md) facilite le déploiement de vos applications hybrides en vous aidant à exposer les services qui résident au sein d’un réseau d’entreprise au cloud public de façon plus sécurisée. Vous pouvez exposer les services sans ouvrir de connexion de pare-feu et sans devoir apporter de modifications intrusives à une infrastructure réseau d’entreprise.

### <a name="what-is-a-relay-namespace"></a>Qu’est-ce qu’un espace de noms Relay ?
A [espace de noms](relay-create-namespace-portal.md) est un conteneur d’étendue que vous pouvez utiliser pour adresser des ressources Relay au sein de votre application. Vous devez créer un espace de noms pour utiliser Relay. Il s’agit de l’une des premières étapes de la prise en main.

### <a name="what-happened-to-service-bus-relay-service"></a>Qu’est-il advenu du service Service Bus Relay ?
Le service anciennement nommé Service Bus Relay s’appelle désormais [Azure Relay](service-bus-relay-tutorial.md). Vous pouvez continuer à utiliser ce service comme vous en avez l’habitude. La fonctionnalité Connexions hybrides est une version mise à jour d’un service transplanté à partir Azure BizTalk Services. Les fonctionnalités Relais WCF et Connexions hybrides continuent d’être prises en charge.

## <a name="pricing"></a>Tarifs
Cette section contient les réponses à certaines questions fréquemment posées sur la structure de tarification de Relay. Pour des informations générales sur la tarification d’Azure, vous pouvez également consulter le [Forum aux questions sur le support technique Azure](https://azure.microsoft.com/support/faq/). Pour obtenir toutes les informations sur la tarification de Relay, consultez [Détails de la tarification de Service Bus][Pricing overview].

### <a name="how-do-you-charge-for-hybrid-connections-and-wcf-relay"></a>Comment sont facturés les services Connexions hybrides et Relais WCF ?
Pour obtenir toutes les informations sur la tarification Relay, consultez le tableau [Connexions hybrides et relais WCF][Pricing overview] dans la page de détails de la tarification Service Bus. Outre les prix mentionnés sur cette page, vous êtes facturé pour les transferts de données associés aux sorties à l’extérieur du centre de données dans lequel votre application est déployée.

### <a name="how-am-i-billed-for-hybrid-connections"></a>Comment est facturé le service Connexions hybrides ?
Voici trois exemples de scénarios facturation pour les Connexions hybrides :

*   Scénario 1 :
    *   Vous avez un seul écouteur, tel qu’une instance du Gestionnaire de Connexions hybrides installée qui s’exécute en continu pendant tout le mois.
    *   Vous envoyez 3 Go de données sur la connexion durant le mois. 
    *   Le total des frais s’élève à 5 $.
*   Scénario 2 :
    *   Vous avez un seul écouteur, tel qu’une instance du Gestionnaire de Connexions hybrides installée qui s’exécute en continu pendant tout le mois.
    *   Vous envoyez 10 Go de données sur la connexion durant le mois.
    *   Le total des frais s’élève à 7,50 $. Ce montant se décompose en 5 $ pour la connexion et les 5 premiers Go de données + 2,50 $ pour les 5 Go de données supplémentaires.
*   Scénario 3 :
    *   Vous avez deux instances, A et B, du Gestionnaire de Connexions hybrides installées qui s’exécutent en continu pendant tout le mois.
    *   Vous envoyez 3 Go de données sur la connexion A durant le mois.
    *   Vous envoyez 6 Go de données sur la connexion B durant le mois.
    *   Le total des frais s’élève à 10,50 $. Ce montant se décompose en 5 $ pour la connexion A + 5 $ pour la connexion B + 0,50 $ (pour le sixième Go sur la connexion B).

Notez que les prix mentionnés dans les exemples sont applicables uniquement durant la période d’évaluation de Connexions hybrides. Les prix sont sujets à modification lors de la disponibilité générale de Connexions hybrides.

### <a name="how-are-hours-calculated-for-relay"></a>Comment les heures sont-elles calculées pour Relay ?

Relais WCF est disponible uniquement dans les espaces de noms de niveau Standard. Autrement, les tarifs et [quotas de connexion](../service-bus-messaging/service-bus-quotas.md) applicables aux relais restent inchangés. Cela signifie que les relais continueront d’être facturés sur la base du nombre de messages (pas d’opérations) et d’heures de relais. Pour plus d’informations, consultez le tableau [Connexions hybrides et relais WCF](https://azure.microsoft.com/pricing/details/service-bus/) dans la page de détails de tarification.

### <a name="what-if-i-have-more-than-one-listener-connected-to-a-specific-relay"></a>Que se passe-t-il si j’ai plus d’un écouteur connecté à un relais donné ?
Dans certains cas, un relais peut avoir plusieurs écouteurs connectés. Un relais est considéré comme ouvert quand au moins un écouteur y est connecté. L’ajout d’écouteurs à un relais ouvert entraîne des heures de relais supplémentaires. Le nombre d’expéditeurs de relais (clients qui appellent ou envoient des messages aux relais) connectés à un relais n’a aucune incidence sur le calcul des heures de relais.

### <a name="how-is-the-messages-meter-calculated-for-wcf-relays"></a>Comment le nombre de messages est-il calculé pour les relais WCF ?
(**Cela s’applique uniquement aux relais WCF. Les messages sont gratuits pour les Connexions hybrides.** )

Dans l’ensemble, les messages facturables pour les relais sont calculés conformément à la méthode décrite plus haut pour les entités réparties (files d’attente, rubriques et abonnements). Toutefois, il existe des différences importantes.

L’envoi d’un message à Azure Relay est traité comme un envoi « complet » à l’écouteur de relais qui reçoit le message. Il n’est pas traité comme une opération d’envoi à Azure Relay, suivie d’une remise à l’écouteur de relais. Un appel de service de type demande-réponse (jusqu’à 64 Ko) à un écouteur de relais entraîne deux messages facturables : l’un pour la demande et l’autre pour la réponse (en supposant que la taille de réponse est également inférieure ou égale à 64 Ko). Cela diffère de l’utilisation d’une file d’attente servant d’intermédiaire entre un client et un service. Si vous utilisez une file d’attente servant d’intermédiaire entre un client et un service, le même modèle de demande-réponse nécessite l’envoi d’une demande à la file d’attente, suivi d’une remise (ou retrait de file d’attente) au service à partir de la file d’attente. Ce processus est suivi de l’envoi d’une réponse à une autre file d’attente, et d’une remise (ou retrait de file d’attente) au client à partir de la file d’attente. Sur la base des mêmes hypothèses de taille (jusqu’à 64 Ko), le modèle de file d’attente servant d’intermédiaire entraîne 4 messages facturables. Pour implémenter le même modèle que celui que vous exécutez en utilisant un relais, vous êtes facturé pour deux fois le nombre de messages. Bien sûr, il existe des avantages à utiliser les files d’attente pour atteindre ce modèle, par exemple, la durabilité et le nivellement de charge. Ces avantages peuvent justifier les coûts supplémentaires.

Les relais ouverts à l’aide de la liaison WCF **netTCPRelay** traitent les messages non pas en tant que messages individuels, mais en tant que flux de données transitant par le système. Lorsque vous utilisez cette liaison, seuls l’expéditeur et l’écouteur peuvent voir la trame des messages envoyés et reçus. Pour les relais qui utilisent la liaison **netTCPRelay**, toutes les données sont traitées comme un flux pour le calcul des messages facturables. Dans ce cas, Service Bus calcule la quantité totale de données envoyées ou reçues via chaque relais sur une base de 5 minutes. Ensuite, il divise cette quantité totale de données par 64 Ko pour déterminer le nombre de messages facturables pour ce relais pendant cette période.

## <a name="quotas"></a>Quotas
| Nom du quota | Étendue |  Notes | Valeur |
| --- | --- | --- | --- |
| Écouteurs simultanés sur un relais |Entité |Les demandes suivantes de connexions supplémentaires sont rejetées et le code appelant reçoit une exception. |25 |
| Connexions Relay simultanées pour tous les points de terminaison Relay dans un espace de noms de service |Espace de noms |- |5 000 |
| Points de terminaison Relay par espace de noms de service |Espace de noms |- |10 000 |
| Taille de message pour les relais [NetOnewayRelayBinding](/dotnet/api/microsoft.servicebus.netonewayrelaybinding) et [NetEventRelayBinding](/dotnet/api/microsoft.servicebus.neteventrelaybinding) |Espace de noms |Les messages entrants dont la taille dépasse ces quotas sont rejetés et le code appelant reçoit une exception. |64 Ko |
| Taille de message pour les relais [HttpRelayTransportBindingElement](/dotnet/api/microsoft.servicebus.httprelaytransportbindingelement) et [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) |Espace de noms |Aucune limite concernant la taille des messages. |Illimité |

### <a name="does-relay-have-any-usage-quotas"></a>Relay a-t-il des quotas d’utilisation ?
Par défaut, pour n’importe quel service cloud, Microsoft définit un quota d’utilisation agrégée mensuel qui est calculé avec tous les abonnements d’un client. Nous sommes conscients que vos besoins peuvent parfois dépasser ces limites. Vous pouvez contacter le service clientèle à tout moment pour nous faire part de vos besoins afin que nous puissions ajuster ces limites de manière appropriée. Pour Service Bus, les quotas d’utilisation d’agrégation sont les suivants :

* 5 milliards de messages
* 2 millions d’heures de relais

Si nous nous réservons le droit de désactiver un compte dépassant son quota mensuel d’utilisation, notre politique consiste à envoyer une notification par e-mail, et à tenter à plusieurs reprises de contacter le client avant toute action. Les clients qui dépassent ces quotas restent responsables de frais de dépassement occasionnés.

### <a name="naming-restrictions"></a>Restrictions concernant l’attribution de noms
Le nom de l’espace de noms du relais doit compter de 6 à 50 caractères.

## <a name="subscription-and-namespace-management"></a>Gestion des abonnements et des espaces de noms
### <a name="how-do-i-migrate-a-namespace-to-another-azure-subscription"></a>Migration d’un espace de noms vers un autre abonnement Azure

Pour déplacer un espace de noms d’un abonnement Azure vers un autre, vous pouvez utiliser le [portail](https://portal.azure.com) ou des commandes PowerShell. Pour pouvoir déplacer un espace de noms vers un autre abonnement, il faut que l’espace de noms soit déjà actif. Il est également impératif que l’utilisateur qui exécute les commandes soit un administrateur des abonnements source et cible.

#### <a name="azure-portal"></a>Portail Azure

Pour utiliser le portail Azure afin de migrer des espaces de noms Azure Relay d’un abonnement vers un autre, voir [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md#use-the-portal). 

#### <a name="powershell"></a>PowerShell

Pour utiliser PowerShell afin de déplacer un espace de noms d’un abonnement Azure vers un autre, utilisez la séquence de commandes suivante. Pour exécuter cette opération, l’espace de noms doit être déjà actif, et l’utilisateur qui exécute les commandes PowerShell doit être administrateur des abonnements source et cible.

```azurepowershell-interactive
# Create a new resource group in the target subscription.
Select-AzSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzResourceGroup -Name 'targetRG' -Location 'East US'

# Move the namespace from the source subscription to the target subscription.
Select-AzSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzResource -DestinationResourceGroupName 'targetRG' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## <a name="troubleshooting"></a>Dépannage
### <a name="what-are-some-of-the-exceptions-generated-by-azure-relay-apis-and-suggested-actions-you-can-take"></a>Quelles sont les exceptions générées par les API Azure Relay et les actions suggérées ?
Pour obtenir une description des exceptions courantes et des actions suggérées, consultez [Exceptions Relay][Relay exceptions].

### <a name="what-is-a-shared-access-signature-and-which-languages-can-i-use-to-generate-a-signature"></a>Qu’est-ce qu’une signature d’accès partagé et quels langages puis-je utiliser pour générer une signature ?
Les signatures d’accès partagé (SAP) sont des mécanismes d’authentification basés sur des hachages sécurisés SHA-256 ou des URI. Pour plus d’informations sur la génération de vos propres signatures en Node.js, PHP, Python, Java, C et C#, consultez [Authentification de Service Bus avec les signatures d’accès partagé][Shared Access Signatures].

### <a name="is-it-possible-to-allow-only-some-relay-endpoints"></a>Est-il possible d’autoriser uniquement certains points de terminaison de relais ?
Oui. Le client de relais établit des connexions au service Azure Relay en utilisant des noms de domaine complets (FQDN). Cela permet aux clients d’ajouter une entrée pour `*.servicebus.windows.net` sur les pare-feu qui prennent en charge les listes d’approbation de DNS.

## <a name="next-steps"></a>Étapes suivantes
* [Créer un espace de noms](relay-create-namespace-portal.md)
* [Prise en main de .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Prise en main de Node](relay-hybrid-connections-node-get-started.md)

[Pricing overview]: https://azure.microsoft.com/pricing/details/service-bus/
[Relay exceptions]: relay-exceptions.md
[Shared Access Signatures]: ../service-bus-messaging/service-bus-sas.md
