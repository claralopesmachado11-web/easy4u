# Central de Demandas

> Projeto independente, desenvolvido por fora da empresa. Não é uma ferramenta ou sistema oficial da organização.

Sistema web que junta a **Central de Demandas** (lê a caixa compartilhada, separa o que pede ação e acompanha prazos) e o **Leitor de NF** (transforma notas fiscais em linhas da planilha de controle).

- **Frontend:** React + TypeScript (Vite), no visual do Leitor de NF.
- **Backend:** Node.js 22 + Fastify + TypeScript.
- **Banco:** PostgreSQL 16 com regras de acesso por linha (RLS).
- **Integrações:** Microsoft Graph (leitura da caixa), SMTP (e-mails), Azure OpenAI (opcional).

## Rodar no seu computador

Pré-requisitos: Node 22 e PostgreSQL 16.

```bash
npm ci
# crie papéis e banco locais (troque as senhas)
psql "postgres://postgres@localhost/postgres" -v owner_pw=dev_owner -v auth_pw=dev_auth -v runtime_pw=dev_runtime -v worker_pw=dev_worker -v db=central -f server/migrations/bootstrap-roles.sql
DATABASE_URL_OWNER=postgres://app_owner:dev_owner@localhost:5432/central npm run migrate -w server
cp .env.example .env   # ajuste para desenvolvimento: NODE_ENV=development, APP_URL=http://localhost:5173, DB_SSL=off
npm run dev -w server   # API em :3000
npm run dev -w web      # interface em :5173
npm run create-admin -w server -- voce@empresa.com.br
```

Sem SMTP configurado (só em desenvolvimento), os e-mails não são enviados; os links aparecem no próprio sistema ou no terminal.

## Testes

```bash
npm test          # 46 testes de API, permissões, segurança, leitura da caixa e notas (PostgreSQL de verdade)
npm run build     # compila servidor e interface
VITE_DEMO=1 npx vite build   # (dentro de web/) prévia estática com dados de exemplo, sem servidor
```

## Documentos

- `docs/PUBLICACAO.md` — o que pedir à TI, publicação no Azure, backups e monitoramento.
- `docs/RELATORIO.md` — funcionalidades, testes, vulnerabilidades encontradas e pendências.

## Estrutura

```
server/
  migrations/      001 tabelas · 002 permissões e RLS · 003 correção da trava de nível · bootstrap-roles.sql
  src/auth/        senhas (Argon2id), sessões, CSRF, níveis
  src/routes/      auth, admin, demandas, notas e fornecedores
  src/services/    Microsoft Graph, classificador, sincronização, agendador
  test/            testes automáticos
web/
  src/pages/       telas
  src/nf/          leitura de NF (XML, PDF, OCR) no navegador
```
