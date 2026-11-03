# Como colocar o ClinPay no ar (gratuito)

Serviços usados (planos gratuitos, conferidos em novembro de 2026):

- **GitHub**: guarda o código.
- **Banco de dados PostgreSQL**: Neon, Aiven ou Supabase (veja o passo 1).
- **Render**: hospeda o site, com endereço https grátis.
- **cron-job.org** (opcional): "despertador" para os lembretes saírem no horário.

Limites do plano gratuito do Render: o site "dorme" após 15 minutos sem acesso e leva cerca de 1 minuto para acordar. O despertador do passo 5 resolve isso.

## 1. Banco de dados: escolha UMA opção

Todas são PostgreSQL. Para o ClinPay, a única diferença é o valor da variável `DATABASE_URL`, e às vezes do certificado `DATABASE_CA`.

| | **Neon** | **Aiven** | **Supabase** |
|---|---|---|---|
| Espaço grátis | cerca de 0,5 a 1 GB | 1 GB | 500 MB |
| Cartão de crédito | não pede | não pede | não pede |
| Pausa por falta de uso | "dorme" após 5 min e acorda sozinho | pode ser desligado se ficar sem uso por muito tempo | **pausa após 7 dias** sem uso e só volta se você reativar manualmente |
| Certificado próprio | não precisa | **precisa** (`DATABASE_CA`) | **precisa** (`DATABASE_CA`) |
| Backup no plano grátis | limitado | sem backup garantido | sem backup diário |

### Opção A: Neon (mais simples)
1. Crie uma conta em https://neon.com e um projeto `clinpay` na região **AWS US East (N. Virginia)**.
2. Clique em **Connect** e copie a connection string (`postgresql://...`).
3. No Render, use só `DATABASE_URL` com essa string.

### Opção B: Aiven (mais espaço grátis)
1. Crie uma conta em https://aiven.io e clique em **Create service** → **PostgreSQL** → plano **Free**.
2. Escolha uma região nos **EUA** (perto do Render) e crie o serviço. Espere ficar **Running**.
3. Na página do serviço, copie a **Service URI** (`postgres://avnadmin:...`). Ela vai na variável `DATABASE_URL`.
4. Na mesma página, em **CA certificate**, clique em **Show** ou **Download** e copie o texto inteiro, de `-----BEGIN CERTIFICATE-----` até `-----END CERTIFICATE-----`. Ele vai na variável `DATABASE_CA`.

### Opção C: Supabase
1. Crie uma conta em https://supabase.com e um projeto `clinpay`. Anote a senha do banco que ele pedir e escolha a região **East US**.
2. Clique em **Connect** e copie a string do **Session pooler** (porta 5432), **não** a "Direct connection", que usa um tipo de endereço (IPv6) que o Render não alcança. Troque `[YOUR-PASSWORD]` pela senha do banco. Essa string vai em `DATABASE_URL`.
3. Em **Project Settings** → **Database** → **SSL Configuration**, baixe o certificado (**Download certificate**), abra-o no Bloco de Notas e copie o texto inteiro. Ele vai em `DATABASE_CA`.
4. Atenção: o projeto **pausa após 7 dias sem uso**. O despertador do passo 5 ajuda, mas confira o painel do Supabase de vez em quando.

Em todas as opções, o ClinPay cria a tabela sozinho ao ligar pela primeira vez.

### Opção D: Railway (site e banco no mesmo lugar, pago após o teste)
O Railway hospeda o site **e** o banco. Ele não "dorme", mas depois do período de teste custa a partir de cerca de US$ 5/mês. Se usar o Railway, pule o Render (passo 3) e siga a seção **"Railway"** no fim deste guia.

## 2. Código no GitHub

1. Crie uma conta em https://github.com.
2. Clique em **New repository** e dê o nome `clinpay`. Marque **Private** (importante) e clique em **Create repository**.
3. Na página do repositório, clique em **uploading an existing file**.
4. Abra a pasta `C:\clinpay`, selecione tudo **menos** a pasta `dados` e arraste para o navegador. Se aparecer uma pasta `node_modules`, ela também fica de fora.
5. Clique em **Commit changes**.

## 3. Site no Render

1. Crie uma conta em https://render.com, entrando com a sua conta do GitHub.
2. Clique em **New +** → **Web Service** e escolha o repositório `clinpay`.
3. Preencha:
   - **Region**: Virginia (US East)
   - **Runtime**: Node
   - **Build Command**: `npm install`
   - **Start Command**: `node server.js`
   - **Instance Type**: **Free**
