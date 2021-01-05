---
title: Résolution des échecs de chargement du SDK pour les applications web JavaScript – Azure Application Insights
description: Comment résoudre les échecs de chargement du SDK pour les applications web JavaScript
ms.topic: conceptual
author: MSNev
ms.author: newylie
ms.date: 06/05/2020
ms.custom: devx-track-js
ms.openlocfilehash: 30c7caef4143b1a7cdba959971ff7689f986cb9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91333254"
---
# <a name="troubleshooting-sdk-load-failure-for-javascript-web-apps"></a>Résolution des échecs de chargement du SDK pour les applications web JavaScript

<!-- 
Editor Note: This link name above "SDK Load Failure" has a direct references by https://go.microsoft.com/fwlink/?linkid=2128109 which is embedded in the snippet and from the JavaScript Page. If you change this text you MUST also update these references.
-->

L’exception d’échec du chargement du Kit de développement logiciel (SDK) est créée et signalée par l’extrait de code JavaScript (v3 ou version ultérieure) lorsqu’il détecte que le script du SDK n’a pas pu être téléchargé ni initialisé. De façon simpliste, le client (navigateur) de vos utilisateurs finaux n’a pas pu télécharger le Kit de développement logiciel (SDK) Application Insights ni l’initialiser à partir de la page d’hébergement identifiée et, par conséquent, aucun événement ni aucune télémétrie ne sera signalé.

![Vue d’ensemble de l’échec du navigateur Portail Azure](./media/javascript-sdk-load-failure/overview.png)

> [!NOTE]
> Cette exception est prise en charge sur tous les principaux navigateurs qui prennent en charge l’API fetch() ou XMLHttpRequest. Cela exclut IE 8 et ses versions antérieures, de sorte qu’ils ne signaleront pas ce type d’exception [sauf si votre environnement inclut un polyfill fetch()].

![détails de l’exception du navigateur](./media/javascript-sdk-load-failure/exception.png)

Les détails de la pile incluent les informations de base avec les URL utilisées par l’utilisateur final.

| Nom                      | Description                                                                                                  |
|---------------------------|--------------------------------------------------------------------------------------------------------------|
| &lt;CDN&nbsp;Endpoint&gt; | URL utilisée (et qui a échoué) pour télécharger le Kit de développement logiciel (SDK).                                                      |
| &lt;Help&nbsp;Link&gt;    | URL qui renvoie à la documentation de résolution des problèmes (la présente page).                                              |
| &lt;Host&nbsp;URL&gt;     | URL complète de la page utilisée par l’utilisateur final.                                                    |
| &lt;Endpoint&nbsp;URL&gt; | URL utilisée pour signaler l’exception. Cette valeur peut être utile pour déterminer si la page d’hébergement a été consultée à partir de l’Internet public ou d’un cloud privé.

Les raisons les plus courantes de cette exception sont les suivantes :

- Échec de connectivité réseau intermittente.
- Panne du réseau de distribution de contenu (CDN) Application Insights.
- Échec de l’initialisation du Kit de développement logiciel (SDK) après le chargement du script.
- Blocage du CDN JavaScript Application Insights.

