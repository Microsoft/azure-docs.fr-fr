---
title: Limites de la taille des demandes adressées au pare-feu d’applications web et listes d’exclusions dans Azure Application Gateway - Portail Azure
description: Cet article fournit des informations sur la configuration des limites de la taille des demandes adressées au pare-feu d’applications web et des listes d’exclusions dans Application Gateway avec le portail Azure.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 02/20/2020
ms.author: victorh
ms.topic: conceptual
ms.openlocfilehash: 2d34641fdecfe334e84347efe1a2f64482cae74b
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040248"
---
# <a name="web-application-firewall-request-size-limits-and-exclusion-lists"></a>Limites de la taille des demandes adressées au pare-feu d’application web et listes d’exclusions

Le pare-feu d’applications web (WAF) Azure Application Gateway fournit une protection pour les applications web. Cet article décrit la configuration des limites de la taille des demandes adressées au WAF et des listes d’exclusions. Ces paramètres se trouvent dans la stratégie WAF associée à votre passerelle Application Gateway. Pour en savoir plus sur les stratégies WAF, consultez [Pare-feu d’applications web Azure sur Azure Application Gateway](ag-overview.md) et [Créer des stratégies de pare-feu d’applications web pour Application Gateway](create-waf-policy-ag.md).

## <a name="waf-exclusion-lists"></a>Listes d’exclusions du WAF

![Limites de la taille des demandes](../media/application-gateway-waf-configuration/waf-policy.png)

Les listes d’exclusion du WAF vous permettent d’omettre certains attributs de la demande dans une évaluation par le WAF. À titre d’exemple courant, citons les jetons Active Directory insérés qui sont utilisés pour les champs d’authentification ou de mot de passe. Ces attributs sont sujets à contenir des caractères spéciaux qui peuvent déclencher un faux positif dans les règles du pare-feu d’applications web. Une fois ajouté à la liste d’exclusions du WAF, un attribut n’est pris en considération par aucune règle du pare-feu d’applications web configurée et active. Les listes d’exclusions ont une portée globale.

Les attributs qui suivent peuvent être ajoutés aux listes d’exclusion par nom. Les valeurs du champ choisi ne sont pas évaluées par rapport aux règles WAF, mais leurs noms le sont toujours (voir l’Exemple 1 ci-dessous, où la valeur de l’en-tête User-Agent est exclue de l’évaluation WAF). Les listes d’exclusion suppriment l’inspection de la valeur du champ.

* En-têtes de requête
* Cookies de requête
* Vous pouvez ajouter le nom d’un attribut de requête (argument) comme un élément d’exclusion, par exemple :

   * Nom du champ de formulaire
   * Entité JSON
   * Arguments de chaîne de requête de l’URL

Vous pouvez spécifier une correspondance exacte avec l'en-tête ou le corps d'une requête, un cookie ou un attribut de chaîne de requête  ou spécifier des correspondances partielles. Les règles d'exclusion ont une portée globale, et s'appliquent à toutes les pages et à toutes les règles.

Voici les opérateurs de critères de correspondance pris en charge :

- **Est égal à**  :  cet opérateur est utilisé pour une correspondance exacte. Par exemple, pour sélectionner l’en-tête **bearerToken** , utilisez l’opérateur d’égalité avec le sélecteur défini sur **bearerToken**.
- **Commence par**  : cet opérateur correspond à tous les champs qui commencent par la valeur de sélecteur spécifiée.
- **Se termine par**  :  cet opérateur correspond à tous les champs de demande qui se terminent par la valeur de sélecteur spécifiée.
- **Contient**  : cet opérateur correspond à tous les champs de demande qui se contiennent la valeur de sélecteur spécifiée.
- **Est égal à** : Cet opérateur correspond à tous les champs de la requête. * sera la valeur du sélecteur.

Dans tous les cas, la correspondance respecte la casse, et les expressions régulières ne sont pas autorisées en guise de sélecteurs.

> [!NOTE]
> Si vous souhaitez en savoir plus et obtenir de l’aide, veuillez consulter la section [Résolution des problèmes WAF](web-application-firewall-troubleshoot.md).

