---
title: Gestion des certificats dans un cluster Service Fabric
description: En savoir plus sur la gestion des certificats dans un cluster Service Fabric sécurisé avec des certificats X.509.
ms.topic: conceptual
ms.date: 04/10/2020
ms.custom: sfrev
ms.openlocfilehash: aba681157d71f94914462b8d9fc13b90d4d6b153
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88653662"
---
# <a name="certificate-management-in-service-fabric-clusters"></a>Gestion des certificats dans des clusters Service Fabric

Cet article traite des aspects liés à la gestion des certificats utilisés pour sécuriser les communications dans les clusters Azure Service Fabric, en complément de la présentation de la [sécurité d’un cluster Service Fabric](service-fabric-cluster-security.md) ainsi que de l’article explicatif [Authentification basée sur un certificat X.509 dans des clusters Service Fabric](cluster-security-certificates.md). Nous partons du principe que le lecteur est familiarisé avec les concepts fondamentaux en matière de sécurité, ainsi qu’avec les contrôles que Service Fabric propose pour configurer la sécurité d’un cluster.  

Aspects abordés sous ce titre :

* Définition de la gestion des certificats
* Rôles et entités impliqués dans la gestion des certificats
* Parcours d’un certificat
* Présentation d’un exemple approfondi
* Résolution des problèmes et forum aux questions

Mais, avant tout chose, il convient de souligner que cet article s’efforce de conjuguer une approche théorique avec des exemples pratiques qui nécessitent des services, technologies et autres ressources spécifiques. Étant donné qu’une partie importante de l’audience est composée de personnel interne de Microsoft, nous faisons référence aux services, technologies et produits spécifiques de Microsoft Azure. N’hésitez pas à demander des éclaircissements ou des conseils dans la section des commentaires lorsque des détails spécifiques de Microsoft ne s’appliquent pas dans votre cas.

## <a name="defining-certificate-management"></a>Définition de la gestion des certificats
Comme nous l’avons vu dans l’[article connexe](cluster-security-certificates.md), un certificat est un objet de chiffrement servant essentiellement à lier une paire de clés asymétriques avec des attributs décrivant l’entité qu’elle représente. Il s’agit cependant aussi d’un objet « périssable », dans la mesure où sa durée de vie est limitée et où il présente des vulnérabilités. En effet, une divulgation accidentelle ou une attaque réussie peuvent rendre un certificat inutile du point de vue de la sécurité. Cela implique la nécessité de changer les certificats soit systématiquement, soit en réponse à un incident de sécurité. Un autre aspect de la gestion (et c’est un sujet à part entière) est la sauvegarde des clés privées ou des secrets des certificats protégeant l’identité des entités impliquées dans l’obtention et l’approvisionnement des certificats. La gestion des certificats englobe les processus et procédures mis en œuvre pour obtenir des certificats et les délivrer sans encombre et en toute sécurité là où ils sont nécessaires. Certaines opérations de gestion, telles l’inscription, la définition de stratégies et les contrôles d’autorisation, sortent du cadre de cet article. D’autres, telles l’approvisionnement, le renouvellement, la régénération des clés ou la révocation, bien qu’elles ne soient liées qu’accessoirement à Service Fabric, sont néanmoins abordées ici dans une certaine mesure, car leur compréhension peut aider à sécuriser correctement les clusters. 

L’objectif est d’automatiser autant que possible la gestion des certificats pour garantir une disponibilité ininterrompue des clusters, et offrir des garanties de sécurité dans la mesure où le processus se déroule sans intervention de l’utilisateur. Cet objectif est actuellement atteignable dans des clusters Azure Service Fabric. Le reste de l’article commence par décortiquer la gestion des certificats avant de se concentrer sur l’activation de leur remplacement automatique.

Plus précisément, les sujets abordés sont les suivants :
  - postulats relatifs au cloisonnement des attributions entre propriétaire et plateforme dans le contexte de la gestion des certificats ;
  - long pipeline des certificats, de l’émission à l’utilisation ;
  - rotation des certificats, pourquoi, comment et quand ;
  - risques potentiels liés à la gestion des certificats.

Des aspects tels que la sécurisation et la gestion des noms de domaine, l’inscription dans des certificats ou la mise en place de contrôles d’autorisation pour l’émission des certificats ne sont pas abordés dans cet article. Nous vous engageons à vous tourner vers l’autorité d’inscription de votre service d’infrastructure à clé publique de prédilection. Les utilisateurs opérant au sein de Microsoft sont invités à contacter Azure Security Center.