Un [échec de connectivité réseau intermittente](#intermittent-network-connectivity-failure) est la raison la plus courante pour cette exception, en particulier dans les scénarios d’itinérance mobile où les utilisateurs perdent la connectivité réseau par intermittence.

Les sections suivantes décrivent comment résoudre les problèmes liés à chaque cause potentielle de cette erreur.

> [!NOTE]
> Plusieurs étapes de résolution des problèmes supposent que votre application contrôle directement la balise &lt;script /&gt; de l’extrait de code et que sa configuration est retournée dans le cadre de la page HTML d’hébergement. Si ce n’est pas le cas, les étapes identifiées ne s’appliquent pas à votre scénario.

## <a name="intermittent-network-connectivity-failure"></a>Échec de connectivité réseau intermittente

**Si l’utilisateur rencontre des échecs de connectivité réseau intermittente, les solutions possibles sont moins nombreuses et les échecs se résolvent généralement d’eux-mêmes en un bref laps de temps.** Par exemple, si l’utilisateur recharge votre site (actualise la page), les fichiers seront (éventuellement) téléchargés et mis en cache localement jusqu’à la publication d’une version mise à jour.

> [!NOTE]
> Si cette exception est persistante et qu’elle se produit chez un grand nombre de vos utilisateurs (diagnostiqués par un niveau rapide et soutenu de cette exception signalée), accompagnée d’une réduction de la télémétrie normale du client, les problèmes de connectivité réseau intermittente ne sont _probablement pas_ la véritable cause du problème et vous devez continuer à diagnostiquer les autres problèmes possibles connus.

Dans ce cas, il est peu probable que l’hébergement du Kit de développement logiciel (SDK) sur votre propre CDN fournisse ou réduise les occurrences de cette exception, car votre propre CDN sera concerné par le même problème.

Il en va de même lorsque vous utilisez le Kit de développement logiciel (SDK) via la solution de packages NPM. Toutefois, du point de vue des utilisateurs finaux, lorsque cela se produit, l’ensemble de votre application ne parvient pas à se charger/s’initialiser (plutôt que _seulement_ le Kit de développement logiciel (SDK) de télémétrie, qu’ils ne peuvent pas voir), de sorte qu’ils actualiseront très probablement votre site jusqu’à son chargement complet.

Vous pouvez également essayer d’utiliser des [packages NPM](#use-npm-packages-to-embed-the-application-insight-sdk) pour incorporer le Kit de développement logiciel (SDK) Application Insights.

Pour minimiser les échecs de connectivité réseau intermittente, nous avons implémenté des en-têtes Cache-Control sur tous les fichiers CDN afin que, une fois que le navigateur de l’utilisateur final a téléchargé la version actuelle du Kit de développement logiciel (SDK), il n’ait pas besoin de la télécharger à nouveau et que le navigateur réutilise la copie précédemment obtenue (voir [le fonctionnement de la mise en cache](../../cdn/cdn-how-caching-works.md)). Si la vérification de la mise en cache échoue ou qu’une nouvelle version a été publiée, le navigateur de l’utilisateur final doit télécharger la version mise à jour. Vous pouvez donc constater un certain niveau de _« bruit »_ en arrière-plan dans le scénario de vérification de l’échec ou un pic temporaire lorsqu’une nouvelle version est mise à la disposition générale (déployée sur le CDN).
 
## <a name="application-insights-cdn-outage"></a>Panne du CDN Application Insights

Vous pouvez vérifier s’il existe une panne du CDN Application Insights en tentant d’accéder au point de terminaison CDN directement à partir du navigateur (par exemple, https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js) à partir d’un emplacement différent de celui des utilisateurs finaux, probablement votre propre ordinateur de développement (en supposant que votre organisation n’a pas bloqué ce domaine).

Si vous confirmez une panne, vous pouvez [créer un ticket de support](https://azure.microsoft.com/support/create-ticket/) ou essayer de modifier l’URL utilisée pour télécharger le Kit de développement logiciel (SDK).

### <a name="change-the-cdn-endpoint"></a>Modifier le point de terminaison CDN
  
Étant donné que l’extrait de code et sa configuration sont retournés par votre application dans le cadre de chaque page générée, vous pouvez modifier la configuration `src` de l’extrait de code pour utiliser une URL différente pour le Kit de développement logiciel (SDK). À l’aide de cette approche, vous pouvez contourner le problème de blocage du CDN, car la nouvelle URL ne devrait pas être bloquée.

Points de terminaison CDN actuels du SDK JavaScript Application Insights
- `https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js`
- `https://js.monitor.azure.com/scripts/b/ai.2.min.js`

> [!NOTE]
> Le point de terminaison `https://js.monitor.azure.com/` est un alias qui nous permet de passer d’un fournisseur de CDN à l’autre en cinq minutes environ, sans que vous ayez besoin de modifier la moindre configuration. Cela nous permet de corriger les problèmes détectés liés au CDN plus rapidement si un fournisseur de CDN est confronté à des problèmes régionaux ou mondiaux, sans que tout le monde ait à ajuster ses paramètres.

## <a name="sdk-failed-to-initialize-after-loading-the-script"></a>Échec de l’initialisation du Kit de développement logiciel (SDK) après le chargement du script

En cas d’échec de l’initialisation du Kit de développement logiciel (SDK), le &lt;script /&gt; a été correctement téléchargé à partir du CDN, mais il échoue pendant l’initialisation. Cela peut être dû à des dépendances manquantes ou non valides ou à une forme d’exception JavaScript.

La première chose à vérifier est si le Kit de développement logiciel (SDK) a été téléchargé avec succès. Si le script n’a PAS été téléchargé, alors ce scénario n’est __pas__ la cause de l’exception d’échec du chargement du Kit de développement logiciel (SDK).

Vérification rapide : à l’aide d’un navigateur prenant en charge les outils de développement (F12), vérifiez sous l’onglet réseau que le script défini dans la configuration d’extrait de code ```src``` a été téléchargé avec un code de réponse 200 (réussite) ou 304 (non modifié). Vous pouvez également utiliser un outil tel que Fiddler pour examiner le trafic.

Les sections ci-dessous comprennent différentes options de création de rapports. Il est recommandé de créer un ticket de support ou de signaler un problème sur GitHub.

 Règles de base pour la création de rapports :

- Si le problème ne concerne qu’un petit nombre d’utilisateurs et une version spécifique ou un sous-ensemble de versions de navigateur (vérifiez les détails sur l’exception signalée), il s’agit probablement d’un problème uniquement lié à l’utilisateur final ou à l’environnement, ce qui nécessitera que votre application fournisse des implémentations `polyfill` supplémentaires. Pour cela, [signalez un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues).
- Si ce problème affecte l’ensemble de votre application et que tous vos utilisateurs sont concernés, il s’agit probablement d’un problème lié à la version et vous devez donc [créer un nouveau ticket de support](https://azure.microsoft.com/support/create-ticket/).

### <a name="javascript-exceptions"></a>Exceptions JavaScript

Commençons par vérifier les exceptions JavaScript, en utilisant un navigateur qui prend en charge les outils de développement (F12) ; chargez la page et vérifiez si des exceptions se sont produites.

Si des exceptions sont signalées dans le script du Kit de développement logiciel (SDK) [par exemple, ai.2.min.js], cela peut indiquer que la configuration transmise dans le SDK contient une configuration requise inattendue ou manquante ou qu’une version défectueuse a été déployée sur le CDN.

Pour vérifier si une configuration est défectueuse, modifiez la configuration transmise dans l’extrait de code (si ce n’est pas déjà fait) afin qu’elle inclue uniquement votre clé d’instrumentation comme valeur de chaîne.

> src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",<br />
> cfg:{<br />
> instrumentationKey: "INSTRUMENTATION_KEY"<br />
> }});<br />

Si, lors de l’utilisation de cette configuration minimale, vous constatez toujours une exception JavaScript dans le script du Kit de développement logiciel (SDK), [créez un ticket de support](https://azure.microsoft.com/support/create-ticket/). Le problème nécessitera la restauration de la build défectueuse, car il s’agit probablement du dysfonctionnement d’une version récemment déployée.

Si l’exception disparaît, le problème est probablement dû à une incompatibilité de type ou à une valeur inattendue. Commencez à rajouter vos options de configuration une par une et testez-les jusqu’à ce que l’exception se reproduise. Vérifiez ensuite la documentation de l’élément à l’origine du problème. Si la documentation n’est pas claire ou si vous avez besoin d’aide, [signalez un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues).

Si votre configuration était déjà déployé et opérationnelle et qu’elle a commencé à signaler cette exception, cela peut être un problème dû à une version récemment déployée. Vérifiez si cela concerne uniquement un petit ensemble d’utilisateurs/de navigateurs et [signalez un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues) ou [créez un ticket de support](https://azure.microsoft.com/support/create-ticket/).

### <a name="enable-console-debugging"></a>Activer le débogage par console

En supposant qu’il n’y ait aucune exception, l’étape suivante consiste à activer le débogage par console en ajoutant le paramètre `loggingLevelConsole` à la configuration, ce qui entraîne l’envoi de tous les avertissements et erreurs d’initialisation à la console du navigateur (normalement disponible via les outils de développement [F12]). Toutes les erreurs signalées doivent être explicites et, si vous avez besoin d’une aide supplémentaire, [signalez un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues).

> cfg:{<br />
> instrumentationKey: "INSTRUMENTATION_KEY",<br />
> loggingLevelConsole: 2<br />
> }});<br />

> [!NOTE]
> Pendant l’initialisation, le Kit de développement logiciel (SDK) effectue quelques vérifications de base pour les principales dépendances connues. Si ces dernières ne sont pas fournies par le runtime actuel, il signalera les échecs à la console sous la forme de messages d’avertissement, mais uniquement si la valeur de `loggingLevelConsole` est supérieure à zéro.

S’il ne parvient toujours pas à s’initialiser, essayez d’activer le paramètre de configuration ```enableDebug```. Toutes les erreurs internes seront alors considérées comme des exceptions (ce qui entraînera la perte de la télémétrie). Comme il s’agit d’un paramètre réservé aux développeurs, le nombre d’exceptions levées dans le cadre de certaines vérifications internes sera probablement très important. Vous devrez examiner chaque exception pour identifier le problème à l’origine de l’échec du Kit de développement logiciel (SDK). Utilisez la version non minifiée du script (notez que l’extension ci-dessous est « .js » et non « .min.js »), sans quoi les exceptions seront illisibles.

> [!WARNING]
> Ce paramètre est réservé aux développeurs et ne doit JAMAIS être activé dans un environnement de production complet, car vous perdriez la télémétrie.

> src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.js",<br />
> cfg:{<br />
> instrumentationKey: "INSTRUMENTATION_KEY",<br />
> enableDebug: true<br />
> }});<br />

