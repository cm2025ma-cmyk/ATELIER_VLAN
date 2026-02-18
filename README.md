# 🧩 TP VLAN & Masques — Comprendre la segmentation réseau

> Atelier pratique pour comprendre le rôle des VLAN et des masques IP dans la segmentation d’un réseau.

---

# 🎯 Objectifs pédagogiques

À la fin de ce TP, vous devrez savoir :

✅ Expliquer le rôle d’un VLAN  
✅ Comprendre le lien VLAN ↔ sous-réseau IP  
✅ Calculer et utiliser des masques IP  
✅ Mettre en place un trunk 802.1Q  
✅ Tester la communication intra/inter-VLAN

---

# 🧠 Rappel théorique simple

## 📌 VLAN
Un VLAN est un **réseau logique** sur un même switch physique.

👉 Chaque VLAN = un domaine de broadcast distinct  
👉 Chaque VLAN = un sous-réseau IP différent

---

## 📌 Masque de sous-réseau
Le masque définit :
- La partie réseau  
- La partie hôte  

| Masque | Nb hôtes |
|------|---------|
   /24 | 254 |
   /25 | 126 |
   /26 | 62 |

---

# 🗺️ Topologie du TP

```
         VLAN 10         VLAN 20
        192.168.10.0/24 192.168.20.0/24

PC1 -------- SW1 -------- R1 -------- PC3
              |  (trunk)
              |
             PC2
```
<img width="627" height="346" alt="image" src="https://github.com/user-attachments/assets/f2e556da-c573-4558-9d95-eabe1c5134ae" />

---

# 🧩 Matériel Packet Tracer

- 1 routeur (2911)  
- 1 switch (2960)  
- 3 PC  

---

# 🌐 Plan d’adressage

## VLAN 10

| Équipement | IP |
|----------|------|
PC1 | 192.168.10.10/24 |
PC2 | 192.168.10.20/24 |
GW | 192.168.10.1 |

---

## VLAN 20

| Équipement | IP |
|----------|------|
PC3 | 192.168.20.10/24 |
GW | 192.168.20.1 |

---

# 🔹 PARTIE 1 — Création des VLAN

Sur le switch :

```
enable
conf t
vlan 10
name ADMIN
vlan 20
name USERS
```
<img width="614" height="188" alt="image" src="https://github.com/user-attachments/assets/60a3cc17-cb46-4a45-926e-8fbd321bb176" />

---

# 🔹 PARTIE 2 — Affectation des ports

```
interface f0/1
switchport mode access
switchport access vlan 10

interface f0/2
switchport mode access
switchport access vlan 10

interface f0/3
switchport mode access
switchport access vlan 20
```
<img width="614" height="188" alt="image" src="https://github.com/user-attachments/assets/60a3cc17-cb46-4a45-926e-8fbd321bb176" />


---

# 🔹 PARTIE 3 — Trunk vers le routeur

```
interface f0/24
switchport mode trunk
```
<img width="503" height="166" alt="image" src="https://github.com/user-attachments/assets/893db40a-c080-4538-a2de-ae9041d7c4d2" />

---

# 🔹 PARTIE 4 — Router-on-a-stick

Sur R1 :

```
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface g0/0
no shutdown
```
<img width="597" height="59" alt="image" src="https://github.com/user-attachments/assets/c0d21437-78b2-4ae4-8bfd-176c6a8ab711" />

---

# 🔹 PARTIE 5 — Configuration IP des PC

Configurer IP + passerelle selon le plan d’adressage.

---

# 🧪 TESTS

## Test 1 — Intra-VLAN
PC1 → PC2  
👉 Doit fonctionner

* * Copie d'écran ici * *  
<img width="815" height="252" alt="image" src="https://github.com/user-attachments/assets/c7314ffa-0bbf-4568-bad9-5cce51015016" />

---

## Test 2 — Inter-VLAN
PC1 → PC3  
👉 Fonctionne uniquement grâce au routeur

