---
title: Gestion des exceptions - Outil Microsoft de modélisation des menaces - Azure | Microsoft Docs
description: En savoir plus sur la gestion des exceptions dans l’outil Threat Modeling Tool. Voir les informations sur l’atténuation et les exemples de code.
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
ms.custom: devx-track-csharp
ms.openlocfilehash: 41532e554623c47e9728c6ccab92d99500e42021
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517425"
---
# <a name="security-frame-exception-management--mitigations"></a>Infrastructure de sécurité : gestion des exceptions | Atténuation des risques 
| Produit/Service | Article |
| --------------- | ------- |
| **WCF** | <ul><li>[WCF - Ne pas inclure le nœud serviceDebug dans le fichier de configuration](#servicedebug)</li><li>[WCF - Ne pas inclure le nœud serviceMetadata dans le fichier de configuration](#servicemetadata)</li></ul> |
| **API Web** | <ul><li>[Vérifier que la gestion des exceptions appropriée est effectuée dans l’API web ASP.NET](#exception)</li></ul> |
| **Application Web** | <ul><li>[Ne pas exposer les détails de sécurité dans les messages d’erreur](#messages)</li><li>[Implémenter la page de gestion des erreurs par défaut](#default)</li><li>[Définir la méthode de déploiement sur Vente au détail dans IIS](#deployment)</li><li>[Les exceptions doivent échouer en toute sécurité](#fail)</li></ul> |

## <a name="wcf--do-not-include-servicedebug-node-in-configuration-file"></a><a id="servicedebug"></a>WCF - Ne pas inclure le nœud serviceDebug dans le fichier de configuration

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_debug_information) |
| **Étapes** | Des services Windows Communication Framework (WCF) peuvent être configurés pour exposer des informations de débogage. Les informations de débogage ne doivent pas être utilisées dans les environnements de production. La balise `<serviceDebug>` définit si la fonctionnalité d’informations de débogage est activée pour un service WCF. Si l’attribut includeExceptionDetailInFaults est défini sur true, les informations sur les exceptions de l’application sont renvoyées aux clients. Les personnes malveillantes peuvent exploiter les informations supplémentaires qu’elles obtiennent de la sortie de débogage pour lancer des attaques ciblées sur l’infrastructure, la base de données ou d’autres ressources utilisées par l’application. |

### <a name="example"></a>Exemple
Le fichier de configuration suivant inclut la balise `<serviceDebug>` : 
```
<configuration> 
<system.serviceModel> 
<behaviors> 
<serviceBehaviors> 
<behavior name=""MyServiceBehavior""> 
<serviceDebug includeExceptionDetailInFaults=""True"" httpHelpPageEnabled=""True""/> 
... 
```
Désactivez les informations de débogage dans le service. Ceci peut être effectué en supprimant la balise `<serviceDebug>` du fichier de configuration de votre application. 

## <a name="wcf--do-not-include-servicemetadata-node-in-configuration-file"></a><a id="servicemetadata"></a>WCF - Ne pas inclure le nœud serviceMetadata dans le fichier de configuration

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | Générique, NET Framework 3 |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff648500(v=pandp.10)), [Fortify Kingdom](https://vulncat.fortify.com/en/detail?id=desc.config.dotnet.wcf_misconfiguration_service_enumeration) |
| **Étapes** | L’exposition publique des informations sur un service peut fournir aux personnes malveillantes une piste précieuse sur la manière dont elles peuvent exploiter le service. La balise `<serviceMetadata>` active la fonctionnalité de publication de métadonnées. Les métadonnées de service peuvent contenir des informations sensibles qui ne doivent pas être accessibles publiquement. Au minimum, autorisez uniquement les utilisateurs approuvés à accéder aux métadonnées et vérifiez que les informations inutiles ne sont pas exposées. Mieux encore, désactivez complètement la possibilité de publier des métadonnées. Une configuration WCF sûre ne contiendra pas la balise `<serviceMetadata>`. |

## <a name="ensure-that-proper-exception-handling-is-done-in-aspnet-web-api"></a><a id="exception"></a>Vérifier que la gestion des exceptions correcte est effectuée dans l’API web ASP.NET

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC 5, MVC 6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Gestion des exceptions dans l’API web ASP.NET](https://www.asp.net/web-api/overview/error-handling/exception-handling), [Validation de modèle dans l’API web ASP.NET](https://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **Étapes** | Par défaut, la plupart des exceptions non interceptées dans l’API web ASP.NET sont converties en réponse HTTP avec le code d’état`500, Internal Server Error`|

### <a name="example"></a>Exemple
Pour contrôler le code d’état renvoyé par l’API, `HttpResponseException` peut être utilisé comme indiqué ci-dessous : 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        throw new HttpResponseException(HttpStatusCode.NotFound);
    }
    return item;
}
```

### <a name="example"></a>Exemple
Pour contrôler davantage la réponse d’exception, la classe `HttpResponseMessage` peut être utilisée comme indiqué ci-dessous : 
```csharp
public Product GetProduct(int id)
{
    Product item = repository.Get(id);
    if (item == null)
    {
        var resp = new HttpResponseMessage(HttpStatusCode.NotFound)
        {
            Content = new StringContent(string.Format("No product with ID = {0}", id)),
            ReasonPhrase = "Product ID Not Found"
        }
        throw new HttpResponseException(resp);
    }
    return item;
}
```
Pour intercepter les exceptions non prises en charge qui ne sont pas du type `HttpResponseException`, des filtres d’exception peuvent être utilisés. Les filtres d’exception implémentent l’interface `System.Web.Http.Filters.IExceptionFilter`. La méthode la plus simple pour écrire un filtre d’exception consiste à le dériver de la classe `System.Web.Http.Filters.ExceptionFilterAttribute` et à remplacer la méthode OnException. 

### <a name="example"></a>Exemple
Voici un filtre qui convertit des `NotImplementedException` exceptions en code d’état HTTP `501, Not Implemented` : 
```csharp
namespace ProductStore.Filters
{
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Web.Http.Filters;

