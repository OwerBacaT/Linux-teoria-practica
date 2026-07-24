# 01 - Permissões e Controle de Acesso em Linux

## Objetivo
Simular um cenário realista de auditoria de permissões de arquivos em um servidor Linux, identificando e corrigindo configurações inseguras — uma das falhas mais comuns encontradas em pentests e auditorias de hardening.

## Ambiente
- WSL com CentOS
- Estrutura simulando uma pasta de aplicação com arquivo de credenciais, script e relatório

## Cenário criado

```bash
mkdir -p ~/projeto_seguranca/app
cd ~/projeto_seguranca/app

echo "DB_USER=admin
DB_PASS=SenhaSuperSecreta123" > config_db.env

echo '#!/bin/bash
echo "Backup rodando..."' > backup.sh

touch relatorio_publico.txt

chmod 777 config_db.env
chmod 644 backup.sh
chmod 600 relatorio_publico.txt
```

## Estado inicial (auditoria)

| Arquivo | Permissão | Problema identificado |
|---|---|---|
| `config_db.env` | `777` (rwxrwxrwx) | 🔴 Crítico — qualquer usuário do sistema podia ler e escrever credenciais de banco de dados |
| `backup.sh` | `644` (rw-r--r--) | 🟡 Faltava bit de execução (`x`) para o dono |
| `relatorio_publico.txt` | `600` (rw-------) | 🟡 Deveria ser público para leitura, mas só o dono tinha acesso |

## Correções aplicadas

```bash
chmod 600 config_db.env      # só o dono lê/escreve — credenciais protegidas
chmod 740 backup.sh          # dono executa, grupo lê/executa, outros nada
chmod 644 relatorio_publico.txt  # dono edita, todos leem
```

## Estado final

```
-rw------- config_db.env
-rwxr----- backup.sh
-rw-r--r-- relatorio_publico.txt
```

## Comandos de auditoria aprendidos

```bash
# Arquivos que qualquer usuário pode ler
find ~/projeto_seguranca -type f -perm -o+r

# Arquivos que outros podem escrever (bandeira vermelha)
find ~/projeto_seguranca -type f -perm -o+w

# Arquivos executáveis por qualquer usuário
find ~/projeto_seguranca -type f -perm -o+x

# Arquivos com permissão total (777)
find ~/projeto_seguranca -type f -perm 777
```

## Principais aprendizados

- Notação simbólica (`rwx`) vs numérica (`chmod 750`, por exemplo) e como converter uma na outra
- Diferença entre `-perm -MODO` (pelo menos essas permissões), `-perm /MODO` (qualquer uma) e `-perm MODO` (exatamente essas)
- `;` executa comandos em sequência independente do resultado; `&&` só executa o próximo se o anterior teve sucesso — importante em scripts de segurança para não mascarar falhas
- Bits de execução desnecessários (`x`) em arquivos que não são scripts "poluem" varreduras de auditoria (ex: CIS Benchmark) e podem mascarar arquivos maliciosos reais

## Próximos passos
- Usuários e grupos (`useradd`, `groupadd`) para controle de acesso baseado em grupo
