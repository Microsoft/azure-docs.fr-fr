---
title: Authentification basée sur un certificat X.509 dans un cluster Service Fabric
description: Découvrez-en davantage sur l’authentification basée sur les certificats dans les clusters Service Fabric et sur la façon de détecter, atténuer et résoudre les problèmes liés aux certificats.
ms.topic: conceptual
ms.date: 03/16/2020
ms.openlocfilehash: 4d81cb9d224bdc2e3002c621c86729df235e0d81
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574766"
---
# <a name="x509-certificate-based-authentication-in-service-fabric-clusters"></a>Authentification basée sur un certificat X.509 dans des clusters Service Fabric

Cet article vient compléter la présentation de [la sécurité des clusters Service Fabric](service-fabric-cluster-security.md) et décrit en détail l’authentification basée sur les certificats dans les clusters Service Fabric. Nous partons du principe que le lecteur est familiarisé avec les concepts fondamentaux en matière de sécurité, ainsi qu’avec les contrôles que Service Fabric expose pour contrôler la sécurité d’un cluster.  

Rubriques traitées sous ce titre :

* Bases de l’authentification par certificat
* Identités et leurs rôles respectifs
* Règles de configuration de certificat
* Résolution des problèmes et forum aux questions

## <a name="certificate-based-authentication-basics"></a>Bases de l’authentification par certificat
Pour rappel, en matière de sécurité, un certificat est un instrument destiné à lier des informations relatives à une entité (l’objet) à sa possession d’une paire de clés de chiffrement asymétriques, et constitue donc une construction centrale du chiffrement à clé publique. Les clés représentées par un certificat peuvent être utilisées pour protéger des données ou pour prouver l’identité des détenteurs de clé. Lorsqu’un certificat est utilisé conjointement avec un système d’infrastructure à clé publique (PKI), il peut représenter des caractéristiques supplémentaires de son objet, telles que la propriété d’un domaine Internet ou certains privilèges qui lui sont accordés par l’émetteur du certificat (appelé autorité de certification, ou CA). Une application courante des certificats est la prise en charge du protocole de chiffrement TLS (Transport Layer Security), qui permet de sécuriser les communications sur un réseau informatique. Plus précisément, le client et le serveur utilisent des certificats pour garantir la confidentialité et l’intégrité de leur communication, et pour effectuer une authentification mutuelle.

Dans Service Fabric, la couche fondamentale d’un cluster (Fédération) s’appuie également sur TLS (parmi d’autres protocoles) pour obtenir un réseau fiable et sécurisé de nœuds participants. Les connexions au cluster via les API clientes Service Fabric utilisent également TLS pour protéger le trafic, ainsi que pour établir les identités des tiers. Plus précisément, lorsqu’un certificat est utilisé pour l’authentification dans Service Fabric, il peut servir à prouver les revendications suivantes : a) le présentateur des informations d’identification de certificat est propriétaire de la clé privée du certificat b) le hachage SHA-1 du certificat (empreinte numérique) correspond à une déclaration incluse dans la définition de cluster, ou c) le nom commun sujet distinctif du certificat correspond à une déclaration incluse , et l’émetteur du certificat est connu ou approuvé.

Dans la liste ci-dessus, « b » est informellement appelé « thumbprint pinning (épinglage d’empreinte numérique) ». Dans ce cas, la déclaration fait référence à un certificat spécifique, et la force du schéma d’authentification repose sur le principe qu’il est impossible de falsifier un certificat qui produit la même valeur de hachage qu’un autre, tout en étant un objet valide et correctement construit à tous les autres égards. L’élément « c » représente une autre forme de déclaration d’un certificat, où la force du schéma repose sur la combinaison de l’objet du certificat et de l’autorité émettrice. Dans ce cas, la déclaration fait référence à une classe de certificats : deux certificats ayant les mêmes caractéristiques sont considérés comme équivalents. 

Les sections suivantes expliquent en détail comment le runtime Service Fabric utilise et valide les certificats pour garantir la sécurité du cluster.

## <a name="identities-and-their-respective-roles"></a>Identités et leurs rôles respectifs
Avant de plonger dans les détails de l’authentification ou de la sécurisation des canaux de communication, il est important de répertorier les acteurs participants et les rôles correspondants qu’ils jouent dans un cluster :
- Le runtime Service Fabric, appelé « système » : l’ensemble des services qui fournissent les abstractions et les fonctionnalités représentant le cluster. Quand nous faisons référence à la communication dans le cluster entre les instances du système, nous utilisons le terme « identité du cluster ». Lorsque nous faisons référence au cluster comme destinataire/cible du trafic à partir de l’extérieur du cluster, nous utilisons le terme « identité du serveur ».
- Les applications hébergées, appelées « applications » : code fourni par le propriétaire du cluster, qui est orchestré et exécuté dans le cluster
- Clients : entités autorisées à se connecter à et à exécuter des fonctionnalités dans un cluster, en fonction de la configuration du cluster. Nous faisons la distinction entre deux niveaux de privilèges : « utilisateur » et « administrateur », respectivement. Un client « utilisateur » est limité principalement aux opérations en lecture seule (mais pas toutes les fonctionnalités en lecture seule), tandis qu’un client « administrateur » dispose d’un accès illimité aux fonctionnalités du cluster. (Pour plus d’informations, reportez-vous aux [Rôles de sécurité dans un cluster Service Fabric](service-fabric-cluster-security-roles.md).)
- (Azure uniquement) Les services Service Fabric qui orchestrent et exposent des contrôles pour l’exploitation et la gestion des clusters Service Fabric sont appelés simplement « services ». Selon l’environnement, le « service » peut faire référence au fournisseur de ressources Azure Service Fabric ou à d’autres fournisseurs de ressources détenus et gérés par l’équipe de Service Fabric.

