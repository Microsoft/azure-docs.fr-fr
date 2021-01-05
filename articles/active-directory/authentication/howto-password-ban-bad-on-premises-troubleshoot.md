---
title: Résoudre les problèmes de protection par mot de passe Azure AD en local
description: Découvrez comment résoudre les problèmes de protection par mot de passe Azure AD dans un environnement Active Directory Domain Services local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6ca00785bfe8a99b8a3d620559c4fa492ee60c63
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741743"
---
# <a name="troubleshoot-on-premises-azure-ad-password-protection"></a>Résoudre les problèmes : Protection par mot de passe Azure AD en local

Après déploiement de la protection par mot de passe Azure AD, il peut s’avérer nécessaire de résoudre des problèmes. Cet article entre dans les détails pour vous aider à comprendre certaines étapes de la résolution des problèmes courants.

## <a name="the-dc-agent-cannot-locate-a-proxy-in-the-directory"></a>L’agent du contrôleur de domaine ne peut pas localiser un proxy dans l’annuaire

Ce problème se traduit principalement par la présence d’événements 30017 dans le journal des événements d’administration de l’agent du contrôleur de domaine.

La cause habituelle de ce problème est qu’un proxy n’a pas encore été inscrit. Si un proxy a été inscrit, en raison de la latence de réplication d’AD, un certain délai peut s’écouler avant qu’un agent du contrôleur de domaine particulier soit en mesure de voir ce proxy.

## <a name="the-dc-agent-is-not-able-to-communicate-with-a-proxy"></a>L’agent du contrôleur de domaine ne peut pas communiquer avec un proxy

Ce problème se traduit principalement par la présence d’événements 30018 dans le journal des événements d’administration de l’agent du contrôleur de domaine. Ce problème peut avoir plusieurs causes :

1. L’agent du contrôleur de domaine se trouve dans une partie isolée du réseau qui n’autorise pas de connectivité réseau au(x) proxy(s) inscrit(s). Ce problème peut être sans gravité tant que d’autres agents du contrôleur de domaine peuvent communiquer avec les proxys pour télécharger des stratégies de mot de passe à partir d’Azure. Une fois ces stratégies téléchargées, le contrôleur de domaine isolé l’obtient via une réplication des fichiers de stratégie du partage sysvol.

1. L’ordinateur hôte proxy bloque l’accès au point de terminaison du mappeur de point de terminaison RPC (port 135).

   Le programme d’installation du proxy de protection par mot de passe Azure AD crée automatiquement une règle de trafic entrant du Pare-feu Windows, qui autorise l’accès au port 135. En cas de suppression ou de désactivation de cette règle par la suite, les agents du contrôleur de domaine ne peuvent plus communiquer avec le service proxy. Si le Pare-feu Windows intégré a été désactivé à la place d’un autre produit de pare-feu, vous devez configurer ce pare-feu pour autoriser l’accès au port 135.

1. L’ordinateur hôte proxy bloque l’accès au point de terminaison RPC (dynamique ou statique) qu’écoute le service proxy.

   Le programme d’installation du proxy de protection par mot de passe Azure AD crée automatiquement une règle de trafic entrant du Pare-feu Windows, qui autorise l’accès à tout port entrant qu’écoute le service proxy de protection par mot de passe Azure AD. En cas de suppression ou de désactivation de cette règle par la suite, les agents du contrôleur de domaine ne peuvent plus communiquer avec le service proxy. Si le Pare-feu Windows intégré a été désactivé à la place d’un autre produit de pare-feu, vous devez configurer ce pare-feu pour autoriser l’accès à tout port entrant qu’écoute le service proxy de protection par mot de passe Azure AD. Cette configuration peut être rendue plus spécifique si le service proxy a été configuré pour écouter un port RPC statique spécifique (à l’aide de l’applet de commande `Set-AzureADPasswordProtectionProxyConfiguration`).

1. L’ordinateur hôte proxy n’est pas configuré pour autoriser les contrôleurs de domaine à se connecter à l’ordinateur. Ce comportement est contrôlé par le biais de l’affectation du privilège utilisateur « Accéder à cet ordinateur à partir du réseau ». Ce privilège doit être accordé à tous les contrôleurs de domaine de tous les domaines de la forêt. Ce paramètre est souvent restreint dans le cadre d’un plus grand effort de renforcement du réseau.