4. Em **Environment Variables**, adicione:

| Nome | Valor |
|---|---|
| `DATABASE_URL` | a connection string do banco (passo 1) |
| `DATABASE_CA` | só para Aiven ou Supabase: o texto do certificado (passo 1) |
| `COOKIE_SECURE` | `1` |
| `TRUST_PROXY` | `1` |
| `NODE_VERSION` | `22` |
| `CRON_SECRET` | uma senha longa inventada por você (mínimo 16 caracteres, só letras e números) |
| `PUBLIC_URL` | deixe para depois (passo 4) |
| `ANTHROPIC_API_KEY` | opcional: a chave da IA |

5. Em **Advanced** → **Health Check Path**, coloque `/api/health`.
6. Clique em **Create Web Service** e aguarde de 2 a 5 minutos. Quando aparecer **Live**, o site está no ar.

## 4. Endereço público

1. No topo da página do serviço aparece o endereço, por exemplo `https://clinpay-abcd.onrender.com`.
2. Vá em **Environment**, adicione `PUBLIC_URL` com esse endereço (sem barra no final) e salve. O Render reinicia sozinho.
3. Abra o endereço: vai aparecer **Primeiro acesso**. Crie a sua conta de administrador ClinPay.

## 5. Despertador dos lembretes (recomendado)

Sem isso, os lembretes de e-mail e WhatsApp só saem quando alguém está usando o site.

1. Crie uma conta gratuita em https://cron-job.org.
2. Clique em **Create cronjob**.
3. Em **URL**, coloque: `https://SEU-ENDERECO.onrender.com/api/cron?key=SUA-CRON_SECRET`
4. Escolha **a cada 10 minutos** e salve.

Isso também mantém o site acordado. Um site acordado o mês todo usa cerca de 744 das 750 horas gratuitas do Render: dá para **um** site no plano gratuito.

## Atualizar o site depois

No GitHub, envie os arquivos novos (Add file → Upload files) e confirme. O Render publica sozinho em poucos minutos. Os dados ficam no Neon e não são afetados.

## Antes de usar com pacientes de verdade

O plano gratuito serve para **demonstração e piloto**. Para clínicas reais, com dados de saúde:

- **Backups:** confira no painel do Neon por quantos dias o plano permite restaurar dados. Faça também cópias periódicas.
- **Hospedagem:** passe o Render para um plano pago (sem "dormir") e o Neon para um plano com mais histórico de restauração.
- **LGPD:** os dados ficam em servidores nos EUA, o que é uma transferência internacional de dados. Formalize isso no contrato com as clínicas.
- **Espaço:** fotos e radiografias ocupam espaço rápido no banco gratuito.
- **Um servidor por vez:** o ClinPay foi feito para rodar em uma única instância. Não aumente o número de instâncias no Render.

## Railway (no lugar do Render)

1. Crie uma conta em https://railway.com, entrando com o GitHub.
2. Clique em **New Project** → **Deploy from GitHub repo** e escolha o repositório `clinpay`.
3. No mesmo projeto, clique em **+ Create** (ou **New**) → **Database** → **PostgreSQL**.
4. Abra o serviço do site (`clinpay`) → **Variables** e adicione:

| Nome | Valor |
|---|---|
| `DATABASE_URL` | `${{Postgres.DATABASE_URL}}` (digite exatamente assim: o Railway preenche sozinho com o endereço interno do banco) |
| `COOKIE_SECURE` | `1` |
| `TRUST_PROXY` | `1` |
| `CRON_SECRET` | uma senha longa inventada por você |
| `ANTHROPIC_API_KEY` | opcional: a chave da IA |

5. Em **Settings** → **Networking**, clique em **Generate Domain**. Vai aparecer um endereço como `clinpay-production.up.railway.app`.
6. Volte em **Variables** e adicione `PUBLIC_URL` com `https://` + esse endereço.
7. Em **Settings** → **Deploy**, coloque `/api/health` em **Healthcheck Path**.
8. Abra o endereço e crie o administrador ClinPay.

Não aumente o número de réplicas: o ClinPay foi feito para rodar em uma única instância. O despertador do passo 5 continua útil para os lembretes saírem no horário.
