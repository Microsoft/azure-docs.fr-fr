---
title: Comment exécuter le service de gestion de certificats OPC Vault en toute sécurité – Azure | Microsoft Docs
description: Explique comment exécuter le service de gestion de certificats OPC Vault de manière sécurisée dans Azure et passe en revue les autres consignes de sécurité à prendre en considération.
author: mregen
ms.author: mregen
ms.date: 8/16/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: f54467c03a67797d52ac499ca9ba455c0f75e240
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071453"
---
# <a name="run-the-opc-vault-certificate-management-service-securely"></a>Exécuter le service de gestion de certificats OPC Vault de manière sécurisée

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article explique comment exécuter le service de gestion de certificats OPC Vault de manière sécurisée dans Azure et passe en revue les autres consignes de sécurité à prendre en considération.

## <a name="roles"></a>Rôles

### <a name="trusted-and-authorized-roles"></a>Rôles approuvés et autorisés

Le microservice OPC Vault prévoit des rôles distincts pour accéder à différentes parties du service.

> [!IMPORTANT]
> Pendant le déploiement, le script ajoute uniquement l’utilisateur qui exécute le script de déploiement en tant qu’utilisateur pour tous les rôles. Pour un déploiement de production, vous devez examiner cette attribution de rôle et la reconfigurer de manière appropriée en suivant les consignes ci-dessous. Cette tâche nécessite une attribution manuelle de rôles et de services sur le portail Applications d’entreprise Azure Active Directory (Azure AD).

### <a name="certificate-management-service-roles"></a>Rôles de service de gestion des certificats

Le microservice OPC Vault définit les rôles suivants :

- **Lecteur** : Par défaut, tout utilisateur authentifié dans le locataire dispose d’un accès en lecture. 
  - Accès en lecture aux applications et aux demandes de certificat. Peut répertorier et interroger les applications et demandes de certificat. Les informations de découverte d’appareil et les certificats publics sont également accessibles avec un accès en lecture.
- **Rédacteur** : Le rôle Rédacteur est attribué à un utilisateur appelé à ajouter des autorisations d’écriture pour certaines tâches. 
  - Accès en lecture/écriture aux applications et aux demandes de certificat. Peut inscrire, mettre à jour et annuler l’inscription des applications. Peut créer des demandes de certificat et obtenir des clés privées et des certificats approuvés. Peut également supprimer des clés privées.
- **Approbateur** : Le rôle Approbateur est attribué à un utilisateur appelé à approuver ou à rejeter des demandes de certificat. Il n’inclut aucun autre rôle.
  - Pour pouvoir signer les certificats, en plus du rôle Approbateur permettant d’accéder à l’API du microservice OPC Vault, l’utilisateur doit aussi disposer de l’autorisation de signature de clé dans Azure Key Vault.
  - Les rôles Rédacteur et Approbateur doit être attribué à des utilisateurs distincts.
  - Le rôle principal de l’approbateur est l’approbation de la génération et du rejet des demandes de certificats.
- **Administrateur** : Le rôle Administrateur est attribué à un utilisateur appelé à gérer les groupes de certificats. Le rôle ne prend pas en charge le rôle Approbateur, mais il comprend le rôle Rédacteur.
  - L’administrateur peut gérer les groupes de certificats, modifier la configuration et révoquer des certificats d’application en émettant une nouvelle liste de révocation de certificats.
  - Idéalement, les rôles Rédacteur, Approbateur et Administrateur sont attribués à des utilisateurs distincts. Pour renforcer la sécurité, un utilisateur auquel est attribué le rôle Approbateur ou Administrateur a aussi besoin de l’autorisation de signature de clé dans Key Vault pour émettre des certificats ou renouveler un certificat d’autorité de certification émettrice.
  - En plus du rôle d’administration du microservice, le rôle inclut notamment les attributions suivantes :
    - Responsabilité d’administration de l’implémentation des pratiques de sécurité de l’autorité de certification.
    - Gestion de la génération, de la révocation et de la suspension des certificats. 
    - Gestion du cycle de vie des clés de chiffrement (par exemple, renouvellement des clés de l’autorité de certification émettrice).
    - Installation, configuration et maintenance des services qui utilisent l’autorité de certification.
    - Conduite au quotidien des services. 
    - Sauvegarde et récupération de base de données et d’autorité de certification.

### <a name="other-role-assignments"></a>Autres attributions de rôles

Prenez aussi en considération les rôles suivants au moment d’exécuter le service :