Dans un cluster sécurisé, chacun de ces rôles peut être configuré avec sa propre identité distincte, déclarée comme le jumelage d’un nom de rôle prédéfini et des informations d’identification correspondantes. Service Fabric prend en charge la déclaration des informations d’identification en tant que certificats ou principal de service basé sur un domaine. (Les identités basées sur Windows/Kerberos sont également prises en charge, mais elles n’entrent pas dans le cadre de cet article. Pour cela, consultez [Sécurité basée sur Windows dans les clusters Service Fabric](service-fabric-windows-cluster-windows-security.md).) Dans les clusters Azure, les rôles clients peuvent également être déclarés comme [identités Azure Active Directory](service-fabric-cluster-creation-setup-aad.md).

Comme indiqué ci-dessus, le runtime Service Fabric définit deux niveaux de privilège dans un cluster : « administrateur » et « utilisateur ». Un client administrateur et un composant « système » fonctionnent tous les deux avec les privilèges « administrateur », et ne peuvent donc pas être distingués l’un de l’autre. Lors de l’établissement d’une connexion dans/au cluster, un appelant authentifié reçoit du runtime Service Fabric un des deux rôles comme base pour l’autorisation suivante. Nous examinerons l’authentification en détail dans les sections suivantes.

## <a name="certificate-configuration-rules"></a>Règles de configuration de certificat
### <a name="validation-rules"></a>Règles de validation
Les paramètres de sécurité d’un cluster Service Fabric décrivent, en principe, les aspects suivants :
- Type d’authentification : il s’agit d’une caractéristique non modifiable définie au moment de la création du cluster. Les exemples de ces paramètres sont « ClusterCredentialType », « ServerCredentialType », et les valeurs autorisées sont « none », « x509 » et « windows ». Cet article se concentre sur l’authentification de type x509.
- Les règles de validation (authentification) : ces paramètres sont définis par le propriétaire du cluster et décrivent les informations d’identification qui doivent être acceptées pour un rôle donné. Les exemples seront examinés en détail immédiatement ci-dessous.
- Paramètres utilisés pour ajuster ou altérer le résultat de l’authentification : les exemples comprennent les indicateurs de restriction (ou d’annulation de restriction) de l’application des listes de révocation de certificats, etc.

> [!NOTE]
> Les exemples de configuration de cluster fournis ci-dessous sont des extraits du manifeste de cluster au format XML, dans le format le plus condensé qui prend directement en charge les fonctionnalités de Service Fabric décrites dans cet article. Les mêmes paramètres peuvent être exprimés directement dans les représentations JSON d’une définition de cluster, qu’il s’agisse d’un manifeste de cluster JSON autonome ou d’un modèle Azure Resource Manager.

Une règle de validation de certificat comprend les éléments suivants :
- Le rôle correspondant : client, client d’administration (rôle privilégié)
- Informations d’identification acceptées pour le rôle, déclarées par empreinte numérique ou nom commun sujet

#### <a name="thumbprint-based-certificate-validation-declarations"></a>Déclarations de validation de certificat basées sur l’empreinte numérique
Dans le cas des règles de validation basées sur les empreintes numériques, les informations d’identification présentées par un appelant demandant une connexion dans/au cluster sont validées comme suit :
  - Les informations d’identification sont un certificat valide et correctement formé : sa chaîne peut être générée, les signatures correspondent
  - Le certificat est dans les limites de validité temporelle (NotBefore < = now < NotAfter)
  - Le hachage SHA-1 du certificat correspond à la déclaration, par comparaison de chaînes ne respectant pas la casse, excluant tous les espaces blancs

Toutes les erreurs d’approbation rencontrées pendant la génération ou la validation de la chaîne seront supprimées pour les déclarations basées sur l’empreinte numérique, à l’exception des certificats arrivés à expiration, bien que des dispositions existent également pour ce cas. Plus précisément, les échecs liés de type état de révocation inconnu ou hors connexion, racine non approuvée, utilisation de clé non valide ou chaîne partielle sont considérés comme des erreurs récupérables. Dans ce cas, le principe est que le certificat est simplement une enveloppe pour une paire de clés. La sécurité repose sur le fait que le propriétaire du cluster a mis en place des mesures pour protéger la clé privée.

L’extrait suivant d’un manifeste de cluster illustre un ensemble de règles de validation basées sur les empreintes numériques :

```xml
<Section Name="Security">
  <Parameter Name="ClusterCredentialType" Value="X509" />
  <Parameter Name="ServerAuthCredentialType" Value="X509" />
  <Parameter Name="AdminClientCertThumbprints" Value="d5ec...4264" />
  <Parameter Name="ClientCertThumbprints" Value="7c8f...01b0" />
  <Parameter Name="ClusterCertThumbprints" Value="abcd...1234,ef01...5678" />
  <Parameter Name="ServerCertThumbprints" Value="ef01...5678" />
</Section>
```

