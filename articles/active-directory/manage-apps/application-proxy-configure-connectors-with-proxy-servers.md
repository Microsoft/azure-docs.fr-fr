---
title: Travailler avec des serveurs proxy locaux existants et Azure Active Directory
description: Explique comment travailler avec des serveurs proxy locaux existants avec Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 04/07/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: contperf-fy21q2
ms.openlocfilehash: 808357b95f4de904ead0741d848480d548a2e26a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030074"
---
# <a name="work-with-existing-on-premises-proxy-servers"></a>Travailler avec des serveurs proxy locaux existants

Cet article explique comment configurer les connecteurs de proxy d’application Azure Active Directory (Azure AD) pour fonctionner avec des serveurs proxy sortants. Il est destiné aux clients avec des environnements réseau qui ont des proxys existants.

Nous allons commencer par examiner les scénarios de déploiement principaux suivants :

* Configurer des connecteurs pour contourner vos proxys sortants locaux.
* Configurer des connecteurs pour utiliser un proxy sortant afin d’accéder au proxy d’application Azure AD.
* Configurer à l’aide d’un proxy entre le connecteur et l’application principale.

Pour plus d’informations sur le fonctionnement des connecteurs, consultez [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md).

## <a name="bypass-outbound-proxies"></a>Proxys sortants de contournement

Les connecteurs ont des composants de système d’exploitation sous-jacents qui effectuent des demandes sortantes. Ces composants tentent automatiquement de trouver un serveur proxy sur le réseau à l’aide de la découverte automatique de proxy web (protocole WPAD).

Les composants du système d’exploitation tentent de localiser un serveur proxy en effectuant une recherche DNS pour wpad.domainsuffix. Si la recherche renvoie un DNS, une requête HTTP est alors effectuée sur l’adresse IP pour wpad.dat. Cette requête devient le script de configuration de proxy dans votre environnement. Le connecteur utilise ce script pour sélectionner un serveur proxy sortant. Toutefois, le trafic du connecteur peut ne pas toujours passer, en raison des paramètres de configuration supplémentaires nécessaires sur le serveur proxy.

Vous pouvez configurer le connecteur de sorte qu’il contourne votre proxy local pour vous assurer qu’il utilise une connexion directe aux services Azure. C’est ce que nous recommandons, tant que votre stratégie réseau le permet, car cela signifie que vous aurez une configuration de moins à mettre à jour.

Pour désactiver l’utilisation d’un proxy sortant pour le connecteur, modifiez le fichier C:\Program Files\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config et ajoutez la section *system.net* indiquée dans cet exemple de code :

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.net>
    <defaultProxy enabled="false"></defaultProxy>
  </system.net>
  <runtime>
    <gcServer enabled="true"/>
  </runtime>
  <appSettings>
    <add key="TraceFilename" value="AadAppProxyConnector.log" />
  </appSettings>
</configuration>
```

Pour vérifier que le service de mise à jour du connecteur ignore également le proxy, apportez une modification semblable au fichier ApplicationProxyConnectorUpdaterService.exe.config. Ce fichier se trouve dans C:\Program Files\Microsoft AAD App Proxy Connector Updater.

Veillez à faire des copies des fichiers d’origine, au cas où vous devriez restaurer les fichiers par défaut.

## <a name="use-the-outbound-proxy-server"></a>Utiliser le serveur proxy sortant

Certains environnements clients requièrent que tout le trafic sortant passe par un proxy sortant, sans exception. Par conséquent, le contournement du serveur proxy n’est pas une option.

Vous pouvez configurer le trafic du connecteur pour passer par le serveur proxy sortant comme indiqué dans le diagramme suivant :

 ![Configuration du trafic de connecteur pour passer par un proxy sortant vers un proxy d’application Azure AD](./media/application-proxy-configure-connectors-with-proxy-servers/configure-proxy-settings.png)

Conséquence de la présence seule de trafic sortant, il est inutile de configurer l’accès entrant à travers vos pare-feu.

> [!NOTE]
> Le proxy d’application ne prend pas en charge l’authentification auprès d’autres proxys. Les comptes du service réseau de mise à jour/connecteur doivent être en mesure de se connecter au proxy sans avoir à se connecter avec l’authentification.

### <a name="step-1-configure-the-connector-and-related-services-to-go-through-the-outbound-proxy"></a>Étape 1 : Configurer le connecteur et les services associés pour passer par le proxy sortant

Si WPAD est activé dans l’environnement et correctement configuré, le connecteur détecte automatiquement le serveur proxy sortant et tente de l’utiliser. Toutefois, vous pouvez configurer explicitement le connecteur pour passer par un proxy sortant.

Pour ce faire, modifiez le fichier C:\Program Files\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config et ajoutez la section *system.net* indiquée dans cet exemple de code. Modifiez *proxyserver:8080* afin de refléter le nom de votre serveur proxy local ou l’adresse IP et le port qu’il écoute. La valeur doit avoir le préfixe http:// même si vous utilisez une adresse IP.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.net>  
    <defaultProxy>   
      <proxy proxyaddress="http://proxyserver:8080" bypassonlocal="True" usesystemdefault="True"/>   
    </defaultProxy>  
  </system.net>
  <runtime>
    <gcServer enabled="true"/>
  </runtime>
  <appSettings>
    <add key="TraceFilename" value="AadAppProxyConnector.log" />
  </appSettings>
</configuration>
```

