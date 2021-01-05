---
title: Présentation des connecteurs de proxy d’application Azure AD | Microsoft Docs
description: Présentation des connecteurs de proxy d’application Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 11/15/2018
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8086bd2a193ac52e76bf8da245063163ab2ea2f9
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97591053"
---
# <a name="understand-azure-ad-application-proxy-connectors"></a>Présentation des connecteurs de proxy d’application Azure AD

Les connecteurs rendent possible le proxy d’application Azure AD. Ils sont simples, faciles à déployer et à gérer et très puissants. Cet article présente les connecteurs, leur fonctionnement et des suggestions pour optimiser le déploiement.

## <a name="what-is-an-application-proxy-connector"></a>Qu’est-ce qu’un connecteur de proxy d’application ?

Les connecteurs sont des agents légers présent en local et qui facilitent la connexion sortante vers le service de proxy d’application. Les connecteurs doivent être installés sur un serveur Windows Server qui a accès à l’application principale. Vous pouvez organiser des connecteurs dans les groupes de connecteurs, et chaque groupe gère le trafic vers des applications spécifiques.

## <a name="requirements-and-deployment"></a>Exigences et déploiement

Pour déployer le proxy d’application avec succès, vous devez disposer d’au moins un connecteur, mais nous vous recommandons d’en utiliser deux ou plusieurs pour assurer une meilleure résilience. Installez le connecteur sur une machine exécutant Windows Server 2012 R2 ou une version ultérieure. Le connecteur doit communiquer avec le service Proxy d’application et les applications locales que vous publiez.

### <a name="windows-server"></a>Windows Server
Vous avez besoin d’un serveur exécutant Windows Server 2012 R2 ou ultérieur, sur lequel vous pouvez installer le connecteur de proxy d’application. Le serveur doit se connecter aux services Proxy d’application dans Azure et aux applications locales que vous publiez.

TLS 1.2 doit être activé sur le serveur Windows Server avant l’installation du connecteur de proxy d’application. Pour activer TLS 1.2 sur le serveur :

1. Définissez les clés de Registre suivantes :

    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

1. Redémarrez le serveur

Pour plus d’informations sur la configuration réseau requise pour le serveur du connecteur, consultez [Prise en main du proxy d’application et de l’installation d’un connecteur](application-proxy-add-on-premises-application.md).

## <a name="maintenance"></a>Maintenance

Les connecteurs et le service se chargent de toutes les tâches de haut niveau de disponibilité. Vous pouvez les ajouter ou supprimer de manière dynamique. Chaque fois qu’une nouvelle requête arrive, elle est acheminée vers un des connecteurs actuellement disponibles. Si un connecteur est temporairement indisponible, il ne répond pas à ce trafic.

Les connecteurs sont sans état et ne disposent d’aucune donnée de configuration sur l’ordinateur. Les seules données qu’ils stockent sont les paramètres de connexion au service et le certificat d’authentification. Lorsqu’ils se connectent au service, ils extraient toutes les données de configuration requises et les actualisent toutes les deux minutes.

Les connecteurs interrogent également le serveur pour déterminer s’il existe une version plus récente du connecteur. S’il en existe une, les connecteurs se mettent à jour.

Vous pouvez surveiller vos connecteurs à partir de l’ordinateur sur lequel ils s’exécutent à l’aide du journal d’événements et des compteurs de performances. Ou vous pouvez afficher leur état à partir de la page du proxy d’application du portail Azure :

![Exemple : Connecteurs de proxy d’application Azure AD](./media/application-proxy-connectors/app-proxy-connectors.png)

Vous n’êtes pas obligé de supprimer manuellement les connecteurs qui ne sont pas utilisés. Lorsqu’un connecteur est en cours d’exécution, il reste actif car il se connecte au service. Les connecteurs inutilisés sont marqués comme _inactifs_ et sont supprimés après 10 jours d’inactivité. Toutefois, si vous souhaitez réellement désinstaller un connecteur, désinstallez le service du connecteur et le service de mise à jour du serveur. Redémarrez votre ordinateur pour supprimer complètement le service.

## <a name="automatic-updates"></a>Mises à jour automatiques

Azure AD fournit les mises à jour automatiques pour tous les connecteurs que vous déployez. Tant que le service de mise à jour du connecteur de proxy d’application est en cours d’exécution, vos connecteurs se mettent automatiquement à jour. Si vous ne voyez pas le service de mise à jour du connecteur sur votre serveur, vous devez [réinstaller votre connecteur](application-proxy-add-on-premises-application.md) afin d’obtenir les mises à jour.

