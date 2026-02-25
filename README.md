# YouTube Transcript API

API REST simples que busca transcrições de vídeos do YouTube. Os transcripts são salvos automaticamente em arquivos `.txt` na pasta `transcripts/`.

---

## Como Rodar

### 1. Instalar dependências

```bash
pip install -r requirements.txt
```

### 2. Configurar o `.env`

Crie um arquivo `.env` na raiz do projeto:

```env
PROXY_AUTH_TOKEN="seu-token-aqui"
UPSTREAM_PROXY=
```

| Variável | Obrigatória | Descrição |
|----------|-------------|-----------|
| `PROXY_AUTH_TOKEN` | Sim | Token de autenticação (enviado no header `X-Proxy-Token`) |
| `UPSTREAM_PROXY` | Não | URL de proxy upstream (ex: `http://user:pass@host:port`) |

### 3. Iniciar o servidor

#### Linux / Mac

```bash
export PROXY_AUTH_TOKEN="seu-token-aqui"
uvicorn app:app --host 0.0.0.0 --port 8000
```

#### Windows (PowerShell)

```powershell
$env:PROXY_AUTH_TOKEN="seu-token-aqui"
uvicorn app:app --host 0.0.0.0 --port 8000
```

#### Docker

```bash
docker build -t youtube-transcript-api .
docker run -p 8000:8000 --env-file .env youtube-transcript-api
```

### 4. Testar

```powershell
# PowerShell
Invoke-RestMethod http://localhost:8000/health
```

```bash
# Linux / Mac
curl http://localhost:8000/health
```

Resposta esperada: `{"status":"ok"}`

---

## Endpoints

| Method | Path | Descrição |
|--------|------|-----------|
| GET | `/health` | Health check (sem token) |
| GET | `/transcript/{video}/text` | Transcrição como texto simples |
| GET | `/transcript/{video}` | Transcrição com timestamps |
| GET | `/transcripts/{video}` | Listar idiomas disponíveis |

- `{video}` aceita um **video ID** (`dQw4w9WgXcQ`) ou uma **URL completa**
- Todos os endpoints (exceto `/health`) exigem o header `X-Proxy-Token`
- O parâmetro `?lang=` define o idioma (padrão: `en`)

---

## Exemplos de Uso

### PowerShell (Windows)

```powershell
$headers = @{ "X-Proxy-Token" = $env:PROXY_AUTH_TOKEN }

# Texto simples
Invoke-RestMethod -Uri "http://localhost:8000/transcript/dQw4w9WgXcQ/text" -Headers $headers

# Com timestamps
Invoke-RestMethod -Uri "http://localhost:8000/transcript/dQw4w9WgXcQ" -Headers $headers

# Listar idiomas
Invoke-RestMethod -Uri "http://localhost:8000/transcripts/dQw4w9WgXcQ" -Headers $headers

# Em português
Invoke-RestMethod -Uri "http://localhost:8000/transcript/dQw4w9WgXcQ?lang=pt" -Headers $headers
```

### curl (Linux / Mac)

```bash
# Texto simples
curl -H "X-Proxy-Token: $PROXY_AUTH_TOKEN" "http://localhost:8000/transcript/dQw4w9WgXcQ/text"

# Com timestamps
curl -H "X-Proxy-Token: $PROXY_AUTH_TOKEN" "http://localhost:8000/transcript/dQw4w9WgXcQ"

# Listar idiomas
curl -H "X-Proxy-Token: $PROXY_AUTH_TOKEN" "http://localhost:8000/transcripts/dQw4w9WgXcQ"

# Em português
curl -H "X-Proxy-Token: $PROXY_AUTH_TOKEN" "http://localhost:8000/transcript/dQw4w9WgXcQ?lang=pt"
```

---

## Respostas

### `/transcript/{video}/text`

```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English",
  "language_code": "en",
  "text": "We're no strangers to love you know the rules and so do I..."
}
```

### `/transcript/{video}`

```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English",
  "language_code": "en",
  "is_generated": true,
  "segments": [
    {"text": "We're no strangers to love", "start": 18.0, "duration": 3.5}
  ]
}
```

---

## Salvamento Automático

Toda transcrição buscada é salva automaticamente como `.txt` na pasta `transcripts/`:

```
transcripts/
├── dQw4w9WgXcQ_en.txt
├── dQw4w9WgXcQ_pt.txt
└── RX-fQTW2To8_en.txt
```

O nome do arquivo segue o formato `{video_id}_{idioma}.txt`.

---

## Deploy com Cloudflare Tunnel

```yaml
- hostname: yt-transcript.yourdomain.com
  service: http://youtube-transcript-api:8000
```

---

## Claude Skill

O repositório inclui uma skill para o Claude (`skill/`) que busca e trabalha com transcrições de forma eficiente.

### Como Funciona

1. **Fetch** — Baixa a transcrição completa, salva em `/home/claude/youtube/<video_id>.json`, retorna apenas metadados
2. **Trabalho local** — Claude usa `grep`, `head`, `cat` no arquivo local ao invés de carregar tudo no contexto
3. **Search** — Busca em todas as transcrições salvas

### Setup da Skill

1. Deploy do serviço em um IP residencial
2. Expor via Cloudflare Tunnel
3. Editar `skill/scripts/youtube.py` com seu domínio e token
4. Zipar a pasta `skill/` e renomear para `.skill`
5. Importar no Claude via Settings → Skills

### Comandos da Skill

```bash
python3 youtube.py fetch <video_url_ou_id> [lang]   # Buscar transcrição
python3 youtube.py fetch <video> --force             # Re-buscar mesmo se cacheado
python3 youtube.py search "pattern"                  # Buscar em transcrições
python3 youtube.py list                              # Listar cache
python3 youtube.py get <video_id>                    # Carregar do cache
python3 youtube.py text <video> [max_chars]          # Texto do cache
python3 youtube.py clear                             # Limpar cache
```

###
example: curl.exe -H "X-Proxy-Token: a7c9f1d34e8b2a6f90d5c3e81b7f4a2d6c0e9a53d1f8b2c47e5a0d9f3c6b1a84" "http://localhost:8000/transcript/ZrBvPoFBVUo/text"

## License

MIT
