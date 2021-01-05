---
title: Meilleures pratiques pour Windows Virtual Desktop - Azure
description: Meilleures pratiques pour sécuriser votre environnement Windows Virtual Desktop.
author: heidilohr
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: a106455ea9d7b7a64fecd7c4255c294d0bd62db8
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562614"
---
# <a name="security-best-practices"></a>Bonnes pratiques de sécurité

Windows Virtual Desktop est un service de bureau virtuel géré qui comprend de nombreuses fonctionnalités de sécurité pour garantir la sécurité de votre organisation. Dans un déploiement Windows Virtual Desktop, Microsoft gère des portions des services au nom du client. Le service dispose de nombreuses fonctionnalités de sécurité avancées intégrées, telles que la connexion inversée, qui réduisent le risque lié au fait que les bureaux à distance sont accessibles depuis n’importe où.

Cet article décrit les étapes supplémentaires que vous pouvez effectuer en tant qu’administrateur pour sécuriser les déploiements Windows Virtual Desktop de vos clients.

## <a name="security-responsibilities"></a>Responsabilités en matière de sécurité

Les services cloud diffèrent des infrastructures de bureau virtuel (VDI) traditionnelles sur site par leur mode de gestion des responsabilités en matière de sécurité. Par exemple, dans une infrastructure VDI locale traditionnelle, le client est responsable de tous les aspects de la sécurité. Toutefois, dans la plupart des services cloud, ces responsabilités sont partagées entre le client et la société.

Lorsque vous utilisez Windows Virtual Desktop, il est important de comprendre que même si certains composants sont déjà sécurisés pour votre environnement, vous devez configurer d’autres zones pour répondre aux besoins de sécurité de votre organisation.

Voici les exigences de sécurité dont vous êtes responsable dans votre déploiement Windows Virtual Desktop :

| Besoins en matière de sécurité | Le client est-il responsable de cette tâche ? |
|---------------|:-------------------------:|
|Identité|Oui|
|Appareils utilisateur (mobiles et PC)|Oui|
|Sécurité de l’application|Oui|
|Système d’exploitation de l’hôte de session|Oui|
|Configuration du déploiement|Oui|
|Contrôles de réseau|Oui|
|Plan de contrôle de virtualisation|Non|
|Hôtes physiques|Non|
|Réseau physique|Non|
|Centre de données physique|Non|

Les besoins de sécurité dont le client n’est pas responsable sont gérés par Microsoft.

## <a name="azure-security-best-practices"></a>Meilleures pratiques relatives à la sécurité Azure

Windows Virtual Desktop est un service sous Azure. Pour optimiser la sécurité de votre déploiement Windows Virtual Desktop, veillez à sécuriser également l’infrastructure Azure et le plan de gestion environnants. Pour sécuriser votre infrastructure, réfléchissez à la façon dont Windows Virtual Desktop s’intègre à votre écosystème Azure plus vaste. Pour en savoir plus sur l’écosystème Azure, consultez [Meilleures pratiques et tendances Azure relatives à la sécurité](../security/fundamentals/best-practices-and-patterns.md).

Cette section décrit les meilleures pratiques pour sécuriser votre écosystème Azure.

### <a name="enable-azure-security-center"></a>Activation d’Azure Security Center

Nous vous recommandons d’activer Azure Security Center Standard pour les abonnements, les machines virtuelles, les coffres de clés et les comptes de stockage.

Avec Azure Security Center Standard, vous pouvez :

* Gérer les vulnérabilités.
* Évaluer la conformité avec des infrastructures courantes telles que PCI.
* Renforcer la sécurité globale de votre environnement.

Pour plus d’informations, consultez [Intégrer un abonnement Azure à Security Center Standard](../security-center/security-center-get-started.md).

### <a name="improve-your-secure-score"></a>Améliorer votre score de sécurité

Le Degré de sécurisation fournit des suggestions et des conseils sur les meilleures pratiques pour améliorer votre sécurité globale. Ces suggestions sont hiérarchisées pour vous aider à choisir celles qui sont les plus importantes, et les options de correction rapide vous aident à résoudre rapidement les vulnérabilités potentielles. Ces suggestions sont également mises à jour au fil du temps, ce qui vous permet de mettre à jour la sécurité de votre environnement. Pour plus d’informations, consultez [Amélioration du degré de sécurisation dans Azure Security Center](../security-center/secure-score-security-controls.md).

