---
title: Synchronisation de l’heure pour les machines virtuelles Windows dans Azure
description: Synchronisation de l’heure pour les machines virtuelles Windows.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/17/2018
ms.author: cynthn
ms.openlocfilehash: 830bdd45be4b0365ac45bc3ea366b99a34882a4c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010620"
---
# <a name="time-sync-for-windows-vms-in-azure"></a>Synchronisation de l’heure pour les machines virtuelles Windows dans Azure

La synchronisation de l’heure est importante pour la sécurité et la corrélation des événements. Elle est parfois utilisée pour l’implémentation de transactions distribuées. La précision de l’heure entre plusieurs systèmes informatiques est obtenue via la synchronisation. La synchronisation peut être influencée par plusieurs facteurs, tels que les redémarrages et le trafic réseau entre la source de l’heure et l’ordinateur qui récupère l’heure. 

Azure est soutenu par une infrastructure exécutant Windows Server 2016. Windows Server 2016 utilise des algorithmes améliorés qui permettent de corriger l’heure et de configurer l’horloge locale pour la synchroniser avec l’heure UTC.  Windows Server 2016 a également amélioré le service VMICTimeSync qui détermine comment synchroniser les machines virtuelles avec l’hôte pour obtenir une heure précise. Les améliorations incluent une heure initiale plus précise au démarrage ou lors de la restauration d’une machine virtuelle et l’interruption de la correction de la latence pour les exemples fournis au service de temps Windows (W32time). 


