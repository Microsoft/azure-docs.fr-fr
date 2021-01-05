---
title: Résoudre les problèmes de récupération d’urgence de connectivité pour Azure avec Azure Site Recovery
description: Résoudre les problèmes de connectivité dans le cadre de la reprise d’activité d’une machine virtuelle Azure
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 04/06/2020
ms.openlocfilehash: 24ffce1528aa5c82fec9666fa0cb7b8717107f54
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652260"
---
# <a name="troubleshoot-azure-to-azure-vm-network-connectivity-issues"></a>Résoudre les problèmes de connectivité du réseau de machines virtuelles Azure vers Azure

Cet article décrit les problèmes courants liés à la connectivité réseau lorsque vous répliquez et récupérez des machines virtuelles Azure d’une région à une autre. Pour plus d’informations sur la configuration requise pour la mise en réseau, consultez la page [Configuration de connectivité pour la réplication des machines virtuelles Azure](azure-to-azure-about-networking.md).

Pour que la réplication Site Recovery fonctionne, une connectivité sortante vers des URL ou des plages d’adresses IP spécifiques est nécessaire à partir de la machine virtuelle. Si votre machine virtuelle se trouve derrière un pare-feu ou utilise des règles de groupe de sécurité réseau pour contrôler la connectivité sortante, vous pouvez rencontrer l’un des problèmes ci-après.

