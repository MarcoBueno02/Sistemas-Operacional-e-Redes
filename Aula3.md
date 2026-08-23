# Estrutura de Diretórios, Pastas do Sistema (FHS) e Permissões Avançadas

## 1. Identificação

- **Nome completo:** Marco André da Costa Bueno Padilha
- **Curso:** Sistemas de Informação
- **Disciplina:** Laboratório de Sistemas Operacional e Redes
- **Data:** [PREENCHER]
- **Título da prática:** Estrutura de Diretórios FHS e Permissões Departamentais no Linux

## 2. Objetivo

Explorar o padrão FHS (`/etc`, `/var`, `/srv`, `/bin`, `/sbin`), criar estruturas de diretórios departamentais aninhadas com `mkdir -p` em `/srv`, associar grupos (`ti-group`, `vendas-group`) a usuários e aplicar permissões octais (`770`/`660`) para isolar o acesso entre departamentos.

## 3. Ambiente

- **Hypervisor:** Oracle VirtualBox
- **Sistema operacional do Host:** Windows
- **ISO do S.O. convidado:** Ubuntu Server 26.04 LTS
- **Usuário administrativo:** `administrador`

## 4. Procedimento

*[PREENCHER após a execução da prática: criação de `/srv/ti-dept` e `/srv/vendas-dept`, criação dos grupos `ti-group`/`vendas-group`, associação de `fulano`/`cicrano`, e aplicação de `chown`/`chmod 770`.]*

## 5. Testes e Validação

*[PREENCHER: saídas dos testes com `su - fulano` (acesso permitido em `ti-dept`) e `su - cicrano` (acesso negado), além do desafio de laboratório com `/srv/diretoria-dept`.]*

## 6. Problemas e Soluções

*[PREENCHER]*

## 7. Conclusão

*[PREENCHER]*
