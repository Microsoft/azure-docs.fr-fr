---
title: Gestion des configurations pour Microsoft Threat Modeling Tool
titleSuffix: Azure
description: En savoir plus sur la gestion de la configuration pour Threat Modeling Tool. Voir les informations sur l’atténuation et les exemples de code.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 8cbe6b39bda0815c4981c497c07750136bcc9dba
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517482"
---
# <a name="security-frame-configuration-management--mitigations"></a>Infrastructure de sécurité : gestion des configurations | Atténuation des risques 
| Produit/Service | Article |
| --------------- | ------- |
| **Application Web** | <ul><li>[Implémenter la stratégie de sécurité de contenu (CSP) et désactiver l’exécution de scripts JavaScript inline](#csp-js)</li><li>[Activer le filtre XSS du navigateur](#xss-filter)</li><li>[Désactiver le traçage et le débogage dans les applications ASP.NET avant le déploiement](#trace-deploy)</li><li>[Accéder aux scripts JavaScript tiers émanant uniquement de sources approuvées](#js-trusted)</li><li>[S’assurer que les pages ASP.NET authentifiées incorporent des techniques de défense contre les attaques par redirection d’interface utilisateur ou détournement de clics](#ui-defenses)</li><li>[S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur les applications web ASP.NET](#cors-aspnet)</li><li>[Activer l’attribut ValidateRequest sur les pages ASP.NET](#validate-aspnet)</li><li>[Utiliser les dernières versions des bibliothèques JavaScript hébergées localement](#local-js)</li><li>[Désactiver la détection MIME automatique](#mime-sniff)</li><li>[Supprimer les en-têtes de serveur standard de l’offre Sites Web Microsoft Azure pour éviter la création d’une empreinte numérique](#standard-finger)</li></ul> |
| **Sauvegarde de la base de données** | <ul><li>[Configurer un pare-feu Windows pour accéder au moteur de base de données](#firewall-db)</li></ul> |
| **API Web** | <ul><li>[S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur API Web ASP.NET](#cors-api)</li><li>[Chiffrer les sections des fichiers de configuration de l’API Web qui contiennent des données sensibles](#config-sensitive)</li></ul> |
| **Appareil IoT** | <ul><li>[S’assurer que toutes les interfaces d’administration sont sécurisées avec des informations d’identification fortes](#admin-strong)</li><li>[S’assurer que le code inconnu ne peut pas s’exécuter sur les appareils](#unknown-exe)</li><li>[Chiffrer les partitions du système d’exploitation et les partitions supplémentaires de l’appareil IoT avec BitLocker](#partition-iot)</li><li>[S’assurer que seuls les services/fonctionnalités minimaux sont activés sur les appareils](#min-enable)</li></ul> |
| **Passerelle de champ IoT** | <ul><li>[Chiffrer les partitions du système d’exploitation et les partitions supplémentaires de la passerelle de champ IoT avec Bitlocker](#field-bit-locker)</li><li>[S’assurer que les informations d’identification de connexion par défaut de la passerelle de champ sont modifiées lors de l’installation](#default-change)</li></ul> |
| **Passerelle de cloud IoT** | <ul><li>[S’assurer que la passerelle cloud implémente un processus garantissant la mise à jour continue du micrologiciel des appareils connectés](#cloud-firmware)</li></ul> |
| **Délimitation d’approbation machine** | <ul><li>[S’assurer que les appareils disposent de contrôles de sécurité des points de terminaison configurés conformément aux directives organisationnelles](#controls-policies)</li></ul> |
| **Stockage Azure** | <ul><li>[Garantir une gestion sécurisée des clés d’accès de stockage Azure](#secure-keys)</li><li>[S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur le stockage Azure](#cors-storage)</li></ul> |
| **WCF** | <ul><li>[Activer la fonctionnalité de limitation de service WCF](#throttling)</li><li>[Divulgation d’informations WCF par le biais des métadonnées](#info-metadata)</li></ul> | 

## <a name="implement-content-security-policy-csp-and-disable-inline-javascript"></a><a id="csp-js"></a>Implémenter la stratégie de sécurité de contenu (CSP) et désactiver l’exécution de scripts JavaScript inline

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [An Introduction to Content Security Policy (Présentation de la stratégie de sécurité de contenu)](https://www.html5rocks.com/en/tutorials/security/content-security-policy/), [Content Security Policy Reference (Informations de référence sur la stratégie de sécurité de contenu)](https://content-security-policy.com/), [Security features (Fonctionnalités de sécurité)](https://developer.microsoft.com/microsoft-edge/platform/documentation/dev-guide/security/), [Introduction to content security policy (Présentation de la stratégie de sécurité de contenu)](https://github.com/webplatform/webplatform.github.io/tree/master/docs/tutorials/content-security-policy), [Can I use CSP? (Puis-je utiliser CSP ?)](https://caniuse.com/#feat=contentsecuritypolicy) |
| **Étapes** | <p>La stratégie de sécurité de contenu (CSP, Content Security Policy) est un mécanisme de protection fiable, correspondant à une norme W3C, qui permet aux propriétaires d’applications web de contrôler le contenu incorporé dans leur site. La stratégie CSP est ajoutée sous la forme d’un en-tête de réponse HTTP sur le serveur web et est appliquée côté client par les navigateurs. Cette stratégie repose sur une liste verte : un site web peut déclarer un ensemble de domaines approuvés à partir desquels un contenu actif tel que JavaScript peut être chargé.</p><p>La stratégie CSP procure les avantages de sécurité suivants :</p><ul><li>**Protection contre l’exécution de scripts intersites (XSS, Cross-Site Scripting) :** si une page est vulnérable aux attaques XSS, un attaquant peut exploiter cette faille de 2 manières :<ul><li>Injection du code `<script>malicious code</script>`. Ce type d’attaque ne fonctionnera pas en raison d’une restriction de base 1 de la stratégie CSP.</li><li>Injection du code `<script src="http://attacker.com/maliciousCode.js"/>`. Ce type d’attaque échouera, car le domaine contrôlé par l’attaquant ne figurera pas dans la autorisés de la stratégie CSP.</li></ul></li><li>**Contrôle de l’exfiltration des données :** si un contenu malveillant présent sur une page web tente de se connecter à un site web externe et de voler des données, la stratégie CSP annule la connexion. Ce comportement découle du fait que le domaine cible ne figure pas dans la liste verte de la stratégie CSP.</li><li>**Protection contre le détournement de clics :** le détournement de clics est une technique d’attaque par laquelle un attaquant superpose un cadre caché à un site web authentique et incite les utilisateurs à cliquer sur des éléments de l’interface utilisateur. À l’heure actuelle, la technique de défense contre le détournement de clics consiste à configurer un en-tête de réponse X-Frame-Options. Toutefois, certains navigateurs ne respectent pas cet en-tête. L’application de la stratégie CSP constitue donc un moyen de protection standard contre le détournement de clics</li><li>**Signalement des attaques en temps réel :** en cas d’attaque par injection sur un site web protégé par la stratégie CSP, les navigateurs envoient automatiquement une notification à un point de terminaison configuré sur le serveur web. De cette façon, la stratégie CSP fait office de système d’avertissement en temps réel.</li></ul> |

### <a name="example"></a>Exemple
Exemple de stratégie : 
```csharp
Content-Security-Policy: default-src 'self'; script-src 'self' www.google-analytics.com 
```
Cette stratégie n’autorise le chargement de scripts qu’à partir du serveur de l’application web et du serveur Google Analytics. Les scripts chargés à partir de tout autre site seront rejetés. Lorsque la stratégie CSP est activée sur un site web, les fonctionnalités ci-après sont automatiquement désactivées afin de prévenir les attaques XSS. 

### <a name="example"></a>Exemple
Les scripts inline ne s’exécuteront pas. Voici quelques exemples de scripts inline. 
```javascript
<script> some Javascript code </script>
Event handling attributes of HTML tags (e.g., <button onclick="function(){}">
javascript:alert(1);
```

### <a name="example"></a>Exemple
Les chaînes ne seront pas évaluées en tant que code. 
```javascript
Example: var str="alert(1)"; eval(str);
```

## <a name="enable-browsers-xss-filter"></a><a id="xss-filter"></a>Activer le filtre XSS du navigateur

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [XSS Protection Filter (Filtre de protection anti-XSS)](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) |
| **Étapes** | <p>La configuration de l’en-tête de réponse X-XSS-Protection contrôle le filtre de protection contre l’exécution de scripts intersites du navigateur. Cet en-tête de réponse peut présenter les valeurs suivantes :</p><ul><li>`0:`. Le filtre est désactivé.</li><li>`1: Filter enabled`. En cas de détection d’une attaque par exécution de script intersites, le navigateur assainit la page afin de neutraliser l’attaque.</li><li>`1: mode=block : Filter enabled`. En cas de détection d’une attaque XSS, le navigateur empêche le rendu de la page au lieu d’assainir la page.</li><li>`1: report=http://[YOURDOMAIN]/your_report_URI : Filter enabled`. Le navigateur assainit la page et signale la violation.</li></ul><p>Il s’agit d’une fonction Chrome utilisant les rapports de violation CSP pour envoyer des détails à un URI de votre choix. Les 2 dernières options sont considérées comme des valeurs sûres.</p>|

## <a name="aspnet-applications-must-disable-tracing-and-debugging-prior-to-deployment"></a><a id="trace-deploy"></a>Désactiver le traçage et le débogage dans les applications ASP.NET avant le déploiement

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Vue d’ensemble du débogage ASP.NET](/previous-versions/ms227556(v=vs.140)), [Vue d’ensemble du traçage ASP.NET](/previous-versions/bb386420(v=vs.140)), [Guide pratique pour activer le traçage d’une application ASP.NET](/previous-versions/0x5wc973(v=vs.140)), [Guide pratique pour activer le débogage pour les applications ASP.NET](https://msdn.microsoft.com/library/e8z01xdh(VS.80).aspx) |
| **Étapes** | Lorsque le traçage est activé pour la page, chaque navigateur demandant la page obtient également les informations de traçage qui contiennent des données sur l’état et le workflow du serveur interne. Ces informations peuvent être liées à la sécurité. Lorsque le débogage est activé pour la page, les erreurs qui surviennent sur le serveur entraînent la présentation de données de trace de la pile complètes au navigateur. Ces données peuvent exposer des informations liées à la sécurité concernant le workflow du serveur. |

## <a name="access-third-party-javascripts-from-trusted-sources-only"></a><a id="js-trusted"></a>Accéder aux scripts JavaScript tiers émanant uniquement de sources approuvées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Les scripts JavaScript tiers doivent uniquement être référencés à partir de sources approuvées. Les points de terminaison de référence doivent toujours se trouver sur TLS. |

## <a name="ensure-that-authenticated-aspnet-pages-incorporate-ui-redressing-or-click-jacking-defenses"></a><a id="ui-defenses"></a>S’assurer que les pages ASP.NET authentifiées incorporent des techniques de défense contre les attaques par redirection d’interface utilisateur ou détournement de clics

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [OWASP Clickjacking Defense Cheat Sheet (Aide-mémoire OWASP sur les techniques de défense contre le détournement de clics)](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html), [IE Internals - Combating ClickJacking With X-Frame-Options (IEInternals - Lutter contre le détournement de clics avec X-Frame-Options)](/archive/blogs/ieinternals/combating-clickjacking-with-x-frame-options) |
| **Étapes** | <p>Le détournement de clics, également appelé « attaque par redirection d’interface utilisateur », se produit lorsqu’un attaquant utilise plusieurs calques transparents ou opaques pour piéger un utilisateur en incitant ce dernier à cliquer sur un bouton ou sur un lien figurant sur une autre page que celle sur laquelle l’utilisateur pensait cliquer.</p><p>Cette superposition de calques s’effectue par l’élaboration d’une page malveillante intégrant un cadre IFrame caché qui charge la page de l’utilisateur piégé. De cette façon, l’attaquant « détourne » les clics destinés à cette page en les redirigeant vers une autre page, appartenant généralement à une autre application et/ou à un autre domaine. Pour prévenir les attaques par détournement de clics, définissez les en-têtes de réponse HTTP X-Frame-Options appropriés, qui demandent au navigateur de ne pas autoriser le chargement dans des cadres à partir d’autres domaines</p>|

### <a name="example"></a>Exemple
L’en-tête X-FRAME-OPTIONS peut être défini par le biais de web.config IIS. Extrait de code du fichier web.config pour les sites qui ne doivent jamais être chargés dans un cadre : 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="DENY"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

### <a name="example"></a>Exemple
Code du fichier web.config pour les sites qui ne doivent être chargés dans un cadre que par des pages appartenant au même domaine : 
```csharp
    <system.webServer>
        <httpProtocol>
            <customHeader>
                <add name="X-FRAME-OPTIONS" value="SAMEORIGIN"/>
            </customHeaders>
        </httpProtocol>
    </system.webServer>
```

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-aspnet-web-applications"></a><a id="cors-aspnet"></a>S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur les applications web ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms, MVC5 |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>La sécurité des navigateurs empêche une page web d’adresser des demandes AJAX à un autre domaine. Cette restriction est appelée stratégie de même origine et empêche un site malveillant de lire des données sensibles à partir d’un autre site. Toutefois, il est parfois nécessaire d’exposer en toute sécurité des API que d’autres sites peuvent consommer. CORS (Cross Origin Resource Sharing, partage des ressources multi-origines) est une norme W3C qui permet à un serveur d’assouplir la stratégie de même origine. Grâce au mécanisme CORS, un serveur peut autoriser explicitement certaines demandes multi-origines tout en en refusant d’autres.</p><p>CORS est plus sûr et plus flexible que les techniques précédentes telles que JSONP. L’activation de CORS se traduit essentiellement par l’ajout d’un petit nombre d’en-têtes de réponse HTTP (Access-Control-*) à l’application web, cette opération pouvant être effectuée de deux manières.</p>|

### <a name="example"></a>Exemple
Si le fichier Web.config est accessible, CORS peut être ajouté par le biais du code suivant : 
```XML
<system.webServer>
    <httpProtocol>
      <customHeaders>
        <clear />
        <add name="Access-Control-Allow-Origin" value="https://example.com" />
      </customHeaders>
    </httpProtocol>
```

### <a name="example"></a>Exemple
Si le fichier web.config n’est pas accessible, CORS est configurable par l’ajout du code CSharp suivant : 
```csharp
HttpContext.Response.AppendHeader("Access-Control-Allow-Origin", "https://example.com")
```

Notez qu’il est indispensable de s’assurer que la liste d’origines dans l’attribut « Access-Control-Allow-Origin » est définie sur un ensemble d’origines fini et approuvé. Une configuration inappropriée de cet attribut (par exemple, la définition de la valeur "*") autorisera les sites malveillants à adresser des demandes multi-origines à l’application web sans aucune restriction, exposant ainsi l’application à des risques d’attaques par falsification de requête intersites (CSRF, Cross Site Request Forgery). 

## <a name="enable-validaterequest-attribute-on-aspnet-pages"></a><a id="validate-aspnet"></a>Activer l’attribut ValidateRequest sur les pages ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Web Forms, MVC5 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Request Validation - Preventing Script Attacks (Validation des demandes - Prévention des attaques par script)](https://www.asp.net/whitepapers/request-validation) |
| **Étapes** | <p>La fonctionnalité de validation des demandes, disponible dans ASP.NET depuis la version 1.1, empêche le serveur d’accepter les contenus intégrant du code HTML non encodé. Cette fonctionnalité est destinée à éviter certaines attaques par injection de script dans lesquelles un code de script client ou HTML peut être, à l’insu de tous, envoyé à un serveur, stocké, puis présenté à d’autres utilisateurs. Nous vous recommandons vivement de valider toutes les données d’entrée et de les encoder en HTML s’il y a lieu.</p><p>La validation des demandes consiste à comparer toutes les données d’entrée à une liste de valeurs potentiellement dangereuses. Si une correspondance est trouvée, ASP.NET lève une exception `HttpRequestValidationException`. La fonctionnalité de validation des demandes est activée par défaut.</p>|

### <a name="example"></a>Exemple
Toutefois, cette fonctionnalité peut être désactivée au niveau de la page : 
```XML
<%@ Page validateRequest="false" %> 
```
ou au niveau de l’application : 
```XML
<configuration>
   <system.web>
      <pages validateRequest="false" />
   </system.web>
</configuration>
```
Notez que la fonctionnalité de validation des demandes n’est pas prise en charge et ne fait pas partie intégrante du pipeline MVC6. 

## <a name="use-locally-hosted-latest-versions-of-javascript-libraries"></a><a id="local-js"></a>Utiliser les dernières versions des bibliothèques JavaScript hébergées localement

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Les développeurs ayant recours aux bibliothèques JavaScript standard telles que JQuery doivent utiliser des versions approuvées des bibliothèques JavaScript courantes qui ne présentent aucun défaut de sécurité connu. Une bonne pratique consiste à utiliser la toute dernière version de ces bibliothèques, car ces versions intègrent des correctifs de sécurité pour les vulnérabilités répertoriées dans les versions antérieures.</p><p>Si la version la plus récente n’est pas utilisable pour des raisons de compatibilité, il convient de recourir aux versions minimales ci-dessous.</p><p>Versions minimales acceptables :</p><ul><li>**JQuery**<ul><li>JQuery 1.7.1</li><li>JQueryUI 1.10.0</li><li>JQuery Validate 1.9</li><li>JQuery Mobile 1.0.1</li><li>JQuery Cycle 2.99</li><li>JQuery DataTables 1.9.0</li></ul></li><li>**Ajax Control Toolkit**<ul><li>Ajax Control Toolkit 40412</li></ul></li><li>**ASP.NET Web Forms et Ajax**<ul><li>ASP.NET Web Forms et Ajax 4</li><li>ASP.NET Ajax 3.5</li></ul></li><li>**ASP.NET MVC**<ul><li>ASP.NET MVC 3.0</li></ul></li></ul><p>Ne chargez jamais une bibliothèque JavaScript à partir de sites externes tels que des réseaux de distribution de contenu (CDN) publics.</p>|

## <a name="disable-automatic-mime-sniffing"></a><a id="mime-sniff"></a>Désactiver la détection MIME automatique

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IE8 Security Part V: Comprehensive Protection (Sécurité IE8 Partie V - Protection complète)](/archive/blogs/ie/ie8-security-part-v-comprehensive-protection), [MIME type (Type MIME)](https://en.wikipedia.org/wiki/Mime_type) |
| **Étapes** | L’en-tête X-Content-Type-Options est un en-tête HTTP permettant aux développeurs de spécifier que leur contenu ne doit pas être détecté par MIME. Cet en-tête est conçu pour limiter les attaques par détection MIME. Pour chaque page susceptible de comporter du contenu contrôlable par l’utilisateur, vous devez utiliser l’en-tête HTTP X-Content-Type-Options:nosniff. Pour activer l’en-tête requis sur toutes les pages de l’application, vous pouvez effectuer l’une des opérations suivantes :|

### <a name="example"></a>Exemple
Ajoutez l’en-tête dans le fichier web.config si l’application est hébergée par Internet Information Services (IIS) 7 ou ses versions ultérieures. 
```XML
<system.webServer>
<httpProtocol>
<customHeaders>
<add name="X-Content-Type-Options" value="nosniff"/>
</customHeaders>
</httpProtocol>
</system.webServer>
```

### <a name="example"></a>Exemple
Ajoutez l’en-tête par le biais de la méthode Global.Application\_BeginRequest 
```csharp
void Application_BeginRequest(object sender, EventArgs e)
{
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
}
```

### <a name="example"></a>Exemple
Implémentez le module HTTP personnalisé 
```csharp
public class XContentTypeOptionsModule : IHttpModule
{
#region IHttpModule Members
public void Dispose()
{
}
public void Init(HttpApplication context)
{
context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders);
}
#endregion
void context_PreSendRequestHeaders(object sender, EventArgs e)
{
HttpApplication application = sender as HttpApplication;
if (application == null)
  return;
if (application.Response.Headers["X-Content-Type-Options "] != null)
  return;
application.Response.Headers.Add("X-Content-Type-Options ", "nosniff");
}
}
```

### <a name="example"></a>Exemple
Vous pouvez activer l’en-tête requis uniquement pour des pages spécifiques en l’ajoutant à des réponses individuelles : 

```csharp
this.Response.Headers["X-Content-Type-Options"] = "nosniff";
```

## <a name="remove-standard-server-headers-on-windows-azure-web-sites-to-avoid-fingerprinting"></a><a id="standard-finger"></a>Supprimer les en-têtes de serveur standard de l’offre Sites Web Microsoft Azure pour éviter la création d’une empreinte numérique

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | EnvironmentType - Azure |
| **Informations de référence**              | [Removing standard server headers on Windows Azure Web Sites (Supprimer les en-têtes de serveur standard de l’offre Sites Web Microsoft Azure)](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/) |
| **Étapes** | Les en-têtes tels que Server, X-Powered-By et X-AspNet-Version affichent des informations relatives au serveur et aux technologies sous-jacentes. Il est donc recommandé de supprimer ces en-têtes afin d’empêcher la création d’une empreinte numérique de l’application |

## <a name="configure-a-windows-firewall-for-database-engine-access"></a><a id="firewall-db"></a>Configurer un pare-feu Windows pour accéder au moteur de base de données

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | SQL Azure, OnPrem |
| **Attributs**              | N/A, Version SQL - V12 |
| **Informations de référence**              | [Vue d’ensemble des règles de pare-feu Azure SQL Database](../../azure-sql/database/firewall-configure.md), [Configurer un pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access) |
| **Étapes** | Les systèmes de pare-feu empêchent les accès non autorisés aux ressources de l'ordinateur. Pour accéder à une instance du moteur de base de données SQL Server de l’autre côté d’un pare-feu, vous devez configurer le pare-feu sur l’ordinateur exécutant SQL Server de façon à autoriser cet accès. |

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-aspnet-web-api"></a><a id="cors-api"></a>S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur API Web ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC 5 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Enabling Cross-Origin Requests in ASP.NET Web API 2 (Activation des demandes multi-origines dans API Web ASP.NET 2)](https://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api), [API Web ASP.NET - Prise en charge de CORS dans l’API Web ASP.NET 2](/archive/msdn-magazine/2013/december/asp-net-web-api-cors-support-in-asp-net-web-api-2) |
| **Étapes** | <p>La sécurité des navigateurs empêche une page web d’adresser des demandes AJAX à un autre domaine. Cette restriction est appelée stratégie de même origine et empêche un site malveillant de lire des données sensibles à partir d’un autre site. Toutefois, il est parfois nécessaire d’exposer en toute sécurité des API que d’autres sites peuvent consommer. CORS (Cross Origin Resource Sharing, partage des ressources multi-origines) est une norme W3C qui permet à un serveur d’assouplir la stratégie de même origine.</p><p>Grâce au mécanisme CORS, un serveur peut autoriser explicitement certaines demandes multi-origines tout en en refusant d’autres. CORS est plus sûr et plus flexible que les techniques précédentes telles que JSONP.</p>|

### <a name="example"></a>Exemple
Dans le fichier App_Start/WebApiConfig.cs, ajoutez le code suivant à la méthode WebApiConfig.Register : 
```csharp
using System.Web.Http;
namespace WebService
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // New code
            config.EnableCors();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
}
```

### <a name="example"></a>Exemple
L’attribut EnableCors peut être appliqué aux méthodes d’action dans un contrôleur comme suit : 

```csharp
public class ResourcesController : ApiController
{
  [EnableCors("http://localhost:55912", // Origin
              null,                     // Request headers
              "GET",                    // HTTP methods
              "bar",                    // Response headers
              SupportsCredentials=true  // Allow credentials
  )]
  public HttpResponseMessage Get(int id)
  {
    var resp = Request.CreateResponse(HttpStatusCode.NoContent);
    resp.Headers.Add("bar", "a bar value");
    return resp;
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "PUT",                          // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  [EnableCors("http://localhost:55912",       // Origin
              "Accept, Origin, Content-Type", // Request headers
              "POST",                         // HTTP methods
              PreflightMaxAge=600             // Preflight cache duration
  )]
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
}
```

Notez qu’il est indispensable de s’assurer que la liste d’origines dans l’attribut EnableCors est définie sur un ensemble d’origines fini et approuvé. Une configuration inappropriée de cet attribut (par exemple, la définition de la valeur "*") autorisera les sites malveillants à adresser des demandes multi-origines à l’API sans aucune restriction, exposant ainsi l’API à des risques d’attaques par falsification de requête intersites. EnableCors peut être décoré au niveau du contrôleur. 

### <a name="example"></a>Exemple
Pour désactiver le mécanisme CORS sur une méthode spécifique d’une classe, il est possible d’utiliser l’attribut DisableCors comme suit : 
```csharp
[EnableCors("https://example.com", "Accept, Origin, Content-Type", "POST")]
public class ResourcesController : ApiController
{
  public HttpResponseMessage Put(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  public HttpResponseMessage Post(Resource data)
  {
    return Request.CreateResponse(HttpStatusCode.OK, data);
  }
  // CORS not allowed because of the [DisableCors] attribute
  [DisableCors]
  public HttpResponseMessage Delete(int id)
  {
    return Request.CreateResponse(HttpStatusCode.NoContent);
  }
}
```

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC 6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Enabling Cross-Origin Requests (CORS) in ASP.NET Core 1.0 (Activation des demandes multi-origines (CORS) dans ASP.NET Core 1.0)](https://docs.asp.net/en/latest/security/cors.html) |
| **Étapes** | <p>Dans ASP.NET Core 1.0, CORS peut être activé au moyen d’un intergiciel (middleware) ou de MVC. Lorsque vous utilisez MVC pour activer CORS, les mêmes services CORS sont utilisés, mais l’intergiciel (middleware) CORS ne l’est pas.</p>|

**Approche 1** Activation de CORS avec un intergiciel (middleware) : pour activer CORS pour la totalité de l’application, ajoutez l’intergiciel (middleware) CORS au pipeline de requête à l’aide de la méthode d’extension UseCors. Vous pouvez spécifier une stratégie multi-origines lors de l’ajout de l’intergiciel (middleware) CORS à l’aide de la classe CorsPolicyBuilder. Il existe deux façons d'effectuer cette opération :

### <a name="example"></a>Exemple
La première façon consiste à appeler UseCors avec une expression lambda. Cette expression lambda utilise un objet CorsPolicyBuilder : 
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseCors(builder =>
        builder.WithOrigins("https://example.com")
        .WithMethods("GET", "POST", "HEAD")
        .WithHeaders("accept", "content-type", "origin", "x-custom-header"));
}
```

### <a name="example"></a>Exemple
La seconde façon consiste à définir une ou plusieurs stratégies CORS nommées, puis à sélectionner la stratégie par son nom au moment de l’exécution. 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin",
            builder => builder.WithOrigins("https://example.com"));
    });
}
public void Configure(IApplicationBuilder app)
{
    app.UseCors("AllowSpecificOrigin");
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Hello World!");
    });
}
```

**Approche 2** Activation de CORS dans MVC : les développeurs peuvent également utiliser MVC pour appliquer une stratégie CORS spécifique par action, par contrôleur ou de façon globale pour tous les contrôleurs.

### <a name="example"></a>Exemple
Par action : pour spécifier une stratégie CORS pour une action spécifique, ajoutez l’attribut [EnableCors] à l’action. Spécifiez le nom de la stratégie. 
```csharp
public class HomeController : Controller
{
    [EnableCors("AllowSpecificOrigin")] 
    public IActionResult Index()
    {
        return View();
    }
```

### <a name="example"></a>Exemple
Par contrôleur : 
```csharp
[EnableCors("AllowSpecificOrigin")]
public class HomeController : Controller
{
```

### <a name="example"></a>Exemple
Au niveau global : 
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.Configure<MvcOptions>(options =>
    {
        options.Filters.Add(new CorsAuthorizationFilterFactory("AllowSpecificOrigin"));
    });
}
```
Notez qu’il est indispensable de s’assurer que la liste d’origines dans l’attribut EnableCors est définie sur un ensemble d’origines fini et approuvé. Une configuration inappropriée de cet attribut (par exemple, la définition de la valeur "*") autorisera les sites malveillants à adresser des demandes multi-origines à l’API sans aucune restriction, exposant ainsi l’API à des risques d’attaques par falsification de requête intersites. 

### <a name="example"></a>Exemple
Pour désactiver une stratégie CORS pour un contrôleur ou une action, utilisez l’attribut [DisableCors]. 
```csharp
[DisableCors]
    public IActionResult About()
    {
        return View();
    }
```

## <a name="encrypt-sections-of-web-apis-configuration-files-that-contain-sensitive-data"></a><a id="config-sensitive"></a>Chiffrer les sections des fichiers de configuration de l’API Web qui contiennent des données sensibles

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Guide pratique pour chiffrer des sections de configuration dans ASP.NET 2.0 à l’aide de DPAPI](/previous-versions/msp-n-p/ff647398(v=pandp.10)), [Spécification d’un fournisseur de configuration protégée](/previous-versions/68ze1hb2(v=vs.140)), [Utilisation d’Azure Key Vault pour protéger la confidentialité des secrets d’application](/azure/architecture/multitenant-identity/web-api) |
| **Étapes** | Les fichiers de configuration tels que web.config et appsettings.json sont souvent utilisés pour stocker des informations sensibles, comme les noms d’utilisateur, les mots de passe, les chaînes de connexion de base de données et les clés de chiffrement. Si vous ne protégez pas ces informations, votre application est vulnérable aux attaquants ou aux personnes malveillantes qui veulent obtenir des informations sensibles, comme les noms d’utilisateur et les mots de passe de comptes, les noms de bases de données et les noms de serveurs. Selon le type de déploiement (azure/local), chiffrez les sections sensibles des fichiers de configuration à l’aide de DPAPI ou de services tels qu’Azure Key Vault. |

## <a name="ensure-that-all-admin-interfaces-are-secured-with-strong-credentials"></a><a id="admin-strong"></a>S’assurer que toutes les interfaces d’administration sont sécurisées avec des informations d’identification fortes

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Toutes les interfaces d’administration exposées par la passerelle d’appareil ou de champ doivent être sécurisées à l’aide d’informations d’identification fortes. Il en est de même pour toutes les autres interfaces exposées telles que WiFi, SSH, Partages de fichiers et FTP. Les mots de passe faibles par défaut ne doivent pas être utilisés. |

## <a name="ensure-that-unknown-code-cannot-execute-on-devices"></a><a id="unknown-exe"></a>S’assurer que le code inconnu ne peut pas s’exécuter sur les appareils

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Enabling Secure Boot and BitLocker Device Encryption on Windows 10 IoT Core (Activation du démarrage sécurisé et du chiffrement d’appareil BitLocker sur Windows 10 IoT Standard)](/windows/iot-core/secure-your-device/securebootandbitlocker) |
| **Étapes** | Le démarrage sécurisé UEFI contraint le système à autoriser uniquement l’exécution de fichiers binaires signés par une autorité spécifiée. Cette fonctionnalité empêche un code inconnu de s’exécuter sur la plateforme et d’affaiblir potentiellement la sécurisation de cette dernière. Activez le démarrage sécurisé UEFI et restreignez la liste des autorités de certification aux autorités approuvées pour la signature du code. Signez l’ensemble du code déployé sur l’appareil à l’aide de l’une des autorités de confiance. |

## <a name="encrypt-os-and-additional-partitions-of-iot-device-with-bit-locker"></a><a id="partition-iot"></a>Chiffrer les partitions du système d’exploitation et les partitions supplémentaires de l’appareil IoT avec BitLocker

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Windows 10 IoT Standard implémente une version allégée de BitLocker Device Encryption, qui dépend fortement de la présence d’un module de plateforme sécurisée (TPM) sur la plateforme, et notamment du protocole preOS nécessaire dans UEFI qui prend les mesures requises. Ces mesures preOS vérifient que le système d’exploitation possède par la suite un enregistrement définitif de son mode de lancement. Chiffrez également les partitions du système d’exploitation à l’aide de Bitlocker, ainsi que toutes les partitions supplémentaires susceptibles de stocker des données sensibles. |

## <a name="ensure-that-only-the-minimum-servicesfeatures-are-enabled-on-devices"></a><a id="min-enable"></a>S’assurer que seuls les services/fonctionnalités minimaux sont activés sur les appareils

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Appareil IoT | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Évitez d’activer ou désactivez les fonctionnalités ou services du système d’exploitation qui ne sont pas requis pour le fonctionnement de la solution. Par exemple, si l’appareil ne nécessite pas le déploiement d’une interface utilisateur, installez Windows IoT Standard sans périphérique de contrôle. |

## <a name="encrypt-os-and-additional-partitions-of-iot-field-gateway-with-bit-locker"></a><a id="field-bit-locker"></a>Chiffrer les partitions du système d’exploitation et les partitions supplémentaires de la passerelle de champ IoT avec Bitlocker

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle de champ IoT | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Windows 10 IoT Standard implémente une version allégée de BitLocker Device Encryption, qui dépend fortement de la présence d’un module de plateforme sécurisée (TPM) sur la plateforme, et notamment du protocole preOS nécessaire dans UEFI qui prend les mesures requises. Ces mesures preOS vérifient que le système d’exploitation possède par la suite un enregistrement définitif de son mode de lancement. Chiffrez également les partitions du système d’exploitation à l’aide de Bitlocker, ainsi que toutes les partitions supplémentaires susceptibles de stocker des données sensibles. |

## <a name="ensure-that-the-default-login-credentials-of-the-field-gateway-are-changed-during-installation"></a><a id="default-change"></a>S’assurer que les informations d’identification de connexion par défaut de la passerelle de champ sont modifiées lors de l’installation

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle de champ IoT | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Assurez-vous que les informations d’identification de connexion par défaut de la passerelle de champ sont modifiées lors de l’installation. |

## <a name="ensure-that-the-cloud-gateway-implements-a-process-to-keep-the-connected-devices-firmware-up-to-date"></a><a id="cloud-firmware"></a>S’assurer que la passerelle cloud implémente un processus garantissant la mise à jour continue du micrologiciel des appareils connectés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Passerelle cloud IoT | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | Choix de passerelle - Azure IoT Hub |
| **Informations de référence**              | [Vue d’ensemble de la gestion des appareils avec IoT Hub](../../iot-hub/iot-hub-device-management-overview.md), [How to update Device Firmware (Mise à jour du micrologiciel des appareils)](../../iot-hub/tutorial-firmware-update.md) |
| **Étapes** | LWM2M est un protocole défini par l’organisme Open Mobile Alliance pour la gestion des appareils IoT. La gestion des appareils IoT Azure permet d’interagir avec les appareils physiques à l’aide de travaux d’appareil. Assurez-vous que la passerelle cloud implémente un processus pour garantir la mise à jour continue des données de l’appareil et d’autres données de configuration à l’aide de la gestion des appareils Azure IoT Hub. |

## <a name="ensure-that-devices-have-end-point-security-controls-configured-as-per-organizational-policies"></a><a id="controls-policies"></a>S’assurer que les appareils disposent de contrôles de sécurité des points de terminaison configurés conformément aux directives organisationnelles

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Délimitation d’approbation machine | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | Assurez-vous que les appareils disposent de contrôles de sécurité pour les points de terminaison, tels que l’outil BitLocker pour le chiffrement au niveau disque, un antivirus avec des signatures mises à jour, un pare-feu basé sur l’hôte, des mises à niveau du système d’exploitation, des stratégies de groupe, etc., qui sont configurés conformément aux stratégies de sécurité organisationnelles. |

## <a name="ensure-secure-management-of-azure-storage-access-keys"></a><a id="secure-keys"></a>Garantir une gestion sécurisée des clés d’accès de stockage Azure

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Guide de sécurité de Stockage Azure - Gestion des clés de compte de stockage](../../storage/blobs/security-recommendations.md#identity-and-access-management) |
| **Étapes** | <p>Stockage des clés : il est recommandé de stocker les clés d’accès de Stockage Azure dans Azure Key Vault sous la forme d’un secret et de demander aux applications de récupérer la clé à partir du coffre de clés. Cette approche est recommandée pour les raisons suivantes :</p><ul><li>L’application ne disposera jamais de la clé de stockage codée en dur dans un fichier de configuration, ce qui rend impossible l’accès aux clés par une personne sans autorisation spécifique.</li><li>L’accès aux clés peut être contrôlé à l’aide d’Azure Active Directory. Cela signifie que le propriétaire d’un compte peut accorder l’accès aux seules applications qui doivent récupérer les clés à partir d’Azure Key Vault. Les autres applications ne pourront pas accéder aux clés sans avoir bénéficié d’une autorisation en particulier.</li><li>Regénération des clés : il est recommandé de mettre en place un processus de regénération des clés d’accès de stockage Azure pour des raisons de sécurité. Des informations détaillées sur les motifs et la procédure de planification d’une régénération des clés sont fournies dans l’article de référence du Guide de sécurité du service Stockage Azure.</li></ul>|

## <a name="ensure-that-only-trusted-origins-are-allowed-if-cors-is-enabled-on-azure-storage"></a><a id="cors-storage"></a>S’assurer que seules les origines approuvées sont autorisées si le mécanisme CORS est activé sur le stockage Azure

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Stockage Azure | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [CORS Support for the Azure Storage Services (Prise en charge du mécanisme Partage des ressources multi-origines (CORS) pour les services Stockage Azure)](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services) |
| **Étapes** | Azure Storage vous permet d’activer le Partage des ressources cross-origin (CORS, Cross Origin Resource Sharing). Pour chaque compte de stockage, vous pouvez spécifier les domaines qui peuvent accéder aux ressources de ce compte de stockage. Par défaut, CORS est désactivé sur tous les services. Vous pouvez activer CORS à l’aide de l’API REST ou de la bibliothèque cliente de stockage afin d’appeler l’une des méthodes pour définir les stratégies de service. |

## <a name="enable-wcfs-service-throttling-feature"></a><a id="throttling"></a>Activer la fonctionnalité de limitation de service WCF

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | .NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com) |
| **Étapes** | <p>L’absence de limitation de l’utilisation des ressources système peut entraîner un épuisement des ressources et, à terme, un déni de service.</p><ul><li>**EXPLICATION :** Windows Communication Foundation (WCF) offre la possibilité de limiter les requêtes de service. Le fait d’autoriser un nombre excessif de demandes de client est susceptible de saturer un système et d’en épuiser les ressources. En revanche, une restriction trop importante du nombre de demandes pouvant être adressées à un service risque d’empêcher des utilisateurs légitimes de recourir à ce service. Il convient donc de régler et de configurer chaque service individuellement afin d’autoriser la quantité de ressources appropriée.</li><li>**RECOMMANDATIONS** Activez la fonctionnalité de limitation de service de WCF et définissez les limites appropriées pour votre application.</li></ul>|

### <a name="example"></a>Exemple
Voici un exemple de configuration dans lequel la limitation est activée :
```
<system.serviceModel> 
  <behaviors>
    <serviceBehaviors>
    <behavior name="Throttled">
    <serviceThrottling maxConcurrentCalls="[YOUR SERVICE VALUE]" maxConcurrentSessions="[YOUR SERVICE VALUE]" maxConcurrentInstances="[YOUR SERVICE VALUE]" /> 
  ...
</system.serviceModel> 
```

## <a name="wcf-information-disclosure-through-metadata"></a><a id="info-metadata"></a>Divulgation d’informations WCF par le biais des métadonnées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | .NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com) |
| **Étapes** | Les métadonnées peuvent aider des attaquants à se renseigner sur le système et à planifier un certain type d’attaque. Les services WCF peuvent être configurés pour exposer les métadonnées. Les métadonnées fournissent des descriptions détaillées des services et ne doivent pas être diffusées dans les environnements de production. Les propriétés `HttpGetEnabled` / `HttpsGetEnabled` de la classe ServiceMetaData définissent si un service exposera ou non les métadonnées. | 

### <a name="example"></a>Exemple
Le code ci-dessous demande à WCF de diffuser les métadonnées d’un service.
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior();
smb.HttpGetEnabled = true; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb); 
```
Ne diffusez pas les métadonnées de service dans un environnement de production. Pour ce faire, définissez les propriétés HttpGetEnabled / HttpsGetEnabled de la classe ServiceMetaData sur la valeur false. 

### <a name="example"></a>Exemple
Le code ci-dessous demande à WCF de ne pas diffuser les métadonnées d’un service. 
```
ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
smb.HttpGetEnabled = false; 
smb.HttpGetUrl = new Uri(EndPointAddress); 
Host.Description.Behaviors.Add(smb);
```