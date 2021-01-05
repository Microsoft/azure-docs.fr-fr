---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 02/27/2020
ms.author: ccompy
ms.openlocfilehash: cec44bbabdb7d528c30a8d3396b819f2eb3c5386
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999431"
---
Cette fonctionnalité est facile à configurer, mais il est possible que vous rencontriez des problèmes. Si vous rencontrez des difficultés pour accéder au point de terminaison souhaité, certains utilitaires vous permettent de tester la connectivité à partir de la console de l’application. Vous pouvez utiliser deux consoles : la console Kudu et la console du Portail Azure. Pour accéder à la console Kudu à partir de votre application, accédez à **Outils** > **Kudu**. Vous pouvez également accéder à la console Kudo via le site [nom du site].scm.azurewebsites.net. Une fois le site chargé, accédez à l’onglet **Console de débogage**. Pour accéder à la console hébergée par le Portail Azure à partir de votre application, accédez à **Outils** > **Console**.

#### <a name="tools"></a>Outils
Dans les applications Windows natives, les outils **ping**, **nslookup** et **tracert** ne fonctionnent pas dans la console en raison de contraintes de sécurité (ils fonctionnent dans des [conteneurs Windows personnalisés](../articles/app-service/quickstart-custom-container.md)). Deux outils distincts ont été ajoutés pour les remplacer. Pour tester les fonctionnalités DNS, nous avons ajouté un outil nommé **nameresolver.exe**. La syntaxe est :

```console
nameresolver.exe hostname [optional: DNS Server]
```

Vous pouvez utiliser nameresolver pour vérifier les noms d’hôte dont dépend votre application. De cette façon, vous pouvez tester si des éléments de votre serveur DNS sont mal configurés ou si vous n’avez pas accès à votre serveur DNS. Vous pouvez identifier le serveur DNS qu’utilise votre application dans la console en examinant les variables d’environnement WEBSITE_DNS_SERVER et WEBSITE_DNS_ALT_SERVER.

> [!NOTE]
> nameresolver.exe ne fonctionne pas actuellement dans les conteneurs Windows personnalisés.
>

Vous pouvez utiliser l’outil suivant pour tester la connectivité TCP avec une combinaison hôte-port. Il s’agit de l’outil **tcpping**, dont la syntaxe est la suivante :

```console
tcpping.exe hostname [optional: port]
```

L’utilitaire **tcpping** vous indique si vous pouvez atteindre un hôte et un port spécifiques. Il ne peut réussir que si une application écoute au niveau de la combinaison hôte-port et si l’accès réseau de votre application à l’hôte et au port spécifiés est disponible.

#### <a name="debug-access-to-virtual-network-hosted-resources"></a>Déboguer l’accès aux ressources hébergées sur un réseau virtuel
Plusieurs choses peuvent empêcher votre application d’atteindre un hôte et un port spécifiques. La plupart du temps, il s’agit de l’une des trois raisons suivantes :

* **Un pare-feu se trouve sur le trajet.** Si vous utilisez un pare-feu sur le trajet, vous dépassez le délai d’expiration TCP. Dans ce cas, il est de 21 secondes. Utilisez l’outil **tcpping** pour tester la connectivité. Les délais d’expiration TCP peuvent avoir de nombreuses autres origines, mais commencez par vérifier ce point.
* **DNS n’est pas accessible.** Le délai d’expiration DNS est de 3 secondes par serveur DNS. Si vous avez deux serveurs DNS, le délai d’expiration est de 6 secondes. Utilisez nameresolver pour vérifier que le DNS fonctionne correctement. Vous ne pouvez pas utiliser nslookup, car il n’utilise pas le DNS avec lequel votre réseau virtuel est configuré. S’il n’est pas accessible, il se peut qu’un pare-feu ou un groupe de sécurité réseau (NSG) bloque l’accès au DNS ou que celui-ci est inopérant.

Si ces éléments ne suffisent pas à résoudre vos problèmes, commencez par vous poser les questions suivantes :

**Intégration au réseau virtuel régional**
* Votre destination est-elle une adresse non RFC1918 alors que la valeur de WEBSITE_VNET_ROUTE_ALL n’est pas 1 ?
* Y a-t-il un groupe de sécurité réseau qui bloque la sortie de votre sous-réseau d’intégration ?
* Si elle transite par Azure ExpressRoute ou un VPN, votre passerelle locale est-elle configurée pour réacheminer le trafic vers Azure ? Si vous pouvez accéder à des points de terminaison dans votre réseau virtuel, mais pas en local, vérifiez vos routes.
* Disposez-vous d’autorisations suffisantes pour définir la délégation sur le sous-réseau d’intégration ? Durant la configuration de l’intégration au réseau virtuel régional, votre sous-réseau d’intégration est délégué à Microsoft.Web/serverFarms. L’interface utilisateur de l’intégration au réseau virtuel délègue automatiquement le sous-réseau à Microsoft.Web/serverFarms. Si votre compte ne dispose pas d’autorisations de mise en réseau suffisantes pour définir la délégation, vous devez demander à une personne autorisée de configurer les attributs de votre sous-réseau d’intégration de manière à déléguer celui-ci. Pour déléguer manuellement le sous-réseau d’intégration, accédez à l’interface utilisateur du sous-réseau de Réseau virtuel Microsoft Azure et définissez la délégation sur Microsoft.Web/serverFarms.