Configurez ensuite le service de mise à jour du connecteur pour utiliser le proxy en apportant une modification similaire au fichier C:\Program Files\Microsoft AAD App Proxy Connector Updater\ApplicationProxyConnectorUpdaterService.exe.config.

### <a name="step-2-configure-the-proxy-to-allow-traffic-from-the-connector-and-related-services-to-flow-through"></a>Étape 2 : Configurer le serveur proxy pour autoriser le passage du trafic à partir du connecteur et des services associés

Il y a quatre aspects à prendre en compte au niveau du proxy sortant :

* Règles sortantes du proxy
* Authentification du proxy
* Ports du proxy
* Inspection TLS

#### <a name="proxy-outbound-rules"></a>Règles sortantes du proxy

Autorisez l'accès aux URL suivantes :

| URL | Port |  Utilisation |
| --- | --- | --- |
| &ast;.msappproxy.net<br>&ast;.servicebus.windows.net | 443/HTTPS | Communication entre le connecteur et le service cloud Proxy d'application |
| crl3.digicert.com<br>crl4.digicert.com<br>ocsp.digicert.com<br>crl.microsoft.com<br>oneocsp.microsoft.com<br>ocsp.msocsp.com<br> | 80/HTTP | Le connecteur utilise ces URL pour vérifier les certificats. |
| login.windows.net<br>secure.aadcdn.microsoftonline-p.com<br>&ast;.microsoftonline.com<br>&ast;.microsoftonline-p.com<br>&ast;.msauth.net<br>&ast;.msauthimages.net<br>&ast;.msecnd.net<br>&ast;.msftauth.net<br>&ast;.msftauthimages.net<br>&ast;.phonefactor.net<br>enterpriseregistration.windows.net<br>management.azure.com<br>policykeyservice.dc.ad.msft.net<br>ctldl.windowsupdate.com | 443/HTTPS | Le connecteur utilise ces URL lors du processus d'inscription. |
| ctldl.windowsupdate.com | 80/HTTP | Le connecteur utilise cette URL lors du processus d'inscription. |

Si votre pare-feu ou proxy vous permet de configurer la mise en liste verte de DN, vous pouvez autoriser les connexions à \*.msappproxy.net et \*.servicebus.windows.net.

Si vous ne pouvez pas autoriser la connectivité par le nom de domaine complet et devez spécifier des plages d’adresses IP à la place, utilisez ces options :

