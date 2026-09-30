# Configuração de Rede Estática via Netplan e Modo Bridge no VirtualBox

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Turma:** BSI 2026.02
- **Data:** 30/09/2026
- **Título da prática:** Configuração de Rede Estática via Netplan e Modo Bridge no VirtualBox

## 2. Objetivo
Reconfigurar a interface de rede da VM Ubuntu Server, migrando do modo NAT (usado nas aulas anteriores) para o modo Adaptador em Ponte (Bridge Adapter) do VirtualBox, fazendo a VM se comportar como um dispositivo independente na rede física local. Em seguida, configurar um endereço IP estático via Netplan (em vez de DHCP), definindo manualmente IP, máscara, gateway e servidores DNS, e validar a conectividade com testes de `ping` bidirecionais e `traceroute`.

## 3. Ambiente
- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Usuário administrativo:** `administrador`
- **Modo de rede:** Adaptador em Ponte (Bridge), usando a placa física "Realtek PCIe GbE Family Controller"
- **Rede utilizada:** rede doméstica do autor (`192.168.100.0/24`), em substituição à rede do laboratório do IFAL (`172.20.20.0/22`) especificada no roteiro original — ver observação na seção 6
- **IP estático atribuído à VM:** `192.168.100.222/24`
- **Gateway:** `192.168.100.1`
- **Servidores DNS:** `192.168.100.199`, `1.1.1.1`, `8.8.8.8`

## 4. Procedimento

**Alteração do adaptador de rede da VM (NAT → Bridge):**
Configurações da VM → Rede → Adaptador 1 → Conectado a: **Adaptador em Ponte**, selecionando a placa de rede física do host.

**Verificação de IP livre na rede local, antes de atribuir (PowerShell):**
```powershell
ping 192.168.100.222
```

**Edição do arquivo de configuração do Netplan:**
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
Conteúdo definido:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.100.222/24]
      gateway4: 192.168.100.1
      nameservers:
        addresses: [192.168.100.199, 1.1.1.1, 8.8.8.8]
```

**Aplicação da configuração:**
```bash
sudo netplan apply
```

**Validação do IP atribuído:**
```bash
ip addr show enp0s3
```

**Testes de conectividade bidirecionais:**
```bash
# Na VM, ping para o Windows Host
ping -c 4 192.168.100.151
```
```powershell
# No Windows, ping para a VM
ping 192.168.100.222
```

**Testes de rastreamento até a internet (na VM):**
```bash
traceroute google.com
traceroute one.one.one.one
```

## 5. Testes e Validação

**Verificação do IP estático (`ip addr show enp0s3`):**
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:39:9c:a9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.222/24 brd 192.168.100.255 scope global enp0s3
```
A interface saiu do endereçamento dinâmico do NAT (`10.0.2.15`) e passou a usar o IP estático definido manualmente, já dentro da faixa da rede doméstica real.

**Ping da VM para o Windows Host:**
```
PING 192.168.100.151 (192.168.100.151) 56(84) bytes of data.
64 bytes from 192.168.100.151: icmp_seq=1 ttl=128 time=2.52 ms
64 bytes from 192.168.100.151: icmp_seq=2 ttl=128 time=0.919 ms
64 bytes from 192.168.100.151: icmp_seq=3 ttl=128 time=0.716 ms
64 bytes from 192.168.100.151: icmp_seq=4 ttl=128 time=0.858 ms

--- 192.168.100.151 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3070ms
```

**Ping do Windows Host para a VM:**
```
Disparando 192.168.100.222 com 32 bytes de dados:
Resposta de 192.168.100.222: bytes=32 tempo<1ms TTL=64
Resposta de 192.168.100.222: bytes=32 tempo<1ms TTL=64
Resposta de 192.168.100.222: bytes=32 tempo=1ms TTL=64
Resposta de 192.168.100.222: bytes=32 tempo<1ms TTL=64

Estatísticas do Ping para 192.168.100.222:
    Pacotes: Enviados = 4, Recebidos = 4, Perdidos = 0 (0% de perda)
```
A comunicação bidirecional confirma que, em modo Bridge, a VM e o Host se enxergam como dois dispositivos independentes na mesma rede física, diferente do isolamento imposto pelo NAT.

**`traceroute google.com` (VM):**
```
traceroute to google.com (2800:3f0:4004:812::200e), 30 hops max, 80 byte packets
 1  2804:d49:61c:e06::1  1.299 ms  1.116 ms  1.001 ms
 2  2804:d40:2:9000::1  5.721 ms  5.611 ms  5.504 ms
 3  * * *
 4  2800:3f0:4004:812::200e  46.847 ms  47.673 ms  50.215 ms
```