Chacune des entrées ci-dessus fait référence à une identité spécifique comme décrit précédemment. Chaque entrée permet également de spécifier plusieurs valeurs, sous la forme d’une liste de chaînes séparées par des virgules. Dans cet exemple, en cas de validation réussie des informations d’identification entrantes, le présentateur d’un certificat avec l’empreinte numérique SHA-1 'd5ec...4264' recevra le rôle « administrateur ». À l’inverse, un appelant qui s’authentifie avec le certificat '7c8f...01b0' reçoit un rôle « utilisateur », limité principalement aux opérations en lecture seule. Un appelant entrant qui présente un certificat dont l’empreinte numérique est 'abcd...1234' ou 'ef01...5678' sera accepté en tant que nœud homologue dans le cluster. Enfin, un client se connectant à un point de terminaison de gestion du cluster s’attend à ce que l’empreinte numérique du certificat de serveur soit 'ef01...5678'. 

Comme nous l’avons vu précédemment, Service Fabric a des dispositions en place pour accepter les certificats arrivés à expiration. La raison de cela est que les certificats ont une durée de vie limitée et, lorsqu’ils sont déclarés par empreinte numérique (qui fait référence à une instance de certificat spécifique), autoriser un certificat à expirer entraînera l’échec de la connexion au cluster, ou un effondrement direct du cluster. Il n’est que trop facile d’oublier ou de négliger la rotation d’un certificat épinglé par empreinte numérique, et malheureusement la récupération à partir d’une telle situation est difficile.

À cette fin, le propriétaire du cluster peut déclarer explicitement que les certificats autosignés déclarés par empreinte numérique seront considérés comme valides, comme suit :

```xml
  <Section Name="Security">
    <Parameter Name="AcceptExpiredPinnedClusterCertificate" Value="true" />
  </Section>
```
Ce comportement ne s’étend pas aux certificats émis par l’autorité de certification. Si tel était le cas, un certificat révoqué, connu pour être compromis, peut devenir « valide » dès qu’il ne figure plus dans la liste de révocation de certificats de l’autorité de certification, ce qui présente un risque de sécurité. Avec les certificats autosignés, le propriétaire du cluster est considéré comme le seul responsable de la protection de la clé privée du certificat, ce qui n’est pas le cas avec les certificats émis par l’autorité de certification. Le propriétaire du cluster peut ne pas savoir comment ou quand son certificat a été déclaré compromis.

#### <a name="common-name-based-certificate-validation-declarations"></a>Déclarations de validation de certificat basées sur un nom commun
Les déclarations basées sur un nom commun prennent une des formes suivantes :
- Nom commun sujet (uniquement)
- Nom commun sujet avec épinglage de l’émetteur

Prenons tout d’abord un extrait d’un manifeste de cluster qui illustre les deux styles de déclaration :
```xml
    <Section Name="Security/ServerX509Names">
      <Parameter Name="server.demo.system.servicefabric.azure-int" Value="" />
    </Section>
    <Section Name="Security/ClusterX509Names">
      <Parameter Name="cluster.demo.system.servicefabric.azure-int" Value="1b45...844d,d7fe...26c8,3ac7...6960,96ea...fb5e" />
    </Section>
```
Les déclarations font référence respectivement aux identités de serveur et de cluster. Notez que les déclarations basées sur un nom commun ont leurs propres sections dans le manifeste de cluster, séparées de la section 'Security' standard. Dans les deux déclarations, le champ 'Name' représente le nom commun sujet distinctif du certificat et le champ 'Value' représente l’émetteur attendu, comme suit :

- Dans le premier cas, la déclaration indique que l’élément de nom commun du sujet distinctif du certificat de serveur est supposé correspondre à la chaîne « server.demo.system.servicefabric.azure-int ». Le champ 'Value' vide indique que la racine de la chaîne de certificats est approuvée sur le nœud/l’ordinateur où le certificat de serveur est en cours de validation. Sur Windows, cela signifie que le certificat peut être chaîné à l’un des certificats installés dans le magasin de l’autorité de certification racine approuvée.
- Dans le second cas, la déclaration indique que le présentateur d’un certificat est accepté comme nœud homologue dans le cluster si le nom commun du certificat correspond à la chaîne « cluster.demo.system.servicefabric.azure-int », *et* que l’empreinte numérique de l’émetteur direct du certificat correspond à l’une des entrées séparées par des virgules dans le champ 'Value'. (Ce type de règle est connu sous le nom de « nom commun avec épinglage de l’émetteur ».)

Dans les deux cas, la chaîne du certificat est générée et devrait être exempte d’erreurs. Autrement dit, les erreurs de révocation et les erreurs de chaîne partielle ou de durée d’approbation non valide sont considérées comme irrécupérables et la validation du certificat échoue. L’épinglage des émetteurs fait que l’état « racine non approuvée » sera considéré comme une erreur récupérable. Malgré les apparences, il s’agit d’une forme de validation plus stricte, car elle permet au propriétaire du cluster de contraindre l’ensemble des émetteurs autorisés/acceptés à leur propre infrastructure à clé publique.

Une fois la chaîne de certificats créée, elle est validée par rapport à une stratégie TLS/SSL standard avec l’objet déclaré comme nom distant. Un certificat est considéré comme une correspondance si son nom commun sujet ou l’un de ses autres noms sujets correspond à la déclaration de nom commun du manifeste de cluster. Les caractères génériques sont pris en charge dans ce cas, et la correspondance de chaîne ne respecte pas la casse.