### <a name="examples"></a>Exemples

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Les exemples suivants montrent comment utiliser les exclusions.

#### <a name="example-1"></a>Exemple 1

Dans cet exemple, vous souhaitez exclure l’en-tête user-agent. Cet en-tête de requête contient une chaîne caractéristique qui permet aux pairs de protocole réseau d’identifier le type d’application, le système d’exploitation, l’éditeur de logiciels ou la version du logiciel de l’agent utilisateur de logiciels demandeur. Pour plus d’informations, consultez la section [Agent utilisateur](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent).

Il peut y avoir plusieurs raisons de désactiver l’évaluation de cet en-tête. Le WAF peut voir une chaîne et la considérer comme malveillante. Par exemple, l’attaque SQL classique « x = x » dans une chaîne. Dans certains cas, il peut s’agir de trafic légitime. Par conséquent, vous devrez peut-être exclure cet en-tête de l’évaluation par le WAF.

La cmdlet Azure PowerShell suivante permet d’exclure l’en-tête user-agent de l’évaluation :

```azurepowershell
$exclusion1 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestHeaderNames" `
   -SelectorMatchOperator "Equals" `
   -Selector "User-Agent"
```
#### <a name="example-2"></a>Exemple 2

Cet exemple exclut la valeur du paramètre *utilisateur* qui est transmis dans la requête via l’URL. Par exemple, imaginons qu’il est courant dans votre environnement que le champ utilisateur contienne une chaîne que le WAF considère comme contenu malveillant, et donc que celle-ci soit bloquée.  Vous pouvez exclure le paramètre utilisateur dans ce cas, afin que le WAF n’évalue rien qui appartienne à ce champ.

La cmdlet Azure PowerShell suivante permet d’exclure le paramètre utilisateur de l’évaluation :

```azurepowershell
$exclusion2 = New-AzApplicationGatewayFirewallExclusionConfig `
   -MatchVariable "RequestArgNames" `
   -SelectorMatchOperator "StartsWith" `
   -Selector "user"
```
Par conséquent, si l'URL `http://www.contoso.com/?user%281%29=fdafdasfda` est transmise au WAF, elle n'évaluera pas la chaîne **fdafdasfda** , mais évaluera toujours le nom de paramètre **user%281%29**. 

## <a name="waf-request-size-limits"></a>Limites de la taille des demandes adressées au WAF



Le pare-feu d’applications web vous permet de configurer des limites maximale et minimale de la taille des demandes. Les configurations de limites à deux tailles suivantes sont disponibles :

- Le champ de la taille maximale du corps de la requête est spécifié en kilo-octets et contrôle la limite globale de la taille de la requête à l’exclusion de tout chargement de fichier. La valeur de ce champ est comprise entre 1 ko et 128 ko. La valeur par défaut pour la taille du corps de la demande est de 128 Ko.
- Le champ de la limite de chargement de fichier est spécifié en Mo et régit la taille maximale autorisée pour la limite de chargement de fichier. Ce champ peut avoir une valeur minimale de 1 Mo et les valeurs maximales suivantes :

   - 100 Mo pour les passerelles v1 WAF moyennes
   - 500 Mo pour les grandes passerelles v1 WAF
   - 750 Mo pour les passerelles v2 WAF 

 La valeur par défaut pour la limite de chargement de fichier est de 100 Mo.

Le WAF offre également un bouton configurable qui permet d’activer ou de désactiver l’inspection du corps de la demande. Par défaut, l’inspection du corps de la demande est activée. Si l’inspection du corps de la requête est désactivée, le WAF n’évalue pas le contenu du corps du message HTTP. Dans ce cas, le WAF continue d’appliquer ses règles sur les en-têtes, les cookies et les URI. Si l’inspection du corps de la demande est désactivée, le champ de la taille maximale du corps de la demande n’est pas applicable et ne peut pas être défini. La désactivation de l’inspection du corps de la demande permet l’envoi au WAF de messages dont la taille est supérieure à 128 Ko, mais le corps du message n’est pas inspecté.

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré vos paramètres du pare-feu d’applications web, vous pouvez apprendre à afficher vos journaux d’activité WAF. Pour plus d’informations, consultez [Diagnostics Application Gateway](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).