- Propriétaire du contrat d’approvisionnement de certificats conclu avec l’autorité de certification racine externe (par exemple, quand le propriétaire achète des certificats à une autorité de certification externe ou utilise une autorité de certification subordonnée à une autorité de certification externe).
- Développement et validation de l’autorité de certification.
- Révision des enregistrements d’audit.
- Le personnel qui participe au support de l’autorité de certification ou à la gestion des installations physiques et cloud, mais qui n’est pas directement approuvé pour effectuer des opérations d’autorité de certification, appartient au rôle *autorisé*. L’ensemble des tâches que les personnes auxquelles est attribué le rôle autorisé peuvent effectuer doivent également être documentées.

### <a name="review-memberships-of-trusted-and-authorized-roles-quarterly"></a>Passer en revue les appartenances des rôles approuvés et autorisés tous les trimestres

Passez en revue l’appartenance des rôles approuvés et autorisés au moins tous les trimestres. Veillez à maintenir l’ensemble des personnes (pour les processus manuels) ou les identités de service (pour les processus automatisés) de chaque rôle au minimum.

### <a name="role-separation-between-certificate-requester-and-approver"></a>Séparation des rôles entre le demandeur de certificat et l’approbateur

Le processus d’émission de certificat doit appliquer le principe de la séparation des rôles entre le demandeur et l’approbateur de certificat (personnes ou systèmes automatisés). L’émission de certificat doit être autorisée par un rôle d’approbateur de certificat qui vérifie que le demandeur de certificat est autorisé à obtenir des certificats. Les personnes qui détiennent le rôle d’approbateur de certificat doivent être formellement autorisées.

### <a name="restrict-assignment-of-privileged-roles"></a>Restreindre l’attribution de rôles privilégiés

Vous avez tout intérêt à restreindre l’attribution de rôles privilégiés, par exemple en autorisant l’appartenance au groupe Administrateurs et Approbateurs, à un ensemble limité de personnes autorisées. Toute modification de rôle privilégié doit avoir un accès révoqué dans les 24 heures. Enfin, passez en revue les attributions de rôles privilégiés tous les trimestres et supprimez les affectations inutiles ou arrivées à expiration.

### <a name="privileged-roles-should-use-two-factor-authentication"></a>Les rôles privilégiés doivent utiliser une authentification à deux facteurs

Utilisez l’authentification multifacteur (aussi connue sous le nom d’authentification à deux facteurs) pour la connexion interactive des approbateurs et des administrateurs au service.

## <a name="certificate-service-operation-guidelines"></a>Instructions opérationnelles concernant le service de certificats

### <a name="operational-contacts"></a>Contacts opérationnels

Le service de certificats doit disposer d’un plan de réponse de sécurité à jour consigné dans un fichier contenant des contacts détaillés pour la réponse aux incidents opérationnels.

### <a name="security-updates"></a>Mises à jour de sécurité

Tous les systèmes doivent être surveillés et mis à jour en permanence avec les mises à jour de sécurité les plus récentes.

> [!IMPORTANT]
> Le dépôt GitHub du service OPC Vault est continuellement mis à jour avec des correctifs de sécurité. Surveillez ces mises à jour et appliquez-les au service à intervalles réguliers.

### <a name="security-monitoring"></a>Surveillance de la sécurité

Abonnez-vous à une solution de supervision de la sécurité ou implémentez-en une. Par exemple, abonnez-vous à une solution de supervision centralisée (comme Azure Security Center ou la solution de supervision Microsoft 365) et configurez-la de sorte que les événements de sécurité lui soient transmis.

