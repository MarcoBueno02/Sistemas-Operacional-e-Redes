# Estrutura de Diretórios, Pastas do Sistema (FHS) e Permissões Avançadas no Linux Server

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Disciplina:** Laboratório de Sistemas Operacional e Redes
- **Data:** 27/08/2026
- **Título da prática:** Estrutura de Diretórios, Pastas do Sistema (FHS) e Permissões Avançadas

## 2. Objetivo
Explorar o padrão FHS (Filesystem Hierarchy Standard) do Linux, criar estruturas de diretórios corporativos de forma recursiva com `mkdir -p`, e configurar isolamento de acesso departamental (`ti-dept`, `vendas-dept` e `diretoria-dept`) através de grupos do sistema e permissões octais, validando os resultados com alternância de sessão via `su -`.

## 3. Ambiente
- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Usuário administrativo:** `administrador`

## 4. Procedimento

**Inspeção de pastas do sistema (padrão FHS):**
```bash
cd /etc
ls -F | head -n 15
pwd
sudo tail -n 10 /var/log/auth.log
```

**Criação recursiva dos diretórios departamentais:**
```bash
cd /srv
sudo mkdir -p ti-dept/projetos vendas-deps/relatorios   # nome digitado incorretamente (ver seção 6)
ls -R
```

**Criação dos grupos departamentais e associação de usuários:**
```bash
sudo groupadd ti-group
sudo groupadd vendas-group
sudo usermod -aG ti-group fulano
sudo usermod -aG vendas-group cicrano
```

**Ajuste de posse e permissões (com correções, ver seção 6):**
```bash
sudo mv vendas-deps vendas-dept
sudo chown administrador:ti-group ti-dept
sudo chown administrador:vendas-group vendas-dept
sudo chmod 770 ti-dept
sudo chmod 770 vendas-dept
sudo chown -R administrador:ti-group ti-dept/
sudo chown -R administrador:vendas-group vendas-dept/
ls -ld ti-dept vendas-dept
```

**Criação do arquivo de teste departamental:**
```bash
sudo touch ti-dept/projetos/arquitetura_rede_vpn.txt
sudo chown administrador:ti-group ti-dept/projetos/arquitetura_rede_vpn.txt
sudo chmod 660 ti-dept/projetos/arquitetura_rede_vpn.txt
ls -l ti-dept/projetos/
```

**Desafio de laboratório — departamento de diretoria:**
```bash
sudo mkdir -p /srv/diretoria-dept
sudo groupadd diretoria-group
sudo usermod -aG diretoria-group beltrano
sudo chown administrador:diretoria-group /srv/diretoria-dept
sudo chmod 770 /srv/diretoria-dept
sudo touch /srv/diretoria-dept/orcamento_ti.txt
sudo chown administrador:diretoria-group /srv/diretoria-dept/orcamento_ti.txt
sudo chmod 660 /srv/diretoria-dept/orcamento_ti.txt
```

## 5. Testes e Validação

**Log de autenticação (`/var/log/auth.log`)** confirmou sessões anteriores de `su -` registradas pelo PAM, mostrando abertura e fechamento de sessão para `fulano`, `novato` e `cicrano`, além de comandos `sudo` executados pelo `administrador` — evidenciando que o sistema audita corretamente as trocas de sessão.

**Estrutura final de diretórios (`ls -ld`):**
```
drwxrwx--- 3 administrador ti-group     4096 Aug 26 22:26 ti-dept
drwxrwx--- 3 administrador vendas-group 4096 Aug 26 22:26 vendas-dept
```

**Arquivo de teste em `ti-dept`:**
```
-rw-rw---- 1 administrador ti-group 0 Aug 27 03:26 arquitetura_rede_vpn.txt
```

**Teste A — `fulano` (grupo `ti-group`):**
```bash
su - fulano
pwd            # /home/fulano
cd /srv/ti-dept
ls -l projetos/
```
Saída:
```
-rw-rw---- 1 administrador ti-group 0 Aug 27 03:26 arquitetura_rede_vpn.txt
```
Acesso concedido corretamente — `fulano` pertence ao grupo `ti-group`.