**`traceroute one.one.one.one` (VM):**
```
traceroute to one.one.one.one (2606:4700:4700::1111), 30 hops max, 80 byte packets
 1  2804:d49:61c:e06::1  1.590 ms  1.492 ms  2.616 ms
 2  2804:d40:2:9000::1  5.716 ms  5.272 ms  5.110 ms
 3  * * *
 4  2804:d40:80:81d::2  55.730 ms  55.441 ms  52.310 ms
 5  2400:cb00:216:3::  50.195 ms  2400:cb00:952:3::  49.200 ms  2400:cb00:216:3::  49.918 ms
 6  2400:cb00:1178:1024::ac40:dd57  46.623 ms  2400:cb00:216:1024::ac45:2a3f  57.600 ms  2400:cb00:1178:1024::ac40:dd2d  44.091 ms
```
Ambos os traceroutes chegaram ao destino final com sucesso (resolução via IPv6, provida pelo próprio provedor de internet residencial), confirmando que a resolução DNS e o roteamento para a internet estão funcionando corretamente a partir da VM já em modo Bridge.

## 6. Problemas e Soluções

- **Adaptação da rede do roteiro (IFAL → rede doméstica):** o roteiro original da aula foi desenhado para ser executado na rede física do laboratório do IFAL (faixa `172.20.20.0/22`, gateway `172.20.20.1`). Como a prática foi realizada em casa, essa rede não estava disponível. **Solução:** manter a mesma metodologia (Bridge Adapter + IP estático via Netplan) substituindo os parâmetros pela rede doméstica real (`192.168.100.0/24`, gateway `192.168.100.1`), preservando o objetivo pedagógico do exercício.

- **Mensagem "Host de destino inacessível" ao testar disponibilidade do IP:** ao verificar se o IP `192.168.100.222` estava livre com `ping`, o Windows retornou "Host de destino inacessível" em vez do "Esgotado o tempo limite do pedido" esperado. Essa mensagem indica que o próprio PC não encontrou nenhum dispositivo respondendo àquele endereço na rede local (falha de resolução ARP), o que, na prática, confirma da mesma forma que o IP está livre. Não foi necessário nenhum ajuste, apenas identificar corretamente o significado da mensagem antes de prosseguir.

- **Aviso de depreciação do `gateway4` no Netplan:** ao rodar `sudo netplan apply`, o sistema exibiu o aviso `WARNING: 'gateway4' has been deprecated, use default routes instead` três vezes. A chave `gateway4` ainda é aceita, mas versões mais recentes do Netplan recomendam declarar a rota padrão através da seção `routes`. Como o aviso não impediu a aplicação da configuração (confirmado pelo `ip addr show enp0s3` mostrando o IP correto), a configuração foi mantida como estava, registrando aqui a recomendação para uma versão futura do arquivo.

## 7. Conclusão
Esta prática evidenciou a diferença fundamental entre os modos de rede NAT e Bridge no VirtualBox: enquanto o NAT isola a VM em uma sub-rede virtual interna, mediada por um gateway artificial (`10.0.2.2`), o modo Bridge faz a VM se comportar como um equipamento físico independente, plenamente integrado à rede local real, visível e alcançável por outros dispositivos da mesma rede — e vice-versa. A configuração manual de IP estático via Netplan, em substituição ao DHCP usado até então, reforçou a compreensão da estrutura de um arquivo YAML de rede (endereço, máscara em notação CIDR, gateway e servidores DNS) e da importância da sintaxe correta (indentação por espaços) para que o `netplan apply` interprete o arquivo sem erros. Os testes de `ping` bidirecionais confirmaram a comunicação direta entre VM e Host na mesma rede física, e os `traceroute` até destinos públicos (`google.com` e `one.one.one.one`) validaram que tanto o roteamento quanto a resolução DNS estão funcionando corretamente a partir da VM. A necessidade de adaptar os parâmetros de rede do roteiro original (pensado para a rede do laboratório do IFAL) para a rede doméstica também reforçou, na prática, que a metodologia de configuração de rede é a mesma independentemente da rede específica utilizada — o que muda são apenas os valores de endereçamento.

<img width="1076" height="768" alt="image" src="https://github.com/user-attachments/assets/14f2e628-c6ca-4ff2-98a4-458bf3d197d3" />
<img width="1076" height="768" alt="image" src="https://github.com/user-attachments/assets/c1fbf5c3-f604-45f8-9e0e-2002d6229cc6" />
<img width="900" height="768" alt="image" src="https://github.com/user-attachments/assets/34aca401-d80e-42b1-918a-42fe594ca747" />
<img width="900" height="768" alt="image" src="https://github.com/user-attachments/assets/f0928dd8-bbcb-428e-a6ff-61c6282a3e5c" />
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/3d0931ca-0e68-4308-8bda-a2f13a4ebf69" />
<img width="900" height="768" alt="image" src="https://github.com/user-attachments/assets/a163087c-a82a-4457-a676-c881e583884c" />