(Nous devrions préciser que la séquence décrite ci-dessus peut être exécutée pour chaque type d’utilisation de clé déclaré par le certificat. Si le certificat spécifie l’utilisation de la clé d’authentification du client, la chaîne est d’abord générée et évaluée pour un rôle client. En cas de réussite, l’évaluation est terminée et la validation est réussie. Si le certificat n’a pas l’utilisation de l’authentification du client ou si la validation a échoué, le runtime Service Fabric crée et évalue la chaîne pour l’authentification du serveur.)

Pour compléter l’exemple, l’extrait suivant illustre la déclaration de certificats clients par nom commun :
```xml
    <Section Name="Security/AdminClientX509Names">
      <Parameter Name="admin.demo.client.servicefabric.azure-int" Value="1b45...844d,d7fe...26c8,3ac7...6960,96ea...fb5e" />
    </Section>
    <Section Name="Security/ClientX509Names">
      <Parameter Name="user.demo.client.servicefabric.azure-int" Value="1b45...844d,d7fe...26c8,3ac7...6960,96ea...fb5e" />
    </Section>
```

Les déclarations ci-dessus correspondent respectivement aux identités d’administrateur et d’utilisateur. La validation des certificats déclarés de cette manière est exactement telle que décrite pour les exemples précédents de certificats de cluster et de serveur.

> [!NOTE]
> Les déclarations basées sur un nom commun sont destinées à simplifier la rotation et, en général, la gestion des certificats de cluster. Toutefois, il est recommandé de respecter les recommandations suivantes pour garantir la disponibilité et la sécurité continues du cluster :
  * Préférer l’épinglage de l’émetteur à l’utilisation des racines de confiance
  * Éviter de mélanger les émetteurs de différentes infrastructures à clé publique
  * Assurez-vous que tous les émetteurs attendus sont répertoriés dans la déclaration de certificat. Un émetteur incompatible entraînera un échec de validation
  * Assurez-vous que les points de terminaison de stratégie de certificat de l’infrastructure à clé publique sont détectables, disponibles et accessibles. Cela signifie que les points de terminaison AIA, CRL ou OCSP sont déclarés sur le certificat feuille, et qu’ils sont accessibles afin que la création de la chaîne de certificats puisse se terminer.

En liant tout cela, lors de la réception d’une demande de connexion dans un cluster sécurisé avec des certificats X.509, le runtime Service Fabric utilise les paramètres de sécurité du cluster pour valider les informations d’identification du tiers distant, comme décrit ci-dessus. En cas de réussite, l’appelant/le tiers distant est considéré comme authentifié. Si les informations d’identification correspondent à plusieurs règles de validation, le runtime accorde à l’appelant le rôle doté des privilèges les plus élevés parmi les règles correspondantes. 

### <a name="presentation-rules"></a>Règles de présentation
La section précédente a décrit le fonctionnement de l’authentification dans un cluster sécurisé par certificat. Cette section explique comment le runtime de Service Fabric détecte et charge les certificats qu’il utilise pour la communication dans le cluster. Nous les appelons règles de « présentation ».

Comme avec les règles de validation, les règles de présentation spécifient un rôle et la déclaration d’informations d’identification associée, sous forme d’empreinte numérique ou de nom commun. Contrairement aux règles de validation, les déclarations basées sur le nom commun ne comportent pas de dispositions pour l’épinglage de l’émetteur. Cela permet une plus grande flexibilité, ainsi qu’une amélioration des performances. Les règles de présentation sont déclarées dans la ou les sections 'NodeType' du manifeste de cluster, pour chaque type de nœud distinct. Les paramètres sont répartis à partir des sections Sécurité du cluster pour permettre à chaque type de nœud d’avoir sa configuration complète dans une section unique. Dans les clusters Azure Service Fabric, les déclarations de certificat de type de nœud sont définies par défaut sur les paramètres correspondants dans la section Sécurité de la définition du cluster.

#### <a name="thumbprint-based-certificate-presentation-declarations"></a>Déclarations de présentation de certificat basées sur l’empreinte numérique
Comme décrit précédemment, le runtime Service Fabric fait la distinction entre son rôle en tant que pair d’autres nœuds du cluster et son rôle de serveur pour les opérations de gestion du cluster. En principe, ces paramètres peuvent être configurés séparément, mais en pratique, ils ont tendance à être alignés. Pour le reste de cet article, nous allons supposer que les paramètres correspondent, à des fins de simplicité.

Prenons l’extrait suivant d’un manifeste de cluster :
```xml
  <NodeTypes>
    <NodeType Name="nt1vm">
      <Certificates>
        <ClusterCertificate X509FindType="FindByThumbprint" X509FindValue="cc71...1984" X509FindValueSecondary="49e2...19d6" X509StoreName="my" Name="ClusterCertificate" />
        <ServerCertificate X509FindValue="cc71...1984" Name="ServerCertificate" />
        <ClientCertificate X509FindValue="cc71...1984" Name="ClientCertificate" />
      </Certificates>
    </NodeType>
  </NodeTypes>
```
L’élément 'ClusterCertificate' illustre le schéma complet, y compris les paramètres facultatifs ('X509FindValueSecondary') ou ceux avec les valeurs par défaut appropriées ('X509StoreName'). Les autres déclarations affichent une forme abrégée. La déclaration de certificat de cluster ci-dessus indique que les paramètres de sécurité des nœuds de type 'nt1vm' sont initialisés avec le certificat principal 'cc71..1984' et le certificat secondaire '49e2..19d6'. Les deux certificats sont supposés se trouver dans le magasin de certificats LocalMachine\'My' (ou le chemin d’accès équivalent Linux, *var/lib/sfcerts*).