**Teste B — `cicrano` (grupo `vendas-group`, sem acesso a TI):**
```bash
su - cicrano
cd /srv/ti-dept
```
Saída: `-bash: cd: /srv/ti-dept: Permission denied` — acesso corretamente negado.

**Desafio — departamento de diretoria:**
```
drwxrwx--- 2 administrador diretoria-group 4096 Aug 27 03:33 /srv/diretoria-dept
-rw-rw---- 1 administrador diretoria-group 0 Aug 27 03:33 orcamento_ti.txt
```

Teste com `beltrano` (membro do `diretoria-group` — deve acessar):
```bash
su - beltrano
cd /srv/diretoria-dept
ls -l
```
Saída:
```
-rw-rw---- 1 administrador diretoria-group 0 Aug 27 03:33 orcamento_ti.txt
```
Acesso concedido corretamente.

Teste com `fulano` (fora do `diretoria-group` — deve ser bloqueado):
```bash
su - fulano
cd /srv/diretoria-dept
```
Saída: `-bash: cd: /srv/diretoria-dept: Permission denied` — acesso corretamente negado.

Todos os testes confirmam o isolamento departamental pretendido: cada grupo (`ti-group`, `vendas-group`, `diretoria-group`) tem acesso exclusivo ao seu próprio diretório, sem vazamento de permissão entre departamentos.

## 6. Problemas e Soluções

- **Nome de diretório digitado incorretamente:** ao criar as pastas com `sudo mkdir -p ti-dept/projetos vendas-deps/relatorios`, a pasta da equipe de vendas foi criada como `vendas-deps` (plural) em vez de `vendas-dept`, conforme o roteiro. Isso causou falha em cascata nos comandos seguintes que referenciavam `vendas-dept`, todos retornando `No such file or directory`. **Solução:** `sudo mv vendas-deps vendas-dept`, renomeando a pasta para o nome correto antes de prosseguir.

- **Confusão entre `chown` e `chmod`:** o comando `sudo chown 770 ti-dept` foi executado no lugar de `sudo chmod 770 ti-dept`. Como `chown` altera posse (dono/grupo) e não permissões, e `770` não corresponde a nenhum usuário nomeado, o sistema interpretou o valor como UID numérico, atribuindo à pasta um dono inválido (UID 770). **Solução:** reexecutar `sudo chown administrador:ti-group ti-dept` para restaurar o dono correto, seguido do `sudo chmod 770 ti-dept` (comando correto para permissões octais).

- **Comando `chown -R` sem `sudo`:** a primeira tentativa de aplicar a posse recursivamente (`chown -R administrador:ti-group ti-dept/`) foi executada sem o prefixo `sudo`, retornando `Operation not permitted`. Isso ocorreu em conjunto com o problema anterior — como o dono da pasta havia virado um UID inválido, apenas o root teria permissão para alterá-lo novamente. **Solução:** repetir o comando com `sudo chown -R administrador:ti-group ti-dept/`.

## 7. Conclusão
Esta prática aprofundou a compreensão do padrão FHS do Linux e da forma como diretórios de serviço (`/srv`) podem ser estruturados para isolar o acesso entre diferentes departamentos de uma organização usando grupos e permissões octais. Os erros cometidos ao longo da execução — um typo no nome de uma pasta, a confusão entre os comandos `chown` (posse) e `chmod` (permissões), e a omissão do `sudo` em uma operação que exigia privilégio elevado — reforçaram, na prática, a importância de revisar a saída de cada comando antes de prosseguir, já que um erro não tratado se propaga para os comandos seguintes. Os testes de isolamento com `su -` confirmaram que o modelo de permissões Linux (dono/grupo/outros) escala corretamente para múltiplos departamentos isolados em um mesmo servidor: cada grupo (`ti-group`, `vendas-group`, `diretoria-group`) obteve acesso exclusivo ao seu diretório, sem que usuários de outros departamentos conseguissem sequer navegar até as pastas alheias — validando o valor prático da segregação de acesso em ambientes corporativos reais.

<img src="imagens/WhatsApp Image 2026-08-27 at 00.37.57.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.37.11.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.30.38.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.29.51.jpeg">
<img src="imagens/WhatsApp Image 2026-08-27 at 00.29.22.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.30.50.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.27.59.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.26.23.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.24.59.jpeg">
