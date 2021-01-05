---
title: Bonnes pratiques
description: Découvrez les bonnes pratiques et les scénarios de résolution des problèmes courants pour votre application s’exécutant dans Azure App Service.
author: dariagrigoriu
ms.assetid: f3359464-fa44-4f4a-9ea6-7821060e8d0d
ms.topic: article
ms.date: 07/01/2016
ms.author: dariac
ms.custom: seodec18
ms.openlocfilehash: 91fd974c730037907258cb4a670f6fa836bfda6c
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144873"
---
# <a name="best-practices-for-azure-app-service"></a>Meilleures pratiques pour Azure App Service
Cet article résume les meilleures pratiques dans l’utilisation de [Azure App Service](./overview.md). 

## <a name="colocation"></a><a name="colocation"></a>Colocalisation
Lorsqu’une base de données et des ressources Azure composant une solution, telle qu’une application web, se trouvent dans des régions différentes, les effets possibles sont les suivants :

* Une latence plus élevée dans la communication entre les ressources
* Des frais plus élevés pour le transfert de données sortantes entre les régions comme indiqué dans la [page de tarification Azure](https://azure.microsoft.com/pricing/details/data-transfers)

La colocalisation dans la même région est idéale pour les ressources Azure composant une solution telle qu’une application web et un compte de stockage ou de base de données utilisé pour conserver du contenu ou des données. Lors de la création de ressources, assurez-vous que celles-ci se trouvent dans la même région Azure, à moins que des motifs commerciaux ou conceptuels spécifiques entravent cette possibilité. Vous pouvez déplacer une application App Service vers la région dans laquelle se trouve votre base de données en utilisant la [fonctionnalité de clonage de l’application App Service](app-service-web-app-cloning.md), actuellement disponible pour les applications du plan App Service Premium.   

## <a name="when-apps-consume-more-memory-than-expected"></a><a name="memoryresources"></a>Quand les applications consomment plus de mémoire que prévu
Si vous remarquez qu’une application consomme plus de mémoire que prévu, ce qui est indiqué par des analyses ou des recommandations de service, envisagez d’exécuter la [fonctionnalité auto-curative d’App Service](https://azure.microsoft.com/blog/auto-healing-windows-azure-web-sites). Une des options de la fonctionnalité auto-curative consiste à prendre des actions personnalisées, basées sur un seuil de mémoire. Les actions vont des notifications par courrier électronique à la prévention immédiate des risques par le recyclage du processus de travail, en passant par l’examen via le vidage de la mémoire. La fonctionnalité auto-curative peut être configurée via le fichier web.config et une interface utilisateur conviviale, comme le décrit ce billet de blog pour l’ [extension du site de support App Service](https://azure.microsoft.com/blog/additional-updates-to-support-site-extension-for-azure-app-service-web-apps).   

## <a name="when-apps-consume-more-cpu-than-expected"></a><a name="CPUresources"></a>Quand les applications consomment plus d’UC que prévu
Si vous remarquez qu’une application consomme plus d’UC que prévu ou qu’elle provoque des pics de consommation d’UC répétés, ce qui indiqué par des analyses ou des recommandations de service, envisagez de faire monter ou descendre en puissance le plan App Service. Si votre application est « avec état », la seule option possible est la montée en charge. Si l’application est « sans état », l’augmentation de la taille des instances vous confère plus de souplesse et un potentiel de mise à l’échelle plus élevé. 

Pour plus d’informations sur les applications « sans état » et « avec état », regardez la vidéo : [Planning a Scalable End-to-End Multi-Tier Application on Azure App Service](https://channel9.msdn.com/Events/TechEd/NorthAmerica/2014/DEV-B414#fbid=?hashlink=fbid). Pour plus d’informations sur les options de montée en charge et de mise à l’échelle automatique, consultez [Mise à l’échelle d’une application web dans Microsoft Azure App Service](manage-scale-up.md).  

## <a name="when-socket-resources-are-exhausted"></a><a name="socketresources"></a>Quand les ressources de socket sont épuisées
L’utilisation des bibliothèques clientes constitue une raison courante de l’épuisement des connexions TCP sortantes ; ces bibliothèques ne sont pas implémentées pour la réutilisation des connexions TCP ou, lorsqu’un protocole de niveau supérieur tel que HTTP - Keep-Alive n’est pas utilisé. Veuillez consulter la documentation pour chacune des bibliothèques référencées par les applications dans votre plan App Service afin de vous assurer que celles-ci sont configurées ou accessibles dans votre code pour une réutilisation efficace des connexions sortantes. Suivez également les instructions de la documentation des bibliothèques pour une création et une mise en production appropriées ainsi que pour un nettoyage servant à éviter la fuite des connexions. Lorsque les examens des bibliothèques clientes sont en cours d’exécution, leur impact peut être atténué en augmentant la taille des instances.

### <a name="nodejs-and-outgoing-http-requests"></a>Node.js et requêtes http sortantes
Quand vous utilisez Node.js et de nombreuses requêtes http sortantes, il est important de prendre en compte le protocole HTTP - Keep-Alive. Vous pouvez utiliser le package [agentkeepalive](https://www.npmjs.com/package/agentkeepalive) `npm` pour en faciliter l’utilisation dans votre code.

Gérez toujours la réponse `http`, même si vous n’effectuez aucune opération dans le gestionnaire. Si vous ne gérez pas la réponse correctement, votre application finit par se bloquer, car aucun socket supplémentaire n’est disponible.

Par exemple, quand vous utilisez le package `http` ou `https` :

```javascript
const request = https.request(options, function(response) {
    response.on('data', function() { /* do nothing */ });
});
```

Si vous exécutez App Service sur Linux sur un ordinateur avec plusieurs cœurs, une autre bonne pratique consiste à utiliser PM2 pour démarrer plusieurs processus Node.js afin d’exécuter votre application. Pour ce faire, il suffit de spécifier une commande de démarrage à votre conteneur.

Par exemple, pour démarrer quatre instances :

```
pm2 start /home/site/wwwroot/app.js --no-daemon -i 4
```

## <a name="when-your-app-backup-starts-failing"></a><a name="appbackup"></a>La sauvegarde de votre application échoue
Les deux raisons les plus courantes qui entraînent l'échec de la sauvegarde d'une application sont les suivantes : des paramètres de stockage non valides et une configuration de base de données non valide. Ces échecs se produisent généralement lorsque des modifications ont été appliquées à des ressources de stockage ou de base de données, ou lorsque la méthode d'accès à ces ressources a changé (par ex. en cas de mise à jour des informations d’identification pour accéder à la base de données sélectionnée dans les paramètres de sauvegarde). En général, les sauvegardes s'exécutent de façon planifiée et requièrent l’accès au stockage (pour générer les fichiers sauvegardés) et aux bases de données (pour copier et lire le contenu à inclure dans la sauvegarde). L'incapacité à accéder à l'une de ces ressources entraîne une erreur de sauvegarde permanente. 

Lorsque des échecs de sauvegarde se produisent, veuillez consulter les résultats les plus récents pour comprendre quel type d'échec se produit. Si vous ne parvenez pas à accéder au stockage, examinez et mettez à jour les paramètres de stockage utilisés dans la configuration de la sauvegarde. Si vous ne parvenez pas à accéder à la base de données, examinez et mettez à jour vos chaînes de connexion dans les paramètres de l’application, puis mettez à jour la configuration de votre sauvegarde afin d’inclure correctement les bases de données requises. Pour plus d’informations sur la sauvegarde d’une application, consultez [Sauvegarder une application web dans Azure App Service](manage-backup.md).

## <a name="when-new-nodejs-apps-are-deployed-to-azure-app-service"></a><a name="nodejs"></a>Lorsque de nouvelles applications Node.js sont déployées sur Azure App Service
La configuration Azure App Service par défaut pour les applications Node.js vise à mieux répondre aux besoins des applications courantes. Si la configuration de votre application Node.js peut bénéficier d’un paramétrage personnalisé pour améliorer les performances ou optimiser l’utilisation des ressources au niveau processeur/mémoire/réseau, consultez [Meilleures pratiques et guide de dépannage pour les applications Node sur Azure App Service](app-service-web-nodejs-best-practices-and-troubleshoot-guide.md). Cet article décrit les paramètres iisnode, que vous devrez peut-être configurer pour votre application Node.js, présente les différents scénarios ou problèmes que votre application peut rencontrer, et indique comment résoudre ces problèmes.


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les meilleures pratiques, consultez [App Service Diagnostics](./overview-diagnostics.md) pour connaître les meilleures pratiques applicables spécifiques à votre ressource.

- Accédez à votre application web dans le [portail Azure](https://portal.azure.com).
- Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes** pour ouvrir Diagnostics App Service.
- Choisissez la vignette de page d’accueil **Bonnes pratiques**.
- Cliquez sur **Bonnes pratiques pour la disponibilité et les performances** ou **Bonnes pratiques pour une configuration optimale** afin d’afficher l’état actuel de votre application en ce qui concerne ces bonnes pratiques.

Vous pouvez également utiliser ce lien pour ouvrir directement Diagnostics App Service pour votre ressource : `https://ms.portal.azure.com/?websitesextension_ext=asd.featurePath%3Ddetectors%2FParentAvailabilityAndPerformance#@microsoft.onmicrosoft.com/resource/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{siteName}/troubleshoot`.