#### <a name="common-name-based-certificate-presentation-declarations"></a>Déclarations de présentation de certificat basées sur le nom commun
Les certificats de type de nœud peuvent également être déclarés par nom commun sujet, comme illustré ci-dessous :

```xml
  <NodeTypes>
    <NodeType Name="nt1vm">
      <Certificates>
        <ClusterCertificate X509FindType="FindBySubjectName" X509FindValue="demo.cluster.azuredocpr.system.servicefabric.azure-int" Name="ClusterCertificate" />
      </Certificates>
    </NodeType>
  </NodeTypes>
```

Pour les deux types de déclaration, un nœud Service Fabric lit la configuration au démarrage, localise et charge les certificats spécifiés, puis les trie dans l’ordre décroissant de leur attribut NotAfter. Les certificats arrivés à expiration sont ignorés, et le premier élément de la liste est sélectionné comme informations d’identification du client pour toute connexion Service Fabric tentée par ce nœud. (En fait, Service Fabric favorise le certificat arrivant à la date d’expiration la plus lointaine.)

Notez que, pour les déclarations de présentation basées sur un nom commun, un certificat est considéré comme une correspondance si son nom commun d’objet est égal au champ X509FindValue (ou X509FindValueSecondary) de la déclaration par comparaison exacte de chaînes respectant la casse. Cela diffère avec les règles de validation, qui prennent en charge la correspondance avec caractères génériques, ainsi que les comparaisons de chaînes ne respectant pas la casse.  

### <a name="miscellaneous-certificate-configuration-settings"></a>Divers paramètres de configuration de certificat
Il a été mentionné précédemment que les paramètres de sécurité d’un cluster Service Fabric permettaient également de modifier légèrement le comportement du code d’authentification. Si l’article sur les [paramètres de cluster Service Fabric](service-fabric-cluster-fabric-settings.md) représente la liste complète et la plus récente des paramètres, nous allons développer la signification d’un certain nombre de paramètres de sécurité ici, afin de compléter l’exposition de l’authentification basée sur les certificats. Pour chaque paramètre, nous expliquerons l’intention, la valeur/le comportement par défaut, la manière dont il affecte l’authentification, et les valeurs qui sont acceptables.

Comme indiqué précédemment, la validation de certificat implique toujours la génération et l’évaluation de la chaîne du certificat. Pour les certificats émis par une autorité de certification, cet appel d’API de système d’exploitation apparemment simple implique généralement plusieurs appels sortants à différents points de terminaison de l’infrastructure à clé publique (PKI) émettrice, la mise en cache des réponses, etc. Compte tenu de la prévalence des appels de validation de certificat dans un cluster Service Fabric, tout problème dans les points de terminaison de l’infrastructure à clé publique peut entraîner une réduction de la disponibilité du cluster, voire un effondrement direct. Si les appels sortants ne peuvent pas être supprimés (voir ci-dessous dans la section FAQ pour plus d’informations sur ce point), vous pouvez utiliser les paramètres suivants pour masquer les erreurs de validation provoquées par les appels infructueux à la liste de révocation de certificats.

  * CrlCheckingFlag : sous la section 'Security', une chaîne convertie en UINT. La valeur de ce paramètre est utilisée par Service Fabric pour masquer les erreurs d’état de la chaîne de certificats en modifiant le comportement de la création de chaînes. Elle est transmise à l’appel Win32 CryptoAPI [CertGetCertificateChain](/windows/win32/api/wincrypt/nf-wincrypt-certgetcertificatechain) en tant que paramètre 'dwFlags' et peut être définie sur n’importe quelle combinaison valide d’indicateurs acceptés par la fonction. La valeur 0 force le runtime Service Fabric à ignorer toutes les erreurs d’état de confiance, ce qui n’est pas recommandé, car son utilisation représente une exposition significative de sécurité. La valeur par défaut est 0x40000000 (CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT).

  Quand l’utiliser : pour les tests locaux, avec des certificats autosignés ou des certificats de développeur qui ne sont pas entièrement formés/ne disposent pas d’une infrastructure de clé publique appropriée pour prendre en charge les certificats. Peut également être utilisé comme atténuation dans les environnements en « air gap » pendant la transition entre les infrastructures à clé publique.

  Comment l’utiliser : nous allons prendre un exemple qui force la vérification de la révocation à accéder uniquement aux URL mises en cache. Nous supposons :
  ```C++
  #define CERT_CHAIN_REVOCATION_CHECK_CACHE_ONLY         0x80000000
  ```
  la déclaration dans le manifeste de cluster devient alors :
  ```xml
    <Section Name="Security">
      <Parameter Name="CrlCheckingFlag" Value="0x80000000" />
    </Section>
  ```

  * IgnoreCrlOfflineError : sous la section « Security », un booléen avec la valeur par défaut 'false'. Représente un raccourci pour supprimer un état d’erreur de création de chaîne ’revocation offline' (ou un état d’erreur de validation de stratégie de chaîne ultérieur).

  Quand l’utiliser : pour les tests locaux ou avec des certificats de développeur qui ne sont pas sauvegardés par une infrastructure à clé publique appropriée. À utiliser en tant qu’atténuation dans les environnements en « air gap » ou lorsque l’infrastructure à clé publique est connue pour être inaccessible.

  Utilisation :
  ```xml
    <Section Name="Security">
      <Parameter Name="IgnoreCrlOfflineError" Value="true" />
    </Section>
  ```

  Autres paramètres notables (tous dans la section 'Security') :
  * AcceptExpiredPinnedClusterCertificate : abordé dans la section dédiée à la validation des certificats basés sur l’empreinte numérique. Autorise l’acceptation des certificats de cluster autosignés arrivés à expiration. 
  * CertificateExpirySafetyMargin : intervalle, exprimé en minutes avant le timestamp NotAfter du certificat et au cours duquel le certificat est considéré comme étant à risque d’expiration. Service Fabric surveille le ou les certificats de cluster et émet régulièrement des rapports d’intégrité sur leur disponibilité restante. Pendant l’intervalle de « sécurité », les rapports d’intégrité sont élevés à l’état « avertissement ». La valeur par défaut est de 30 jours.
  * CertificateHealthReportingInterval : contrôle la fréquence des rapports d’intégrité relatifs au temps de validité restant des certificats de cluster. Les rapports ne seront émis qu’une seule fois par intervalle. La valeur est exprimée en secondes, avec une valeur par défaut de 8 heures.
  * EnforcePrevalidationOnSecurityChanges : booléen, contrôle le comportement de la mise à niveau de cluster lors de la détection des modifications des paramètres de sécurité. S’il est défini sur « true », la mise à niveau du cluster tente de s’assurer qu’au moins l’un des certificats répondant à l’une des règles de présentation peut transmettre une règle de validation correspondante. La validation préalable est exécutée avant que les nouveaux paramètres ne soient appliqués à un nœud, mais elle n’est exécutée que sur le nœud qui héberge le réplica principal du service Cluster Manager au démarrage de la mise à niveau. Au moment de la rédaction de cet article, le paramètre a la valeur par défaut « false », et sera défini sur « true » pour les nouveaux clusters Azure Service Fabric avec un runtime version 7.1 ou version ultérieure.
 
