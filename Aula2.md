# Administração de Usuários, Grupos e Permissões no Linux

## 1. Identificação
- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Disciplina:** Laboratório de Sistemas Operacional e Redes
- **Data:** 26/08/2026
- **Título da prática:** Administração de Usuários, Grupos e Permissões no Ubuntu Server

## 2. Objetivo
Criar e gerenciar usuários (`fulano`, `cicrano`, `beltrano`, `novato`) e um grupo de trabalho (`devs`) no Ubuntu Server, aplicando controle de posse (`chown`/`chgrp`) e permissões octais (`chmod`) sobre o diretório compartilhado `/srv/projeto`, além de validar o isolamento de acesso alternando sessões com `su -`.

## 3. Ambiente
- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Usuário administrativo:** `administrador`

## 4. Procedimento

**Criação dos usuários:**
```bash
sudo adduser fulano
sudo adduser beltrano
sudo adduser novato
```
Cada comando solicitou senha e dados do usuário (Full Name preenchido, demais campos deixados em branco).

**Verificação das contas criadas:**
```bash
tail -n 4 /etc/passwd
```
Saída:
```
sshd:x:983:65534::/run/sshd:/usr/sbin/nologin
fulano:x:1001:1001:Fulano,,,:/home/fulano:/bin/bash
beltrano:x:1002:1002:Beltrano de tal,,,:/home/beltrano:/bin/bash
novato:x:1003:1003:Novato de tal,,,:/home/novato:/bin/bash
```

**Criação do grupo `devs` e associação dos usuários:**
```bash
sudo groupadd devs
sudo usermod -aG devs fulano
sudo usermod -aG devs cicrano   # falhou — usuário ainda não existia (ver seção 6)
sudo usermod -aG devs beltrano
sudo usermod -aG devs novato    # associação incorreta, corrigida posteriormente (ver seção 6)
```

**Verificação do grupo:**
```bash
grep "devs" /etc/group
```
Saída (antes da correção):
```
devs:x:1004:fulano,beltrano,novato
```

**Criação do diretório compartilhado e definição de posse/permissões:**
```bash
sudo mkdir -p /srv/projeto
ls -ld /srv/projeto
sudo chown administrador /srv/projeto
sudo chgrp devs /srv/projeto
ls -ld /srv/projeto
sudo chmod 770 /srv/projeto
ls -ld /srv/projeto
```

**Criação e ajuste do arquivo de teste:**
```bash
echo "Especificacao tecnica do roteador de borda" > /srv/projeto/config_redes.txt
ls -l /srv/projeto/config_redes.txt
chmod 660 /srv/projeto/config_redes.txt
ls -l /srv/projeto/config_redes.txt
```

## 5. Testes e Validação

**Estado do diretório `/srv/projeto` antes da configuração:**
```
drwxr-xr-x 2 root root 4096 Aug 26 21:02 /srv/projeto
```

**Após `chown administrador` + `chgrp devs`:**
```
drwxr-xr-x 2 administrador devs 4096 Aug 26 21:02 /srv/projeto
```

**Após `chmod 770`:**
```
drwxrwx--- 2 administrador devs 4096 Aug 26 21:02 /srv/projeto
```
Confirma o isolamento pretendido: dono e grupo `devs` com acesso total, e nenhum acesso para outros usuários.

**Arquivo `config_redes.txt` criado e ajustado:**
```
-rw-r--r-- 1 administrador administrador 43 Aug 26 21:06 /srv/projeto/config_redes.txt
-rw-rw---- 1 administrador administrador 43 Aug 26 21:06 /srv/projeto/config_redes.txt   # após chmod 660
```
Observação: o arquivo foi criado com o grupo `administrador` (grupo primário de quem o criou), e não `devs` — ver detalhes na seção 6.

**Teste A — `fulano` (grupo `devs`), após correção do grupo do arquivo:**
```bash
su - fulano
cd /srv/projeto
echo "Revisado por Fulano" >> config_redes.txt
cat config_redes.txt
```
Saída:
```
Especificacao tecnica do roteador de borda
Revisado por Fulano
```
Acesso de escrita concedido corretamente, confirmando que `fulano` pertence ao grupo `devs`.

**Teste B — `novato` (fora do grupo `devs`):**
```bash
su - novato
cd /srv/projeto
ls -l /srv/projeto
```
Resultado: acesso negado, conforme esperado — `novato` não pertence ao grupo `devs` e a pasta não concede permissão a "outros".

**Exercício de fixação — grupo `financeiro`:**
```bash
sudo groupadd financeiro
sudo usermod -aG financeiro cicrano
sudo usermod -aG financeiro beltrano
sudo mkdir -p /srv/financeiro
sudo chown administrador /srv/financeiro
sudo chgrp financeiro /srv/financeiro
sudo chmod 770 /srv/financeiro
ls -ld /srv/financeiro
```
Saída:
```
drwxrwx--- 2 administrador financeiro 4096 Aug 26 22:06 /srv/financeiro
```

Teste com `cicrano` (deve conseguir criar arquivo — pertence ao grupo `financeiro`):
```bash
su - cicrano
echo "Relatorio Q3" > /srv/financeiro/relatorio.txt
cat /srv/financeiro/relatorio.txt
```
Saída: `Relatorio Q3` — sucesso.