    public class NotImplExceptionFilterAttribute : ExceptionFilterAttribute 
    {
        public override void OnException(HttpActionExecutedContext context)
        {
            if (context.Exception is NotImplementedException)
            {
                context.Response = new HttpResponseMessage(HttpStatusCode.NotImplemented);
            }
        }
    }
}
```

Plusieurs méthodes pour enregistrer un filtre d’exception d’API web sont possibles :
- Par action
- Par contrôleur
- Globalement

### <a name="example"></a>Exemple
Pour appliquer le filtre à une action spécifique, ajoutez le filtre en tant qu’attribut à l’action : 
```csharp
public class ProductsController : ApiController
{
    [NotImplExceptionFilter]
    public Contact GetContact(int id)
    {
        throw new NotImplementedException("This method is not implemented");
    }
}
```
### <a name="example"></a>Exemple
Pour appliquer le filtre à toutes les actions sur un `controller`, ajoutez le filtre en tant qu’attribut à la classe `controller` : 

```csharp
[NotImplExceptionFilter]
public class ProductsController : ApiController
{
    // ...
}
```

### <a name="example"></a>Exemple
Pour appliquer le filtre globalement à tous les contrôleurs d’API web, ajoutez une instance du filtre à la collection `GlobalConfiguration.Configuration.Filters`. Les filtres d’exception de cette collection s’appliquent à n’importe quelle action de contrôleur d’API web. 
```csharp
GlobalConfiguration.Configuration.Filters.Add(
    new ProductStore.NotImplExceptionFilterAttribute());