## <a name="roles-and-entities-involved-in-certificate-management"></a>Rôles et entités impliqués dans la gestion des certificats
En vertu de l’approche de la sécurité dans un cluster Service Fabric, le propriétaire déclare et le runtime Service Fabric applique. Nous entendons par là que pratiquement aucun des certificats, clés ou autres informations d’identification des identités contribuant au fonctionnement d’un cluster ne proviennent du service proprement dit, mais que tous sont déclarés par le propriétaire du cluster. De plus, le propriétaire du cluster est également responsable de l’approvisionnement des certificats dans le cluster, de leur renouvellement en fonction des besoins, et de la garantie de leur sécurité à tout moment. Plus précisément, le propriétaire du cluster doit s’assurer que les conditions suivantes sont réunies :
  - les certificats déclarés dans la section NodeType du manifeste de cluster se trouvent sur chaque nœud de ce type, conformément aux [règles de présentation](cluster-security-certificates.md#presentation-rules) ;
  - les certificats déclarés ci-dessus sont installés, ainsi que leurs clés privées correspondantes ;
  - les certificats déclarés dans les règles de présentation doivent être conformes aux [règles de validation](cluster-security-certificates.md#validation-rules). 

Service Fabric, pour sa part, se charge des opérations suivantes :
  - localisation/recherche de certificats correspondant aux déclarations figurant dans la définition du cluster ;  
  - octroi aux entités contrôlées par Service Fabric de l’accès aux clés privées correspondantes en fonction des besoins ;
  - validation des certificats dans le strict respect des meilleures pratiques de sécurité établies et la définition du cluster ;
  - lancement d’alertes concernant l’expiration imminente des certificats ou l’échec d’exécution des étapes de base de validation des certificats ;
  - validation (dans une certaine mesure) du fait que la configuration sous-jacente des hôtes respecte les aspects liés au certificat de la définition du cluster. 

Il s’ensuit que la charge de gestion des certificats (en tant qu’opérations actives) incombe uniquement au propriétaire du cluster. Les sections suivantes examinent de plus près chacune des opérations de gestion, avec les mécanismes disponibles et leur impact sur le cluster.

## <a name="the-journey-of-a-certificate"></a>Parcours d’un certificat
Revenons rapidement sur la progression d’un certificat, de son émission à son utilisation, dans le contexte d’un cluster Service Fabric:

  1. Un propriétaire de domaine inscrit auprès de l’autorité d’inscription d’une infrastructure à clé publique un domaine ou objet qu’il souhaite associer à l’émission de certificats résultants. Ceux-ci constituent à leur tour des preuves de la propriété dudit domaine ou objet.
  2. Le propriétaire du domaine désigne également à l’autorité d’inscription les identités des demandeurs autorisés, qui sont habilitées à demander l’inscription de certificats avec le domaine ou l’objet spécifiés. Dans Microsoft Azure, le fournisseur d’identité par défaut est Azure Active Directory, et les demandeurs autorisés sont désignés par leur identité AAD correspondante (ou via des groupes de sécurité).
  3. Un demandeur autorisé s’inscrit ensuite dans un certificat via un service de gestion des secrets. Dans Microsoft Azure, il s’agit du service Azure Key Vault qui stocke les secrets/certificats en toute sécurité, et permet leur récupération par les entités autorisées. Azure Key Vault renouvelle/recrée également le certificat tel que configuré dans la stratégie de certificat associée (Azure Key Vault utilise AAD en tant que fournisseur d’identité).
  4. Un récupérateur autorisé, que nous appellerons « agent d’approvisionnement », récupère le certificat avec sa clé privée dans le coffre, et l’installe sur les machines hébergeant le cluster.
  5. La plateforme Service Fabric (s’exécutant avec élévation de privilèges sur chaque nœud) accorde l’accès au certificat aux entités Service Fabric autorisées. Celles-ci sont désignées par des groupes locaux et réparties entre ServiceFabricAdministrators et ServiceFabricAllowedUsers.
  6. Le runtime Service Fabric accède au certificat et l’utilise pour établir une fédération ou pour s’authentifier auprès de demandes entrantes de clients autorisés.
  7. L’agent d’approvisionnement surveille le certificat du coffre et déclenche le flux d’approvisionnement lors de la détection d’un renouvellement. Par la suite, le propriétaire du cluster met à jour la définition de celui-ci, si nécessaire, pour indiquer l’intention de reconduire le certificat.
  8. L’agent d’approvisionnement ou le propriétaire du cluster sont également responsables du nettoyage ou de la suppression des certificats inutilisés.
  
Pour nos besoins, les deux premières étapes de la séquence ci-dessus ne sont essentiellement indépendantes. Le seul lien entre elles est que le nom commun d’objet du certificat est le nom DNS déclaré dans la définition du cluster.

Ces étapes sont illustrées ci-dessous. Notez les différences d’approvisionnement entre les certificats déclarés respectivement par empreinte et par nom commun d’objet.

*Fig. 1.* Flux d’émission et d’approvisionnement de certificats déclarés par empreinte.
![Approvisionnement de certificats déclarés par empreinte][Image1]

*Fig. 2.* Flux d’émission et d’approvisionnement de certificats déclarés par nom commun d’objet.
![Approvisionnement de certificats déclarés par nom commun d’objet][Image2]

### <a name="certificate-enrollment"></a>Inscription de certificat
Ce sujet est traité en détail dans la [documentation sur Key Vault](../key-vault/certificates/create-certificate.md). Nous en incluons ici un résumé par souci de continuité et pour faciliter l’accès aux informations. En continuant dans le contexte d’Azure et en utilisant Azure Key Vault comme service de gestion des secrets, un demandeur de certificat autorisé doit disposer au moins d’autorisations de gestion des certificats sur le coffre, accordées par le propriétaire de celui-ci. Le demandeur doit alors s’inscrire dans un certificat comme suit :
    - Il crée une stratégie de certificat dans Azure Key Vault (AKV) qui spécifie le domaine ou l’objet du certificat, l’émetteur souhaité, le type et la longueur de la clé, l’utilisation prévue de la clé, etc. (pour plus de détails, voir [Certificats dans Azure Key Vault](../key-vault/certificates/certificate-scenarios.md)). 
    - Il crée un certificat dans le même coffre avec la stratégie spécifiée ci-dessus, ce qui a pour effet de générer une paire de clés en tant qu’objets de coffre, ainsi qu’une demande de signature de certificat signée avec la clé privée, qui est ensuite transmise à l’émetteur désigné pour signature.
    - Lorsque l’émetteur (autorité de certification) répond avec le certificat signé, le résultat est fusionné dans le coffre et le certificat est disponible pour les opérations suivantes :
      - sous {vaultUri}/certificates/{name}, le certificat comprenant la clé publique et les métadonnées
      - sous {vaultUri}/keys/{name}, la clé privée du certificat, disponible pour les opérations de chiffrement (envelopper/désenvelopper, signer/vérifier)
      - sous {vaultUri}/secrets/{Name}, le certificat avec sa clé privée, disponible pour le téléchargement en tant que fichier pfx ou pem non protégé  
    Rappelez-vous qu’un certificat de coffre est, en fait, une ligne chronologique d’instances de certificat partageant une stratégie. Les versions des certificats sont créées en fonction des attributs de durée de vie et de renouvellement de la stratégie. Il est fortement recommandé d’éviter que des certificats de coffre partagent des objets ou domaines/noms DNS. Dans un cluster, il peut être perturbant d’approvisionner des instances de certificat à partir de différents certificats de coffre avec des objets identiques mais d’autres attributs sensiblement différents, tels que l’émetteur, les utilisations de clés, etc.

À ce stade, il existe dans le coffre un certificat prêt pour utilisation. Ultérieurement :

### <a name="certificate-provisioning"></a>Approvisionnement du certificat
Nous avons évoqué un « agent d’approvisionnement », qui est l’entité qui récupère le certificat avec sa clé privée dans le coffre, et l’installe sur chacun des hôtes du cluster (n’oubliez pas que Service Fabric ne fournit pas de certificats). Dans notre contexte, le cluster sera hébergé sur une collection de machines virtuelles Azure ou dans des groupes de machines virtuelles identiques. Dans Azure, l’approvisionnement d’un certificat à partir d’un coffre vers une machine virtuelle ou un groupe de machines virtuelles identiques est possible grâce aux mécanismes ci-dessous, en supposant, comme ci-dessus, que l’agent d’approvisionnement ait préalablement reçu du propriétaire du coffre des autorisations « get » sur le celui-ci : 
  - Mécanisme ad hoc : un opérateur récupère le certificat dans le coffre (comme pfx/PKCS n°12 ou pem), et l’installe sur chaque nœud.
  - En tant que « secret » du groupe de machines virtuelles identiques pendant le déploiement. En utilisant son identité interne pour le compte de l’opérateur, le service de calcul récupère le certificat à partir d’un coffre activé pour le déploiement de modèle, et l’installe sur chaque nœud du groupe de machines virtuelles identiques ([comme décrit ici](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#certificates)). Notez que cela permet l’approvisionnement uniquement de secrets dont les versions sont gérées.
  - En utilisant l’[extension de machine virtuelle de Key Vault](../virtual-machines/extensions/key-vault-windows.md). Cela permet d’approvisionner les certificats à l’aide de déclarations sans version, avec un rafraîchissement périodique des certificats observés. Dans ce cas, la machine virtuelle ou le groupe de machines virtuelles identiques devraient avoir une [identité managée](../virtual-machines/security-policy.md#managed-identities-for-azure-resources), c’est-à-dire une identité autorisée à accéder aux coffres contenant les certificats observés.

Le mécanisme ad-hoc n’étant pas recommandé pour plusieurs raisons, allant de la sécurité à la disponibilité, nous n’en dirons pas davantage ici. Pour plus de détails, consultez [Certificats dans des groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#certificates).

Si l’approvisionnement basé sur groupe de machines virtuelles identiques/calcul présente des avantages en termes de sécurité et de disponibilité, il présente également des restrictions. En vertu de sa conception, il oblige à déclarer les certificats en tant que secrets dont les versions sont gérées. Il convient donc uniquement pour des clusters sécurisés avec des certificats déclarés par empreinte. A contrario, l’approvisionnement basé sur une extension de machine virtuelle de Key Vault installe toujours la dernière version de chaque certificat observé, de sorte qu’il ne convient que pour des clusters sécurisés avec des certificats déclarés par nom commun d’objet. Évitez absolument d’utiliser un mécanisme d’approvisionnement par actualisation automatique (comme l’extension de machine virtuelle de KeyVault) pour les certificats déclarés par une instance (c’est-à-dire, par empreinte). Cela vous exposerait à un risque de perte de disponibilité considérable.

Il existe peut-être d’autres mécanismes d’approvisionnement, mais ceux décrits ci-dessus sont actuellement acceptés pour les clusters Azure Service Fabric.

### <a name="certificate-consumption-and-monitoring"></a>Utilisation et surveillance des certificats
Comme mentionné précédemment, le runtime Service Fabric est responsable de la localisation et de l’utilisation des certificats déclarés dans la définition de cluster. L’article concernant l’[authentification basée sur les certificats](cluster-security-certificates.md) explique en détail comment Service Fabric implémente les règles de présentation et de validation. Nous n’y reviendrons pas ici. Nous allons examiner l’octroi d’accès et d’autorisation, ainsi que la surveillance.

Rappelez-vous que les certificats dans Service Fabric ont de nombreuses fonctions, de l’authentification mutuelle dans la couche de fédération à l’authentification TLS pour les points de terminaison de gestion. À cet effet, divers composants ou services système doivent avoir accès à la clé privée des certificats. Le runtime Service Fabric analyse régulièrement le magasin de certificats, à la recherche de correspondances pour chacune des règles de présentation connues. Pour chaque certificat épinglé, la clé privée correspondante est localisée et sa liste de contrôle d’accès discrétionnaire mise à jour pour inclure les autorisations, généralement en lecture et exécution, accordées à l’identité qui les requiert (ce processus est appelé officieusement « mise en liste ACL »). Le processus s’exécute à une cadence d’une fois par minute et couvre également les certificats « d’application », tels que ceux utilisés pour chiffrer des paramètres ou en tant que certificats de point de terminaison. La liste ACL suivant les règles de présentation, il est important de garder à l’esprit que les certificats déclarés par empreinte et actualisés automatiquement sans mise à jour de la configuration de cluster qui en résulte ne seront pas accessibles.    

### <a name="certificate-rotation"></a>Rotation des certificats
Remarque : La norme [RFC 3647](https://tools.ietf.org/html/rfc3647) de l’IETF définit formellement le concept de [renouvellement](https://tools.ietf.org/html/rfc3647#section-4.4.6) comme l’émission d’un certificat présentant les mêmes attributs que le certificat qu’il remplace (l’émetteur, la clé publique du sujet et ses informations sont préservés), et le concept de [régénération des clés](https://tools.ietf.org/html/rfc3647#section-4.4.7) comme l’émission d’un certificat avec une nouvelle paire de clés, et aucune restriction quant à la possibilité de changer l’émetteur. Étant donné que cette distinction peut avoir de l’importance (par exemple, dans le cas de certificats déclarés par nom commun de l’objet avec épinglage de l’émetteur), nous préférons utiliser le terme neutre de « rotation » afin de couvrir les deux scénarios (gardez à l’esprit qu’utilisé de manière informelle, le terme « renouvellement » désigne en réalité une régénération des clés). 

Nous avons vu précédemment qu’Azure Key Vault prend en charge la rotation automatique des certificats. La stratégie de certificat associée définit le moment (point dans le temps exprimé en jours avant l’expiration du certificat ou en pourcentage de la durée de vie totale de celui-ci) auquel a lieu la rotation du certificat dans le coffre. L’agent d’approvisionnement doit être appelé après ce point dans le temps et avant l’expiration du certificat désormais précédent, afin de distribuer ce nouveau certificat à tous les nœuds du cluster. Service Fabric vous aide en déclenchant un avertissement d’intégrité lorsque la date d’expiration d’un certificat (actuellement utilisé dans le cluster) est antérieure à la fin d’un intervalle prédéterminé. Un agent d’approvisionnement automatique (à savoir, l’extension de machine virtuelle de KeyVault) configuré pour observer le certificat du coffre interroge périodiquement celui-ci, détecte la rotation, puis récupère et installe le nouveau certificat. L’approvisionnement effectué via la fonction « secrets » de machine virtuelle/groupe de machines virtuelles identiques nécessite qu’un opérateur autorisé mette à jour l’instance de machine virtuelle/groupe de machines virtuelles identiques avec l’URI de KeyVault (version gérée) correspondant au nouveau certificat.

Dans les deux cas, le certificat après rotation est approvisionné sur tous les nœuds, et nous avons décrit le mécanisme que Service Fabric utilise pour détecter les rotations. Voyons ce qui se passe ensuite, en supposant que la rotation a été appliquée au certificat de cluster déclaré par le nom commun de l’objet (tout cela étant applicable au moment de la rédaction de ce document et dans le runtime Service Fabric 7.1.409) :
  - pour les nouvelles connexions au cluster et à l’intérieur de celui-ci, le runtime Service Fabric trouve et sélectionne le certificat correspondant dont la date d’expiration est la plus lointaine (propriété « NotAfter » du certificat, souvent abrégée en « na ») ;
  - les connexions existantes sont maintenues en vie/autorisées à expirer naturellement ou à prendre fin autrement (un gestionnaire interne aura été informé de l’existence d’une nouvelle correspondance).

Cela se traduit par les observations importantes suivantes :
  - Le certificat de renouvellement peut être ignoré si sa date d’expiration est antérieure à celle du certificat actuellement utilisé.
  - La disponibilité du cluster ou des applications hébergées prend le pas sur la directive de rotation du certificat. Le cluster finit par converger sur le nouveau certificat, mais sans garantie de timing. Résultat :
  - Il peut ne pas être immédiatement évident pour l’observateur que le certificat après rotation a complètement remplacé son prédécesseur. La seule façon de s’en assurer (pour les certificats de cluster) est de redémarrer les machines hôtes. Notez qu’il ne suffit pas de redémarrer les nœuds Service Fabric, car les composants en mode noyau qui forment des connexions de bail dans un cluster ne seront pas affectés. Notez également que le redémarrage de la machine virtuelle ou du groupe de machines virtuelles identiques peut entraîner une perte temporaire de disponibilité (pour des certificats d’application, il suffit de redémarrer les instances d’application respectives).
  - L’introduction d’un certificat avec clé régénérée qui ne respecte pas les règles de validation peut effectivement anéantir le cluster. L’exemple le plus courant de cela est le cas d’un émetteur inattendu : les certificats de cluster sont déclarés par nom commun de l’objet avec épinglage de l’émetteur, mais le certificat après rotation a été émis par un émetteur nouveau ou non déclaré.     

### <a name="certificate-cleanup"></a>Nettoyage de certificat
À l’heure actuelle, il n’existe aucune disposition dans Azure pour la suppression explicite des certificats. Il est souvent difficile de déterminer si un certificat donné est utilisé à un moment donné. Cela est plus difficile pour les certificats d’application que pour les certificats de cluster. Service Fabric n’étant pas l’agent d’approvisionnement, il ne supprime en aucun cas un certificat déclaré par l’utilisateur. Pour les mécanismes d’approvisionnement standard :
  - Les certificats déclarés en tant que secrets de machine virtuelle/groupe de machines virtuelles identiques sont approvisionnés aussi longtemps qu’ils sont référencés dans la définition de machine virtuelle/groupe de machines virtuelles identiques, et qu’ils peuvent être récupérés à partir du coffre (la suppression d’un secret/certificat de coffre entraîne l’échec des déploiements subséquents de machine virtuelle/groupe de machines virtuelles identiques ; de même, la désactivation d’une version de secret dans le coffre entraîne également l’échec des déploiements de machine virtuelle/groupe de machines virtuelles identiques faisant référence à cette version de secret).
  - Les versions précédentes des certificats approvisionnés via l’extension de machine virtuelle de KeyVault peuvent ou non être présentes sur un nœud de machine virtuelle/groupe de machines virtuelles identiques. L’agent ne récupère et installe que la version actuelle, et ne supprime aucun certificat. Le réimageage d’un nœud (qui se produit généralement chaque mois) a pour effet de réinitialiser le magasin de certificats avec le contenu de l’image de système d’exploitation, et implicitement de supprimer les versions précédentes. Sachez cependant que la mise à l’échelle vers le haut d’un groupe de machines virtuelles identiques a pour effet de n’installer que la version actuelle des certificats observés. Elle ne permet nullement de supposer que les nœuds sont homogènes en ce qui concerne les certificats installés.   

## <a name="simplifying-management---an-autorollover-example"></a>Simplification de la gestion : un exemple de substitution automatique
Nous avons décrit des mécanismes et restrictions, exposé des règles et définitions complexes et prédit des pannes calamiteuses. Il est peut-être temps de montrer comment configurer la gestion automatique des certificats pour éliminer tous ces soucis. Nous le faisons dans le contexte d’un cluster Azure Service Fabric s’exécutant sur un groupe de machines virtuelles identiques PaaSv2, en utilisant Azure Key Vault pour la gestion des secrets et en tirant parti des identités gérées comme suit :
  - La validation des certificats passe de l’épinglage d’empreinte à l’épinglage d’objet + émetteur : tout certificat avec un objet donné d’un émetteur donné est approuvé de la même manière.
    - Les certificats sont inscrits dans un magasin de confiance (Key Vault) et obtenus à partir de celui-ci, ainsi qu’actualisés par un agent, en l’occurrence, l’extension de machine virtuelle de KeyVault.
    - L’approvisionnement des certificats passe d’un système basé sur l’heure de déploiement et la version (comme le fait ComputeRP) à un post-déploiement avec utilisation d’URI de KeyVault sans gestion des versions.
    - L’accès au KeyVault est accordé via des identités gérées attribuées par l’utilisateur. L’identité de l’agent utilisateur est créée et attribuée au groupe de machines virtuelles identiques lors du déploiement.
    - Après le déploiement, l’agent (extension de machine virtuelle du KeyVault) interroge et actualise les certificats observés sur chaque nœud du groupe de machines virtuelles identiques. La rotation des certificats est ainsi entièrement automatisée, car Service Fabric récupère automatiquement le certificat valide dont la date d’expiration est la plus lointaine.

La séquence, qui peut être entièrement consignée dans un script et automatisée, permet un déploiement initial sans intervention de l’utilisateur d’un cluster configuré pour la substitution automatique de certificat. Des étapes détaillées sont fournies ci-dessous. Nous allons utiliser un mélange de cmdlets PowerShell et de fragments de modèles json. Il est possible d’obtenir la même fonctionnalité avec tous les moyens pris en charge d’interaction avec Azure.

[!NOTE] Cet exemple part du principe qu’il existe déjà un certificat dans le coffre. L’inscription et le renouvellement d’un certificat géré par KeyVault nécessitent l’exécution des étapes manuelles préalables décrites plus haut dans cet article. Pour les environnements de production, utilisez des certificats gérés par KeyVault. Vous trouverez ci-dessous un exemple de script spécifique d’une infrastructure de clé publique (PKI) interne de Microsoft.
La substitution automatique de certificat n’a de sens que pour des certificats émis par une autorité de certification. L’utilisation de certificats auto-signés, y compris de ceux générés lors du déploiement d’un cluster Service Fabric dans le portail Azure, est dénuée de sens, mais toujours possible, pour des déploiements locaux ou hébergés par un développeur, en déclarant que l’empreinte de l’émetteur est la même que celle du certificat feuille.

### <a name="starting-point"></a>Point de départ
Par souci de concision, nous partons du principe que l’état de départ est le suivant :
  - Le cluster Service Fabric existe et est sécurisé avec un certificat émis par une autorité de certification et déclaré par empreinte.
  - Le certificat est stocké dans un coffre et approvisionné en tant que secret de groupe de machines virtuelles identiques.
  - Le même certificat sera utilisé pour convertir le cluster en déclarations de certificat basées sur un nom commun. Il peut donc être validé par objet et émetteur. Si ce n’est pas le cas, procurez-vous le certificat émis par une autorité de certification prévu à cet effet, puis ajoutez-le à la définition du cluster par empreinte, comme expliqué [ici](service-fabric-cluster-security-update-certs-azure.md).

Voici un json extrait d’un modèle correspondant à un tel état. Notez qu’il omet de nombreux paramètres obligatoires et n’illustre que les aspects liés au certificat :
```json
  "resources": [
    {   ## VMSS definition
      "apiVersion": "[variables('vmssApiVersion')]",
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "name": "[variables('vmNodeTypeName')]",
      "location": "[variables('computeLocation')]",
      "properties": {
        "virtualMachineProfile": {
          "extensionProfile": {
            "extensions": [
            {
                "name": "[concat('ServiceFabricNodeVmExt','_vmNodeTypeName')]",
                "properties": {
                  "type": "ServiceFabricNode",
                  "autoUpgradeMinorVersion": true,
                  "publisher": "Microsoft.Azure.ServiceFabric",
                  "settings": {
                    "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                    "nodeTypeRef": "[variables('vmNodeTypeName')]",
                    "dataPath": "D:\\SvcFab",
                    "durabilityLevel": "Bronze",
                    "certificate": {
                        "thumbprint": "[parameters('primaryClusterCertificateTP')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },
                  "typeHandlerVersion": "1.1"
                }
            },}},
          "osProfile": {
            "adminPassword": "[parameters('adminPassword')]",
            "adminUsername": "[parameters('adminUsername')]",
            "secrets": [
            {
                "sourceVault": {
                    "id": "[resourceId('Microsoft.KeyVault/vaults', parameters('keyVaultName'))]"
                },
                "vaultCertificates": [
                {
                    "certificateStore": "[parameters('certificateStoreValue')]",
                    "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
                },
            ]}]
        },
    },
    {   ## cluster definition
        "apiVersion": "[variables('sfrpApiVersion')]",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "certificate": {
            "thumbprint": "[parameters('primaryClusterCertificateTP')]",
            "x509StoreName": "[parameters('certificateStoreValue')]"
        },
    }
  ]
```   

Ce qui précède indique essentiellement que le certificat avec empreinte ```json [parameters('primaryClusterCertificateTP')] ``` et qui se trouve à l’URI de KeyVault ```json [parameters('clusterCertificateUrlValue')] ``` est déclaré par empreinte comme l’unique certificat du cluster. Ensuite, nous allons configurer les ressources supplémentaires nécessaires pour garantir la substitution automatique du certificat.

### <a name="setting-up-prerequisite-resources"></a>Configuration des ressources préalables
Comme mentionné précédemment, un certificat approvisionné en tant que secret de groupe de machines virtuelles identiques est récupéré du coffre par le service Fournisseur de ressources de calcul Microsoft, en utilisant son identité de première partie et pour le compte de l’opérateur de déploiement. Pour la substitution automatique, cela changera. Nous passerons à l’utilisation d’une identité managée, attribuée au groupe de machines virtuelles identiques, qui est autorisée à accéder aux secrets du coffre.

Tous les extraits suivants doivent être déployés de manière concomitante. Ils sont répertoriés individuellement pour une analyse et des explications ad hoc.

Commencez par définir une identité attribuée par l’utilisateur (les valeurs par défaut sont incluses à titre d’exemples). Pour obtenir des informations à jour, consultez la [documentation officielle](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity) :
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "userAssignedIdentityName": {
      "type": "string",
      "defaultValue": "sftstuaicus",
      "metadata": {
        "description": "User-assigned managed identity name"
      }
    },
  },
  "variables": {
      "vmssApiVersion": "2018-06-01",
      "sfrpApiVersion": "2018-02-01",
      "miApiVersion": "2018-11-30",
      "kvApiVersion": "2018-02-14",
      "userAssignedIdentityResourceId": "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('userAssignedIdentityName'))]"
  },    
  "resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('userAssignedIdentityName')]",
      "apiVersion": "[variables('miApiVersion')]",
      "location": "[resourceGroup().location]"
    },
  ]}
```

Accordez ensuite à cette identité l’accès aux secrets du coffre. Pour obtenir des informations à jour, consultez la [documentation officielle](/rest/api/keyvault/vaults/updateaccesspolicy) :
```json
  "resources":
  [{
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "name": "[concat(parameters('keyVaultName'), '/add')]",
      "apiVersion": "[variables('kvApiVersion')]",
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[reference(variables('userAssignedIdentityResourceId'), variables('miApiVersion')).tenantId]",
            "objectId": "[reference(variables('userAssignedIdentityResourceId'), variables('miApiVersion')).principalId]",
            "dependsOn": [
              "[variables('userAssignedIdentityResourceId')]"
            ],
            "permissions": {
              "secrets": [
                "get",
                "list"
              ]}}]}}]
