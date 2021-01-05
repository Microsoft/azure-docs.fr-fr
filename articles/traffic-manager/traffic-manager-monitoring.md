---
title: Surveillance des points de terminaison Azure Traffic Manager | Microsoft Docs
description: Cet article explique comment Traffic Manager utilise la surveillance des points de terminaison et le basculement automatique des points de terminaison pour aider les clients Azure à déployer des applications haute disponibilité
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/04/2018
ms.author: duau
ms.openlocfilehash: 78a1681c743f65081b30657f4fd747ff8aaef5f5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89392831"
---
# <a name="traffic-manager-endpoint-monitoring"></a>Surveillance des points de terminaison Traffic Manager

Azure Traffic Manager inclut la surveillance intégrée des points de terminaison et leur basculement automatique. Cette fonctionnalité vous aide à fournir des applications haute disponibilité résistantes aux défaillances des points de terminaison, notamment les défaillances des régions Azure.

## <a name="configure-endpoint-monitoring"></a>Configuration de la surveillance des points de terminaison

Pour configurer la surveillance des points de terminaison, vous devez spécifier les paramètres suivants sur votre profil Traffic Manager :

* **Protocole**. Sélectionnez HTTP, HTTPS ou TCP comme protocole utilisé par Traffic Manager lors de la détection du point de terminaison pour contrôler son intégrité. Notez que la surveillance HTTPS ne vérifie pas si votre certificat TLS/SSL est valide, mais uniquement s’il est présent.
* **Port**. Choisissez le port utilisé pour la requête.
* **Path**. Ce paramètre de configuration est valide uniquement pour les protocoles HTTP et HTTPS pour lesquels la configuration du chemin est obligatoire. La configuration de ce paramètre pour le protocole de surveillance TCP provoque une erreur. Pour le protocole HTTP et HTTPS, indiquez le chemin relatif et le nom du fichier ou de la page web auxquels la surveillance accède. Une barre oblique (/) est une entrée valide pour le chemin d’accès relatif. Cette valeur indique que le fichier est dans le répertoire racine (par défaut).
* **Paramètres d’en-tête personnalisé**. Ce paramètre de configuration vous permet d’ajouter des en-têtes HTTP spécifiques aux vérifications d’intégrité que Traffic Manager envoie aux points de terminaison par le biais d’un profil. Les en-têtes personnalisés peuvent être spécifiés au niveau du profil (pour s’appliquer à tous les points de terminaison de ce profil) et/ou au niveau du point de terminaison (applicable uniquement à ce point de terminaison). Vous pouvez utiliser des en-têtes personnalisés pour que les vérifications d’intégrité effectuées sur des points de terminaison d’un environnement multilocataire soient correctement acheminées vers leur destination en spécifiant un en-tête d’hôte. Vous pouvez également utiliser ce paramètre en ajoutant des en-têtes uniques qui peuvent servir à identifier les requêtes HTTP(S) provenant de Traffic Manager et les traiter différemment. Vous pouvez spécifier jusqu’à huit paires header:value séparées par une virgule. Par exemple, « header1:value1,header2:value2 ». 
* **Plages de code d’état prévues**. Ce paramètre vous permet de spécifier plusieurs plages de code de réussite au format 200-299, 301-301. Si, suite au lancement d’une vérification de l’intégrité, un point de terminaison envoie ces codes d’état en tant que réponse, Traffic Manager considère que ces points de terminaison sont sains. Vous pouvez spécifier un maximum de 8 plages de code d’état. Ce paramètre s’applique uniquement aux protocoles HTTP et HTTPS, et à tous les points de terminaison. Ce paramètre se trouve au niveau du profil Traffic Manager. Par défaut, la valeur du code d’état de réussite est définie sur 200.
* **Intervalle de détection**. Cette valeur spécifie la fréquence à laquelle l’intégrité d’un point de terminaison est contrôlée par un agent de détection Traffic Manager. Vous pouvez spécifier deux valeurs ici : 30 secondes (détection normale) et 10 secondes (détection rapide). Si aucune valeur n’est fournie, le profil définit par défaut la valeur sur 30 secondes. Visitez la page [Tarification de Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager) pour en savoir plus sur la tarification pour la détection rapide.
* **Nombre d’échecs tolérés**. Cette valeur spécifie le nombre d’échecs tolérés par un agent de détection de Traffic Manager avant que le point de terminaison soit considéré comme défectueux. La valeur peut être comprise entre 0 et 9. Une valeur égale à 0 signifie qu’il suffit d’un seul échec lors de l’analyse pour que le point de terminaison soit considéré comme défectueux. Si aucune valeur n’est spécifiée, la valeur par défaut est 3.
* **Délai d’expiration de la détection**. Cette propriété spécifie le temps que l’agent de détection de Traffic Manager doit attendre avant de considérer que la procédure comme un échec lorsqu’une sonde de vérification de l’intégrité est envoyée au point de terminaison. Si l’intervalle de détection est défini sur 30 secondes, vous pouvez définir la valeur du délai d’expiration entre 5 et 10 secondes. Si aucune valeur n’est spécifiée, la valeur par défaut est 10 secondes. Si l’intervalle de détection est défini sur 10 secondes, vous pouvez définir la valeur du délai d’expiration entre 5 et 9 secondes. Si aucune valeur n’est spécifiée pour le délai d’expiration, la valeur par défaut est 9 secondes.

    ![Surveillance des points de terminaison Traffic Manager](./media/traffic-manager-monitoring/endpoint-monitoring-settings.png)

    **Figure :  Supervision des points de terminaison Traffic Manager**

