---
title: Hébergement de plusieurs sites sur Azure Application Gateway
description: Cet article fournit une vue d’ensemble de la prise en charge de sites multiples pour la passerelle Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.date: 07/20/2020
ms.author: surmb
ms.topic: conceptual
ms.openlocfilehash: 53f6f37454de886934a483b40daad24204958baf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87474323"
---
# <a name="application-gateway-multiple-site-hosting"></a>Hébergement de plusieurs sites Application Gateway

L’hébergement de plusieurs sites vous permet de configurer plusieurs applications web sur le même port d’une passerelle Application Gateway. Vous pouvez ainsi configurer une topologie plus efficace pour vos déploiements en ajoutant plus de 100 sites web à une même passerelle d’application. Chaque site web peut être dirigé vers son propre pool principal. Par exemple, trois domaines (contoso.com, fabrikam.com et adatum.com) pointent vers l’adresse IP de la passerelle d’application. Vous créez trois écouteurs multisites, et vous configurez les paramètres de port et de protocole de chaque écouteur. 

Vous pouvez également définir des noms d’hôtes avec caractères génériques dans un écouteur multisite et jusqu’à cinq noms d’hôtes par écouteur. Pour plus d’informations, consultez [Noms d’hôtes avec caractères génériques dans l’écouteur](#wildcard-host-names-in-listener-preview).

:::image type="content" source="./media/multiple-site-overview/multisite.png" alt-text="Passerelle d’application multisite":::

> [!IMPORTANT]
> Pour la référence SKU v1, les règles sont traitées dans l’ordre où elles sont répertoriées sur le portail. Pour la référence SKU v2, les correspondances exactes ont une priorité plus élevée. Il est vivement recommandé de configurer des écouteurs multisites avant un écouteur de base.  De cette façon, vous êtes sûr que le trafic est acheminé vers le serveur principal approprié. Si un écouteur de base est indiqué en premier et correspond à une demande entrante, elle est traitée par cet écouteur.

Les requêtes adressées à `http://contoso.com` sont acheminées vers ContosoServerPool, tandis que les requêtes adressées à `http://fabrikam.com` sont acheminées vers FabrikamServerPool.

De même, vous pouvez héberger de nombreux sous-domaines du même domaine parent sur le même déploiement de passerelle d’application. Par exemple, vous pouvez héberger `http://blog.contoso.com` et `http://app.contoso.com` sur un déploiement unique de passerelle d’application.

## <a name="wildcard-host-names-in-listener-preview"></a>Noms d’hôtes avec caractères génériques dans l’écouteur (préversion)

Application Gateway autorise l’acheminement en fonction de l’hôte à l’aide de l’écouteur HTTP(S) multisite. Il est maintenant possible d’utiliser des caractères génériques comme l’astérisque (*) et le point d’interrogation (?) dans le nom d’hôte, et jusqu’à cinq noms d’hôtes par écouteur HTTP(S) multisite. Par exemple : `*.contoso.com`.

En utilisant un caractère générique dans le nom d’hôte, vous pouvez faire correspondre plusieurs noms d’hôtes dans un seul écouteur. Par exemple, `*.contoso.com` peut correspondre à `ecom.contoso.com`, `b2b.contoso.com` ainsi qu’à `customer1.b2b.contoso.com`, etc. À l’aide d’un tableau de noms d’hôtes, il est possible de configurer plusieurs noms d’hôtes pour un écouteur, pour acheminer les demandes vers un pool principal. Par exemple, un écouteur peut contenir `contoso.com, fabrikam.com` qui accepte les demandes pour les deux noms d’hôtes.

:::image type="content" source="./media/multiple-site-overview/wildcard-listener-diag.png" alt-text="Passerelle d’application multisite":::

>[!NOTE]
> Cette fonctionnalité, en préversion, n’est disponible que pour les références SKU Standard_v2 et WAF_v2 d’Application Gateway. Pour plus d’informations sur les préversions, consultez les [conditions d’utilisation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

>[!NOTE]
>Cette fonctionnalité n’est à l’heure actuelle disponible que par le biais [d’Azure PowerShell](tutorial-multiple-sites-powershell.md) et [d’Azure CLI](tutorial-multiple-sites-cli.md). La prise en charge du Portail sera bientôt disponible.
> Notez que dans la mesure où la prise en charge du portail n’est pas complètement disponible, si vous n’utilisez que le paramètre HostNames, l’écouteur apparaît comme un écouteur de base dans le portail et la colonne Nom d'hôte du mode liste de l’écouteur n’affiche pas les noms d’hôte configurés. Pour toute modification apportée à un écouteur avec caractères génériques, assurez-vous d’utiliser Azure PowerShell ou CLI jusqu’à ce qu’il soit pris en charge dans le portail.

Dans [Azure PowerShell](tutorial-multiple-sites-powershell.md), vous devez utiliser `-HostNames` au lieu de `-HostName`. Avec -HostNames, vous pouvez spécifier jusqu’à cinq noms d’hôtes sous forme de valeurs séparées par des virgules et utiliser des caractères génériques. Par exemple : `-HostNames "*.contoso.com,*.fabrikam.com"`

Dans [Azure CLI](tutorial-multiple-sites-cli.md), vous devez utiliser `--host-names` au lieu de `--host-name`. Avec --host-names, vous pouvez spécifier jusqu’à cinq noms d’hôtes sous forme de valeurs séparées par des virgules et utiliser des caractères génériques. Par exemple : `--host-names "*.contoso.com,*.fabrikam.com"`

### <a name="allowed-characters-in-the-host-names-field"></a>Caractères autorisés dans le champ des noms d’hôtes

* `(A-Z,a-z,0-9)` : caractères alphanumériques
* `-` : trait d’union ou signe moins
* `.` : point comme délimiteur
*   `*` : peut correspondre à plusieurs caractères dans la plage autorisée
*   `?` : peut correspondre à un seul caractère dans la plage autorisée

### <a name="conditions-for-using-wildcard-characters-and-multiple-host-names-in-a-listener"></a>Conditions pour utiliser des caractères génériques et plusieurs noms d’hôtes dans un écouteur

*   Il n’est possible de spécifier que cinq noms d’hôtes maximum dans un même écouteur.
*   L’astérisque `*` ne peut être spécifié qu’une seule fois dans un composant de nom de style de domaine ou de nom d’hôte, par exemple, composant1 *.composant2*.composant3. `(*.contoso-*.com)` est valide.
*   Un nom d’hôte ne peut contenir que deux astérisques `*` maximum. Par exemple, `*.contoso.*` est valide, `*.contoso.*.*.com` non.
*   Un nom d’hôte ne peut contenir que quatre caractères génériques maximum. Par exemple, `????.contoso.com` et `w??.contoso*.edu.*` sont valides, `????.contoso.*` non.
*   Il n’est pas possible d’utiliser conjointement un astérisque `*` et un point d’interrogation `?` dans un composant de nom d’hôte (`*?`, `?*` ou `**`). Par exemple, ni `*?.contoso.com` ni `**.contoso.com` ne sont valides.

### <a name="considerations-and-limitations-of-using-wildcard-or-multiple-host-names-in-a-listener"></a>Considérations et limitations liées à l’utilisation d’un nom d’hôte avec caractères génériques ou de plusieurs noms d’hôtes dans un écouteur

*   [L’arrêt SSL et le protocole SSL de bout en bout](ssl-overview.md) obligent à configurer le protocole en tant que protocole HTTPS et à charger un certificat qui sera utilisé dans la configuration de l’écouteur. S’il s’agit d’un écouteur multisite, vous pouvez également entrer le nom d’hôte, généralement le nom commun du certificat SSL. Lorsque vous spécifiez plusieurs noms d’hôtes dans l’écouteur ou utilisez des caractères génériques, vous devez tenir compte des points suivants :
    *   S’il s’agit d’un nom d’hôte avec caractères génériques comme *.contoso.com, vous devez charger un certificat générique avec nom commun du type *.contoso.com.
    *   Si plusieurs noms d’hôtes sont spécifiés dans le même écouteur, vous devez charger un certificat SAN (autre nom de l’objet) dont le nom commun correspond aux noms d’hôtes.
*   Il n’est pas possible de recourir à une expression régulière pour spécifier le nom d’hôte. Seuls des caractères génériques comme l’astérisque (*) et le point d’interrogation (?) peuvent être utilisés pour former le modèle de nom d’hôte.
*   Pour le contrôle d’intégrité du serveur principal, il n’est pas possible d’associer [plusieurs sondes personnalisées](application-gateway-probe-overview.md) par paramètre HTTP. Vous pouvez en revanche sonder l’un des sites web du serveur principal ou utiliser « 127.0.0.1 » pour sonder localhost sur le serveur principal. Toutefois, si un nom d’hôte avec caractères génériques ou plusieurs noms d’hôtes sont utilisés dans un écouteur, les demandes de tous les modèles de domaine spécifiés sont acheminées vers le pool principal en fonction du type de règle (de base ou basée sur le chemin).
*   La propriété « hostname » prend en entrée une chaîne, dans laquelle vous ne pouvez mentionner qu’un seul nom de domaine sans caractères génériques ; la propriété « hostnames » prend en entrée un tableau de chaînes, où peuvent être mentionnés jusqu’à cinq noms de domaine avec caractères génériques. Il n’est cependant pas possible d’utiliser les deux propriétés en même temps.
*   Vous ne pouvez pas créer une règle de [redirection](redirect-overview.md) avec un écouteur cible qui utilise un nom d’hôte avec caractères génériques ou plusieurs noms d’hôtes.

Pour des instructions pas à pas de configuration des noms d’hôtes avec caractères génériques dans un écouteur multisite, consultez [Création de plusieurs sites à l’aide d’Azure PowerShell](tutorial-multiple-sites-powershell.md) ou [à l’aide d’Azure CLI](tutorial-multiple-sites-cli.md).

## <a name="host-headers-and-server-name-indication-sni"></a>En-têtes d’hôte et Indication du nom du serveur (SNI)

Il existe trois mécanismes pour activer l’hébergement de plusieurs sites dans la même infrastructure.

1. Hébergez plusieurs applications web, chacune sur une adresse IP unique.
2. Utilisez le nom d’hôte pour héberger plusieurs applications web sur la même adresse IP.
3. Utilisez des ports différents pour héberger plusieurs applications web sur la même adresse IP.

Actuellement, le service Application Gateway prend en charge une seule IP publique sur laquelle il écoute le trafic. Par conséquent, la prise en charge de plusieurs applications, chacune avec sa propre adresse IP, n’est pas disponible actuellement. 

Application Gateway prend en charge plusieurs applications qui écoutent sur des ports différents, mais ce scénario requiert que les applications acceptent le trafic sur les ports non standard. Il ne s’agit généralement pas de la configuration de votre choix.

Application Gateway se base sur des en-têtes d’hôte HTTP 1.1 pour héberger plusieurs sites web sur les mêmes adresses IP et port. Les sites hébergés sur Application Gateway peuvent également prendre en charge le déchargement TLS avec l’extension Indication du nom du serveur (SNI) TLS. Ce scénario signifie que le navigateur du client et la batterie de serveurs web principale doivent prendre en charge HTTP/1.1 et l’extension TLS définie dans RFC 6066.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment configurer l’hébergement de plusieurs sites dans Application Gateway
* [En passant par le portail Azure](create-multiple-sites-portal.md)
* [Utilisation de Microsoft Azure PowerShell](tutorial-multiple-sites-powershell.md) 
* [Utilisation de l’interface de ligne de commande Azure](tutorial-multiple-sites-cli.md)

Vous pouvez consulter le [modèle Resource Manager avec l’hébergement de sites multiples](https://github.com/Azure/azure-quickstart-templates/blob/master/201-application-gateway-multihosting) pour un déploiement basé sur un modèle de bout en bout.
