# Instalação — Sinatra for GA4 (Server-Side)

> Guia de instalação da tag server-side (`template.tpl` deste repositório) num container sGTM. Para a versão web/client-side, veja [`INSTALL-web.md` no repo client](https://github.com/sinatrapro/sinatra-for-ga4-client/blob/main/docs/INSTALL-web.md).

**Última atualização:** 2026-08-24

---

## Quando usar esta versão

Use a tag **server-side** se o cliente já tem um **server-side GTM container** rodando, ou precisa capturar 100% dos eventos sem depender de interceptação no browser (a versão web perde hits quando há `server_container_url` first-party configurado — ver [limitações conhecidas da versão web](https://github.com/sinatrapro/sinatra-for-ga4-client/blob/main/docs/INSTALL-web.md#limitações-conhecidas)).

---

## Pré-requisitos

- Um **server-side GTM container** já provisionado (Cloud Run, App Engine, etc.) e apontado no site (`server_container_url` no config do GA4)
- O **GA4 client** configurado nesse container — a tag depende dele ter processado o request antes de disparar
- Acesso de **Publish** ou **Edit** no container server-side
- **Account ID** e **Token** gerados na plataforma Sinatra
- O arquivo `template.tpl` deste repositório

---

## Como funciona

A tag roda **depois** do GA4 client no fluxo do sGTM e lê o request HTTP **original** recebido (via `getRequestQueryParameters()` + `getRequestBody()`), antes de qualquer parsing/normalização do GTM. Ela espelha o mesmo método (GET ou POST) e o mesmo body que o browser enviou — sem reconstrução, sem mapeamento, sem perda de campos.

```
Browser → sGTM container → GA4 client processa o request
                         → Sinatra tag replica (verbatim) → integrations.sinatra.pro
                         → GA4 tag encaminha pro Google
```

---

## Passo a passo

### 1. Importar o template

1. No container **server-side** do GTM, vá em **Templates** → **Tag Templates** → **New**
2. Menu `⋮` → **Import** → selecione `template.tpl`
3. **Save**

### 2. Criar a tag

1. **Tags** → **New**
2. Tipo: **Sinatra for Google Analytics 4 (Server-Side)**
3. Preencha:

| Campo | Obrigatório | Valor |
|---|---|---|
| **Account ID** | Sim | Workspace do cliente no Sinatra |
| **Token** | Sim | Token gerado na plataforma |
| **GA4 Measurement ID** | Não | ID da propriedade (G-XXXXXXXX) |
| **Request timeout (ms)** | Não | Padrão 5000ms — ajuste se o endpoint Sinatra estiver lento |
| **Respeitar Google Consent Mode** | Não | Vem **ligado por padrão** — mantenha assim salvo instrução em contrário |

### 3. Configurar o trigger

Use **All Events** (ou o trigger customizado que já captura todos os eventos GA4 que chegam no container, caso o setup do cliente use algo mais granular).

A tag internamente já filtra: só forwarda requests com `tid=G-*` (query param). Eventos não-GA4 que passarem pelo mesmo trigger são ignorados sem erro.

### 4. Testar

1. Ative **Preview Mode** no container server-side
2. Dispare eventos no site (client) apontando pro ambiente de preview do sGTM
3. No preview do GTM, confirme que a tag **Sinatra for Google Analytics 4 (Server-Side)** disparou com status de sucesso
4. Confirme na dashboard do Sinatra que os eventos chegaram — método deve espelhar o original (GET para hits simples, POST para hits com body, ex: `add_to_cart`, `purchase`)
5. Valide que headers como `Content-Type` foram preservados corretamente em requests POST

### 5. Publicar

Publish normal do container server-side.

---

## Rodando em múltiplos ambientes

O mesmo padrão de [Lookup Table por `{{Environment Name}}`](https://github.com/sinatrapro/sinatra-for-ga4-client/blob/main/docs/INSTALL-web.md#rodando-em-múltiplos-ambientes-stagingteste-antes-de-produção) descrito na versão web se aplica aqui — troque os campos **Account ID**/**Token** por variáveis condicionadas ao ambiente do sGTM, e use **Admin → Environments** no container server-side pra isolar teste de produção.

---

## Consent Mode

O gate de consentimento lê `x-ga-gcs` (equivalente ao `gcs` da versão web) direto do request original. Formato `G1XX`, onde o índice 3 indica `analytics_storage` (`0` = denied, `1` = granted). Com **Respeitar Google Consent Mode** ligado (padrão), hits com `analytics_storage=denied` são descartados **antes** de chegar ao Sinatra — mesmo comportamento da versão web, aplicado no server.

---

## Segurança

- O `token` vai na query string do request forwardado (GET ou POST) — o canal sGTM → Sinatra deve ser HTTPS (padrão)
- Prefira tokens rotacionáveis e de escopo curto
- Considere rate limit + IP allowlist no backend do Sinatra para o tráfego vindo do container sGTM

---

## Troubleshooting

| Sintoma | Causa provável | Fix |
|---|---|---|
| Tag não dispara | GA4 client não está configurado no container, ou request não tem `tid=G-*` | Confirmar client GA4 ativo e testar com evento real do GA4 |
| `accountId e token são obrigatórios` no log | Campos vazios na tag | Preencher Account ID / Token |
| Erro HTTP no `gtmOnFailure` | Timeout ou endpoint indisponível | Aumentar **Request timeout (ms)**, verificar status do endpoint Sinatra |
| Eventos com `analytics_storage=denied` não chegam | Comportamento esperado — consent gate ativo | Se intencional, ok. Se não, desmarcar "Respeitar Google Consent Mode" (só se já há base legal própria) |
| Body de POST (`purchase`, `add_to_cart`) não bate com o original | `Content-Type` não preservado | Confirmar que o header `content-type` está acessível via `read_request` permission (já configurado no template) |

---

**Desenvolvido por [Sinatra](https://www.sinatra.pro)**