## <a name="proxy-service-is-unable-to-communicate-with-azure"></a>Le service proxy n'est pas en mesure de communiquer avec Azure

1. Vérifiez que l’ordinateur proxy dispose d’une connectivité aux points de terminaison répertoriés dans les [conditions requises pour le déploiement](howto-password-ban-bad-on-premises-deploy.md).

1. Assurez-vous que la forêt et tous les serveurs proxy sont inscrits auprès du même locataire Azure.

   Vous pouvez vérifier cette exigence en exécutant les cmdlets PowerShell `Get-AzureADPasswordProtectionProxy` et `Get-AzureADPasswordProtectionDCAgent`, puis en comparant la propriété `AzureTenant` de chaque élément retourné. Pour que tout fonctionne correctement, le nom du locataire rapporté doit être identique pour l’ensemble des agents du contrôleur de domaine et des serveurs proxy.

   S’il n’existe pas de condition d’incompatibilité d’inscription du locataire Azure, ce problème peut être résolu en exécutant les cmdlets PowerShell `Register-AzureADPasswordProtectionProxy` ou `Register-AzureADPasswordProtectionForest` si besoin, en veillant à utiliser les informations d’identification du même locataire Azure pour toutes les inscriptions.

## <a name="dc-agent-is-unable-to-encrypt-or-decrypt-password-policy-files"></a>L’agent du contrôleur de domaine ne peut pas chiffrer ou déchiffrer des fichiers de stratégie de mot de passe

La protection par mot de passe Azure AD dépend fortement des fonctionnalités de chiffrement et de déchiffrement fournies par le service de distribution de clés Microsoft. Des défaillances de chiffrement ou de déchiffrement peuvent se manifester avec différents symptômes et avoir plusieurs causes potentielles.

1. Vérifiez que le service de distribution de clés est activé et opérationnel sur tous les contrôleurs de domaine Windows Server 2012 et ultérieur au sein d’un domaine.

   Par défaut, le mode de démarrage du service de distribution de clés est Manuel (Déclencher le démarrage). Cette configuration a pour effet que, la première fois qu’un client essaie d’utiliser le service, celui-ci est démarré à la demande. Ce mode de démarrage par défaut du service permet que la protection par mot de passe Azure AD fonctionne.

   Si le mode de démarrage du service de distribution de clés est désactivé, cette configuration doit être corrigée pour que la protection par mot de passe Azure AD fonctionne correctement.

   Un test simple en lien avec ce problème consiste à démarrer manuellement le service de distribution de clés, soit via la console MMC de management des services, ou à l’aide d’autres outils de gestion (par exemple, en tapant la commande « net start kdssvc » dans une console d’invite de commandes). Le service de distribution de clés est censé démarrer correctement et continuer à s’exécuter.

   La cause racine la plus courante de l’incapacité du service de distribution de clés à démarrer est que l’objet contrôleur de domaine Active Directory se trouve en dehors de l’unité d’organisation Contrôleurs de domaine par défaut. Cette configuration n’est pas pris en charge par le service de distribution de clés et ne constitue pas une limitation imposée par la protection par mot de passe Azure AD. La solution pour corriger cette situation consiste à déplacer l’objet contrôleur de domaine vers un emplacement situé sous l’unité d’organisation Contrôleurs de domaine par défaut.

1. Incompatibilité du format des tampons chiffrés par le service de distribution de clés de Windows Server 2012 R2 vers Windows Server 2016

   Un correctif de sécurité du service de distribution de clés a été introduit dans Windows Server 2016. Il modifie le format des tampons chiffrés par le service de distribution de clés. Il est parfois impossible de déchiffrer ces tampons sur Windows Server 2012 et Windows Server 2012 R2. En revanche, cela fonctionne dans le sens inverse : les tampons chiffrés par le service de distribution de clés sur Windows Server 2012 et Windows Server 2012 R2 sont toujours déchiffrés correctement sur Windows Server 2016 et versions ultérieures. Si les contrôleurs de domaine de vos domaines Active Directory exécutent une combinaison de ces systèmes d’exploitation, des échecs du déchiffrement de la protection par mot de passe Azure AD risquent d’être signalés de temps en temps. Il n’est pas possible de prédire avec précision le moment ou les symptômes de ces échecs au vu de la nature du correctif de sécurité, et de ce fait, il n’est pas possible de déterminer quel agent de contrôleur de domaine de la protection par mot de passe Azure AD va chiffrer les données à un moment donné sur quel contrôleur de domaine.

   Il n’existe pas de solution de contournement pour ce problème si ce n’est d’éviter d’exécuter une combinaison de ces systèmes d’exploitation incompatibles dans votre/vos domaine(s) Active Directory. En d’autres termes, vous devez exécuter uniquement des contrôleurs de domaine Windows Server 2012 et Windows Server 2012 R2 OU uniquement des contrôleurs de domaine Windows Server 2016 et ultérieur.

