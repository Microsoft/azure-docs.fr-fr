---
title: Supervision des journaux et des métriques dans Azure Front Door | Microsoft Docs
description: Cet article décrit les différents journaux d’accès et métriques pris en charge par Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: yuajia
ms.openlocfilehash: cd99be40700ab1c34176f2bf7497e4debf5cd424
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483795"
---
# <a name="monitoring-metrics-and-logs-in-azure-front-door"></a>Supervision des journaux et des métriques dans Azure Front Door

Azure Front Door vous permet de superviser les ressources des manières suivantes :

- **Métriques**. Azure Front Door dispose actuellement de huit métriques pour afficher les compteurs de performances.
- **Journaux**. Les journaux d’activité et de diagnostic permettent d’enregistrer ou de consommer les données de performances, d’accès et autres provenant d’une ressource à des fins de supervision.

### <a name="metrics"></a>Mesures

Les métriques représentent une fonctionnalité de certaines ressources Azure qui vous permettent de voir les compteurs de performances dans le portail. Les métriques Front Door disponibles sont les suivantes :

| Métrique | Nom d’affichage de la métrique | Unité | Dimensions | Description |
| --- | --- | --- | --- | --- |
| RequestCount | Nombre de requêtes | Count | httpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Nombre de requêtes de clients prises en charge par Front Door  |
| RequestSize | Taille de la requête | Octets | httpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Nombre d’octets envoyés en tant que requêtes de clients à Front Door. |
| ResponseSize | Taille de la réponse | Octets | httpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Nombre d’octets envoyés en tant que réponses de Front Door aux clients. |
| TotalLatency | Latence totale | Millisecondes | httpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Durée totale de la requête du client reçue par Front Door jusqu’au dernier octet de réponse envoyé d’AFD au client. |
| BackendRequestCount | Nombre de requêtes de backend | Count | httpStatus</br>HttpStatusGroup</br>Backend | Nombre de requêtes envoyées de Front Door aux backends. |
| BackendRequestLatency | Latence de requête du backend | Millisecondes | Backend | Temps calculé à partir du moment où la requête est envoyée par Front Door au backend jusqu’à ce que Front Door reçoive le dernier octet de la réponse du backend. |
| BackendHealthPercentage | Pourcentage d’intégrité du backend | Pourcentage | Backend</br>BackendPool | Pourcentage de sondes d’intégrité réussies de Front Door vers les backends. |
| WebApplicationFirewallRequestCount | Nombre de requêtes du pare-feu d’applications web | Count | PolicyName</br>RuleName</br>Action | Nombre de requêtes de clients traitées par la sécurité de couche Application de Front Door. |

## <a name="activity-logs"></a><a name="activity-log"></a>Journaux d’activité

Les journaux d’activité fournissent des informations sur les opérations effectuées sur Front Door. Ils répondent également aux questions quoi, qui et quand pour toutes les opérations d’écriture (put, post ou delete) exécutées sur Front Door.

>[!NOTE]
>Les journaux d’activité n’incluent pas les opérations de lecture (get). Ils n’incluent pas non plus les opérations que vous effectuez à l’aide du portail Azure ou de l’API Gestion d’origine.

Vous pouvez accéder aux journaux d’activité de votre service Front Door ou à tous les journaux de vos ressources Azure dans Azure Monitor. Pour afficher les journaux d’activité :

1. Sélectionnez votre instance de Front Door.
2. Sélectionnez **Journal d’activité**.

    :::image type="content" source="./media/front-door-diagnostics/activity-log.png" alt-text="Journal d’activité":::

3. Choisissez une étendue de filtrage, puis sélectionnez **Appliquer**.

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Journaux de diagnostic
Les journaux de diagnostic offrent des informations détaillées sur les opérations et erreurs qui sont importantes pour l’audit et le dépannage. Les journaux de diagnostic diffèrent des journaux d’activité.

Les journaux d’activité fournissent des informations détaillées sur les opérations effectuées sur des ressources Azure. Les journaux de diagnostic fournissent des informations détaillées sur les opérations effectuées par votre ressource. Pour plus d’informations, consultez [Journaux de diagnostic Azure Monitor](../azure-monitor/platform/platform-logs-overview.md).

:::image type="content" source="./media/front-door-diagnostics/diagnostic-log.png" alt-text="Journaux de diagnostic":::

Pour configurer les journaux de diagnostic pour votre service Front Door :

1. Sélectionnez votre service Azure Front Door.

2. Choisissez **Paramètres de diagnostic**.

3. Sélectionnez **Activer les diagnostics**. Archivez les journaux de diagnostic et les métriques dans un compte de stockage, envoyez-les en streaming à un hub d’événements ou envoyez-les aux journaux Azure Monitor.

Front Door fournit actuellement des journaux de diagnostic (par lot toutes les heures). Les journaux de diagnostic présentent chaque requête d’API individuellement, chaque entrée ayant le schéma suivant :

