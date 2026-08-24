# Sinatra for Google Analytics 4 (Server-Side)

Template oficial do **server-side GTM** para integrar seus eventos do GA4 com o [**Sinatra**](https://www.sinatra.pro) — plataforma de data activation.

> Não é um webhook genérico. O endpoint é fixo (`integrations.sinatra.pro`) e os eventos chegam direto na sua workspace do Sinatra, autenticados pelo Account ID + Token gerados na plataforma.

Procurando a versão **web/client-side**? Ela vive em [`sinatrapro/sinatra-for-ga4-client`](https://github.com/sinatrapro/sinatra-for-ga4-client).

---

## Como funciona

A tag roda no server-side GTM container e lê o request HTTP **original** que o browser enviou (via `getRequestQueryParameters()` + `getRequestBody()`), antes do GA4 client parsear/normalizar. Forwarda o conjunto de params verbatim como **GET** para o Sinatra. Sem reconstrução, sem mapeamento, sem perda de campos — chega exatamente o mesmo wire format que o GA4 receberia.

```
Browser → sGTM container → GA4 client processa
                         → Sinatra tag replica (verbatim) → integrations.sinatra.pro
                         → GA4 tag encaminha para Google
```

O template é **autossuficiente**: todo o código vive dentro do próprio `template.tpl` (incluindo testes na seção `___TESTS___`), sem dependências externas.

## Quando usar esta versão

Use a tag **server-side** se o cliente já tem um **server-side GTM container** rodando, ou precisa capturar 100% dos eventos sem depender de interceptação no browser — a versão web perde hits quando há `server_container_url` first-party configurado.

| | Web (client-side) | Server-Side (este repo) |
|---|---|---|
| **Requisito** | Qualquer container web GTM | Server-side GTM container |
| **Cobertura com sGTM ativo** | Limitada | Total |
| **Instalação** | Simples, funciona em qualquer setup | Requer sGTM configurado com GA4 client |
| **Custo extra** | Nenhum | Hosting do server container |

## Configuração

| Campo | Obrigatório | Descrição |
|---|---|---|
| **Account ID** | Sim | Identificador da workspace no Sinatra |
| **Token** | Sim | Token de autenticação |
| **GA4 Measurement ID** | Não | ID da propriedade GA4 (G-XXXXXXXX) |
| **Request timeout (ms)** | Não | Timeout da requisição (padrão: 5000ms) |
| **Respeitar Google Consent Mode** | Não (default: **ligado**) | Descarta eventos com `x-ga-gcs` indicando `analytics_storage=denied`. Vem marcado por padrão; só desmarque se o consentimento é tratado em outra camada |
| **Campos a excluir** | Não | Lista de campos a remover do wire format (query string **e** body) antes de enviar — data minimization. Aceita wildcard `*` no final. Ex: `uafvl, uaa, ep.user_id, ecid, sst.*` |

## Instalação

Guia completo, passo a passo, com troubleshooting e setup de múltiplos ambientes: [`docs/INSTALL-server.md`](docs/INSTALL-server.md)

Resumo rápido:

1. No container **server-side** do GTM, vá em **Templates → Tag Templates → New**
2. Menu `⋮` → **Import** → selecione o `template.tpl` deste repositório
3. Salve, crie a tag com Account ID + Token e use o trigger **All Events**

## LGPD e privacidade

Os mesmos dados que o GA4 coleta são também encaminhados pra sua workspace no Sinatra. O consent gate (**Respeitar Google Consent Mode**) vem **ligado por padrão**: hits com `x-ga-gcs` indicando `analytics_storage=denied` são descartados antes de chegar ao Sinatra.

📄 Documentação completa de privacidade (catalogação de campos, bases legais, direitos do titular): [`docs/PRIVACY.md` no repo client](https://github.com/sinatrapro/sinatra-for-ga4-client/blob/main/docs/PRIVACY.md)

---

**Desenvolvido por [Sinatra](https://www.sinatra.pro)**
