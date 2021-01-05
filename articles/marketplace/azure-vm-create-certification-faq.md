---
title: Résolution des problèmes de certification de machine virtuelle pour Place de marché Azure
description: Résolvez les problèmes courants liés au test et à la certification des images de machine virtuelle pour Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: troubleshooting
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 10/19/2020
ms.openlocfilehash: bc1ae4bc2cf64c3e2f996709c086eb23cb8b8385
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602595"
---
# <a name="troubleshoot-virtual-machine-certification"></a>Résoudre les problèmes de certification des machines virtuelles

Lorsque vous publiez l’image d’une machine virtuelle sur Place de marché Azure, l’équipe Azure la valide pour s’assurer qu’elle est démarrable, sécurisée et compatible avec Azure. Si votre image de machine virtuelle échoue à l’un des tests de haute qualité, elle ne sera pas publiée. Vous recevrez un message d’erreur qui décrit le problème.

Cet article présente les messages d'erreur les plus couramment rencontrés lors de la publication d'images de machine virtuelle, ainsi que les solutions associées.

> [!NOTE]
> Si vous avez des questions sur cet article ou des suggestions d’amélioration, contactez le [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport).

## <a name="approved-base-image"></a>Image de base approuvée

Lorsque vous envoyez une demande de nouvelle publication de votre image avec des mises à jour, le cas de test de vérification du numéro de référence peut échouer. Votre image ne sera alors pas approuvée.

Cette erreur se produit lorsque vous utilisez une image de base qui appartient à un autre éditeur et que vous avez procédé à une mise à jour de l'image. Dans ce cas, vous n’êtes pas autorisé à publier votre image.

Pour résoudre ce problème, récupérez l'image à partir de la Place de marché Azure et apportez-y des modifications. Pour plus d’informations, consultez les articles suivants :

- [Images Linux](../virtual-machines/linux/endorsed-distros.md?toc=/azure/virtual-machines/linux/toc.json)
- [Images Windows](azure-vm-create-using-approved-base.md)

> [!Note]
> Si vous utilisez une image de base Linux qui n’est pas extraite de Place de marché Azure, vous pouvez décaler la première partition de 2048 ko. Cela permet d’utiliser l’espace non formaté pour ajouter de nouvelles informations de facturation et permet à Azure de procéder à la publication de votre machine virtuelle sur Place de marché Azure.  

## <a name="vm-extension-failure"></a>Échec de l’extension de machine virtuelle

Déterminez si votre image prend en charge les extensions de machine virtuelle.

Pour activer les extensions de machine virtuelle :

1. Sélectionnez votre machine virtuelle Linux.
1. Accédez à **Paramètres de diagnostic**.
1. Activez les matrices de base en mettant à jour le **compte de stockage**.
1. Sélectionnez **Enregistrer**.

   ![Capture d’écran montrant comment activer la supervision au niveau de l’invité.](./media/create-vm/vm-certification-issues-solutions-1.png)

Pour vérifier que les extensions de machine virtuelle sont correctement activées :

1. Dans la machine virtuelle, sélectionnez l'onglet **Extensions de machine virtuelle**, puis vérifiez l'état de l'**Extension de diagnostic Linux**.
1. Vérifiez l’état de l’approvisionnement.

   - Si l'état est *Approvisionnement réussi*, le cas de test Extensions est réussi.  
   - Si l’état est *Échec de l’approvisionnement*, le cas de test Extensions est un échec et vous devez définir l’indicateur Renforcé.

   ![Capture d’écran montrant un approvisionnement réussi.](./media/create-vm/vm-certification-issues-solutions-2.png)

   Si l'extension de machine virtuelle échoue, accédez à [Utiliser l'Extension de diagnostic Linux pour superviser les métriques et les journaux](../virtual-machines/extensions/diagnostics-linux.md) afin de l'activer. Si vous ne souhaitez pas que l'extension de machine virtuelle soit activée, contactez l'équipe du support technique et demandez-lui de la désactiver.

## <a name="vm-provisioning-issue"></a>Problème d'approvisionnement de machine virtuelle

Assurez-vous d'avoir rigoureusement suivi le processus d'approvisionnement de la machine virtuelle avant de soumettre votre offre. Pour afficher le format JSON à des fins d’approvisionnement de la machine virtuelle, consultez [Tester une image de machine virtuelle](azure-vm-image-test.md).

Les problèmes de provisionnement peuvent inclure les scénarios d’échec suivants :

|Scénario|Error|Motif|Solution|
|---|---|---|---|
|1|Disque dur virtuel (VHD) non valide|Si la valeur du cookie spécifiée dans le pied de page du disque dur virtuel est incorrecte, le disque dur virtuel sera considéré comme non valide.|Recréez l'image et envoyez la demande.|
|2|Type d'objet blob non valide|L'approvisionnement de la machine virtuelle a échoué car le bloc utilisé est un type d'objet blob et non un type de page.|Recréez l'image et envoyez la demande.|
|3|Expiration du délai de provisionnement ou généralisation incorrecte|Il y a un problème avec la généralisation de la machine virtuelle.|Recréez l'image avec la généralisation et envoyez la demande.|