* * Copie d'écran ici * *  
  <img width="508" height="577" alt="image" src="https://github.com/user-attachments/assets/9be5b95b-4fbf-4d7d-bf4a-509718cade64" />

---

# ❓ Questions de réflexion

1. Pourquoi PC1 ne voit-il pas PC3 sans routeur ? -> Car ils sont sur deux réseaux différent et sur 2 vlan différent dans ce cas il necessite un niveau 3 pour router les paquets.
2. Quel rôle joue le masque /24 ? -> Le masque défini la partie hôte et réseau ici 255.255.255.0 Donc les 3 premiers octets défini le réseau et le dernier octet défini la partie machine
3. Que se passe-t-il si VLAN 10 et VLAN 20 ont le même réseau IP ? -> Les réseaux vont s'entre choquer et entrainera des défailliances.
4. Pourquoi un trunk est-il nécessaire ? -> Le trunk est necessaire car l'on veut que plusieurs VLAN puisse passer par le même port/câble. (évite d'utiliser 4096 câble pour chaque vlan sachant que les routeurs ont souvant moins de ports que les switchs).

---

# ⭐ Travail sur les Masques

Changer VLAN 10 en :

```
192.168.10.0/25
```
<img width="449" height="117" alt="image" src="https://github.com/user-attachments/assets/7ada07c4-9541-480e-9f26-406ee465a905" />

Questions :
- Combien d’hôtes max ?  126 hôtes possible car le dernier octet 2^7 = 128-2 (une adresse réseau et une broadcast)
- Quelle plage IP valide ?  192.168.10.1-127
- Peut-on encore communiquer avec VLAN 20 ?  Oui car le routeur fais le liens avec le VLAN 20 via le GW configurer sur chaque ordinateur qui indique l'adresse du routeur.

---

# 🚀 Extensions

- Ajouter VLAN 30
Sur le routeur
```
interface g0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
```
<img width="590" height="128" alt="image" src="https://github.com/user-attachments/assets/6f5a67bc-fc55-4c52-ba88-812ff664e525" />


Sur le switch

```
interface f0/4
switchport mode access
switchport access vlan 30
```

<img width="573" height="147" alt="image" src="https://github.com/user-attachments/assets/454df1c7-cd51-42c4-90cc-7193dd293133" />

- Mettre un DHCP par VLAN
```
 en
 conf t
 ip dhcp pool vlan10
 network 192.168.10.0 255.255.255.128
default-router 192.168.10.1
dns-server 8.8.8.8
exit
ip dhcp pool vlan20
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8
exit
ip dhcp pool vlan30
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
dns-server 8.8.8.8
exit
```
 
<img width="589" height="487" alt="image" src="https://github.com/user-attachments/assets/ac73edad-4dc1-421b-b4ab-dfa2b1cb6b98" />
<img width="1826" height="630" alt="image" src="https://github.com/user-attachments/assets/5364d44f-0443-4481-b94f-869f602eb9b6" />

CE N'ETAIT PAS SPECIFIE MAIS VOICI UN PC4 DANS LE VLAN30
<img width="1221" height="780" alt="image" src="https://github.com/user-attachments/assets/29a6a5ed-048c-4388-9b6d-bba4046332a0" />

---

# 📝 Évaluation (/20)

| Critère | Points |
|--------|-------|
VLAN créés correctement | 4 |
Ports bien affectés | 2 |
Trunk opérationnel | 4 |
Inter-VLAN fonctionnel | 4 |
Travail sur les masques | 4 |  
Extention | 2 |  
  
# ✅ Fin du TP

Si vous savez expliquer :
> "Pourquoi deux VLAN ne communiquent pas sans routeur ?"
Les VLANs ne peut communiquer sans routeur car ils sont sur 2 réseaux différent et qu'il y a une séparation logique via les tags. Pour communiquer sans routeur il faudrait un switch de niveau 3 avec un routage inter vlan.
Alors vous avez compris la segmentation réseau 👍
