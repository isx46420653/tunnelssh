# Teoria de Túnels SSH

### Conceptes bàsics

+ Host local: Host des d'on treballes i on crees el túnel. Per exemple, l'ordinador de casa quan  fas SSH a la AMI de Amazon.

+ Host destí: El host destí es on fas l'SSH, en aquest cas a la IP pública de l'AMI d'Amazon.

+ Host remot: És el host que està dins de la xarxa privada del host destí, no és directament accessible des d'una xarxa externa, per accedir-hi necessitem crear el túnel al host destí.