Si vous ne souhaitez pas attendre le chargement d’une mise à jour automatique sur votre connecteur, vous pouvez effectuer une mise à niveau manuelle. Accédez à la [page de téléchargement du connecteur](https://download.msappproxy.net/subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/connector/download) sur le serveur où votre connecteur se trouve et sélectionnez **Télécharger**. Ce processus lance une mise à niveau du connecteur local.

Pour les abonnés avec plusieurs connecteurs, les mises à jour automatiques ciblent un seul connecteur à la fois dans chaque groupe afin d’éviter les temps d’arrêt dans votre environnement.

Vous pouvez rencontrer des temps d’arrêt lors de la mise à jour de votre connecteur si :
  
- Vous n’avez qu’un seul connecteur. Nous vous recommandons d’en installer un deuxième et de [créer un groupe de connecteurs](application-proxy-connector-groups.md) pour éviter les temps morts et accroître la disponibilité.  
- Un connecteur se trouvait au milieu d’une transaction lorsque la mise à jour a commencé. Bien que la transaction d’origine soit perdue, votre navigateur devrait automatiquement relancer l’opération, ou vous pouvez actualiser votre page. Lorsque la demande est renvoyée, le trafic est acheminé vers un connecteur de secours.

Pour avoir des informations sur les versions précédentes et les changements qu’elles incluent, consultez [Proxy d’application - Historique des versions](application-proxy-release-version-history.md).

## <a name="creating-connector-groups"></a>Créer des groupe de connecteurs

Les groupes de connecteurs vous permettent d’assigner des connecteurs spécifiques afin de servir des applications spécifiques. Vous pouvez regrouper plusieurs connecteurs, puis assigner chaque application à un groupe.

Les groupes de connecteurs facilitent la gestion de déploiements à grande échelle. Ils améliorent aussi la latence pour les locataires dont les applications sont hébergées dans différentes régions, car vous pouvez créer des groupes de connecteurs basés sur un emplacement pour servir uniquement des applications locales.

Pour en savoir plus sur les groupes de connecteurs, consultez [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md).

## <a name="capacity-planning"></a>planification de la capacité

Il est important de planifier une capacité suffisante entre les connecteurs pour gérer le volume de trafic attendu. Nous recommandons la présence d’au moins deux connecteurs par groupe de connecteurs à des fins de haute disponibilité et de mise à l’échelle. La présence de trois connecteurs est idéale si vous devez effectuer une maintenance sur l’une de vos machines.

En général, plus il y a d’utilisateurs, plus l’ordinateur doit avoir des capacités importantes. Le tableau ci-dessous présente le volume et la latence attendue que les différentes machines peuvent gérer. Veuillez noter qu’il s’appuie entièrement sur le nombre attendu de transactions par seconde (TPS) plutôt que par utilisateur, étant donné que les modèles d’utilisation varient et ne peuvent servir à prévoir une charge. Il y aura par ailleurs des différences en fonction de la taille des réponses et du temps de réponse de l’application principale : plus la réponse est volumineuse et le temps de réponse lent, plus le TPS maximal est faible. Nous vous recommandons également de prévoir des machines supplémentaires pour permettre à la charge distribuée des machines d'offrir suffisamment de mémoire tampon. Cette capacité additionnelle garantit résilience et haute disponibilité.

|Cœurs|RAM|Latence attendue (MS)-P99|TPS max.|
| ----- | ----- | ----- | ----- |
|2|8|325|586|
|4|16|320|1150|
|8|32|270|1190|
|16|64|245|1200*|

\* Cette machine a utilisé un paramètre personnalisé pour déclencher certaines limites de connexion par défaut au-delà des paramètres .NET recommandés. Nous vous recommandons d’exécuter un test avec les paramètres par défaut avant de contacter le support technique pour que cette limite soit modifiée pour votre abonné.

> [!NOTE]
> L’utilisation d’une machine utilisant 4, 8 ou 16 cœurs n’entraîne pas de grandes différences au niveau des TPS maximales. La principale différence entre ces machines se situe au niveau de la latence attendue.
>
> Ce tableau est également axé sur les performances attendues d’un connecteur en fonction du type de machine sur laquelle il est installé. Pour les seuils de limitation du service proxy d’application, consultez [Restrictions et limites du service](../enterprise-users/directory-service-limits-restrictions.md).

## <a name="security-and-networking"></a>Sécurité et mise en réseau

Les connecteurs peuvent être installés n’importe où sur le réseau pourvu qu’ils puissent envoyer des requêtes vers le service de proxy d’application. L’important est que l’ordinateur qui exécute le connecteur dispose également d’un accès à vos applications. Vous pouvez installer les connecteurs à l’intérieur de votre réseau d’entreprise ou sur une machine virtuelle qui s’exécute dans le cloud. Les connecteurs peuvent s’exécuter dans réseau de périmètre, également appelé zone démilitarisée (DMZ), mais ce n’est pas nécessaire car tout le trafic est sortant afin sécuriser votre réseau.

Les connecteurs envoient uniquement des demandes sortantes. Le trafic sortant est envoyé au service de proxy d’application et aux applications publiées. Il n’est pas nécessaire pour ouvrir des ports d’entrée car le trafic passe dans les deux sens une fois qu’une session est établie. Il n’est pas non plus nécessaire de configurer l’accès entrant à travers les pare-feu.

Pour plus d’informations sur la configuration des règles sortantes de pare-feu, consultez [Travailler avec des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md).

## <a name="performance-and-scalability"></a>Performances et évolutivité

La mise à l’échelle pour le proxy d’application est transparente, mais l’échelle est un facteur lorsqu’il s’agit de connecteurs. Vous devez disposer de suffisamment de connecteurs pour gérer les pics de trafic. Puisqu’ils sont sans état, les connecteurs ne sont pas affectés par le nombre d’utilisateurs ou de sessions. Ils varient plutôt selon le nombre de requêtes et leur taille de charge utile. Avec le trafic web standard, une machine moyenne peut gérer quelques milliers de requêtes par seconde. La capacité spécifique dépend des caractéristiques exactes de la machine.

Les performances du connecteur sont liées au processeur et à la mise en réseau. Un processeur performant est nécessaire au chiffrement et au déchiffrement TLS, tandis que le réseau est important pour obtenir une connectivité rapide aux applications et au service en ligne dans Azure.

En revanche, la mémoire est moins problématique pour les connecteurs. Le service en ligne s’occupe de la majeure partie du traitement et de tout le trafic non authentifié. Tout ce qui peut être effectué dans le cloud est réalisé dans le cloud.

Si, pour une raison quelconque, la machine ou le connecteur deviennent indisponibles, le trafic passe à un autre connecteur du groupe. Cette résilience est également la raison pour laquelle nous vous recommandons d’avoir plusieurs connecteurs.

Un autre facteur affectant les performances est la qualité de la connexion réseau entre les connecteurs, y compris :

- **Le service en ligne** : les connexions à latence faible ou élevée au service du proxy d’application d’Azure influencent les performances du connecteur. Pour des performances optimales, connectez votre organisation à Azure avec Express Route. Sinon, assurez-vous que l’équipe réseau garantit que les connexions à Azure sont gérées de la façon la plus efficace.
- **Applications principales** : dans certains cas, il existe des proxys supplémentaires entre le connecteur et les applications principales capables de ralentir ou d’empêcher des connexions. Pour résoudre les problèmes de ce scénario, ouvrez un navigateur à partir du serveur du connecteur et essayez d’accéder à l’application. Si vous exécutez les connecteurs dans Azure alors que les applications sont locales, l’expérience peut ne pas être celle que vos utilisateurs attendent.
- **Contrôleurs de domaine** : Si les connecteurs assurent l’authentification unique (SSO) à l’aide de la délégation Kerberos contrainte, ils contactent les contrôleurs de domaine avant d’envoyer la demande au serveur principal. Les connecteurs ont un cache de tickets Kerberos, mais dans un environnement occupé, la réactivité des contrôleurs de domaine peut affecter les performances. Ce problème est plus courant pour les connecteurs qui s’exécutent dans Azure mais qui communiques avec les contrôleurs de domaine locaux.

Pour plus d’informations sur l’optimisation de votre réseau, consultez [Considérations sur la topologie du réseau lors de l’utilisation du proxy d’application Azure Active Directory](application-proxy-network-topology.md).

## <a name="domain-joining"></a>Jonction de domaine

Les connecteurs peuvent s’exécuter sur une machine qui n’est pas jointe à un domaine. Toutefois, si vous souhaitez utiliser l’authentification unique (SSO) pour les applications qui utilisent l’authentification Windows intégrée (IWA), vous avez besoin d’un ordinateur joint à un domaine. Dans ce cas, les ordinateurs du connecteur doivent être joints à un domaine qui peut effectuer la délégation [Kerberos](https://web.mit.edu/kerberos) contrainte pour le compte des utilisateurs pour les applications publiées.

Les connecteurs peuvent également être joints à des domaines ou forêts qui disposent d’une approbation partielle, ou à des contrôleurs de domaine en lecture seule.

## <a name="connector-deployments-on-hardened-environments"></a>Déploiements des connecteurs sur les environnements de sécurité renforcés

Généralement, le déploiement de connecteurs est simple et ne nécessite aucune configuration spéciale. Mais certaines conditions uniques doivent être prises en compte :

- Les organisations qui limitent le trafic sortant doivent [ouvrir les ports requis](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).
- Des machines conformes aux normes FIPS peuvent être requises pour modifier leur configuration afin d’autoriser les processus du connecteur à générer et stocker un certificat.
- Les organisations qui verrouillent leur environnement sur la base des processus qui émettent des requêtes réseau doivent s’assurer que les deux services de connecteur sont activés pour accéder à tous les ports et adresses IP nécessaires.
- Dans certains cas, les proxys de transfert sortants peuvent arrêter l’authentification de certificat bidirectionnelle et entraîner un échec de la communication.

## <a name="connector-authentication"></a>Authentification du connecteur

Afin de fournir un service sécurisé, les connecteurs doivent s’authentifier sur le service, et le service doit s’authentifier sur les connecteurs. L’authentification est effectuée à l’aide de certificats client et serveur lorsque les connecteurs établissent la connexion. De cette façon, le nom d’utilisateur et le mot de passe de l’administrateur ne sont pas stockés sur l’ordinateur du connecteur.

Les certificats utilisés sont spécifiques au service de proxy d’application. Ils sont créés lors de l’inscription initiale et sont renouvelés automatiquement par les connecteurs tous les quelques mois.

Après le premier renouvellement de certificat réussi, le service du connecteur de proxy d’application Azure AD (service réseau) n’a pas l’autorisation de supprimer l’ancien certificat du magasin de la machine locale. Si le certificat a expiré ou s’il n’est plus utilisé par le service, vous pouvez le supprimer sans problème.

Pour éviter les problèmes de renouvellement de certificat, vérifiez que la communication réseau du connecteur vers les [destinations documentées](./application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment) est activée.

Si un connecteur n’est pas connecté au service pendant plusieurs mois, ses certificats ont peut-être expiré. Dans ce cas, désinstallez et réinstallez le connecteur pour déclencher l’inscription. Vous pouvez exécuter les commandes PowerShell suivantes :

```
Import-module AppProxyPSModule
Register-AppProxyConnector -EnvironmentName "AzureCloud"
```

Pour les administrations, utilisez `-EnvironmentName "AzureUSGovernment"`. Pour plus de détails, consultez [Installer l’agent pour le cloud Azure Government](../hybrid/reference-connect-government-cloud.md#install-the-agent-for-the-azure-government-cloud).

Pour en savoir plus sur la façon de vérifier le certificat et de résoudre les problèmes, consultez [Vérifier la prise en charge du certificat de confiance du proxy d’application par la machine et les composants back-end](application-proxy-connector-installation-problem.md#verify-machine-and-backend-components-support-for-application-proxy-trust-certificate).

## <a name="under-the-hood"></a>Sous le capot

Comme les connecteurs sont basés sur le proxy d’application web Windows Server, la plupart ont les mêmes outils de gestion, y compris les journaux d’activité d’événements Windows

![Gérez les journaux d’événements avec l’Observateur d’événements](./media/application-proxy-connectors/event-view-window.png)

et les compteurs de performances Windows.

![Ajouter des compteurs au connecteur avec l’Analyseur de performances](./media/application-proxy-connectors/performance-monitor.png)

Les connecteurs ont des journaux de **session** et d’**administration**. Le journal d’**administration** inclut les événements principaux et leurs erreurs. Le journal de **session** contient toutes les transactions et les détails de leur traitement.

Pour voir les journaux, ouvrez l’**Observateur d’événements** et accédez à **Journaux des applications et des services** > **Microsoft** > **AadApplicationProxy** > **Connecteur**. Pour rendre le journal de **session** visible, dans le menu **Affichage**, sélectionnez **Afficher les journaux d’analyse et de débogage**. Le journal de **session** est généralement utilisé pour la résolution des problèmes. Il est désactivé par défaut. Activez-le pour commencer à collecter des événements, et désactivez-le lorsqu’il n’est plus nécessaire.

Vous pouvez examiner l’état du service dans la fenêtre Services. Le connecteur se compose de deux services Windows : le connecteur lui-même et le programme de mise à jour. Tous deux doivent s’exécuter en permanence.

 ![Exemple : Fenêtre Services indiquant des services locaux Azure AD](./media/application-proxy-connectors/aad-connector-services.png)

## <a name="next-steps"></a>Étapes suivantes

- [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md)
- [Travailler avec des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md)
- [Résoudre les erreurs du proxy d’application et du connecteur](application-proxy-troubleshoot.md)
- [Comment installer silencieusement le connecteur du Proxy d'application Azure AD](application-proxy-register-connector-powershell.md)