## <a name="dc-agent-thinks-the-forest-has-not-been-registered"></a>L’agent DC estime que la forêt n’a pas été inscrite

Le symptôme de ce problème est la journalisation d’événements 30016 dans le canal DC Agent/Admin qui indique :

```text
The forest has not been registered with Azure. Password policies cannot be downloaded from Azure unless this is corrected.
```

Il existe deux causes possibles pour ce problème.

1. La forêt n’a en effet pas été inscrite. Pour résoudre le problème, exécutez la commande Register-AzureADPasswordProtectionForest comme décrit dans les [exigences de déploiement](howto-password-ban-bad-on-premises-deploy.md).
1. La forêt a été inscrite, mais l’agent DC ne peut pas déchiffrer les données d’inscription de la forêt. Ce cas de figure a la même cause racine que le problème #2 indiqué ci-dessus sous [l’agent DC ne peut pas chiffrer ou déchiffrer les fichiers de stratégie de mot de passe](howto-password-ban-bad-on-premises-troubleshoot.md#dc-agent-is-unable-to-encrypt-or-decrypt-password-policy-files). Un moyen simple de confirmer cette théorie est que vous verrez cette erreur uniquement sur les agents DC qui s’exécutent sur les contrôleurs de domaine Windows Server 2012 ou Windows Server 2012R2, tandis que les agents DC exécutés sur les contrôleurs de domaine Windows Server 2016 et versions ultérieures sont corrects. La solution de contournement est la même : mettre à niveau tous les contrôleurs de domaine vers Windows Server 2016 ou version ultérieure.

## <a name="weak-passwords-are-being-accepted-but-should-not-be"></a>Les mots de passe faibles sont acceptés alors qu’ils ne devraient pas l’être

Ce problème peut avoir plusieurs causes.

