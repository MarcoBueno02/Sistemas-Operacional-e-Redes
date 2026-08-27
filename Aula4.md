# Manipulação, Edição, Permissões e Automação de Arquivos no Linux

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Disciplina:** Laboratório de Sistemas Operacional e Redes
- **Turma:** 2026.02
- **Data:** 27/08/2026
- **Título da prática:** Manipulação, Edição, Permissões e Automação de Arquivos no Linux

## 2. Objetivo
Praticar a manipulação de arquivos e diretórios no terminal (criação, cópia, movimentação e remoção segura), utilizar o editor de texto Nano, e automatizar o cadastro de um lote de 20 usuários através de dois scripts em Shell, validando o resultado com `getent` e login de teste via `su -`.

## 3. Ambiente
- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Recursos da VM:** 2 GB RAM, 2 vCPUs, 20 GB de disco
- **Usuário administrativo:** `administrador`

## 4. Procedimento

**Criação e edição básica de arquivo (Nano):**
```bash
cd ~
touch configuracao.conf
nano configuracao.conf
```
Conteúdo inserido:
```
# Configuração de teste do Laboratório
PORTA=8080
TIMEOUT=3
```

**Cópia, movimentação e remoção:**
```bash
mkdir backups
cp configuracao.conf backups/
mv configuracao.conf config_antiga.conf
rm -i config_antiga.conf
rm -rf backups
```

**Criação da lista de usuários:**
```bash
nano usuarios.txt
```
Conteúdo: 20 linhas, `aluno01` a `aluno20`.

**Script de criação em lote (`passo1_criar.sh`):**
```bash
#!/bin/bash

# Script de criação de usuário em lote
for usuario in $(cat usuarios.txt); do
    echo "Processando criação do usuário: $usuario"
    sudo useradd -m -s /bin/bash $usuario
done

echo "Processo de criação concluído!"
```

**Script de definição de senhas em lote (`passo2_senhas.sh`):**
```bash
#!/bin/bash

# Script de definição de senha em lote
for usuario in $(cat usuarios.txt); do
    echo "Definindo senha padronizada para: $usuario"
    echo "$usuario:$usuario" | sudo chpasswd
done

echo "Todas as senhas foram atualizadas com sucesso"
```

**Permissão de execução e execução dos scripts:**
```bash
chmod +x passo1_criar.sh
chmod +x passo2_senhas.sh
./passo1_criar.sh
./passo2_senhas.sh
```

## 5. Testes e Validação

**Execução do `passo1_criar.sh`:** processou os 20 usuários (`aluno01` a `aluno20`) em sequência, solicitando autenticação `sudo` apenas uma vez (a sessão de `sudo` permaneceu válida para as chamadas seguintes dentro do laço), finalizando com `Processo de criação concluído!`.

**Execução do `passo2_senhas.sh`:** definiu a senha de cada um dos 20 usuários (idêntica ao respectivo nome de usuário) via `chpasswd`, finalizando com `Todas as senhas foram atualizadas com sucesso`.

**Validação via `getent passwd | tail -n 20`:**
```
aluno01:x:1006:1010::/home/aluno01:/bin/bash
aluno02:x:1007:1011::/home/aluno02:/bin/bash
...
aluno20:x:1025:1029::/home/aluno20:/bin/bash
```
Confirma que os 20 usuários foram integrados corretamente à base do sistema, cada um com UID sequencial, diretório home próprio (`/home/alunoXX`) e shell `/bin/bash`.

**Validação via `getent group | tail -n 20`:** confirma a criação automática de um grupo próprio para cada usuário (comportamento padrão do `useradd -m`, que cria um grupo privado com o mesmo nome do usuário).

**Teste de login (`su - aluno01`):**
```bash
su - aluno01
pwd
```
Saída: `/home/aluno01` — confirma que o `su -` carregou corretamente o ambiente completo do novo usuário, incluindo sua home individual criada pelo script.

## 6. Problemas e Soluções

- **Tentativa de criar arquivo em diretório sem permissão:** o primeiro `touch configuracao.conf` foi executado ainda dentro de `/srv` (diretório de trabalho remanescente da Aula 3), retornando `Permission denied`, já que essa pasta pertence ao `root` e não concede escrita ao usuário `administrador` fora dos subdiretórios departamentais já configurados. **Solução:** retornar à home do usuário com `cd ~` antes de criar o arquivo.

- **Erro de digitação no valor de uma variável:** ao editar `configuracao.conf` no Nano, foi digitado `TIMEOUT=3` em vez de `TIMEOUT=30` (dígito faltando). Como o arquivo era apenas de prática e foi removido logo em seguida (`rm -i` / `rm -rf`), não gerou impacto na atividade, mas reforça a importância de revisar o conteúdo antes de salvar (`Ctrl+O`) em arquivos de configuração reais.

- **Falha de autenticação no primeiro `su - aluno01`:** a primeira tentativa de login retornou `su: Authentication failure`, provavelmente por um erro de digitação da senha (que é idêntica ao nome de usuário, definida via `chpasswd` no script). **Solução:** repetir o comando `su - aluno01` e digitar a senha com atenção, obtendo sucesso na segunda tentativa.

## 7. Conclusão
Esta prática demonstrou como a administração de sistemas Linux se apoia fortemente em arquivos de texto simples e na automação via scripts Shell. O uso do editor Nano e dos comandos básicos de manipulação (`touch`, `cp`, `mv`, `rm -i`, `rm -rf`) reforçou o cuidado necessário ao trabalhar diretamente no terminal, especialmente quanto ao diretório de trabalho atual e às permissões vigentes nele. A parte mais relevante da aula, porém, foi a automação: dividir o processo de cadastro em dois scripts especializados (criação de contas e definição de senhas), utilizando uma estrutura de repetição `for` sobre uma lista de nomes em `usuarios.txt`, permitiu cadastrar 20 usuários em segundos — uma tarefa que, se feita manualmente como na Aula 2, exigiria dezenas de comandos repetidos. Os comandos `getent passwd` e `getent group` se mostraram ferramentas de validação mais adequadas que a leitura direta de `/etc/passwd`, por consultarem a base de dados do sistema de forma estruturada. Essa prática evidencia, de forma direta, por que scripts de automação são indispensáveis para a escalabilidade da administração de redes em ambientes corporativos com um grande número de usuários.

<img src="imagens/WhatsApp Image 2026-08-27 at 01.06.50 (3).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.06.50 (2).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.06.50 (1).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.06.50.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.06.11.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.00.44 (1).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.00.44.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 01.00.36.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.50.57 (2).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.50.57 (1).jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.50.57.jpeg">
