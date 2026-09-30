# Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Turma:** BSI 2026.02
- **Data:** 30/09/2026
- **Título da prática:** Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

## 2. Objetivo
Configurar o acesso remoto seguro via SSH à VM Ubuntu Server através do mecanismo de Redirecionamento de Portas (Port Forwarding) do modo NAT do VirtualBox, além de realizar diagnóstico de rede em ambos os lados — interfaces, rota padrão, rastreamento de pacotes (`traceroute`), sessões ativas (`w`) no Linux, e monitoramento de portas TCP (`netstat`) no Windows Host.

## 3. Ambiente
- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Usuário administrativo:** `administrador`
- **Modo de rede:** NAT (Adaptador 1), com regra de redirecionamento de porta configurada (Host `127.0.0.1:5222` → Guest `10.0.2.15:22`)

## 4. Procedimento

**Diagnóstico inicial na VM (Ubuntu):**
```bash
netplan status
dpkg -l | grep openssh-server
sudo apt update
sudo apt install -y net-tools traceroute
ifconfig
route -n
traceroute 8.8.8.8
w
```

**Diagnóstico no Windows Host (PowerShell), antes do redirecionamento:**
```powershell
ipconfig /all
netstat -an | findstr 5222
```

**Configuração do redirecionamento de porta no VirtualBox:**
Configurações da VM → Rede → Adaptador 1 (NAT) → Avançado → Redirecionamento de Portas → nova regra:

| Nome | Protocolo | IP Hospedeiro | Porta Hospedeiro | IP Convidado | Porta Convidado |
|---|---|---|---|---|---|
| SSH | TCP | 127.0.0.1 | 5222 | 10.0.2.15 | 22 |

**Validação e conexão remota:**
```powershell
netstat -an | findstr 5222
ssh -p 5222 administrador@127.0.0.1
```
Dentro da sessão SSH:
```bash
w
exit
```

## 5. Testes e Validação

**`ifconfig` (VM):**
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fe39:9ca9  prefixlen 64  scopeid 0x20<link>
        inet6 fd17:625c:f037:2:a00:27ff:fe39:9ca9  prefixlen 64  scopeid 0x0<global>
        ether 08:00:27:39:9c:a9  txqueuelen 1000  (Ethernet)
        RX packets 3824  bytes 5183478 (5.1 MB)
        TX packets 2279  bytes 173109 (173.1 KB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
```
A interface `enp0s3` confirma o IP padrão do modo NAT (`10.0.2.15/24`) e o MAC address `08:00:27:39:9c:a9`.

**`route -n` (VM):**
```
Kernel IP routing table
Destination     Gateway       Genmask         Flags Metric Ref  Use Iface
0.0.0.0         10.0.2.2      0.0.0.0         UG    100    0     0  enp0s3
10.0.2.0        0.0.0.0       255.255.255.0   U     100    0     0  enp0s3
10.0.2.2        0.0.0.0       255.255.255.255 UH    100    0     0  enp0s3
192.168.100.199 10.0.2.2      255.255.255.255 UGH   100    0     0  enp0s3
```
A rota padrão (`0.0.0.0`) aponta para `10.0.2.2`, que é o gateway virtual do modo NAT do VirtualBox.

**`traceroute 8.8.8.8` (VM):**
```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.0.2.2)  0.561 ms  0.882 ms  0.898 ms
 2  _gateway (10.0.2.2)  2.131 ms  1.974 ms  2.176 ms
```
O traçado mostra apenas o gateway NAT (`10.0.2.2`) respondendo nos dois primeiros saltos, evidenciando que o VirtualBox faz NAT/mascaramento de todo o tráfego de saída da VM antes de chegar à internet real — os saltos seguintes até `8.8.8.8` não retornam resposta ICMP visível dentro do ambiente virtualizado.

**`netstat -an | findstr 5222` no Windows, ANTES do redirecionamento:**
```
TCP    192.168.100.151:56201  57.144.165.32:5222     ESTABLISHED
```
*(Ver seção 6 — essa linha não tem relação com a VM; é uma coincidência de outro processo do Windows usando a porta 5222.)*

**Regra de redirecionamento criada no VirtualBox:**

| Nome | Protocolo | Endereço IP do Hospedeiro | Porta do Hospedeiro | IP Convidado | Porta Convidado |
|---|---|---|---|---|---|
| SSH | TCP | 127.0.0.1 | 5222 | 10.0.2.15 | 22 |

**`netstat -an | findstr 5222` no Windows, DEPOIS da regra ativa:**
```
[PREENCHER — deve mostrar TCP 127.0.0.1:5222 ... LISTENING]
```

**Conexão SSH estabelecida (`ssh -p 5222 administrador@127.0.0.1`):** [PREENCHER — print do terminal logado remotamente]

**`netstat -an | findstr 5222` no Windows, DURANTE a sessão SSH ativa:**
```
[PREENCHER — deve mostrar uma linha LISTENING e um par ESTABLISHED]
```

**`w` dentro da sessão SSH remota:**
```
[PREENCHER — deve mostrar duas sessões: tty1 (console local) e pts/0 (sessão remota via SSH, vindo de 10.0.2.2)]
```

## 6. Problemas e Soluções
[PREENCHER — registrar aqui qualquer erro enfrentado, como recusa de conexão, firewall bloqueando a porta 5222, erro de digitação na regra do VirtualBox, senha incorreta no primeiro acesso SSH etc. Se não houve nenhum problema relevante, registrar isso explicitamente.]

## 7. Conclusão
[PREENCHER — refletir sobre a importância de entender o fluxo de portas em redes virtualizadas (NAT), a diferença entre uma sessão local (tty1) e uma sessão remota (pts/0), e a utilidade prática dos comandos de auditoria (`route -n`, `netstat`, `w`, `traceroute`) para diagnosticar conectividade e monitorar acessos remotos em um servidor.]
