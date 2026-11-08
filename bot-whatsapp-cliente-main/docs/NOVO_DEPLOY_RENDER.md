# Instalar no Render e recuperar os dados

Repositório: https://github.com/ruancoa1-hash/Rota--Manda-HAHA-

## 1. Criar o serviço pelo Blueprint

1. Entre na conta do Render que vai receber o bot.
2. Abra https://render.com/deploy?repo=https://github.com/ruancoa1-hash/Rota--Manda-HAHA-.
3. Se necessário, conecte o GitHub `ruancoa1-hash` ao Render e libere o acesso ao repositório.
4. Selecione a branch `main` e o arquivo **`render.yaml` da raiz**.
5. Confira o serviço `rota-manda-haha`: Docker, 1 CPU / 2 GB e disco de 5 GB montado em `/data`.
6. Preencha `PANEL_USERS` e `PANEL_ADMIN_EMAILS` conforme a próxima seção.
7. Revise o valor do plano e do disco exibido pelo Render e confirme a criação.
8. Aguarde **Live**. Abra a URL do serviço e faça login.

O plano é uma configuração inicial, não uma garantia de capacidade para todos os
clientes. Para muitas análises simultâneas, aumente CPU e RAM e depois ajuste os
limites de OCR. O disco mantém os dados entre reinícios; o plano Free não permite
adicionar esse disco.

## 2. Configurar o acesso de administrador

Em **Environment**, use seu e-mail e uma senha escolhida por você:

```text
PANEL_USERS=seuemail@exemplo.com:SUA_SENHA:admin
PANEL_ADMIN_EMAILS=seuemail@exemplo.com
```

As linhas acima são exemplos. Não use vírgula nem `:` dentro da senha.
O exemplo configura um administrador; novos clientes podem ser cadastrados pelo
painel. Para recriar clientes que eram configurados pelo ambiente no serviço
anterior, copie também as entradas correspondentes de `PANEL_USERS`.

O Blueprint gera `PANEL_SESSION_SECRET`. Mantenha esse valor estável durante os
deploys. Na instalação manual, gere-o em um computador com Node.js:

