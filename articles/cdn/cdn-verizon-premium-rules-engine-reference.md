---
title: Référence du moteur de règles Azure CDN | Microsoft Docs
description: Documentation de référence sur les fonctionnalités et conditions de correspondance du moteur de règles Azure CDN.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: 295bc0a20a547bf944f48af6711b18af34571b02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91362578"
---
# <a name="azure-cdn-from-verizon-premium-rules-engine-reference"></a>Documentation de référence sur le moteur de règles Azure CDN de Verizon Premium

Cet article fournit les descriptions détaillées des conditions de correspondance et fonctionnalités disponibles pour le [moteur de règles](cdn-verizon-premium-rules-engine.md) Azure Content Delivery Network (CDN).

Le moteur de règles est conçu pour être l’autorité finale sur la façon dont certains types de demandes sont traités par le CDN.

**Utilisations courantes** :

- Remplacer ou définir une stratégie de cache personnalisée.
- Sécuriser ou refuser les demandes de contenu sensible.
- Rediriger les demandes.
- Stocker les données de journal personnalisé.
## <a name="key-concepts"></a>Concepts clés
Les concepts clés de la configuration du moteur de règles sont décrits ci-dessous.
### <a name="draft"></a>Brouillon
Le brouillon d’une stratégie est constitué d’une ou de plusieurs règles destinées à identifier les demandes et l’ensemble des actions qui leur seront appliquées. Il s’agit d’un travail en cours qui permet des mises à jour de configuration fréquentes sans affecter le trafic du site. Une fois prêt à être finalisé, le brouillon doit être converti en une stratégie en lecture seule.

### <a name="rule"></a>Règle
Une règle identifie un ou plusieurs types de demandes et l’ensemble des actions qui leur seront appliquées.

Elle comprend : 

- Un ensemble d’expressions conditionnelles qui définissent la logique d’identification des demandes.
- Un ensemble de conditions de correspondance qui définissent les critères d’identification des demandes.
- Un ensemble de fonctionnalités qui définissent la façon dont le réseau CDN gère les requêtes ci-dessus.
Ces éléments sont mis en surbrillance dans l’illustration suivante.

![Capture d’écran avec des étiquettes montrant l’expression conditionnelle, la correspondance et les fonctionnalités d’une règle.](./media/cdn-verizon-premium-rules-engine-reference/verizon-rules-engine-reference.png)

### <a name="policy"></a>Stratégie
Une stratégie, qui se compose d’un ensemble de règles en lecture seule, permet d’effectuer les opérations suivantes :

- Créer, stocker et gérer plusieurs variantes des règles.
- Restaurer une version déployée auparavant.
- Préparer à l’avance des règles propres aux événements (par exemple, une règle qui redirige le trafic à la suite d’une opération de maintenance provenant du client).

> [!NOTE]
> Bien que seule une stratégie soit autorisée par environnement, les stratégies peuvent être déployées en fonction des besoins.

### <a name="deploy-request"></a>Demande de déploiement
Une demande de déploiement représente une procédure simple et rationalisée par le biais de laquelle une stratégie peut être appliquée rapidement à l’environnement intermédiaire ou de production. Un historique des demandes de déploiement est fourni pour faciliter le suivi des modifications appliquées à ces environnements.

> [!NOTE]
> Seules les demandes non approuvées par notre système automatisé de validation et de détection des erreurs réclament une révision et une approbation manuelles.

### <a name="rule-precedence"></a>Précédence des règles
Les règles contenues dans une stratégie sont généralement traitées dans l’ordre dans lequel elles apparaissent (soit de haut en bas). Si la demande correspond à des règles en conflit, la dernière règle à traiter est prioritaire.

### <a name="policy-deployment-workflow"></a>Workflow de déploiement de stratégies
Le workflow par le biais duquel une stratégie peut être appliquée à l’environnement de production ou intermédiaire est illustré ci-dessous.

