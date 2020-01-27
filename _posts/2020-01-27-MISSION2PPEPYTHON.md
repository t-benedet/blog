---
layout: post
title: Automatisation des sauvegardes via un script python
---

Thomas a proposé l'idée de créer un script permettant d'automatiser la sauvegarde des configurations des switchs et routers. Voici la première version du script que j'ai réalisé en __python__, grâce à la libraire __netmiko__ :

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from netmiko import ConnectHandler

# Configuration :
tftp_server = '172.25.19.210'
tftp_folder1 = 'TEST'
tftp_folder2 = 'SAVE'
filename = 'test1.bin'

# Switch connection configuration :
rubis = {
    'device_type': 'cisco_ios',
    'ip': 'xxx.xxx.xxx.xxx',
    'password': 'xxxxxxxx',
    'secret': 'xxxxxxxx',
  # 'global_delay_factor': 2,
}

net_connect = ConnectHandler(**rubis)
net_connect.enable()

cmd = 'copy start tftp://'+tftp_server+'/'+tftp_folder1+'/'+tftp_folder2+'/'+filename

output = net_connect.send_command(
    cmd,
    expect_string=r'Address or name of remote host [172.25.19.210]?'
)

output += net_connect.send_command('\n', expect_string=r'Destination filename [TEST/SAVE/test1.bin]?')
output += net_connect.send_command('\n', expect_string=r'#')
print(output)

net_connect.exit_enable_mode()
```

C'est une permière version qui effectue la sauvegarde de la configuration en cours sur le switch  Rubis et qui l'envoie automatiquement vers le servers TFTP. 

Une version plus évoluée sera mise en place par la suite, aussi bien dans le code ( création de fonctions pour chaques switchs et routers qui permettra d'appliquer ce script à toute l'infrastructure ), mais aussi dans la forme puisque l'objectif est de faire une interface web permettant d'executer ce script sur le matériel voulu. 
