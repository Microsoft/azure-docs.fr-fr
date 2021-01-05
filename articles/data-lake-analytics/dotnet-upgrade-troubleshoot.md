---
title: Guide pratique pour résoudre les défaillances des travaux U-SQL Azure Data Lake Analytics dues à la mise à niveau vers .NET Framework 4.7.2
description: Résolvez les défaillances des travaux U-SQL dues à la mise à niveau vers .NET 4.7.2.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: troubleshooting
ms.date: 10/11/2019
ms.openlocfilehash: ab03ea8a88187289f5dce55f8a396a9d51346a3f
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92217675"
---
# <a name="azure-data-lake-analytics-is-upgrading-to-the-net-framework-v472"></a>Azure Data Lake Analytics est mis à niveau vers .NET Framework v4.7.2

Le runtime par défaut d’Azure Data Lake Analytics est mis à niveau de .NET Framework v4.5.2 à .NET Framework v4.7.2. Cette modification introduit un léger risque de changement cassant si votre code U-SQL utilise des assemblys personnalisés et que ces derniers utilisent des bibliothèques .NET.

Cette mise à niveau de .NET Framework 4.5.2 vers la version 4.7.2 signifie que le .NET Framework déployé dans un runtime U-SQL (le runtime par défaut) sera désormais toujours 4.7.2. Il n’existe pas d’option côte à côte pour les versions de .NET Framework.

Une fois cette mise à niveau vers .NET Framework 4.7.2 effectuée, le code managé du système s’exécutera sous la version 4.7.2 et les bibliothèques fournies par l’utilisateur, comme les assemblys personnalisés U-SQL, s’exécuteront dans le mode de compatibilité descendante correspondant à la version pour laquelle l’assembly a été généré.

- Si vos DLL d’assembly sont générées pour la version 4.5.2, l’infrastructure déployée les traitera comme des bibliothèques 4.5.2, en fournissant (avec quelques exceptions) la sémantique 4.5.2.
- Vous pouvez maintenant utiliser des assemblys U-SQL personnalisés qui utilisent les fonctionnalités 4.7.2 de version, si vous ciblez le .NET Framework 4.7.2.

Il est possible que cette mise à niveau vers .NET Framework 4.7.2 introduise des changements cassants dans les travaux U-SQL qui utilisent des assemblys personnalisés .NET. Nous vous suggérons de vérifier les problèmes de compatibilité descendante à l’aide de la procédure ci-dessous.

## <a name="how-to-check-for-backwards-compatibility-issues"></a>Comment vérifier les problèmes de compatibilité descendante

Vérifiez la possibilité que des problèmes majeurs de compatibilité descendante se produisent en exécutant les vérifications de compatibilité .NET sur votre code .NET dans vos assemblys personnalisés U-SQL.

> [!Note]
> L’outil ne détecte pas de réels changements cassants. Il identifie uniquement les API .NET appelées qui peuvent provoquer des problèmes (pour certaines entrées). Si vous êtes informé d’un problème, votre code peut toujours être correct, mais vous devez vérifier davantage de détails.