| **Nom**                  | **Commercial**                               | **Secteur public**                                 | **Description** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Stockage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net` | Nécessaire pour que les données puissent être écrites dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. Si vous connaissez tous les comptes de stockage de cache pour vos machines virtuelles, vous pouvez utiliser une liste verte pour les URL de compte de stockage spécifiques. Par exemple, `cache1.blob.core.windows.net` et `cache2.blob.core.windows.net` au lieu de `*.blob.core.windows.net`. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Nécessaire pour l’autorisation et l’authentification aux URL du service Site Recovery. |
| Réplication               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Nécessaire pour que la communication du service Site Recovery puisse avoir lieu à partir de la machine virtuelle. Vous pouvez utiliser l’_Adresse IP de Site Recovery_ correspondante si votre proxy de pare-feu prend en charge les adresses IP. |
| Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Nécessaire pour que les données de surveillance et de diagnostic Site Recovery puissent être écrites à partir de la machine virtuelle. Vous pouvez utiliser l’_Adresse IP de supervision de Site Recovery_ correspondante si votre proxy de pare-feu prend en charge les adresses IP. |

## <a name="outbound-connectivity-for-site-recovery-urls-or-ip-ranges-error-code-151037-or-151072"></a>Connectivité sortante pour les plages d’adresses IP ou les URL Site Recovery (code d’erreur 151037 ou 151072)

### <a name="issue-1-failed-to-register-azure-virtual-machine-with-site-recovery-151195"></a>Problème 1 : Échec de l’inscription de la machine virtuelle Azure auprès de Site Recovery (151195)

#### <a name="possible-cause"></a>Cause probable

Il est impossible d’établir une connexion aux points de terminaison Site Recovery en raison d’un échec de la résolution DNS. Ce problème survient le plus souvent pendant la reprotection, quand vous avez procédé au basculement de la machine virtuelle mais que le serveur DNS n’est pas accessible à partir de la région de reprise d’activité après sinistre.

#### <a name="resolution"></a>Résolution

Si vous utilisez DNS personnalisé, assurez-vous que le serveur DNS est accessible à partir de la région de récupération d’urgence.

Pour vérifier si la machine virtuelle utilise un paramètre DNS personnalisé :

1. Ouvrez **Machines virtuelles** et sélectionnez la machine virtuelle.
1. Accédez aux **Paramètres** de la machine virtuelle et sélectionnez **Réseau**.
1. Dans **Réseau/sous-réseau virtuel**, sélectionnez le lien pour ouvrir la page des ressources du réseau virtuel.
1. Accédez à **Paramètres** et sélectionnez **Serveurs DNS**.

Essayez d’accéder au serveur DNS à partir de la machine virtuelle. Si le serveur DNS n’est pas accessible, rendez-le accessible en faisant basculer le serveur DNS ou en créant la ligne de site entre le réseau de récupération d’urgence et le DNS.

  :::image type="content" source="./media/azure-to-azure-troubleshoot-errors/custom_dns.png" alt-text="com-error":::

### <a name="issue-2-site-recovery-configuration-failed-151196"></a>Problème2 : Échec de la configuration de Site Recovery (151196)

> [!NOTE]
> Si les machines virtuelles se trouvent derrière un équilibreur de charge interne **standard**, par défaut, elles n'ont pas accès aux adresses IP Microsoft 365, comme `login.microsoftonline.com`. Remplacez le type d’équilibreur de charge interne par le type **De base** ou créez un accès sortant comme décrit dans l’article [Configurer des règles d’équilibrage de charge et des règles de trafic sortant dans Standard Load Balancer à l’aide d’Azure CLI](../load-balancer/quickstart-load-balancer-standard-public-cli.md?tabs=option-1-create-load-balancer-standard#create-outbound-rule-configuration).

#### <a name="possible-cause"></a>Cause probable

Impossible d'établir une connexion avec les points de terminaison IP4 d'identité et d'authentification Microsoft 365.

#### <a name="resolution"></a>Résolution

- Azure Site Recovery exige l'accès aux plages d'adresses IP Microsoft 365 pour l'authentification.
- Si vous utilisez un proxy de règles/pare-feu de groupe de sécurité réseau Azure pour contrôler la connectivité réseau sortante sur la machine virtuelle, veillez à autoriser la communication avec les plages IP Microsoft 365. Créer une règle de groupe de sécurité réseau [basée sur une balise de service Azure Active Directory (Azure AD)](../virtual-network/network-security-groups-overview.md#service-tags) pour autoriser l’accès à toutes les adresses IP correspondant à Azure AD.
- Si de nouvelles adresses sont ajoutées à Azure AD à l’avenir, vous devez créer des règles de groupe de sécurité réseau.

### <a name="example-nsg-configuration"></a>Exemple de configuration de groupe de sécurité réseau

Cet exemple montre comment configurer des règles de groupes de sécurité réseau pour une machine virtuelle à répliquer.

- Si vous utilisez des règles de groupe de sécurité réseau pour contrôler la connectivité sortante, utilisez des règles **Autoriser le trafic HTTPS sortant** sur port:443 pour toutes les plages d’adresses IP requises.
- L’exemple part du principe que **USA Est** est l’emplacement source de la machine virtuelle, et que **USA Centre** en est l’emplacement cible.

#### <a name="nsg-rules---east-us"></a>Règles de groupe de sécurité réseau - USA Est

1. Créez une règle de sécurité sortante HTTPS pour le groupe de sécurité réseau, comme illustré dans la capture d’écran suivante. Cet exemple utilise la **balise de service de destination** : _Storage.EastUS_ et les **Plages de ports de destination** : _443_.

     :::image type="content" source="./media/azure-to-azure-about-networking/storage-tag.png" alt-text="Capture d'écran représentant le volet Ajouter une règle de sécurité de trafic sortant d'une règle de sécurité pour le point de stockage USA Est.":::

1. Créez une règle de sécurité sortante HTTPS pour le groupe de sécurité réseau, comme illustré dans la capture d’écran suivante. Cet exemple utilise la **balise de service de destination** : _AzureActiveDirectory_ et les **Plages de ports de destination** : _443_.

     :::image type="content" source="./media/azure-to-azure-about-networking/aad-tag.png" alt-text="Capture d'écran représentant le volet Ajouter une règle de sécurité de trafic sortant d'une règle de sécurité pour Azure Active Directory.":::

1. Comme pour les règles de sécurité ci-dessus, créez une règle de sécurité HTTPS sortante (443) pour « EventHub.CentralUS » sur le groupe de sécurité réseau qui correspond à la position cible. Celle-ci permet d’accéder à la supervision de Site Recovery.
1. Créez une règle de sécurité HTTPS sortante (443) pour « AzureSiteRecovery » sur le groupe de sécurité réseau. Celle-ci permet d’accéder au service Site Recovery dans n’importe quelle région.

#### <a name="nsg-rules---central-us"></a>Règles de groupe de sécurité réseau - USA Centre

Pour cet exemple, ces règles de groupe de sécurité réseau sont nécessaires pour que la réplication puisse être activée de la région cible vers la région source après le basculement :

1. Créez une règle de sécurité sortante HTTPS pour _Storage.CentralUS_ :

   - **Balise d’identification de destination** : _Storage.CentralUS_
   - **Plages de ports de destination** : _443_

1. Créez une règle de sécurité sortante HTTPS pour _AzureActiveDirectory_.

   - **Balise d’identification de destination** : _AzureActiveDirectory_
   - **Plages de ports de destination** : _443_

1. Comme pour les règles de sécurité ci-dessus, créez une règle de sécurité HTTPS sortante (443) pour « EventHub.Eastus » sur le groupe de sécurité réseau qui correspond à la position source. Celle-ci permet d’accéder à la supervision de Site Recovery.
1. Créez une règle de sécurité HTTPS sortante (443) pour « AzureSiteRecovery » sur le groupe de sécurité réseau. Celle-ci permet d’accéder au service Site Recovery dans n’importe quelle région.

### <a name="issue-3-site-recovery-configuration-failed-151197"></a>Problème 3 : Échec de la configuration de Site Recovery (151197)

#### <a name="possible-cause"></a>Cause probable

Il est impossible d’établir une connexion aux points de terminaison de service Azure Site Recovery.

#### <a name="resolution"></a>Résolution

Si vous utilisez un proxy de règles/pare-feu de groupe de sécurité réseau Azure pour contrôler la connectivité réseau sortante sur la machine, vous devez autoriser plusieurs balises de service. [Plus d’informations](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags)

### <a name="issue-4-azure-to-azure-replication-failed-when-the-network-traffic-goes-through-on-premises-proxy-server-151072"></a>Problème 4 : Échec de la réplication Azure vers Azure lorsque le trafic réseau transite par un serveur proxy local (151072)

#### <a name="possible-cause"></a>Cause probable

Les paramètres de proxy personnalisés sont incorrects, et l’agent Mobility Service d’Azure Site Recovery n’a pas détecté automatiquement les paramètres de proxy d’Internet Explorer (IE).

#### <a name="resolution"></a>Résolution

1. L’agent du service Mobilité détecte les paramètres de proxy à partir d’Internet Explorer sur Windows et à l’emplacement `/etc/environment` sur Linux.
1. Si vous préférez définir le proxy uniquement pour Azure Site Recovery Mobility Service, vous pouvez fournir les détails du proxy dans _ProxyInfo.conf_ à l’emplacement suivant :

   - **Linux** : `/usr/local/InMage/config/`
   - **Windows** : `C:\ProgramData\Microsoft Azure Site Recovery\Config`

1. Le fichier _ProxyInfo.conf_ doit inclure les paramètres de proxy au format _INI_ suivant :

   ```plaintext
   [proxy]
   Address=http://1.2.3.4
   Port=567
   ```

> [!NOTE]
> L’agent du Mobility Service Azure Site Recovery prend uniquement en charge les **proxys non authentifiés**.

### <a name="fix-the-problem"></a>Résoudre le problème

Pour autoriser les [URL requises](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) ou les [plages d’adresses IP requises](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags), suivez les étapes fournies dans ce [document d’aide à la mise en réseau](./azure-to-azure-about-networking.md).

## <a name="next-steps"></a>Étapes suivantes

[Répliquer des machines virtuelles Azure dans une autre région Azure](azure-to-azure-how-to-enable-replication.md)