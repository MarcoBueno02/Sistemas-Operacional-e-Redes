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
TCP    127.0.0.1:5222         0.0.0.0:0              LISTENING
TCP    192.168.100.151:56201  57.144.165.32:5222     ESTABLISHED
```
A porta `5222` já aparece em `LISTENING` no `127.0.0.1`, confirmando que o VirtualBox está escutando localmente e pronto para redirecionar a conexão para a VM. *(A segunda linha continua sendo a conexão não relacionada de outro processo — ver seção 6.)*

**Conexão SSH estabelecida (`ssh -p 5222 administrador@127.0.0.1`):**
```
PS C:\Users\marqu> ssh -p 5222 administrador@127.0.0.1
The authenticity of host '[127.0.0.1]:5222 ([127.0.0.1]:5222)' can't be established.
ED25519 key fingerprint is SHA256:m6HWsPBaadPE/qbQq5dEpW65VYx6V0UH/4SK+sQ2Lo.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:5222' (ED25519) to the list of known hosts.
administrador@127.0.0.1's password:
Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-30-generic x86_64)

  System load:  0.73                Usage of /:  18.5% of 15.58GB
  Memory usage: 15%                 Processes:   120

administrador@ubuntuserver:~$
```
A conexão foi estabelecida com sucesso pela primeira vez, exigindo aceitação da fingerprint ED25519 do host (comportamento padrão do SSH na primeira conexão a um servidor desconhecido) e autenticação por senha, caindo diretamente no shell remoto da VM.

**`netstat` dentro da própria VM, durante a sessão SSH ativa:**
```
administrador@ubuntuserver:~$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 ubuntuserver:ssh        _gateway:58511          ESTABLISHED
```
Rodado a partir do próprio Linux, confirma do lado do servidor que existe uma conexão TCP ativa na porta `ssh` (22) com origem em `_gateway` (o endereço `10.0.2.2`, que representa o Host visto de dentro do NAT) — evidência equivalente à do lado Windows de que a sessão remota está de fato estabelecida.

**`w` dentro da sessão SSH remota (rodado na aba do PowerShell logada via SSH):**
```
administrador@ubuntuserver:~$ w
 21:57:19 up 6 min,  2 users,  load average: 0.13, 0.16, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
administ pts/0    10.0.2.2         21:56    0.00s  0.09s  ?    w
administ tty1     -                21:52    2:27   0.10s  0.10s -bash
```
Confirma as duas sessões simultâneas: `tty1` (console local da VM, sem origem, sessão aberta desde o login inicial) e `pts/0`, com origem `10.0.2.2` (o Host, visto pela VM através do NAT) — comprovando que a sessão remota via SSH está ativa e distinta da sessão local.

## 6. Problemas e Soluções

- **Falso positivo no filtro `netstat -an | findstr 5222` no Windows:** logo na primeira execução, antes mesmo de criar a regra de redirecionamento, o filtro já retornou uma linha `ESTABLISHED` para o endereço `57.144.165.32:5222`. A princípio pareceu que já existia algo usando a porta 5222, mas se tratava apenas de uma coincidência — outro processo do Windows (porta 5222 é comumente usada por protocolos como XMPP, presente em alguns aplicativos de chat) já mantinha uma conexão de saída usando essa porta, sem nenhuma relação com o redirecionamento SSH da VM. **Solução:** observar que a conexão relevante é sempre a que tem `127.0.0.1` como endereço local, e não se confundir com outras linhas retornadas pelo mesmo filtro numérico.

- **Comando `findstr` executado dentro do terminal Linux:** ao tentar repetir o filtro de porta durante a sessão SSH, o comando `netstat -an | findstr 5222` foi digitado por engano dentro do terminal da própria VM (Ubuntu), retornando `findstr: command not found`. O erro ocorreu porque `findstr` é um utilitário exclusivo do Windows (PowerShell/CMD); no Linux o equivalente seria `grep`. **Solução:** usar apenas `netstat` (sem filtro) diretamente na VM para conferir a conexão SSH ativa do lado do servidor.

- **Primeira tentativa do comando `w` na janela errada:** o comando `w` foi executado inicialmente na janela local da VM (console `tty1`) em vez de na aba do PowerShell onde a sessão SSH estava aberta, resultando em uma saída mostrando apenas uma sessão (a local), sem o `pts/0` esperado. **Solução:** repetir o comando digitando diretamente na aba SSH ativa, o que revelou corretamente as duas sessões simultâneas (`tty1` e `pts/0`).

## 7. Conclusão
Esta prática consolidou o entendimento sobre como o VirtualBox implementa o acesso a serviços de uma VM em modo NAT através do Redirecionamento de Portas, já que por padrão o modo NAT isola a VM da rede do Host, tornando necessário mapear explicitamente uma porta do Hospedeiro (`127.0.0.1:5222`) para a porta de destino no Convidado (`10.0.2.15:22`). Os diagnósticos com `ifconfig` e `route -n` reforçaram como a VM enxerga sua própria rede interna (gateway `10.0.2.2`), enquanto o `traceroute` evidenciou que todo o tráfego de saída passa por esse gateway virtual antes de qualquer roteamento externo. O uso do `netstat` nos dois lados da conexão (Windows e Linux) mostrou, na prática, os três estados de uma porta TCP — inexistente, `LISTENING` e `ESTABLISHED` — permitindo acompanhar o ciclo de vida completo da conexão SSH. Por fim, o comando `w` demonstrou de forma clara a diferença entre uma sessão local (`tty1`) e uma sessão remota (`pts/0`), habilidade fundamental para qualquer administrador auditar quem está acessando um servidor e de onde. Os próprios erros cometidos — confundir uma porta coincidente em uso por outro processo, tentar um comando exclusivo do Windows dentro do Linux, e rodar um comando na janela errada — reforçaram, na prática, a importância de prestar atenção em qual terminal e qual sistema operacional cada comando deve ser executado.