```

À l’étape suivante, nous allons :
  - attribuer l’identité affectée par l’utilisateur au groupe de machines virtuelles identiques ;
  - déclarer la dépendance du groupe de machines virtuelles identiques de la création de l’identité managée et du résultat de l’octroi de l’accès au coffre ;
  - déclarer l’extension de machine virtuelle de KeyVault, en exigeant qu’elle récupère les certificats observés au démarrage ([documentation officielle](../virtual-machines/extensions/key-vault-windows.md)) ;
  - mettre à jour la définition de l’extension de machine virtuelle de Service Fabric pour qu’elle dépende de l’extension de machine virtuelle de KeyVault et convertisse le certificat de cluster en nom commun (nous apportons ces modifications en une seule étape, car elles relèvent de la même ressource).

```json
  "parameters": {
    "kvvmextPollingInterval": {
      "type": "string",
      "defaultValue": "3600",
      "metadata": {
        "description": "kv vm extension polling interval in seconds"
      }
    },
    "kvvmextLocalStoreName": {
      "type": "string",
      "defaultValue": "MY",
      "metadata": {
        "description": "kv vm extension local store name"
      }
    },
    "kvvmextLocalStoreLocation": {
      "type": "string",
      "defaultValue": "LocalMachine",
      "metadata": {
        "description": "kv vm extension local store location"
      }
    },
    "kvvmextObservedCertificates": {
      "type": "array",
      "defaultValue": [
                "https://sftestcus.vault.azure.net/secrets/sftstcncluster",
                "https://sftestcus.vault.azure.net/secrets/sftstcnserver"
            ],
      "metadata": {
        "description": "kv vm extension observed certificates versionless uri"
      }
    },
    "certificateCommonName": {
      "type": "string",
      "defaultValue": "cus.cluster.sftstcn.system.servicefabric.azure-int",
      "metadata": {
        "description": "Certificate Common name"
      }
    },
  },
  "resources": [
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[variables('vmNodeTypeName')]",
    "location": "[variables('computeLocation')]",
    "dependsOn": [
      "[variables('userAssignedIdentityResourceId')]",
      "[concat('Microsoft.KeyVault/vaults/', concat(parameters('keyVaultName'), '/accessPolicies/add'))]"
    ],
    "identity": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
        "[variables('userAssignedIdentityResourceId')]": {}
      }
    },
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
        {
          "name": "KVVMExtension",
          "properties": {
            "publisher": "Microsoft.Azure.KeyVault",
            "type": "KeyVaultForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "secretsManagementSettings": {
                    "pollingIntervalInS": "[parameters('kvvmextPollingInterval')]",
                    "linkOnRenewal": false,
                    "observedCertificates": "[parameters('kvvmextObservedCertificates')]",
                    "requireInitialSync": true
                }
            }
          }
        },
        {
        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeTypeName')]",
        "properties": {
          "type": "ServiceFabricNode",
          "provisionAfterExtensions" : [ "KVVMExtension" ],
          "publisher": "Microsoft.Azure.ServiceFabric",
          "settings": {
            "certificate": {
                "commonNames": [
                    "[parameters('certificateCommonName')]"
                ],
                "x509StoreName": "[parameters('certificateStoreValue')]"
            }
            },
            "typeHandlerVersion": "1.0"
          }
        },
  ] } ## extension profile
  },  ## VM profile
  "osProfile": {
    "adminPassword": "[parameters('adminPassword')]",
    "adminUsername": "[parameters('adminUsername')]",
  } 
  }
  ]
