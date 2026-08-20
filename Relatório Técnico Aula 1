# Prática de Virtualização com Oracle VirtualBox e Ubuntu Server

## 1. Identificação

- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Disciplina:** Laboratório de Sistemas Operacional e Redes
- **Data:** 19/08/2026
- **Título da prática:** Instalação e Configuração de Máquina Virtual com Ubuntu Server no Oracle VirtualBox

## 2. Objetivo

Esta prática teve como finalidade instalar e configurar uma máquina virtual (VM) com o sistema operacional Ubuntu Server no hypervisor Oracle VirtualBox, realizando a criação de um usuário administrador, a verificação da configuração de rede, a validação do particionamento em LVM (Logical Volume Manager) e a atualização dos pacotes do sistema via `apt-get update`. O exercício visa consolidar conceitos práticos de virtualização, administração de sistemas Linux e configuração de ambientes de servidor isolados.

## 3. Ambiente

- **Host físico:**
- **Sistema operacional do Host:** Windows
- **Hypervisor:** Oracle VirtualBox — versão [PREENCHER]
- **ISO do S.O. convidado:** Ubuntu Server — versão 22.04
- **Nome da VM:** VM-MARCO
- **Configurações da VM:**
  - Memória RAM alocada: 2GB
  - vCPUs: 2
  - Disco virtual: 20 GB,
  - Placa de rede: enp0s3
  - Usuário administrador criado: `administrador`
  - Hostname: `ubuntuserver`

## 4. Procedimento

1. Download da imagem ISO do Ubuntu Server no site oficial.
2. Criação de uma nova VM no Oracle VirtualBox, definindo memória RAM, número de vCPUs e disco virtual.
3. Inicialização da VM a partir da ISO montada e início do instalador do Ubuntu Server.
4. Configuração de idioma, layout de teclado e rede durante o instalador.
5. **Particionamento em LVM:** seleção do particionamento guiado com uso de LVM, criando o *volume group* `ubuntu-vg` e o *logical volume* `ubuntu-lv`, além de uma partição separada `/boot` (fora do LVM, conforme padrão do instalador).
6. **Criação do usuário administrador:** definição do nome do servidor (`ubuntuserver`), nome completo do usuário, nome de usuário (`administrador`) e senha, com posterior configuração de privilégios `sudo`.
7. Conclusão da instalação, remoção da mídia de instalação e reinicialização da VM.
8. Login via terminal com o usuário `administrador`.
9. Execução do comando `sudo apt-get update` para atualizar a lista de pacotes disponíveis.
10. Execução do comando `ip addr` para verificar a configuração de rede da interface `enp0s3`.
11. Execução do comando `df -h` para validar o particionamento e a distribuição do LVM.

## 5. Testes e Validação

**Verificação de rede (`ip addr`):**
A interface `enp0s3` foi configurada corretamente via DHCP, recebendo o endereço IPv4 `10.0.2.15/24` e endereços IPv6 (global e link-local), com MAC address `08:00:27:27:c6:fd`. A interface de loopback (`lo`) também está ativa e funcional, confirmando que a pilha de rede da VM está operacional.

**Verificação do particionamento (`df -h`):**
O comando confirmou o uso de LVM, exibindo o volume lógico `/dev/mapper/ubuntu-vg-ubuntu-lv` montado em `/` com 12 GB de tamanho total, 5,3 GB usados (49%) e 5,5 GB disponíveis. A partição `/boot`, separada do LVM, aparece como `/dev/sda2`, com 2,0 GB de tamanho, 187 MB usados (11%). Os sistemas `tmpfs` também estão corretamente montados em `/run`, `/dev/shm`, `/tmp` e `/run/user/1000`.

**Atualização de pacotes (`sudo apt-get update`):**
O comando foi executado com sucesso após autenticação `sudo`, retornando "Hit" (sem alterações pendentes) para os repositórios `security`, `main`, `updates` e `backports` do Ubuntu, e finalizando com "Reading package lists... Done", confirmando conectividade de rede e acesso aos repositórios oficiais.

*(Evidências completas nas capturas de tela do terminal anexadas a este repositório.)*

## 6. Problemas e Soluções

Não foram enfrentados erros críticos durante a execução da prática; o processo de instalação e configuração ocorreu conforme o esperado.

## 7. Conclusão

Esta prática permitiu consolidar, de forma aplicada, os conceitos fundamentais de virtualização utilizando o Oracle VirtualBox como hypervisor tipo 2. A instalação do Ubuntu Server evidenciou a importância do particionamento LVM para flexibilidade futura no gerenciamento de armazenamento, bem como a relevância da criação correta de um usuário administrador com privilégios `sudo` para a segurança e operação do sistema. A validação por meio dos comandos `ip addr`, `df -h` e `apt-get update` demonstrou, na prática, como verificar a integridade de rede, armazenamento e conectividade de um servidor recém-instalado — habilidades essenciais para a administração de ambientes Linux, sejam eles virtuais ou físicos.