### <a name="end-to-end-scenario-examples"></a>Scénario de bout en bout (exemples)
Nous avons examiné les règles de présentation, les règles de validation et les indicateurs d’optimisation, mais comment le tout fonctionne-t-il ensemble ? Dans cette section, nous allons nous pencher sur deux exemples de bout en bout illustrant comment les paramètres de sécurité peuvent être exploités pour des mises à niveau de cluster sécurisées. Notez qu’il ne s’agit pas d’une étude complète sur la bonne gestion des certificats dans Service Fabric. Vous pouvez rechercher un article complémentaire sur ce sujet.

La séparation des règles de présentation et de validation pose la question (ou préoccupation) évidente de savoir si elles peuvent être divergentes et quelles sont les conséquences dans ce cas. Il est en effet possible que la sélection d’un certificat d’authentification d’un nœud ne réponde pas aux règles de validation d’un autre nœud. En fait, cette différence est la principale cause des incidents liés à l’authentification. Dans le même temps, la séparation de ces règles permet à un cluster de continuer à fonctionner pendant une mise à niveau qui modifie les paramètres de sécurité du cluster. Supposons que, en augmentant d’abord les règles de validation comme première étape, tous les nœuds du cluster convergeront sur les nouveaux paramètres tout en continuant à utiliser les informations d’identification actuelles. 

Souvenez-vous que, dans un cluster Service Fabric, une mise à niveau progresse à travers jusqu’à 5 « domaines de mise à niveau » ou UD. Seuls les nœuds de l’UD actuel sont mis à niveau/modifiés à un moment donné, et la mise à niveau se poursuit jusqu’au UD suivant uniquement si la disponibilité du cluster le permet. (Pour plus d’informations, consultez [Mises à niveau de clusters Service Fabric](service-fabric-cluster-upgrade.md) et d’autres articles à ce sujet.) Les modifications de certificat/sécurité sont particulièrement risquées, car elles peuvent isoler les nœuds du cluster ou laisser le cluster à la limite de la perte de quorum.

Nous allons utiliser la notation suivante pour décrire les paramètres de sécurité d’un nœud :

Nk: {P:{TP=A}, V:{TP=A}}, où :
  - 'NK' représente un nœud dans le domaine de mise à niveau *k*
  - 'P' représente les règles de présentation actuelles du nœud (en supposant que nous faisons référence aux certificats de cluster uniquement) ; 
  - 'V' représente les règles de validation actuelles du nœud (certificat de cluster uniquement)
  - 'TP=A' représente une déclaration basée sur une empreinte numérique, 'A' étant une empreinte numérique de certificat
  - 'CN = B' représente une déclaration basée sur le nom commun (CN), 'B' étant le nom commun sujet du certificat 

