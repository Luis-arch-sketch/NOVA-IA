# NOVA AI

Uma nova geração de inteligência artificial.

Plataforma completa de IA com contas reais, chat persistente, projetos, créditos calculados no servidor e integração oficial com a API da OpenAI.

## Requisitos

- Node.js 18 ou superior
- PostgreSQL 13 ou superior
- Uma chave da OpenAI (ou endpoint compatível)

## Instalação

```bash
# Instale as dependências
npm install

# Copie o arquivo de ambiente
cp .env.example .env
```

## PostgreSQL

Crie o banco e o usuário:

```bash
# Acesse o PostgreSQL
sudo -u postgres psql

# No prompt do psql:
CREATE USER nova WITH PASSWORD 'sua_senha_segura';
CREATE DATABASE nova_ai OWNER nova;
GRANT ALL PRIVILEGES ON DATABASE nova_ai TO nova;
\c nova_ai
GRANT ALL ON SCHEMA public TO nova;
```

Atualize `DATABASE_URL` no arquivo `.env`:

```env
DATABASE_URL=postgres://nova:sua_senha_segura@127.0.0.1:5432/nova_ai
```

O schema (users, session, projects, conversations, messages, credit_transactions, referrals) é criado automaticamente na inicialização.

## Variáveis de ambiente

Edite o `.env`:

```env
OPENAI_API_KEY=sk-sua-chave
OPENAI_MODEL=gpt-4o-mini
OPENAI_BASE_URL=https://api.openai.com/v1
DATABASE_URL=postgres://nova:sua_senha_segura@127.0.0.1:5432/nova_ai
SESSION_SECRET=uma-string-longa-e-aleatoria
PORT=3000
NODE_ENV=development
INITIAL_CREDITS=1000000000
CREDIT_GRANT_INTERVAL_HOURS=24
REFERRAL_REWARD_CREDITS=3000000000
COMMAND_CREDIT_COST=175000
```

Nunca coloque `OPENAI_API_KEY` no frontend. A chave permanece apenas no servidor.

## Como iniciar

```bash
# Desenvolvimento ou produção
npm start
```

A aplicação escuta em `0.0.0.0` e usa `process.env.PORT`.

Abra `http://localhost:3000`.

## OPENAI_API_KEY

1. Crie uma chave em https://platform.openai.com/api-keys
2. Coloque a chave em `OPENAI_API_KEY` no `.env`
3. Opcionalmente altere `OPENAI_MODEL` (exemplo: `gpt-4o-mini`, `gpt-4o`)
4. Reinicie o servidor

Sem essa variável, o cadastro, o login e o restante da plataforma funcionam, mas o chat retorna um erro amigável pedindo a configuração da API.

Para um provedor compatível com a API da OpenAI, defina também `OPENAI_BASE_URL`.

## Sistema de créditos

- Toda conta nova recebe **1.000.000.000** créditos.
- Os créditos **não expiram**.
- Novos créditos só são concedidos quando o saldo está **zero** e o ciclo diário (24h) já passou.
- Se o usuário gastar parte do saldo, o restante permanece no dia seguinte.
- Cada comando enviado à IA custa **175.000** créditos, cobrados no servidor após a resposta.
- O valor é fixo por mensagem (`COMMAND_CREDIT_COST`), independente do tamanho da resposta.
- O desconto usa transação PostgreSQL com `SELECT ... FOR UPDATE` para evitar saldo negativo em requisições simultâneas.
- O frontend nunca define o saldo. O servidor é a única fonte da verdade.

## Indicar e ganhar

No mesmo esquema de produtos como o Lovable:

- Cada usuário recebe um código único e um link `/r/CODIGO`.
- Quando um amigo cria a conta por esse link, o indicador ganha **3.000.000.000** créditos.
- A recompensa é creditada no servidor, em transação PostgreSQL, uma única vez por novo usuário.
- Autoindicação não vale. O saldo extra não expira.

## API

| Método | Rota | Descrição |
| --- | --- | --- |
| POST | `/api/auth/register` | Criar conta |
| POST | `/api/auth/login` | Entrar |
| POST | `/api/auth/logout` | Sair |
| GET | `/api/me` | Usuário autenticado |
| POST | `/api/chat` | Enviar mensagem à IA |
| GET | `/api/conversations` | Listar conversas |
| POST | `/api/conversations` | Criar conversa |
| GET | `/api/conversations/:id` | Abrir conversa |
| DELETE | `/api/conversations/:id` | Excluir conversa |
| GET | `/api/projects` | Listar projetos |
| POST | `/api/projects` | Criar projeto |
| GET | `/api/projects/:id` | Abrir projeto |
| DELETE | `/api/projects/:id` | Excluir projeto |
| GET | `/api/credits` | Saldo |
| GET | `/api/credit-transactions` | Extrato |
| GET | `/api/referrals` | Link, código e convites |
| GET | `/r/:code` | Atalho público do convite |

Sessões usam cookie HTTP-only (`nova.sid`) armazenado no PostgreSQL.

## Deploy (Render)

1. Crie um Web Service apontando para este repositório.
2. Adicione um PostgreSQL e copie a Internal Database URL para `DATABASE_URL`.
3. Defina:
   - `OPENAI_API_KEY`
   - `OPENAI_MODEL`
   - `SESSION_SECRET`
   - `NODE_ENV=production`
4. Build: `npm install`
5. Start: `npm start`

O servidor já escuta em `0.0.0.0` e respeita `process.env.PORT`.

Há também um `render.yaml` e um `Procfile` na raiz.

Em produção, o cookie de sessão é marcado como `secure`. Use HTTPS.

## Estrutura

```text
src/
  index.js
  db/          pool e migração
  middleware/  sessão, auth, erros
  routes/      REST
  services/    IA e créditos
  utils/       validação
public/        interface
```

## Segurança

- Senhas com bcrypt (12 rounds)
- Cookies HTTP-only
- Queries parametrizadas
- Helmet + rate limit
- `OPENAI_API_KEY` apenas no servidor
- Isolamento de conversas e projetos por usuário
- 
