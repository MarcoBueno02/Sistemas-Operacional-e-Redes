# Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Turma:** BSI 2026.02
- **Data:** 30/09
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
[PREENCHER — colar a saída completa, destacando inet 10.0.2.15 e o MAC address]
```

**`route -n` (VM):**
```
[PREENCHER — colar a saída, destacando a linha do gateway padrão 10.0.2.2]
```

**`traceroute 8.8.8.8` (VM):**
```
[PREENCHER — colar os saltos exibidos]
```

**`netstat -an | findstr 5222` no Windows, ANTES do redirecionamento:**
```
[PREENCHER — deve vir vazio, sem nenhuma linha retornada]
```

**Regra de redirecionamento criada no VirtualBox:** [PREENCHER — print da tabela com a regra SSH]

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