## <a name="how-endpoint-monitoring-works"></a>Fonctionnement de la surveillance des points de terminaison

Si le protocole de surveillance est défini sur HTTP ou HTTPS, l’agent de détection de Traffic Manager effectue une requête GET vers le point de terminaison à l’aide du protocole, du port et du chemin d’accès relatif donnés. S’il obtient la réponse 200-OK, ou l’une des réponses configurées dans les **plages\* de code d’état prévues**, alors ce point de terminaison est considéré comme sain. Si la réponse est une valeur différente, ou si aucune réponse n’est reçue dans le délai spécifié, l’agent de détection Traffic Manager refait une tentative en fonction du nombre d’échecs tolérés défini (aucune nouvelle tentative si cette valeur est définie sur 0). Si le nombre d’échecs consécutifs est supérieur au nombre d’échecs tolérés, ce point de terminaison est considéré comme défectueux. 

Si le protocole de surveillance est TCP, l’agent de détection de Traffic Manager lance une demande de connexion TCP à l’aide du port spécifié. Si le point de terminaison répond positivement à la requête de connexion, le contrôle d’intégrité est considéré comme une réussite et l’agent de détection de Traffic Manager rétablit la connexion TCP. Si la réponse est différente ou si aucune réponse n’est reçue dans le délai spécifié, l’agent de détection Traffic Manager refait une tentative en fonction du nombre d’échecs tolérés défini (aucune nouvelle tentative si cette valeur est définie sur 0). Si le nombre d’échecs consécutifs est supérieur au nombre d’échecs tolérés, ce point de terminaison est considéré comme défectueux.

Dans tous les cas, Traffic Manager sonde plusieurs emplacements et identifie les échecs dans chaque région. Cela signifie également que les points de terminaison reçoivent des sondes d’intégrité à partir de Traffic Manager à une fréquence supérieure à celle du paramètre utilisé pour l’intervalle de détection.

>[!NOTE]
>Pour le protocole de surveillance HTTP ou HTTPS, il convient d’implémenter une page personnalisée dans l’application, côté point de terminaison. Par exemple : /health.aspx. À l’aide de ce chemin d’accès pour la surveillance, vous pouvez effectuer des vérifications spécifiques aux applications, telles que la vérification des compteurs de performance ou de la disponibilité de la base de données. Sur la base de ces contrôles personnalisés, la page renvoie un code d’état HTTP approprié.

