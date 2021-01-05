---
title: Sécuriser un cluster Azure Service Fabric
description: Découvrez des scénarios de sécurité relatifs aux clusters Azure Service Fabric, ainsi que les différentes technologies que vous pouvez utiliser pour les implémenter.
ms.topic: conceptual
ms.date: 08/14/2018
ms.openlocfilehash: 6f7bb785184938fe5c1e20e3c915b0112c7723ee
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573066"
---
# <a name="service-fabric-cluster-security-scenarios"></a>Scénarios de sécurité d’un cluster Service Fabric

Un cluster Azure Service Fabric est une ressource que vous possédez. Il vous incombe la responsabilité de sécuriser vos clusters pour empêcher les utilisateurs non autorisés de s’y connecter. La sécurisation des clusters est particulièrement importante lorsque vous exécutez des charges de travail de production sur le cluster. Il est possible de créer des clusters non sécurisés ; cependant, ils permettent aux utilisateurs anonymes de s’y connecter si les points de terminaison de gestion sont exposés sur l’Internet public. Les clusters non sécurisés ne sont pas gérés pour les charges de travail de production. 

Cet article présente des scénarios de sécurité relatifs aux clusters Azure et aux clusters autonomes, ainsi que les différentes technologies que vous pouvez utiliser pour implémenter ces scénarios :

* Sécurité nœud à nœud
* Sécurité client à nœud
* Contrôle d’accès en fonction du rôle Service Fabric

## <a name="node-to-node-security"></a>Sécurité nœud à nœud

La sécurité nœud à nœud aide à sécuriser la communication entre les machines virtuelles ou les ordinateurs d’un cluster. Ce scénario de sécurité garantit que seuls les ordinateurs qui sont autorisés à rejoindre le cluster peuvent participer à l’hébergement des applications et des services du cluster.

![Diagramme de communication nœud à nœud][Node-to-Node]

Les clusters exécutés dans Azure et les clusters autonomes exécutés sur Windows peuvent utiliser la [sécurité par certificat](/previous-versions/msp-n-p/ff649801(v=pandp.10)) ou la [sécurité Windows](/previous-versions/msp-n-p/ff649396(v=pandp.10)) pour les ordinateurs Windows Server.

### <a name="node-to-node-certificate-security"></a>Sécurité de certificat de nœud à nœud

Service Fabric utilise des certificats de serveur X.509 que vous spécifiez dans le cadre de la configuration du type de nœud, lorsque vous créez un cluster. La fin de cet article propose un rapide aperçu de ce que sont ces certificats et de la façon dont vous pouvez les acquérir ou les créer.

