---
title: Fichier include
description: Fichier include
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 11/12/2019
ms.author: cynthn;kareni
ms.custom: include file
ms.openlocfilehash: a9146099951aba223a7b201c1613e1ec0ba617d4
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96509466"
---
**Dernière mise à jour du document** : 12 novembre 2019 10:00 PST.

La divulgation d’une [nouvelle classe de vulnérabilités de processeur](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002) appelées attaques par canal latéral de l’exécution spéculative a généré des questions de la part de clients recherchant plus d’explications.  

Microsoft a déployé des solutions d’atténuation des risques sur l’ensemble de ses services cloud. L’infrastructure qui exécute Azure et isole les différentes charges de travail des clients est protégée. Cela signifie qu’un éventuel attaquant utilisant la même infrastructure ne peut pas attaquer votre application à l’aide de ces vulnérabilités.

Azure utilise autant que possible la [maintenance avec préservation de la mémoire](../articles/virtual-machines/maintenance-and-updates.md?bc=%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json%252c%2fazure%2fvirtual-machines%2fwindows%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json%253ftoc%253d%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#maintenance-that-doesnt-require-a-reboot) afin de réduire l’impact sur le client et d’éliminer la nécessité de redémarrages. Azure continuera à utiliser ces méthodes lors des mises à jour à l’échelle du système sur l’hôte et à protéger nos clients.

Plus d’informations sur l’intégration de la sécurité dans chaque aspect d’Azure sont disponibles sur le site de [documentation sur la sécurité Azure](../articles/security/index.yml). 

> [!NOTE] 
> Depuis la première publication de ce document, plusieurs variantes de cette classe de vulnérabilités ont été divulguées. Microsoft poursuit ses efforts d’investissement dans la protection de nos clients et de conseils. Cette page sera mise à jour à mesure que nous publierons d’autres correctifs. 
> 
> Le 12 novembre 2019, [Intel a publié](https://software.intel.com/security-software-guidance/insights/deep-dive-intel-transactional-synchronization-extensions-intel-tsx-asynchronous-abort) un avis technique sur la vulnérabilité TAA (Transaction Asynchronous Abort) de l’extension Intel® TSX (Transactional Synchronization Extensions) qui porte le numéro [CVE-2019-11135](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-11135). Cette vulnérabilité affecte les processeurs Intel® Core® et Intel® Xeon®.  Microsoft Azure a publié des mises à jour de système d’exploitation et déploie le nouveau microcode, dès sa mise à disposition par Intel, sur notre parc pour protéger nos clients face à ces nouvelles vulnérabilités.   Azure travaille en étroite collaboration avec Intel pour tester et valider le nouveau microcode avant sa publication officielle sur la plateforme. 
>
> **Les clients qui exécutent du code non approuvé sur leur machine virtuelle** doivent prendre des mesures pour se protéger contre ces vulnérabilités en lisant ce qui suit pour en savoir plus sur les vulnérabilités de canal côté exécution spéculative (Microsoft Advisories ADV [180002](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV180002), [180018](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/adv180018) et [190013](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/ADV190013)).
>
> Les autres clients doivent évaluer ces vulnérabilités d’un point de vue de la Défense en profondeur et prendre en compte les conséquences en termes de sécurité et de performances de la configuration choisie.
> 



## <a name="keeping-your-operating-systems-up-to-date"></a>Maintien à jour de vos systèmes d’exploitation

Alors qu’une mise à jour du système d’exploitation n’est pas nécessaire pour isoler vos applications s’exécutant sur Azure d’autres clients Azure, il est toujours judicieux de maintenir vos logiciels à jour. Les derniers correctifs cumulatifs de sécurité pour Windows contiennent des atténuations des risques pour plusieurs vulnérabilités de canal côté exécution spéculative. De même, les distributions Linux ont mis à disposition plusieurs mises à jour pour résoudre ces vulnérabilités. Vous trouverez ci-dessous nos actions recommandées pour mettre à jour votre système d’exploitation :

| Offre | Action recommandée  |
|----------|---------------------|
| Services cloud Azure  | Activez la [mise à jour automatique](../articles/cloud-services/cloud-services-how-to-configure-portal.md) ou vérifiez que vous exécutez le dernier système d’exploitation invité. |
| Machines virtuelles Linux Azure | Installez les mises à jour de votre fournisseur de système d’exploitation. Pour plus d’informations, consultez [Linux](#linux) plus loin dans ce document. |
| Machines virtuelles Windows Azure  | Installez le dernier correctif cumulatif de sécurité.
| Autres services PaaS Azure | Aucune action n’est requise de la part des clients utilisant ces services. Azure maintient automatiquement à jour les versions du système d’exploitation. |

## <a name="additional-guidance-if-you-are-running-untrusted-code"></a>Conseils supplémentaires si vous exécutez du code non approuvé 

Les clients qui autorisent les utilisateurs non approuvés à exécuter du code arbitraire peuvent souhaiter implémenter des fonctionnalités de sécurité supplémentaires dans leurs machines virtuelles Azure ou services cloud. Ces fonctionnalités protègent contre les vecteurs de divulgation intra-processus décrits par plusieurs vulnérabilités d’exécution spéculative.

Exemples de scénarios dans lesquels des fonctionnalités de sécurité supplémentaires sont recommandées :

- Vous autorisez l’exécution de code non approuvé dans votre machine virtuelle.  
    - *Par exemple, vous autorisez un de vos clients à charger un fichier binaire ou un script que vous exécutez ensuite dans votre application*. 
- Vous autorisez des utilisateurs qui ne sont pas approuvés à se connecter à votre machine virtuelle à l’aide de comptes à privilèges faibles.   
    - *Par exemple, vous autorisez un utilisateur à privilèges faibles de vous connecter à l’une de vos machines virtuelles à l’aide du Bureau à distance ou SSH*.  
- Vous autorisez des utilisateurs non approuvés à accéder à des machines virtuelles implémentées par le biais de la virtualisation imbriquée.  
    - *Par exemple, vous contrôlez l’hôte Hyper-V mais vous allouez les machines virtuelles à des utilisateurs non approuvés*. 

Les clients qui n’implémentent pas un scénario impliquant du code non approuvé ne doivent pas nécessairement activer ces fonctionnalités de sécurité supplémentaires. 

## <a name="enabling-additional-security"></a>Activer la sécurité supplémentaire 

Vous pouvez activer des fonctionnalités de sécurité supplémentaires dans votre machine virtuelle ou service cloud si vous exécutez du code non approuvé. En parallèle, vérifiez que votre système d’exploitation est à jour pour activer les fonctionnalités de sécurité au sein de votre machine virtuelle ou de votre service Cloud

### <a name="windows"></a>Windows 

Votre système d’exploitation cible doit être à jour pour pouvoir activer ces fonctionnalités de sécurité supplémentaires. Bien que de nombreuses atténuations des risques de canal côté exécution spéculative soient activées par défaut, les fonctionnalités supplémentaires décrites ici doivent être activées manuellement et peuvent affecter les performances. 


**Étape 1 : Désactivez Hyper-Threading sur la machine virtuelle** : les clients qui exécutent du code non approuvé sur une machine virtuelle multithreads doivent désactiver la fonctionnalité Hyper-Threading ou passer à une taille de machine virtuelle non multithreads. Consutlez [ce document](../articles/virtual-machines/acu.md) pour obtenir la liste des tailles de machine virtuelle multithreads (dont le rapport processeur virtual/cœur est 2:1). Pour vérifier si la fonctionnalité Hyper-Threading est activée sur votre machine, reportez-vous au script ci-dessous lorsque vous utilisez la ligne de commande Windows à partir de la machine virtuelle.

Entrez `wmic` pour ouvrir l’interface interactive. Puis, entrez ce qui suit pour afficher la quantité de processeurs physiques et logiques sur la machine virtuelle.

```console
CPU Get NumberOfCores,NumberOfLogicalProcessors /Format:List
```

Si le nombre de processeurs logiques est supérieur à celui des processeurs physiques (cœurs), la fonctionnalité Hyper-Threading est activée.  Si vous exécutez une machine virtuelle multithreads, [contactez le support Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) pour obtenir la désactivation de la fonctionnalité Hyper-Threading.  Une fois celle-ci désactivée, **la prise en charge de cette modification requiert un redémarrage complet de la machine virtuelle**. Reportez-vous à [Nombre de cœurs](#core-count) pour comprendre pourquoi le nombre de cœurs de la machine virtuelle a diminué.


**Étape 2** : Parallèlement à l’étape 1, suivez les instructions de l’article [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) pour vérifier que les protections sont activées à l’aide du module PowerShell [SpeculationControl](https://aka.ms/SpeculationControlPS).

> [!NOTE]
> Si vous avez déjà téléchargé ce module, vous devez installer la dernière version.
>


Le résultat du script PowerShell doit comporter les valeurs ci-dessous pour valider les protections activées permettant de lutter contre ces vulnérabilités :

```
Windows OS support for branch target injection mitigation is enabled: True
Windows OS support for kernel VA shadow is enabled: True
Windows OS support for speculative store bypass disable is enabled system-wide: False
Windows OS support for L1 terminal fault mitigation is enabled: True
Windows OS support for MDS mitigation is enabled: True
Windows OS support for TAA mitigation is enabled: True
```

Si la sortie comporte `MDS mitigation is enabled: False`, [contactez le support Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) pour prendre connaissance des options d’atténuation disponibles.



**Étape 3** : Pour activer la fonctionnalité Kernel Virtual Address (KVA) Shadowing (Copie shadow d’adresse virtuelle du noyau) et la prise en charge de système d’exploitation Branch Target Injection (BTI), suivez les instructions figurant dans [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) pour activer la protection à l’aide des clés de `Session Manager`. Un redémarrage est nécessaire.


**Étape 4** : Pour les déploiements utilisant la [virtualisation imbriquée](../articles/virtual-machines/windows/nested-virtualization.md) (D3 et E3 uniquement) : Ces instructions s’appliquent dans la machine virtuelle que vous utilisez comme hôte Hyper-V.

1.  Suivez les instructions de l’article [KB4072698](https://support.microsoft.com/help/4072698/windows-server-guidance-to-protect-against-the-speculative-execution) pour activer les protections via les clés de Registre `MinVmVersionForCpuBasedMitigations`.
2.  Définissez le type de planificateur d’hyperviseur sur `Core` en suivant les instructions fournies [ici](/windows-server/virtualization/hyper-v/manage/manage-hyper-v-scheduler-types).


### <a name="linux"></a>Linux

<a name="linux"></a>L’activation de l’ensemble de fonctionnalités de sécurité supplémentaires nécessite que le système d’exploitation cible soit entièrement à jour. Certaines atténuations des risques sont activées par défaut. La section suivante décrit les fonctionnalités qui sont désactivées par défaut et/ou qui dépendent de la prise en charge matérielle (microcode). L’activation de ces fonctionnalités peut affecter les performances. Consultez la documentation du fournisseur de votre système d’exploitation pour obtenir des instructions


**Étape 1 : Désactivez Hyper-Threading sur la machine virtuelle** : les clients qui exécutent du code non approuvé sur une machine virtuelle mutithreads doivent désactiver la fonctionnalité Hyper-Threading ou passer à une machine virtuelle non multithreads.  Consutlez [ce document](../articles/virtual-machines/acu.md) pour obtenir la liste des tailles de machine virtuelle multithreads (dont le rapport processeur virtual/cœur est 2:1). Pour vérifier si vous exécutez une machine virtuelle multithreads, exécutez la commande `lscpu` dans la machine virtuelle Linux. 

Si `Thread(s) per core = 2`, la fonctionnalité Hyper-Threading a été activée. 

Si `Thread(s) per core = 1`, la fonctionnalité Hyper-Threading a été désactivée. 

 
Exemple de sortie pour une machine virtuelle sur laquelle la fonctionnalité Hyper-Threading est activée : 

```console
CPU Architecture:      x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0-7
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1

```

Si vous exécutez une machine virtuelle multithreads, [contactez le support Azure](https://aka.ms/MicrocodeEnablementRequest-SupportTechnical) pour obtenir la désactivation de la fonctionnalité Hyper-Threading.  Une fois celle-ci désactivée, **la prise en charge de cette modification requiert un redémarrage complet de la machine virtuelle**. Reportez-vous à [Nombre de cœurs](#core-count) pour comprendre pourquoi le nombre de cœurs de la machine virtuelle a diminué.



**Étape 2** : Pour limiter les vulnérabilités de canal côté exécution spéculative, reportez-vous à la documentation du fournisseur de votre système d’exploitation :   
 
- [Redhat et CentOS](https://access.redhat.com/security/vulnerabilities) 
- [SUSE](https://www.suse.com/support/kb/?doctype%5B%5D=DT_SUSESDB_PSDB_1_1&startIndex=1&maxIndex=0) 
- [Ubuntu](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/) 


### <a name="core-count"></a>Nombre de cœurs

Lors de la création d’une machine virtuelle multithreads, Azure alloue 2 threads par cœur, que l’on appelle processeurs virtuels. Lorsque la fonctionnalité Hyper-Threading est désactivée, Azure supprime un thread et monte des cœurs multithreads uniques (cœurs physiques). Le ratio processeur virtuel/processeur est de 2:1. Ainsi, une fois la fonctionnalité Hyper-Threading désactivée, le nombre de processeurs de la machine virtuelle semble avoir diminué de moitié. Par exemple, une machine virtuelle D8_v3 est une machine virtuelle multithreads s’exécutant sur 8 processeurs virtuels (2 threads par cœurs x 4 cœurs).  Lorsque la fonctionnalité Hyper-Threading est désactivée, les processeurs passent à 4 cœurs physiques avec 1 thread par cœur. 

## <a name="next-steps"></a>Étapes suivantes

Cet article donne des conseils relatifs aux attaques par canal côté exécution spéculative, dont de nombreux processeurs modernes sont la cible :

[Spectre Meltdown](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180002) :
- CVE-2017-5715 - Branch Target Injection (BTI)  
- CVE-2017-5754 - Kernel Page Table Isolation (KPTI)
- CVE-2018-3639 – Speculative Store Bypass (KPTI) 
- [CVE-2019-1125](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-1125) – informations sur le noyau Windows – variante de spectre 1
 
[L1 Erreur de Terminal (L1TF)](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV180018) :
- CVE-2018-3615 - Intel Software Guard Extensions (Intel SGX)
- CVE-2018-3620 - Operating Systems (OS) and System Management Mode (SMM)
- CVE-2018-3646 – impacts Virtual Machine Manager (VMM)

[Microarchitectural Data Sampling](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190013) : 
- CVE-2019-11091 - Microarchitectural Data Sampling Uncacheable Memory (MDSUM)
- CVE-2018-12126 - Microarchitectural Store Buffer Data Sampling (MSBDS)
- CVE-2018-12127 - Microarchitectural Load Port Data Sampling (MLPDS)
- CVE-2018-12130 - Microarchitectural Fill Buffer Data Sampling (MFBDS)

Transaction Asynchronous Abort de l’extension Transactional Synchronization Extensions (Intel® TSX) :  
- [CVE-2019-11135](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2019-11135) – TSX Transaction Asynchronous Abort (TAA)