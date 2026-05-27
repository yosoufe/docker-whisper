[English](README.md) | [简体中文](README-zh.md) | [繁體中文](README-zh-Hant.md) | [Русский](README-ru.md)

# Whisper — распознавание речи на Docker

[![Статус сборки](https://github.com/hwdsl2/docker-whisper/actions/workflows/main.yml/badge.svg)](https://github.com/hwdsl2/docker-whisper/actions/workflows/main.yml) &nbsp;[![Docker Pulls](https://raw.githubusercontent.com/hwdsl2/badges/main/img/docker-pulls-whisper-server.svg)](https://hub.docker.com/r/hwdsl2/whisper-server) &nbsp;[![License: MIT](docs/images/license.svg)](https://opensource.org/licenses/MIT) &nbsp;[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://vpnsetup.net/whisper-notebook)

Часть [Docker AI Stack](https://github.com/hwdsl2/docker-ai-stack/blob/main/README-ru.md) — разверните полный самостоятельно размещённый AI-стек одной командой.

Docker-образ для запуска сервера распознавания речи [Whisper](https://github.com/openai/whisper) на базе [faster-whisper](https://github.com/SYSTRAN/faster-whisper). Предоставляет совместимые с OpenAI API для транскрибирования и перевода аудио. Основан на Debian (python:3.12-slim). Простой, приватный, для самостоятельного развёртывания.

**Возможности:**

- Совместимые с OpenAI эндпоинты `POST /v1/audio/transcriptions` и `POST /v1/audio/translations` — любое приложение, использующее OpenAI Whisper API, переключается с изменением одной строки
- Поддержка всех моделей Whisper: `tiny`, `base`, `small`, `medium`, `large-v3`, `large-v3-turbo` и других
- Диаризация говорящих — определение, кто говорит в каждом сегменте (опционально, через [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx))
- Управление моделями через вспомогательный скрипт (`whisper_manage`)
- Аудиоданные остаются на вашем сервере — никакие данные не отправляются третьим сторонам
- Поддержка всех популярных аудиоформатов (mp3, m4a, wav, webm, ogg, flac и всех форматов ffmpeg)
- Несколько форматов ответа: JSON, простой текст, подробный JSON, субтитры SRT, субтитры WebVTT
- Потоковое транскрибирование — добавьте `stream=true`, чтобы получать сегменты через SSE по мере декодирования, без ожидания обработки всего файла
- Ускорение на GPU NVIDIA (CUDA) для более быстрого инференса (тег образа `:cuda`)
- Офлайн-режим — работа без доступа к интернету с предварительно кэшированными моделями (`WHISPER_LOCAL_ONLY`)
- Автоматически собирается и публикуется через [GitHub Actions](https://github.com/hwdsl2/docker-whisper/actions/workflows/main.yml)
- Постоянный кэш моделей через Docker-том
- Поддержка нескольких архитектур: `linux/amd64`, `linux/arm64`

**Также доступно:**

- Попробовать онлайн: [Открыть в Colab](https://vpnsetup.net/whisper-notebook) — Docker и установка не требуются
- ИИ/Аудио: [WhisperLive (STT в реальном времени)](https://github.com/hwdsl2/docker-whisper-live/blob/main/README-ru.md), [Kokoro (TTS)](https://github.com/hwdsl2/docker-kokoro/blob/main/README-ru.md), [Embeddings](https://github.com/hwdsl2/docker-embeddings/blob/main/README-ru.md), [LiteLLM](https://github.com/hwdsl2/docker-litellm/blob/main/README-ru.md), [Ollama (LLM)](https://github.com/hwdsl2/docker-ollama/blob/main/README-ru.md), [Docling](https://github.com/hwdsl2/docker-docling/blob/main/README-ru.md)
- VPN: [WireGuard](https://github.com/hwdsl2/docker-wireguard/blob/main/README-ru.md), [OpenVPN](https://github.com/hwdsl2/docker-openvpn/blob/main/README-ru.md), [IPsec VPN](https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-ru.md), [Headscale](https://github.com/hwdsl2/docker-headscale/blob/main/README-ru.md)
- Инструменты: [MCP Gateway](https://github.com/hwdsl2/docker-mcp-gateway/blob/main/README-ru.md)

**Совет:** Whisper, Kokoro, Embeddings, LiteLLM, Ollama, Docling и MCP-шлюз можно [использовать совместно](#использование-с-другими-ai-сервисами) для построения полного self-hosted AI-стека на собственном сервере.

## Сообщество

- [Подписаться на обновления проектов](https://selfhostedstack.beehiiv.com/subscribe?utm_campaign=ai) (1–2 письма в месяц) — получить бесплатные руководства по развёртыванию AI и VPN (PDF, на английском)
- Присоединяйтесь к сообществу [r/selfhostedstack](https://www.reddit.com/r/selfhostedstack/) для обсуждений и демонстрации проектов

## Whisper или WhisperLive?

| | **docker-whisper** | [docker-whisper-live](https://github.com/hwdsl2/docker-whisper-live/blob/main/README-ru.md) |
|---|---|---|
| **Назначение** | Транскрибирование готовых аудиофайлов | Живой микрофон / потоковое аудио в реальном времени |
| **Протокол** | HTTP REST | WebSocket (потоковый) + HTTP REST |
| **Задержка** | Ответ после обработки всего файла | Почти мгновенно, слово за словом |
| **Подходит для** | Записи совещаний, загруженные аудиофайлы | Захват в браузере, RTSP-потоки, живые субтитры |
| **Размер образа** | ~190 МБ (~3,1 ГБ для `:cuda`) | ~750 МБ (~4,5 ГБ для `:cuda`) |

## Быстрый старт

Запустите сервер Whisper следующей командой:

```bash
docker run \
    --name whisper \
    --restart=always \
    -v whisper-data:/var/lib/whisper \
    -p 9000:9000 \
    -d hwdsl2/whisper-server
```

<details>
<summary><strong>Быстрый старт с GPU (NVIDIA CUDA)</strong></summary>

Если у вас есть GPU NVIDIA, используйте образ `:cuda` для аппаратного ускорения инференса:

```bash
docker run \
    --name whisper \
    --restart=always \
    --gpus=all \
    -v whisper-data:/var/lib/whisper \
    -p 9000:9000 \
    -d hwdsl2/whisper-server:cuda
```

**Требования:** GPU NVIDIA, [драйвер NVIDIA](https://www.nvidia.com/en-us/drivers/) 535+, установленный на хосте [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). Образ `:cuda` поддерживает только `linux/amd64`.

</details>

**Важно:** Для работы образа с моделью `base` по умолчанию требуется не менее 700 МБ свободной оперативной памяти. Системы с 512 МБ ОЗУ и менее не поддерживаются.

**Примечание:** Для развёртываний, доступных из интернета, **настоятельно рекомендуется** добавить HTTPS с помощью [обратного прокси](#использование-обратного-прокси). В этом случае также замените `-p 9000:9000` на `-p 127.0.0.1:9000:9000` в команде `docker run` выше, чтобы исключить прямой доступ к незашифрованному порту извне. Установите `WHISPER_API_KEY` в файле `env`, когда сервер доступен из публичного интернета.

При первом запуске модель Whisper `base` (~145 МБ) автоматически загружается и кэшируется. Проверьте логи, чтобы убедиться в готовности сервера:

```bash
docker logs whisper
```

После появления сообщения "Whisper speech-to-text server is ready" можно начинать транскрибирование:

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@audio.mp3 \
    -F model=whisper-1
```

**Ответ:**
```json
{"text": "Здесь отображается распознанный текст."}
```

**Совет:** Нужен образец аудиофайла для тестирования? Можно использовать этот образец английской речи (WAV, лицензия MIT) из репозитория [Azure Samples](https://github.com/Azure-Samples/cognitive-services-speech-sdk):

```bash
curl -L -o sample_speech.wav \
    "https://github.com/Azure-Samples/cognitive-services-speech-sdk/raw/master/sampledata/audiofiles/katiesteve.wav"

curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@sample_speech.wav \
    -F model=whisper-1
```

В качестве альтернативы вы можете [настроить Whisper без Docker](https://github.com/hwdsl2/whisper-install/blob/main/README-ru.md). Чтобы узнать больше об использовании этого образа, ознакомьтесь с разделами ниже.

## Требования

- Сервер Linux (локальный или облачный) с установленным Docker
- Поддерживаемые архитектуры: `amd64` (x86_64), `arm64` (например, Raspberry Pi 4/5, AWS Graviton)
- Минимум оперативной памяти: ~700 МБ для модели `base` по умолчанию (см. [таблицу моделей](#переключение-моделей))
- Доступ в интернет при первом запуске для загрузки модели (затем модель кэшируется локально). Не требуется при использовании `WHISPER_LOCAL_ONLY=true` с предварительно кэшированными моделями.

**Для ускорения на GPU (образ `:cuda`):**

- GPU NVIDIA с поддержкой CUDA (Compute Capability 6.0+)
- [Драйвер NVIDIA](https://www.nvidia.com/en-us/drivers/) версии 535 или новее на хосте
- Установленный [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
- Образ `:cuda` поддерживает только `linux/amd64`

Для развёртывания с выходом в интернет см. раздел [Использование обратного прокси](#использование-обратного-прокси) для включения HTTPS.

## Скачать

Получите доверенную сборку из [Docker Hub](https://hub.docker.com/r/hwdsl2/whisper-server/):

```bash
docker pull hwdsl2/whisper-server
```

Для ускорения на GPU NVIDIA загрузите тег `:cuda`:

```bash
docker pull hwdsl2/whisper-server:cuda
```

Либо скачайте из [Quay.io](https://quay.io/repository/hwdsl2/whisper-server):

```bash
docker pull quay.io/hwdsl2/whisper-server
docker image tag quay.io/hwdsl2/whisper-server hwdsl2/whisper-server
```

Поддерживаемые платформы: `linux/amd64` и `linux/arm64`. Тег `:cuda` поддерживает только `linux/amd64`.

## Переменные окружения

Все переменные являются необязательными. Задайте `WHISPER_API_KEY` для включения аутентификации Bearer Token.

Этот Docker-образ использует следующие переменные, которые можно объявить в файле `env` (см. [пример](whisper.env.example)):

| Переменная | Описание | По умолчанию |
|---|---|---|
| `WHISPER_MODEL` | Модель Whisper для использования. См. [таблицу моделей](#переключение-моделей). | `base` |
| `WHISPER_LANGUAGE` | Язык транскрибирования по умолчанию. Код BCP-47 (напр. `ru`, `en`, `zh`) или `auto` для автоопределения. | `auto` |
| `WHISPER_PORT` | HTTP-порт для API (1–65535). | `9000` |
| `WHISPER_DEVICE` | Устройство вычислений: `cpu`, `cuda` или `auto`. Используйте `cuda` с образом `:cuda` для ускорения на GPU. `auto` определяет GPU автоматически, при отсутствии — переключается на CPU. | `cpu` |
| `WHISPER_COMPUTE_TYPE` | Тип квантования / вычислений. Для CPU рекомендуется `int8`; для CUDA рекомендуется `float16`. | `int8` (CPU) / `float16` (CUDA) |
| `WHISPER_THREADS` | Количество потоков CPU для инференса. Установите значение, равное числу физических ядер, для минимальной задержки. | `2` |
| `WHISPER_API_KEY` | Необязательный Bearer-токен. Если задан, все запросы должны содержать `Authorization: Bearer <key>`. | *(не задан)* |
| `WHISPER_LOG_LEVEL` | Уровень логирования: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. | `INFO` |
| `WHISPER_BEAM` | Ширина луча при декодировании транскрипции. Большие значения могут улучшить точность за счёт скорости. Используйте `1` для наиболее быстрого (жадного) декодирования. | `5` |
| `WHISPER_LOCAL_ONLY` | Если задано любое непустое значение (например, `true`), отключает все загрузки моделей с HuggingFace. Для изолированных или офлайн-развёртываний с предварительно кэшированными моделями. | *(не задан)* |
| `WHISPER_WORD_TIMESTAMPS` | При значении `true` глобально включает пословные метки времени. Вывод `verbose_json` будет содержать массив `words` на верхнем уровне с временем начала/конца и вероятностью для каждого слова. Также можно включить через `timestamp_granularities[]=word`. | *(не задан)* |
| `WHISPER_DIARIZATION` | При значении `true` включает диаризацию говорящих. Использует [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) с моделями pyannote segmentation-3.0 в формате ONNX (~45 МБ, загружаются автоматически при первом использовании). Не поддерживается в потоковом режиме. | *(не задан)* |
| `WHISPER_DIARIZE_NUM_SPEAKERS` | Точное количество говорящих (если известно). Повышает точность кластеризации. Установите `-1` или оставьте пустым для автоопределения. | `-1` |
| `WHISPER_DIARIZE_MAX_SPEAKERS` | Максимальное количество говорящих. Используется только когда `NUM_SPEAKERS` не задан. | `-1` |
| `WHISPER_DIARIZE_THRESHOLD` | Порог кластеризации. Меньше = больше говорящих, больше = меньше. | `0.5` |

**Примечание:** В файле `env` значения можно заключать в одинарные кавычки, например `VAR='value'`. Не добавляйте пробелы вокруг `=`. Если вы изменяете `WHISPER_PORT`, обновите флаг `-p` в команде `docker run` соответственно.

Пример использования файла `env`:

```bash
cp whisper.env.example whisper.env
# Отредактируйте whisper.env, затем:
docker run \
    --name whisper \
    --restart=always \
    -v whisper-data:/var/lib/whisper \
    -v ./whisper.env:/whisper.env:ro \
    -p 9000:9000 \
    -d hwdsl2/whisper-server
```

Файл `env` монтируется в контейнер, изменения применяются при каждом перезапуске без пересоздания контейнера.

<details>
<summary>Либо передайте его через <code>--env-file</code></summary>

```bash
docker run \
    --name whisper \
    --restart=always \
    -v whisper-data:/var/lib/whisper \
    -p 9000:9000 \
    --env-file=whisper.env \
    -d hwdsl2/whisper-server
```

</details>

## Использование docker-compose

```bash
cp whisper.env.example whisper.env
# Отредактируйте whisper.env при необходимости, затем:
docker compose up -d
docker logs whisper
```

Пример `docker-compose.yml` (уже включён в проект):

```yaml
services:
  whisper:
    image: hwdsl2/whisper-server
    container_name: whisper
    restart: always
    ports:
      - "9000:9000/tcp"  # Для хостового обратного прокси измените на "127.0.0.1:9000:9000/tcp"
    volumes:
      - whisper-data:/var/lib/whisper
      - ./whisper.env:/whisper.env:ro

volumes:
  whisper-data:
    name: whisper-data
```

**Примечание:** Для развёртывания с выходом в интернет настоятельно рекомендуется использовать [обратный прокси](#использование-обратного-прокси) для добавления HTTPS. В этом случае также измените `"9000:9000/tcp"` на `"127.0.0.1:9000:9000/tcp"` в `docker-compose.yml`, чтобы предотвратить прямой доступ к незашифрованному порту. Установите `WHISPER_API_KEY` в файле `env`, когда сервер доступен из публичного интернета.

<details>
<summary><strong>Использование docker-compose с GPU (NVIDIA CUDA)</strong></summary>

Для развёртывания с GPU используется отдельный файл `docker-compose.cuda.yml`:

```bash
cp whisper.env.example whisper.env
# Отредактируйте whisper.env при необходимости, затем:
docker compose -f docker-compose.cuda.yml up -d
docker logs whisper
```

Пример `docker-compose.cuda.yml` (уже включён в проект):

```yaml
services:
  whisper:
    image: hwdsl2/whisper-server:cuda
    container_name: whisper
    restart: always
    ports:
      - "9000:9000/tcp"  # Для хостового обратного прокси измените на "127.0.0.1:9000:9000/tcp"
    volumes:
      - whisper-data:/var/lib/whisper
      - ./whisper.env:/whisper.env:ro
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

volumes:
  whisper-data:
    name: whisper-data
```

</details>

## Справочник по API

API полностью совместим с [эндпоинтом транскрибирования](https://developers.openai.com/api/reference/resources/audio/subresources/transcriptions/methods/create) и [эндпоинтом перевода](https://developers.openai.com/api/reference/resources/audio/subresources/translations/methods/create) OpenAI. Любое приложение, использующее `https://api.openai.com/v1/audio/transcriptions`, может переключиться на собственный хостинг, установив:

```
OPENAI_BASE_URL=http://IP_вашего_сервера:9000
```

### Транскрибирование аудио

```
POST /v1/audio/transcriptions
Content-Type: multipart/form-data
```

**Параметры:**

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `file` | файл | ✅ | Аудиофайл. Поддерживаемые форматы: `mp3`, `mp4`, `m4a`, `wav`, `webm`, `ogg`, `flac` и все форматы, поддерживаемые ffmpeg. |
| `model` | строка | ✅ | Передайте `whisper-1` (значение принимается, но всегда используется активная модель). |
| `language` | строка | — | Код языка BCP-47. Переопределяет `WHISPER_LANGUAGE` для данного запроса. |
| `prompt` | строка | — | Необязательный текст для управления стилем модели или продолжения предыдущего сегмента. |
| `response_format` | строка | — | Формат вывода. По умолчанию: `json`. См. [форматы ответа](#форматы-ответа). Игнорируется при `stream=true`. |
| `temperature` | число с плавающей точкой | — | Температура сэмплирования (0–1). По умолчанию: `0`. |
| `stream` | логическое | — | Включить потоковую передачу SSE. При значении `true` сегменты возвращаются как события `text/event-stream` по мере декодирования. По умолчанию: `false`. |
| `timestamp_granularities[]` | массив | — | Гранулярность меток времени. Значения: `word`, `segment`. При наличии `word` вывод `verbose_json` содержит массив `words` на верхнем уровне. По умолчанию: `["segment"]`. |

**Пример:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@meeting.m4a \
    -F model=whisper-1 \
    -F language=ru
```

С аутентификацией по API-ключу:

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -H "Authorization: Bearer your_api_key" \
    -F file=@audio.mp3 \
    -F model=whisper-1
```

### Форматы ответа

| `response_format` | Описание |
|---|---|
| `json` | `{"text": "..."}` — по умолчанию, соответствует базовому ответу OpenAI |
| `text` | Простой текст без обёртки JSON |
| `verbose_json` | Полный JSON с языком, длительностью, временны́ми метками сегментов и логарифмическими вероятностями |
| `srt` | Формат субтитров SubRip (`.srt`) |
| `vtt` | Формат субтитров WebVTT (`.vtt`) |

**Пример — потоковое получение сегментов по мере декодирования:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@long-audio.mp3 \
    -F model=whisper-1 \
    -F stream=true
```

**SSE-ответ** (используется [протокол потоковой транскрипции OpenAI](https://developers.openai.com/api/docs/guides/speech-to-text#streaming)):

```
data: {"type":"transcript.text.delta","delta":"Привет, как дела?"}

data: {"type":"transcript.text.delta","delta":" Всё хорошо, спасибо."}

data: {"type":"transcript.text.done","text":"Привет, как дела? Всё хорошо, спасибо."}

data: [DONE]
```

Первый инкрементальный текст обычно приходит через 1–3 секунды после загрузки. Каждое событие `transcript.text.delta` содержит инкрементальный текст только что декодированного сегмента. Финальное событие `transcript.text.done` содержит полный собранный текст транскрипции — аналог стандартного ответа `json`.

<details>
<summary><strong>Пример — потоковая передача через браузерный <code>fetch</code></strong></summary>

```javascript
const form = new FormData();
form.append("file", audioBlob, "audio.webm");
form.append("model", "whisper-1");
form.append("stream", "true");

const res = await fetch("http://IP_вашего_сервера:9000/v1/audio/transcriptions", {
  method: "POST", body: form,
});

const reader = res.body.getReader();
const decoder = new TextDecoder();
let buffer = "";

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  buffer += decoder.decode(value, { stream: true });
  // SSE frames are separated by "\n\n"; split and process complete frames
  const frames = buffer.split("\n\n");
  buffer = frames.pop(); // keep any incomplete trailing frame
  for (const frame of frames) {
    if (!frame.startsWith("data: ")) continue;
    const payload = frame.slice(6);
    if (payload.startsWith("[DONE]")) break;
    const event = JSON.parse(payload);
    if (event.type === "transcript.text.delta") console.log(event.delta);
    if (event.type === "transcript.text.done") console.log("Full text:", event.text);
  }
}
```

</details>

**Пример — получение субтитров SRT:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@video.mp4 \
    -F model=whisper-1 \
    -F response_format=srt
```

**Пример — подробный JSON с временны́ми метками:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@audio.mp3 \
    -F model=whisper-1 \
    -F response_format=verbose_json
```

**Пример — подробный JSON с пословными метками времени:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/transcriptions \
    -F file=@audio.mp3 \
    -F model=whisper-1 \
    -F response_format=verbose_json \
    -F "timestamp_granularities[]=word"
```

При наличии `word` в `timestamp_granularities[]` ответ `verbose_json` содержит массив `words` на верхнем уровне:

```json
{
  "word": "hello",
  "start": 0.5,
  "end": 0.8,
  "probability": 0.98
}
```

### Перевод аудио

```
POST /v1/audio/translations
Content-Type: multipart/form-data
```

Перевод аудио с любого языка на английский текст. Совместим с [эндпоинтом перевода OpenAI](https://developers.openai.com/api/reference/resources/audio/subresources/translations/methods/create). Принимает те же параметры, что и эндпоинт транскрибирования. Вывод всегда на английском.

> **Примечание:** Модели только для английского (`.en`) не поддерживают перевод. Используйте многоязычную модель (например, `base`, `small`, `large-v3-turbo`).

**Пример:**

```bash
curl http://IP_вашего_сервера:9000/v1/audio/translations \
    -F file=@french_audio.mp3 \
    -F model=whisper-1
```

### Список моделей

```
GET /v1/models
```

Возвращает активную модель в совместимом с OpenAI формате.

```bash
curl http://IP_вашего_сервера:9000/v1/models
```

### Интерактивная документация API

Интерактивный Swagger UI доступен по адресу:

```
http://IP_вашего_сервера:9000/docs
```

## Постоянные данные

Все данные сервера хранятся в Docker-томе (`/var/lib/whisper` внутри контейнера):

```
/var/lib/whisper/
├── models--Systran--faster-whisper-*/   # Кэшированные файлы модели Whisper (скачаны с HuggingFace)
├── .port                 # Активный порт (используется whisper_manage)
├── .model                # Активное название модели (используется whisper_manage)
└── .server_addr          # Кэшированный IP сервера (используется whisper_manage)
```

Создайте резервную копию Docker-тома для сохранения скачанных моделей. Модели занимают значительный объём (145 МБ – 3 ГБ) и могут скачиваться несколько минут при первом запуске; сохранение тома позволяет избежать повторной загрузки при пересоздании контейнера.

**Совет:** Том `/var/lib/whisper` использует тот же формат кэша HuggingFace, что и том `/var/lib/whisper-live` в `docker-whisper-live`. Если вы уже скачали модель через `docker-whisper-live`, можно примонтировать тот же каталог тома, чтобы не загружать её повторно.

## Управление сервером

Используйте `whisper_manage` внутри запущенного контейнера для просмотра информации о сервере и управления им.

**Показать информацию о сервере:**

```bash
docker exec whisper whisper_manage --showinfo
```

**Список доступных моделей:**

```bash
docker exec whisper whisper_manage --listmodels
```

**Предварительная загрузка модели:**

```bash
docker exec whisper whisper_manage --downloadmodel large-v3-turbo
```

## Переключение моделей

Для смены активной модели:

1. *(Необязательно, но рекомендуется)* Предварительно загрузите новую модель, пока сервер работает:
   ```bash
   docker exec whisper whisper_manage --downloadmodel large-v3-turbo
   ```

2. Обновите `WHISPER_MODEL` в файле `whisper.env` (или добавьте `-e WHISPER_MODEL=large-v3-turbo` в команду `docker run`).

3. Перезапустите контейнер:
   ```bash
   docker restart whisper
   ```

**Доступные модели:**

| Модель | Диск | ОЗУ (примерно) | Примечания |
|---|---|---|---|
| `tiny` | ~75 МБ | ~250 МБ | Самая быстрая; низкая точность |
| `tiny.en` | ~75 МБ | ~250 МБ | Только английский |
| `base` | ~145 МБ | ~700 МБ | Хороший баланс — **по умолчанию** |
| `base.en` | ~145 МБ | ~700 МБ | Только английский |
| `small` | ~465 МБ | ~1,5 ГБ | Повышенная точность |
| `small.en` | ~465 МБ | ~1,5 ГБ | Только английский |
| `medium` | ~1,5 ГБ | ~5 ГБ | Высокая точность |
| `medium.en` | ~1,5 ГБ | ~5 ГБ | Только английский |
| `large-v1` | ~3 ГБ | ~10 ГБ | Старая большая модель |
| `large-v2` | ~3 ГБ | ~10 ГБ | Очень высокая точность |
| `large-v3` | ~3 ГБ | ~10 ГБ | Наивысшая точность |
| `large-v3-turbo` | ~1,6 ГБ | ~6 ГБ | Быстрая + высокая точность ⭐ |
| `turbo` | ~1,6 ГБ | ~6 ГБ | Псевдоним для `large-v3-turbo` |

> **Совет:** `large-v3-turbo` обеспечивает точность, близкую к `large-v3`, при вдвое меньшем потреблении ресурсов. Для большинства производственных развёртываний это рекомендуемый вариант обновления с `base`.

Данные по памяти являются приблизительными и учитывают квантование INT8 (по умолчанию). Модели кэшируются в Docker-томе `/var/lib/whisper` и загружаются только один раз.

## Использование обратного прокси

Для развёртывания с выходом в интернет разместите обратный прокси перед Whisper для обработки HTTPS-терминации. Сервер работает без HTTPS в локальной или доверенной сети, но HTTPS рекомендуется при открытом доступе к API-эндпоинту из интернета.

Используйте один из следующих адресов для доступа к контейнеру Whisper из обратного прокси:

- **`whisper:9000`** — если ваш обратный прокси работает как контейнер в **той же Docker-сети**, что и Whisper (например, определён в том же `docker-compose.yml`).
- **`127.0.0.1:9000`** — если ваш обратный прокси работает **на хосте** и порт `9000` опубликован (по умолчанию `docker-compose.yml` публикует его).

**Пример с [Caddy](https://caddyserver.com/docs/) ([Docker-образ](https://hub.docker.com/_/caddy))** (автоматический TLS через Let's Encrypt, обратный прокси в той же Docker-сети):

`Caddyfile`:
```
whisper.example.com {
  reverse_proxy whisper:9000
}
```

**Пример с nginx** (обратный прокси на хосте):

```nginx
server {
    listen 443 ssl;
    server_name whisper.example.com;

    ssl_certificate     /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # Аудиофайлы могут быть большими — увеличьте лимит загрузки при необходимости
    client_max_body_size 100M;

    location / {
        proxy_pass         http://127.0.0.1:9000;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;       # требуется для потоковой передачи (SSE)
        proxy_read_timeout 300s;
    }
}
```

Установите `WHISPER_API_KEY` в файле `env`, если сервер доступен из публичного интернета.

## Обновление Docker-образа

Для обновления Docker-образа и контейнера сначала [скачайте](#скачать) последнюю версию:

```bash
docker pull hwdsl2/whisper-server
```

Если образ уже актуален, вы увидите:

```
Status: Image is up to date for hwdsl2/whisper-server:latest
```

В противном случае будет скачана последняя версия. Удалите и пересоздайте контейнер:

```bash
docker rm -f whisper
# Затем повторно выполните команду docker run из раздела "Быстрый старт" с теми же томом и портом.
```

Скачанные модели сохранятся в томе `whisper-data`.

## Использование с другими AI-сервисами

Образы Whisper (STT), Embeddings, LiteLLM, Kokoro (TTS), Ollama (LLM), Docling и MCP-шлюз можно объединить для создания полного self-hosted AI-стека на собственном сервере — от голосового ввода/вывода до RAG-поиска с ответами. Whisper, Kokoro и Embeddings работают полностью локально. Ollama выполняет весь инференс LLM локально, данные не отправляются третьим сторонам. Если вы настроите LiteLLM с внешними провайдерами (например, OpenAI, Anthropic), ваши данные будут переданы этим провайдерам для обработки.

| Сервис | Назначение | Порт по умолчанию |
|---|---|---|
| **[Whisper (STT)](https://github.com/hwdsl2/docker-whisper/blob/main/README-ru.md)** | Транскрибирование готовых аудиофайлов через REST API | `9000` |
| **[Embeddings](https://github.com/hwdsl2/docker-embeddings/blob/main/README-ru.md)** | Преобразует текст в векторы для семантического поиска и RAG | `8000` |
| **[LiteLLM](https://github.com/hwdsl2/docker-litellm/blob/main/README-ru.md)** | AI-шлюз — маршрутизирует запросы к OpenAI, Anthropic, Ollama и 100+ другим провайдерам | `4000` |
| **[Kokoro (TTS)](https://github.com/hwdsl2/docker-kokoro/blob/main/README-ru.md)** | Синтезирует естественно звучащую речь из текста | `8880` |
| **[Ollama (LLM)](https://github.com/hwdsl2/docker-ollama/blob/main/README-ru.md)** | Запускает локальные LLM-модели (llama3, qwen, mistral и др.) | `11434` |
| **[MCP-шлюз](https://github.com/hwdsl2/docker-mcp-gateway/blob/main/README-ru.md)** | Предоставляет сервисы ИИ как MCP-инструменты для ИИ-ассистентов (Claude, Cursor и др.) | `3000` |
| **[Docling](https://github.com/hwdsl2/docker-docling/blob/main/README-ru.md)** | Конвертирует документы (PDF, DOCX и др.) в структурированный текст/Markdown | `5001` |

**См. также: [Docker AI Stack](https://github.com/hwdsl2/docker-ai-stack)** — разверните полный стек одной командой, с готовыми конфигурациями и примерами конвейеров.

## Диаризация говорящих

Диаризация определяет, *кто говорит* в каждом транскрибированном сегменте. Работает на базе [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) с использованием модели pyannote segmentation-3.0, экспортированной в формат ONNX.

**Включение диаризации:**

```bash
# В whisper.env:
WHISPER_DIARIZATION=true
```

ONNX-модели (~45 МБ суммарно) автоматически загружаются при первом использовании и кэшируются в томе `/var/lib/whisper`. Предварительная загрузка:

```bash
docker exec whisper whisper_manage --downloaddiarize
```

**Вывод с включённой диаризацией:**

`verbose_json` добавляет поле `speaker` в каждый сегмент:

```json
{
  "segments": [
    {"id": 0, "start": 1.0, "end": 3.5, "text": "Запускаем на следующей неделе.", "speaker": "SPEAKER_00"},
    {"id": 1, "start": 4.0, "end": 6.2, "text": "Думаю, QA нужно ещё два дня.", "speaker": "SPEAKER_01"}
  ]
}
```

`srt` и `vtt` добавляют метку говорящего:

```
1
00:00:01,000 --> 00:00:03,500
[SPEAKER_00] Запускаем на следующей неделе.

2
00:00:04,000 --> 00:00:06,200
[SPEAKER_01] Думаю, QA нужно ещё два дня.
```

Формат `text` показывает метку при смене говорящего:

```
[SPEAKER_00] Запускаем на следующей неделе.
[SPEAKER_01] Думаю, QA нужно ещё два дня.
```

**Примечания:**
- Диаризация требует анализа полного аудио и **не поддерживается в потоковом режиме** (`stream=true`). Если оба включены, диаризация пропускается.
- Установите `WHISPER_DIARIZE_NUM_SPEAKERS`, если известно точное количество говорящих, для повышения точности.
- Конвейер диаризации запускается после транскрибирования, добавляя небольшое время обработки, пропорциональное длительности аудио.

## Технические подробности

- Базовый образ: `python:3.12-slim` (Debian) для `:latest`; `nvidia/cuda:12.9.1-cudnn-runtime-ubuntu24.04` для `:cuda`
- Среда выполнения: Python 3 (виртуальное окружение в `/opt/venv`)
- STT-движок: [faster-whisper](https://github.com/SYSTRAN/faster-whisper) + CTranslate2 (INT8 по умолчанию на CPU, FP16 на CUDA)
- API-фреймворк: [FastAPI](https://fastapi.tiangolo.com/) + [Uvicorn](https://www.uvicorn.org/)
- Декодирование аудио: [PyAV](https://github.com/PyAV-Org/PyAV) (встроенные библиотеки FFmpeg)
- Директория данных: `/var/lib/whisper` (Docker-том)
- Хранение моделей: формат HuggingFace Hub внутри тома — скачивается один раз, переиспользуется при перезапусках

## Лицензия

**Примечание:** Программные компоненты внутри готового образа (такие как faster-whisper и его зависимости) распространяются под лицензиями, выбранными соответствующими правообладателями. При использовании готового образа пользователь несёт ответственность за соблюдение всех соответствующих лицензий на программное обеспечение, содержащееся в образе.

Copyright (C) 2026 Lin Song   
Данная работа распространяется под [лицензией MIT](https://opensource.org/licenses/MIT).

**faster-whisper** является собственностью SYSTRAN и распространяется под [лицензией MIT](https://github.com/SYSTRAN/faster-whisper/blob/master/LICENSE).

Данный проект представляет собой независимую Docker-обёртку для Whisper и не аффилирован с OpenAI или SYSTRAN, не одобрен и не спонсирован ими.