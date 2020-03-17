# Exercicis de túnels SSH

# Túnel directe
ssh -i new_key.pem -L 50000:localhost:13 fedora@54.235.238.163

50000 --> Port origen al host local, a quin port està el túnel
localhost:13 --> On apunta el túnel, al localhost del host destí
fedora@IP --> EL host destí