1. Vos agents du contrôleur de domaine exécutent une préversion publique du logiciel qui a expiré. Voir [Expiration de la préversion publique du logiciel de l’agent de contrôleur de domaine](howto-password-ban-bad-on-premises-troubleshoot.md#public-preview-dc-agent-software-has-expired).

1. Les agents de votre contrôleur de domaine ne peuvent pas télécharger une stratégie ou déchiffrer des stratégies existantes. Recherchez les causes possibles dans les rubriques ci-dessus.

1. Le mode d’application de la stratégie de mot de passe est toujours défini sur Audit. Si c’est le cas, reconfigurez-le pour l’appliquer via le portail de protection par mot de passe Azure AD. Pour plus d’informations, consultez [Modes de fonctionnement](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

1. La stratégie de mot de passe a été désactivée. Si c’est le cas, reconfigurez-le pour l’activer via le portail de protection par mot de passe Azure AD. Pour plus d’informations, consultez [Modes de fonctionnement](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

1. Vous n’avez pas installé le logiciel de l’agent du contrôleur de domaine sur tous les contrôleurs de domaine dans le domaine. Dans ce cas, il est difficile de s’assurer que les clients Windows distants ciblent un contrôleur de domaine particulier pendant une opération de modification de mot de passe. Si vous pensez avoir ciblé avec succès un contrôleur de domaine particulier sur lequel le logiciel de l’agent du contrôleur de domaine est installé, vous pouvez examiner attentivement le journal des événements d’administration de l’agent du contrôleur de domaine. Quel que soit le résultat, il y aura au moins un événement pour expliquer le résultat du processus de validation du mot de passe. En l’absence d’événement en relation avec l’utilisateur dont le mot de passe a changé, la modification du mot de passe a probablement été traitée par un autre contrôleur de domaine.

   Un autre test consiste à essayer de définir ou modifier les mots de passe pendant que vous êtes connecté directement à un contrôleur de domaine sur lequel le logiciel de l’agent du contrôleur de domaine est installé. Cette technique n’est pas recommandée pour les domaines Active Directory de production.

   Bien qu’un déploiement incrémentiel du logiciel de l’agent du contrôleur de domaine soit pris en charge dans le respect de ces limitations, Microsoft recommande vivement d’installer au plus tôt le logiciel de l’agent du contrôleur de domaine sur tous les contrôleurs de domaine au sein d’un domaine.

1. En définitive, l’algorithme de validation de mot de passe fonctionne peut-être comme prévu. Voir [Comment les mots de passe sont évalués](concept-password-ban-bad.md#how-are-passwords-evaluated).

## <a name="ntdsutilexe-fails-to-set-a-weak-dsrm-password"></a>Ntdsutil.exe ne parvient pas à définir un mot de passe DSRM faible

Active Directory valide toujours un nouveau mot de passe en mode de réparation des services d’annuaire pour s’assurer qu’il répond aux exigences de complexité du mot de passe du domaine. Cette validation appelle aussi les dll de filtre de mot de passe, comme la protection par mot de passe Azure AD. Si le nouveau mot de passe DSRM est rejeté, le message d’erreur suivant s’affiche :

```text
C:\>ntdsutil.exe
ntdsutil: set dsrm password
Reset DSRM Administrator Password: reset password on server null
Please type password for DS Restore Mode Administrator Account: ********
Please confirm new password: ********
Setting password failed.
        WIN32 Error Code: 0xa91
        Error Message: Password doesn't meet the requirements of the filter dll's
```

Lorsque la protection par mot de passe Azure AD enregistre les événements de journal des événements de validation d'un mot de passe DSRM Active Directory, il est possible que les messages du journal des événements n'inclut pas de nom d'utilisateur. Ce comportement est dû au fait que le compte DSRM est un compte local qui ne fait pas partie du domaine Active Directory réel.  

## <a name="domain-controller-replica-promotion-fails-because-of-a-weak-dsrm-password"></a>La promotion du réplica du contrôleur de domaine échoue en raison d’un mot de passe DSRM faible

Lors du processus de promotion DC, le nouveau mot de passe du mode de réparation des services d'annuaire est soumis à un DC existant à des fins de validation. Si le nouveau mot de passe DSRM est rejeté, le message d’erreur suivant s’affiche :

```powershell
Install-ADDSDomainController : Verification of prerequisites for Domain Controller promotion failed. The Directory Services Restore Mode password does not meet a requirement of the password filter(s). Supply a suitable password.
```

Comme dans le problème ci-dessus, l'événement de validation du mot de passe de protection par mot de passe Azure AD ne présentera pas de nom d'utilisateur pour ce scénario.

## <a name="domain-controller-demotion-fails-due-to-a-weak-local-administrator-password"></a>La rétrogradation du contrôleur de domaine échoue en raison d’un mot de passe administrateur local faible

Il est possible de rétrograder un contrôleur de domaine exécutant le logiciel de l’agent DC. Les administrateurs doivent toutefois savoir que le logiciel de l’agent du contrôleur de domaine continue d’appliquer la stratégie de mot de passe en vigueur lors de la procédure de rétrogradation. Le nouveau mot de passe du compte administrateur local (spécifié dans le cadre de l’opération de rétrogradation) est validé comme n’importe quel autre mot de passe. Dans le cadre d'une procédure de régression DC, Microsoft recommande de choisir des mots de passe sécurisés pour les comptes d’administrateur local.

Une fois que la rétrogradation a réussi et que le contrôleur de domaine a été redémarré et est à nouveau en cours d’exécution en tant que serveur de membre normal, le logiciel de l’agent DC repasse à une exécution en mode passif. Il peut alors être désinstallé à tout moment.

## <a name="booting-into-directory-services-repair-mode"></a>Démarrage en mode de réparation des services d'annuaire

Si le contrôleur de domaine est démarré en mode de réparation des services d’annuaire, la DLL de filtre de mots de passe de l’agent du contrôleur de domaine le détecte, ce qui a pour effet de désactiver toutes les activités de validation ou d’application du mot de passe, quelle que soit la configuration de la stratégie active. La DLL de filtre de mots de passe de l’agent du contrôleur de domaine consigne un événement d’avertissement 10023 dans le journal des événements de l’administrateur, par exemple :

```text
The password filter dll is loaded but the machine appears to be a domain controller that has been booted into Directory Services Repair Mode. All password change and set requests will be automatically approved. No further messages will be logged until after the next reboot.
```
## <a name="public-preview-dc-agent-software-has-expired"></a>Le logiciel de l’agent du contrôleur de domaine en préversion publique a expiré

Au cours de la période de préversion publique de la protection par mot de passe Azure AD, le logiciel de l’agent du contrôleur de domaine est codé en dur pour arrêter le traitement des demandes de validation de mot de passe aux dates suivantes :

* La version 1.2.65.0 cessera de traiter les demandes de validation de mot de passe le 1er septembre 2019.
* Les versions 1.2.25.0 et antérieures ont cessé de traiter les demandes de validation de mot de passe le 1er juillet 2019.

À l’approche de la date d’échéance, toutes les versions de l’agent de contrôleur de domaine limitées dans le temps émettront dans le journal des événements Admin de l’agent du contrôleur de domaine un événement 10021 au démarrage :

```text
The password filter dll has successfully loaded and initialized.

The allowable trial period is nearing expiration. Once the trial period has expired, the password filter dll will no longer process passwords. Please contact Microsoft for an newer supported version of the software.

Expiration date:  9/01/2019 0:00:00 AM

This message will not be repeated until the next reboot.
```

Une fois la date d’échéance atteinte, toutes les versions de l’agent de contrôleur de domaine limitées dans le temps émettront dans le journal des événements Admin de l’agent du contrôleur de domaine un événement 10022 au démarrage :

```text
The password filter dll is loaded but the allowable trial period has expired. All password change and set requests will be automatically approved. Please contact Microsoft for a newer supported version of the software.

No further messages will be logged until after the next reboot.
```

Étant donné que la date d’échéance n’est vérifiée qu’au démarrage initial, vous ne verrez peut-être pas ces événements avant longtemps après le dépassement de l’échéance du calendrier. Une fois l’échéance reconnue, le seul effet sur le contrôleur de domaine ou l’environnement au sens large est que tous les mots de passe sont automatiquement approuvés.

> [!IMPORTANT]
> Microsoft recommande que les agents du contrôleur de domaine en préversion publique expirés soient immédiatement mis à niveau vers la dernière version.

Pour découvrir facilement les agents du contrôleur de domaine dans votre environnement qui doivent être mis à niveau, exécutez la cmdlet `Get-AzureADPasswordProtectionDCAgent`, par exemple :

```powershell
PS C:\> Get-AzureADPasswordProtectionDCAgent

ServerFQDN            : bpl1.bpl.com
SoftwareVersion       : 1.2.125.0
Domain                : bpl.com
Forest                : bpl.com
PasswordPolicyDateUTC : 8/1/2019 9:18:05 PM
HeartbeatUTC          : 8/1/2019 10:00:00 PM
AzureTenant           : bpltest.onmicrosoft.com
```

Pour cette rubrique, le champ SoftwareVersion est à l’évidence la propriété de clé à examiner. Vous pouvez également utiliser un filtrage PowerShell pour filtrer les agents du contrôleur de domaine qui sont déjà au niveau de la version de référence requise ou à un niveau supérieur, par exemple :

```powershell
PS C:\> $LatestAzureADPasswordProtectionVersion = "1.2.125.0"
PS C:\> Get-AzureADPasswordProtectionDCAgent | Where-Object {$_.SoftwareVersion -lt $LatestAzureADPasswordProtectionVersion}
```

Le logiciel proxy de protection par mot de passe Azure AD n’est limité dans le temps dans aucune version. Microsoft recommande toujours de mettre à niveau les agents de contrôleur de domaine et de proxy vers les dernières versions à mesure que celles-ci sont publiées. La cmdlet `Get-AzureADPasswordProtectionProxy` permet de rechercher des agents proxy qui nécessitent des mises à niveau, comme dans l’exemple ci-dessus pour les agents du contrôleur de domaine.

Pour plus d’informations sur les procédures de mise à niveau spécifiques, consultez [Mise à niveau de l’agent du contrôleur de domaine](howto-password-ban-bad-on-premises-deploy.md#upgrading-the-dc-agent) et [Mise à niveau du service proxy](howto-password-ban-bad-on-premises-deploy.md#upgrading-the-proxy-service).

## <a name="emergency-remediation"></a>Correction d’urgence

Si le service d’agent DC cause des problèmes, vous devez l’arrêter immédiatement. La DLL de filtre de mot de passe d’agent DC tente encore d’appeler le service non exécuté et enregistrera des événements d’avertissement (10012, 10013), mais tous les mots de passe entrants sont acceptés pendant ce temps. Le service d’agent DC peut également être configuré via le Gestionnaire de contrôle des services Windows avec un type de démarrage « Désactivé » en fonction des besoins.

Une autre mesure de correction consisterait à définir le mode d’activation sur Non dans le portail de la protection par mot de passe Azure AD. Une fois la stratégie mise à jour téléchargée, chaque service d’agent du contrôleur de domaine passe en mode inactif où tous les mots de passe sont acceptés tels quels. Pour plus d’informations, consultez [Modes de fonctionnement](howto-password-ban-bad-on-premises-operations.md#modes-of-operation).

## <a name="removal"></a>Suppression

Si vous décidez de désinstaller le logiciel de protection par mot de passe Azure AD et de nettoyer tout état associé des domaines et de la forêt, vous pouvez accomplir cette tâche en procédant comme suit :

> [!IMPORTANT]
> Il est important d’effectuer ces étapes dans l’ordre. Si une instance du service Proxy reste en cours d’exécution, elle recrée périodiquement son objet serviceConnectionPoint. Si une instance du service d’agent DC reste en cours d’exécution, elle recrée périodiquement son objet serviceConnectionPoint et l’état sysvol.

1. Désinstallez le logiciel de proxy sur tous les ordinateurs. Cette étape ne requiert **pas** de redémarrage.
2. Désinstallez le logiciel de l’agent DC sur tous les contrôleurs de domaine. Cette étape **requiert** un redémarrage.
3. Supprimez manuellement tous les points de connexion du service Proxy dans chaque contexte de nommage de domaine. Vous pouvez détecter l’emplacement de ces objets avec la commande PowerShell Active Directory suivante :

   ```powershell
   $scp = "serviceConnectionPoint"
   $keywords = "{ebefb703-6113-413d-9167-9f8dd4d24468}*"
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   N’omettez pas l’astérisque (« * ») à la fin de la valeur de la variable $keywords.

   Les objets trouvés via la commande `Get-ADObject` peuvent ensuite être dirigés vers `Remove-ADObject` ou supprimés manuellement.

4. Supprimez manuellement tous les points de connexion d’agent DC dans chaque contexte de nommage de domaine. Il peut exister un de ces objets par contrôleur de domaine dans la forêt, selon la portée du déploiement du logiciel. Vous pouvez détecter l’emplacement de cet objet avec la commande PowerShell Active Directory suivante :

   ```powershell
   $scp = "serviceConnectionPoint"
   $keywords = "{2bac71e6-a293-4d5b-ba3b-50b995237946}*"
   Get-ADObject -SearchScope Subtree -Filter { objectClass -eq $scp -and keywords -like $keywords }
   ```

   Les objets trouvés via la commande `Get-ADObject` peuvent ensuite être dirigés vers `Remove-ADObject` ou supprimés manuellement.

   N’omettez pas l’astérisque (« * ») à la fin de la valeur de la variable $keywords.

5. Supprimez manuellement l’état de configuration au niveau de la forêt. L’état de configuration de la forêt est conservé dans un conteneur du contexte de nommage de configuration Active Directory. Il peut être détecté et supprimé comme suit :

   ```powershell
   $passwordProtectionConfigContainer = "CN=Azure AD Password Protection,CN=Services," + (Get-ADRootDSE).configurationNamingContext
   Remove-ADObject -Recursive $passwordProtectionConfigContainer
   ```

6. Supprimez tous les états sysvol manuellement en supprimant manuellement le dossier suivant et tout son contenu :

   `\\<domain>\sysvol\<domain fqdn>\AzureADPasswordProtection`

   Si nécessaire, ce chemin est aussi accessible localement sur un contrôleur de domaine donné ; l’emplacement par défaut ressemble au chemin d’accès suivant :

   `%windir%\sysvol\domain\Policies\AzureADPasswordProtection`

   Ce chemin d’accès est différent si le partage sysvol a été configuré à un emplacement non défini par défaut.

## <a name="next-steps"></a>Étapes suivantes

[Forum au questions sur la protection par mot de passe Azure AD](howto-password-ban-bad-on-premises-faq.md)

Pour plus d’informations sur les listes de mots de passe interdits globales et personnalisées, consultez l’article [Éliminer les mots de passe interdits](concept-password-ban-bad.md).
