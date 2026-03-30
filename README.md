# Portfolio Access Tracker

## PT-BR

Projeto desenvolvido em **Python** para monitorar e notificar a quantidade de visualizações no meu [portfólio pessoal](https://marllonmendez.vercel.app/pt-br).

O sistema funciona como um microsserviço de análise **ultra-leve e seguro**, contabilizando acessos reais em tempo quase real e enviando **relatórios diários por e-mail**, sem coleta de dados pessoais.

### Tecnologias Utilizadas

- **Linguagem:** Python 3
- **Framework Web:** Flask (com Flask-CORS)
- **Filtro de Bots:** Pacote `user-agents`
- **Servidor de Produção:** Gunicorn (com suporte a threads)
- **Banco de Dados (Cache):** Redis (via [Upstash](https://upstash.com/))
- **Template Engine:** Jinja2 (HTML/CSS Dinâmico)
- **Containerização:** Docker
- **Serviço de E-mail:** [Resend](https://resend.com/)
- **Plataforma de Hospedagem:** [Render](https://render.com/)

### Funcionalidades

- **Contagem Anônima de Visitas**
  - Nenhum IP, cookie, fingerprint ou identificador persistente é armazenado.
  - Apenas contadores numéricos agregados são persistidos no Redis.

- **Filtro Automático de Bots**
  - Identificação de bots e crawlers via análise de User-Agent.
  - Acessos identificados como bots são descartados automaticamente.

- **Proteção contra Abuso**
  - Validação de domínio de origem via parsing de URL (`Origin` / `Referer`).
  - Autenticação opcional via token customizado (`X-Track-Token`).
  - Rate limiting por IP (10 requisições/minuto) para prevenção de flood.
  - Bloqueio explícito de requisições não autorizadas.

- **Relatórios Diários via E-mail**
  - Relatório automático referente ao **dia anterior** (baseado no fuso horário de Brasília - BRT / UTC-3).
  - Total de visitas agregadas.
  - Distribuição de acessos por faixa horária.
  - Template HTML renderizado com Jinja2.
  - Envio realizado via Resend.

- **Gerenciamento de Recursos**
  - TTL automático de 48 horas para chaves no Redis.
  - Projeto otimizado para planos gratuitos.

- **Tolerância a Falhas (Graceful Degradation)**
  - O sistema tolera instabilidades no Redis. Conexões perdidas geram alertas de log silenciosos, enquanto os clientes recebem resposta status `204 No Content` para evitar que o frontend trave.
  - Retorna erro `503` caso o Redis fique ausente ou não seja configurado.

### Visualização do Relatório

Para garantir a melhor experiência visual e facilitar a manutenção do design do e-mail sem a necessidade de disparos reais de SMTP, o projeto conta com um script de visualização local.

Para ver como o relatório está ficando, execute o comando abaixo no seu terminal (dentro da pasta do projeto):

```bash
python .\preview.py
```

### Endpoints

#### `POST /track-visit`

Endpoint responsável por registrar uma visita válida.

Regras:

- Origem deve pertencer ao domínio autorizado.
- Token de rastreamento deve ser válido (quando configurado).
- Bots são ignorados automaticamente.

Headers esperados:

- `Origin` ou `Referer`
- `User-Agent`
- `X-Track-Token` (opcional, se habilitado)

---

#### `POST /send-report`

Endpoint responsável por gerar e enviar o relatório diário.

- Protegido por autenticação via **Bearer Token**.
- Deve ser acionado por um job externo (cron, pipeline, scheduler).

Header obrigatório:

- `Authorization: Bearer <CRON_SECRET>`

### Fluxo de Funcionamento

1. O front-end do portfólio envia um `POST` para `/track-visit`.
2. O serviço valida domínio (via URL parsing), token e User-Agent.
3. Requisições que excedem o rate limit são descartadas silenciosamente.
4. Visitas válidas são agregadas no Redis por data e horário.
5. Um job externo executa um `POST` em `/send-report`.
6. O relatório do dia anterior é gerado e enviado por e-mail.

### Variáveis de Ambiente

| Variável                 | Descrição                                              |
| ------------------------ | ------------------------------------------------------ |
| `RESEND_API_KEY`         | Chave de API do serviço Resend.                        |
| `EMAIL_TO`               | Endereço de e-mail que receberá o relatório.           |
| `RESEND_FROM`            | E-mail remetente utilizado no envio.                   |
| `UPSTASH_REDIS_REST_URL` | URL de conexão com o Redis (Upstash).                  |
| `CRON_SECRET`            | Token Bearer para autenticar o envio do relatório.     |
| `TRACK_TOKEN`            | Token opcional para autenticar o endpoint de tracking. |
| `ALLOWED_DOMAIN`         | Domínio autorizado a registrar visitas.                |
| `PORT`                   | Porta de execução da aplicação (default: 8080).        |

### Segurança

- Validação de domínio de origem via parsing de URL (comparação exata do hostname).
- Configuração estrita de **CORS** limitando requisições APENAS da origem listada no `ALLOWED_DOMAIN`.
- Autenticação por token no tracking e no disparo de relatórios.
- Rate limiting por IP (com leitura do real IP através do header `X-Forwarded-For`) para prevenção de flood e abuso.
- Comparação de tokens em tempo constante (proteção contra timing attacks).
- Nenhum dado sensível do visitante é coletado ou persistido.

### Privacidade e Transparência

Este projeto foi construído com foco absoluto em privacidade.

**Nenhum dado pessoal é coletado, processado ou armazenado de forma permanente.**  
O sistema opera exclusivamente com **contadores agregados**, sem qualquer forma de identificação do usuário final.

Endereços IP são utilizados **exclusivamente para rate limiting**, armazenados em chaves Redis com TTL de 60 segundos e sem associação a qualquer dado de navegação.

Por não lidar com dados pessoais identificáveis, o projeto está alinhado às diretrizes da **LGPD**.

---

## EN

Project developed in **Python** to monitor and notify the number of views on my [personal portfolio](https://marllonmendez.vercel.app/en).

The system works as a **secure ultra-lightweight analytics microservice**, counting real visits and sending **daily email reports**, without collecting personal data.

### Technologies Used

- **Language:** Python 3
- **Web Framework:** Flask (with Flask-CORS)
- **Bot Filtering:** `user-agents` package
- **Production Server:** Gunicorn (with thread support)
- **Database (Cache):** Redis (via [Upstash](https://upstash.com/))
- **Template Engine**: Jinja2 (Dynamic HTML/CSS)
- **Containerization:** Docker
- **Email Service:** [Resend](https://resend.com/)
- **Hosting Platform:** [Render](https://render.com/)

### Features

- **Anonymous Visit Counting**
  - No IPs, cookies, fingerprints, or user identifiers are stored.
  - Only aggregated numeric counters are persisted.

- **Bot Filtering**
  - Automatic bot and crawler detection via User-Agent analysis.
  - Bot traffic is ignored.

- **Abuse Protection**
  - Allowed domain validation via URL parsing.
  - Optional tracking token authentication.
  - IP-based rate limiting (10 requests/minute) to prevent flooding.
  - Explicit request blocking when validation fails.

- **Daily Email Reports**
  - Report generated for the previous day (strictly tracked using Brasilia Timezone - BRT / UTC-3).
  - Total visit count and time-based distribution.
  - HTML email rendered with Jinja2.
  - Delivery handled by Resend.

- **Resource Management**
  - Redis keys expire automatically after 48 hours.

- **Fault Tolerance & Resiliency**
  - Application gracefully degrades when Redis fluctuates. Tracking errors result in silent logs processing a `204 No Content` for clients, ensuring the front-end user experience remains resilient.
  - Generates an early return (HTTP `503`) if the Redis client drops connection on boot.

### Email Preview

To ensure the best visual experience and facilitate email design maintenance without the need for actual SMTP triggers, the project includes a local preview script.

To see what the report looks like, run:

```bash
python .\preview.py
```

### Endpoints

#### `POST /track-visit`

Endpoint responsible for registering a valid visit.

Rules:

- The request origin must belong to an authorized domain.
- The tracking token must be valid (when configured).
- Bots are automatically ignored.

Expected headers:

- `Origin` or `Referer`
- `User-Agent`
- `X-Track-Token` (optional, if enabled)

---

#### `POST /send-report`

Endpoint responsible for generating and sending the daily report.

- Protected by **Bearer Token** authentication.
- Must be triggered by an external job (cron, pipeline, scheduler).

Required header:

- `Authorization: Bearer <CRON_SECRET>`

### Execution Flow

1. The portfolio front-end sends a `POST` request to `/track-visit`.
2. The service validates the domain (via URL parsing), token, and User-Agent.
3. Requests exceeding the rate limit are silently discarded.
4. Valid visits are aggregated in Redis by date and hour.
5. An external job triggers a `POST` request to `/send-report`.
6. The report for the previous day is generated and sent via email.

### Environment Variables

| Variable                 | Description                                       |
| ------------------------ | ------------------------------------------------- |
| `RESEND_API_KEY`         | Resend API key.                                   |
| `EMAIL_TO`               | Email address that will receive the report.       |
| `RESEND_FROM`            | Sender email used by Resend.                      |
| `UPSTASH_REDIS_REST_URL` | Redis connection URL (Upstash).                   |
| `CRON_SECRET`            | Bearer token to trigger the report endpoint.      |
| `TRACK_TOKEN`            | Optional token to authenticate tracking requests. |
| `ALLOWED_DOMAIN`         | Authorized domain for visit tracking.             |
| `PORT`                   | Application port (default: 8080).                 |

### Security

- Origin domain validation via URL parsing (exact hostname comparison).
- Strict **CORS restriction** via `Flask-CORS` allowing only requests from `ALLOWED_DOMAIN`.
- Token-based authentication for both tracking and report triggering.
- IP-based rate limiting with proxy / load balancer support (via `X-Forwarded-For` header) to prevent abuse.
- Constant-time token comparison (timing attack protection).
- No sensitive visitor data is collected or persisted.

### Privacy & Transparency

This project was built with a strict privacy-first approach.

**No personal data is collected, processed, or permanently stored.**  
The system operates exclusively with aggregated counters, without any form of end-user identification.

IP addresses are used **solely for rate limiting**, stored in Redis keys with a 60-second TTL, and are never associated with any browsing data.

By not handling personally identifiable data, the project is inherently compliant with **LGPD** guidelines.
