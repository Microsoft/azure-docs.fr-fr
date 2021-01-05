---
title: Questions fréquentes (FAQ) - HSM dédié Azure | Microsoft Docs
description: Trouvez des réponses aux questions courantes sur le module de sécurité du matériel dédié Azure, comme les informations de base, l’interopérabilité, la haute disponibilité et la prise en charge.
services: dedicated-hsm
author: johncdawson
manager: rkarlin
tags: azure-resource-manager
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/05/2020
ms.author: mbaldwin
ms.openlocfilehash: 0d3309283279c887c00a475ccedfe1edaf311ae5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88189877"
---
# <a name="frequently-asked-questions-faq"></a>Forum Aux Questions (FAQ)

Trouvez des réponses aux questions fréquentes sur le service HSM dédié Microsoft Azure.

## <a name="the-basics"></a>Concepts de base

### <a name="q-what-is-a-hardware-security-module-hsm"></a>Q : Qu’est-ce qu’un module de sécurité matériel (HSM) ?

Un module de sécurité matériel (HSM) est un appareil informatique physique qui sert à protéger et à gérer les clés de chiffrement. Les clés stockées dans les modules HSM peuvent être utilisées pour les opérations de chiffrement. Elles sont conservées en toute sécurité dans des modules matériels inviolables. Le module HSM autorise uniquement les applications authentifiées et autorisées à utiliser les clés. Les clés ne quittent jamais la limite de protection du module HSM.

### <a name="q-what-is-the-azure-dedicated-hsm-offering"></a>Q : En quoi consiste l’offre du service HSM dédié Azure ?

Le service HSM dédié Azure est un service cloud qui fournit des modules HSM hébergés dans des centres de données Azure qui sont directement connectés au réseau virtuel d’un client. Ces modules HSM sont des appliances réseau dédiées (SafeNet Network HSM 7 modèle A790 de Gemalto). Ils sont déployés directement dans l’espace d’adressage IP privé du client et Microsoft n’a pas accès à la fonctionnalité de chiffrement des modules HSM. Seul le client dispose d’un contrôle administratif et cryptographique complet sur ces appareils. Les clients sont responsables de la gestion des appareils et peuvent obtenir des journaux d’activité complets directement à partir de leurs appareils. Les modules HSM dédiés permettent aux clients de respecter les exigences de conformité/réglementaires comme FIPS 140-2 de niveau 3, HIPAA, PCI-DSS, eIDAS et bien d’autres.

### <a name="q-what-hardware-is-used-for-dedicated-hsm"></a>Q : Quel est le matériel utilisé pour le service HSM dédié ?

Microsoft a conclu un partenariat avec Gemalto pour fournir le service HSM dédié Azure. L’appareil utilisé est le [SafeNet Luna Network HSM 7 modèle A790](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/). En plus d’intégrer un microprogramme certifié FIPS 140-2 de niveau 3, cet appareil offre aussi une faible latence, de hautes performances et une capacité élevée via 10 partitions. 

### <a name="q-what-is-an-hsm-used-for"></a>Q : À quoi sert un HSM ?

Un module HSM sert à stocker les clés de chiffrement utilisées pour les fonctionnalités de chiffrement comme TLS (Transport Layer Security), le chiffrement de données, l’infrastructure à clé publique (PKI), la gestion des droits numériques (DRM) et la signature de documents.

### <a name="q-how-does-dedicated-hsm-work"></a>Q : Comment fonctionne un HSM dédié ?

Les clients peuvent provisionner des modules HSM dans des régions spécifiques à l’aide de PowerShell ou d’une interface de ligne de commande. Le client spécifie le réseau virtuel auquel les modules HSM seront connectés et une fois provisionnés, ceux-ci seront disponibles dans le sous-réseau désigné à des adresses IP attribuées dans l’espace d’adressage IP privé du client. Les clients peuvent ensuite se connecter aux modules HSM en utilisant SSH pour la gestion et l’administration des appliances, configurer les connexions clientes HSM, initialiser des modules HSM, créer des partitions, définir et attribuer des rôles tels que responsable de partition, responsable de chiffrement et utilisateur de chiffrement. Ensuite, les clients utilisent les outils/SDK/logiciels du client HSM fournis par Gemalto pour effectuer des opérations de chiffrement à partir de leurs applications.

