# Guia de Implementação — Projeto RC2 (2025/2026)

> **Escola Superior de Tecnologia e Gestão — Licenciatura em Engenharia Informática**
> Disciplina: Redes de Computadores 2 | Docentes: Armando Ventura e Pedro Moreira
> Alunos: 21198 / 24105 | Entrega: 11 de Junho de 2026 | Discussão: 12 de Junho de 2026

---

## Índice

1. [Exercício 1 — VLSM](#exercício-1--vlsm)
2. [Exercício 2 — Topologia GNS3](#exercício-2--topologia-gns3)
3. [Exercício 3 — Interfaces Loopback](#exercício-3--interfaces-loopback)
4. [Exercício 4 — OSPF](#exercício-4--ospf)
5. [Exercício 5 — Acesso à Internet (VM-W/L)](#exercício-5--acesso-à-internet-vm-wl)
6. [Exercício 6 — Firewall entre PCs clientes](#exercício-6--firewall-entre-pcs-clientes)
7. [Exercício 7 — Telnet apenas PC1 para R2](#exercício-7--telnet-apenas-pc1-para-r2)
8. [Exercício 8 — Port Knocking no R1](#exercício-8--port-knocking-no-r1)
9. [Exercício 9 — Restrição de acesso da VM-W/L](#exercício-9--restrição-de-acesso-da-vm-wl)
10. [Exercício 10 — Restrição de acesso do PC2](#exercício-10--restrição-de-acesso-do-pc2)
11. [Exercício 11 — Bloqueio de tráfego entre VLANs no R3](#exercício-11--bloqueio-de-tráfego-entre-vlans-no-r3)
12. [Exercício 12 — API MikroTik](#exercício-12--api-mikrotik)

---

## Exercício 1 — VLSM

**Cotação: até 2 valores**

### Cálculo de F

- Menor número de aluno: **21198**
- F = 21198 % 256 = **206**
- Rede base: **10.206.0.0/16**

### Tabela de Necessidades e Prefixos

| Subrede   | Necessidade | 2^n necessário | Prefixo | Hosts Úteis |
|-----------|-------------|----------------|---------|-------------|
| VLAN 30   | 1023        | 2^11 = 2048    | /21     | 2046        |
| SUBREDE D | 500         | 2^9 = 512      | /23     | 510         |
| VLAN 20   | 200         | 2^8 = 256      | /24     | 254         |
| SUBREDE A | 20          | 2^5 = 32       | /27     | 30          |
| VLAN 10   | 6           | 2^3 = 8        | /29     | 6           |
| SUBREDE B | 2           | 2^2 = 4        | /30     | 2           |
| SUBREDE C | 2           | 2^2 = 4        | /30     | 2           |

### Atribuição VLSM (ordenado do maior para o menor)

| Subrede   | Endereço de Rede | Máscara         | Prefixo | 1º Host       | Último Host    | Broadcast      | Hosts Úteis |
|-----------|------------------|-----------------|---------|---------------|----------------|----------------|-------------|
| VLAN 30   | 10.206.0.0       | 255.255.248.0   | /21     | 10.206.0.1    | 10.206.7.254   | 10.206.7.255   | 2046 ✅     |
| SUBREDE D | 10.206.8.0       | 255.255.254.0   | /23     | 10.206.8.1    | 10.206.9.254   | 10.206.9.255   | 510 ✅      |
| VLAN 20   | 10.206.10.0      | 255.255.255.0   | /24     | 10.206.10.1   | 10.206.10.254  | 10.206.10.255  | 254 ✅      |
| SUBREDE A | 10.206.11.0      | 255.255.255.224 | /27     | 10.206.11.1   | 10.206.11.30   | 10.206.11.31   | 30 ✅       |
| VLAN 10   | 10.206.11.32     | 255.255.255.248 | /29     | 10.206.11.33  | 10.206.11.38   | 10.206.11.39   | 6 ✅        |
| SUBREDE B | 10.206.11.40     | 255.255.255.252 | /30     | 10.206.11.41  | 10.206.11.42   | 10.206.11.43   | 2 ✅        |
| SUBREDE C | 10.206.11.44     | 255.255.255.252 | /30     | 10.206.11.45  | 10.206.11.46   | 10.206.11.47   | 2 ✅        |

> Todas as subredes estão dentro da rede 10.206.0.0/16 sem sobreposição.

---

## Exercício 2 — Topologia GNS3

**Cotação: até 1 valor**

### Dispositivos necessários

| Dispositivo | Tipo no GNS3               | Como obter                                      |
|-------------|----------------------------|-------------------------------------------------|
| Router1     | MikroTik RouterOS x86      | https://mikrotik.com/download → Cloud Hosted Router (.img) |
| Router2     | Cisco C2691                | IOS image: `c2691-adventerprisek9-mz.124-xx.bin` (pedir ao docente) |
| Router3     | Cisco C2691                | Mesmo IOS do Router2                            |
| Switch1     | Ethernet Switch (nativo)   | Já incluído no GNS3                             |
| Switch2     | EtherSwitch Router C2691   | Mesmo IOS do C2691 + módulo NM-16ESW            |
| PC1–PC4     | VPCS (nativo)              | Já incluído no GNS3                             |
| VM-W/L      | VirtualBox VM              | Máquina virtual Windows ou Linux com modo gráfico |

### Módulos a adicionar nos Cisco C2691

| Dispositivo | Slot | Módulo  | Finalidade                        |
|-------------|------|---------|-----------------------------------|
| Router2     | WIC  | WIC-2T  | Porta serial para SUBREDE C       |
| Router3     | WIC  | WIC-2T  | Porta serial para SUBREDE C       |
| Switch2     | NM   | NM-16ESW| Portas de switch para VLANs       |

> Para adicionar módulos: botão direito no dispositivo → **Configure** → **Slots**

### Como adicionar o MikroTik no GNS3

1. Descarrega o **Raw disk image (.img)** em https://mikrotik.com/download
2. GNS3 → **Edit → Preferences → QEMU VMs → New**
3. Nome: `MikroTik RouterOS`
4. Disk image: seleciona o ficheiro `.img`
5. RAM: **256 MB**
6. Adiciona pelo menos **4 interfaces de rede**

### Como adicionar o Cisco C2691 no GNS3

1. GNS3 → **Edit → Preferences → Dynamips → IOS Routers → New**
2. Seleciona o ficheiro `.bin` do IOS
3. Deixa o GNS3 calcular o **Idle-PC** (evita sobrecarga de CPU)
4. Em **Slots**, adiciona os módulos necessários (WIC-2T / NM-16ESW)

### Ligações da Topologia

| Ligação       | Dispositivo A | Interface A | Dispositivo B | Interface B | Tipo de cabo |
|---------------|---------------|-------------|---------------|-------------|--------------|
| WAN           | Router1       | ether1      | Cloud/NAT     | —           | UTP          |
| SUBREDE A     | Router1       | ether2      | PC1           | e0          | UTP          |
| SUBREDE B     | Router1       | ether3      | Router2       | f0/0        | UTP          |
| SUBREDE C     | Router2       | s0/0        | Router3       | s0/0        | **Serial**   |
| SUBREDE D     | Router2       | f0/1        | Switch1       | —           | UTP          |
| Switch1–PC2   | Switch1       | —           | PC2           | e0          | UTP          |
| Switch1–SW2   | Switch1       | —           | Switch2       | f0/0        | UTP          |
| VLAN 10       | Switch2       | —           | VM-W/L        | —           | UTP          |
| VLAN 20       | Switch2       | —           | PC4           | e0          | UTP          |
| VLAN 30       | Switch2       | —           | PC3           | e0          | UTP          |

### Endereços IP das interfaces (baseado no VLSM)

| Dispositivo | Interface  | Endereço IP      | Subrede        |
|-------------|------------|------------------|----------------|
| Router1     | ether1     | DHCP (WAN)       | Internet       |
| Router1     | ether2     | 10.206.11.1/27   | SUBREDE A      |
| Router1     | ether3     | 10.206.11.41/30  | SUBREDE B      |
| Router2     | f0/0       | 10.206.11.42/30  | SUBREDE B      |
| Router2     | s0/0       | 10.206.11.45/30  | SUBREDE C      |
| Router2     | f0/1       | 10.206.8.1/23    | SUBREDE D      |
| Router3     | s0/0       | 10.206.11.46/30  | SUBREDE C      |
| Router3     | f0/0       | (ligação Switch2)| Inter-VLAN     |
| VM-W/L      | —          | DHCP             | VLAN 10        |
| PC1         | e0         | DHCP             | SUBREDE A      |
| PC2         | e0         | DHCP             | SUBREDE D      |
| PC3         | e0         | DHCP             | VLAN 30        |
| PC4         | e0         | DHCP             | VLAN 20        |

### Configuração Router1 (MikroTik)

```bash
# Interface WAN (DHCP client)
/ip dhcp-client add interface=ether1 disabled=no

# Interface SUBREDE A
/ip address add address=10.206.11.1/27 interface=ether2

# Interface SUBREDE B
/ip address add address=10.206.11.41/30 interface=ether3
```

### Configuração Router2 (Cisco C2691)

```cisco
interface FastEthernet0/0
 ip address 10.206.11.42 255.255.255.252
 no shutdown

interface Serial0/0
 ip address 10.206.11.45 255.255.255.252
 clock rate 64000
 no shutdown

interface FastEthernet0/1
 ip address 10.206.8.1 255.255.254.0
 no shutdown
```

### Configuração Router3 (Cisco C2691)

```cisco
interface Serial0/0
 ip address 10.206.11.46 255.255.255.252
 no shutdown

interface FastEthernet0/0
 no shutdown
```

### Configuração VLANs no Switch2 (EtherSwitch C2691)

```cisco
vlan database
 vlan 10 name VLAN10
 vlan 20 name VLAN20
 vlan 30 name VLAN30

interface FastEthernet1/1
 switchport mode access
 switchport access vlan 10

interface FastEthernet1/2
 switchport mode access
 switchport access vlan 30

interface FastEthernet1/3
 switchport mode access
 switchport access vlan 20

interface FastEthernet0/0
 switchport mode trunk
```

### Configuração Sub-interfaces Inter-VLAN no Router3

```cisco
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.206.11.33 255.255.255.248

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.206.10.1 255.255.255.0

interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.206.0.1 255.255.248.0
```

### DHCP nos Routers (para PCs e VM)

**Router1 — SUBREDE A (PC1):**
```bash
/ip pool add name=pool-subA ranges=10.206.11.2-10.206.11.30
/ip dhcp-server add interface=ether2 address-pool=pool-subA disabled=no
/ip dhcp-server network add address=10.206.11.0/27 gateway=10.206.11.1 dns-server=8.8.8.8
```

**Router2 — SUBREDE D (PC2):**
```cisco
ip dhcp pool SUBREDE_D
 network 10.206.8.0 255.255.254.0
 default-router 10.206.8.1
 dns-server 8.8.8.8

ip dhcp excluded-address 10.206.8.1 10.206.8.10
```

**Router3 — VLANs (PC3, PC4, VM-W/L):**
```cisco
ip dhcp pool VLAN10
 network 10.206.11.32 255.255.255.248
 default-router 10.206.11.33
 dns-server 8.8.8.8

ip dhcp pool VLAN20
 network 10.206.10.0 255.255.255.0
 default-router 10.206.10.1
 dns-server 8.8.8.8

ip dhcp pool VLAN30
 network 10.206.0.0 255.255.248.0
 default-router 10.206.0.1
 dns-server 8.8.8.8

ip dhcp excluded-address 10.206.11.33
ip dhcp excluded-address 10.206.10.1
ip dhcp excluded-address 10.206.0.1
```

---

## Exercício 3 — Interfaces Loopback

**Cotação: até 0.5 valores**

```cisco
! Router1 (MikroTik)
/ip address add address=1.1.1.1/32 interface=loopback

! Router2 (Cisco)
interface Loopback0
 ip address 2.2.2.2 255.255.255.255

! Router3 (Cisco)
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
```

> As gamas de IP dos loopbacks são livres. Os endereços acima são sugestões simples e fáceis de memorizar.

---

## Exercício 4 — OSPF

**Cotação: até 1 valor**

### Router2 (Cisco)

```cisco
router ospf 1
 network 10.206.11.40 0.0.0.3 area 0
 network 10.206.11.44 0.0.0.3 area 0
 network 10.206.8.0 0.0.1.255 area 0
 network 2.2.2.2 0.0.0.0 area 0
```

### Router3 (Cisco)

```cisco
router ospf 1
 network 10.206.11.44 0.0.0.3 area 0
 network 10.206.11.32 0.0.0.7 area 0
 network 10.206.10.0 0.0.0.255 area 0
 network 10.206.0.0 0.0.7.255 area 0
 network 3.3.3.3 0.0.0.0 area 0
```

### Router1 (MikroTik)

```bash
/routing ospf instance add name=default router-id=1.1.1.1
/routing ospf network add network=10.206.11.40/30 area=backbone
/routing ospf network add network=10.206.11.0/27 area=backbone
/routing ospf network add network=1.1.1.1/32 area=backbone
```

> Verifica a adjacência OSPF com `show ip ospf neighbor` nos Cisco e `/routing ospf neighbor print` no MikroTik.

---

## Exercício 5 — Acesso à Internet (VM-W/L)

**Cotação: até 1 valor**

Para que a VM-W/L aceda à Internet (ping 8.8.8.8 e sites), é necessário:

### 1. NAT no Router1 (MikroTik)

```bash
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

### 2. Rota default no Router1

```bash
/ip route add dst-address=0.0.0.0/0 gateway=[gateway-dhcp-wan]
```

> O gateway é atribuído automaticamente pelo DHCP da interface WAN.

### 3. Redistribuição da rota default no OSPF (Router1)

```bash
/routing ospf instance set default redistribute-default=always
```

### 4. Nos Cisco — aceitar rota default via OSPF

```cisco
router ospf 1
 default-information originate
```

### 5. Verificação

Na VM-W/L:
```bash
ping 8.8.8.8
curl https://www.google.com
```

---

## Exercício 6 — Firewall entre PCs clientes

**Cotação: até 1 valor**

PC2, PC3 e PC4 não podem comunicar entre si.

### Implementação no Router2 e Router3 com ACLs

**Router2 — bloquear tráfego entre SUBREDE D e VLANs:**

```cisco
ip access-list extended BLOCK_PC_INTER
 deny ip 10.206.8.0 0.0.1.255 10.206.0.0 0.0.7.255
 deny ip 10.206.8.0 0.0.1.255 10.206.10.0 0.0.0.255
 deny ip 10.206.8.0 0.0.1.255 10.206.11.32 0.0.0.7
 permit ip any any

interface FastEthernet0/1
 ip access-group BLOCK_PC_INTER in
```

**Router3 — bloquear tráfego entre VLANs (nos sentidos relevantes):**

```cisco
ip access-list extended BLOCK_VLAN_INTER
 deny ip 10.206.0.0 0.0.7.255 10.206.10.0 0.0.0.255
 deny ip 10.206.0.0 0.0.7.255 10.206.11.32 0.0.0.7
 deny ip 10.206.10.0 0.0.0.255 10.206.0.0 0.0.7.255
 deny ip 10.206.10.0 0.0.0.255 10.206.11.32 0.0.0.7
 deny ip 10.206.11.32 0.0.0.7 10.206.0.0 0.0.7.255
 deny ip 10.206.11.32 0.0.0.7 10.206.10.0 0.0.0.255
 permit ip any any

interface FastEthernet0/0.30
 ip access-group BLOCK_VLAN_INTER in

interface FastEthernet0/0.20
 ip access-group BLOCK_VLAN_INTER in

interface FastEthernet0/0.10
 ip access-group BLOCK_VLAN_INTER in
```

> **Nota no relatório:** As regras foram implementadas com ACLs Extended nos Routers 2 e 3, bloqueando o tráfego IP entre os segmentos onde residem PC2 (SUBREDE D), PC3 (VLAN 30) e PC4 (VLAN 20).

---

## Exercício 7 — Telnet apenas PC1 para R2

**Cotação: até 0.5 valores**

### Router2 (Cisco)

```cisco
! Definir password de acesso telnet
line vty 0 4
 password rc2
 login
 transport input telnet

! Password modo privilegiado
enable password rc2

! ACL para permitir apenas PC1
ip access-list standard TELNET_PC1
 permit host [IP do PC1]
 deny any

line vty 0 4
 access-class TELNET_PC1 in
```

> O IP do PC1 é atribuído por DHCP via Router1 na SUBREDE A (10.206.11.1–10.206.11.30). Recomenda-se configurar o PC1 com IP fixo ou reserva DHCP para garantir consistência.

**Reserva de IP fixo para PC1 no MikroTik:**
```bash
/ip dhcp-server lease add address=10.206.11.2 mac-address=[MAC do PC1] server=dhcp1
```

---

## Exercício 8 — Port Knocking no R1

**Cotação: até 2 valores**

O Port Knocking no MikroTik é implementado com **Firewall Filter** usando address-lists temporárias.

```bash
# Stage 1 — knock na porta 3000 TCP
/ip firewall filter add chain=input protocol=tcp dst-port=3000 \
  src-address-list=!knock2 src-address-list=!knock3 \
  action=add-src-to-address-list address-list=knock1 \
  address-list-timeout=15s comment="Port Knock Stage 1"

# Stage 2 — knock na porta 4000 TCP (apenas quem fez knock1)
/ip firewall filter add chain=input protocol=tcp dst-port=4000 \
  src-address-list=knock1 \
  action=add-src-to-address-list address-list=knock2 \
  address-list-timeout=15s comment="Port Knock Stage 2"

# Stage 3 — knock na porta 5000 TCP (apenas quem fez knock1 e knock2)
/ip firewall filter add chain=input protocol=tcp dst-port=5000 \
  src-address-list=knock2 \
  action=add-src-to-address-list address-list=authorised \
  address-list-timeout=30m comment="Port Knock Stage 3 - autoriza login 30min"

# Permitir login apenas a IPs autorizados
/ip firewall filter add chain=input src-address-list=authorised \
  action=accept comment="Allow login after knock"

# Bloquear acesso ao router por defeito (adicionar no final da chain input)
/ip firewall filter add chain=input action=drop \
  comment="Drop all other input"
```

> **Notas:**
> - Cada stage tem timeout de **15 segundos** (address-list-timeout=15s)
> - Após completar os 3 knocks, o IP fica autorizado por **30 minutos**
> - Para testar: usa `nmap -p 3000 [IP_R1]`, depois `nmap -p 4000 [IP_R1]`, depois `nmap -p 5000 [IP_R1]`, e por fim conecta via SSH/Winbox

---

## Exercício 9 — Restrição de acesso da VM-W/L

**Cotação: até 1 valor**

A VM-W/L só pode aceder a:
- https://www.portaldasfinancas.gov.pt/
- https://www.seg-social.pt/

Apenas entre as **9:00–12:00** e **14:00–17:00**. Tudo o resto é negado.

### Implementação no Router1 (MikroTik)

```bash
# Time schedules
/ip firewall mangle add chain=prerouting \
  src-address=10.206.11.33/29 \
  time=9h-12h,mon,tue,wed,thu,fri,sat,sun \
  action=mark-connection new-connection-mark=vm-time-ok passthrough=yes

/ip firewall mangle add chain=prerouting \
  src-address=10.206.11.33/29 \
  time=14h-17h,mon,tue,wed,thu,fri,sat,sun \
  action=mark-connection new-connection-mark=vm-time-ok passthrough=yes

# Permitir apenas os sites autorizados no horário correto
/ip firewall filter add chain=forward \
  src-address=10.206.11.33/29 \
  connection-mark=vm-time-ok \
  dst-address=91.84.2.0/24 \
  action=accept comment="Allow portaldasfinancas no horario"

/ip firewall filter add chain=forward \
  src-address=10.206.11.33/29 \
  connection-mark=vm-time-ok \
  dst-address=194.210.35.0/24 \
  action=accept comment="Allow seg-social no horario"

# Bloquear todo o resto da VM-W/L
/ip firewall filter add chain=forward \
  src-address=10.206.11.33/29 \
  action=drop comment="Block all other VM-W/L traffic"
```

> **Nota:** Os endereços IP dos sites podem variar. Resolve com `nslookup portaldasfinancas.gov.pt` e `nslookup seg-social.pt` antes de configurar e ajusta os blocos de rede conforme necessário.

---

## Exercício 10 — Restrição de acesso do PC2

**Cotação: até 1 valor**

PC2 só pode aceder à Internet entre as **17:00 e as 23:00, de segunda a sexta**, com velocidade limitada a **1 Mbit/s** (download e upload).

### Implementação no Router2 (Cisco) — ACL com time-range

```cisco
! Definir time-range
time-range HORARIO_PC2
 periodic weekdays 17:00 to 23:00

! ACL com time-range
ip access-list extended PC2_INTERNET
 permit ip host [IP_PC2] any time-range HORARIO_PC2
 deny ip host [IP_PC2] any

interface FastEthernet0/1
 ip access-group PC2_INTERNET in
```

### Limitação de velocidade (traffic shaping) no Router2

```cisco
! Policy map para limitar a 1Mbit
class-map match-any PC2_CLASS
 match access-group name PC2_TRAFFIC

ip access-list extended PC2_TRAFFIC
 permit ip host [IP_PC2] any
 permit ip any host [IP_PC2]

policy-map LIMIT_PC2
 class PC2_CLASS
  police rate 1000000 bps

interface FastEthernet0/1
 service-policy input LIMIT_PC2
 service-policy output LIMIT_PC2
```

> O Cisco C2691 tem suporte limitado a traffic policing. Alternativamente, a limitação de velocidade pode ser feita no Router1 (MikroTik) com Queue Trees, que têm melhor suporte.

**Alternativa no MikroTik (Queue):**
```bash
/queue simple add name=PC2-limit target=[IP_PC2]/32 \
  max-limit=1M/1M \
  time=17h-23h,mon,tue,wed,thu,fri
```

---

## Exercício 11 — Bloqueio de tráfego entre VLANs no R3

**Cotação: até 1 valor**

O Router R3 deve bloquear tráfego entre VLANs, permitindo apenas comunicação pelos **portos 80 (HTTP) e 443 (HTTPS)**.

### Router3 (Cisco)

```cisco
ip access-list extended INTER_VLAN_POLICY
 permit tcp 10.206.0.0 0.0.7.255 10.206.10.0 0.0.0.255 eq 80
 permit tcp 10.206.0.0 0.0.7.255 10.206.10.0 0.0.0.255 eq 443
 permit tcp 10.206.0.0 0.0.7.255 10.206.11.32 0.0.0.7 eq 80
 permit tcp 10.206.0.0 0.0.7.255 10.206.11.32 0.0.0.7 eq 443
 permit tcp 10.206.10.0 0.0.0.255 10.206.0.0 0.0.7.255 eq 80
 permit tcp 10.206.10.0 0.0.0.255 10.206.0.0 0.0.7.255 eq 443
 permit tcp 10.206.10.0 0.0.0.255 10.206.11.32 0.0.0.7 eq 80
 permit tcp 10.206.10.0 0.0.0.255 10.206.11.32 0.0.0.7 eq 443
 permit tcp 10.206.11.32 0.0.0.7 10.206.0.0 0.0.7.255 eq 80
 permit tcp 10.206.11.32 0.0.0.7 10.206.0.0 0.0.7.255 eq 443
 permit tcp 10.206.11.32 0.0.0.7 10.206.10.0 0.0.0.255 eq 80
 permit tcp 10.206.11.32 0.0.0.7 10.206.10.0 0.0.0.255 eq 443
 deny ip 10.206.0.0 0.0.7.255 10.206.10.0 0.0.0.255
 deny ip 10.206.0.0 0.0.7.255 10.206.11.32 0.0.0.7
 deny ip 10.206.10.0 0.0.0.255 10.206.0.0 0.0.7.255
 deny ip 10.206.10.0 0.0.0.255 10.206.11.32 0.0.0.7
 deny ip 10.206.11.32 0.0.0.7 10.206.0.0 0.0.7.255
 deny ip 10.206.11.32 0.0.0.7 10.206.10.0 0.0.0.255
 permit ip any any

interface FastEthernet0/0.10
 ip access-group INTER_VLAN_POLICY in

interface FastEthernet0/0.20
 ip access-group INTER_VLAN_POLICY in

interface FastEthernet0/0.30
 ip access-group INTER_VLAN_POLICY in
```

---

## Exercício 12 — API MikroTik

**Cotação: até 3 valores**

### Ativar a API no Router1 (MikroTik)

```bash
/ip service enable api
/ip service set api port=8728
```

### Exemplo em Python — Ligar e obter nome e IPs

```python
import socket
import hashlib

class MikroTikAPI:
    def __init__(self, host, username, password, port=8728):
        self.host = host
        self.username = username
        self.password = password
        self.port = port
        self.sock = None

    def connect(self):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.connect((self.host, self.port))

    def send_sentence(self, sentence):
        for word in sentence:
            encoded = word.encode('utf-8')
            length = len(encoded)
            self.sock.sendall(self._encode_length(length) + encoded)
        self.sock.sendall(b'\x00')  # end of sentence

    def _encode_length(self, length):
        if length < 0x80:
            return bytes([length])
        elif length < 0x4000:
            length |= 0x8000
            return bytes([(length >> 8) & 0xFF, length & 0xFF])
        else:
            length |= 0xC00000
            return bytes([(length >> 16) & 0xFF, (length >> 8) & 0xFF, length & 0xFF])

    def read_sentence(self):
        sentence = []
        while True:
            word = self._read_word()
            if word == '':
                break
            sentence.append(word)
        return sentence

    def _read_word(self):
        length = self._read_length()
        if length == 0:
            return ''
        data = b''
        while len(data) < length:
            chunk = self.sock.recv(length - len(data))
            if not chunk:
                raise ConnectionError("Connection closed")
            data += chunk
        return data.decode('utf-8')

    def _read_length(self):
        b = self.sock.recv(1)[0]
        if b < 0x80:
            return b
        elif b < 0xC0:
            b2 = self.sock.recv(1)[0]
            return ((b & ~0x80) << 8) | b2
        else:
            b2 = self.sock.recv(1)[0]
            b3 = self.sock.recv(1)[0]
            return ((b & ~0xC0) << 16) | (b2 << 8) | b3

    def login(self):
        self.send_sentence(['/login', f'=name={self.username}', f'=password={self.password}'])
        response = self.read_sentence()
        return '!done' in response

    def command(self, cmd, params=None):
        sentence = [cmd]
        if params:
            sentence += [f'={k}={v}' for k, v in params.items()]
        self.send_sentence(sentence)
        results = []
        while True:
            reply = self.read_sentence()
            if '!done' in reply:
                break
            if '!re' in reply:
                result = {}
                for item in reply:
                    if item.startswith('=') and '=' in item[1:]:
                        k, v = item[1:].split('=', 1)
                        result[k] = v
                results.append(result)
        return results

    def get_identity(self):
        result = self.command('/system/identity/print')
        return result[0].get('name', 'N/A') if result else 'N/A'

    def get_ip_addresses(self):
        return self.command('/ip/address/print')

    def change_password(self, old_password, new_password):
        # Verificar password antiga
        test = MikroTikAPI(self.host, self.username, old_password, self.port)
        test.connect()
        if not test.login():
            print("Password antiga incorreta!")
            return False
        # Alterar password
        self.command('/user/set', {
            '.id': '*1',  # admin user ID
            'password': new_password
        })
        print("Password alterada com sucesso!")
        return True


# =====================
# PROGRAMA PRINCIPAL
# =====================

def main():
    HOST = '10.206.11.1'  # IP do Router1 (MikroTik) na SUBREDE A
    USER = 'admin'
    PASS = ''  # password padrão MikroTik

    api = MikroTikAPI(HOST, USER, PASS)
    api.connect()

    if not api.login():
        print("Erro no login!")
        return

    print("=== Ligado ao MikroTik ===\n")

    # Nome do router
    nome = api.get_identity()
    print(f"Nome do Router: {nome}")

    # Endereços IP
    print("\nEndereços IP configurados:")
    ips = api.get_ip_addresses()
    for ip in ips:
        print(f"  Interface: {ip.get('interface','?')} | IP: {ip.get('address','?')}")

    # Alterar password
    print("\n--- Alterar Password ---")
    old_pass = input("Password atual: ")
    new_pass = input("Nova password: ")
    api.change_password(old_pass, new_pass)


if __name__ == '__main__':
    main()
```

### Como executar

```bash
pip install # sem dependências externas — usa apenas socket nativo
python mikrotik_api.py
```

> **Referência oficial da API MikroTik:** https://help.mikrotik.com/docs/display/ROS/API

---

## Resumo das Cotações

| Exercício | Descrição                          | Cotação |
|-----------|------------------------------------|---------|
| 1         | VLSM                               | 2 val   |
| 2         | Topologia GNS3 + configurações     | 1 val   |
| 3         | Interfaces Loopback                | 0.5 val |
| 4         | OSPF                               | 1 val   |
| 5         | Acesso Internet VM-W/L             | 1 val   |
| 6         | Firewall entre PCs clientes        | 1 val   |
| 7         | Telnet PC1 → R2                    | 0.5 val |
| 8         | Port Knocking R1                   | 2 val   |
| 9         | Restrição acesso VM-W/L            | 1 val   |
| 10        | Restrição acesso PC2               | 1 val   |
| 11        | Bloqueio inter-VLAN (portos 80/443)| 1 val   |
| 12        | API MikroTik                       | 3 val   |
| 13        | Qualidade do relatório             | 5 val   |
| **Total** |                                    | **20 val** |

---

*Guia elaborado com base no enunciado do Projeto RC2 2025/2026 — IPBeja ESTG*