```
Notez que, même si cela n’est pas explicitement mentionné ci-dessus, nous avons supprimé l’URL du certificat de coffre de la section « OsProfile » du groupe de machines virtuelles identiques.
La dernière étape consiste à mettre à jour la définition de cluster afin de modifier la déclaration de certificat d’empreinte en nom commun. Nous épinglons également ici les empreintes de l’émetteur :

```json
  "parameters": {
    "certificateCommonName": {
      "type": "string",
      "defaultValue": "cus.cluster.sftstcn.system.servicefabric.azure-int",
      "metadata": {
        "description": "Certificate Common name"
      }
    },
    "certificateIssuerThumbprint": {
      "type": "string",
      "defaultValue": "1b45ec255e0668375043ed5fe78a09ff1655844d,d7fe717b5ff3593764f4d90654d86e8362ec26c8,3ac7c3cac8de0dd392c02789c8be97474f456960,96ea05926e2e42cc207e358668be2c316857fb5e",
      "metadata": {
        "description": "Certificate issuer thumbprints separated by comma"
      }
    },
  },
  "resources": [
    {
      "apiVersion": "[variables('sfrpApiVersion')]",
      "type": "Microsoft.ServiceFabric/clusters",
      "name": "[parameters('clusterName')]",
      "location": "[parameters('clusterLocation')]",
      "properties": {
        "certificateCommonNames": {
          "commonNames": [{
              "certificateCommonName": "[parameters('certificateCommonName')]",
              "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
          }],
          "x509StoreName": "[parameters('certificateStoreValue')]"
        },
  ]
```

À ce stade, vous pouvez exécuter les mises à jour mentionnées ci-dessus dans un seul déploiement. De son côté, le service de fournisseur de ressources Service Fabric fractionnera la mise à niveau du cluster en plusieurs étapes, comme décrit dans le segment consacré à la [conversion des certificats de cluster d’empreinte en nom commun](cluster-security-certificates.md#converting-a-cluster-from-thumbprint--to-common-name-based-certificate-declarations).

### <a name="analysis-and-observations"></a>Analyse et observations
Cette section est un fourre-tout pour expliquer les étapes détaillées ci-dessus, ainsi que pour attirer l’attention sur des aspects importants.

#### <a name="certificate-provisioning-explained"></a>Explication de l’approvisionnement de certificats
L’extension de machine virtuelle de KeyVault, en tant qu’agent d’approvisionnement, s’exécute en continu à une fréquence prédéterminée. En cas d’échec de récupération d’un certificat observé, elle passe au certificat suivant dans la ligne, puis se met en veille prolongée jusqu’au cycle suivant. L’extension de machine virtuelle de Service Fabric, en tant qu’agent de démarrage du cluster, exige les certificats déclarés avant que le cluster puisse se former. Cela implique que l’extension de machine virtuelle de Service Fabric ne peut s’exécuter qu’après la récupération effective du ou des certificats de cluster, indiquée ici par la clause ```json "provisionAfterExtensions" : [ "KVVMExtension" ]"``` et par le paramètre ```json "requireInitialSync": true``` de l’extension de machine virtuelle de KeyVault. Cela indique à l’extension de machine virtuelle de KeyVault que, lors de la première exécution (après déploiement ou redémarrage), elle doit parcourir ses certificats observés jusqu’à ce que tous soient correctement téléchargés. La définition de ce paramètre sur false, associée à un échec de récupération du ou des certificats de cluster, entraînerait un échec de déploiement du cluster. Inversement, exiger une synchronisation initiale avec une liste incorrecte/non valide de certificats observés entraînerait un échec de l’extension de machine virtuelle de KeyVault, et donc, une fois de plus, un échec de déploiement du cluster.  

#### <a name="certificate-linking-explained"></a>Explication du lien de certificat
Vous avez peut-être remarqué l’indicateur « linkOnRenewal » de l’extension de machine virtuelle de KeyVault, et le fait qu’il est défini sur false. Nous abordons ici en profondeur le comportement contrôlé par cet indicateur et ses conséquences sur le fonctionnement d’un cluster. Notez que ce comportement est spécifique de Windows.

En fonction de sa [définition](../virtual-machines/extensions/key-vault-windows.md#extension-schema) :
```json
"linkOnRenewal": <Only Windows. This feature enables auto-rotation of SSL certificates, without necessitating a re-deployment or binding.  e.g.: false>,
```

Les certificats utilisés pour établir une connexion TLS sont généralement [acquis en tant que descripteur](/windows/win32/api/sspi/nf-sspi-acquirecredentialshandlea) via le fournisseur SSP (Security Support Provider) du canal S. Autrement dit, le client n’accède pas directement à la clé privée du certificat proprement dit. Le canal S prend en charge la redirection (liaison) des informations d’identification sous la forme d’une extension de certificat ([CERT_RENEWAL_PROP_ID](/windows/win32/api/wincrypt/nf-wincrypt-certsetcertificatecontextproperty#cert_renewal_prop_id)) : si cette propriété est définie, sa valeur représente l’empreinte du certificat de « renouvellement », de sorte que le canal S tente à la place de charger le certificat lié. En fait, il parcourt cette liste liée (et, espérons-le, acyclique) jusqu’à ce qu’il se retrouve avec le certificat « final » (sans marque de renouvellement). Utilisée judicieusement, cette fonctionnalité limite efficacement la perte de disponibilité due à l’expiration de certificats (par exemple). Dans d’autres cas, elle peut occasionner des pannes difficiles à diagnostiquer et à atténuer. Le canal S exécute le balayage des certificats sur leurs propriétés de renouvellement de façon inconditionnelle, sans tenir compte de l’objet, des émetteurs ou de tout autre attribut spécifique participant à la validation du certificat obtenu par le client. Il est possible, en effet, que le certificat obtenu n’ait pas de clé privée associée, ou que la clé n’ait pas été mise en liste ACL pour son utilisateur potentiel. 
 
Si la liaison est activée, l’extension de machine virtuelle de KeyVault, lors de la récupération d’un certificat observé à partir du coffre, tente de trouver des certificats existants correspondants afin de les lier via la propriété d’extension de renouvellement. La correspondance est (exclusivement) basée sur l’autre nom de l’objet (SAN ) et fonctionne comme illustré ci-dessous.
Supposons deux certificats existants, comme suit : A : CN = “Accessoires d’Alice”, SAN = {“alice.universalexports.com”}, renewal = ‘’ B: CN = “bits de Bob”, SAN = {“bob.universalexports.com”, “bob.universalexports.net”}, renewal = ‘’
 
Supposons qu’un certificat C soit récupéré par l’extension de machine virtuelle de KeyVault : CN = “Programme malveillant de Mallory”, SAN = {“alice.universalexports.com”, “bob.universalexports.com”, “mallory.universalexports.com”}
 
La liste SAN de A est entièrement incluse dans celle de C, de sorte que A.renewal = C.thumbprint. La liste SAN de B présente une intersection avec celle de C, mais n’y est pas entièrement incluse, de sorte que B.renewal reste vide.
 
Toute tentative d’invoquer AcquireCredentialsHandle (canal S) dans cet état sur le certificat A aboutira en réalité à l’envoi de C à la partie distante. Dans le cas de Service Fabric, le [Sous-système de transport](service-fabric-architecture.md#transport-subsystem) d’un cluster utilise le canal S pour l’authentification mutuelle. Par conséquent, le comportement décrit ci-dessus affecte directement la communication fondamentale du cluster. Toujours sur la base de l’exemple ci-dessus, en supposant que A est le certificat de cluster, ce qui se passe ensuite dépend des circonstances :
  - Si la clé privée de C ne figure pas dans la liste ACL pour le compte sous l’identité duquel Fabric s’exécute, des échecs se produisent en lien avec l’acquisition de la clé privée (SEC_E_UNKNOWN_CREDENTIALS ou similaires).
  - Si la clé privée de C est accessible, des échecs d’autorisation sont renvoyés par les autres nœuds (CertificateNotMatched, non autorisé, etc.). 
 
Dans les deux cas, le transport échoue et le cluster peut cesser de fonctionner. Les symptômes varient. Pour aggraver les choses, la liaison dépend de l’ordre de renouvellement qui est dicté par l’ordre de la liste des certificats observés de l’extension de machine virtuelle de KeyVault, par le calendrier de renouvellement dans le coffre, voire par des erreurs transitoires qui modifieraient l’ordre de récupération.

Pour minimiser de tels incidents, suivez les recommandons suivantes :
  - Ne mélangez pas les SAN de différents certificats de coffre. Chaque certificat de coffre doit avoir un objectif propre, et son objet ainsi que son SAN doivent refléter cela de façon spécifique.
  - Incluez le nom commun de l’objet dans la liste SAN (littéralement, sous la forme "CN=<subject common name>").  
  - En cas de doute, désactivez la liaison lors du renouvellement des certificats approvisionnés avec l’extension de machine virtuelle de KeyVault. 

#### <a name="why-use-a-user-assigned-managed-identity-what-are-the-implications-of-using-it"></a>Pourquoi utiliser une identité managée affectée par l’utilisateur ? Quelles sont les implications de son utilisation ?
Comme le montrent les extraits de code json ci-dessus, un séquencement spécifique des opérations/mises à jour est nécessaire pour garantir le succès de la conversion et maintenir la disponibilité du cluster. Plus précisément, la ressource de groupe de machines virtuelles identiques déclare et utilise son identité pour récupérer des secrets en une seule mise à jour (dans la perspective de l’utilisateur). L’extension de machine virtuelle de Service Fabric (qui amorce le cluster) dépend de l’exécution de l’extension de machine virtuelle de KeyVault, qui dépend du succès de la récupération des certificats observés. L’extension de machine virtuelle de KeyVault utilise l’identité du groupe de machines virtuelles identiques pour accéder au coffre, ce qui signifie que la stratégie d’accès sur le coffre doit avoir déjà été mise à jour avant le déploiement du groupe de machines virtuelles identiques. 

Pour disposer de la création d’une identité managée ou pour l’affecter à une autre ressource, l’opérateur de déploiement doit avoir le rôle requis (ManagedIdentityOperator) dans l’abonnement ou le groupe de ressources, en plus des rôles requis pour gérer les autres ressources référencées dans le modèle. 

Du point de vue de la sécurité, rappelez-vous que la machine virtuelle (ou le groupe de machines virtuelles identiques) est considérée comme une limite de sécurité en lien avec son identité Azure. Cela signifie que toute application hébergée sur la machine virtuelle pourrait, en principe, obtenir un jeton d’accès représentant la machine virtuelle. Les jetons d’accès d’identité managée sont obtenus à partir du point de terminaison Instance Metadata Service [IMDS] non authentifié. Si vous considérez la machine virtuelle comme un environnement partagé ou mutualisé, cette méthode de récupération des certificats de cluster n’est peut-être pas indiquée. Il s’agit, cependant, du seul mécanisme d’approvisionnement adéquat pour le renouvellement automatique de certificat.

## <a name="troubleshooting-and-frequently-asked-questions"></a>Résolution des problèmes et foire aux questions

*Q* : Comment s’inscrire par programme dans un certificat géré par KeyVault ?
*R* : Découvrez le nom de l’émetteur dans la documentation de KeyVault, puis remplacez-le dans le script ci-dessous.  
```PowerShell
  $issuerName=<depends on your PKI of choice>
    $clusterVault="sftestcus"
    $clusterCertVaultName="sftstcncluster"
    $clusterCertCN="cus.cluster.sftstcn.system.servicefabric.azure-int"
    Set-AzKeyVaultCertificateIssuer -VaultName $clusterVault -Name $issuerName -IssuerProvider $issuerName
    $distinguishedName="CN=" + $clusterCertCN
    $policy = New-AzKeyVaultCertificatePolicy `
        -IssuerName $issuerName `
        -SubjectName $distinguishedName `
        -SecretContentType 'application/x-pkcs12' `
        -Ekus "1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2" `
        -ValidityInMonths 4 `
        -KeyType 'RSA' `
        -RenewAtPercentageLifetime 75        
    Add-AzKeyVaultCertificate -VaultName $clusterVault -Name $clusterCertVaultName -CertificatePolicy $policy
    
    # poll the result of the enrollment
    Get-AzKeyVaultCertificateOperation -VaultName $clusterVault -Name $clusterCertVaultName 
```

*Q* : Que se passe-t-il quand un certificat est émis par un émetteur non déclaré ou non spécifié ? Où puis-je obtenir la liste exhaustive des émetteurs actifs d’une infrastructure de clé publique donnée ?
*R* : Si la déclaration de certificat spécifie les empreintes de l’émetteur et si l’émetteur direct du certificat n’est pas inclus dans la liste des émetteurs épinglés, le certificat est considéré comme non valide, que sa racine soit approuvée ou non par le client. Il est donc indispensable de s’assurer que la liste des émetteurs est actuelle/à jour. L’introduction d’un nouvel émetteur est un événement rare qui devrait être largement annoncé avant que commence l’émission de certificats. 

En général, une infrastructure de clé publique publie et conserve une déclaration de pratique de certification, conformément à la norme [RFC 7382](https://tools.ietf.org/html/rfc7382) de l’IETF. Entre autres informations, elle inclut tous les émetteurs actifs. La récupération de cette liste par programme peut différer d’une infrastructure de clé publique à l’autre.   

Pour les infrastructures de clé publique internes de Microsoft, veuillez consulter la documentation interne sur les points de terminaison/kits de développement logiciel (SDK) utilisés pour récupérer les émetteurs autorisés. Il incombe au propriétaire du cluster d’examiner régulièrement cette liste pour s’assurer que la définition de son cluster inclut *tous* les émetteurs attendus.

*Q* : Plusieurs infrastructures de clé publique sont-elles prises en charge ? 
*R* : Oui. Vous ne pouvez pas déclarer plusieurs entrées de nom commun dans le manifeste de cluster ayant la même valeur, mais vous pouvez répertorier les émetteurs de plusieurs infrastructures de clé publique correspondant au même nom commun. Cette pratique n’est pas recommandée et les pratiques de transparence des certificats peuvent empêcher l’émission de tels certificats. Il s’agit cependant d’un mécanisme acceptable comme moyen de migrer d’une infrastructure de clé publique vers une autre.

*Q* : Que se passe-t-il si le certificat de cluster actuel n’est pas émis par une autorité de certification ou n’a pas l’objet souhaité ? 
*R* : Obtenez un certificat avec l’objet souhaité et ajoutez-le par empreinte à la définition du cluster en tant que certificat secondaire. Une fois la mise à niveau accomplie, lancez une autre mise à niveau de la configuration du cluster pour convertir la déclaration de certificat en nom commun. 

[Image1]:./media/security-cluster-certificate-mgmt/certificate-journey-thumbprint.png
[Image2]:./media/security-cluster-certificate-mgmt/certificate-journey-common-name.png