* Autoriser l’accès sortant du connecteur vers toutes les destinations.
* Autorisez l’accès sortant du connecteur à toutes les plages d’adresses IP de centre de données Azure. Le problème lié à l’utilisation de la liste de plages d’adresses IP de centre de données Azure est qu’elle est mise à jour chaque semaine. Vous devez mettre un processus en place pour garantir que vos règles d’accès sont mises à jour en conséquence. La seule utilisation d’un sous-ensemble des adresses IP peut entraîner une rupture de votre configuration. Pour télécharger les dernières plages d’adresses IP du centre de données Azure, accédez à [https://download.microsoft.com](https://download.microsoft.com) et recherchez « Balises de service et plages d’adresses IP Azure ». Veillez à sélectionner le cloud approprié. Par exemple, pour connaître les plages d’adresses IP du cloud public, recherchez « Balises de service et plages d’adresses IP Azure – Cloud public ». Vous trouverez le cloud US Government en recherchant « Balises de service et plages d’adresses IP Azure – Cloud US Goverment ».

#### <a name="proxy-authentication"></a>Authentification du proxy

L’authentification proxy n'est actuellement pas prise en charge. Notre recommandation actuelle est de permettre au connecteur d’accéder de façon anonyme aux destinations Internet.

#### <a name="proxy-ports"></a>Ports du proxy

Le connecteur établit les connexions sortantes TLS à l’aide de la méthode CONNECT. Cette méthode sert à définir un tunnel via le serveur proxy sortant. Configurez le serveur proxy pour autoriser le tunneling vers les ports 443 et 80.

> [!NOTE]
> Lorsque Service Bus s’exécute via le protocole HTTPS, il utilise le port 443. Toutefois, par défaut, Service Bus tente des connexions TCP directes et bascule sur HTTPS uniquement si la connectivité directe échoue.

#### <a name="tls-inspection"></a>Inspection TLS

N’utilisez pas l’inspection TLS pour le trafic de connecteur, car cela entraîne des problèmes au niveau de celui-ci. Le connecteur utilise un certificat pour s’authentifier auprès du service Proxy d’application, et ce certificat peut se perdre pendant l’inspection TLS.

## <a name="configure-using-a-proxy-between-the-connector-and-backend-application"></a>Configurer à l’aide d’un proxy entre le connecteur et l’application principale
L’utilisation d’un proxy direct pour la communication vers l’application principale peut être une exigence spéciale dans certains environnements.
Pour ce faire, suivez les étapes suivantes :

### <a name="step-1-add-the-required-registry-value-to-the-server"></a>Étape 1 : Ajouter la valeur de registre requise au serveur
1. Pour activer l’utilisation du proxy par défaut, ajoutez la valeur de registre suivante (DWORD) `UseDefaultProxyForBackendRequests = 1` à la clé de registre de configuration du connecteur située dans « HKEY_LOCAL_MACHINE\Software\Microsoft\Microsoft AAD App Proxy Connector ».

### <a name="step-2-configure-the-proxy-server-manually-using-netsh-command"></a>Étape 2 : Configurer manuellement le serveur proxy à l’aide de la commande netsh
1.  Activez la stratégie de groupe Paramètres machine du serveur proxy. Elle se trouve à l’emplacement suivant : Configuration ordinateur\Stratégies\Modèles d’administration\Composants Windows\Internet Explorer. Cela doit être défini sur cette valeur au lieu d’avoir cette stratégie définie avec des paramètres individualisés.
2.  Exécutez `gpupdate /force` sur le serveur ou redémarrez le serveur pour vous assurer qu’il utilise les paramètres de stratégie de groupe mis à jour.
3.  Lancez une invite de commandes avec élévation de privilèges avec des droits d’administrateur et saisissez `control inetcpl.cpl`.
4.  Configurez les paramètres de proxy requis. 

Ces paramètres font en sorte que le connecteur utilise le même proxy de transfert pour la communication avec Azure et avec l’application principale. Si le connecteur pour la communication avec Azure ne requiert pas de proxy de transfert ou requiert un proxy de transfert différent, vous pouvez le configurer en modifiant le fichier ApplicationProxyConnectorService.exe.config, comme décrit dans les sections Proxys sortants de contournement et Utiliser le serveur proxy sortant.

> [!NOTE]
> Il existe plusieurs façons de configurer le proxy Internet dans le système d’exploitation. Les paramètres de proxy configurés via NETSH WINHTTP (exécutez `NETSH WINHTTP SHOW PROXY` pour vérifier) remplacent les paramètres de proxy que vous avez configurés à l’étape 2. 

Le service de mise à jour du connecteur utilisera également le proxy de la machine. Ce comportement peut être modifié en modifiant le fichier ApplicationProxyConnectorUpdaterService.exe.config.

## <a name="troubleshoot-connector-proxy-problems-and-service-connectivity-issues"></a>Résoudre les problèmes courants de proxy de connecteur et de connectivité du service

Vous devriez maintenant voir tout le trafic transitant par le proxy. Si vous rencontrez des problèmes, les informations de résolution des problèmes suivantes devraient vous aider.

Le meilleur moyen d’identifier et de résoudre les problèmes de connectivité de connecteur consiste à prendre une capture réseau au démarrage du service de connecteur. Voici quelques conseils rapides sur la capture et le filtrage de traces réseau.

Vous pouvez utiliser l’outil de surveillance de votre choix. Dans le cadre de cet article, nous avons utilisé Microsoft Message Analyzer.

> [!NOTE]
> Le 25 novembre 2019, [Microsoft Message Analyzer (MMA) a été mis hors service](https://docs.microsoft.com/openspecs/blog/ms-winintbloglp/dd98b93c-0a75-4eb0-b92e-e760c502394f) et ses packages de téléchargement ont été supprimés des sites microsoft.com.  Aucune fonctionnalité Microsoft n'est actuellement en développement pour remplacer Microsoft Message Analyzer.  Pour bénéficier d'une fonctionnalité similaire, vous devez envisager d'utiliser un outil d'analyse de protocole réseau tiers tel que Wireshark.

Les exemples suivants sont spécifiques de Message Analyser, mais les principes peuvent être appliqués à n’importe quel outil d’analyse.

### <a name="take-a-capture-of-connector-traffic"></a>Effectuez une capture du trafic de connecteur

Pour commencer la résolution des problèmes, procédez comme suit :

1. À partir de services.msc, arrêtez le service de connecteur de proxy d’application Azure AD.

   ![Service de connecteur de proxy d’application Azure AD dans services.msc](./media/application-proxy-configure-connectors-with-proxy-servers/services-local.png)

1. Exécutez Message Analyser en tant qu’administrateur.
1. Sélectionnez **Start local trace (Démarrer la trace locale)** .
1. Démarrez le service de connecteur de proxy d’application Azure AD.
1. Arrêtez la capture réseau.

   ![Capture d’écran montrant le bouton Arrêter la capture réseau](./media/application-proxy-configure-connectors-with-proxy-servers/stop-trace.png)

### <a name="check-if-the-connector-traffic-bypasses-outbound-proxies"></a>Vérifier si le trafic du connecteur ignore les proxys sortants

Si vous avez configuré votre connecteur de proxy d’application de sorte à ce qu’il ignore les serveurs proxy et qu’il se connecte directement au service de proxy d’application, recherchez dans la capture réseau les tentatives de connexion TCP qui ont échoué.

Utilisez le filtre Message Analyzer pour identifier ces tentatives. Entrez `property.TCPSynRetransmit` dans la zone de filtre, puis sélectionnez **Appliquer**.

Un paquet SYN est le premier paquet envoyé pour établir une connexion TCP. Si ce paquet ne renvoie aucune réponse, le SYN est renvoyé. Vous pouvez utiliser le filtre précédent pour voir les SYN retransmis. Ensuite, vous pouvez vérifiez si ces SYN correspondent à du trafic lié à un connecteur.

Si vous vous attendez à ce que le connecteur établisse des connexions directes aux services Azure, les réponses SynRetransmit sur le port 443 sont une indication que vous avez un problème de réseau ou de pare-feu.

### <a name="check-if-the-connector-traffic-uses-outbound-proxies"></a>Vérifier si le trafic du connecteur utilise des proxys sortants

Si vous avez configuré le trafic du connecteur de votre proxy d’application de sorte à ce qu’il passe par les serveurs proxy, recherchez les connexions HTTPS à votre serveur proxy qui ont échoué.

Pour filtrer la capture réseau de sorte à obtenir ces tentatives de connexion, entrez `(https.Request or https.Response) and tcp.port==8080` dans le filtre Message Analyzer, en remplaçant 8080 par le port de votre service proxy. Sélectionnez **Appliquer** pour voir les résultats du filtre.

Le filtre précédent affiche uniquement les requêtes et réponses HTTPS vers/depuis le port du proxy. Vous recherchez les requêtes CONNECT indiquant une communication avec le serveur proxy. En cas de succès, vous obtenez une réponse HTTP OK (200).

Si vous voyez d’autres codes de réponse, comme 407 ou 502, cela signifie que le proxy nécessite une authentification ou n’autorise pas le trafic pour une raison quelconque. À ce stade, vous impliquez l’équipe de support technique de votre serveur proxy.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md)
* Si vous avez des problèmes de connectivité du connecteur, posez votre question sur la [Page de questions Microsoft Q&A pour Azure Active Directory](/answers/topics/azure-active-directory.html) ou créez un ticket auprès de notre équipe de support.