> [!IMPORTANT]
> Par défaut, le service OPC Vault est déployé avec [Azure Application Insights](../azure-monitor/app/devops.md) en guise de solution de supervision. L’ajout d’une solution de sécurité telle qu’[Azure Security Center](https://azure.microsoft.com/services/security-center/) est fortement recommandé.

### <a name="assess-the-security-of-open-source-software-components"></a>Évaluer la sécurité des composants logiciels open source

Tous les composants open source utilisés au sein d’un produit ou service doivent être exempts de failles de sécurité modérées ou importantes.

> [!IMPORTANT]
> Pendant les builds d’intégration continue, le dépôt GitHub du service OPC Vault analyse tous les composants à la recherche de vulnérabilités. Surveillez ces mises à jour sur GitHub et appliquez-les au service à intervalle réguliers.

### <a name="maintain-an-inventory"></a>Tenue d’inventaire

Tenez un inventaire des ressources pour tous les hôtes de production (y compris les machines virtuelles persistantes), appareils, plages d’adresses IP internes, adresses IP virtuelles et noms de domaine DNS publics. Chaque fois que vous ajoutez ou supprimez un système, une adresse IP d’appareil, une adresse IP virtuelle ou un domaine DNS public, vous devez mettre à jour l’inventaire dans un délai de 30 jours.

#### <a name="inventory-of-the-default-azure-opc-vault-microservice-production-deployment"></a>Inventaire du déploiement de production par défaut du microservice Azure OPC Vault 

Dans Azure :
- **App Service Plan** (Plan App Service) : Plan App service pour les hôtes de service. Par défaut S1.
- **App service** pour microservice : Hôte du service OPC Vault.
- **App service** pour exemple d’application : Exemple d’application hôte d’OPC Vault.
- **Key Vault Standard** : Pour stocker les secrets et les clés Azure Cosmos DB pour les services web.
- **Key Vault Premium** : Pour héberger les clés de l’autorité de certification émettrice, pour le service de signature et pour la configuration du coffre et le stockage des clés privées d’application.
- **Azure Cosmos DB** : Base de données pour les demandes d’application et de certificat. 
- **Application Insights** : solution de surveillance (facultative) pour le service web et l’application.
- **Inscription d’application Azure AD** : Inscription de l’exemple d’application, du service et du module Edge.

Pour les services cloud, tous les noms d’hôte, groupes de ressources, noms de ressource, ID d’abonnement et ID de locataire utilisés pour déployer le service doivent être documentés. 

Dans Azure IoT Edge ou un serveur IoT Edge local :
- **Module IoT Edge OPC Vault** : Pour prendre en charge un serveur de découverte global OPC UA de réseau de fabrique. 

Pour les appareils IoT Edge, les noms d’hôte et les adresses IP doivent être documentés. 

### <a name="document-the-certification-authorities-cas"></a>Documenter les autorités de certification

La documentation de la hiérarchie d’autorités de certification doit contenir toutes les autorités de certification utilisées. Cela englobe l’ensemble des autorités de certification secondaires, des autorités de certification parentes et des autorités de certification racines associées, même quand elles ne sont pas gérées par le service. Plutôt qu’une documentation formelle, vous pouvez fournir un ensemble exhaustif de tous les certificats d’autorité de certification non expirés.

> [!NOTE]
> L’exemple d’application OPC Vault prend en charge le téléchargement de tous les certificats utilisés et produits dans le service à des fins de documentation.

### <a name="document-the-issued-certificates-by-all-certification-authorities-cas"></a>Documenter les certificats émis par toutes les autorités de certification

Fournissez un ensemble exhaustif de tous les certificats émis au cours des 12 derniers mois.

> [!NOTE]
> L’exemple d’application OPC Vault prend en charge le téléchargement de tous les certificats utilisés et produits dans le service à des fins de documentation.

### <a name="document-the-standard-operating-procedure-for-securely-deleting-cryptographic-keys"></a>Documenter la procédure d’exploitation standard pour supprimer les clés de chiffrement de manière sécurisée

Pendant la durée de vie d’une autorité de certification, il peut être très rare que des clés soient supprimées. C’est pourquoi aucun utilisateur ne dispose d’un droit de suppression de certificat Key Vault et qu’aucune API n’est exposée pour supprimer un certificat d’autorité de certification émettrice. La procédure d’exploitation standard manuelle pour la suppression sécurisée de clés de chiffrement d’autorité de certification n’est disponible qu’en accédant directement à Key Vault sur le portail Azure. Vous pouvez aussi supprimer le groupe de certificats dans Key Vault. Pour assurer une suppression immédiate, désactivez la fonctionnalité de [suppression réversible de Key Vault](../key-vault/general/soft-delete-overview.md).

## <a name="certificates"></a>Certificats

### <a name="certificates-must-comply-with-minimum-certificate-profile"></a>Les certificats doivent être conformes au profil de certificat minimal

Le service OPC Vault est une autorité de certification en ligne qui émet des certificats d’entité finale pour des abonnés. Le microservice OPC Vault respect ces directives dans on implémentation par défaut.

- Tous les certificats doivent inclure les champs X.509 suivants, comme indiqué ci-dessous :
  - Le contenu du champ Version doit être v3. 
  - Le contenu du champ serialNumber doit inclure au moins 8 octets d’entropie obtenus auprès d’un générateur de nombres aléatoires certifié conforme à la norme FIPS (Federal Information Processing Standards) 140.<br>
    > [!IMPORTANT]
    > Par défaut, le numéro de série d’OPC Vault est de 20 octets et est obtenu auprès du générateur de nombres aléatoires de chiffrement du système d’exploitation. Le générateur de nombres aléatoires est certifié conforme à la norme FIPS 140 sur les appareils Windows, mais pas sur Linux. Tenez-en compte au moment de choisir un déploiement de service qui utilise des machines virtuelles Linux ou des conteneurs Docker Linux sur lesquels la technologie sous-jacente OpenSSL n’est pas certifiée conforme à la norme FIPS 140.
  - Les champs issuerUniqueID et subjectUniqueID ne doivent pas être présents.
  - Les certificats d’entité finale doivent être identifiés avec l’extension de contraintes de base, conformément à la norme IETF RFC 5280.
  - Le champ pathLenConstraint doit avoir la valeur 0 pour le certificat de l’autorité de certification émettrice. 
  - L’extension Utilisation améliorée de la clé doit être présente et contenir l’ensemble minimal d’identificateurs d’objet (OID) pour l’Utilisation améliorée de la clé. L’OID anyExtendedKeyUsage (2.5.29.37.0) ne doit pas être spécifié. 
  - L’extension Point de distribution de liste de révocation de certificats (CDP) doit être présente dans le certificat de l’autorité de certification émettrice.<br>
    > [!IMPORTANT]
    > L’extension CDP est présente dans les certificats d’autorité de certification OPC Vault. Néanmoins, les appareils OPC UA utilisent des méthodes personnalisées pour distribuer les listes de révocation de certificats.
  - L’extension Accès aux informations de l’autorité doit être présente dans les certificats de l’abonné.<br>
    > [!IMPORTANT]
    > L’extension Accès aux informations de l’autorité doit être présente dans les certificats d’abonné OPC Vault. Néanmoins, les appareils OPC UA utilisent des méthodes personnalisées pour distribuer les informations sur l’autorité de certification émettrice.
- Des algorithmes asymétriques approuvés, des longueurs de clé, des fonctions de hachage et des modes de remplissage doivent être utilisés.
  - Les algorithmes RSA et SHA-2 sont les seuls pris en charge.
  - L’algorithme RSA peut être utilisé pour le chiffrement, l’échange de clés et la signature.
  - Le chiffrement RSA doit utiliser uniquement les modes de remplissage OAEP, RSA-KEM ou RSA-PSS.
  - Des longueurs de clé supérieures ou égales à 2 048 bits sont exigées.
  - Utilisez la famille d’algorithmes de hachage SHA-2 (SHA256, SHA384 et SHA512).
  - Les clés d’autorité de certification racine RSA d’une durée de vie habituelle de 20 ans et plus doivent comporter au minimum 4 096 bits.
  - Les clés de l’autorité de certification émettrice RSA doivent comporter au moins 2 048 bits. Si la date d’expiration du certificat de l’autorité de certification est postérieure à 2030, la clé de l’autorité de certification doit comporter au minimum 4 096 bits.
- Durée de vie des certificats
  - Certificats d’autorité de certification racine : La période maximale de validité du certificat pour les autorités de certification racine ne doit pas dépasser 25 ans.
  - Certificats d’autorité de certification d’abonné ou d’autorité de certification émettrice en ligne : La période maximale de validité des certificats pour les autorités de certification qui sont en ligne et n’émettent que des certificats d’abonné ne peut pas dépasser six ans. Pour ces autorités de certification, la clé de signature privée associée ne doit pas être utilisée pendant plus de trois ans pour émettre de nouveaux certificats.<br>
    > [!IMPORTANT]
    > Le certificat émetteur, tel qu’il est généré dans le microservice OPC Vault par défaut sans autorité de certification racine externe, est traité comme une sous-autorité de certification en ligne avec des exigences et durées de vie associées. La durée de vie définie par défaut est de cinq ans, avec une longueur de clé supérieure ou égale à 2 048.
  - Toutes les clés asymétriques doivent avoir une durée de vie maximale de cinq ans et une durée de vie recommandée d’un an.<br>
    > [!IMPORTANT]
    > Par défaut, la durée de vie des certificats d’application émis avec OPC Vault est de deux ans et ils doivent être remplacés tous les ans. 
  - Un certificat est chaque fois renouvelé avec une nouvelle clé.
- Extensions spécifiques d’OPC UA dans les certificats d’instance d’application
  - L’extension subjectAltName comprend l’URI et les noms d’hôte de l’application. Ils peuvent aussi inclure des adresses de nom de domaine complet (FQDN), IPv4 et IPv6.
  - La valeur keyUsage inclut les informations digitalSignature, nonRepudiation, keyEncipherment et dataEncipherment.
  - La valeur extendedKeyUsage inclut les informations serverAuth et clientAuth.
  - La valeur authorityKeyIdentifier est spécifiée dans les certificats signés.

### <a name="ca-keys-and-certificates-must-meet-minimum-requirements"></a>Les clés et certificats d’autorité de certification doivent répondre à des exigences minimales

- **Clés privées** : Les clés RSA doivent comporter au moins 2 048 bits. Si la date d’expiration du certificat de l’autorité de certification est postérieure à 2030, la clé de l’autorité de certification doit comporter au minimum 4 096 bits.
- **Durée de vie** : La période maximale de validité des certificats pour les autorités de certification qui sont en ligne et n’émettent que des certificats d’abonné ne peut pas dépasser six ans. Pour ces autorités de certification, la clé de signature privée associée ne doit pas être utilisée pendant plus de trois ans pour émettre de nouveaux certificats.

### <a name="ca-keys-are-protected-using-hardware-security-modules"></a>Les clés d’autorité de certification sont protégées à l’aide de modules de sécurité matériels

OpcVault utilise Azure Key Vault Premium, et les clés sont protégées par des modules de sécurité matériels (HSM) FIPS 140-2 de niveau 2. 

Les modules de chiffrement utilisés par Key Vault, HSM ou logiciels, sont conformes aux normes FIPS. Les clés créées ou importées comme clés protégées par module de sécurité matériel (HSM) sont traitées dans un HSM certifié conforme à la norme FIPS 140-2 de niveau 2. Les clés créées ou importées comme clés protégées par logiciel sont traitées dans des modules de chiffrement certifiés conformes à la norme FIPS 140-2 de niveau 1.

## <a name="operational-practices"></a>Pratiques opérationnelles

### <a name="document-and-maintain-standard-operational-pki-practices-for-certificate-enrollment"></a>Documenter et tenir à jour les pratiques d’infrastructure à clé publique (PKI) opérationnelles standard pour l’inscription de certificat

Documentez et tenez à jour les procédures opérationnelles standard (SOP) régissant la manière dont les autorités de certification (CA) émettent les certificats, à savoir : 
- Mode d’identification et d’authentification de l’abonné. 
- Mode de traitement et de validation des demandes de certificats (le cas échéant, précisez également comment les demandes de renouvellement de certificats et de recréation de clés sont traitées). 
- Mode de distribution des certificats émis aux abonnés. 

Les procédures SOP du microservice OPC Vault sont décrites dans [Architecture d’OPC Vault](overview-opc-vault-architecture.md) et [Gérer le service de certificats OPC Vault](howto-opc-vault-manage.md). Les pratiques suivent le standard « OPC Unified Architecture Specification Part 12 : Discovery and Global Services ».


### <a name="document-and-maintain-standard-operational-pki-practices-for-certificate-revocation"></a>Documenter et tenir à jour les pratiques d’infrastructure à clé publique (PKI) opérationnelles standard pour la révocation de certificat

Le processus de révocation de certificat est décrit dans [Architecture d’OPC Vault](overview-opc-vault-architecture.md) et [Gérer le service de certificats OPC Vault](howto-opc-vault-manage.md).
    
### <a name="document-ca-key-generation-ceremony"></a>Documenter le cérémonial de génération de clés de l’autorité de certification 

La génération de clés par l’autorité de certification émettrice dans le microservice OPC Vault est simplifiée en raison du stockage sécurisé dans Azure Key Vault. Pour plus d’informations, consultez [Gérer le service de certificats OPC Vault](howto-opc-vault-manage.md).

Cependant, quand vous avez recours à une autorité de certification racine externe, le cérémonial de génération de clés de l’autorité de certification doit répondre aux exigences suivantes.

Le cérémonial de génération de clé de l’autorité de certification doit être conforme à un script documenté spécifiant au moins les points suivants : 
- Définition des rôles et des responsabilités des participants.
- Approbation de la conduite du cérémonial de génération de clés de l’autorité de certification.
- Matériel de chiffrement et supports d’activation nécessaires au cérémonial.
- Préparation du matériel (y compris mise à jour et validation des informations relatives aux ressources et à la configuration).
- Installation du système d’exploitation.
- Étapes spécifiques effectuées pendant le cérémonial de génération de clé de l’autorité de certification, par exemple : 
  - Installation et configuration de l’application de l’autorité de certification.
  - Génération de clés de l’autorité de certification.
  - Sauvegarde des clés de l’autorité de certification.
  - Signature des certificats de l’autorité de certification.
  - Importation de clés signées dans le module de sécurité matériel (HSM) protégé du service.
  - Arrêt du système de l’autorité de certification.
  - Préparation des supports pour le stockage.


## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment gérer OPC Vault de manière sécurisée, vous pouvez :

> [!div class="nextstepaction"]
> [Sécuriser des appareils OPC UA avec OPC Vault](howto-opc-vault-secure.md)