---
title: Questions fréquentes (FAQ) sur les conteneurs intégrés
description: Trouvez des réponses aux questions fréquentes sur les conteneurs Linux intégrés dans Azure App Service.
keywords: azure app service, application web, faq, linux, oss, web app pour conteneurs, multi-conteneur, multiconteneur
author: msangapu-msft
ms.topic: article
ms.date: 10/30/2018
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: e2ab39f090124aaf590798795f446bda5b0fe48a
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97008552"
---
# <a name="azure-app-service-on-linux-faq"></a>FAQ d’Azure App Service sur Linux

Avec la publication d’App Service sur Linux, nous travaillons à l’ajout de fonctionnalités et à l’amélioration de notre plateforme. Cet article fournit des réponses aux questions que nos clients nous ont posées récemment.

Si vous avez une question, un commentaire sur cet article.

## <a name="built-in-images"></a>Images prédéfinies

**Je veux répliquer les conteneurs Docker intégrés fournis par la plateforme. Où se trouvent ces fichiers ?**

Vous trouverez tous les fichiers Docker sur [GitHub](https://github.com/azure-app-service). Vous trouverez tous les conteneurs Docker sur [Docker Hub](https://hub.docker.com/u/appsvc/).

<a id="#startup-file"></a>

**Quelles sont les valeurs attendues de la section Fichier de démarrage lorsque je configure la pile d’exécution ?**

| Pile           | Valeur attendue                                                                         |
|-----------------|----------------------------------------------------------------------------------------|
| Java SE         | la commande pour démarrer votre application JAR (par exemple, `java -jar /home/site/wwwroot/app.jar --server.port=80`) |
| Tomcat          | l’emplacement d’un script pour effectuer toutes les configurations nécessaires (par exemple, `/home/site/deployments/tools/startup_script.sh`)          |
| Node.js         | le fichier de configuration PM2 ou votre fichier de script                                |
| .NET Core       | le nom de la DLL compilée en tant que `dotnet <myapp>.dll`                                 |
| Ruby            | le script Ruby avec lequel initialiser votre application                     |

Ces commandes ou scripts sont exécutés après le démarrage du conteneur Docker intégré, mais avant le démarrage de votre code d’application.

## <a name="management"></a>Gestion

**Que se passe-t-il lorsque j’appuie sur le bouton Redémarrer dans le portail Azure ?**

Cette action revient à redémarrer Docker.

**Puis-je utiliser Secure Shell (SSH) pour me connecter à la machine virtuelle du conteneur d’application ?**

Oui, vous pouvez le faire via le site de gestion de contrôle de code source (SCM).

> [!NOTE]
> Vous pouvez également vous connecter au conteneur d’application directement à partir de votre ordinateur de développement local à l’aide de SSH, SFTP ou Visual Studio Code (pour le débogage dynamique des applications Node.js). Pour plus d’informations, consultez [Remote debugging and SSH in App Service on Linux (Débogage à distance et technologie SSH dans App Service sur Linux)](https://azure.github.io/AppService/2018/05/07/New-SSH-Experience-and-Remote-Debugging-for-Linux-Web-Apps.html).
>

**Comment puis-je créer un plan App Service Linux via un kit de développement ou un modèle Azure Resource Manager ?**

Définissez le champ **reserved** du service d’application sur *true*.

## <a name="continuous-integration-and-deployment"></a>Intégration et déploiement continus

**Mon application web utilise toujours une ancienne image de conteneur Docker après la mise à jour de l’image sur Docker Hub. Prenez-vous en charge l’intégration et le déploiement continus de conteneurs personnalisés ?**

Oui. Pour configurer l’intégration/le déploiement continu(e) pour Azure Container Registry ou DockerHub, consultez [Déploiement continu avec Web App pour conteneurs](./deploy-ci-cd-custom-container.md). Pour les registres privés, vous pouvez actualiser le conteneur en arrêtant puis démarrant votre application web. Vous pouvez également modifier ou ajouter un paramètre d’application factice pour forcer une actualisation de votre conteneur.

**Prenez-vous en charge les environnements intermédiaires ?**

Oui.

**Puis-je utiliser *WebDeploy/MSDeploy* pour déployer mon application web ?**

Oui, vous devez définir le paramètre d’application `WEBSITE_WEBDEPLOY_USE_SCM` sur *false*.

**Le déploiement Git de mon application échoue quand j’utilise une application web Linux. Comment puis-je résoudre ce problème ?**

Si le déploiement Git sur votre application web Linux échoue, choisissez l’une des options suivantes pour déployer le code de votre application :

- Utilisez la fonctionnalité Livraison continue (préversion) : Vous pouvez stocker le code source de votre application dans un référentiel Git Azure DevOps ou un référentiel GitHub pour utiliser la livraison continue Azure. Pour plus d’informations, consultez [Configurer la livraison continue pour une application web Linux](https://blogs.msdn.microsoft.com/devops/2017/05/10/use-azure-portal-to-setup-continuous-delivery-for-web-app-on-linux/).

- Utilisez l’[API de déploiement ZIP](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file) : pour utiliser cette API, [connectez-vous via SSH à votre application web](configure-linux-open-ssh-session.md) et accédez au dossier où vous souhaitez déployer votre code. Exécutez le code ci-dessous :

   ```bash
   curl -X POST -u <user> --data-binary @<zipfile> https://{your-sitename}.scm.azurewebsites.net/api/zipdeploy
   ```

   Si vous obtenez une erreur stipulant que la commande `curl` est introuvable, veillez à installer curl à l’aide de `apt-get install curl` avant d’exécuter la commande `curl` précédente.

## <a name="language-support"></a>Support multilingue

**Je souhaite utiliser des sockets web dans mon application Node.js. Y a-t-il des configurations ou des paramètres spéciaux à définir ?**

Oui, vous devez désactiver `perMessageDeflate` dans votre code Node.js côté serveur. Par exemple, si vous utilisez socket.io, utilisez le code suivant :

```nodejs
const io = require('socket.io')(server,{
  perMessageDeflate :false
});
```

**Prenez-vous en charge les applications .NET Core non compilées ?**

Oui.

**Prenez-vous en charge Composer en tant que gestionnaire de dépendances pour les applications PHP ?**

Oui. Lors d’un déploiement Git, Kudu doit détecter que vous déployez une application PHP (grâce à la présence d’un fichier composer.lock) et déclenchera une installation Composer.

## <a name="custom-containers"></a>Conteneurs personnalisés

**J’utilise mon propre conteneur personnalisé. Je souhaite que la plateforme monte un partage SMB dans le répertoire `/home/`.**

Si le paramètre `WEBSITES_ENABLE_APP_SERVICE_STORAGE` est **non spécifié** ou est défini sur *true*, le répertoire `/home/`**sera partagé** par les instances, et les fichiers écrits **seront conservés** après chaque redémarrage. Si vous définissez explicitement `WEBSITES_ENABLE_APP_SERVICE_STORAGE` sur *false*, le montage est désactivé.

**Mon conteneur personnalisé met longtemps à démarrer, et la plateforme le redémarre avant qu’il ait terminé.**

Vous pouvez configurer le temps que la plateforme doit attendre avant qu’elle redémarre votre conteneur. Pour ce faire, définissez le paramètre d’application `WEBSITES_CONTAINER_START_TIME_LIMIT` sur la valeur souhaitée. La valeur par défaut est de 230 secondes, la valeur maximale de 1800 secondes.

**Quel est le format de l’URL du serveur de registre privé ?**

Vous devez fournir l’URL de registre complète, y compris `http://` ou `https://`.

**Quel est le format du nom d’image dans l’option de registre privé ?**

Ajoutez le nom complet de l’image, comprenant l’URL de registre privé (par exemple, myacr.azurecr.io/dotnet:latest). Les noms d’image qui utilisent un port personnalisé [ne peuvent pas être entrés par le biais du portail](https://feedback.azure.com/forums/169385-web-apps/suggestions/31304650). Pour définir `docker-custom-image-name`, utilisez [l’outil en ligne de commande `az`](/cli/azure/webapp/config/container#az-webapp-config-container-set).

**Je veux exposer plusieurs ports sur l’image de mon conteneur personnalisé.**

Nous ne prenons pas en charge l’exposition de plusieurs ports.

**Puis-je apporter mon propre système de stockage ?**

Oui, le mode [Bring Your Own Storage](./configure-connect-to-azure-storage.md) est disponible en préversion.

**Pourquoi est-il impossible de parcourir le système de fichiers de mon conteneur personnalisé à partir du site SCM ?**

Le site SCM s’exécute dans un conteneur distinct. Vous ne pouvez pas vérifier le système de fichiers ou les processus en cours d’exécution du conteneur d’application.

**Mon conteneur personnalisé écoute un autre port que le port 80. Comment puis-je configurer mon application pour acheminer les demandes vers ce port ?**

Nous avons la détection automatique du port. Vous pouvez également spécifier le paramètre d’application *WEBSITES_PORT* et lui attribuer la valeur du numéro de port attendu. Auparavant, la plateforme utilisait le paramètre d’application *PORT*. Nous avons l’intention de déconseiller ce paramètre d’application pour utiliser exclusivement *WEBSITES_PORT*.

**Dois-je implémenter HTTPS dans mon conteneur personnalisé ?**

Non, la plateforme gère l’annulation HTTPS au niveau des serveurs frontaux partagés.

## <a name="multi-container-with-docker-compose"></a>Plusieurs conteneurs avec Docker Compose

**Comment configurer Azure Container Registry (ACR) de manière à utiliser plusieurs conteneurs ?**

Pour utiliser ACR avec plusieurs conteneurs, **toutes les images de conteneur** doivent être hébergées sur le même serveur de Registre ACR. Une fois qu’elles se trouvent sur le même serveur de Registre, vous devez créer des paramètres d’application, puis mettre à jour le fichier config Docker Compose en y ajoutant le nom de l’image ACR.

Créez les paramètres d’application suivants :

- DOCKER_REGISTRY_SERVER_USERNAME
- DOCKER_REGISTRY_SERVER_URL (URL complète, p. ex. : `https://<server-name>.azurecr.io`)
- DOCKER_REGISTRY_SERVER_PASSWORD (activez l’accès administrateur dans les paramètres ACR)

Dans le fichier de configuration, référencez votre image ACR comme dans l’exemple suivant :

```yaml
image: <server-name>.azurecr.io/<image-name>:<tag>
```

**Comment savoir quel conteneur est accessible via Internet ?**

- Un seul conteneur peut être ouvert en vue d’y accéder
- Seuls les ports 80 et 8080 sont accessibles (ports exposés)

Voici les règles pour déterminer quel conteneur est accessible, par ordre d’importance :

- Le paramètre d’application `WEBSITES_WEB_CONTAINER_NAME` doit être défini sur le nom du conteneur.
- Le premier conteneur doit définir le port 80 ou 8080.
- Si aucune de ces règles n’est appliquée, le premier conteneur défini dans le fichier sera accessible (exposé).


## <a name="web-sockets"></a>Web Sockets

Les Web Sockets sont pris en charge sur les applications Linux.

> [!IMPORTANT]
> Les Web Sockets ne sont actuellement pas pris en charge pour les applications Linux sur les plans Free App Service. Nous travaillons à la suppression de cette limitation et nous prévoyons de prendre en charge jusqu’à 5 connexions de Web Socket sur les plans Free App Service.

## <a name="pricing-and-sla"></a>Tarifs et contrat SLA

**À présent que le service est disponible généralement, quels sont les tarifs ?**

La tarification varie en fonction de la référence et de la région, mais vous pouvez voir plus de détails sur notre page de tarification : [Tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/linux/).

## <a name="other-questions"></a>Autres questions

**Que signifie « La fonctionnalité demandée n'est pas disponible dans le groupe de ressources » ?**

Ce message peut apparaître lors de la création d'une application web à l'aide d'Azure Resource Manager (ARM). Sur la base d'une limitation actuelle, pour un même groupe de ressources, vous ne pouvez pas mélanger des applications Windows et Linux au sein d'une même région.

**Quels sont les caractères pris en charge dans les noms de paramètres d’application ?**

Vous pouvez utiliser uniquement des lettres (A-Z, a-z), des chiffres (0-9) et le trait de soulignement (_) comme paramètres d’application.

**Où puis-je demander de nouvelles fonctionnalités ?**

Vous pouvez proposer votre idée sur le [Forum de commentaires pour Web Apps](https://aka.ms/webapps-uservoice). Ajoutez « [Linux] » au titre de votre idée.

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’Azure App Service sur Linux ?](overview.md#app-service-on-linux)
- [Configurer des environnements intermédiaires dans Azure App Service](deploy-staging-slots.md)
- [Déploiement continu avec Web App pour conteneurs](./deploy-ci-cd-custom-container.md)