| Propriété  | Description |
| ------------- | ------------- |
| BackendHostname | Si la demande a été transférée à un serveur principal, ce champ représente le nom d’hôte du serveur principal. Ce champ est vide si la requête est redirigée ou transférée vers un cache régional (lorsque la mise en cache est activée pour la règle d’acheminement). |
| CacheStatus | Pour les scénarios de mise en cache, ce champ définit une absence/une correspondance dans le cache au niveau POP |
| ClientIp | Adresse IP du client à l’origine de la demande. S’il existait un en-tête X-Forwarded-For dans la demande, l’adresse IP du client est sélectionnée de la même façon. |
| ClientPort | Adresse IP du port du client qui a effectué la requête. |
| HttpMethod | Méthode HTTP utilisée par la requête. |
| HttpStatusCode | Code d’état HTTP retourné par le proxy. |
| HttpStatusDetails | État résultant de la requête. Vous trouverez la signification de cette valeur de chaîne dans la Table de référence des états. |
| HttpVersion | Type de la requête ou de la connexion. |
| POP | Nom abrégé de la périphérie où la demande est arrivée. |
| RequestBytes | Taille du message de requête HTTP en octets, en-têtes de requête et corps de requête compris. |
| RequestUri | URI de la requête reçue. |
| ResponseBytes | Octets envoyés en tant que réponse par le serveur back-end.  |
| RoutingRuleName | Nom de la règle de routage correspondant à la requête. |
| RulesEngineMatchNames | Noms des règles correspondant à la demande. |
| SecurityProtocol | Version du protocole TLS/SSL utilisée par la requête, ou Null si aucun chiffrement. |
| SentToOriginShield </br> (déconseillé) * **Consultez les notes sur la désapprobation dans la section suivante.**| Si la valeur est true, cela signifie que la requête a été traitée à partir du cache de protection d’origine au lieu du pop de périphérie. La protection d’origine est un cache parent utilisé pour améliorer le taux d’accès au cache. |
| isReceivedFromClient | Si la valeur est true, cela signifie que la requête provient du client. Si la valeur est false, la requête est en échec dans la périphérie (POP enfant) et reçoit une réponse à partir de la protection d’origine (POP parent). |
| TimeTaken | Durée, en secondes, écoulée entre le premier octet de la requête Front Door et le dernier octet de la réponse. |
| TrackingReference | Chaîne de référence unique qui identifie une requête traitée par Front Door, également envoyée en tant qu’en-tête X-Azure-Ref au client. Nécessaire pour pouvoir effectuer une recherche détaillée dans les journaux d’accès pour une requête spécifique. |
| UserAgent | Type de navigateur utilisé par le client. |
| ErrorInfo | Ce champ contient le type d’erreur spécifique pour la résolution des problèmes. </br> Les valeurs possibles incluent : </br> **NoError** : Indique qu’aucune erreur n’a été trouvée. </br> **CertificateError** : Erreur de certificat SSL générique.</br> **CertificateNameCheckFailed** : Le nom d’hôte dans le certificat SSL n’est pas valide ou ne correspond pas. </br> **ClientDisconnected** : Échec de la demande en raison de la connexion réseau du client. </br> **UnspecifiedClientError** : Erreur du client générique. </br> **InvalidRequest** : Demande non valide. Cela peut se produire en raison d’un en-tête, d’un corps et d’une URL incorrect(e)s. </br> **DNSFailure** : Échec DNS. </br> **DNSNameNotResolved** : Le nom ou l’adresse du serveur n’a pas pu être résolue. </br> **OriginConnectionAborted** : La connexion avec l’origine a été arrêtée brusquement. </br> **OriginConnectionError** : Erreur de connexion d’origine générique. </br> **OriginConnectionRefused** : La connexion avec l’origine n’a pas pu être établie. </br> **OriginError** : Erreur d’origine générique. </br> **OriginInvalidResponse** : L’origine a renvoyé une réponse non valide ou non reconnue. </br> **OriginTimeout** : Le délai d’expiration de la demande d’origine a expiré. </br> **ResponseHeaderTooBig** : L’origine a retourné un en-tête de réponse trop grand. </br> **RestrictedIP** : La demande a été bloquée en raison d’une adresse IP restreinte. </br> **SSLHandshakeError** : Impossible d’établir la connexion avec l’origine en raison d’un échec d’établissement d’une liaison SSL. </br> **UnspecifiedError** : Une erreur ne correspondant à aucune des erreurs dans le tableau s’est produite. |

### <a name="sent-to-origin-shield-deprecation"></a>Dépréciation de Sent to origin shield
La propriété de journal brut **isSentToOriginShield** est déconseillée et remplacée par un nouveau champ **isReceivedFromClient**. Utilisez le nouveau champ si vous utilisez déjà le champ déconseillé. 

Les journaux bruts incluent les journaux générés à partir de la périphérie de CDN (POP enfant) et de la protection d’origine. La protection d’origine fait référence aux nœuds parents qui se trouvent stratégiquement dans le monde entier. Ces nœuds communiquent avec les serveurs d’origine et réduisent la charge du trafic sur l’origine. 

Pour chaque requête qui accède à la protection d’origine, il y a deux entrées de journal :

* une pour les nœud de périphérie
* l’autre pour la protection d’origine 