Teste com `fulano` (não deve conseguir — pertence ao `devs`, não ao `financeiro`):
```bash
su - fulano
cd /srv/financeiro
```
Saída: `-bash: cd: /srv/financeiro: Permission denied` — acesso corretamente negado.

Teste com `novato` (não deve conseguir — não pertence a nenhum dos dois grupos):
```bash
su - novato
cd /srv/financeiro
```
Saída: `-bash: cd: /srv/financeiro: Permission denied` — acesso corretamente negado.

Todos os resultados confirmam o isolamento de acesso pretendido: cada grupo (`devs` e `financeiro`) tem acesso exclusivo ao seu próprio diretório, e usuários fora do grupo (incluindo `novato`, sem nenhuma associação) são bloqueados em ambos.

## 6. Problemas e Soluções

- **Erro de digitação com efeito colateral:** o comando `tail -n 4 /ec;passwd` continha dois erros — o caminho `/ec` (faltando `t` de `/etc`) e um `;` em vez de `/`. Como o `;` separa comandos no shell, isso executou `tail -n 4 /ec` (que falhou, "No such file or directory") seguido de `passwd` isolado, que tentou trocar a senha do próprio usuário `administrador`. O comando falhou com `Authentication token manipulation error` e a senha permaneceu inalterada — sem impacto, mas serviu de alerta para conferir o comando antes de executar (Enter) quando ele envolve caracteres especiais.

- **Usuário `cicrano` não foi criado:** ao montar a sequência de `adduser`, o usuário `cicrano` foi esquecido, criando apenas `fulano`, `beltrano` e `novato`. Isso causou falha nos dois comandos seguintes (`sudo usermod -aG devs cicrano` e a tentativa com `Cicrano` maiúsculo), ambos retornando `usermod: user 'cicrano' does not exist`. **Solução:** executar `sudo adduser cicrano` e, em seguida, `sudo usermod -aG devs cicrano`.

- **`novato` associado incorretamente ao grupo `devs`:** por engano, `sudo usermod -aG devs novato` foi executado, incluindo o usuário `novato` no grupo `devs` — o que compromete o objetivo do exercício, já que `novato` deveria representar o perfil externo (sem acesso) nos testes de permissão. **Solução:** remover a associação com `sudo gpasswd -d novato devs` antes de prosseguir com os testes de `su -`.

- **Grupo do arquivo `config_redes.txt` diferente do esperado:** ao criar o arquivo com `echo ... > /srv/projeto/config_redes.txt`, o grupo associado ficou como `administrador` em vez de `devs`. Isso ocorre porque, sem o bit SGID ativado no diretório pai, arquivos novos herdam o grupo primário de quem os cria, não o grupo do diretório. Consequência prática: o usuário `fulano`, mesmo pertencendo ao grupo `devs` e tendo acesso de escrita no diretório, recebeu `Permission denied` ao tentar editar o arquivo diretamente, pois o grupo do arquivo ainda era `administrador`. **Solução:** `sudo chgrp devs /srv/projeto/config_redes.txt`, que corrigiu o problema e permitiu a escrita no Teste A. Para que arquivos futuros herdem automaticamente o grupo `devs` sem precisar repetir esse ajuste, seria possível ativar o bit SGID no diretório com `sudo chmod g+s /srv/projeto`.

- **Comando `usermod` sem `sudo`:** ao tentar corrigir a associação do `cicrano` ao grupo `devs`, o comando `usermod -aG devs cicrano` foi executado sem o prefixo `sudo`, resultando em `Permission denied` e `usermod: cannot lock /etc/passwd; try again later`. **Solução:** repetir o comando com `sudo usermod -aG devs cicrano`, que funcionou normalmente.

## 7. Conclusão
Esta prática consolidou, na prática, os conceitos de administração de usuários, grupos e permissões no Linux. A criação de contas com `adduser`, a organização em grupos de trabalho com `groupadd`/`usermod -aG`, e o controle de acesso via `chown`, `chgrp` e `chmod` em notação octal (770 para diretórios, 660 para arquivos) mostraram-se ferramentas fundamentais para isolar recursos entre diferentes equipes em um mesmo servidor. Os erros encontrados ao longo do processo — o esquecimento na criação de um usuário, uma associação incorreta a um grupo, um comando executado sem privilégios elevados e a diferença entre a permissão de um diretório e a de um arquivo dentro dele — reforçaram, na prática, um ponto central da administração Linux: permissões de diretório e de arquivo são independentes, e o comportamento de herança de grupo depende do bit SGID, não sendo automático por padrão. O exercício de fixação com o grupo `financeiro` validou que o mesmo modelo de permissões escala corretamente para múltiplos grupos isolados em um mesmo sistema, com cada equipe tendo acesso exclusivo ao seu próprio diretório de trabalho.

<img src="imagens/WhatsApp Image 2026-08-26 at 17.58.17.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 17.59.35.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.01.37.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.03.05.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.04.13.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.05.24.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.07.35.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.09.08.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.10.33.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.09.53.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.08.39.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 19.00.53.jpeg">