>[!NOTE]
>Pour une présentation rapide du service d’heure de Windows, regardez cette [vidéo de présentation globale](https://aka.ms/WS2016TimeVideo).
>
> Pour plus d’informations, consultez la rubrique [Précision de l’heure sur Windows Server 2016](/windows-server/networking/windows-time-service/accurate-time). 

## <a name="overview"></a>Vue d’ensemble

La précision de l’horloge de l’ordinateur est évaluée en fonction de la proximité de l’horloge de l’ordinateur par rapport à la norme de temps universel coordonné (UTC). L’heure UTC est définie par l’exemple multinational des horloges atomiques précises qui ne peuvent être décalées que d’une seconde en 300 ans. Toutefois, la lecture directe de l’heure UTC requiert du matériel spécialisé. Au lieu de cela, les serveurs de temps sont synchronisés à l’heure UTC et sont accessibles depuis d’autres ordinateurs dans un souci d’évolutivité et de fiabilité. Chaque ordinateur possède un service de synchronisation d’heure en cours d’exécution qui sait quels serveurs de temps utiliser et vérifie régulièrement si l’horloge de l’ordinateur doit être corrigée, et ajuste l’heure si nécessaire. 

Les hôtes Azure sont synchronisés avec les serveurs de temps internes Microsoft qui extraient l’heure depuis des appareils de couche 1 appartenant à Microsoft, grâce à des antennes GPS. Les machines virtuelles dans Azure peuvent dépendre de leur hôte pour transférer l’heure précise (*heure de l’hôte*) sur la machine virtuelle. La machine virtuelle peut également obtenir directement l’heure à partir d’un serveur de temps ou par une combinaison des deux. 

Les interactions de la machine virtuelle avec l’hôte peuvent également influer sur l’horloge. Au cours de la [maintenance avec préservation de la mémoire](../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot), les machines virtuelles sont interrompues jusqu’à 30 secondes. Par exemple, avant le début de la maintenance, l’horloge de la machine virtuelle affiche 10:00:00 pendant 28 secondes. Une fois que la machine virtuelle reprend, l’horloge affiche toujours 10:00:00 et est donc décalée de 28 secondes. Pour corriger cela, le service VMICTimeSync surveille ce qu’il se passe sur l’hôte et demande des modifications sur les machines virtuelles pour compenser.

Le service VMICTimeSync fonctionne en mode échantillon ou synchronisation et impactera uniquement l’horloge vers l’avant. Le mode échantillon, dont l’exécution nécessite W32time, le service VMICTimeSync interroge l’hôte toutes les 5 secondes et fournit des exemples de durée à W32time. Environ toutes les 30 secondes, le service W32time récupère l’échantillon de temps le plus récent et l’utilise pour influencer l’horloge de l’invité. Le mode de synchronisation s’active si un invité a repris son activité ou si l’horloge d’un invité retarde de plus de 5 secondes par rapport à l’horloge de l’ordinateur hôte. Si le service W32time est correctement exécuté, ce dernier cas ne doit jamais se produire.

Si la synchronisation de l’heure ne fonctionne pas, l’horloge de la machine virtuelle accumule les erreurs. S’il n’y a qu’une seule machine virtuelle, l’effet n’est pas forcément significatif, sauf si la charge de travail nécessite une synchronisation extrêmement précise de l’horloge. Mais dans la plupart des cas, nous possédons plusieurs machines virtuelles interconnectées qui utilisent l’heure pour suivre les transactions. L’heure doit donc être cohérente tout au long du déploiement. Lorsque l’heure est différente d’une machine virtuelle à l’autre, vous pouvez rencontrer les effets suivants :

- L’authentification échoue. Les protocoles de sécurité, comme Kerberos, ou les technologies dépendantes d’un certificat reposent sur la précision de l’heure entre les systèmes. 
- Il est très difficile de déterminer ce qu’il peut se passer dans un système si l’heure des journaux d’activité (ou d’autres données) diffère. Un même événement peut avoir l’air de s’être produit à différents moments, rendant la corrélation difficile.
- Si l’horloge est désactivée, la facturation peut être calculée de manière incorrecte.

Les meilleurs résultats des déploiements Windows sont obtenus à l’aide de Windows Server 2016 en tant que système d’exploitation invité, ce qui garantit que vous utilisez les dernières améliorations en matière de synchronisation de l’heure.

## <a name="configuration-options"></a>Options de configuration

Il existe trois options de configuration de la synchronisation de l’heure pour vos machines virtuelles Windows hébergées dans Azure :

- L’heure de l’hôte et time.windows.com. Il s’agit de la configuration par défaut utilisée dans les images de la Place de marché Azure.
- Host-only.
- Utilisez un autre serveur de temps externe, en utilisant ou pas l’heure de l’hôte.


### <a name="use-the-default"></a>Utiliser la valeur par défaut

Par défaut, les images de machine virtuelle du système d’exploitation Windows sont configurées pour w32time afin de synchroniser à partir de deux sources : 

- Le fournisseur NtpClient, qui obtient des informations à partir de time.windows.com.
- Le service VMICTimeSync, utilisé pour communiquer l’heure de l’hôte aux machines virtuelles et apporter des corrections une fois que la machine virtuelle est interrompue pour maintenance. Les hôtes Azure utilisent des appareils de couche 1 appartenant à Microsoft pour maintenir une heure précise.

W32Time privilégie l’ordre de priorité suivant en ce qui concerne le fournisseur de l’heure : niveau de couche, délai de la racine, dispersion de la racine, décalage horaire. Dans la plupart des cas, w32time sur une machine virtuelle Azure préfère l’heure de l’hôte en raison de l’évaluation à effectuer pour comparer les deux sources de temps. 

Pour les ordinateurs joints à un domaine, le domaine lui-même établit la hiérarchie de synchronisation de l’heure, mais la racine de la forêt doit toujours obtenir l’heure depuis un emplacement et les considérations suivantes seraient toujours vraies.


### <a name="host-only"></a>Hôte uniquement 

Étant donné que time.windows.com est un serveur NTP public, son utilisation pour la synchronisation de l’heure nécessite l’envoie du trafic sur Internet. Plusieurs retards de paquets peuvent affecter la qualité de la synchronisation de l’heure. La suppression de time.windows.com en basculant vers la synchronisation host-only peut parfois améliorer vos résultats de synchronisation de l’heure.

Le passage à la synchronisation de l’heure avec l’hôte uniquement est judicieux si vous rencontrez des problèmes de synchronisation de l’heure avec la configuration par défaut. Essayez la synchronisation host-only pour voir si cela améliore la synchronisation de l’heure sur la machine virtuelle. 

### <a name="external-time-server"></a>Serveur d’heure externe

Si vous avez des exigences spécifiques concernant la synchronisation de l’heure, vous pouvez également utiliser des serveurs de temps externes. Les serveurs de temps externes peuvent fournir une heure spécifique, ce qui s’avère parfois utile pour les scénarios de test. Ils garantissent l’uniformité de l’heure avec les machines hébergées dans des centres de données non Microsoft ou gèrent les secondes intercalaires d’une façon particulière.

Vous pouvez combiner des serveurs externes avec le service VMICTimeSync et VMICTimeProvider pour fournir des résultats similaires à la configuration par défaut. 

## <a name="check-your-configuration"></a>Vérifier votre configuration


Vérifiez si le fournisseur de temps NtpClient est configuré pour utiliser des serveurs NTP explicites (NTP) ou la synchronisation de l’heure de domaines (NT5DS).

```
w32tm /dumpreg /subkey:Parameters | findstr /i "type"
```

Si la machine virtuelle utilise NTP, vous verrez la sortie suivante :

```
Value Name                 Value Type          Value Data
Type                       REG_SZ              NTP
```

Pour voir le serveur de temps utilisé par le fournisseur de temps NtpClient, à une invite de commandes avec élévation de privilèges, tapez :

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

Si la machine virtuelle utilise les valeurs par défaut, la sortie ressemble à ceci :

```
NtpServer                  REG_SZ              time.windows.com,0x8
```


Pour voir le fournisseur de temps actuellement utilisé.

```
w32tm /query /source
```


Voici la sortie que vous pouvez voir et ce que cela signifie :
    
- **time.windows.com** : dans la configuration par défaut, w32time doit obtenir l’heure par time.windows.com. La qualité de la synchronisation de l’heure dépend de sa connectivité Internet et est affectée par les retards de paquets. Il s’agit de la sortie habituelle que vous obtenez sur une machine physique.
- **Fournisseur de synchronisation de l’heure IC de machine virtuelle** : la machine virtuelle synchronise l’heure à partir de l’hôte. Il s’agit de la sortie habituelle que vous obtenez sur une machine virtuelle s’exécutant sur Azure. 
- *Votre serveur de domaine* : la machine actuelle est dans un domaine, et le domaine définit la hiérarchie de synchronisation de l’heure.
- *Un autre serveur* : w32time est explicitement configuré pour obtenir l’heure à partir de l’autre serveur. La qualité de synchronisation de l’heure dépend de la qualité de ce serveur de temps.
- **Horloge CMOS locale** : l’horloge n’est pas synchronisée. Vous pouvez obtenir cette sortie si w32time n’a pas eu suffisamment de temps pour démarrer après un redémarrage, ou lorsqu’aucune source temporelle configurée n’est disponible.


## <a name="opt-in-for-host-only-time-sync"></a>Choisir la synchronisation de l’heure host-only

Azure travaille en permanence à l’amélioration de la synchronisation de l’heure sur les ordinateurs hôtes et garantit que toutes les infrastructures de synchronisation de l’heure sont colocalisées dans les centres de données appartenant à Microsoft. Si vous rencontrez des problèmes de synchronisation de l’heure avec la configuration par défaut, qui préfère utiliser time.windows.com comme principale source temporelle, vous pouvez utiliser les commandes suivantes pour choisir la synchronisation de l’heure host-only.

Indiquez que le fournisseur VMIC est activé. 

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\VMICTimeProvider /v Enabled /t REG_DWORD /d 1 /f
```

Indiquez que le fournisseur NTPClient est désactivé.

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpClient /v Enabled /t REG_DWORD /d 0 /f
```

Redémarrez le service w32time.

```
net stop w32time && net start w32time
```


## <a name="windows-server-2012-and-r2-vms"></a>Machines virtuelles Windows Server 2012 et Windows Server 2012 R2 

Windows Server 2012 et Windows Server 2012 R2 possèdent des paramètres par défaut différents pour la synchronisation de l’heure. Le w32time par défaut est configuré d’une telle manière qu’il préfère une faible surcharge du service à une heure précise. 

Si vous souhaitez déplacer vos déploiements Windows Server 2012 et 2012 R2 pour utiliser les valeurs par défaut plus récentes qui privilègient une heure précise, vous pouvez appliquer les paramètres suivants.

Mettez à jour l’interrogation w32time ainsi que les intervalles pour qu’ils correspondent aux paramètres de Windows Server 2016.

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MinPollInterval /t REG_DWORD /d 6 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v MaxPollInterval /t REG_DWORD /d 10 /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config /v UpdateInterval /t REG_DWORD /d 100 /f
w32tm /config /update
```

Pour que w32time soit en mesure d’utiliser les nouveaux intervalles d’interrogation, les NtpServers doivent être marqués comme les utilisant. Si les serveurs sont annotés avec le masque d’indicateur de bits 0x1, le mécanisme devrait être remplacé, et w32time utiliserait SpecialPollInterval à la place. Assurez-vous que les serveurs NTP spécifiés utilisent l’indicateur 0x8 ou aucun indicateur :

Vérifiez quels indicateurs sont utilisés pour les serveurs NTP utilisés.

```
w32tm /dumpreg /subkey:Parameters | findstr /i "ntpserver"
```

## <a name="next-steps"></a>Étapes suivantes

Voici des liens pour plus d’informations sur la synchronisation de l’heure :

- [Paramètres et outils du service de temps Windows](/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings)
- [Améliorations de Windows Server 2016](/windows-server/networking/windows-time-service/windows-server-2016-improvements)
- [Heure précise de Windows Server 2016](/windows-server/networking/windows-time-service/accurate-time)
- [Limites de prise en charge pour configurer le service de temps Windows pour les environnements de haute-précision](/windows-server/networking/windows-time-service/support-boundary)
