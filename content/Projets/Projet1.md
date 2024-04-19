---
title: "Conversion machines Hyper-V vers ESXi"
date: 2024-04-18T19:53:33+05:30
draft: false
author: "William"
tags:
  - Hyper-V
  - ESXi
  - QEMU-IMG
image: /images/qemu.jpg
description: ""
toc: true
mathjax: true
---

## Conversion VHDX vers VMDK

Il est possible de convertir des machines virtuelles au format VHDX, donc HYPER-V, au format VMDK, accepté pour les systèmes ESXi.

Pour cela, le logiciel QEMU-IMG est très utile, car en une seule ligne de commande, il se charge de convertir le fichier.

Il suffit en premier lieu d'exporter la machine présente sur l'HYPER-V, de préférence dans le dossier où se trouve le logiciel QEMU-IMG. On déplace ensuite le fichier VHDX à la racine du dossier contenant le logiciel, ouvrir une invite de commande, et de taper la commande suivante: 

```
.\qemu-img.exe convert NOMVM.vhdx -O vmdk NOMSOUHAITÉ.vmdk -p
Attention, c'est -O (la lettre en majuscule) et non -0 (le chiffre)
```

Il faut ensuite se connecter à l'interface WEB de l'ESXi, de préférence depuis l'Hyper-V pour plus de facilité, et de télécharger dans la banque de données le fichier VMDK converti, de préférence dans un dossier dédié (VMDK par exemple). (Une fois le fichier transféré sur l'ESXi, vous pouvez sans problème supprimer le fichier VHDX et VMDK présent à la racine du dossier logiciel.

Pour permettre à l'ESXi d'utiliser le disque virtuel, il faut passer par une étape de provisionnement du disque (thin en anglais). Pour cela, veillez à ce que le service SSH soit activé(onglet Hôte/actions/Services/activer Secure Shell (SSH)) et lancez le logiciel PuTTY ou tout autre logiciel semblable afin d'accéder à la banque de données, présente dans le dossier vmfs/volumes. La banque est présente sous le nom "datastoreX" (le "X" est replacé par un chiffre, dépendant du nombre de datastore que vous possédez.

Une fois dans ce datastore puis dans le dossier dédié que vous aurez créé, la commande suivante sera utilisée pour le provisionnement du disque virtuel.

```
vmkfstools -i NOMVM.vmdk -d thin NOMSOUHAITÉ-thin.vmdk
```

Une fois la procédure terminée, il ne reste plus qu'à créer une nouvelle machine virtuelle qui utilisera le disque provisionné pour retrouver toutes les données de la machine! (pensez à changer l'IP de la machine si elle était en statique).



Dépannage:

Si le provisionnement de la machine n'avance plus:
- Appuyez simplement sur la touche "Entrée" et répétez l'opération à chaque fois que vous sentez un arrêt de la procédure.

Si la machine ne démarre pas avec le disque virtuel:
- Vérifiez qu'aucun autre disque n'existe dans les options de la machine virtuelle
- Passez la machine virtuelle en mode BIOS (Modifier les paramètres/Options VM/Options de démarrage/BIOS)
