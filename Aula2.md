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

**Pendente:** os testes de alternância de sessão (`su - fulano` validando acesso permitido, e `su - novato` validando acesso negado) e o exercício de fixação com o grupo `financeiro` ainda não foram executados nesta prática — a documentação será complementada assim que forem realizados.

## 6. Problemas e Soluções

- **Erro de digitação com efeito colateral:** o comando `tail -n 4 /ec;passwd` continha dois erros — o caminho `/ec` (faltando `t` de `/etc`) e um `;` em vez de `/`. Como o `;` separa comandos no shell, isso executou `tail -n 4 /ec` (que falhou, "No such file or directory") seguido de `passwd` isolado, que tentou trocar a senha do próprio usuário `administrador`. O comando falhou com `Authentication token manipulation error` e a senha permaneceu inalterada — sem impacto, mas serviu de alerta para conferir o comando antes de executar (Enter) quando ele envolve caracteres especiais.

- **Usuário `cicrano` não foi criado:** ao montar a sequência de `adduser`, o usuário `cicrano` foi esquecido, criando apenas `fulano`, `beltrano` e `novato`. Isso causou falha nos dois comandos seguintes (`sudo usermod -aG devs cicrano` e a tentativa com `Cicrano` maiúsculo), ambos retornando `usermod: user 'cicrano' does not exist`. **Solução:** executar `sudo adduser cicrano` e, em seguida, `sudo usermod -aG devs cicrano`.

- **`novato` associado incorretamente ao grupo `devs`:** por engano, `sudo usermod -aG devs novato` foi executado, incluindo o usuário `novato` no grupo `devs` — o que compromete o objetivo do exercício, já que `novato` deveria representar o perfil externo (sem acesso) nos testes de permissão. **Solução:** remover a associação com `sudo gpasswd -d novato devs` antes de prosseguir com os testes de `su -`.

- **Grupo do arquivo `config_redes.txt` diferente do esperado:** ao criar o arquivo com `echo ... > /srv/projeto/config_redes.txt`, o grupo associado ficou como `administrador` em vez de `devs`. Isso ocorre porque, sem o bit SGID ativado no diretório pai, arquivos novos herdam o grupo primário de quem os cria, não o grupo do diretório. Para que arquivos futuros herdem automaticamente o grupo `devs`, seria necessário `sudo chmod g+s /srv/projeto`; para corrigir o arquivo já existente, `sudo chgrp devs /srv/projeto/config_redes.txt`.

## 7. Conclusão
*[PREENCHER após concluir os testes de `su -` e o exercício de fixação com o grupo `financeiro` — a conclusão deve refletir o aprendizado completo da prática, incluindo os erros da seção 6.]*

<img src="imagens/WhatsApp Image 2026-08-26 at 17.58.17.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 17.59.35.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.01.37.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.03.05.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.04.13.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.05.24.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.07.35.jpeg">
<img src="imagens/WhatsApp Image 2026-08-26 at 18.09.08.jpeg">