Si cela n’apporte toujours rien, vous devez [signaler un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues) avec les détails et un exemple de site si vous en avez un. Incluez la version du navigateur, le système d’exploitation et les détails de l’infrastructure JS pour aider à identifier le problème.

## <a name="the-application-insights-javascript-cdn-has-been-blocked"></a>Blocage du CDN JavaScript Application Insights

Le blocage du CDN est possible lorsqu’un point de terminaison CDN du Kit de développement logiciel (SDK) JavaScript Application Insights a été signalé ou identifié comme étant dangereux. Lorsque cela se produit, le point de terminaison est inscrit sur une liste rouge publique et les consommateurs de ces listes commencent à bloquer tout accès.

Pour résoudre ce problème, le propriétaire du point de terminaison CDN doit travailler avec l’entité de la liste rouge qui a marqué le point de terminaison comme dangereux afin qu’il puisse être retiré de la liste concernée.

Vérifiez si le point de terminaison CDN a été identifié comme dangereux.
- [Rapport de transparence Google](https://transparencyreport.google.com/safe-browsing/search)
- [VirusTotal](https://www.virustotal.com/gui/home/url)
- [Sucuri SiteCheck](https://sitecheck.sucuri.net/)

Selon la fréquence à laquelle l’application, le pare-feu ou l’environnement mettent à jour leurs copies locales de ces listes, cela peut prendre un temps considérable ou nécessiter une intervention manuelle de la part des utilisateurs finaux ou des services informatiques de l’entreprise pour forcer une mise à jour ou autoriser explicitement les points de terminaison CDN à résoudre le problème.

Si le point de terminaison CDN est identifié comme dangereux, [créez un ticket de support](https://azure.microsoft.com/support/create-ticket/) pour vous assurer que le problème est résolu dès que possible.

Pour *potentiellement* contourner ce problème plus rapidement, vous pouvez [modifier le point de terminaison CDN du Kit de développement logiciel (SDK)](#change-the-cdn-endpoint).

### <a name="application-insights-javascript-cdn-is-blocked-by-end-user---blocked-by-browser-installed-blocker-personal-firewall"></a>Blocage du CDN JavaScript Application Insights (par l’utilisateur final ou par le navigateur : bloqueur installé, pare-feu personnel)

Vérifiez si vos utilisateurs finaux ont :

- installé un plug-in de navigateur (généralement une forme de bloqueur de publicité, de programme malveillant ou de fenêtres contextuelles) ;
- bloqué (ou non autorisé) les points de terminaison CDN Application Insights dans leur navigateur ou proxy ;
- configuré une règle de pare-feu qui entraîne le blocage du domaine CDN pour le SDK (ou la non résolution de l’entrée DNS).

S’ils ont configuré l’une de ces options, vous devrez les aider à autoriser les points de terminaison CDN (ou fournir une documentation).

Il est possible que le plug-in qu’ils ont installé utilise la liste rouge publique. Si ce n’est pas le cas, il est probable qu’une autre solution soit configurée manuellement ou qu’il utilise une liste rouge de domaine privé.

#### <a name="add-exceptions-for-cdn-endpoints"></a>Ajouter des exceptions pour les points de terminaison CDN

Travaillez avec vos utilisateurs finaux ou fournissez-leur une documentation les informant qu’ils doivent autoriser le téléchargement de scripts à partir des points de terminaison CDN Application Insights en les incluant dans la liste des exceptions des règles de plug-in ou de pare-feu de leur navigateur (cela varie selon l’environnement de l’utilisateur final).

Voici un exemple de configuration de [Chrome pour autoriser ou refuser l’accès aux sites web.](https://support.google.com/chrome/a/answer/7532419?hl=en)

### <a name="application-insights-cdn-is-blocked-by-corporate-firewall"></a>Blocage du CDN Application Insights (par le pare-feu de l’entreprise)

Si vos utilisateurs finaux se trouvent sur un réseau d’entreprise, c’est probablement dû à leur solution de pare-feu et du fait que leur service informatique a implémenté une forme de système de filtrage Internet. Dans ce cas, vous devrez les aider à définir les règles nécessaires pour vos utilisateurs finaux.

#### <a name="add-exceptions-for-cdn-endpoints-for-corporations"></a>Ajouter des exceptions pour les points de terminaison CDN pour les entreprises

  Cela est similaire à [l’ajout d’exceptions pour les utilisateurs finaux](#add-exceptions-for-cdn-endpoints), mais vous devez collaborer avec le service informatique de l’entreprise pour qu’il configure les points de terminaison CDN Application Insights à télécharger en les incluant (ou en les supprimant) dans tout service de liste rouge ou de liste verte de domaine.

  > [!WARNING]
  > Si votre utilisateur d’entreprise utilise un [cloud privé](https://azure.microsoft.com/overview/what-is-a-private-cloud/) et qu’il ne peut autoriser aucune forme d’exception pour fournir à ses utilisateurs internes un accès public pour les points de terminaison CDN, vous devez utiliser les [packages NPM d’Application Insights](https://www.npmjs.com/package/applicationinsights) ou héberger le Kit de développement logiciel (SDK) Application Insights sur votre propre CDN.  

### <a name="additional-troubleshooting-for-blocked-cdn"></a>Résolution de problèmes supplémentaires pour le CDN bloqué :

> [!NOTE]
> Si vos utilisateurs utilisent un [cloud privé](https://azure.microsoft.com/overview/what-is-a-private-cloud/) et qu’ils n’ont pas accès à l’Internet public, vous devez héberger le Kit de développement logiciel (SDK) sur votre propre CDN ou utiliser des packages NPM.

#### <a name="host-the-sdk-on-your-own-cdn"></a>Héberger le Kit de développement logiciel (SDK) sur votre propre CDN

 Au lieu que vos utilisateurs finaux téléchargent le Kit de développement logiciel (SDK) Application Insights à partir du CDN public, vous pouvez l’héberger sur votre propre point de terminaison CDN. Il est recommandé d’utiliser une version spécifique (ai.2.#.#.min.js) afin de faciliter l’identification de la version que vous utilisez. Mettez aussi régulièrement à jour la version actuelle (ai.2.min.js) afin de pouvoir tirer parti des correctifs de bogues et des nouvelles fonctionnalités qui deviennent disponibles.

#### <a name="use-npm-packages-to-embed-the-application-insight-sdk"></a>Utiliser des packages NPM pour incorporer le Kit de développement logiciel (SDK) Application Insights

Au lieu d’utiliser l’extrait de code et les points de terminaison du CDN public, vous pouvez utiliser les [packages NPM](https://www.npmjs.com/package/applicationinsights) pour inclure le Kit de développement logiciel (SDK) dans vos propres fichiers JavaScript. Le Kit de développement logiciel (SDK) deviendra simplement un autre package dans vos scripts.

> [!NOTE]
> Lors de l’utilisation de packages NPM, il est recommandé d’utiliser également une forme de [bundler JavaScript](https://www.bing.com/search?q=javascript+bundler) pour faciliter le fractionnement et la minimisation du code.

Comme pour l’extrait de code, il est également possible que vos propres scripts (avec ou sans les packages NPM du SDK) soient concernés par les mêmes problèmes de blocage répertoriés ici. En fonction de votre application, de vos utilisateurs et de votre infrastructure, vous pouvez envisager d’implémenter quelque chose de similaire à la logique de l’extrait de code pour détecter et signaler ces problèmes.


## <a name="next-steps"></a>Étapes suivantes 
- [Obtenir de l’aide supplémentaire en soumettant un problème sur GitHub](https://github.com/Microsoft/ApplicationInsights-JS/issues)
- [Surveiller l’utilisation des pages web](javascript.md)