### <a name="q-what-software-is-provided-with-the-dedicated-hsm-service"></a>Q : Quels sont les logiciels fournis avec le service HSM dédié ?

Gemalto fournit tous les logiciels pour l’appareil HSM une fois qu’il est provisionné par Microsoft. Les logiciels sont disponibles sur le [portail du service clientèle Gemalto](https://supportportal.gemalto.com/csm/). Les clients qui utilisent le service HSM dédié doivent s’inscrire pour bénéficier du service de support Gemalto et disposer d’un ID client pour pouvoir accéder aux logiciels en question et les télécharger. Le logiciel client pris en charge est la version 7.2, qui est compatible avec la version 7.0.3 du microprogramme certifié FIPS 140-2 de niveau 3. 

### <a name="q-does-azure-dedicated-hsm-offer-password-based-and-ped-based-authentication"></a>Q : Le service HSM dédié Azure offre-t-il une authentification par mot de passe et par PED ?

Pour le moment, le service HSM dédié Azure fournit uniquement des HSM avec une authentification par mot de passe.

### <a name="q-will-azure-dedicated-hsm-host-my-hsms-for-me"></a>Q : Le service HSM dédié Azure peut-il héberger mes HSM pour moi ?

Microsoft propose uniquement le HSM de Gemalto SafeNet Luna Network par le biais du service HSM dédié, et ne peut pas héberger les appareils fournis par les clients.

### <a name="q-does-azure-dedicated-hsm-support-payment-pineft-features"></a>Q : Le service Azure Dedicated HSM prend-il en charge les fonctionnalités de paiement (code PIN/EFT) ?

Le service HSM dédié Azure utilise des appareils SafeNet Luna Network HSM 7 (modèle A790). Ces appareils ne prennent pas en charge les fonctionnalités de paiement propres à HSM (par exemple, code PIN ou EFT) ni les certifications. Si vous voulez que le service HSM dédié Azure prenne en charge les HSM de paiement à l’avenir, indiquez-le en commentaire à votre chargé de compte Microsoft.

### <a name="q-which-azure-regions-is-dedicated-hsm-available-in"></a>Q : Dans quelles régions Azure le service HSM dédié est-il disponible ?

Depuis fin mars 2019, le service HSM dédié est disponible dans les 14 régions répertoriées ci-dessous. D’autres régions sont prévues et peuvent être abordées par le biais de votre chargé de compte Microsoft.

* USA Est
* USA Est 2
* USA Ouest
* USA Ouest 2
* États-Unis - partie centrale méridionale
* Asie Sud-Est
* Asie Est
* Inde Centre
* Sud de l’Inde
* Japon Est
* OuJapon Est
* Europe Nord
* Europe Ouest
* Sud du Royaume-Uni
* Ouest du Royaume-Uni
* Centre du Canada
* Est du Canada
* Australie Est
* Sud-Australie Est
* Suisse Nord
* Suisse Ouest
* Gouvernement américain - Virginie
* Gouvernement des États-Unis – Texas

## <a name="interoperability"></a>Interopérabilité

### <a name="q-how-does-my-application-connect-to-a-dedicated-hsm"></a>Q : Comment mon application se connecte-t-elle à un HSM dédié ?

Vous devez utiliser les outils/SDK/logiciels du client HSM fournis par Gemalto pour effectuer des opérations de chiffrement à partir de vos applications. Les logiciels sont disponibles sur le [portail du service clientèle Gemalto](https://supportportal.gemalto.com/csm/). Les clients qui utilisent le service HSM dédié doivent s’inscrire pour bénéficier du service de support Gemalto et disposer d’un ID client pour pouvoir accéder aux logiciels en question et les télécharger.

### <a name="q-can-an-application-connect-to-dedicated-hsm-from-a-different-vnet-in-or-across-regions"></a>Q : Une application peut-elle se connecter à un HSM dédié à partir d’un autre réseau virtuel situé dans la même région ou des régions différentes ?

Oui, vous devrez utiliser [VNET Peering](../virtual-network/virtual-network-peering-overview.md) au sein d’une même région pour établir une connectivité entre les différents réseaux virtuels. Pour une connectivité inter-région, vous devrez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="q-can-i-synchronize-dedicated-hsm-with-on-premises-hsms"></a>Q : Puis-je synchroniser un HSM dédié avec des HSM locaux ?

Oui, vous pouvez synchroniser des modules HSM locaux avec le service HSM dédié. Il est possible d’utiliser un [VPN point à point ou une connectivité point à site](../vpn-gateway/vpn-gateway-about-vpngateways.md) pour établir une connectivité avec votre réseau local.

### <a name="q-can-i-encrypt-data-used-by-other-azure-services-using-keys-stored-in-dedicated-hsm"></a>Q : Puis-je chiffrer les données utilisées par d’autres services Azure à l’aide des clés stockées dans un HSM dédié ?

Non. Les modules HSM dédiés Azure sont accessibles uniquement à l’intérieur de votre réseau virtuel.

### <a name="q-can-i-import-keys-from-an-existing-on-premises-hsm-to-dedicated-hsm"></a>Q : Puis-je importer les clés d’un HSM local existant vers un HSM dédié ?

Oui, si vous disposez de modules HSM SafeNet Gemalto locaux. Il existe plusieurs méthodes. Consultez la documentation sur les modules HSM Gemalto.

### <a name="q-what-operating-systems-are-supported-by-dedicated-hsm-client-software"></a>Q : Quels systèmes d’exploitation les logiciels clients de HSM dédié prennent-ils en charge ?

* Windows, Linux, Solaris, AIX, HP-UX, FreeBSD
* Virtuel : VMware, Hyper-V, Xen, KVM

### <a name="q-how-do-i-configure-my-client-application-to-create-a-high-availability-configuration-with-multiple-partitions-from-multiple-hsms"></a>Q : Comment configurer mon application cliente pour créer une configuration de haute disponibilité avec plusieurs partitions issues de plusieurs HSM ?

Pour bénéficier d’une haute disponibilité, vous devez configurer votre application cliente HSM de sorte qu’elle utilise les partitions de chaque module HSM. Consultez la documentation des logiciels clients HSM Gemalto.

### <a name="q-what-authentication-mechanisms-are-supported-by-dedicated-hsm"></a>Q : Quels sont les mécanismes d’authentification pris en charge par un HSM dédié ?

Le service HSM dédié Azure utilise des appliances SafeNet Network HSM 7 (modèle A790) qui prennent en charge l’authentification par mot de passe.

### <a name="q-what-sdks-apis-client-software-is-available-to-use-with-dedicated-hsm"></a>Q : Quels SDK, API et logiciels clients est-il possible d’utiliser avec un HSM dédié ?

PKCS #11, Java (JCA/JCE), Microsoft CAPI, CNG, OpenSSL

### <a name="q-can-i-importmigrate-keys-from-luna-56-hsms-to-azure-dedicated-hsms"></a>Q : Puis-je importer/migrer des clés de HSM Luna 5/6 vers des HSM dédiés Azure ?

Oui. Consultez le guide de migration Gemalto. 

## <a name="using-your-hsm"></a>Utilisation du module HSM

### <a name="q-how-do-i-decide-whether-to-use-azure-key-vault-or-azure-dedicated-hsm"></a>Q : Comment choisir entre Azure Key Vault et le service HSM dédié Azure ?

Le service HSM dédié Azure est tout indiqué pour les entreprises qui migrent vers des applications locales Azure qui utilisent des modules HSM. Les modules HSM dédiés proposent une option qui permet de migrer une application moyennant des modifications minimes. Si des opérations de chiffrement sont effectuées dans le code de l’application s’exécutant dans une machine virtuelle ou une application web Azure, elles peuvent utiliser HSM dédié. En règle générale, les logiciels prêts à l’emploi s’exécutant selon des modèles IaaS (infrastructure as a service) et qui prennent en charge les modules HSM comme magasin de clés peuvent utiliser le service HSM dédié, notamment Application Gateway ou Traffic Manager pour le TLS sans clé, les services de certificats Active Directory (AD CS), ou des outils d’infrastructure à clé publique (PKI) similaires, des outils/applications utilisés pour la signature de documents, la signature de code ou un serveur SQL Server (IaaS) configuré avec le chiffrement transparent des données (TDE) en ayant la clé principale dans un module HSM utilisant un fournisseur de gestion de clés extensible (EKM). Azure Key Vault convient pour des applications « natives du cloud » ou pour des scénarios de chiffrement au repos où les données des clients sont traitées par des scénarios PaaS (Platform as a Service) ou SaaS (Software as a Service), comme le chiffrement Clé client dans Office 365, Azure Information Protection, Azure Disk Encryption, Azure Data Lake Store avec la clé gérée par le client, le chiffrement Stockage Azure avec la clé gérée par le client et Azure SQL avec la clé gérée par le client.

### <a name="q-what-usage-scenarios-best-suit-azure-dedicated-hsm"></a>Q : Quels sont les scénarios d’utilisation les plus adaptés au service HSM dédié Azure ?

Le service HSM dédié Azure est tout particulièrement indiqué pour les scénarios de migration. Cela vaut pour la migration d’applications locales vers Azure qui utilisent déjà des modules HSM. Il propose une option de migration sans accroc vers Azure qui ne demande qu’un minimum de modifications au niveau de l’application. Si des opérations de chiffrement sont effectuées dans le code de l’application s’exécutant dans une machine virtuelle ou une application web Azure, le service HSM dédié peut être utilisé. En règle générale, les logiciels prêts à l’emploi s’exécutant selon des modèles IaaS (infrastructure as a service) et qui prennent en charge les modules HSM comme magasin de clés peuvent utiliser le service HSM dédié, notamment :

* Application Gateway ou Traffic Manager pour le TLS sans clé
* Services de certificats Active Directory (AD CS)
* Outils d’infrastructure à clé publique (PKI) similaires
* Outils/applications utilisés pour la signature de document
* Signature de code
* SQL Server (IaaS) configuré avec le chiffrement transparent des données (TDE) en ayant la clé principale dans un module HSM utilisant un fournisseur de gestion de clés extensible (EKM)

### <a name="q-can-dedicated-hsm-be-used-with-office-365-customer-key-azure-information-protection-azure-data-lake-store-disk-encryption-azure-storage-encryption-azure-sql-tde"></a>Q : Un HSM dédié peut-il être utilisé avec la Clé client dans Office 365, Azure Information Protection, Azure Data Lake Store, Disk Encryption, le chiffrement de Stockage Azure, le chiffrement TDE Azure SQL ?

Non. Le service Dedicated HSM étant approvisionné directement dans l’espace d’adressage IP privé d’un client, il n’est pas accessible aux autres services Azure ou Microsoft.

## <a name="administration-access-and-control"></a>Administration, accès et contrôle

### <a name="q-does-the-customer-get-full-exclusive-control-over-the-hsms-with-dedicated-hsms"></a>Q : Le client a-t-il un contrôle exclusif total sur les HSM avec les HSM dédiés ?

Oui. Chaque appliance HSM est entièrement dédiée à un client unique et personne d’autre ne dispose d’un contrôle administratif une fois qu’elle est provisionnée et que le mot de passe administrateur a été changé.

### <a name="q-what-level-of-access-does-microsoft-have-to-my-hsm"></a>Q : Quel niveau d’accès Microsoft a-t-il sur mon HSM ?

Microsoft ne dispose d’aucun contrôle administratif ou cryptographique sur le module HSM. Microsoft dispose d’un accès de niveau superviseur via la connexion de port série pour récupérer des données de télémétrie de base comme la température et l’intégrité de composant. Microsoft peut ainsi adresser des notifications proactives en cas de problèmes d’intégrité. Si nécessaire, les client peut désactiver ce compte.

### <a name="q-what-is-the-tenantadmin-account-microsoft-uses-i-am-used-to-the-admin-user-being-admin-on-safenet-hsms"></a>Q : Qu’est-ce que le compte « tenantadmin » utilisé par Microsoft. Je suis habitué à l’utilisateur administrateur « admin » sur les appareils HSM SafeNet ?

L’appareil HSM est livré avec un utilisateur administrateur par défaut et son mot de passe par défaut habituel. Microsoft ne voulait pas utiliser des mots de passe par défaut lorsqu’un appareil est dans un pool en attente d’approvisionnement par les clients. Cela ne respecterait pas nos exigences de sécurité strictes. Pour cette raison, nous définissons un mot de passe fort, qui est abandonné au moment du provisionnement. En outre, lors de l’approvisionnement, nous créons un autre utilisateur avec le rôle d’administrateur, appelé « tenantadmin ». Cet utilisateur a le mot de passe par défaut, et les clients remplacent cette valeur dès la première connexion à l’appareil qui vient d’être approvisionné. Ce processus permet de garantir une sécurité optimale et de tenir notre promesse d’un contrôle administratif exclusif pour nos clients. Il convient de noter que l’utilisateur « tenantadmin » peut être utilisé pour réinitialiser le mot de passe administrateur si un client souhaite utiliser ce compte. 

### <a name="q-can-microsoft-or-anyone-at-microsoft-access-keys-in-my-dedicated-hsm"></a>Q : Microsoft, ou toute personne travaillant chez Microsoft, peut-il accéder aux clés de mon HSM dédié ?

Non. Microsoft n’a pas accès aux clés stockées dans le module HSM dédié alloué par le client.

### <a name="q-can-i-upgrade-softwarefirmware-on-hsms-allocated-to-me"></a>Q : Puis-je mettre à niveau les logiciels/microprogrammes sur les HSM qui me sont alloués ?

Pour une prise en charge optimale, Microsoft recommande vivement de ne pas à mettre à niveau les logiciels/microprogrammes sur les modules HSM. Cependant, le client dispose d’un contrôle administratif total, ce qui lui permet de mettre à niveau les logiciels/microprogrammes s’il a besoin de fonctionnalités spécifiques de versions différentes du microprogramme. Avant d’apporter des modifications, il convient d’en saisir les répercussions, car cela pourrait par exemple avoir un effet sur le statut de certification FIPS. 

### <a name="q-how-do-i-manage-dedicated-hsm"></a>Q : Comment gérer un HSM dédié ?

Vous pouvez gérer les modules HSM dédiés en y accédant avec SSH.

### <a name="q-how-do-i-manage-partitions-on-the-dedicated-hsm"></a>Q : Comment gérer les partitions sur le HSM dédié ?

Le logiciel client HSM Gemalto sert à gérer les modules HSM et les partitions.

### <a name="q-how-do-i-monitor-my-hsm"></a>Q : Comment superviser mon HSM ?

Un client dispose d’un accès complet aux journaux d’activité HSM via syslog et SNMP. Pour recevoir les journaux d’activité ou les événements des modules HSM, le client doit configurer un serveur syslog ou un serveur SNMP.

### <a name="q-can-i-get-full-access-log-of-all-hsm-operations-from-dedicated-hsm"></a>Q : Puis-je obtenir un journal d’accès complet de toutes les opérations HSM du HSM dédié ?

Oui. Vous pouvez envoyer des journaux d’activité de l’appliance HSM vers un serveur syslog.

## <a name="high-availability"></a>Haute disponibilité

### <a name="q-is-it-possible-to-configure-high-availability-in-the-same-region-or-across-multiple-regions"></a>Q : Puis-je configurer la haute disponibilité dans une même région ou dans plusieurs régions ?

Oui. La haute disponibilité s’installe et se configure dans le logiciel client HSM fourni par Gemalto. Les modules HSM situés sur un même réseau virtuel ou sur différents réseaux virtuels d’une même région ou de régions différentes, ou les modules HSM locaux connectés à un réseau virtuel utilisant un VPN site à site ou point à point peuvent être ajoutés à une même configuration de haute disponibilité. Il convient de noter que cette opération synchronise uniquement le matériel principal et les éléments de configuration non spécifiques, tels que les rôles.

### <a name="q-can-i-add-hsms-from-my-on-premises-network-to-a-high-availability-group-with-azure-dedicated-hsm"></a>Q : Le service HSM dédié Azure me permet-il d’ajouter des modules HSM de mon réseau local à un groupe de haute disponibilité ?

Oui. Ils doivent répondre aux exigences de haute disponibilité pour SafeNet Luna Network HSM 7.

### <a name="q-can-i-add-luna-56-hsms-from-on-premises-networks-to-a-high-availability-group-with-azure-dedicated-hsm"></a>Q : Le service HSM dédié Azure me permet-il d’ajouter des modules HSM Luna 5/6 de réseaux locaux à un groupe de haute disponibilité ?

Non.

### <a name="q-how-many-hsms-can-i-add-to-the-same-high-availability-configuration-from-one-single-application"></a>Q : Combien de HSM puis-je ajouter à une même configuration de haute disponibilité pour une seule application ?

16 membres d’un groupe à haute disponibilité ont été soumis à un test de limitation et ont obtenu d’excellents résultats.

## <a name="support"></a>Support

### <a name="q-what-is-the-sla-for-dedicated-hsm-service"></a>Q : Quel est le contrat SLA du service HSM dédié ?

Il n’existe aucune garantie de temps d’activité spécifique pour le service HSM dédié. Cependant, comme Microsoft garantit un accès de niveau réseau à l’appareil, ce sont les contrats SLA de gestion réseau Azure standard qui s’appliquent.

### <a name="q-how-are-the-hsms-used-in-azure-dedicated-hsm-protected"></a>Q : Comment les modules HSM utilisés dans le service HSM dédié Azure sont-ils protégés ?

Les centres de données Azure disposent de contrôles de sécurité physique et procédurale étendus. Par ailleurs, les modules HSM dédiés sont hébergés dans une zone du centre de données plus restrictive en termes d’accès. Pour renforcer la sécurité, ces zones offrent des contrôles d’accès physique supplémentaires et des caméras de surveillance vidéo.

### <a name="q-what-happens-if-there-is-a-security-breach-or-hardware-tampering-event"></a>Q : Que se passe-t-il en cas de violation de la sécurité ou d’altération matérielle ?

Le service HSM dédié utilise 7 des appliances SafeNet Network HSM 7. Ces appliances prennent en charge la détection des altérations physiques et logiques. En cas d’altération, les modules HSM sont automatiquement mis à zéro.

### <a name="q-how-do-i-ensure-that-keys-in-my-dedicated-hsms-are-not-lost-due-to-error-or-a-malicious-insider-attack"></a>Q : Comment éviter la perte des clés stockées dans les HSM dédiés à la suite d’une erreur ou d’une attaque interne malveillante ?

Il est vivement recommandé d’utiliser une unité de sauvegarde HSM locale pour sauvegarder de façon périodique et régulière les modules HSM à des fins de récupération d’urgence. Vous devrez utiliser une connexion VPN pair à pair ou site à site pour accéder à une station de travail locale connectée à une unité de sauvegarde HSM.

### <a name="q-how-do-i-get-support-for-dedicated-hsm"></a>Q : Comment bénéficier du support pour un HSM dédié ?

La prise en charge est fournie par Microsoft et Gemalto.  Si vous rencontrez un problème avec le matériel ou l’accès réseau, ouvrez une demande de support auprès de Microsoft, et si vous rencontrez un problème avec la configuration HSM, les logiciels et le développement d’applications, ouvrez une demande de support auprès de Gemalto. Si vous rencontrez un problème indéterminé, ouvrez une demande de support auprès de Microsoft, puis impliquez Gemalto si nécessaire. 

### <a name="q-how-do-i-get-the-client-software-documentation-and-access-to-integration-guidance-for-the-safenet-luna-7-hsm"></a>Q : Comment obtenir le client logiciel, la documentation et l’accès à des conseils d’intégration pour le module HSM SafeNet Luna 7 ?

Après vous être inscrit pour le service, un ID de client Gemalto vous est fourni pour l’inscription sur le portail de support client Gemalto. Cela active l’accès à tous les logiciels et à la documentation, ainsi que l’activation des demandes de support directement auprès de Gemalto.

### <a name="q-if-there-is-a-security-vulnerability-found-and-a-patch-is-released-by-gemalto-who-is-responsible-for-upgradingpatching-osfirmware"></a>Q : Si une faille de sécurité est détectée et qu’un correctif est publié par Gemalto, qui est chargé de la mise à niveau/mise à jour corrective du système d’exploitation ou du microprogramme ?

Microsoft n’a pas la possibilité de se connecter aux modules HSM alloués aux clients. Les clients doivent eux-mêmes mettre à niveau et corriger leurs modules HSM.

### <a name="q-what-if-i-need-to-reboot-my-hsm"></a>Q : Que se passe-t-il si j’ai besoin de redémarrer mon module HSM ?

Le module HSM dispose d’une option de redémarrage en ligne de commande. Toutefois, nous rencontrons des problèmes où le redémarrage cesse de répondre par intermittence. C’est pourquoi, pour un redémarrage plus sûr, nous vous recommandons d’ouvrir une demande de support auprès de Microsoft pour que l’appareil soit physiquement redémarré. 

## <a name="cryptography-and-standards"></a>Chiffrement et standards

### <a name="q-is-it-safe-to-store-encryption-keys-for-my-most-important-data-in-dedicated-hsm"></a>Q : Le stockage des clés de chiffrement dans un HSM dédié est-il sécurisé pour mes données les plus importantes ?

Oui, le service HSM dédié provisionne les appliances SafeNet Network HSM 7 qui utilisent des modules HSM certifiés FIPS 140-2 de niveau 3. 

### <a name="q-what-cryptographic-keys-and-algorithms-are-supported-by-dedicated-hsm"></a>Q : Quels algorithmes et quelles clés de chiffrement un HSM dédié prend-il en charge ?

Le service HSM dédié provisionne des appliances SafeNet Network HSM 7. Elles prennent en charge un large éventail d’algorithmes et de types de clé de chiffrement, notamment : Prise en charge complète de la suite B

* Asymétrique :
  * RSA
  * DSA
  * Diffie-Hellman
  * Courbe elliptique
  * Chiffrement (ECDSA, ECDH, Ed25519, ECIES) avec courbes nommées, définies par l’utilisateur et Brainpool, KCDSA
* Symétrique :
  * AES-GCM
  * Triple DES
  * DES
  * ARIA, SEED
  * RC2
  * RC4
  * RC5
  * CAST
  * Code de hachage/Synthèse du message/HMAC : SHA-1, SHA-2, SM3
  * Dérivation de clé : Mode de compteur sp800-108
  * Wrapping de clé : SP800-38F
  * Génération de nombres aléatoires : DRGB agréé par FIPS 140-2 (mode CTR SP 800-90), conforme à BSI DRG.4

### <a name="q-is-dedicated-hsm-fips-140-2-level-3-validated"></a>Q : Le HSM dédié est-il certifié FIPS 140-2 niveau 3 ?

Oui. Le service HSM dédié provisionne les appliances SafeNet Network HSM 7 qui utilisent des modules HSM certifiés FIPS 140-2 de niveau 3.

### <a name="q-what-do-i-need-to-do-to-make-sure-i-operate-dedicated-hsm-in-fips-140-2-level-3-validated-mode"></a>Q : Que faire pour que le HSM dédié fonctionne en mode certifié FIPS 140-2 niveau 3 ?

Le service HSM dédié provisionne des appliances SafeNet Luna Network HSM 7. Ces appliances utilisent des modules HSM certifiés FIPS 140-2 de niveau 3. La configuration, le système d’exploitation et le microprogramme déployés par défaut sont aussi certifiés FIPS. Vous n’avez rien à faire pour bénéficier de la conformité à FIPS 140-2 de niveau 3.

### <a name="q-how-does-a-customer-ensure-that-when-an-hsm-is-deprovisioned-all-the-key-material-is-wiped-out"></a>Q : Que doit faire un client pour être certain que les clés sont effacées d’un module HSM déprovisionné ?

Avant de demander le déprovisionnement, le client doit mettre le module HSM à zéro à l’aide des outils clients HSM Gemalto.

## <a name="performance-and-scale"></a>Performances et mise à l’échelle

### <a name="q-how-many-cryptographic-operations-are-supported-per-second-with-dedicated-hsm"></a>Q : Combien d’opérations de chiffrement par seconde sont prises en charge avec un HSM dédié ?

Le service HSM dédié provisionne des appliances SafeNet Network HSM 7 (modèle A790). Voici un résumé des performances maximales pour certaines opérations : 

* RSA-2048 : 10 000 transactions par seconde
* ECC P256 : 20 000 transactions par seconde
* AES-GCM : 17 000 transactions par seconde

### <a name="q-how-many-partitions-can-be-created-in-dedicated-hsm"></a>Q : Combien de partitions peut-on créer dans un HSM dédié ?

Le modèle SafeNet Luna HSM 7 790 utilisé inclut une licence pour 10 partitions dans le coût du service. L’appareil a une limite de 100 partitions. L’ajout de partitions jusqu’à cette limite entraîne des coûts de licences supplémentaires et nécessite l’installation d’un nouveau fichier de licence sur l’appareil.

### <a name="q-how-many-keys-can-be-supported-in-dedicated-hsm"></a>Q : Combien de clés un HSM dédié prend-il en charge ?

Le nombre maximal de clés dépend de la mémoire disponible. Le modèle SafeNet Luna 7 A790 en cours d’utilisation a 32 Mo de mémoire. Les chiffres suivants valent aussi pour les paires de clés si vous utilisez des clés asymétriques.

* RSA-2048 - 19 000
* ECC-P256 - 91 000

La capacité varie en fonction des attributs de clé spécifiques définis dans le modèle de génération de clés et du nombre de partitions.