**Intégration au réseau virtuel avec passerelle obligatoire**
* La plage d’adresses de point à site se trouve-t-elle dans les plages RFC 1918 (10.0.0.0 à 10.255.255.255, 172.16.0.0 à 172.31.255.255 ou 192.168.0.0 à 192.168.255.255) ?
* Le portail indique-t-il que la passerelle fonctionne ? Si votre passerelle est en panne, rétablissez-la.
* Les certificats apparaissent-ils comme étant synchronisés ou soupçonnez-vous une modification de la configuration réseau ?  Si vos certificats ne sont pas synchronisés ou si vous pensez qu’une modification apportée à la configuration de votre réseau virtuel n’a pas été synchronisée avec vos ASP, sélectionnez **Synchroniser le réseau**.
* Si vous utilisez un VPN, la passerelle locale est-elle configurée pour réacheminer le trafic vers Azure ? Si vous pouvez accéder à des points de terminaison dans votre réseau virtuel, mais pas en local, vérifiez vos routes.
* Essayez-vous d’utiliser une passerelle de coexistence qui prend en charge à la fois la connectivité point à site et la connectivité ExpressRoute ? Les passerelles de coexistence ne sont pas prises en charge avec l’intégration au réseau virtuel.

Le débogage des problèmes de mise en réseau constitue un défi, car cette opération ne vous permet pas de voir ce qui bloque l’accès à une combinaison hôte-port spécifique. Les causes peuvent inclure :

* Un pare-feu activé sur l’hôte empêche l’accès au port de l’application à partir de votre plage d’adresses IP de point à site. Des sous-réseaux croisés requièrent souvent un accès public.
* Votre hôte cible est hors-service.
* Votre application est arrêtée.
* L’IP ou le nom d’hôte sont incorrects.
* Votre application est à l’écoute sur un port autre que celui auquel vous vous attendiez. Vous pouvez faire correspondre votre ID de processus avec le port d’écoute en utilisant « netstat -aon » sur l’hôte du point de terminaison.
* Vos groupes de sécurité réseau sont configurés de telle sorte qu’ils empêchent l’accès à l’hôte et au port de votre application à partir de votre plage d’adresses IP de point à site.

Vous ne savez pas quelle adresse votre application utilise réellement. Cela peut être n’importe quelle adresse de la plage d’adresses de sous-réseau d’intégration ou de point à site. De ce fait, vous devez autoriser l’accès depuis toutes les adresses de la plage.

Étapes de débogage supplémentaires :

* Connectez-vous à une machine virtuelle de votre réseau virtuel et essayez d’atteindre la combinaison hôte-port de vos ressources. Pour tester l’accès à TCP, utilisez la commande PowerShell **test-netconnection**. La syntaxe est :

```powershell
test-netconnection hostname [optional: -Port]
```

* Démarrez une application sur une machine virtuelle, puis testez l’accès à ces hôte et port à partir de la console de votre application en utilisant l’outil **tcpping**.

#### <a name="on-premises-resources"></a>Ressources locales ####

Si votre application ne peut pas accéder à une ressource locale, vérifiez si vous pouvez atteindre la ressource à partir de votre réseau virtuel. Utilisez la commande PowerShell **test-netconnection** pour vérifier l’accès à TCP. Si votre machine virtuelle ne peut pas accéder à votre ressource locale, votre connexion VPN ou ExpressRoute n’est peut-être pas configurée correctement.

Si votre machine virtuelle hébergée sur le réseau virtuel peut atteindre votre système local, mais que votre application ne le peut pas, la raison en est probablement l’une des suivantes :

* Vos routes ne sont pas configurés avec vos plages d’adresses de sous-réseau ou de point à site dans votre passerelle locale.
* Vos groupes de sécurité réseau bloquent l’accès de votre plage d’adresses IP de point à site.
* Vos pare-feu locaux bloquent le trafic provenant de votre plage d’adresses IP de point à site.
* Vous tentez d’atteindre une adresse non RFC 1918 en utilisant la fonctionnalité d’intégration au réseau virtuel régional.
