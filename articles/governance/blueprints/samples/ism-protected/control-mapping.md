---
title: Contrôles de l’exemple de blueprint Australian Government ISM PROTECTED
description: Correspondance des contrôles de l’exemple de blueprint Australian Government ISM PROTECTED. Chaque contrôle est mis en correspondance avec une ou plusieurs définitions Azure Policy qui simplifient l’évaluation.
ms.date: 09/11/2020
ms.topic: sample
ms.openlocfilehash: 9f036c7e1a749043a0b81717b75202a5c6c37c8c
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932224"
---
# <a name="control-mapping-of-the-australian-government-ism-protected-blueprint-sample"></a>Correspondance des contrôles de l’exemple de blueprint Australian Government ISM PROTECTED

L’article suivant explique en détail comment l’exemple de blueprint Australian Government ISM PROTECTED Azure Blueprints est mis en correspondance avec les contrôles ISM PROTECTED. Pour plus d’informations sur les contrôles, consultez [ISM PROTECTED](https://www.cyber.gov.au/ism).

Les correspondantes suivantes concernent les contrôles **ISM PROTECTED**. Utilisez le volet de navigation de droite pour accéder directement à la correspondance d’un contrôle spécifique. De nombreux contrôles mappés sont implémentés avec une initiative [Azure Policy](../../../policy/overview.md). Pour examiner l’initiative complète, ouvrez **Stratégie** dans le portail Azure et sélectionnez la page **Définitions**. Ensuite, recherchez et sélectionnez l’initiative de stratégie intégrée **\[Préversion\] : Auditer les contrôles Australian Government ISM PROTECTED et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit**.

> [!IMPORTANT]
> Chaque contrôle ci-dessous est associé à une ou plusieurs définitions [Azure Policy](../../../policy/overview.md). Ces stratégies peuvent vous aider à [évaluer la conformité](../../../policy/how-to/get-compliance-data.md) avec le contrôle ; toutefois, il n’existe pas souvent de correspondance un-à-un ou parfaite entre un contrôle et une ou plusieurs stratégies. Ainsi, la **conformité** dans Azure Policy fait uniquement référence aux stratégies elles-mêmes ; cela ne garantit pas que vous êtes entièrement conforme à toutes les exigences d’un contrôle. En outre, la norme de conformité comprend des contrôles qui ne sont traités par aucune définition Azure Policy pour l’instant. Par conséquent, la conformité dans Azure Policy n’est qu’une vue partielle de l’état de conformité global. Les associations entre les contrôles et les définitions Azure Policy pour cet exemple de blueprint de conformité peuvent changer au fil du temps. Pour afficher l’historique des changements, consultez l’[historique des validations GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/ism-protected/control-mapping.md).


## <a name="location-constraints"></a>Contraintes d’emplacement

Ce blueprint vous permet de limiter l’emplacement du déploiement de toutes les ressources et de tous les groupes de ressources aux régions « Australie Centre », « Australie Centre2 », « Australie Est » et « Australie Sud-Est » en attribuant les définitions Azure Policy suivantes :

- Lieux autorisés (ont été codés en dur sur « Australie Centre », « Australie Centre2 », « Australie Est » et « Australie Sud-Est »)
- Lieux autorisés pour les groupes de ressources (ont été codés en dur sur « Australie Centre », « Australie Centre2 », « Australie Est » et « Australie Sud-Est »)

## <a name="guidelines-for-personnel-security---access-to-systems-and-their-resources"></a>Instructions pour la sécurité personnelle - Accès aux systèmes et à leurs ressources

### <a name="0414-personnel-granted-access-to-a-system-and-its-resources-are-uniquely-identifiable"></a>0414 Le personnel autorisé à accéder à un système et à ses ressources est identifiable de façon unique

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement

### <a name="1503-standard-access-to-systems-applications-and-data-repositories-is-limited-to-that-required-for-personnel-to-undertake-their-duties"></a>1503 L’accès standard aux systèmes, aux applications et aux référentiels de données est limité à celui requis par le personnel pour assumer ses fonctions.

- Trois propriétaires au plus doivent être désignés pour votre abonnement
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Afficher les résultats de l’audit des machines virtuelles Windows dans lesquelles le groupe Administrateurs contient l’un des membres spécifiés
- Prérequis de déploiement pour auditer les machines virtuelles Windows dans lesquelles le groupe Administrateurs contient un membre spécifié

### <a name="1507-privileged-access-to-systems-applications-and-data-repositories-is-validated-when-first-requested-and-revalidated-on-an-annual-or-more-frequent-basis"></a>1507 L’accès privilégié aux systèmes, aux applications et aux référentiels de données est validé lors de la première demande, puis revalidé sur une base annuelle ou plus fréquente

- Afficher les résultats de l’audit des machines virtuelles Windows dans lesquelles le groupe Administrateurs contient l’un des membres spécifiés
- Prérequis de déploiement pour auditer les machines virtuelles Windows dans lesquelles le groupe Administrateurs contient un membre spécifié

### <a name="1508-privileged-access-to-systems-applications-and-data-repositories-is-limited-to-that-required-for-personnel-to-undertake-their-duties"></a>1508 L’accès privilégié aux systèmes, aux applications et aux référentiels de données est limité à celui requis par le personnel pour assumer ses fonctions.

- Trois propriétaires au plus doivent être désignés pour votre abonnement
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Afficher les résultats de l’audit des machines virtuelles Windows dans lesquelles le groupe Administrateurs contient l’un des membres spécifiés
- Prérequis de déploiement pour auditer les machines virtuelles Windows dans lesquelles le groupe Administrateurs contient un membre spécifié
- Le contrôle d’accès réseau juste-à-temps doit être appliqué sur les machines virtuelles

### <a name="0415-the-use-of-shared-user-accounts-is-strictly-controlled-and-personnel-using-such-accounts-are-uniquely-identifiable"></a>0415 L’utilisation de comptes d’utilisateur partagés est strictement contrôlée, et le personnel qui utilise ces comptes est identifiable de façon unique

- Afficher les résultats de l’audit des machines virtuelles Windows dans lesquelles le groupe Administrateurs contient l’un des membres spécifiés
- Prérequis de déploiement pour auditer les machines virtuelles Windows dans lesquelles le groupe Administrateurs contient un membre spécifié

### <a name="0445-privileged-users-are-assigned-a-dedicated-privileged-account-to-be-used-solely-for-tasks-requiring-privileged-access"></a>0445 Un compte privilégié dédié, utilisable exclusivement pour les tâches nécessitant un accès privilégié, est attribué aux utilisateurs privilégiés

- Afficher les résultats de l’audit des machines virtuelles Windows dans lesquelles le groupe Administrateurs contient l’un des membres spécifiés
- Prérequis de déploiement pour auditer les machines virtuelles Windows dans lesquelles le groupe Administrateurs contient un membre spécifié

### <a name="0430-access-to-systems-applications-and-data-repositories-is-removed-or-suspended-on-the-same-day-personnel-no-longer-have-a-legitimate-requirement-for-access"></a>0430 L’accès aux systèmes, aux applications et aux référentiels de données est supprimé ou suspendu le jour même où le personnel n’a plus besoin d’un accès légitime

- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

### <a name="0441-when-personnel-are-granted-temporary-access-to-a-system-effective-security-controls-are-put-in-place-to-restrict-their-access-to-only-information-required-for-them-to-undertake-their-duties"></a>0441 Lorsque le personnel reçoit un accès temporaire à un système, des contrôles de sécurité efficaces sont mis en place pour limiter son accès uniquement aux informations nécessaires à l’accomplissement de ses tâches.

- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement
- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

## <a name="guidelines-for-system-hardening---operating-system-hardening"></a>Instructions pour le durcissement du système - Durcissement du système d’exploitation

### <a name="1407-the-latest-version-n-or-n-1-version-of-an-operating-system-is-used-for-standard-operating-environments-soes"></a>1407 La version la plus récente (N), ou la version N-1, d’un système d’exploitation est utilisée pour les environnements d’exploitation standard

- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système doivent être installées sur les groupes de machines virtuelles identiques

### <a name="0380-unneeded-operating-system-accounts-software-components-services-and-functionality-are-removed-or-disabled"></a>0380 Les comptes de système d’exploitation, les logiciels, les composants, les services et les fonctionnalités superflus sont supprimés ou désactivés.

- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

### <a name="1490-an-application-whitelisting-solution-is-implemented-on-all-servers-to-restrict-the-execution-of-executables-software-libraries-scripts-and-installers-to-an-approved-set"></a>1490 Une solution de mise en liste verte d’application est implémentée sur tous les serveurs afin de restreindre l’exécution des exécutables, bibliothèques logicielles, scripts et programmes d’installation à un ensemble approuvé

- Les ces contrôles d’application adaptatifs doit être activés sur les machines virtuelles

### <a name="1417-antivirus-software-is-implemented-on-workstations-and-servers-and-configured-with-signature-based-detection-enabled-and-set-to-a-high-level-heuristic-based-detection-enabled-and-set-to-a-high-level-detection-signatures-checked-for-currency-and-updated-on-at-least-a-daily-basis-automatic-and-regular-scanning-configured-for-all-fixed-disks-and-removable-media"></a>1417 Un logiciel antivirus est implémenté sur les stations de travail et les serveurs, et configuré avec : détection basée sur la signature activée et définie sur un niveau élevé, détection heuristique activée et définie sur un niveau élevé, signatures de détection vérifiées pour la devise et mises à jour au moins une fois par jour, analyse automatique et régulière configurée pour tous les disques fixes et supports amovibles

- L’extension Microsoft IaaSAntimalware doit être déployée sur les serveurs Windows
- La solution de protection des points de terminaison doit être installée sur les groupes de machines virtuelles identiques
- Superviser les agents Endpoint Protection manquants dans Azure Security Center

## <a name="guidelines-for-system-hardening---authentication-hardening"></a>Instructions pour le durcissement du système - Durcissement de l’authentification

### <a name="1546-users-are-authenticated-before-they-are-granted-access-to-a-system-and-its-resources"></a>1546 Les utilisateurs sont authentifiés avant de se voir accorder l’accès à un système et à ses ressources

- Auditer l'accès réseau non restreint aux comptes de stockage
- Les clusters Service Fabric ne doivent utiliser Azure Active Directory que pour l’authentification client
- Afficher les résultats d’audit des machines virtuelles Linux qui autorisent les connexions à distance à partir des comptes sans mot de passe
- Déployer des prérequis pour auditer les machines virtuelles Linux qui autorisent les connexions à distance à partir des comptes sans mot de passe
- Afficher les résultats d’audit des machines virtuelles Linux qui ont des comptes sans mot de passe
- Déployer les prérequis pour l’audit des machines virtuelles Linux qui ont des comptes sans mot de passe

### <a name="0974-multi-factor-authentication-is-used-to-authenticate-standard-users"></a>0974 L’authentification multifacteur est utilisée pour authentifier les utilisateurs standard

- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement

### <a name="1173-multi-factor-authentication-is-used-to-authenticate-all-privileged-users-and-any-other-positions-of-trust"></a>1173 L’authentification multifacteur est utilisée pour authentifier tous les utilisateurs privilégiés et toute autre position de confiance

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement

### <a name="0421-passphrases-used-for-single-factor-authentication-are-a-minimum-of-14-characters-with-complexity-ideally-as-4-random-words"></a>0421 Les phrases secrètes utilisées pour l’authentification à un seul facteur ont un minimum de 14 caractères avec complexité, idéalement sous la forme de quatre mots aléatoires

- Afficher les résultats de l’audit des configurations de machines virtuelles Windows dans « Paramètres de sécurité - Stratégies de compte »
- Déployer les prérequis pour l’audit des configurations de machines virtuelles Windows dans « Paramètres de sécurité - Stratégies de compte »

## <a name="guidelines-for-system-management---system-administration"></a>Instructions pour la gestion de système - Administration de système

### <a name="1384-multi-factor-authentication-is-used-to-authenticate-users-each-time-they-perform-privileged-actions"></a>1384 L’authentification multifacteur est utilisée pour authentifier les utilisateurs chaque fois qu’ils effectuent des actions privilégiées

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement

### <a name="1386-management-traffic-is-only-allowed-to-originate-from-network-zones-that-are-used-to-administer-systems-and-applications"></a>1386 Le trafic de gestion n’est autorisé qu’à partir de zones réseau utilisées pour administrer des systèmes et des applications

- Le contrôle d’accès réseau juste-à-temps doit être appliqué sur les machines virtuelles
- Le débogage à distance doit être désactivé pour les applications API
- Le débogage à distance doit être désactivé pour les applications de fonction
- Le débogage à distance doit être désactivé pour les applications web

## <a name="guidelines-for-system-management---system-patching"></a>Instructions pour la gestion de système - Mise à jour corrective de système

### <a name="1144-security-vulnerabilities-in-applications-and-drivers-assessed-as-extreme-risk-are-patched-updated-or-mitigated-within-48-hours-of-the-security-vulnerabilities-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1144 Les vulnérabilités de sécurité dans les applications et les pilotes évaluées comme présentant un risque extrême sont corrigées, mises à jour ou atténuées dans les 48 heures suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

### <a name="0940-security-vulnerabilities-in-applications-and-drivers-assessed-as-high-risk-are-patched-updated-or-mitigated-within-two-weeks-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>0940 Les vulnérabilités de sécurité dans les applications et les pilotes évaluées comme présentant un risque élevé sont corrigées, mises à jour ou atténuées dans les deux semaines suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

### <a name="1472-security-vulnerabilities-in-applications-and-drivers-assessed-as-moderate-or-low-risk-are-patched-updated-or-mitigated-within-one-month-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1472 Les vulnérabilités de sécurité dans les applications et les pilotes évaluées comme présentant un risque modéré ou faible sont corrigées, mises à jour ou atténuées dans le mois suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

### <a name="1494-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-extreme-risk-are-patched-updated-or-mitigated-within-48-hours-of-the-security-vulnerabilities-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1494 Les vulnérabilités de sécurité dans les systèmes d’exploitation et les microprogrammes évaluées comme présentant un risque extrême sont corrigées, mises à jour ou atténuées dans les 48 heures suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

### <a name="1495-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-high-risk-are-patched-updated-or-mitigated-within-two-weeks-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1495 Les vulnérabilités de sécurité dans les systèmes d’exploitation et les microprogrammes évaluées comme présentant un risque élevé sont corrigées, mises à jour ou atténuées dans les deux semaines suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

### <a name="1496-security-vulnerabilities-in-operating-systems-and-firmware-assessed-as-moderate-or-low-risk-are-patched-updated-or-mitigated-within-one-month-of-the-security-vulnerability-being-identified-by-vendors-independent-third-parties-system-managers-or-users"></a>1496 Les vulnérabilités de sécurité dans les systèmes d’exploitation et les microprogrammes évaluées comme présentant un risque modéré ou faible sont corrigées, mises à jour ou atténuées dans le mois suivant l’identification des failles de sécurité par les fournisseurs, les tiers indépendants, les gestionnaires système ou les utilisateurs

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées
- Les paramètres d’évaluation des vulnérabilités de SQL Server doivent contenir une adresse e-mail pour recevoir les rapports d’analyse

## <a name="guidelines-for-system-management---data-backup-and-restoration"></a>Instructions pour la gestion de système - Sauvegarde et restauration de données

### <a name="1511-backups-of-important-information-software-and-configuration-settings-are-performed-at-least-daily"></a>1511 Les sauvegardes d’informations, de logiciels et de paramètres de configuration importants sont effectuées au moins une fois par jour

- Auditer des machines virtuelles pour lesquelles la reprise d’activité après sinistre n’est pas configurée

## <a name="guidelines-for-system-monitoring---event-logging-and-auditing"></a>Instructions pour la supervision du système - Journalisation des événements et audit

### <a name="1405-a-centralised-logging-facility-is-implemented-and-systems-are-configured-to-save-event-logs-to-the-centralised-logging-facility-as-soon-as-possible-after-each-event-occurs"></a>1405 Une fonctionnalité de journalisation centralisée est implémentée et les systèmes sont configurés pour enregistrer les journaux des événements dans la fonctionnalité de journalisation centralisée dès que possible après chaque événement

- Les abonnements Azure doivent avoir un profil de journal pour le journal d’activité

### <a name="0582-the-following-events-are-logged-for-operating-systems-access-to-important-data-and-processes-application-crashes-and-any-error-messages-attempts-to-use-special-privileges-changes-to-accounts-changes-to-security-policy-changes-to-system-configurations-domain-name-system-dns-and-hypertext-transfer-protocol-http-requests-failed-attempts-to-access-data-and-system-resources-service-failures-and-restarts-system-startup-and-shutdown-transfer-of-data-to-external-media-user-or-group-management-use-of-special-privileges"></a>0582 Les événements suivants sont enregistrés pour les systèmes d’exploitation : accès à des données et des processus importants, plantages d’applications et messages d’erreur, tentatives d’utilisations de privilèges spéciaux, modifications de comptes, modifications apportées à une stratégie de sécurité, modifications apportées aux configurations système, requêtes DNS (Domain Name System) et HTTP (Hypertext Transfer Protocol), échecs de tentative d’accès aux données et ressources système, échecs et redémarrage de service, démarrage et arrêt de système, transfert de données vers un média externe, gestion des utilisateurs ou des groupes, utilisation de privilèges spéciaux

- \[Préversion\] : Auditer le déploiement de Log Analytics Agent - Image de machine virtuelle (système d’exploitation) non listée
- Auditer le déploiement de Log Analytics Agent dans VMSS - Image de machine virtuelle (système d’exploitation) non listée
- Vérifier les machines virtuelles de l’espace de travail Log Analytics - Signaler les incompatibilités
- Auditer le paramètre de diagnostic

### <a name="1537-the-following-events-are-logged-for-databases-access-to-particularly-important-information-addition-of-new-users-especially-privileged-users-any-query-containing-comments-any-query-containing-multiple-embedded-queries-any-query-or-database-alerts-or-failures-attempts-to-elevate-privileges-attempted-access-that-is-successful-or-unsuccessful-changes-to-the-database-structure-changes-to-user-roles-or-database-permissions-database-administrator-actions-database-logons-and-logoffs-modifications-to-data-use-of-executable-commands"></a>1537 Les événements suivants sont enregistrés pour les bases de données : accès aux informations particulièrement importantes, ajout de nouveaux utilisateurs, en particulier des utilisateurs privilégiés, toute requête contenant des commentaires, toute requête contenant plusieurs requêtes incorporées, tout échec ou alerte de requête ou de base de données, tentative d’élévation de privilèges, tentative d’accès ayant réussi ou échoué, modifications de la structure de la base de données, modifications apportées aux rôles d’utilisateur ou aux autorisations de base de données, actions de l’administrateur de base de données, ouvertures et fermetures de session de base de données, modifications des données, utilisation de commandes exécutables

- Advanced Data Security doit être activé sur vos serveurs SQL
- Auditer le paramètre de diagnostic
- Advanced Data Security doit être activée sur vos instances managées SQL.

## <a name="guidelines-for-system-monitoring---vulnerability-management"></a>Instructions pour la supervision du système - Gestion des vulnérabilités

### <a name="0911-vulnerability-assessments-and-penetration-tests-are-conducted-by-suitably-skilled-personnel-before-a-system-is-deployed-after-a-significant-change-to-a-system-and-at-least-annually-or-as-specified-by-the-system-owner"></a>0911 Des évaluations de vulnérabilité et des tests de pénétration sont effectués par du personnel compétent avant le déploiement d’un système, après une modification significative d’un système, et au moins une fois par an ou tel que spécifié par le propriétaire du système

- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur vos instances managées SQL
- L’évaluation des vulnérabilités doit être activée sur les machines virtuelles
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées

## <a name="guidelines-for-database-systems-management---database-servers"></a>Instructions pour la gestion des systèmes de base de données - Serveurs de base de données

### <a name="1425-hard-disks-of-database-servers-are-encrypted-using-full-disk-encryption"></a>1425 Les disques durs des serveurs de base de données sont chiffrés à l’aide du chiffrement de disque complet

- Le chiffrement de disque doit être appliqué sur les machines virtuelles
- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL

### <a name="1277-information-communicated-between-database-servers-and-web-applications-is-encrypted"></a>1277 Les informations communiquées entre les serveurs de base de données et les applications web sont chiffrées

- Seules les connexions sécurisées à votre cache Redis doivent être activées
- La sécurisation du transfert vers des comptes de stockage doit être activée
- Afficher les résultats d’audit des serveurs web Windows qui n’utilisent pas de protocole de communication sécurisé
- Déployer des prérequis pour auditer les serveurs web Windows qui n’utilisent pas de protocole de communication sécurisé

## <a name="guidelines-for-database-systems-management---database-management-system-software"></a>Instructions pour la gestion des systèmes de base de données - Logiciels de système de gestion de base de données

### <a name="1260-default-database-administrator-accounts-are-disabled-renamed-or-have-their-passphrases-changed"></a>1260 Les comptes d’administrateur de base de données par défaut sont désactivés ou renommés, ou leurs phrases secrètes sont modifiées

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL

### <a name="1262-database-administrators-have-unique-and-identifiable-accounts"></a>1262 Les administrateurs de base de données disposent de comptes uniques et identifiables

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL

### <a name="1261-database-administrator-accounts-are-not-shared-across-different-databases"></a>1261 Les comptes d’administrateur de base de données ne sont pas partagés entre différentes bases de données

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL

### <a name="1263-database-administrator-accounts-are-used-exclusively-for-administrative-tasks-with-standard-database-accounts-used-for-general-purpose-interactions-with-database"></a>1263 Les comptes d’administrateur de base de données sont utilisés exclusivement pour les tâches administratives, les comptes de base de données standard étant quant à eux utilisés pour les interactions à usage général avec la base de données

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL

### <a name="1264-database-administrator-access-is-restricted-to-defined-roles-rather-than-accounts-with-default-administrative-permissions-or-all-permissions"></a>1264 L’accès administrateur à la base de données est limité à des rôles définis plutôt qu’à des comptes disposant d’autorisations d’administration par défaut ou de toutes les autorisations

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL

## <a name="guidelines-for-using-cryptography---cryptographic-fundamentals"></a>Instructions relatives à l’utilisation du chiffrement - Principes de base du chiffrement

### <a name="0459-encryption-software-used-for-data-at-rest-implements-full-disk-encryption-or-partial-encryption-where-access-controls-will-only-allow-writing-to-the-encrypted-partition"></a>0459 Le logiciel de chiffrement utilisé pour les données au repos implémente un chiffrement de disque complet, ou un chiffrement partiel là où les contrôles d’accès autorisent uniquement l’écriture dans la partition chiffrée

- Le chiffrement de disque doit être appliqué sur les machines virtuelles

## <a name="guidelines-for-using-cryptography---transport-layer-security"></a>Instructions relatives à l’utilisation du chiffrement - Protocole TLS (Transport Layer Security)

### <a name="1139-only-the-latest-version-of-tls-is-used"></a>1139 Seule la version la plus récente de TLS est utilisée

- La dernière version de TLS doit être utilisée dans votre application API
- La dernière version de TLS doit être utilisée dans votre application web
- La dernière version de TLS doit être utilisée dans votre application de fonction
- Déployer des prérequis pour auditer les serveurs web Windows qui n’utilisent pas de protocole de communication sécurisé
- Afficher les résultats d’audit des serveurs web Windows qui n’utilisent pas de protocole de communication sécurisé

## <a name="guidelines-for-data-transfers-and-content-filtering---content-filtering"></a>Instructions pour le transfert de données et le filtrage de contenu - Filtrage de contenu

### <a name="1288-antivirus-scanning-using-multiple-different-scanning-engines-is-performed-on-all-content"></a>1288 Une analyse antivirus, utilisant plusieurs moteurs d’analyse différents, est effectuée sur tout le contenu

- L’extension Microsoft IaaSAntimalware doit être déployée sur les serveurs Windows
- La solution de protection des points de terminaison doit être installée sur les groupes de machines virtuelles identiques
- Superviser les agents Endpoint Protection manquants dans Azure Security Center

## <a name="guidelines-for-data-transfers-and-content-filtering---web-application-development"></a>Instructions pour le transfert de données et le filtrage de contenu - Développement d’applications web

### <a name="1552-all-web-application-content-is-offered-exclusively-using-https"></a>1552 Tout le contenu d’application web est proposé exclusivement à l’aide du protocole HTTPS

- Function App ne doit pas être accessible via HTTPS
- L'application API doit uniquement être accessible via HTTPS
- L'application web ne doit pas être accessible via HTTPS
- Seules les connexions sécurisées à votre cache Redis doivent être activées

### <a name="1424-web-browser-based-security-controls-are-implemented-for-web-applications-in-order-to-help-protect-both-web-applications-and-their-users"></a>1424 Des contrôles de sécurité basés sur le navigateur web sont implémentés pour les applications web afin de contribuer à la protection des applications web et de leurs utilisateurs

- CORS ne doit pas autoriser toutes les ressources à accéder à vos applications web

## <a name="guidelines-for-network-management---network-design-and-configuration"></a>Instructions pour la gestion de réseau - Conception et configuration du réseau

### <a name="0520-network-access-controls-are-implemented-on-networks-to-prevent-the-connection-of-unauthorised-network-devices"></a>0520 Des contrôles d’accès réseau sont implémentés sur les réseaux pour empêcher la connexion d’appareils réseau non autorisés

- Auditer l'accès réseau non restreint aux comptes de stockage

### <a name="1182-network-access-controls-are-implemented-to-limit-traffic-within-and-between-network-segments-to-only-those-that-are-required-for-business-purposes"></a>1182 Des contrôles d’accès réseau sont implémentés pour restreindre le trafic dans et entre les segments réseau à celui requis à des fins professionnelles

- Les machines virtuelles accessibles à partir d’Internet doivent être protégées avec des groupes de sécurité réseau
- Auditer l'accès réseau non restreint aux comptes de stockage
- Les recommandations de renforcement de réseau adaptatif doivent être appliquées sur les machines virtuelles accessibles à partir d’Internet

## <a name="guidelines-for-network-management---service-continuity-for-online-services"></a>Instructions pour la gestion de réseau - Continuité de service pour les services en ligne

### <a name="1431-denial-of-service-attack-prevention-and-mitigation-strategies-are-discussed-with-service-providers-specifically-their-capacity-to-withstand-denial-of-service-attacks-any-costs-likely-to-be-incurred-by-customers-resulting-from-denial-of-service-attacks-thresholds-for-notifying-customers-or-turning-off-their-online-services-during-denial-of-service-attacks-pre-approved-actions-that-can-be-undertaken-during-denial-of-service-attacks-denial-of-service-attack-prevention-arrangements-with-upstream-providers-to-block-malicious-traffic-as-far-upstream-as-possible"></a>1431 Les stratégies de prévention et d’atténuation des attaques par déni de service sont abordées avec les fournisseurs de services, en particulier : leur capacité à résister aux attaques par déni de service, les coûts susceptibles d’être engendrés par les clients suite aux attaques par déni de service, les seuils pour la notification des clients ou la désactivation de leurs services en ligne pendant les attaques par déni de service, les actions pré-approuvées qui peuvent être entreprises pendant les attaques par déni de service, les mécanismes de prévention des attaques par déni de service avec les fournisseurs en amont pour bloquer le trafic malveillant le plus en amont possible

- DDoS Protection Standard doit être activé


> [!NOTE]
> La disponibilité des définitions Azure Policy peut varier dans Azure Government et dans d’autres clouds nationaux. 

## <a name="next-steps"></a>Étapes suivantes

Autres articles sur les blueprints et la manière de les utiliser :

> [!div class="nextstepaction"]
> [Blueprint ISM PROTECTED - Vue d’ensemble](./index.md)
> [Blueprint ISM PROTECTED - Étapes de déploiement](./deploy.md)

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