Pour configurer la sécurité par certificat lors de la création du cluster, vous pouvez utiliser un modèle Azure Resource Manager dans le portail Azure, ou un modèle JSON autonome. Le comportement par défaut du SDK Service Fabric est de déployer et d’installer le certificat dont la date d’expiration est la plus lointaine ; le comportement classique permettait de définir des certificats principaux et secondaires pour autoriser les déploiements manuels, mais cette approche n’est pas recommandée avec cette nouvelle fonctionnalité. Les certificats principaux qui seront utilisés présentant la date d’expiration la plus lointaine doivent être différents de ceux du client d’administration et du client en lecture seule que vous spécifiez pour la [sécurité client à nœud](#client-to-node-security).

Pour plus d’informations sur la configuration de la sécurité par certificat dans un cluster Azure, consultez [Configurer un cluster à l’aide d’un modèle Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).

Pour plus d’informations sur la configuration de la sécurité par certificat dans un cluster autonome Windows Server, consultez [Sécuriser un cluster autonome sur Windows à l’aide de certificats X.509](service-fabric-windows-cluster-x509-security.md).

### <a name="node-to-node-windows-security"></a>Sécurité Windows de nœud à nœud

> [!NOTE]
> L’authentification Windows est basée sur Kerberos. NTLM n’est pas pris en charge comme type d’authentification.
>
> Quand c’est possible, utilisez l’authentification par certificat X.509 pour les clusters Service Fabric.

Pour plus d’informations sur la configuration Windows dans un cluster autonome Windows Server, consultez [Sécuriser un cluster autonome sous Windows avec la sécurité Windows](service-fabric-windows-cluster-windows-security.md).

## <a name="client-to-node-security"></a>Sécurité client à nœud

La sécurité client à nœud authentifie les clients et sécurise la communication entre un client et les nœuds du cluster. Ce type de sécurité aide à garantir que seuls les utilisateurs autorisés peuvent accéder au cluster et aux applications déployées sur le cluster. Les clients sont identifiés de manière unique grâce à leurs informations d’identification de sécurité Windows ou à leurs informations d’identification de sécurité de certificat.

![Diagramme de communication client à nœud][Client-to-Node]

Les clusters qui s’exécutent sur Azure et les clusters autonomes qui s’exécutent sur Windows peuvent utiliser la [sécurité par certificat](/previous-versions/msp-n-p/ff649801(v=pandp.10)) ou la [sécurité Windows](/previous-versions/msp-n-p/ff649396(v=pandp.10)), bien que la recommandation soit d’utiliser un certificat d’authentification X.509 quand c’est possible.

### <a name="client-to-node-certificate-security"></a>Sécurité par certificat de client à nœud

Pour configurer la sécurité par certificat de type « client à nœud » lors de la création du cluster, vous pouvez utiliser un modèle Azure Resource Manager dans le portail Azure, ou un modèle JSON autonome. Pour créer le certificat, spécifiez un certificat client d’administration ou un certificat client utilisateur. Selon les bonnes pratiques, les certificats client d’administration et client utilisateur que vous spécifiez doivent être différents des certificats principaux et secondaires que vous spécifiez pour la [sécurité de nœud à nœud](#node-to-node-security). Les certificats de cluster ont les mêmes droits que les certificats d’administrateur client. Toutefois, il est conseillé pour des raisons de sécurité que seul le cluster les utilise, et non les utilisateurs administratifs.

Les clients se connectant au cluster avec le certificat d’administration ont un accès complet aux fonctions de gestion. Les clients se connectant au cluster avec le certificat client utilisateur en lecture seule ont uniquement un accès en lecture aux fonctions de gestion. Ces certificats sont utilisés pour le contrôle d’accès en fonction du rôle (RBAC) Service Fabric, qui est abordé plus loin dans cet article.

Pour plus d’informations sur la configuration de la sécurité par certificat dans un cluster Azure, consultez [Configurer un cluster à l’aide d’un modèle Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).

Pour plus d’informations sur la configuration de la sécurité par certificat dans un cluster autonome Windows Server, consultez [Sécuriser un cluster autonome sur Windows à l’aide de certificats X.509](service-fabric-windows-cluster-x509-security.md).

### <a name="client-to-node-azure-active-directory-security-on-azure"></a>Sécurité Azure Active Directory de client à nœud sur Azure

Azure AD permet aux organisation (appelées locataires) de gérer l’accès utilisateur aux applications. Ces dernières se composent d’applications avec une interface utilisateur de connexion web et d’applications avec une expérience client natif. Si vous n’avez pas encore créé de locataire, commencez par lire la section [Obtention d’un locataire Azure Active Directory][active-directory-howto-tenant].

Un cluster Service Fabric offre plusieurs points d’entrée pour ses fonctionnalités de gestion, notamment les outils web [Service Fabric Explorer][service-fabric-visualizing-your-cluster]et [Visual Studio][service-fabric-manage-application-in-visual-studio]. Par conséquent, vous allez créer deux applications Azure AD pour contrôler l’accès au cluster : une application web et une application native.

Pour les clusters exécutés dans Azure, vous pouvez également sécuriser l’accès aux points de terminaison de gestion à l’aide d’Azure Active Directory (Azure AD). Pour apprendre à créer les artefacts Azure AD requis et à les renseigner lors de la création du cluster, consultez la section [Configurer Azure AD pour authentifier les clients](service-fabric-cluster-creation-setup-aad.md).

## <a name="security-recommendations"></a>Recommandations de sécurité

Pour les clusters Service Fabric déployés dans un réseau public hébergé dans Azure, les recommandations concernant l’authentification mutuelle client à nœud sont les suivantes :

* Utiliser Azure Active Directory pour l’identité client
* Certificat pour l’identité du serveur et le chiffrement TLS de la communication HTTP

Pour les clusters Service Fabric déployés dans un réseau public hébergé dans Azure, les recommandations concernant la sécurité de nœud à nœud incitent à utiliser un certificat de cluster pour authentifier les nœuds.

Pour les clusters Windows Server autonomes, nous vous recommandons d’utiliser la sécurité Windows avec des comptes de service gérés de groupe si vous disposez de Windows Server 2012 R2 et d’Active Directory. Sinon, continuez à utiliser la sécurité Windows avec les comptes Windows.

## <a name="service-fabric-role-based-access-control"></a>Contrôle d’accès en fonction du rôle Service Fabric

Vous pouvez utiliser le contrôle d’accès pour limiter l’accès à certaines opérations de cluster pour différents groupes d’utilisateurs. Ainsi, vous rendez le cluster plus sécurisé. Deux types de contrôle d’accès sont pris en charge pour les clients qui se connectent à un cluster : le rôle Administrateur et le rôle Utilisateur.

Les utilisateurs qui reçoivent le rôle Administrateur ont un accès complet aux fonctionnalités de gestion (y compris les fonctionnalités de lecture/écriture). Les utilisateurs qui reçoivent le rôle Utilisateur ne disposent, par défaut, que d’un accès en lecture aux fonctionnalités de gestion (par exemple, aux fonctionnalités de requête). Ils peuvent également résoudre des applications et des services.

Configurez les rôles clients Administrateur et Utilisateur lorsque vous créez le cluster. Attribuez des rôles en fournissant des identités distinctes (par exemple, en utilisant des certificats ou Azure AD) pour chaque type de rôle. Pour plus d’informations sur les paramètres de contrôle d’accès par défaut et sur la modification des paramètres par défaut, consultez [Contrôle d’accès en fonction du rôle Service Fabric pour les clients Service Fabric](service-fabric-cluster-security-roles.md).

## <a name="x509-certificates-and-service-fabric"></a>Certificats X.509 et Service Fabric

Les certificats numériques X.509 sont couramment utilisés pour authentifier les clients et les serveurs. Ils sont également utilisés pour chiffrer et signer numériquement les messages. Service Fabric utilise des certificats X.509 pour sécuriser un cluster et fournir des fonctionnalités de sécurité d’applications. Pour plus d’informations sur les certificats numériques X.509, consultez [Utilisation des certificats](/dotnet/framework/wcf/feature-details/working-with-certificates). [Key Vault](../key-vault/general/overview.md) sert à gérer des certificats pour des clusters Service Fabric dans Azure.

Quelques éléments importants à prendre en compte :

* Si vous voulez créer des certificats pour des clusters qui exécutent des charges de travail de production, utilisez un service de certificats Windows Server correctement configuré ou obtenu auprès d’une [autorité de certification](https://en.wikipedia.org/wiki/Certificate_authority) approuvée.
* N’utilisez jamais de certificats temporaires ou de certificats de test créés à l’aide d’outils tels que MakeCert.exe dans un environnement de production.
* Vous pouvez utiliser un certificat auto-signé, mais uniquement dans un cluster de test. N’utilisez pas un certificat auto-signé dans un environnement de production.
* Au moment de la génération de l’empreinte de certificat, veillez à générer une empreinte numérique SHA1. Le hachage SHA1 est celui utilisé lors de la configuration des empreintes de certificat Client et Cluster.

### <a name="cluster-and-server-certificate-required"></a>Certificat de cluster et de serveur (obligatoire)

Ces certificats (un principal et éventuellement un secondaire) sont requis pour sécuriser un cluster et empêcher l’accès non autorisé à celui-ci. Ces certificats fournissent l’authentification du cluster et du serveur.

L’authentification du cluster authentifie la communication nœud à nœud pour la fédération du cluster. Seuls les nœuds qui peuvent prouver leur identité avec ce certificat peuvent être ajoutés au cluster. L’authentification du serveur authentifie les points de terminaison de gestion du cluster sur un client de gestion, afin que le client de gestion sache qu’il communique avec le véritable cluster et pas avec un « intercepteur ». Ce certificat fournit également un protocole TLS pour l’API de gestion HTTPS et Service Fabric Explorer par le biais de HTTPS. Lorsqu’un nœud ou un client authentifie un nœud, l’une des premières choses à faire est de vérifier la valeur du nom commun du champ **Objet**. Ce nom commun ou l’un des autres noms d’objet (SAN) du certificat doit figurer dans la liste des noms communs autorisés.

Le certificat doit répondre aux exigences suivantes :

* Le certificat doit contenir une clé privée. Ces certificats ont généralement les extensions .pfx ou .pem  
* Le certificat doit être créé pour l’échange de clés, qui peut faire l’objet d’un export vers un fichier .pfx (Personal Information Exchange).
* Le **nom d’objet du certificat doit correspondre au domaine utilisé pour accéder au cluster Service Fabric**. Cela est nécessaire pour la fourniture d’un protocole TLS pour le point de terminaison de gestion HTTPS du cluster et pour Service Fabric Explorer. Vous ne pouvez pas obtenir de certificat TLS/SSL auprès d’une autorité de certification (AC) pour le domaine *.cloudapp.azure.com. Vous devez obtenir un nom de domaine personnalisé pour votre cluster. Lorsque vous demandez un certificat auprès d’une autorité de certification, le nom de sujet du certificat doit correspondre au nom de domaine personnalisé utilisé pour votre cluster.

Voici quelques autres points importants à prendre en compte :

* Le champ **Objet** peut avoir plusieurs valeurs. Chaque valeur est préfixée par une initialisation pour indiquer le type valeur. En règle générale, l’initialisation est **CN** (pour *common name* (nom commun)), par exemple, **CN = www\.contoso.com**.
* Le champ **Objet** peut être vide.
* Si le champ facultatif **Autre nom de l’objet** est renseigné, il doit contenir à la fois le nom commun du certificat et une entrée par autre nom d’objet. Ces noms sont entrés sous forme de valeurs de **Nom DNS**. Pour savoir comment générer des certificats ayant des noms SAN, consultez [Comment ajouter un autre nom d’objet à un certificat LDAP sécurisé](https://support.microsoft.com/kb/931351).
* La valeur du champ **Rôles prévus** du certificat doit inclure une valeur appropriée, comme **Authentification serveur** ou **Authentification client**.

### <a name="application-certificates-optional"></a>Certificats d’application (facultatif)

Un nombre quelconque de certificats supplémentaires peut être installé sur un cluster pour sécuriser une application. Avant de créer votre cluster, examinez les scénarios de sécurité d’application qui nécessitent l’installation d’un certificat sur les nœuds, notamment :

* Le chiffrement et déchiffrement de valeurs de configuration d’application.
* Le chiffrement des données sur les nœuds lors de la réplication.

La méthode de création de clusters sécurisés est la même pour les clusters Linux et Windows.

### <a name="client-authentication-certificates-optional"></a>Certificats d’authentification client (facultatif)

Autant de certificats supplémentaires que vous souhaitez peuvent être spécifiés pour les opérations de client utilisateur ou administrateur. Le client peut utiliser ces certificats lorsque l’authentification mutuelle est nécessaire. Les certificats clients ne sont généralement pas émis par une autorité de certification tierce. Au lieu de cela, le magasin personnel de l’emplacement actuel de l’utilisateur contient généralement des certificats clients placés là par une autorité racine. Le certificat doit avoir la valeur **Rôles prévus** définie sur **Authentification du client**.  

Par défaut, le certificat de cluster a des privilèges de client administrateur. Ces certificats client supplémentaires ne doivent pas être installés dans le cluster, mais sont spécifiés comme étant autorisés dans la configuration du cluster.  Toutefois, les certificats client doivent être installés sur les ordinateurs clients pour se connecter au cluster et effectuer des opérations.

> [!NOTE]
> Toutes les opérations de gestion sur un cluster Service Fabric requièrent des certificats de serveur. Les certificats clients ne peuvent pas être utilisés pour la gestion.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un cluster dans Azure à l’aide d’un modèle Resource Manager](service-fabric-cluster-creation-via-arm.md)
* [Créer un cluster à l’aide du portail Azure](service-fabric-cluster-creation-via-portal.md)

<!--Image references-->
[Node-to-Node]: ./media/service-fabric-cluster-security/node-to-node.png
[Client-to-Node]: ./media/service-fabric-cluster-security/client-to-node.png

[active-directory-howto-tenant]:../active-directory/develop/quickstart-create-new-tenant.md
[service-fabric-visualizing-your-cluster]: service-fabric-visualizing-your-cluster.md
[service-fabric-manage-application-in-visual-studio]: service-fabric-manage-application-in-visual-studio.md