```

### <a name="example"></a>Exemple
Pour valider un modèle, l’état du modèle peut être transmis à la méthode CreateErrorResponse comme indiqué ci-dessous : 
```csharp
public HttpResponseMessage PostProduct(Product item)
{
    if (!ModelState.IsValid)
    {
        return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
    }
    // Implementation not shown...
}
```

Vérifiez les liens dans la section Références pour plus d’informations sur la gestion exceptionnelle et la validation de modèle dans l’API web ASP.NET 

## <a name="do-not-expose-security-details-in-error-messages"></a><a id="messages"></a>Ne pas exposer les détails de sécurité dans les messages d’erreur

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Des messages d’erreur génériques sont fournis directement à l’utilisateur sans inclure de données d’application sensibles. Voici quelques exemples de données sensibles :</p><ul><li>Noms de serveur</li><li>Chaînes de connexion</li><li>Noms d’utilisateur</li><li>Mots de passe</li><li>Procédures SQL</li><li>Détail des échecs SQL dynamiques</li><li>Arborescence des appels de procédure et lignes de code</li><li>Variables stockées dans la mémoire</li><li>Emplacements de lecteur et de dossier</li><li>Points d’installation d’application</li><li>Paramètres de configuration d’hôte</li><li>Autres détails de l’application internes</li></ul><p>L’interception de toutes les erreurs dans une application, la fourniture de messages d’erreur génériques, ainsi que l’activation des messages d’erreur personnalisés dans IIS permettent d’éviter la divulgation d’informations. La base de données SQL Server et la gestion des exceptions .NET, entre autres architectures de gestion des erreurs, sont particulièrement détaillées et extrêmement utiles à un utilisateur malveillant profilant votre application. N’affichez pas directement le contenu d’une classe dérivée de la classe d’exceptions .NET et vérifiez que vous disposez de la gestion des exceptions appropriée afin qu’une exception inattendue ne soit pas présentée directement par inadvertance à l’utilisateur.</p><ul><li>Fournissez des messages d’erreur génériques directement à l’utilisateur qui résument les détails spécifiques trouvés directement dans le message d’exception/d’erreur</li><li>Ne présentez pas le contenu d’une classe d’exceptions .NET directement à l’utilisateur</li><li>Interceptez tous les messages d’erreur et, le cas échéant, informez l’utilisateur à l’aide d’un message d’erreur générique envoyé au client de l’application</li><li>N’exposez pas le contenu de la classe d’exceptions directement à l’utilisateur, et plus particulièrement la valeur renvoyée par `.ToString()`, ou les valeurs des propriétés Message ou StackTrace. Journalisez ces informations de manière sécurisée et présentez un message plus inoffensif à l’utilisateur</li></ul>|

## <a name="implement-default-error-handling-page"></a><a id="default"></a>Implémenter la page de gestion des erreurs par défaut

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Modifier la boîte de dialogue des paramètres des pages d’erreur ASP.NET](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd569096(v=ws.10)) |
| **Étapes** | <p>Lorsqu’une application ASP.NET échoue et provoque une erreur serveur interne HTTP/1.x 500, ou que la configuration d’une fonctionnalité (comme le filtrage des demandes) empêche l’affichage d’une page, un message d’erreur est généré. Les administrateurs peuvent choisir si l’application doit afficher ou non un message convivial pour le client, un message d’erreur détaillé au client ou un message d’erreur détaillé à l’hôte local uniquement. La balise `<customErrors>` dans le fichier web.config comporte trois modes :</p><ul><li>**On :** spécifie que les erreurs personnalisées sont activées. Si aucun attribut defaultRedirect n’est spécifié, les utilisateurs voient une erreur générique. Les erreurs personnalisées sont présentées aux clients distants et à l’hôte local</li><li>**Off :** spécifie que les erreurs personnalisées sont désactivées. Les erreurs ASP.NET détaillées sont présentées aux clients distants et à l’hôte local</li><li>**RemoteOnly :** spécifie que les erreurs personnalisées sont visibles uniquement pour les clients distants, et que les erreurs ASP.NET sont visibles pour l’hôte local. Il s'agit de la valeur par défaut</li></ul><p>Ouvrez le fichier `web.config` pour l’application/le site et vérifiez que la balise est définie sur `<customErrors mode="RemoteOnly" />` ou `<customErrors mode="On" />`.</p>|

## <a name="set-deployment-method-to-retail-in-iis"></a><a id="deployment"></a>Définir la méthode de déploiement sur Vente au détail dans IIS

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Déploiement |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Élément de déploiement (schéma des paramètres ASP.NET)](/previous-versions/dotnet/netframework-2.0/ms228298(v=vs.80)) |
| **Étapes** | <p>Le commutateur `<deployment retail>` est conçu pour être utilisé par les serveurs IIS de production. Ce commutateur est utilisé pour permettre aux applications de s’exécuter avec les meilleurs niveaux de performances et le moins de fuites d’informations de sécurité possibles en désactivant la capacité de l’application à générer la sortie de trace sur une page, en désactivant la capacité à afficher des messages d’erreur détaillés pour les utilisateurs finaux et en désactivant le commutateur de débogage.</p><p>Les commutateurs et les options destinés aux développeurs, comme le suivi et le débogage des échecs de demande, sont généralement activés pendant le développement actif. Il est recommandé de définir la méthode de déploiement de n’importe quel serveur de production sur Vente au détail. Ouvrez le fichier machine.config et vérifiez que `<deployment retail="true" />` est toujours défini sur true.</p>|

## <a name="exceptions-should-fail-safely"></a><a id="fail"></a>Les exceptions doivent échouer en toute sécurité

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Échec en toute sécurité](https://owasp.org/www-community/Fail_securely) |
| **Étapes** | L’application doit échouer en toute sécurité. Toute méthode renvoyant une valeur booléenne, en fonction de la décision prise, doit comporter un bloc d’exception créé avec soin. Il existe un grand nombre d’erreurs logiques dues à des problèmes de sécurité lorsque le bloc d’exception n’est pas écrit correctement.|

### <a name="example"></a>Exemple
```csharp
        public static bool ValidateDomain(string pathToValidate, Uri currentUrl)
        {
            try
            {
                if (!string.IsNullOrWhiteSpace(pathToValidate))
                {
                    var domain = RetrieveDomain(currentUrl);
                    var replyPath = new Uri(pathToValidate);
                    var replyDomain = RetrieveDomain(replyPath);

                    if (string.Compare(domain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                    {
                        //// Adding additional check to enable CMS urls if they are not hosted on same domain.
                        if (!string.IsNullOrWhiteSpace(Utilities.CmsBase))
                        {
                            var cmsDomain = RetrieveDomain(new Uri(Utilities.Base.Trim()));
                            if (string.Compare(cmDomain, replyDomain, StringComparison.OrdinalIgnoreCase) != 0)
                            {
                                return false;
                            }
                            else
                            {
                                return true;
                            }
                        }

                        return false;
                    }
                }

                return true;
            }
            catch (UriFormatException ex)
            {
                LogHelper.LogException("Utilities:ValidateDomain", ex);
                return true;
            }
        }
```
La méthode ci-dessus renvoie toujours la valeur True si une exception se produit. Si l’utilisateur final fournit une URL incorrecte acceptée par le navigateur mais pas par le constructeur `Uri()`, une exception est levée et la victime est dirigée vers l’URL valide mais mal formée.