# Projeto RC2 - 2025/2026
Repositório para o Projeto de Redes de Computadores 2.

---

## Grupo
* 21198 - Beatriz Caixeiro 
* 24105 - Antonio Rosa


---

## 1. Visão Geral do Projeto
O presente projeto visa a construção, configuração e administração de uma infraestrutura de rede simulada no ambiente **GNS3**, englobando múltiplos vendors (Cisco e MikroTik). O foco principal assenta no endereçamento eficiente (VLSM), encaminhamento dinâmico (OSPF), implementação de políticas de segurança (Firewalls, ACLs), gestão de tráfego e automação através de API.

---

## 2. Endereçamento e Cálculo VLSM
A rede base atribuída ao grupo é **10.F.0.0/16**. O valor de *F* é calculado com base no resto da divisão inteira do menor número de aluno por 256.

---

## 3. Topologia da Rede
A topologia é constituída por um misto de equipamentos Cisco e MikroTik, garantindo interoperabilidade:

* **Router 1 (R1):** MikroTik RouterOS x86 (Gateway / Ligação WAN via DHCP)
* **Router 2 (R2):** Cisco Router C2691
* **Router 3 (R3):** Cisco Router C2691
* **Switch 1:** Ethernet Switch GNS3 (Subrede D)
* **Switch 2:** EtherSwitch Router C2691 (Gestão das VLANs 10, 20 e 30)
* **End Devices:** 4x VPCS (PC1 a PC4) e 1x Máquina Virtual (VM-W/L) a correr Windows/Linux.

---

## 4. Encaminhamento e Conectividade
* **Interfaces Loopback:** Configuração de interfaces lógicas (Loopbacks) em todos os routers para estabilidade do processo de routing e identificação dos equipamentos.
* **Routing Dinâmico (OSPF):** Implementação do protocolo OSPF em toda a topologia para garantir a descoberta dinâmica de rotas e total convergência da rede, permitindo comunicação entre as diversas subredes e VLANs.
* **Acesso à Internet:** Configuração de NAT/Masquerade no MikroTik (R1) para garantir que a VM-W/L consegue resolver nomes (DNS público) e efetuar pings para o exterior (ex: `8.8.8.8`).

---

## 5. Segurança e Controlo de Acesso
* **Isolamento de Clientes:** Implementação de regras (ACLs/Firewall) que bloqueiam a comunicação direta entre as máquinas PC2, PC3 e PC4.
* **Acesso Remoto Seguro (Telnet):**
	* O router **R2** foi configurado para aceitar ligações Telnet *apenas* provenientes do **PC1**.
	* Acesso protegido pela password `rc2` (incluindo acesso ao modo privilegiado/enable).
* **Port Knocking (MikroTik - R1):**
	* Implementação de um mecanismo de segurança avançado no R1.
	* Para obter permissão de login, o cliente tem de enviar pedidos TCP numa sequência exata: **Porta 3000 -> Porta 4000 -> Porta 5000**.
	* Janela de tempo de 15 segundos entre cada "knock". Após sucesso, o IP ganha acesso ao router por 30 minutos.

---

## 6. Políticas de Tráfego e Firewall Avançada
* **VM-W/L (Controlo de Horário e Destino):**
	* Acesso restrito *apenas* aos sites: Portal das Finanças e Segurança Social.
	* Horário permitido: 09:00 - 12:00 e 14:00 - 17:00.
	* Todo o restante tráfego web bloqueado por defeito.
* **PC2 (Quality of Service e Controlo Parental):**
	* Acesso livre à Internet permitido apenas nos dias úteis (Segunda a Sexta-feira) entre as 17:00 e as 23:00.
	* Aplicação de *Traffic Shaping*: Velocidade limitada a 1 Mbps de Download e 1 Mbps de Upload.
* **Comunicação Inter-VLAN (R3):**
	* Isolamento das VLANs: Bloqueio do tráfego geral.
	* Exceções configuradas (ACLs): Permitido apenas tráfego HTTP (Porta 80) e HTTPS (Porta 443) entre as VLANs.

---

## 7. Automação e Integração (MikroTik API)
Desenvolvimento de um script externo (Python) utilizando a API do RouterOS para gestão do R1:
* **Funcionalidades Base:** Obtenção e visualização da identidade do router (System Identity) e das respetivas configurações de endereços IP.
* **Gestão de Credenciais:** Implementação de uma função que permite validar a password atual e alterar para uma nova password remotamente via código.

---

## 8. Conclusão e Demonstração
O projeto demonstra a capacidade de integrar protocolos padrão da indústria num ambiente heterogéneo. Todas as restrições, políticas de routing, e mecanismos de segurança foram devidamente testados. 