1. Exécutez le vérificateur de compatibilité descendante sur vos DLL .NET en :
   1. Utilisant l’extension Visual Studio sur [Extension Visual Studio de l’analyseur de portabilité .NET](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer)
   1. Téléchargeant et utilisant l’outil autonome à partir de [GitHub dotnetapiport](https://github.com/microsoft/dotnet-apiport). Les instructions relatives à l’exécution de l’outil autonome se trouvent dans [Changements cassants de GitHub dotnetapiport](https://github.com/microsoft/dotnet-apiport/blob/dev/docs/HowTo/BreakingChanges.md)
   1. Pour la version 4.7.2. pour une compatibilité, `read isRetargeting == True` identifie les problèmes éventuels.
2. Si l’outil indique que votre code peut être affecté par l’une des incompatibilités descendantes possibles (des exemples courants d’incompatibilités sont répertoriés ci-dessous), vous pouvez effectuer des vérifications supplémentaires en :
   1. Analysant votre code et en identifiant si votre code transmet des valeurs aux API affectées
   1. Effectuant une vérification du runtime. Le déploiement du runtime n’est pas effectué côte à côte dans ADLA. Vous pouvez effectuer une vérification du runtime avant la mise à niveau, en utilisant l’exécution locale de VisualStudio avec un .NET Framework 4.7.2 local sur un jeu de données représentatif.
3. Si vous êtes effectivement affecté par une incompatibilité descendante, prenez les mesures nécessaires pour la résoudre (par exemple, en corrigeant vos données ou la logique de code).

Dans la plupart des cas, vous ne devez pas être affecté par l’incompatibilité descendante.

## <a name="timeline"></a>Chronologie

Vous pouvez vérifier le déploiement du nouveau runtime ici [Résolution des problèmes de runtime](runtime-troubleshoot.md) et en examinant tout travail réussi effectué antérieurement.

### <a name="what-if-i-cant-get-my-code-reviewed-in-time"></a>Que se passe-t-il si je ne parviens pas à faire réviser mon code à temps

Vous pouvez soumettre votre travail à l’ancienne version du runtime (qui est conçue pour cibler 4.5.2). Toutefois, en raison de l’absence de fonctionnalités côte à côte .NET Framework, elle ne s’exécutera toujours qu’en mode de compatibilité 4.5.2. Il se peut que vous rencontriez toujours certains des problèmes de compatibilité descendante en raison de ce comportement.

### <a name="what-are-the-most-common-backwards-compatibility-issues-you-may-encounter"></a>Quels sont les problèmes de compatibilité descendante les plus courants que vous pouvez rencontrer

Les incompatibilités descendantes les plus courantes et que l’outil de vérification est susceptible d’identifier sont (nous avons généré cette liste en exécutant l’outil de vérification sur nos propres travaux ADLA internes), quelles bibliothèques sont affectées (remarque : vous pouvez appeler les bibliothèques uniquement indirectement, il est donc important de prendre les mesures nécessaires #1 pour vérifier si vos travaux sont affectés) et les actions possibles pour y remédier. Remarque : Dans presque tous les cas pour nos propres travaux, les avertissements étaient des faux positifs en raison des natures étroites de la plupart des changements cassants.

- La propriété IAsyncResult.CompletedSynchronously doit être correcte pour que la tâche obtenue se termine
  - Lors de l’appel de TaskFactory.FromAsync, l’implémentation de la propriété IAsyncResult.CompletedSynchronously doit être correcte pour que la tâche obtenue se termine. Autrement dit, la propriété doit retourner la valeur true si, et uniquement si, l’implémentation s’est terminée de façon synchrone. Auparavant, la propriété n’était pas vérifiée.
  - Bibliothèques impactées : mscorlib, System.Threading.Tasks
  - Action suggérée : vérifiez que TaskFactory.FromAsync retourne la valeur true correctement

- DataObject.GetData récupère maintenant les données au format UTF-8
  - Pour les applications qui ciblent le .NET Framework 4 ou qui s’exécutent sur le .NET Framework 4.5.1 ou versions antérieures, DataObject.GetData récupère les données au format HTML sous forme de chaîne ASCII. Par conséquent, les caractères non-ASCII (caractères dont les codes ASCII sont supérieurs à 0x7F) sont représentés par deux caractères aléatoires.#N##N#Pour les applications qui ciblent le .NET Framework 4.5 ou version ultérieure et qui s’exécutent sur le .NET Framework 4.5.2, `DataObject.GetData` récupère les données formatées HTML au format UTF-8, qui permet la représentation correcte des caractères supérieurs à 0x7F.
  - Bibliothèques affectées : Glo
  - Action suggérée : s’assurer que les données récupérées ont le format souhaité

- XmlWriter lève des paires de substitution non valides
  - Pour les applications qui ciblent .NET Framework 4.5.2 ou les versions antérieures, l’écriture d’une paire de substitution non valide à l’aide de la gestion des exceptions de secours ne lève pas toujours une exception. Pour les applications qui ciblent .NET Framework 4.6, toute tentative d’écriture d’une paire de substitution non valide lève une `ArgumentException`.
  - Bibliothèques affectées : System.Xml, System.Xml.ReaderWriter
  - Action suggérée : vérifiez que vous n’écrivez pas une paire de substitution non valide qui provoquera l’exception d’argument

- HtmlTextWriter n’affiche pas correctement l’élément `<br/>`
  - À partir de .NET Framework 4.6, l’appel de `HtmlTextWriter.RenderBeginTag()` et de `HtmlTextWriter.RenderEndTag()` avec un élément `<BR />` insère correctement un seul `<BR />` (au lieu de deux)
  - Bibliothèques affectées : System.Web
  - Action suggérée : veillez à insérer la quantité de `<BR />` que vous souhaitez obtenir afin qu’aucun comportement aléatoire ne se produise dans le travail de production

- L’appel de CreateDefaultAuthorizationContext avec un argument Null a été modifié
  - L’implémentation du AuthorizationContext retourné par un appel au `CreateDefaultAuthorizationContext(IList<IAuthorizationPolicy>)` avec un argument authorizationPolicies Null a changé son implémentation dans .NET Framework 4.6.
  - Bibliothèques affectées : System.IdentityModel
  - Action suggérée : assurez-vous que vous gérez le nouveau comportement attendu lorsqu’il existe une stratégie d’autorisation Null
  
- L’algorithme RSACng charge désormais correctement les clés RSA de taille de clé non standard
  - Dans les versions .NET Framework antérieures à la version 4.6.2, les clients avec des tailles de clé non standard pour les certificats RSA ne peuvent pas accéder à ces clés via les méthodes d’extension `GetRSAPublicKey()` et `GetRSAPrivateKey()`. Une `CryptographicException` comportant le message « la taille de clé demandée n’est pas prise en charge » est générée. Ce problème a été résolu avec .NET Framework 4.6.2. De même, `RSA.ImportParameters()` et `RSACng.ImportParameters()` fonctionnent désormais avec des tailles de clé non standard sans générer de `CryptographicException`.
  - Bibliothèques impactées : mscorlib, System.Core
  - Action suggérée : vérifier que les clés RSA fonctionnent comme prévu

- Les vérifications des signes deux-points dans les chemins d’accès sont plus strictes
  - Dans .NET Framework 4.6.2, un certain nombre de modifications ont été apportées pour prendre en charge les chemins d’accès précédemment non pris en charge (à la fois en termes de longueur et de format). Les vérifications de bonne syntaxe de séparateur de lecteur (le signe deux-points) ont été rendues plus correctes, ce qui a eu pour effet secondaire de bloquer certains chemins d’URI dans quelques API de sélection de chemin d’accès, où ils étaient tolérés.
  - Bibliothèques impactées : mscorlib, System.Runtime.Extensions
  - Action suggérée :

- appels aux constructeurs ClaimsIdentity
  - À compter de .NET Framework 4.6.2, la façon dont les constructeurs `T:System.Security.Claims.ClaimsIdentity` avec un paramètre `T:System.Security.Principal.IIdentity` définissent la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` est modifiée. Si l’argument `T:System.Security.Principal.IIdentity` est un objet `T:System.Security.Claims.ClaimsIdentity` et que la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` de cet objet `T:System.Security.Claims.ClaimsIdentity` n’est pas `null`, la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` est attachée à l’aide de la méthode `M:System.Security.Claims.ClaimsIdentity.Clone`. Dans Framework 4.6.1 et versions antérieures, la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` est attachée en tant que référence existante. En raison de cette modification, à compter de .NET Framework 4.6.2, la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` du nouvel objet `T:System.Security.Claims.ClaimsIdentity` n’est pas égale à la propriété `P:System.Security.Claims.ClaimsIdentify.Actor` de l’argument `T:System.Security.Principal.IIdentity` du constructeur. Dans .NET Framework 4.6.1 et les versions antérieures, elle est égale.
  - Bibliothèques impactées : mscorlib
  - Action suggérée : vérifier que ClaimsIdentity fonctionne comme prévu sur le nouveau runtime

- La sérialisation des caractères de contrôle avec DataContractJsonSerializer est désormais compatible avec ECMAScript V6 et V8
  - Dans .NET Framework 4.6.2 et les versions antérieures, le DataContractJsonSerializer n’a pas sérialisé certains caractères de contrôle spéciaux, tels que \b, \f et \t, d’une manière compatible avec les normes ECMAScript V6 et V8. À partir de .NET Framework 4.7, la sérialisation de ces caractères de contrôle est compatible avec ECMAScript V6 et V8.
  - Bibliothèques affectées : System.Runtime.Serialization.Json
  - Action suggérée : garantir le même comportement avec DataContractJsonSerializer