## <a name="windows-virtual-desktop-security-best-practices"></a>Meilleures pratiques pour Windows Virtual Desktop

Windows Virtual Desktop possède de nombreux contrôles de sécurité intégrés. Dans cette section, vous découvrirez les contrôles de sécurité permettant de sécuriser vos utilisateurs et vos données.

### <a name="require-multi-factor-authentication"></a>Exiger une authentification multifacteur

L’authentification multifacteur obligatoire pour tous les utilisateurs et administrateurs dans Windows Virtual Desktop améliore la sécurité de l’ensemble de votre déploiement. Pour plus d’informations, consultez [Activer Azure AD Multi-Factor Authentication pour Windows Virtual Desktop](set-up-mfa.md).

### <a name="enable-conditional-access"></a>Activer l’accès conditionnel

L’activation de l’[accès conditionnel](../active-directory/conditional-access/overview.md) vous permet de gérer les risques avant d’accorder aux utilisateurs l’accès à votre environnement Windows Virtual Desktop. Lorsque vous choisissez les utilisateurs auxquels accorder l’accès, nous vous recommandons également de déterminer qui est l’utilisateur, comment il se connecte et quel appareil il utilise.

### <a name="collect-audit-logs"></a>Collecter des journaux d’audit

L’activation de la collecte des journaux d’audit vous permet d’afficher l’activité des utilisateurs et des administrateurs en rapport avec Windows Virtual Desktop. Voici quelques exemples de journaux d’audit clés :

