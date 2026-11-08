# Rota Manda HAHA

Bot de rotas do WhatsApp com painel de clientes e administração, leitura de imagens,
preferências por bairro, validação de rotas, financeiro e backups criptografados.

O código do aplicativo fica em `bot-whatsapp-cliente-main/`.
O `Dockerfile` da raiz permite o deploy com o diretório raiz e o contexto Docker
padrões do Render. Também existe um Dockerfile próprio na pasta do aplicativo.

## Subir do zero no Render

[Criar serviço no Render](https://render.com/deploy?repo=https://github.com/ruancoa1-hash/Rota--Manda-HAHA-)

O `render.yaml` da raiz prepara um **serviço pago de 1 CPU / 2 GB**, disco de
**5 GB em `/data`**, Docker e deploy automático da branch `main`. Confira o preço
no Render antes de confirmar a criação. O dimensionamento inicial pode precisar
de ajuste conforme a quantidade de clientes e análises simultâneas.

No formulário, preencha somente as duas variáveis obrigatórias:

| Variável | O que preencher |
| --- | --- |
| `PANEL_USERS` | `seuemail@exemplo.com:SUA_SENHA:admin` |
| `PANEL_ADMIN_EMAILS` | `seuemail@exemplo.com` |

A senha não pode conter vírgula nem dois-pontos, pois eles separam os campos de
`PANEL_USERS`. O Render gera `PANEL_SESSION_SECRET` automaticamente nesse caminho.
Guarde esse valor para migrações futuras.

Quando aparecer **Live**, abra a URL do serviço e entre com o e-mail e a senha
configurados. A nova instalação começa vazia; os clientes, históricos e sessões
anteriores precisam ser restaurados por backup.

**[Passo a passo completo, instalação manual e recuperação de dados](bot-whatsapp-cliente-main/docs/NOVO_DEPLOY_RENDER.md)**

## Rodar localmente

Use Node.js 24. Na pasta `bot-whatsapp-cliente-main`, execute:

```sh
npm ci
npm run build
npm start
```

O arquivo [`.env.example`](bot-whatsapp-cliente-main/.env.example) documenta as
variáveis. No Render, configure-as em **Environment**; localmente, exporte-as no
terminal ou use `node --env-file=.env dist/server.js` após criar seu `.env`.

Os dados de clientes e as credenciais de produção ficam no disco persistente e
nas variáveis do serviço. Eles não fazem parte da publicação do código.
