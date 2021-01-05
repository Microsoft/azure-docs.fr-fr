---
title: Contrôles de l’exemple de blueprint Media
description: Correspondance des contrôles des exemples de blueprint Media. Chaque contrôle est mis en correspondance avec une ou plusieurs définitions Azure Policy qui simplifient l’évaluation.
ms.date: 08/13/2020
ms.topic: sample
ms.openlocfilehash: ec4850b4e3061d267610bfd9c2c0e33c670472b4
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91931867"
---
# <a name="control-mapping-of-the-media-blueprint-sample"></a>Correspondance des contrôles de l’exemple de blueprint Media

L’article suivant explique en détails comment l’exemple de blueprint Media d’Azure Blueprints correspond aux contrôles Media. Pour plus d’informations sur les contrôles, consultez [Media](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

Les correspondances suivantes concernent les contrôles **Media**. Utilisez le volet de navigation de droite pour accéder directement à la correspondance d’un contrôle spécifique. De nombreux contrôles mappés sont implémentés avec une initiative [Azure Policy](../../../policy/overview.md). Pour examiner l’initiative complète, ouvrez **Stratégie** dans le portail Azure et sélectionnez la page **Définitions**. Ensuite, recherchez et sélectionnez l’initiative de stratégie intégrée **\[Préversion\] : Auditer les contrôles Media**.

> [!IMPORTANT]
> Chaque contrôle ci-dessous est associé à une ou plusieurs définitions [Azure Policy](../../../policy/overview.md). Ces stratégies peuvent vous aider à [évaluer la conformité](../../../policy/how-to/get-compliance-data.md) avec le contrôle ; toutefois, il n’existe pas souvent de correspondance un-à-un ou parfaite entre un contrôle et une ou plusieurs stratégies. Ainsi, la **conformité** dans Azure Policy fait uniquement référence aux stratégies elles-mêmes ; cela ne garantit pas que vous êtes entièrement conforme à toutes les exigences d’un contrôle. En outre, la norme de conformité comprend des contrôles qui ne sont traités par aucune définition Azure Policy pour l’instant. Par conséquent, la conformité dans Azure Policy n’est qu’une vue partielle de l’état de conformité global. Les associations entre les contrôles et les définitions Azure Policy pour cet exemple de blueprint de conformité peuvent changer au fil du temps. Pour afficher l’historique des changements, consultez l’[historique des validations GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/media/control-mapping.md).

## <a name="access-control"></a>Contrôle d’accès

### <a name="ac-11--ensure-no-root-access-key-exists"></a>AC-1.1 - Veiller à ce qu’aucune clé d’accès racine n’existe.

- \[Préversion\] : Prérequis de déploiement pour l’audit des machines virtuelles Windows qui ne contiennent pas les certificats spécifiés dans la racine de confiance

### <a name="ac-12---passwords-pins-and-tokens-must-be-protected"></a>AC-1.2 - Les mots de passe, les codes PIN et les jetons doivent être protégés.

- \[Préversion\] : Déployer des prérequis pour auditer les machines virtuelles Windows qui ne limitent pas la longueur minimale du mot de passe à 14 caractères

### <a name="ac-18---shared-account-access-is-prohibited"></a>AC-1.8 - L’accès au compte partagé est interdit.

- Toutes les règles d’autorisation, sauf RootManageSharedAccessKey, doivent être supprimées de l’espace de noms Service Bus

### <a name="ac-19--system-must-restrict-access-to-authorized-users"></a>AC-1.9 - Le système doit restreindre l’accès aux utilisateurs autorisés.

- Auditer l'accès réseau non restreint aux comptes de stockage

### <a name="ac--114--system-must-enforce-access-rights"></a>AC-1.14 - Le système doit appliquer les droits d’accès.

- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Attribution de droits utilisateur »

### <a name="ac--115--prevent-unauthorized-access-to-security-relevant-information-or-functions"></a>AC-1.15 - Empêcher tout accès non autorisé aux informations ou fonctions relatives à la sécurité.

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Paramètres système »

### <a name="ac-1-21---separation-of-duties-must-be-enforced-through-appropriate-assignment-of-role"></a>AC-1-21 - Une attribution de rôle appropriée doit assurer la séparation des tâches.

- [Préversion\] : Le contrôle d’accès en fonction du rôle (RBAC) doit être utilisé sans Kubernetes Services.

### <a name="ac-140--ensure-that-systems-are-not-connecting-trusted-network-and-untrusted-networks-at-the-same-time"></a>AC-1.40 - Veiller à ce que les systèmes ne connectent pas un réseau approuvé et des réseaux non approuvés en même temps.

- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Accès réseau »

### <a name="ac-142--ac--143---remote-access-for-non-employees-must-be-restricted-to-allow-access-only-to-specifically-approved-information-systems"></a>AC-1.42 et AC-1.43 - L’accès à distance des utilisateurs non employés doit être limité aux systèmes d’information spécifiquement approuvés.

- \[Préversion\] : Afficher les résultats d’audit des machines virtuelles Linux qui autorisent les connexions à distance à partir des comptes sans mot de passe

### <a name="ac-150--log-security-related-events-for-all-information-system-components"></a>AC-1.50 - Journaliser les événements liés à la sécurité pour tous les composants du système d’information.

- Les journaux de diagnostic dans Logic Apps doivent être activés.

### <a name="ac-154--ensure-multi-factor-authentication-mfa-is-enabled-for-all-cloud-console-users"></a>AC-1.54 - Veiller à ce que l’authentification multifacteur (MFA) soit activée pour tous les utilisateurs de la console cloud.

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- MFA (Multi-Factor Authentication) doit être activé pour tous les comptes de l'abonnement disposant de privilèges d'accès en écriture pour éviter une violation des comptes ou des ressources.

## <a name="auditing--logging"></a>Audit et journalisation

### <a name="al-21--successful-and-unsuccessful-events-must-be-logged"></a>AL-2.1 - Les événements ayant réussi et ceux ayant échoué doivent être journalisés.

- Les journaux de diagnostic dans les services Search doivent être activés.

### <a name="al--216---network-devicesinstances-must-log-any-event-classified-as-a-critical-security-event-by-that-network-deviceinstance-elbs-web-application-firewalls-etc"></a>AL-2.16 - Les instances/appareils réseau doivent journaliser tout événement classifié comme événement de sécurité critique par l’instance/l’appareil réseau concerné (ELB, pare-feu d’applications web, etc.)

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Comptes »

### <a name="al-217--serversinstances-must-log-any-event-classified-as-a-critical-security-event-by-that-serverinstance"></a>AL-2.17 - Les serveurs/instances doivent journaliser tout événement classifié comme événement de sécurité critique par le serveur/l’instance concerné.

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Comptes »

### <a name="al-219---domain-events-must-log-any-event-classified-as-a-critical-or-high-security-event-by-the-domain-management-software"></a>AL-2.19 - Les événements de domaine doivent journaliser tout événement classifié comme événement de sécurité majeur ou critique par le logiciel de gestion de domaine.

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Comptes »
- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Client réseau Microsoft »

### <a name="al-220--domain-events-must-log-any-event-classified-as-a-critical-security-event-by-domain-security-controls"></a>AL-2.20 - Les événements de domaine doivent journaliser tout événement classifié comme événement de sécurité critique par des contrôles de sécurité de domaine

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Comptes »

### <a name="al-221--domain-events-must-log-any-access-or-changes-to-the-domain-log"></a>AL-2.21 - Les événements de domaine doivent journaliser tout accès au journal de domaine et toute modification de ce dernier.

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Console de récupération »

## <a name="cryptographic-controls"></a>Contrôles de chiffrement

### <a name="cc-42--applications-and-systems-must-use-current-cryptographic-solutions-for-protecting-data"></a>CC-4.2 - Les applications et les systèmes doivent utiliser les solutions de chiffrement actuelles pour la protection des données.

- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL
- Le chiffrement transparent des données doit être activé pour protéger les données au repos et respecter les conditions de conformité requises

### <a name="cc-45--digital-certificates-must-be-signed-by-an-approved-certificate-authority"></a>CC-4.5 - Les certificats numériques doivent être signés par une autorité de certification approuvée.

- \[Préversion\] : Afficher les résultats de l’audit des machines virtuelles Windows qui contiennent des certificats arrivant à expiration dans le nombre de jours spécifié

### <a name="cc-46--digital-certificates-must-be-uniquely-assigned-to-a-user-or-device"></a>CC-4.6 - Les certificats numériques doivent être attribués à un utilisateur ou à un appareil de manière unique.

- \[Préversion\] : Prérequis de déploiement pour l’audit des machines virtuelles Windows qui contiennent des certificats arrivant à expiration dans le nombre de jours spécifié

### <a name="cc-47--cryptographic-material-must-be-stored-to-enable-decryption-of-the-records-for-the-length-of-time-the-records-are-retained"></a>CC-4.7 - Pour permettre le déchiffrement des enregistrements, le matériel de chiffrement doit être stocké tout au long de la période de conservation des enregistrements.

- Le chiffrement de disque doit être appliqué sur les machines virtuelles
- Les machines virtuelles sans chiffrement de disque activé seront supervisées par Azure Security Center en tant que recommandation

### <a name="cc-48--secret-and-private-keys-must-be-stored-securely"></a>CC-4.8 - Les clés secrètes et privées doivent être stockées de manière sécurisée.

- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL
- Le chiffrement transparent des données doit être activé pour protéger les données au repos et respecter les conditions de conformité requises

## <a name="change--config-management"></a>Gestion des changements et de la configuration

### <a name="cm-52--only-authorized-users-may-implement-approved-changes-on-the-system"></a>CM-5.2 - Seuls les utilisateurs autorisés peuvent implémenter des changements approuvés sur le système.

- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation

### <a name="cm-512--maintain-an-up-to-date-complete-accurate-and-readily-available-baseline-configuration-of-the-information-system"></a>CM-5.12 - Disposer d’une configuration de base à jour, complète, précise et disponible du système d’information.

- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation

### <a name="cm-513--employ-automated-tools-to-maintain-a-baseline-configuration-of-the-information-system"></a>CM-5.13 - Utiliser des outils automatisés pour tenir à jour une configuration de base du système d’information.

- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation

### <a name="cm-514--identify-and-disable-unnecessary-andor-non-secure-functions-ports-protocols-and-services"></a>CM-5.14 - Identifier et désactiver les fonctions, ports, protocoles et services non nécessaires et/ou non sécurisés.

- Les interfaces réseau doivent désactiver le transfert IP
- \[Préversion\] : Le transfert IP doit être désactivé sur votre machine virtuelle.

### <a name="cm-519--monitor-changes-to-the-security-configuration-settings"></a>CM-5.19 - Surveiller les modifications apportées aux paramètres de configuration de la sécurité.

- Déployer les paramètres de diagnostic pour les groupes de sécurité réseau

### <a name="cm-522--ensure-that-only-authorized-software-and-updates-are-installed-on-company-systems"></a>CM-5.22 - Veiller à ce que seuls les logiciels et mises à jour autorisés soient installés sur les systèmes de l’entreprise.

- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système de sécurité manquantes sur vos serveurs seront supervisées par Azure Security Center en tant que recommandation

## <a name="identity--authentication"></a>Identité et authentification

### <a name="ia-71--user-accounts-must-be-uniquely-assigned-to-individuals-for-access-to-information-that-is-not-classified-as-public-account-ids-must-be-constructed-using-a-standardized-logical-format"></a>IA-7.1 - Les comptes d’utilisateur doivent être attribués de manière unique pour l’accès aux informations qui ne sont pas classifiées comme publiques. Les ID de compte doivent être créés suivant un format logique standardisé.

- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes avec des autorisations de type propriétaire doivent être supprimés de votre abonnement pour empêcher un accès non contrôlé.

## <a name="network-security"></a>Sécurité réseau

### <a name="ns-92--access-to-network-device-management-functionality-is-restricted-to-authorized-users"></a>NS-9.2 - L’accès à la fonctionnalité de gestion des appareils réseau est limité aux utilisateurs autorisés.

- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Accès réseau »

### <a name="ns-93--all-network-devices-must-be-configured-using-their-most-secure-configurations"></a>NS-9.3 - Les configurations les plus sécurisées doivent être appliquées à tous les appareils réseau.

- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Options de sécurité - Accès réseau »

### <a name="ns-95--all-network-connections-to-a-system-through-a-firewall-must-be-approved-and-audited-on-a-regular-basis"></a>NS-9.5 - Toutes les connexions réseau à un système par le biais d’un pare-feu doivent être approuvées et auditées régulièrement.

- \[Préversion\] : Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Propriétés du pare-feu Windows »

### <a name="ns-97--appropriate-controls-must-be-present-at-any-boundary-between-a-trusted-network-and-any-untrusted-or-public-network"></a>NS-9.7 - Des contrôles appropriés doivent être appliqués à chaque limite entre un réseau approuvé et tout réseau non approuvé ou public.

- \[Préversion\] : Prérequis de déploiement pour l’audit des configurations de machines virtuelles Windows dans « Propriétés du pare-feu Windows »

## <a name="security-planning"></a>Planification de la sécurité

### <a name="sp-113--threats-must-be-identified-that-could-negatively-impact-the-confidentiality-integrity-or-availability-of-company-information-and-content-along-with-the-likelihood-of-their-occurrence"></a>SP-11.3 - Les menaces pouvant avoir un impact négatif sur la confidentialité, l’intégrité ou la disponibilité des informations et contenus de l’entreprise doivent être identifiées, et la probabilité d’occurrence de ces menaces doit être déterminée.

- Les types Advanced Threat Protection doivent être définis sur « Tous » dans les paramètres Advanced Data Security de l’instance gérée SQL.

### <a name="security-continuity"></a>Continuité de la sécurité

## <a name="sc-125--data-in-long-term-storage-must-be-accessible-throughout-the-retention-period-and-protected-against-media-degradation-and-technology-changes"></a>SC-12.5 - Les données stockées dans un système de stockage à long terme doivent être accessibles tout au long de leur période de conservation et doivent être préservées en cas de dégradation des médias ou de changement de technologie.

- Les serveurs SQL doivent être configurés avec une période de conservation d’audit supérieure à 90 jours.
- Auditez les serveurs SQL configurés avec une période de conservation d’audit inférieure à 90 jours.

## <a name="system-integrity"></a>Intégrité du système

### <a name="si-143--only-authorized-personnel-may-monitor-network-and-user-activities"></a>SI-14.3 - Seul le personnel autorisé peut superviser les activités des utilisateurs et du réseau.

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- Supervisez les résultats de l'analyse des vulnérabilités et les recommandations sur la correction des vulnérabilités liées aux bases de données.

### <a name="si-144--internet-facing-systems-must-have-intrusion-detection"></a>SI-14.4 - Les systèmes connectés à Internet doivent disposer d’un mécanisme de détection d’intrusion.

- Déployer la détection de menaces sur les serveurs SQL

### <a name="si-1413--standardized-centrally-managed-anti-malware-software-should-be-implemented-across-the-company"></a>SI-14.13 - Un logiciel anti-programme malveillant standardisé et géré de manière centralisée doit être implémenté à l’échelle de l’entreprise.

- Déployer l’extension Microsoft IaaSAntimalware par défaut pour Windows Server

### <a name="si-1414--anti-malware-software-must-scan-computers-and-media-weekly-at-a-minimum"></a>SI-14.14 - Les ordinateurs et médias doivent être analysés par un logiciel anti-programme malveillant au minimum une fois par semaine.

- Déployer l’extension Microsoft IaaSAntimalware par défaut pour Windows Server

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

### <a name="vm-154--ensure-that-applications-are-scanned-for-vulnerabilities-on-a-monthly-basis"></a>VM-15.4 - Veiller à ce que les vulnérabilités des applications soient analysées sur une base mensuelle.

- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Auditez les vulnérabilités de système d’exploitation dans vos groupes de machines virtuelles identiques pour les protéger des attaques.

### <a name="vm-155--ensure-that-vulnerabilities-are-identified-paired-to-threats-and-evaluated-for-risk"></a>VM-15.5 - Veiller à ce que les vulnérabilités soient identifiées, à ce que les menaces correspondantes soient déterminées et à ce que le risque encouru soit évalué.

- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Auditez les vulnérabilités de système d’exploitation dans vos groupes de machines virtuelles identiques pour les protéger des attaques.

### <a name="vm-156--ensure-that-identified-vulnerabilities-have-been-remediated-within-a-mutually-agreed-upon-timeline"></a>VM-15.6 - Veiller à ce que les vulnérabilités identifiées soient corrigées selon un calendrier convenu mutuellement.

- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Auditez les vulnérabilités de système d’exploitation dans vos groupes de machines virtuelles identiques pour les protéger des attaques.

### <a name="vm-157--access-to-and-use-of-vulnerability-management-systems-must-be-restricted-to-authorized-personnel"></a>VM-15.7- L’accès aux systèmes de gestion des vulnérabilités et l’utilisation de ces systèmes doivent être limités au personnel autorisé.

- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Auditez les vulnérabilités de système d’exploitation dans vos groupes de machines virtuelles identiques pour les protéger des attaques.

> [!NOTE]
> La disponibilité des définitions Azure Policy peut varier dans Azure Government et dans d’autres clouds nationaux. 

## <a name="next-steps"></a>Étapes suivantes

Vous avez pris connaissance de la correspondance des contrôles de l’exemple de blueprint Media. Continuez avec les articles suivants pour découvrir une vue d’ensemble de cet exemple et pour savoir comment le déployer :

> [!div class="next step action"]
> [Blueprint Media - Correspondance des contrôles](./control-mapping.md)
> [Blueprint Media - Étapes de déploiement](./deploy.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