Pour différencier la sortie ou les réponses des nœuds de périphérie par rapport à la protection d’origine, vous pouvez utiliser le champ **isReceivedFromClient** pour récupérer les données correctes. 

Si la valeur est false, cela signifie que la requête reçoit une réponse de la protection d’origine jusqu’aux nœuds de périphérie. Cette approche est efficace pour comparer les journaux bruts aux données de facturation. Les frais ne sont pas facturés pour la sortie de la protection d’origine jusqu’aux nœuds de périphérie. Des frais sont facturés pour la sortie des nœuds de périphérie jusqu’aux clients. 

**Exemple de requête Kusto pour exclure les journaux générés sur la protection d’origine dans Log Analytics.**

`AzureDiagnostics 
| where Category == "FrontdoorAccessLog" and isReceivedFromClient_b == true`

> [!NOTE]
> Pour les différentes configurations de routage et les différents comportements de trafic, certains des champs, comme backendHostname, cacheStatus, isReceivedFromClient et POP peuvent renvoyer des valeurs différentes. Le tableau ci-dessous explique les différentes valeurs que contiennent ces champs dans différents scénarios :

| Scénarios | Nombre d’entrées de journal | POP | BackendHostname | isReceivedFromClient | CacheStatus |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Règle d’acheminement sans mise en cache | 1 | Code POP Edge | Serveur principal où la demande a été envoyée | True | CONFIG_NOCACHE |
| Règle d’acheminement avec mise en cache. Correspondance dans le cache au niveau de la périphérie POP | 1 | Code POP Edge | Vide | True | HIT |
| Règle d’acheminement avec mise en cache. Absence dans le cache au niveau de la périphérie POP, mais correspondance dans le cache au niveau POP du cache parent | 2 | 1. Code POP Edge</br>2. Code POP du cache parent | 1. Nom d’hôte POP du cache parent</br>2. Vide | 1. True</br>2. False | 1. MISS</br>2. HIT |
| Règle d’acheminement avec mise en cache. Absence dans le cache au niveau de la périphérie POP, mais correspondance PARTIAL dans le cache au niveau POP du cache parent | 2 | 1. Code POP Edge</br>2. Code POP du cache parent | 1. Nom d’hôte POP du cache parent</br>2. Serveur principal qui permet de remplir le cache | 1. True</br>2. False | 1. MISS</br>2. PARTIAL_HIT |
| Règle d’acheminement avec mise en cache. Cache PARTIAL_HIT au niveau de la périphérie POP, mais correspondance dans cache au niveau POP du cache parent | 2 | 1. Code POP Edge</br>2. Code POP du cache parent | 1. Code POP Edge</br>2. Code POP du cache parent | 1. True</br>2. False | 1. PARTIAL_HIT</br>2. HIT |
| Règle d’acheminement avec mise en cache. Absence dans le cache à la fois dans la périphérie et dans le POP du cache parent | 2 | 1. Code POP Edge</br>2. Code POP du cache parent | 1. Code POP Edge</br>2. Code POP du cache parent | 1. True</br>2. False | 1. MISS</br>2. MISS |

> [!NOTE]
> Pour les scénarios de mise en cache, la valeur de l’état du cache est partial_hit lorsque certains octets d’une requête sont traités à partir du cache de la périphérie Front Door ou du cache de la protection d’origine, tandis que certains octets sont traités à partir de l’origine pour les objets volumineux.

Front Door utilise une technique appelée segmentation d’objets. Quand un fichier volumineux est demandé, l’instance Front Door récupère les petits éléments du fichier à partir de l’origine. Une fois que le serveur POP de l’instance Front Door a reçu une requête de fichier complet ou de plage d’octets de fichier, le serveur de périphérie Front Door demande le fichier au serveur d’origine par segments de 8 Mo.

Quand un segment arrive à la périphérie Front Door, il est mis en cache et immédiatement servi à l’utilisateur. L’instance Front Door se prépare ensuite à récupérer le bloc suivant en parallèle. Cette prérécupération garantit que le contenu a un bloc d’avance sur l’utilisateur, ce qui a réduit la latence. Ce processus se poursuit jusqu'à ce que le fichier entier soit téléchargé (si nécessaire), que toutes les plages d’octets soient disponibles (si nécessaire), ou que le client mette fin à la connexion. Pour plus d’informations sur la demande de plage d’octets, voir RFC 7233. L’instance Front Door met en cache les blocs au fur et à mesure de leur réception. Le fichier entier n’a pas besoin d’être mis en cache sur le cache Front Door. Les demandes suivantes concernant le fichier ou les plages d’octets sont traitées à partir du cache Front Door. Si tous les blocs sont mis en cache sur l’instance Front Door, une prérécupération est utilisée pour demander des blocs de l’origine. Cette optimisation s’appuie sur la capacité du serveur d’origine à prendre en charge des demandes de plages d’octets. Si le serveur d’origine ne prend pas en charge les demandes de plages d’octets, cette optimisation n’est pas effective.

## <a name="next-steps"></a>Étapes suivantes

- [Créer un profil Front Door](quickstart-create-front-door.md)
- [Fonctionnement de Front Door](front-door-routing-architecture.md)
