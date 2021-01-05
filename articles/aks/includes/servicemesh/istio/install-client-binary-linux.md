---
author: paulbouwer
ms.topic: include
ms.date: 10/02/2020
ms.author: pabouwer
ms.openlocfilehash: 164844a9da09563f8fbefe7ec60aff7eb05ace2d
ms.sourcegitcommit: 67e8e1caa8427c1d78f6426c70bf8339a8b4e01d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91666704"
---
## <a name="download-and-install-the-istio-istioctl-client-binary"></a>Télécharger et installer le binaire client istioctl d’Istio

Dans un interpréteur de commandes bash sur Linux ou sur [Sous-système Windows pour Linux][install-wsl], utilisez `curl` pour télécharger la version d’Istio, puis extrayez-la avec `tar`, de la manière suivante :

```bash
# Specify the Istio version that will be leveraged throughout these instructions
ISTIO_VERSION=1.7.3

curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istioctl-$ISTIO_VERSION-linux-amd64.tar.gz" | tar xz
```

Le binaire client `istioctl` s’exécute sur votre machine cliente et vous permet d’installer et de gérer Istio dans votre cluster AKS. Utilisez les commandes suivantes pour installer le binaire client Istio `istioctl` dans un interpréteur de commandes bash sur Linux ou le [sous-système Windows pour Linux][install-wsl]. Ces commandes copient le binaire client `istioctl` à l’emplacement du programme utilisateur standard dans votre `PATH`.

```bash
sudo mv ./istioctl /usr/local/bin/istioctl
sudo chmod +x /usr/local/bin/istioctl
```

Si vous souhaitez utiliser la complétion de ligne de commande pour le binaire client `istioctl` Istio, configurez-la comme suit :

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && istioctl collateral --bash -o ~/completions
source ~/completions/istioctl.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/istioctl.bash" >> ~/.bashrc
```

<!-- LINKS - external -->
[install-wsl]: /windows/wsl/install-win10