#### <a name="rotating-a-cluster-certificate-declared-by-thumbprint"></a>Rotation d’un certificat de cluster déclaré par empreinte numérique
La séquence suivante décrit comment une mise à niveau en 2 phases peut être utilisée pour introduire en toute sécurité un certificat de cluster secondaire, déclaré par empreinte numérique. La première phase introduit la nouvelle déclaration de certificat dans les règles de validation, et la deuxième phase l’introduit dans les règles de présentation :
  - état initial : N0 = {P:{TP=A}, V:{TP=A}}, ... Nk = {P:{TP=A}, V:{TP=A}} : le cluster est au repos, tous les nœuds partagent une configuration commune
  - après la mise à niveau du domaine 0 : N0 = {P:{TP=A}, V:{TP=A, TP=B}}, ... Nk = {P:{TP=A}, V:{TP=A}} : les nœuds de UD0 présentent le certificat A et acceptent les certificats A ou B. Tous les autres nœuds présentent et acceptent le certificat A uniquement
  - après la mise à niveau du dernier domaine : N0 = {P:{TP=A}, V:{TP=A, TP=B}}, ... Nk = {P:{TP=A}, V:{TP=A, TP=B}} : tous les nœuds présentent le certificat A, tous les nœuds acceptent le certificat A ou B
      
À ce stade, le cluster est à nouveau en équilibre et la deuxième phase de la mise à niveau/de changement des paramètres de sécurité peut commencer :
  - après la mise à niveau du domaine 0 : N0 = {P:{TP=A, TP=B}, V:{TP=A, TP=B}}, ... Nk = {P:{TP=A}, V:{TP=A, TP=B}} : les nœuds dans UD0 commencent à présenter B, qui est accepté par tous les autres nœuds du cluster.
  - après la mise à niveau du dernier domaine : N0 = {P:{TP=A, TP=B}, V:{TP=A, TP=B}}, ... Nk = {P:{TP=A, TP=B}, V:{TP=A, TP=B}} : tous les nœuds sont passés à la présentation du certificat B. Le certificat A peut désormais être retiré/supprimé de la définition du cluster avec un ensemble de mises à niveau ultérieur.

#### <a name="converting-a-cluster-from-thumbprint--to-common-name-based-certificate-declarations"></a>Conversion du type de déclaration d’un cluster d’empreinte numérique à nom commun
De même, la modification du type de déclaration de certificat (d’empreinte numérique à nom commun) suit le même modèle que ci-dessus. Notez que les règles de validation permettent la déclaration des certificats d’un rôle donné à la fois en fonction de l’empreinte et du nom commun dans la même définition de cluster. En revanche, les règles de présentation n’autorisent qu’une seule forme de déclaration. En fait, l’approche sûre de la conversion du type de déclaration d’un certificat de cluster d’empreinte numérique à nom commun consiste à introduire le certificat cible souhaité d’abord par empreinte numérique, puis à le remplacer par une autre déclaration basée sur un nom commun. Dans l’exemple suivant, nous supposons que l’empreinte numérique 'A' et le nom commun sujet 'B' font référence au même certificat. 

  - état initial : N0 = {P:{TP=A}, V:{TP=A}}, ... Nk = {P:{TP=A}, V:{TP=A}} : le cluster est au repos, tous les nœuds partagent une configuration commune, A étant l’empreinte numérique du certificat principal
  - après la mise à niveau du domaine 0 : N0 = {P:{TP=A}, V:{TP=A, CN=B}}, ... Nk = {P:{TP=A}, V:{TP=A}} : les nœuds de UD0 présentent le certificat A et acceptent les certificats avec l’empreinte numérique A ou le nom commun B. Tous les autres nœuds présentent et acceptent le certificat A uniquement
  - après la mise à niveau du dernier domaine : N0 = {P:{TP=A}, V:{TP=A, CN=B}}, ... Nk = {P:{TP=A}, V:{TP=A, CN=B}} : tous les nœuds présentent le certificat A, tous les nœuds acceptent le certificat A (TP) ou B (CN)

À ce stade, nous pouvons procéder à la modification des règles de présentation avec une autre mise à niveau :
  - après la mise à niveau du domaine 0 : N0 = {P:{CN=B}, V:{TP=A, CN=B}}, ... Nk = {P:{TP=A}, V:{TP=A, CN=B}} : les nœuds dans UD0 présenteront le certificat B trouvé par CN et accepteront les certificats avec l’empreinte A ou le nom commun B. Tous les autres nœuds présentent et acceptent le certificat A uniquement, sélectionné par empreinte numérique
  - après la mise à niveau du dernier domaine : N0 = {P:{CN=B}, V:{TP=A, CN=B}}, ... Nk = {P:{CN=B}, V:{TP=A, CN=B}} : tous les nœuds présentent le certificat B trouvé par CN, tous les nœuds acceptent le certificat A (TP) ou B (CN)
    
L’achèvement de la phase 2 marque également la conversion du cluster vers des certificats basés sur le nom commun. Les déclarations de validation basées sur l’empreinte numérique peuvent être supprimées lors d’une mise à niveau de cluster ultérieure.

> [!NOTE]
> Dans les clusters Azure Service Fabric, les flux de travail présentés ci-dessus sont orchestrés par le fournisseur de ressources Service Fabric. Le propriétaire du cluster est toujours responsable de la configuration des certificats dans le cluster en fonction des règles indiquées (présentation ou validation) et il est encouragé à effectuer des modifications en plusieurs étapes.