![Workflow de déploiement de stratégies](./media/cdn-verizon-premium-rules-engine-reference/policy-deployment-workflow.png)

|Étape |Description |
|---------|---------|
|[Créer un brouillon](https://docs.vdms.com/cdn/index.html#HRE/AdministeringDraftsandRules.htm#Create)    |    Un brouillon est constitué d’un ensemble de règles qui définissent la façon dont les demandes de contenu doivent être gérées par le réseau CDN.     |
|Verrouiller le brouillon   |     Une fois finalisé, le brouillon doit être verrouillé et converti en une stratégie en lecture seule.    |
|[Envoyer une demande de déploiement](https://docs.vdms.com/cdn/index.html#HRE/DeployRequest.htm)   |   <br> Une demande de déploiement permet d’appliquer une stratégie à un trafic de test ou de production.</br> <br>Envoyez une demande de déploiement à l’environnement intermédiaire ou de production.</br>     |
|Vérifier la demande de déploiement   |    <br>La demande de déploiement subit une validation et une détection d’erreurs automatiques.</br><br>Si la majorité d’entre elles sont approuvées automatiquement, une révision manuelle reste nécessaire pour les stratégies complexes.</br>   |
|Déploiement de la stratégie ([Intermédiaire](https://docs.vdms.com/cdn/index.html#HRE/Environment.htm#Staging))   |  <br> Après approbation de la demande de déploiement dans l’environnement intermédiaire, la stratégie est appliquée à l’environnement intermédiaire. Cet environnement permet de tester une stratégie sur du trafic de site fictif.</br><br>Une fois la stratégie prête à être appliquée au trafic de site actif, une nouvelle demande de déploiement dans l’environnement de production doit être envoyée.</br>      |
|Déploiement de la stratégie ([Production](https://docs.vdms.com/cdn/index.html#HRE/Environment.htm#Producti))   |  Après approbation de la demande de déploiement dans l’environnement de production, une stratégie est appliquée à l’environnement de production. Cet environnement permet à la stratégie de servir d’autorité finale pour déterminer comment le réseau CDN doit gérer le trafic actif.     |
## <a name="syntax"></a>Syntaxe

La façon dont les caractères spéciaux sont traités varie en fonction de la façon dont une condition de correspondance ou une fonctionnalité gère les valeurs de texte. Une condition de correspondance ou une fonctionnalité peut interpréter le texte de l’une des manières suivantes :

- [**Valeurs littérales**](#literal-values)
- [**Valeurs de caractère générique**](#wildcard-values)
- [**Expressions régulières**](#regular-expressions)

### <a name="literal-values"></a>Valeurs littérales

Le texte interprété comme une valeur littérale traite tous les caractères spéciaux, à l’exception du symbole %, comme une partie de la valeur qui doit être mise en correspondance. En d’autres termes, une condition de correspondance littérale définie sur `\'*'\` n’est remplie que si cette valeur exacte (c’est-à-dire `\'*'\`) est trouvée.

Un symbole de pourcentage est utilisé pour indiquer l’encodage des URL (par exemple, `%20`).

### <a name="wildcard-values"></a>Valeurs de caractère générique

Le texte interprété comme un caractère générique attribue une signification supplémentaire aux caractères spéciaux. Le tableau ci-dessous décrit comment le jeu de caractères suivant est interprété :

Caractère | Description
----------|------------
\ | Une barre oblique inverse est utilisée pour échapper les caractères spécifiés dans ce tableau. Une barre oblique inverse doit être spécifiée juste avant le caractère spécial à échapper.<br/>Par exemple, la syntaxe suivante échappe un astérisque : `\*`
% | Un symbole de pourcentage est utilisé pour indiquer l’encodage des URL (par exemple, `%20`).
\* | Un astérisque est un caractère générique représentant un ou plusieurs caractères.
Espace | Un caractère d’espace indique qu’une condition de correspondance peut être remplie par les valeurs ou les modèles spécifiés.
'valeur' | Un guillemet simple n’a pas de signification particulière. Toutefois, un jeu de guillemets simples est utilisé pour indiquer qu’une valeur doit être traitée comme une valeur littérale. Il peut être utilisé selon les manières suivantes :<br><br/>- Il permet de remplir une condition de correspondance lorsque la valeur spécifiée correspond à une partie de la valeur de comparaison.  Par exemple, `'ma'` peut correspondre à l’une des chaînes suivantes : <br/><br/>/business/**ma**rathon/asset.htm<br/>**ma**p.gif<br/>/business/template.**ma**p<br /><br />- Il permet de spécifier un caractère spécial en tant que caractère littéral. Par exemple, vous pouvez spécifier un caractère d’espace littéral en plaçant un caractère d’espace dans un jeu de guillemets simples (c’est-à-dire `' '` ou `'sample value'`).<br/>- Il permet de spécifier une valeur vide. Spécifiez une valeur vide en entrant un jeu de guillemets simples (c’est-à-dire '').<br /><br/>**Important :**<br/>- Si la valeur spécifiée ne contient pas de caractère générique, elle est automatiquement considérée comme une valeur littérale, ce qui signifie qu’il n’est pas nécessaire de spécifier un jeu de guillemets simples.<br/>- Si une barre oblique inverse n’échappe pas à un autre caractère dans ce tableau, elle est ignorée si un jeu de guillemets simples est entré.<br/>- Une autre manière de spécifier un caractère spécial en tant que caractère littéral consiste à l’échapper à l’aide d’une barre oblique inverse (c’est-à-dire `\`).

### <a name="regular-expressions"></a>Expressions régulières

Les expressions régulières définissent un modèle qui est recherché dans une valeur de texte. La notation d’une expression régulière définit des significations spécifiques pour une variété de symboles. Le tableau ci-dessous indique comment les caractères spéciaux sont traités par les conditions de correspondance et les fonctionnalités prenant en charge les expressions régulières.

Caractère spécial | Description
------------------|------------
\ | Une barre oblique inverse échappe le caractère qui la suit, ce caractère est alors traité comme une valeur littérale plutôt que d’utiliser sa signification d’expression régulière. Par exemple, la syntaxe suivante échappe un astérisque : `\*`
% | La signification d’un symbole de pourcentage dépend de son utilisation.<br/><br/> `%{HTTPVariable}`: cette syntaxe identifie une variable HTTP.<br/>`%{HTTPVariable%Pattern}`: cette syntaxe utilise un symbole de pourcentage pour identifier une variable HTTP et comme délimiteur.<br />`\%`: l’échappement d’un symbole de pourcentage permet de l’utiliser comme une valeur littérale ou d’indiquer l’encodage des URL (par exemple, `\%20`).
\* | Un astérisque permet de mettre en correspondance une ou plusieurs fois le caractère qui le précède.
Espace | Un caractère d’espace est généralement traité comme un caractère littéral.
'valeur' | Les guillemets simples sont traités comme des caractères littéraux. Un jeu de guillemets simples n’a pas de signification particulière.

Les conditions et fonctionnalités de correspondance qui prennent en charge les expressions régulières acceptent les modèles définis par des expressions régulières compatibles Perl (PCRE).



## <a name="next-steps"></a>Étapes suivantes

- [Conditions de correspondance du moteur de règles](cdn-verizon-premium-rules-engine-reference-match-conditions.md)
- [Expressions conditionnelles du moteur de règles](cdn-verizon-premium-rules-engine-reference-conditional-expressions.md)
- [Fonctionnalités du moteur de règles](cdn-verizon-premium-rules-engine-reference-features.md)
- [Remplacement du comportement HTTP à l’aide du moteur de règles](cdn-verizon-premium-rules-engine.md)
- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