-   [Journal d'activité Azure](../azure-monitor/platform/activity-log.md)
-   [Journal d’activité Azure Active Directory](../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md)
-   [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)
-   [Hôtes de session](../azure-monitor/platform/agent-windows.md)
-   [Journal de diagnostic Windows Virtual Desktop](../virtual-desktop/diagnostics-log-analytics.md)
-   [Journaux d’activité Key Vault](../key-vault/general/logging.md)

### <a name="use-remoteapps"></a>Utiliser RemoteApps

Lors du choix d’un modèle de déploiement, vous pouvez fournir aux utilisateurs distants l’accès à des bureaux virtuels entiers ou uniquement à des applications sélectionnées. Les applications distantes, ou RemoteApps, fournissent une expérience transparente lorsque l’utilisateur travaille avec des applications sur son bureau virtuel. Les RemoteApps réduisent les risques en permettant uniquement à l’utilisateur de travailler avec un sous-ensemble de l’ordinateur distant exposé par l’application.

### <a name="monitor-usage-with-azure-monitor"></a>Superviser l’utilisation avec Azure Monitor

Surveillez l’utilisation et la disponibilité de votre service Windows Virtual Desktop avec [Azure Monitor](https://azure.microsoft.com/services/monitor/). Envisagez de créer des [alertes d’intégrité de service](../service-health/alerts-activity-log-service-notifications-portal.md) pour le service Windows Virtual Desktop afin de recevoir des notifications chaque fois qu’un événement impacte le service.

## <a name="session-host-security-best-practices"></a>Meilleures pratiques pour la sécurité de l’hôte de session

Les hôtes de session sont des machines virtuelles qui s’exécutent au sein d’un abonnement Azure et d’un réseau virtuel. La sécurité globale de votre déploiement Windows Virtual Desktop dépend des contrôles de sécurité que vous placez sur vos hôtes de session. Cette section décrit les meilleures pratiques pour assurer la sécurité de vos hôtes de session.

### <a name="enable-screen-capture-protection-preview"></a>Activer la protection contre la capture d’écran (préversion)

La fonctionnalité de protection contre la capture d’écran empêche la capture d’informations sensibles sur les points de terminaison clients. Lorsque vous activez cette fonctionnalité, le contenu distant est automatiquement bloqué ou masqué dans les captures d’écran et les partages d’écran. Il est également caché des logiciels malveillants qui peuvent capturer en permanence le contenu de votre écran. Nous vous recommandons de désactiver la redirection du Presse-papiers pour empêcher la copie du contenu distant vers les points de terminaison lors de l’utilisation de cette fonctionnalité.

Cette stratégie est appliquée au niveau de l’hôte en configurant une clé de Registre. Pour activer cette stratégie, ouvrez PowerShell et définissez la clé de Registre **fEnableScreenCaptureProtection** en exécutant la cmdlet suivante :

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableScreenCaptureProtection /t REG_DWORD /d 1
```

Pour tester cette nouvelle fonctionnalité :

- Assurez-vous que vos pools d’hôtes sont configurés dans l’environnement de validation.
- Vérifiez que vous avez téléchargé et installé le client Windows Desktop, version 1.2.1526 ou ultérieure.

>[!NOTE]
>Pendant la préversion, seules les connexions de bureau complètes à partir de points de terminaison Windows 10 prennent en charge cette fonctionnalité.


### <a name="enable-endpoint-protection"></a>Activer Endpoint Protection

Pour protéger votre déploiement contre les logiciels malveillants connus, nous vous recommandons d’activer Endpoint Protection sur tous les hôtes de session. Vous pouvez utiliser l’antivirus Windows Defender ou un programme tiers. Pour plus d’informations, consultez [Guide de déploiement de l’antivirus Windows Defender dans un environnement VDI](/windows/security/threat-protection/windows-defender-antivirus/deployment-vdi-windows-defender-antivirus).

Pour les solutions de profil, telles que FSLogix ou d’autres solutions, qui effectuent le montage de fichiers VHD, nous vous recommandons d’exclure les extensions de fichier VHD.

### <a name="install-an-endpoint-detection-and-response-product"></a>Installer un produit de détection et de réponse de point de terminaison

Nous vous recommandons d’installer un produit de détection et de réponse de point de terminaison (EDR) pour fournir des fonctionnalités avancées de détection et de réponse. Pour les systèmes d’exploitation serveur sur lesquels [Azure Security Center](../security-center/security-center-services.md) est activé, l’installation d’un produit EDR déploie Defender ATP. Pour les systèmes d’exploitation client, vous pouvez déployer [Defender](/windows/security/threat-protection/microsoft-defender-atp/onboarding) ou un produit tiers sur ces points de terminaison.

### <a name="enable-threat-and-vulnerability-management-assessments"></a>Activer les évaluations de gestion des menaces et des vulnérabilités

L’identification des vulnérabilités logicielles qui existent dans les systèmes d’exploitation et les applications est essentielle pour garantir la sécurité de votre environnement. Azure Security Center peut vous aider à identifier les points posant problème grâce à des évaluations de vulnérabilités pour les systèmes d’exploitation serveur. Vous pouvez également utiliser Defender ATP, qui permet de gérer les menaces et les vulnérabilités pour les systèmes d’exploitation de bureau. Vous pouvez également utiliser des produits tiers si vous le souhaitez, mais nous vous recommandons d'utiliser Azure Security Center et Defender ATP.

### <a name="patch-software-vulnerabilities-in-your-environment"></a>Corriger les vulnérabilités des logiciels dans votre environnement

Une fois que vous avez identifié une vulnérabilité, vous devez la corriger. Cela s’applique également aux environnements virtuels, qui comprennent les systèmes d’exploitation en cours d’exécution, les applications qui y sont déployées et les images à partir desquelles vous créez de nouvelles machines. Soyez attentif aux messages de notification de correctifs et appliquez les correctifs en temps utile. Nous vous recommandons d’appliquer des correctifs mensuels à vos images de base pour vous assurer que les machines nouvellement déployées sont aussi sécurisées que possible.

### <a name="establish-maximum-inactive-time-and-disconnection-policies"></a>Établir des stratégies de temps d’inactivité et de déconnexion maximales

La déconnexion des utilisateurs inactifs préserve les ressources et empêche l'accès des utilisateurs non autorisés. De préférence, les délais d’expiration équilibrent la productivité des utilisateurs ainsi que l’utilisation des ressources. Pour les utilisateurs qui interagissent avec des applications sans état, envisagez des stratégies plus agressives qui éteignent les machines et préservent les ressources. La déconnexion d’applications en cours d’exécution depuis longtemps et qui continuent à fonctionner si un utilisateur est inactif, comme une simulation ou un rendu CAO, peut interrompre le travail de l’utilisateur et peut même nécessiter le redémarrage de l’ordinateur.

### <a name="set-up-screen-locks-for-idle-sessions"></a>Configurer des verrous d’écran pour les sessions inactives

Vous pouvez empêcher l’accès au système indésirable en configurant Windows Virtual Desktop de façon à verrouiller l’écran d’une machine pendant la durée d’inactivité et exiger l’authentification pour le déverrouiller.

### <a name="establish-tiered-admin-access"></a>Établir un accès administrateur à plusieurs niveaux

Nous vous recommandons de ne pas accorder à vos utilisateurs un accès administrateur aux bureaux virtuels. Si vous avez besoin de logiciels, nous vous recommandons de les rendre disponibles par le biais d’utilitaires de gestion de la configuration comme Microsoft Endpoint Manager. Dans un environnement multi-session, nous vous recommandons de ne pas laisser les utilisateurs installer directement les logiciels.

### <a name="consider-which-users-should-access-which-resources"></a>Déterminer quels utilisateurs doivent accéder à telles ou telles ressources

Considérez les hôtes de session comme une extension de votre déploiement de postes de travail existant. Nous vous recommandons de contrôler l’accès aux ressources du réseau de la même manière que vous le feriez pour les autres ordinateurs de votre environnement, par exemple en utilisant la segmentation et le filtrage du réseau. Par défaut, les hôtes de session peuvent se connecter à n’importe quelle ressource sur Internet. Vous pouvez limiter le trafic de plusieurs façons, notamment en utilisant le Pare-feu Azure, des appliances virtuelles réseau ou des proxies. Si vous devez limiter le trafic, veillez à ajouter les règles appropriées afin que Windows Virtual Desktop puisse fonctionner correctement.

### <a name="manage-office-pro-plus-security"></a>Gérer la sécurité d’Office Pro plus

En plus de sécuriser vos hôtes de session, il est important de sécuriser également les applications qui s'y exécutent. Office Pro plus est l’une des applications les plus courantes déployées dans les hôtes de session. Pour améliorer la sécurité du déploiement d’Office, nous vous recommandons d’utiliser le [Conseiller en stratégie de sécurité](/DeployOffice/overview-of-security-policy-advisor) pour Microsoft 365 Apps for enterprise. Cet outil identifie les stratégies que vous pouvez appliquer à votre déploiement pour davantage de sécurité. Conseiller en stratégie de sécurité recommande également des stratégies en fonction de leur impact sur votre sécurité et votre productivité.

### <a name="other-security-tips-for-session-hosts"></a>Autres conseils de sécurité pour les hôtes de session

En limitant les capacités du système d'exploitation, vous pouvez renforcer la sécurité de vos hôtes de session. Voici quelques opérations que vous pouvez effectuer :

- Contrôlez la redirection des périphériques en redirigeant les lecteurs, les imprimantes et les périphériques USB vers le périphérique local d’un utilisateur dans une session de bureau à distance. Nous vous recommandons d'évaluer vos besoins en matière de sécurité et de vérifier si ces fonctionnalités doivent être désactivées ou non.

- Limitez l’accès de l’Explorateur Windows en masquant les mappages de lecteurs locaux et distants. Cela empêche les utilisateurs de découvrir des informations non désirées sur la configuration du système et les utilisateurs.

- Empêchez l’accès RDP direct aux hôtes de session dans votre environnement. Si vous avez besoin d’un accès RDP direct pour l’administration ou le dépannage, activez l’accès [juste-à-temps](../security-center/security-center-just-in-time.md) pour limiter la surface d’attaque potentielle sur un hôte de session.

- Accordez aux utilisateurs des autorisations limitées lorsqu’ils accèdent à des systèmes de fichiers locaux et distants. Vous pouvez limiter les autorisations en vous assurant que vos systèmes de fichiers locaux et distants utilisent des listes de contrôle d’accès avec le moins de privilèges possible. De cette façon, les utilisateurs peuvent accéder uniquement à ce dont ils ont besoin et ne peuvent pas modifier ou supprimer les ressources critiques.

- Empêchez l’exécution de logiciels indésirables sur les hôtes de session. Vous pouvez activer App Locker pour une sécurité supplémentaire sur les hôtes de session, en vous assurant que seules les applications que vous autorisez peuvent fonctionner sur l'hôte.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment activer l’authentification multifacteur, consultez [Configurer l’authentification multifacteur](set-up-mfa.md).