Dans un autre article, nous traiterons du sujet de la gestion et de l’approvisionnement des certificats dans un cluster Service Fabric.

## <a name="troubleshooting-and-frequently-asked-questions"></a>Résolution des problèmes et forum aux questions
Bien qu’il ne soit pas facile de déboguer les problèmes liés à l’authentification dans les clusters Service Fabric, nous espérons que les conseils suivants peuvent vous aider. Le moyen le plus simple de commencer des enquêtes consiste à examiner les journaux des événements Service Fabric sur les nœuds du cluster, pas nécessairement uniquement ceux qui présentent des symptômes, mais également les nœuds qui sont opérationnels mais ne peuvent pas se connecter à l’un de leurs voisins. Sur Windows, les événements notables sont généralement enregistrés sous les canaux 'Applications and Services Logs\Microsoft-ServiceFabric\Admin' ou 'Operational', respectivement. Parfois, il peut être utile [d’activer la journalisation CAPI2](/archive/blogs/benjaminperkins/enable-capi2-event-logging-to-troubleshoot-pki-and-ssl-certificate-issues), afin de capturer plus de détails sur la validation du certificat, la récupération des listes CRL/CTL, etc. (N’oubliez pas de la désactiver une fois la reproduction terminée, ce journal peut être très détaillé.)

Les symptômes typiques qui se manifestent dans un cluster rencontrant des problèmes d’authentification sont les suivants : 
  - les nœuds sont arrêtés/en boucle 
  - les tentatives de connexion sont rejetées
  - les tentatives de connexion expirent

Chacun des symptômes peut être dû à différents problèmes, et la même cause racine peut se manifester différemment. Par conséquent, nous allons simplement énumérer un petit échantillon de problèmes typiques, avec des recommandations pour les résoudre. 

* Les nœuds peuvent échanger des messages mais ne peuvent pas se connecter. Une cause possible d’arrêt de tentatives de connexion est l’erreur 'certificate not matched' : une des parties d’une connexion Service Fabric-à-Service Fabric présente un certificat qui ne répond pas aux règles de validation du destinataire. Cela peut s’accompagner d’une des erreurs suivantes : 
  ```C++
  0x80071c44    -2147017660 FABRIC_E_SERVER_AUTHENTICATION_FAILED
  ```
  Pour diagnostiquer/examiner plus en détail : sur chaque nœud qui tente la connexion, déterminez le certificat en cours de présentation. Examinez le certificat et essayez d’émuler les règles de validation (vérifiez l’empreinte numérique ou l’égalité des noms communs, vérifiez les empreintes numériques de l’émetteur si elles sont spécifiées).

  Un autre code d’erreur commun qui peut s’afficher est :
  ```C++
  0x800b0109    -2146762487 CERT_E_UNTRUSTEDROOT
  ```
  Dans ce cas, le certificat est déclaré par nom commun, et l’une ou l’autre des conditions suivantes s’applique :
    - les émetteurs ne sont pas épinglés et le certificat racine n’est pas approuvé, ou
    - les émetteurs sont épinglés, mais la déclaration n’inclut pas l’empreinte numérique de l’émetteur direct de ce certificat

* Un nœud est opérationnel, mais ne peut pas se connecter à d’autres nœuds. Les autres nœuds ne reçoivent pas le trafic entrant du nœud défaillant. Dans ce cas, il est possible que le chargement du certificat échoue sur le nœud local. Recherchez les erreurs suivantes :
  - certificate not found : vérifiez que les certificats déclarés dans les règles de présentation peuvent être résolus par le contenu du magasin de certificats LocalMachine\My (ou comme spécifié). 
    Les causes possibles de l’échec peuvent être les suivantes : 
      - caractères non valides dans la déclaration de l’empreinte numérique
      - le certificat n’est pas installé
      - le certificat a expiré
      - la déclaration de nom commun comprend le préfixe 'CN='
      - la déclaration spécifie un caractère générique et aucune correspondance exacte n’existe dans le magasin de certificats (déclaration : CN=*.mondomaine.com, certificat réel : CN=serveur.mondomaine.com)

  - unknown credentials : indique une clé privée manquante correspondant au certificat, généralement avec le code d’erreur : 
    ```C++ 
    0x8009030d  -2146893043 SEC_E_UNKNOWN_CREDENTIALS
    0x8009030e  -2146893042 SEC_E_NO_CREDENTIALS
    ```
    Pour résoudre ce problème, vérifiez l’existence de la clé privée. Vérifiez que SFAdmins dispose d’un accès 'read|execute' à la clé privée.

  - bad provider type : indique un certificat CNG (Crypto New Generation) (« Fournisseur de stockage de clés des logiciels Microsoft »). À ce stade, Service Fabric prend en charge uniquement les certificats CAPI1. Cela s’accompagne généralement du code d’erreur :
    ```C++
    0x80090014  -2146893804 NTE_BAD_PROV_TYPE
    ```
    Pour y remédier, recréez le certificat de cluster avec un CAPI1 (par ex. le fournisseur « Microsoft Enhanced RSA and AES Cryptographic Provider »). Pour plus d’informations sur les fournisseurs de chiffrement, consultez [Comprendre les fournisseurs de chiffrement](/windows/win32/seccertenroll/understanding-cryptographic-providers)