Tous les points de terminaison dans un profil Traffic Manager partagent les mêmes paramètres de surveillance. Si vous devez utiliser différents paramètres de surveillance pour les différents points de terminaison, vous pouvez créer des [profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md#example-5-per-endpoint-monitoring-settings).

## <a name="endpoint-and-profile-status"></a>État des points de terminaison et des profils

Vous pouvez activer et désactiver les points de terminaison et les profils Traffic Manager. Toutefois, un changement d’état du point de terminaison peut survenir également comme conséquence des paramètres et des processus automatisés de Traffic Manager.

### <a name="endpoint-status"></a>État du point de terminaison

Vous pouvez activer ou désactiver un point de terminaison spécifique. Le service sous-jacent, qui peut toujours être sain, n’est pas affecté. En modifiant l’état du point de terminaison, vous contrôlez la disponibilité du point de terminaison dans le profil Traffic Manager. Lorsque l’état d’un point de terminaison est désactivé, Traffic Manager ne vérifie pas son intégrité et le point de terminaison n’est pas inclus dans une réponse DNS.

### <a name="profile-status"></a>État du profil

En utilisant le paramètre d’état de profil, vous pouvez activer ou désactiver un profil spécifique. Alors que l’état du point de terminaison affecte un seul point de terminaison, l’état du profil affecte le profil complet, donc l’ensemble des points de terminaison. Lorsque vous désactivez un profil, l’intégrité des points de terminaison n’est pas vérifiée et aucun point de terminaison n’est inclus dans une réponse DNS. Un code de réponse [NXDOMAIN](https://tools.ietf.org/html/rfc2308) est renvoyé pour la requête DNS.

### <a name="endpoint-monitor-status"></a>État de surveillance des points de terminaison

L’état de surveillance des points de terminaison est une valeur générée par Traffic Manager qui affiche l’état du point de terminaison. Vous ne pouvez pas modifier ce paramètre manuellement. L’état de surveillance des points de terminaison est une combinaison des résultats de la surveillance du point de terminaison et de l’état du point de terminaison configuré. Les valeurs possibles de l’état de surveillance des points de terminaison sont affichées dans la table suivante :

| État du profil | État du point de terminaison | État de surveillance des points de terminaison | Notes |
| --- | --- | --- | --- |
| Désactivé |activé |Inactif |Le profil a été désactivé. Même si l’état du point de terminaison est Activé, l’état du profil (Désactivé) a la priorité. Dans les profils désactivés, les points de terminaison ne sont pas surveillés. Un code de réponse NXDOMAIN est renvoyé pour la requête DNS. |
| &lt;any&gt; |Désactivé |Désactivé |Le point de terminaison a été désactivé. Les points de terminaison désactivés ne sont pas surveillés. Le point de terminaison n’est pas inclus dans les réponses DNS, et par conséquent, il ne reçoit pas de trafic. |
| activé |activé |En ligne |Le point de terminaison est surveillé et sain. Il est inclus dans les réponses DNS et peut recevoir le trafic. |
| activé |activé |Détérioré |Les contrôles d’intégrité de surveillance des points de terminaison sont défaillants. Le point de terminaison n’est pas inclus dans les réponses DNS et ne reçoit pas de trafic. <br>Exception : si tous les points de terminaison sont défectueux, auquel cas l’on considère qu’ils sont tous renvoyés dans la réponse à la requête.</br>|
| activé |activé |CheckingEndpoint |Le point de terminaison est surveillé, mais les résultats de la première analyse n’ont pas encore été reçus. CheckingEndpoint est un état temporaire qui survient généralement immédiatement après l’ajout ou l’activation d’un point de terminaison dans le profil. Un point de terminaison avec cet état est inclus dans les réponses DNS et peut recevoir le trafic. |
| activé |activé |Arrêté |L’application web vers laquelle le point de terminaison pointe ne s’exécute pas. Vérifiez les paramètres de l’application web. Cela peut également se produire si le point de terminaison est de type imbriqué et que le profil enfant est désactivé ou inactif. <br>Un point de terminaison avec l’état Arrêté n’est pas surveillé. Il n’est pas inclus dans les réponses DNS et ne reçoit pas de trafic. Exception : si tous les points de terminaison sont défectueux, auquel cas l’on considère qu’ils sont tous renvoyés dans la réponse à la requête.</br>|

Pour savoir comment l’état de surveillance des points de terminaison est calculé pour les points de terminaison imbriqués, consultez la section relative aux [profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md).

>[!NOTE]
> Un état du moniteur Point de terminaison arrêté peut se produire sur App Service si votre application web ne s’exécute pas au niveau Standard ou à un niveau supérieur. Pour plus d’informations, voir [Intégration de Traffic Manager avec App Service](/azure/app-service/web-sites-traffic-manager).

### <a name="profile-monitor-status"></a>État de surveillance du profil

L’état de surveillance du profil est une combinaison de l’état du profil configuré et des valeurs de l’état de surveillance de l’ensemble des points de terminaison. Les valeurs possibles sont décrites dans le tableau suivant :

| État du profil (conformément à la configuration) | État de surveillance des points de terminaison | État de surveillance du profil | Notes |
| --- | --- | --- | --- |
| Désactivé |&lt;any&gt;  ou profil sans points de terminaison définis. |Désactivé |Le profil a été désactivé. |
| activé |Un ou plusieurs points de terminaison ont l’état Détérioré. |Détérioré |Passez en revue les valeurs d’état de chaque point de terminaison pour déterminer quels points de terminaison doivent être examinés. |
| activé |Un ou plusieurs points de terminaison ont l’état En ligne. Aucun point de terminaison n’a l’état Détérioré. |En ligne |Le service accepte le trafic. Aucune action supplémentaire n’est requise. |
| activé |Un ou plusieurs points de terminaison ont l’état CheckingEndpoint. Aucun point de terminaison n’a l’état En ligne ou Détérioré. |CheckingEndpoints |Cet état de transition se produit lorsqu’un profil est créé ou activé. L’intégrité du point de terminaison est vérifiée pour la première fois. |
| activé |Tous les points de terminaison dans le profil ont l’état Désactivé ou Arrêté, ou le profil n’inclut aucun point de terminaison défini. |Inactif |Aucun point de terminaison n’est actif, mais le profil est toujours Activé. |

## <a name="endpoint-failover-and-recovery"></a>Basculement et récupération du point de terminaison

Traffic Manager vérifie périodiquement l’intégrité de chaque point de terminaison, y compris des points de terminaison défectueux. Traffic Manager détecte lorsque l’intégrité d’un point de terminaison est rétablie et ramène celui-ci en rotation.

Un point de terminaison est défectueux lorsque l’un des événements suivants se produit :

- Si le protocole de surveillance est HTTP ou HTTPS :
    - Une réponse autre que 200, ou une réponse qui n’inclut pas la plage spécifiée dans le paramètre **Plages de code d’état prévues**, est reçue (y compris un code 2xx différent ou une redirection 301/302).
- Si le protocole de surveillance est TCP : 
    - Réception d’une réponse autre que ACK ou SYN-ACK, en réponse à la requête SYN envoyée par Traffic Manager pour tenter d’établir une connexion.
- Délai d’expiration. 
- Tout autre problème de connexion qui fait que le point de terminaison n’est ne pas accessible.

Pour plus d’informations sur le dépannage des vérifications en échec, consultez [Résolution des problèmes liés à l’état Détérioré d’Azure Traffic Manager](traffic-manager-troubleshooting-degraded.md). 

La chronologie de la figure suivante représente une description détaillée du processus de surveillance du point de terminaison de Traffic Manager, avec les paramètres suivants : protocole de surveillance HTTP, intervalle de détection de 3 secondes, nombre d’échecs tolérés égal à 10, délai d’expiration de 30 secondes et durée de vie DNS de 30 secondes.

![Séquence de basculement et de restauration automatique des points de terminaison Traffic Manager](./media/traffic-manager-monitoring/timeline.png)

**Figure :  Séquence de basculement et de récupération des points de terminaison Traffic Manager**

1. **GET**. Pour chaque point de terminaison, le système de surveillance de Traffic Manager exécute une requête GET sur le chemin d’accès spécifié dans les paramètres de surveillance.
2. **200 OK ou plage de code personnalisée spécifiée dans les paramètres de surveillance du profil Traffic Manager**. Le système de surveillance attend que le message HTTP 200 OK ou celui de la plage de code personnalisée spécifiée dans les paramètres du profil Traffic Manager soit retourné sous 10 secondes. Lorsqu’il reçoit cette réponse, il reconnaît que le service est disponible.
3. **30 secondes entre les contrôles**. Le contrôle d’intégrité du point de terminaison est répété toutes les 30 secondes.
4. **Service indisponible**. Le service devient indisponible. Traffic Manager n’est pas averti avant le prochain contrôle d’intégrité.
5. **Tente d’accéder au chemin d’accès de surveillance**. Le système de surveillance exécute une requête GET, mais ne reçoit pas de réponse au cours du délai d’expiration de 10 secondes (sinon, une réponse autre que 200 peut être reçue). Il effectue ensuite trois essais supplémentaires par intervalles de 30 secondes. Lorsque l’un des essais aboutit, le nombre d’essais est réinitialisé.
6. **État défini comme Détérioré**. Après un quatrième échec consécutif, le système de surveillance marque l’état du point de terminaison non disponible comme Détérioré.
7. **Le trafic est détourné vers les autres points de terminaison**. Les serveurs de noms DNS Traffic Manager sont mis à jour et Traffic Manager ne renvoie plus le point de terminaison en réponse aux requêtes DNS. Les nouvelles connexions sont dirigées vers les autres points de terminaison disponibles. Toutefois, les réponses DNS précédentes incluant ce point de terminaison peuvent toujours être mises en cache par des serveurs DNS et des clients DNS récursifs. Les clients continuent à utiliser le point de terminaison jusqu’à l’expiration du cache DNS. Lorsque les caches DNS expirent, les clients exécutent de nouvelles requêtes DNS et sont dirigés vers des points de terminaison différents. La durée du cache est contrôlée par le paramètre TTL dans le profil Traffic Manager (par exemple, 30 secondes).
8. **Les contrôles d’intégrité continuent**. Traffic Manager continue à contrôler l’intégrité du point de terminaison pendant qu’il se trouve dans l’état Détérioré. Traffic Manager détecte les points de terminaison dont l’intégrité a été restaurée.
9. **Le service est remis en ligne**. Le service devient disponible. Le point de terminaison conserve sont état Détérioré dans Traffic Manager jusqu’à ce que le système de surveillance effectue le contrôle d’intégrité suivant.
10. **Le trafic vers le service est rétabli**. Traffic Manager envoie une requête GET et reçoit une réponse d’état 200 OK. Le service a retrouvé son intégrité. Les serveurs de noms Traffic Manager sont mis à jour et commencent à distribuer le nom DNS du service dans les réponses DNS. Le trafic retourne au point de terminaison tandis que les réponses DNS mises en cache et renvoyées vers d’autres points de terminaison expirent et que les connexions existantes à d’autres points de terminaison s’arrêtent.

    > [!NOTE]
    > Étant donné que Traffic Manager fonctionne au niveau du DNS, il ne peut pas modifier les connexions existantes à un point de terminaison. Lorsqu’il dirige le trafic entre les points de terminaison (en modifiant les paramètres de profil, ou lors d’un basculement ou d’une restauration automatique), Traffic Manager dirige les nouvelles connexions vers les points de terminaison disponibles. Toutefois, les autres points de terminaison peuvent continuer à recevoir le trafic via les connexions existantes, jusqu’à ce que ces sessions s’arrêtent. Pour libérer le trafic à partir des connexions existantes, les applications doivent limiter la durée de session utilisée avec chaque point de terminaison.

## <a name="traffic-routing-methods"></a>Méthodes de routage du trafic

Lorsqu’un point de terminaison est détérioré, il n’est plus renvoyé en réponse aux requêtes DNS. Au lieu de cela, un autre point de terminaison est choisi et renvoyé. La méthode de routage du trafic configurée dans le profil détermine la manière dont le point de terminaison de remplacement est sélectionné.

* **Priorité**. Les points de terminaison forment une liste hiérarchisée. Le premier point de terminaison disponible sur la liste est toujours renvoyé. Si l’état d’un point de terminaison est Détérioré, alors le point de terminaison disponible suivant est renvoyé.
* **Pondération**. Le point de terminaison de remplacement est choisi de façon aléatoire parmi tous les points de terminaison disponibles en fonction des pondérations qui leur ont été affectées et des pondérations des autres points de terminaison disponibles.
* **Performances**. Le point de terminaison le plus proche de l’utilisateur final est renvoyé. Si ce point de terminaison n’est pas disponible, Traffic Manager déplace le trafic vers les points de terminaison dans la région Azure suivante la plus proche. Vous pouvez configurer d’autres plans de basculement pour le routage du trafic à l’aide des [profils Traffic Manager imbriqués](traffic-manager-nested-profiles.md#example-4-controlling-performance-traffic-routing-between-multiple-endpoints-in-the-same-region).
* **Géographique**. Le point de terminaison mappé pour servir la zone géographique en fonction de l’adresse IP de la requête est renvoyé. Si ce point de terminaison n’est pas disponible, aucun autre point de terminaison ne sera sélectionné pour le basculement, car un emplacement géographique ne peut être mappé qu’à un seul point de terminaison dans un profil (plus de détails dans la rubrique [FAQ](traffic-manager-FAQs.md#traffic-manager-geographic-traffic-routing-method)). Lorsque le routage géographique est utilisé, nous recommandons aux clients d’utiliser des profiles Traffic Manager imbriqués avec plusieurs points de terminaison comme points de terminaison du profil.
* **Valeurs multiples** Plusieurs points de terminaison mappés sur des adresses IPv4/IPv6 sont retournés. Lorsqu’une requête est reçue pour ce profil, les points de terminaison sains sont retournés en fonction de la valeur du **nombre maximal d’enregistrements de la réponse** que vous avez spécifiée. Par défaut, le nombre de réponses correspond à deux points de terminaison.
* **Sous-réseau** Le point de terminaison mappé à un ensemble de plages d’adresses IP est retourné. Lorsqu’une requête est reçue à partir de cette adresse IP, le point de terminaison retourné est celui qui est mappé pour cette adresse IP. 

Pour plus d’informations, consultez la rubrique relative aux [méthodes de routage du trafic dans Traffic Manager](traffic-manager-routing-methods.md).

> [!NOTE]
> Une exception au comportement normal de routage du trafic se produit lorsque tous les points de terminaison éligibles sont détériorés. Traffic Manager effectue alors un service de type « meilleur effort » et *réagit comme si tous les points de terminaison présentant un état Détérioré étaient en fait en ligne*. Ce comportement est préférable à l’autre solution, qui serait de ne renvoyer aucun point de terminaison dans la réponse DNS. Les points de terminaison désactivés ou arrêtés ne sont pas surveillés et ne sont par conséquent pas éligibles pour le trafic.
>
> Cette situation est généralement due à une configuration incorrecte du service, par exemple :
>
> * Une ACL bloque les contrôles d’intégrité de Traffic Manager.
> * Une mauvaise configuration du port ou du protocole de surveillance dans le profil Traffic Manager.
>
> La conséquence de ce comportement est la suivante : lorsque les contrôles d’intégrité de Traffic Manager ne sont pas configurés correctement, du point de vue du routage du trafic il semble que Traffic Manager *fonctionne* correctement. Toutefois, dans ce cas, le basculement de point de terminaison ne peut pas se produire, ce qui affecte la disponibilité globale de l’application. Il est donc important de vérifier que le profil affiche un état En ligne et non Détérioré. L’état En ligne indique que les contrôles d’intégrité de Traffic Manager fonctionnent comme prévu.

Pour plus d’informations sur le dépannage des contrôles d’intégrité, consultez la section [Résolution des problèmes liés à l’état Détérioré d’Azure Traffic Manager](traffic-manager-troubleshooting-degraded.md).

## <a name="faqs"></a>FAQ

* [Traffic Manager est-il résistant aux défaillances de régions Azure ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#is-traffic-manager-resilient-to-azure-region-failures)

* [En quoi le choix de l’emplacement du groupe de ressources affecte-t-il Traffic Manager ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-the-choice-of-resource-group-location-affect-traffic-manager)

* [Comment déterminer l’état d’intégrité actuel de chaque point de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-do-i-determine-the-current-health-of-each-endpoint)

* [Puis-je superviser les points de terminaison HTTPS ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-monitor-https-endpoints)

* [Dois-je utiliser une adresse IP ou un nom DNS lors de l’ajout d’un point de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#do-i-use-an-ip-address-or-a-dns-name-when-adding-an-endpoint)

* [Quels types d’adresses IP puis-je utiliser lors de l’ajout d’un point de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-types-of-ip-addresses-can-i-use-when-adding-an-endpoint)

* [Puis-je utiliser différents types d’adressage de point de terminaison dans un même profil ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-different-endpoint-addressing-types-within-a-single-profile)

* [Que se passe-t-il quand le type d’enregistrement d’une requête entrante est différent du type d’enregistrement associé au type d’adressage des points de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-happens-when-an-incoming-querys-record-type-is-different-from-the-record-type-associated-with-the-addressing-type-of-the-endpoints)

* [Puis-je utiliser un profil avec des points de terminaison IPv4/IPv6 dans un profil imbriqué ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-a-profile-with-ipv4--ipv6-addressed-endpoints-in-a-nested-profile)

* [J’ai arrêté un point de terminaison d’application web dans mon profil Traffic Manager, mais je ne reçois pas de trafic même après un redémarrage. Comment puis-je résoudre ce problème ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#i-stopped-an-web-application-endpoint-in-my-traffic-manager-profile-but-i-am-not-receiving-any-traffic-even-after-i-restarted-it-how-can-i-fix-this)

* [Puis-je utiliser Traffic Manager même si mon application ne prend pas en charge HTTP ou HTTPS ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-traffic-manager-even-if-my-application-does-not-have-support-for-http-or-https)

* [Quelles réponses spécifiques du point de terminaison sont obligatoires quand la surveillance TCP est utilisée ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-specific-responses-are-required-from-the-endpoint-when-using-tcp-monitoring)

* [Avec quelle rapidité Traffic Manager éloigne-t-il mes utilisateurs d’un point de terminaison non sain ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-fast-does-traffic-manager-move-my-users-away-from-an-unhealthy-endpoint)

* [Comment spécifier différents paramètres de surveillance pour les différents points de terminaison dans un profil ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-specify-different-monitoring-settings-for-different-endpoints-in-a-profile)

* [Comment puis-je attribuer des en-têtes HTTP aux contrôles d’intégrité Traffic Manager pour mes points de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-assign-http-headers-to-the-traffic-manager-health-checks-to-my-endpoints)

* [Quel en-tête hôte est utilisé pour les contrôles d’intégrité des points de terminaison ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-host-header-do-endpoint-health-checks-use)

* [Quelles sont les adresses IP à l’origine des contrôles d’intégrité ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-the-ip-addresses-from-which-the-health-checks-originate)

* [Combien de contrôles d’intégrité de mon point de terminaison Traffic Manager effectue-t-il ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-many-health-checks-to-my-endpoint-can-i-expect-from-traffic-manager)

* [Comment être informé si l’un de mes points de terminaison tombe en panne ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-get-notified-if-one-of-my-endpoints-goes-down)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le [fonctionnement de Traffic Manager](traffic-manager-how-it-works.md)

En savoir plus sur les [méthodes de routage du trafic](traffic-manager-routing-methods.md) prises en charge par Traffic Manager

En savoir plus sur la [création d’un profil Traffic Manager](traffic-manager-manage-profiles.md)

[Résolution des problèmes liés à l’état Détérioré](traffic-manager-troubleshooting-degraded.md) sur un point de terminaison Traffic Manager