```sh
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Salve o resultado em `PANEL_SESSION_SECRET` no Render.

## 3. Alternativa: criar um Web Service manualmente

Se não usar o Blueprint, clique em **New > Web Service**, conecte o repositório e
configure:

| Campo | Valor |
| --- | --- |
| Repository | `ruancoa1-hash/Rota--Manda-HAHA-` |
| Branch | `main` |
| Language / Runtime | `Docker` |
| Root Directory | vazio (raiz do repositório) |
| Dockerfile Path | `./Dockerfile` |
| Docker Build Context | `.` |
| Health Check Path | `/api/ping` |
| Auto-Deploy | `On Commit` |
| Instance | plano pago com memória suficiente para os clientes |
| Persistent Disk mount path | `/data` |
| Persistent Disk size | `5 GB` |

O Dockerfile já instala Node.js 24 e Tesseract e define o build e o comando de
inicialização. Não configure comandos de um Static Site ou um serviço Node
nativo nesse fluxo.

Se o serviço já usa `bot-whatsapp-cliente-main` como Root Directory, o Dockerfile
nessa pasta também funciona com caminho `./Dockerfile` e contexto `.`. Use o
Dockerfile correspondente ao contexto escolhido.

Se aparecer `failed to read dockerfile: open Dockerfile: no such file or directory`,
confira **Settings > Build & Deploy**: Root Directory vazio, Dockerfile Path
`./Dockerfile` e Docker Build Context `.`. Depois execute **Manual Deploy > Deploy
latest commit**, incluindo a correção que adiciona o Dockerfile da raiz.

Adicione estas variáveis em **Environment**:

```text
NODE_ENV=production
DATA_DIR=/data
PORT=10000
TZ=America/Sao_Paulo
BOT_PERFORMANCE_PROFILE=high
BOT_EAGER_WORKERS=true
OCR_GLOBAL_CONCURRENCY=1
OCR_TURBO_MODE=true
OCR_VARIANT_CONCURRENCY=2
UV_THREADPOOL_SIZE=4
KEEP_ALIVE_WHEN_MONITORING=false
DAILY_SESSION_RESET_ENABLED=false
```

Adicione também `PANEL_USERS`, `PANEL_ADMIN_EMAILS` e `PANEL_SESSION_SECRET` da
seção anterior. Depois crie o serviço.

## 4. Recuperar clientes, preferências e histórico

Publicar o código não transfere os dados do serviço antigo. O sistema novo começa
vazio até você importar um backup.

### Preferência: arquivo portátil

1. No sistema antigo, entre como administrador e abra **Config > Backup portátil**.
2. Exporte um backup completo com uma senha de pelo menos 12 caracteres e guarde o arquivo e a senha.
3. No sistema novo, entre como administrador e abra **Config > Backup portátil**.
4. Selecione o arquivo, informe a mesma senha e visualize os dados encontrados.
5. Confira as quantidades de clientes e rotas antes de aplicar a restauração.
6. Aguarde a conclusão e o reinício. Faça login novamente se o painel solicitar.
7. Confira **Clientes**, as preferências, **Rotas validadas**, **Rotas apagadas** e **Custos**.

Contas que eram definidas em `PANEL_USERS` no serviço antigo devem existir nessa
variável no novo serviço. O backup preserva os registros do painel, mas não copia
as variáveis secretas do Render.

### Se o backup disponível estiver no Cloudflare R2

Primeiro baixe do bucket uma cópia do snapshot antigo que contém os dados. Guarde
também a chave de criptografia original. Somente depois configure no novo Render:

```text
R2_ENDPOINT=https://SEU_ACCOUNT_ID.r2.cloudflarestorage.com
R2_BUCKET=SEU_BUCKET
R2_ACCESS_KEY_ID=SUA_ACCESS_KEY
R2_SECRET_ACCESS_KEY=SUA_SECRET_KEY
R2_BACKUP_ENCRYPTION_KEY=A_MESMA_CHAVE_DO_BACKUP_ANTIGO
```

O endpoint termina em `.r2.cloudflarestorage.com`, sem `/NOME_DO_BUCKET`.
Esses valores são exemplos; use as credenciais e a chave que você guardou.

Com todas as variáveis presentes, o backup automático inicia cerca de 30 segundos
após o boot. Uma instalação vazia pode atualizar `latest.rmbak` com dados vazios;
por isso preserve a cópia antiga antes. No painel **Config > Backup externo**,
escolha explicitamente o snapshot anterior à migração e restaure. Confira o
resultado nas abas do painel; tamanho do arquivo sozinho não comprova conteúdo.

O arquivo R2 e o arquivo portátil usam chaves de abertura diferentes: para o
portátil, use a senha escolhida na exportação; para o R2, mantenha a chave original
`R2_BACKUP_ENCRYPTION_KEY`.

## 5. Conectar o WhatsApp e conferir o funcionamento

Antes de ativar o bot no novo serviço, pare o bot correspondente no serviço
anterior. Evite operar a mesma conta de WhatsApp nos dois serviços durante a
migração.

No novo painel:

1. Abra a conta do cliente e confira se o WhatsApp aparece conectado.
2. Se pedir reconexão, leia o QR Code pelo WhatsApp em **Aparelhos conectados**.
3. Confira o grupo alvo, o nome de envio, os bairros e as condições salvas.
4. Faça um teste controlado em um grupo de teste antes de armar o grupo real.
5. Abra `/api/ping` na URL do serviço para verificar se o servidor responde.

## 6. Atualizações futuras

Os próximos commits devem ser publicados em `main` deste repositório. Com o GitHub
conectado e **Auto-Deploy: On Commit**, o Render aplica as novas versões.

Use **Deploy latest commit** se precisar disparar manualmente. **Deploy a specific
commit** desativa o deploy automático pelo painel do Render; reative-o depois se
quiser receber os próximos commits.

## Referências

- Blueprint e caminhos Docker: https://render.com/docs/blueprint-spec
- Diretório raiz de serviços: https://render.com/docs/monorepo-support
- Disco persistente: https://render.com/docs/disks
- Conectar GitHub e habilitar atualizações: https://render.com/docs/git-provider
- Deploy automático e manual: https://render.com/docs/deploys