> [!NOTE]
> Pour plus d'informations sur la généralisation d'une machine virtuelle, consultez :
> - [Documentation Linux](azure-vm-create-using-approved-base.md#generalize-the-image)
> - [Documentation Windows](../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep)


## <a name="vhd-specifications"></a>Spécifications du disque dur virtuel (VHD)

### <a name="conectix-cookie-and-other-vhd-specifications"></a>Cookie Conectix et autres spécifications du disque dur virtuel

La chaîne « conectix » fait partie de la spécification du disque dur virtuel. Elle est définie en tant que cookie de 8 octets dans le pied de page du disque dur virtuel qui identifie le créateur du fichier. Tous les fichiers VHD créés par Microsoft possèdent ce cookie. 

Un blob au format VHD doit avoir un pied de page de 512 octets au format suivant :

|Champs du pied de page du disque dur|Taille (en octets)|
|---|---|
Cookie|8
Fonctionnalités|4
Version de format du fichier|4
Décalage des données|8
Horodatage|4
Application de création|4
Version de création|4
Système d’exploitation hôte de création|4
Taille d'origine|8
Taille actuelle|8
Géométrie du disque|4
Type du disque|4
Somme de contrôle|4
ID unique|16
État enregistré|1
Réservé|427


### <a name="vhd-specifications"></a>Spécifications du disque dur virtuel (VHD)

Pour garantir une expérience de publication fluide, assurez-vous que le disque dur virtuel répond aux critères suivants :

- Le cookie contient la chaîne « conectix ».
- Le type de disque est fixe.
- La taille virtuelle du disque dur virtuel est d’au moins 20 Mo.
- Le disque dur virtuel est aligné. La taille virtuelle doit être un multiple de 1 Mo.
- La longueur du blob VHD est égale à la taille virtuelle plus la longueur du pied de page VHD (512).

Téléchargez la [spécification VHD](https://www.microsoft.com/download/details.aspx?id=23850).

## <a name="software-compliance-for-windows"></a>Conformité logicielle pour Windows

Si votre demande d’image Windows est rejetée en raison d’un problème de conformité logicielle, vous avez peut-être créé une image Windows sur laquelle une instance SQL est installée. Au lieu de cela, vous devez prendre l’image de base de la version SQL Server appropriée accessible sur Place de marché Azure.

Ne créez pas d’image Windows dotée de SQL Server. Utilisez les images de base SQL Server approuvées (Entreprise/Standard/Web) provenant de Place de marché Azure.

Si vous essayez d'installer Visual Studio ou un produit sous licence Office, contactez l'équipe du support technique pour obtenir une approbation préalable.

Pour plus d’informations sur la sélection d’une base approuvée, consultez [Créer une machine virtuelle à partir d’une base approuvée](azure-vm-create-using-approved-base.md).

## <a name="toolkit-test-case-execution-failed"></a>Échec de l’exécution du cas de test du kit de ressources

Le kit de ressources de certification Microsoft peut vous aider à exécuter des cas de test et à vérifier que votre disque dur virtuel ou votre image est compatible avec l'environnement Azure.

Téléchargez le [kit de ressources de certification Microsoft](azure-vm-image-test.md).

### <a name="linux-test-cases"></a>Cas de test Linux

Le tableau suivant répertorie les cas de test Linux que le kit de ressources exécutera. La validation de test est indiquée dans la description.

|Scénario|Cas de test|Description|
|---|---|---|
|1|Historique Bash|Les fichiers d'historique Bash doivent être effacés avant la création de l'image de machine virtuelle.|
|2|Version de l'agent Linux|L'agent Linux Azure 2.2.41 ou version ultérieure doit être installé.|
|3|Paramètres de noyau requis|Vérifie que les paramètres de noyau suivants sont définis : <br>console=ttyS0<br>earlyprintk=ttyS0<br>rootdelay=300 |
|4|Partition d'échange sur le disque du système d'exploitation|Vérifie qu’aucune partition d’échange n’est créée sur le disque du système d’exploitation.|
|5|Partition racine sur le disque du système d'exploitation|Créez une partition racine unique pour le disque de système d’exploitation.|
|6|Version d'OpenSSL|OpenSSL version 0.9.8 ou ultérieure doit être installé.|
|7|Version Python|Python version 2.6 ou ultérieure est fortement recommandé.|
|8|Intervalle d’activité du client|Affectez la valeur 180 à ClientAliveInterval. Pour les besoins de l'application, vous pouvez affecter une valeur comprise entre 30 et 235. Si vous activez le protocole SSH pour vos utilisateurs finaux, cette valeur doit être définie comme expliqué.|
|9|Architecture du système d’exploitation|Seuls les systèmes d’exploitation 64 bits sont pris en charge.|
|10|Mise à jour automatique|Indique si la mise à jour automatique de l’agent Linux est activée.|

### <a name="common-test-case-errors"></a>Erreurs courantes de cas de test

Reportez-vous au tableau suivant pour connaître les erreurs courantes que vous pouvez rencontrer lors de l’exécution de cas de test :

| Scénario | Cas de test | Error | Solution |
| --- | --- | --- | --- |
| 1 | Cas de test Version de l'agent Linux | La version minimale de l’agent Linux est 2.2.41 ou ultérieure. Cette exigence est obligatoire depuis le 1er mai 2020. | Mettez à jour la version de l’agent Linux. Il doit s’agir de la version 2.241 ou ultérieure. Pour plus d’informations, consultez la [page de mise à jour de la version de l’agent Linux](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support). |
| 2 | Cas de test Historique Bash | Une erreur se produit si la taille de l’historique Bash de l’image que vous avez envoyée est supérieure à 1 kilo-octet (ko). La taille est limitée à 1 ko pour garantir que votre fichier d’historique Bash ne contient pas d’informations potentiellement sensibles. | Résolvez l’erreur en montant le disque dur virtuel sur une autre machine virtuelle opérationnelle et apportez des modifications pour réduire la taille à 1 ko ou moins. Par exemple, supprimez les fichiers d’historique `.bash`. |
| 3 | Cas de test Paramètres de noyau requis | Vous recevez cette erreur quand la valeur de `console` n’est pas `ttyS0`. Vérifiez en exécutant la commande suivante : <br /> `cat /proc/cmdline` | Affectez la valeur `ttyS0` à `console` et renvoyez la demande. |
| 4 | Cas de test Intervalle d'activité du client | Si le résultat du kit de ressources est un échec pour ce cas de test, cela signifie que la valeur de `ClientAliveInterval` est incorrecte. | Affectez une valeur inférieure ou égale à 235 à `ClientAliveInterval`, puis renvoyez la demande. |


### <a name="windows-test-cases"></a>Cas de test Windows

Le tableau suivant répertorie les cas de test Windows que le kit de ressources exécutera, et fournit une description de la validation du test :

|Scénario |Scénarios de test|Description|
|---|---|---|---|
|1|Architecture du système d’exploitation|Azure prend uniquement en charge les systèmes d'exploitation 64 bits.|
|2|Dépendance de compte d’utilisateur|L'exécution de l'application ne doit pas dépendre du compte d'administrateur.|
|3|Cluster de basculement|La fonctionnalité de clustering de basculement Windows Server n'est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|4|IPV6|Le protocole IPv6 n’est pas encore pris en charge dans l’environnement Azure. L’application ne doit pas être dépendante de cette fonctionnalité.|
|5|DHCP|Le rôle serveur DHCP (Dynamic Host Configuration Protocol) n’est pas encore pris en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|6|Hyper-V|Le rôle serveur Hyper-V n’est pas encore pris en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|7|Accès à distance|Le rôle serveur Accès à distance (Accès direct) n’est pas encore pris en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|8|Rights Management Services|Rights Management Services. Le rôle serveur n’est pas encore pris en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|9|Services de déploiement Windows|Services de déploiement Windows. Le rôle serveur n’est pas encore pris en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|10|Chiffrement de lecteur BitLocker|Le chiffrement de lecteur BitLocker n'est pas pris en charge sur le disque dur du système d'exploitation, mais peut être utilisé sur les disques de données.|
|11|Serveur de nom de stockage Internet|La fonctionnalité Serveur de nom de stockage Internet n'est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|12|MPIO (Multipath I/O)|MPIO (Multipath I/O). Cette fonctionnalité de serveur n’est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|13|Équilibrage de charge réseau|Équilibrage de charge réseau. Cette fonctionnalité de serveur n’est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|14|Protocole PNRP|Protocole PNRP. Cette fonctionnalité de serveur n’est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|15|Services SNMP|La fonctionnalité Services SNMP n'est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|16|Service WINS (Windows Internet Name Service)|Service WINS (Windows Internet Name Service). Cette fonctionnalité de serveur n’est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|
|17|Service de réseau local sans fil|Service de réseau local sans fil. Cette fonctionnalité de serveur n’est pas encore prise en charge. L’application ne doit pas être dépendante de cette fonctionnalité.|

Si vous observez des échecs avec les cas de test précédents, consultez la colonne **Description** du tableau pour connaître la solution. Pour plus d’informations, contactez l’équipe du support technique. 

## <a name="data-disk-size-verification"></a>Vérification de la taille du disque de données

Les demandes de disques de données d’une taille supérieure à 1023 gigaoctets (Go) ne sont pas approuvées. Cette règle s'applique à la fois à Linux et à Windows.

Renvoyez la demande avec une taille inférieure ou égale à 1023 Go.

## <a name="os-disk-size-validation"></a>Validation de la taille du disque du système d’exploitation

Pour plus d’informations sur les limitations relatives à la taille du disque du système d’exploitation, consultez les règles suivantes. Quand vous envoyez une demande, vérifiez que la taille du disque du système d’exploitation est comprise dans les limites de Linux ou de Windows.

|Système d''exploitation|Taille de disque dur virtuel recommandée|
|---|---|
|Linux|1 à 1023 Go|
|Windows|30 à 250 Go|

Comme les machines virtuelles autorisent l’accès au système d’exploitation sous-jacent, assurez-vous que la taille du disque dur virtuel est suffisamment grande pour le disque dur virtuel. Les disques ne sont pas extensibles sans temps d’arrêt. Utilisez une taille de disque comprise entre 30 Go et 50 Go.

|Taille de disque dur virtuel|Taille réelle occupée|Solution|
|---|---|---|
|> 500 tébioctets (Tio).|n/a|Contactez l'équipe du support technique pour obtenir une approbation d'exception.|
|250 à 500 Tio|> 200 gibioctets (Gio) de différence par rapport à la taille de l'objet blob|Contactez l'équipe du support technique pour obtenir une approbation d'exception.|

> [!NOTE]
> Les disques de plus grande taille génèrent des coûts plus élevés et entraînent un retard pendant le processus d’installation et de réplication. En raison de ce retard et de ce coût, l'équipe du support technique est susceptible de demander la justification de l'approbation d'exception.

## <a name="wannacry-patch-verification-test-for-windows"></a>Test de vérification de correctif WannaCry pour Windows

Pour prévenir toute attaque liée au virus WannaCry, veillez à ce que toutes les demandes d'image Windows soient mises à jour avec le correctif le plus récent.

Vous pouvez vérifier la version du fichier image à partir de `C:\windows\system32\drivers\srv.sys` ou `srv2.sys`.

Le tableau suivant présente la version corrigée minimale de Windows Server : 

|Système d''exploitation|Version|
|---|---|
|Windows Server 2008 R2|6.1.7601.23689|
|Windows Server 2012|6.2.9200.22099|
|Windows Server 2012 R2|6.3.9600.18604|
|Windows Server 2016|10.0.14393.953|
|Windows Server 2019|N/D|

> [!NOTE]
> Aucune spécification de version obligatoire ne s'applique à Windows Server 2019.

## <a name="sack-vulnerability-patch-verification"></a>Vérification du correctif de vulnérabilité SACK

Lorsque vous envoyez une image Linux, votre demande peut être rejetée en raison de problèmes liés à la version du noyau.

Mettez à jour le noyau avec une version approuvée et renvoyez la demande. Vous trouverez la version du noyau approuvée dans le tableau suivant. Le numéro de version doit être supérieur ou égal à celui indiqué ici.

Si votre image n'est pas installée avec l'une des versions de noyau suivantes, mettez-la à jour avec les correctifs appropriés. Demandez l'approbation nécessaire à l'équipe du support technique une fois que l'image a été mise à jour avec les correctifs requis suivants :

- CVE-2019-11477 
- CVE-2019-11478 
- CVE-2019-11479

|Famille de système d’exploitation|Version|Noyau|
|---|---|---|
|Ubuntu|14.04 LTS|4.4.0-151| 
||14.04 LTS|4.15.0-1049-*-azure|
||LTS 16.04|4.15.0-1049|
||18.04 LTS|4.18.0-1023|
||18.04 LTS|5.0.0-1025|
||18.10|4.18.0-1023|
||19.04|5.0.0-1010|
||19.04|5.3.0-1004|
|RHEL et CentOS|6.10|2.6.32-754.15.3|
||7.2|3.10.0-327.79.2|
||7.3|3.10.0-514.66.2|
||7.4|3.10.0-693.50.3|
||7.5|3.10.0-862.34.2|
||7.6|3.10.0-957.21.3|
||7,7|3.10.0-1062.1.1|
||8.0|4.18.0-80.4.2|
||8.1|4.18.0-147|
||"7-RAW" (7.6)||
||"7-LVM" (7.6)|3.10.0-957.21.3|
||RHEL-SAP 7.4|TBD|
||RHEL-SAP 7.5|TBD|
|SLES|SLES11SP4 (y compris SAP)|3.0.101-108.95.2|
||SLES12SP1 pour SAP|3.12.74-60.64.115.1|
||SLES12SP2 pour SAP|4.4.121-92.114.1|
||SLES12SP3|4.4180-4.31.1 (noyau-Azure)|
||SLES12SP3 pour SAP|4.4.180-94.97.1|
||SLES12SP4|4.12.14-6.15.2 (noyau-Azure)|
||SLES12SP4 pour SAP|4.12.14-95.19.1|
||SLES15|4.12.14-5.30.1 (noyau-Azure)|
||SLES15 pour SAP|4.12.14-5.30.1 (noyau-Azure)|
||SLES15SP1|4.12.14-5.30.1 (noyau-Azure)|
|Oracle|6.10|UEK2 2.6.39-400.312.2<br>UEK3 3.8.13-118.35.2<br>RHCK 2.6.32-754.15.3 
||7.0-7.5|UEK3 3.8.13-118.35.2<br>UEK4 4.1.12-124.28.3<br>RHCK suit RHEL ci-dessus|
||7.6|RHCK 3.10.0-957.21.3<br>UEK5 4.14.35-1902.2.0|
|CoreOS Stable 2079.6.0|4.19.43*|
||Bêta 2135.3.1|4.19.50*|
||Alpha 2163.2.1|4.19.50*|
|Debian|jessie (sécurité)|3.16.68-2|
||jessie (rétroportage)|4.9.168-1+deb9u3|
||stretch (sécurité)|4.9.168-1+deb9u3|
||Debian GNU/Linux 10 (buster)|Debian 6.3.0-18+deb9u1|
||buster, sid (rétroportage stretch)|4.19.37-5|

## <a name="image-size-should-be-in-multiples-of-megabytes"></a>La taille de l’image doit être exprimée en multiples de mégaoctets.

La taille virtuelle de tous les disques durs virtuels exécutés sur Azure doit être alignée sur des multiples de 1 mégaoctet (Mo). Si votre disque dur virtuel n’est pas conforme à la taille virtuelle recommandée, votre demande peut être rejetée.

Suivez les instructions lors de la conversion d’un disque brut en disque dur virtuel. Assurez-vous que la taille du disque brut est un multiple de 1 Mo. Pour plus d'informations, consultez [Informations concernant les distributions non approuvées](../virtual-machines/linux/create-upload-generic.md).

## <a name="vm-access-denied"></a>Accès refusé à la machine virtuelle

Un problème d’_accès refusé_ lié à l’exécution d’un cas de test sur la machine virtuelle peut être dû à des privilèges insuffisants.

Vérifiez que vous avez activé l’accès approprié pour le compte sur lequel les cas d’auto-test sont exécutés. Activez l’accès pour exécuter des cas de test si ce n’est pas le cas. Si vous ne souhaitez pas activer l'accès, vous pouvez partager les résultats de l'auto-test avec l'équipe du support technique.

Pour envoyer votre demande avec une image désactivée SSH pour le processus de certification :

1. Exécutez l’[outil de test de certification le plus récent pour les machines virtuelles Azure](https://aka.ms/AzureCertificationTestTool) sur votre image.

2. Soumettez un [ticket de support](https://aka.ms/marketplacepublishersupport). Veillez à joindre le rapport du kit d’outils et fournissez les détails de l’offre :
   - Nom de l’offre
   - Nom du serveur de publication
   - ID/SKU du plan et version

3. Soumettez à nouveau votre demande de certification.

## <a name="download-failure"></a>Échec du téléchargement
    
Reportez-vous au tableau suivant pour tout problème survenant lorsque vous téléchargez l’image de machine virtuelle à l’aide d’une URL de signature d’accès partagé (SAP).

|Scénario|Error|Motif|Solution|
|---|---|---|---|
|1|Objet blob introuvable|Le disque dur virtuel a peut-être été supprimé ou déplacé de l'emplacement spécifié.|| 
|2|Objet blob en cours d'utilisation|Le disque dur virtuel est utilisé par un autre processus interne.|Le disque dur virtuel doit être dans un état Utilisé lorsque vous le téléchargez à l’aide d’une URL de signature d’accès partagé.|
|3|URL SAS non valide|L'URL de signature d'accès partagé associée au disque dur virtuel est incorrecte.|Récupérez l’URL SAS correcte.|
|4|Signature incorrecte|L'URL de signature d'accès partagé associée au disque dur virtuel est incorrecte.|Récupérez l’URL SAS correcte.|
|6|En-tête HTTP conditionnel|L'URL de signature d'accès partagé n'est pas valide.|Récupérez l’URL SAS correcte.|
|7|Nom de disque dur virtuel non valide|Recherchez la présence de caractères spéciaux, comme un signe de pourcentage `%` ou des guillemets `"` dans le nom du disque dur virtuel.|Renommez le fichier VHD en supprimant les caractères spéciaux.|

## <a name="first-mb-2048-kb-partition-linux-only"></a>Première partition en Mo (2048 ko) [Linux uniquement]

Lors de l’envoi du disque dur virtuel, vérifiez que les 2 048 premiers ko de celui-ci sont vides. Si ce n’est pas le cas, votre demande échouera.

>[!NOTE]
>Pour certaines images spéciales, telles que celles basées sur les images de base Windows Azure issues de Place de marché Azure, nous vérifions la présence d’une balise de facturation et ignorons la partition en Mo si la balise de facturation est présente et correspond à nos valeurs internes disponibles.

### <a name="create-a-first-mb-2048-kb-partition-on-an-empty-vhd"></a>Créer une première partition en Mo (2048 ko) sur un disque dur virtuel vide

Ces étapes s’appliquent uniquement à Linux.

1. Créez n’importe quel type de machine virtuelle Linux, par exemple Ubuntu, Cent OS ou autre. Renseignez les champs requis et sélectionnez **Suivant : Disques >** .

   ![Capture d’écran montrant la page Créer une machine virtuelle avec le bouton « Suivant : Commande des disques » en évidence.](./media/create-vm/vm-certification-issues-solutions-15.png)

1. Créez un disque non managé pour votre machine virtuelle.

   Utilisez les valeurs par défaut ou spécifiez une valeur pour les champs comme Carte réseau, NSG et IP publique.

   ![Image de capture d’écran de la page « Disques de données » dans le flux Créer une machine virtuelle.](./media/create-vm/vm-certification-issues-solutions-16.png)

1. Après avoir créé la machine virtuelle, sélectionnez **Disques** dans le volet gauche.

   ![Capture d’écran montrant comment sélectionner des disques pour une machine virtuelle.](./media/create-vm/vm-certification-issues-solutions-17.png)

1. Attachez votre disque dur virtuel à votre machine virtuelle en tant que disque de données pour créer la table de partition.

   1. Sélectionnez **Ajouter DataDisk** > **Blob existant**.

      ![Capture d’écran montrant comment ajouter un disque de données à votre disque dur virtuel.](./media/create-vm/vm-certification-issues-solutions-18.png)

   1. Recherchez votre compte de stockage VHD.
   1. Sélectionnez **Conteneur** puis sélectionnez votre disque dur virtuel.
   1. Sélectionnez **OK**.

      ![Capture d’écran de la page Attacher un disque non managé.](./media/create-vm/vm-certification-issues-solutions-19.png)

      Votre disque dur virtuel sera ajouté en tant que disque de données LUN 0.

   1. Redémarrez la machine virtuelle.

1. Après avoir redémarré la machine virtuelle, connectez-vous à celle-ci à l’aide de Putty ou d’un autre client et exécutez la commande `sudo  -i` pour obtenir un accès racine.

   ![Capture d’écran de la ligne de commande du client Putty montrant la commande sudo -i.](./media/create-vm/vm-certification-issues-solutions-20.png)

1. Créez une partition sur votre disque dur virtuel.

   1. Entrez la commande `fdisk /dev/sdb`.
   1. Pour afficher la liste des partitions existantes de votre disque dur virtuel, entrez `p`.
   1. Entrez `d` pour supprimer toutes les partitions existantes disponibles dans votre disque dur virtuel. Vous pouvez ignorer cette étape si elle n’est pas nécessaire.

      ![Capture d’écran de la ligne de commande du client Putty montrant les commandes permettant de supprimer les partitions existantes.](./media/create-vm/vm-certification-issues-solutions-21.png)

   1. Entrez `n` pour créer une nouvelle partition et sélectionnez `p` pour (partition principale).

   1. Entrez 2048 comme valeur de _premier secteur_. Vous pouvez conserver _dernier secteur_ comme valeur par défaut.

      >[!IMPORTANT]
      >Toutes les données existantes seront effacées jusqu’à atteindre 2048 ko. Sauvegardez le disque dur virtuel avant de créer une nouvelle partition.

      ![Capture d’écran de la ligne de commande du client Putty montrant les commandes et la sortie des données effacées.](./media/create-vm/vm-certification-issues-solutions-22.png)

   1. Entrez `w` pour confirmer la création de la partition. 

      ![Capture d’écran de la ligne de commande du client Putty montrant les commandes permettant de créer une partition.](./media/create-vm/vm-certification-issues-solutions-23.png)

   1. Vous pouvez vérifier la table de partition en exécutant la commande `n fdisk /dev/sdb` et en saisissant `p`. Vous verrez que la partition est créée avec une valeur de décalage de 2048. 

      ![Capture d’écran de la ligne de commande du client Putty montrant les commandes permettant de créer un décalage de 2048.](./media/create-vm/vm-certification-issues-solutions-24.png)

1. Détachez le disque dur virtuel de la machine virtuelle et supprimez la machine virtuelle.

### <a name="create-a-first-mb-2048-kb-partition-by-moving-existing-data-on-vhd"></a>Créer une première partition en Mo (2048 ko) en déplaçant les données existantes sur le disque dur virtuel

Ces étapes s’appliquent uniquement à Linux.

1. Créez n’importe quel type de machine virtuelle Linux, par exemple Ubuntu, Cent OS ou autre. Renseignez les champs requis et sélectionnez **Suivant : Disques >** .

   ![Capture d’écran montrant la page Créer une machine virtuelle avec le bouton « Suivant : Commande des disques » en évidence.](./media/create-vm/vm-certification-issues-solutions-15.png)

1. Créez un disque non managé pour votre machine virtuelle.

   ![Image de capture d’écran de la page « Disques de données » dans le flux Créer une machine virtuelle.](./media/create-vm/vm-certification-issues-solutions-16.png)

   Utilisez les valeurs par défaut ou spécifiez une valeur pour les champs comme Carte réseau, NSG et IP publique.

1. Après avoir créé la machine virtuelle, sélectionnez **Disques** dans le volet gauche.

   ![Capture d’écran montrant comment sélectionner des disques pour une machine virtuelle.](./media/create-vm/vm-certification-issues-solutions-17.png)

1. Attachez votre disque dur virtuel à votre machine virtuelle en tant que disque de données pour créer la table de partition.

   1. Attachez votre disque dur virtuel à votre machine virtuelle en tant que disque de données pour créer la table de partition.

   1. Sélectionnez **Ajouter DataDisk** > **Blob existant**.

      ![Capture d’écran montrant comment ajouter un disque de données à votre disque dur virtuel.](./media/create-vm/vm-certification-issues-solutions-18.png)

   1. Recherchez votre compte de stockage VHD.
   1. Sélectionnez **Conteneur** puis sélectionnez votre disque dur virtuel.
   1. Sélectionnez **OK**.

      ![Capture d’écran de la page Attacher un disque non managé.](./media/create-vm/vm-certification-issues-solutions-19.png)

      Votre disque dur virtuel sera ajouté en tant que disque de données LUN 0.

   1. Redémarrez la machine virtuelle.

1. Connectez-vous à la machine virtuelle à l’aide de Putty ou d’un autre client et exécutez la commande `sudo  -i` pour obtenir un accès racine.

   ![Capture d’écran de la ligne de commande du client Putty montrant la connexion et la commande sudo -i.](./media/create-vm/vm-certification-issues-solutions-20.png)

1. Exécutez la commande `echo '+1M,' | sfdisk --move-data /dev/sdc -N 1`.

   ![Capture d’écran de la ligne de commande du client Putty montrant l’exécution des commandes.](./media/create-vm/vm-certification-issues-solutions-25.png)

   >[!NOTE]
   >L’exécution de cette commande peut prendre un certain temps, car elle dépend de la taille du disque.

1. Détachez le disque dur virtuel de la machine virtuelle et supprimez la machine virtuelle.


## <a name="default-credentials"></a>Informations d’identification par défaut

N’envoyez jamais d’informations d’identification par défaut avec le disque dur virtuel soumis. L’ajout d’informations d’identification par défaut rend le VHD plus vulnérable aux menaces de sécurité. Au lieu de cela, créez vos propres informations d'identification lors de l'envoi du disque dur virtuel.
  
## <a name="datadisk-mapped-incorrectly"></a>DataDisk mappé de manière incorrecte

Un problème de mappage peut se produire lorsqu’une demande est envoyée avec plusieurs disques de données qui ne sont pas en séquence. Par exemple, l’ordre de numérotation pour trois disques de données doit être *0, 1, 2*. Tout autre ordre sera traité comme un problème de mappage.

Renvoyez la demande avec un séquencement approprié des disques de données.

## <a name="incorrect-os-mapping"></a>Mappage du système d’exploitation incorrect

Lorsqu'une image est créée, elle peut être mappée ou attribuée à une étiquette de système d'exploitation incorrecte. Par exemple, lorsque vous sélectionnez **Windows** comme partie du nom du système d'exploitation lors de la création de l'image, le disque du système d'exploitation doit uniquement être installé avec Windows. La même exigence s'applique à Linux.

## <a name="vm-not-generalized"></a>Machine virtuelle non généralisée

Si toutes les images issues de la Place de marché Azure doivent être réutilisées, le disque dur virtuel du système d'exploitation doit être généralisé.

* Pour **Linux**, le processus suivant généralise une machine virtuelle Linux et la redéploie sous la forme d'une machine virtuelle distincte.

  Dans la fenêtre SSH, entrez la commande suivante : `sudo waagent -deprovision+user`.

* Pour **Windows**, vous pouvez généraliser les images Windows à l'aide de `sysreptool`.

  Pour plus d’informations sur l’outil `sysreptool`, consultez [Présentation de l’outil de préparation système (Sysprep)](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

## <a name="datadisk-errors"></a>Erreurs DataDisk

Pour remédier aux erreurs liées au disque de données, utilisez le tableau suivant :

|Error|Motif|Solution|
|---|---|---|
|`DataDisk- InvalidUrl:`|Cette erreur peut survenir lorsqu’un numéro d’unité logique (LUN) non valide lors de l’envoi de l’offre.|Vérifiez que la séquence de numéros d'unités logiques du disque de données figure dans l'Espace partenaires.|
|`DataDisk- NotFound:`|Cette erreur peut survenir lorsqu’un disque de données ne se trouve pas à l’URL de signature d’accès partagé spécifiée.|Vérifiez que le disque de données se trouve à l’URL de signature d’accès partagé spécifiée.|

## <a name="remote-access-issue"></a>Problème d'accès à distance

Vous obtiendrez cette erreur si l’option RDP (Remote Desktop Protocol) n’est pas activée pour l’image Windows.

Activez l'accès RDP pour les images Windows avant de les envoyer.

## <a name="bash-history-failed"></a>Échec de l’historique Bash

Vous obtiendrez cette erreur si la taille de l’historique Bash de l’image que vous avez envoyée est supérieure à 1 kilooctet (ko). La taille est limitée à 1 ko pour empêcher le fichier de contenir des informations potentiellement sensibles.

Pour supprimer l’historique Bash :

1. Déployez la machine virtuelle et sélectionnez l’option **Run Command** sur le portail Azure.

   ![Capture d’écran du portail Azure avec l’option « Run Command » dans le volet gauche.](./media/create-vm/vm-certification-issues-solutions-3.png)

1. Sélectionnez la première option **RunShellScript**, puis exécutez la commande `cat /dev/null > ~/.bash_history && history -c`.

   ![Capture d’écran de la page « Script Run Command » sur le portail Azure.](./media/create-vm/vm-certification-issues-solutions-4.png)

1. Une fois la commande exécutée, redémarrez la machine virtuelle.

1. Généralisez la machine virtuelle, prenez l’image du disque dur virtuel et arrêtez la machine virtuelle.

1. Soumettez à nouveau l’image généralisée.

## <a name="request-an-exception-on-vm-images-for-select-tests"></a>Demander une exception sur les images de machine virtuelle pour certains tests

Les éditeurs peuvent demander des exceptions pour quelques tests effectués pendant la certification de la machine virtuelle. Des exceptions sont prévues dans de rares cas lorsqu’un éditeur fournit des preuves à l’appui de la demande. L’équipe de certification se réserve le droit de refuser ou d’approuver des exceptions à tout moment.

Cette section décrit les scénarios généraux dans lesquels les éditeurs demandent une exception et la façon d’en demander une.

### <a name="scenarios-for-exception"></a>Scénarios pour une exception

Les éditeurs demandent généralement des exceptions dans les cas suivants :

- **Exception pour un ou plusieurs cas de test**. Contactez le [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport) pour demander des exceptions pour des cas de test.

- **Machines virtuelles verrouillées/aucun accès racine**. Quelques éditeurs ont des scénarios dans lesquels les machines virtuelles doivent être verrouillées, car des logiciels tels que des pare-feu sont installés sur la machine virtuelle. Dans ce cas, téléchargez l’[outil de test certifié](https://aka.ms/AzureCertificationTestTool) et envoyez le rapport au [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport).

- **Modèles personnalisés**. Certains éditeurs publient des images de machine virtuelle qui nécessitent un modèle Azure Resource Manager personnalisé pour déployer les machines virtuelles. Dans ce cas, envoyez les modèles personnalisés au [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport) afin que l’équipe de certification puisse les utiliser à des fins de validation.

### <a name="information-to-provide-for-exception-scenarios"></a>Informations à fournir pour les scénarios d’exception

Contactez le [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport) pour demander une exception pour l’un des scénarios et incluez les informations suivantes :

- **ID de l’éditeur**. Saisissez votre ID d’éditeur sur le portail Espace partenaires.
- **ID/nom d’offre**. Entrez l’ID ou le nom de l’offre.
- **ID/SKU du plan**. Saisissez l’ID ou la référence SKU du plan de l’offre de machine virtuelle.
- **Version**. Entrez la version de l’offre de machine virtuelle qui requiert une exception.
- **Type d’exception**. Choisissez parmi les tests, les machines virtuelles verrouillées ou les modèles personnalisés.
- **Motif de la demande**. Incluez la raison de la demande d’exception, ainsi que toutes les informations sur les exemptions de test.
- **Chronologie**. Entrez la date de fin de l’exception.
- **Pièces jointes**. Documents de preuve importants joints :

  - Pour les machines virtuelles verrouillées, joignez le rapport de test.
  - Pour les modèles personnalisés, fournissez le modèle Resource Manager personnalisé en pièce jointe.

  Si vous ne les joignez pas, votre demande sera refusée.

## <a name="address-a-vulnerability-or-an-exploit-in-a-vm-offer"></a>Corriger une vulnérabilité ou un code malveillant exploitant une faille de sécurité dans une offre de machine virtuelle

Cette décrit comment fournir une nouvelle image de machine virtuelle quand une vulnérabilité ou un code malveillant exploitant une faille de sécurité est détecté dans l’une de vos images de machine virtuelle. Elle s’applique uniquement aux offres de machine virtuelle Azure publiées sur Place de marché Azure.

> [!NOTE]
> Vous ne pouvez pas supprimer la dernière image de machine virtuelle d’un plan ni arrêter la vente du dernier plan d’une offre.

Effectuez l’une des actions suivantes :

- Si vous disposez d’une nouvelle image de machine virtuelle pour remplacer l’image de machine virtuelle vulnérable, consultez [Fournir une image de machine virtuelle corrigée](#provide-a-fixed-vm-image).
- Si vous n’avez pas de nouvelle image de machine virtuelle pour remplacer la seule image de machine virtuelle dans un plan ou si vous avez terminé le plan, [arrêtez la vente du plan](partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan).
- Si vous n’envisagez pas de remplacer la seule image de machine virtuelle dans l’offre, nous vous recommandons d’[arrêter de vendre l’offre](partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan).

### <a name="provide-a-fixed-vm-image"></a>Fournir une image de machine virtuelle corrigée

Pour fournir une image de machine virtuelle corrigée afin de remplacer une image de machine virtuelle qui présente une vulnérabilité ou un code malveillant exploitant une faille de sécurité :

1. Fournissez une nouvelle image de machine virtuelle pour résoudre la vulnérabilité ou le code malveillant exploitant une faille de sécurité.
1. Supprimez l’image de machine virtuelle présentant la vulnérabilité de sécurité ou le code malveillant exploitant une faille de sécurité.
1. Republiez l’offre.

#### <a name="provide-a-new-vm-image-to-address-the-security-vulnerability-or-exploit"></a>Fournir une nouvelle image de machine virtuelle pour résoudre la vulnérabilité ou le code malveillant exploitant une faille de sécurité

Pour effectuer ces étapes, préparez les ressources techniques pour l’image de machine virtuelle que vous souhaitez ajouter. Pour plus d’informations, consultez [Créer une machine virtuelle à l’aide d’une base approuvée](azure-vm-create-using-approved-base.md) ou [Créer une machine virtuelle à l’aide de votre propre image](azure-vm-create-using-own-image.md) et [Générer un URI SAP pour votre image de machine virtuelle](azure-vm-get-sas-uri.md).

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
1. Dans le volet gauche, sélectionnez **Marketplace commercial** > **Vue d’ensemble**.
1. Dans la colonne **Alias de l’offre**, sélectionnez l’offre.
1. Sous l’onglet **Vue d’ensemble du plan**, dans la colonne **Nom**, sélectionnez le plan approprié.
1. Sous l’onglet **Configuration technique**, sous **Images de machine virtuelle**, sélectionnez **+ Ajouter une image de machine virtuelle**.

   > [!NOTE]
   > Vous ne pouvez ajouter à un plan qu’une seule image de machine virtuelle à la fois. Pour ajouter plusieurs images de machine virtuelle, publiez la première avant d’ajouter l’image de machine virtuelle suivante.

1. Dans les zones qui s’affichent, indiquez une nouvelle version du disque et l’image de la machine virtuelle.
1. Sélectionnez **Enregistrer le brouillon**.

Ensuite, supprimez l’image de machine virtuelle présentant la vulnérabilité de sécurité.

#### <a name="remove-the-vm-image-with-the-security-vulnerability-or-exploit"></a>Supprimer l’image de machine virtuelle présentant la vulnérabilité de sécurité ou le code malveillant exploitant une faille de sécurité

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le volet gauche, sélectionnez **Marketplace commercial** > **Vue d’ensemble**.
3. Dans la colonne **Alias de l’offre**, sélectionnez l’offre.
4. Sous l’onglet **Vue d’ensemble du plan**, dans la colonne **Nom**, sélectionnez le plan approprié.
5. Sous l’onglet **Configuration technique**, sous **Images de machine virtuelle**, à côté de l’image de machine virtuelle que vous souhaitez supprimer, sélectionnez **Supprimer une image de machine virtuelle**.
6. Dans la boîte de dialogue, sélectionnez **Continuer**.
7. Sélectionnez **Enregistrer le brouillon**.

Ensuite, republiez l’offre.

#### <a name="republish-the-offer"></a>Republier l’offre

1. Sélectionnez **Vérifier et publier**.
2. Si vous avez besoin de fournir des informations à l’équipe de certification, ajoutez-les dans la zone **Notes pour la certification**.
3. Sélectionnez **Publier**.

Pour terminer le processus de publication, consultez [Réviser et publier des offres](review-publish-offer.md).

## <a name="next-steps"></a>Étapes suivantes

- [Configurer les propriétés d’une offre de machine virtuelle](azure-vm-create-properties.md)
- [Activer les Récompenses de la place de marché](partner-center-portal/marketplace-rewards.md)
- Si vous avez des questions ou des suggestions d’amélioration, contactez le [support d’Espace partenaires](https://aka.ms/marketplacepublishersupport).