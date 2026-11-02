# Lizard
On-device LLM runtime for weak hardware. Run compressed AI models offline on phones, Raspberry Pi, browsers and cheap VPS. No cloud. No GPU. No data leaks.


<div align="center">

# 🦎 Lizard

**ИИ, который работает на слабом железе. Офлайн. Приватно.**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0--alpha-green.svg)](https://github.com/lizard-ai/lizard/releases)
[![Status](https://img.shields.io/badge/status-experimental-yellow.svg)](#status)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

[Демо](https://demo.lizard.ai) · [Документация](https://docs.lizard.ai) · [Discord](https://discord.gg/lizard) · [Roadmap](#roadmap)

</div>

---

## Что это

**Lizard** — это рантайм и SDK для запуска языковых моделей прямо на устройстве пользователя: смартфоне, Raspberry Pi, в браузере или на дешёвом VPS. Без облака. Без GPU. Без утечек данных.

Экспериментальная сжатая модель + лёгкий рантайм + единый SDK для всех платформ.

```js
import { Lizard } from '@lizard/sdk';

const ai = await Lizard.init({ model: 'qwen-0.5b', backend: 'auto' });
const reply = await ai.chat('Привет! Как дела?');
console.log(reply.text);
```

---

## Зачем это нужно

Современные LLM работают в облаке. Это создаёт три проблемы:

| Проблема | Как решает Lizard |
|---|---|
| **Приватность** — данные уходят на чужой сервер | Всё работает локально, данные не покидают устройство |
| **Стоимость** — платишь за каждый запрос | Один раз скачал модель — работаешь бесплатно |
| **Зависимость** — нет интернета = нет ИИ | Модель работает офлайн, после первой загрузки |

Lizard не заменяет ChatGPT. Он решает задачи, где облако **неуместно**: медицина, банки, промышленный IoT, устройства без интернета, приложения для параноиков.

---

## Возможности

- 🪶 **Работает офлайн** — модель целиком на устройстве
- 🔒 **Приватно по умолчанию** — нулевые сетевые запросы после загрузки
- ⚡ **Без GPU** — оптимизирован под CPU: ARM, x86, WASM
- 📦 **Один SDK — все платформы** — iOS, Android, Linux, Windows, браузер
- 🧩 **Готовые сценарии** — чат, суммаризация, извлечение, классификация
- 🧪 **Открытые эксперименты** — делимся бенчмарками и методами квантования

---

## Демо

Попробуйте прямо сейчас: **[demo.lizard.ai](https://jazzy-buttercream-8d62ad.netlify.app/)**

Запускает Qwen 0.5B в браузере через WebGPU. Без регистрации. Без сервера.

Что показывает демо:
- Загрузка модели ~350 МБ с прогрессом
- Стриминг ответа в реальном времени
- Метрики: время до первого токена, токены/сек
- Работа офлайн (отключите интернет — модель продолжит отвечать)

---

## Требования

| Платформа | RAM | Бэкенд | Статус |
|---|---|---|---|
| 🌐 Браузер (Chrome 113+) | 2 ГБ | WebGPU / WASM | ✅ работает |
| 📱 iOS 17+ | 2 ГБ | Metal / Core ML | 🚧 в разработке |
| 🤖 Android 10+ | 2 ГБ | Vulkan / CPU | 🚧 в разработке |
| 🐧 Linux ARM64 | 512 МБ | CPU | ✅ работает |
| 🍓 Raspberry Pi 4/5 | 1 ГБ | CPU | ✅ работает |
| 🪟 Windows | 2 ГБ | CPU / DirectX | 🚧 в разработке |

**GPU не требуется.** Но с WebGPU/Metal/Vulkan работает в 5–15 раз быстрее.

---

## Быстрый старт

### В браузере

```bash
npm install @lizard/sdk
```

```javascript
import { Lizard } from '@lizard/sdk';

const ai = await Lizard.init({
  model: 'qwen-0.5b',
  backend: 'auto',
  onProgress: (p) => console.log(`Загрузка: ${p}%`),
});

// Простой запрос
const reply = await ai.chat('Что такое квантование?');
console.log(reply.text);

// Стриминг
const stream = await ai.chatStream('Придумай слоган');
for await (const token of stream) {
  process.stdout.write(token);
}

// Отмена
const controller = new AbortController();
ai.chat('Долгий запрос...', { signal: controller.signal });
controller.abort();
```

### В Node.js (для серверов)

```bash
npm install @lizard/sdk
```

```javascript
import { Lizard } from '@lizard/sdk/node';

const ai = await Lizard.init({ model: 'qwen-0.5b' });
const reply = await ai.chat('Привет');
```

### В Linux-контейнере

```bash
docker run -it --rm lizard/runtime:cpu
```

---

## Доступные модели

| Модель | Размер | RAM | Скорость (CPU) | Качество |
|---|---|---|---|---|
| Qwen 2.5 · 0.5B | 180 МБ | ~1 ГБ | ~15 ток/с | базовая |
| Qwen 2.5 · 1.5B | 950 МБ | ~2.5 ГБ | ~6 ток/с | хорошая |
| Llama 3.2 · 1B | 800 МБ | ~2 ГБ | ~8 ток/с | хорошая |
| **Lizard 180M** (наша) | 180 МБ | ~800 МБ | ~25 ток/с | узкая (извлечение) |

Все модели квантованы в 4 бита (q4). Хостятся на Hugging Face Hub и кэшируются в браузере через Cache API.

---

## Архитектура

```
┌─────────────────────────────────────────────┐
│  Apps: Web, iOS, Android, Desktop, IoT      │
├─────────────────────────────────────────────┤
│  SDK: JS / Swift / Kotlin / Python          │
├─────────────────────────────────────────────┤
│  Runtime: WebGPU / Metal / Vulkan / CPU     │
├─────────────────────────────────────────────┤
│  Model Hub: CDN + HF + квантование          │
└─────────────────────────────────────────────┘
```

**Ключевые решения:**
- **ONNX + WebGPU** для браузера — кроссбраузерность + скорость
- **4-битное квантование** — уменьшает модель в 4 раза без критичной потери качества
- **IndexedDB / Cache API** для кэша — второй запуск в 10 раз быстрее
- **Streaming через TextStreamer** — первый токен за 1–2 сек

Подробнее — в [ARCHITECTURE.md](docs/ARCHITECTURE.md).

---

## Roadmap

- [x] **v0.1** — браузерная демка на Transformers.js
- [ ] **v0.2** — npm-пакет `@lizard/sdk` + документация
- [ ] **v0.3** — Python-пакет + CLI для Linux
- [ ] **v0.4** — iOS (Swift Package) + Android (AAR)
- [ ] **v0.5** — свой CDN моделей + кастомное квантование
- [ ] **v1.0** — стабильный API + SLA + enterprise-фичи

Полный roadmap — в [ROADMAP.md](ROADMAP.md).

---



## Статус

⚠️ **Экспериментальный проект.** API меняется. Не используйте в продакшене до v1.0.

- Версия: `0.1.0-alpha`
- Стабильность API: низкая
- Покрытие тестами: ~40%
- Поддерживаемые платформы: браузер, Linux

---

## Участие

Мы рады любой помощи. Начните с:

- 🐛 [Открыть issue](https://github.com/lizard-ai/lizard/issues/new)
- 💬 [Обсудить в Discord](https://discord.gg/lizard)
- 📖 [CONTRIBUTING.md](CONTRIBUTING.md) — как контрибьютить
- 🎯 [Good first issues](https://github.com/lizard-ai/lizard/labels/good%20first%20issue) — начать легко

Особенно ищем помощь в:
- Тестирование на разных устройствах (Raspberry Pi, старые ноутбуки, мобилки)
- Оптимизация WASM-бэкенда
- Документация на английском
- Примеры использования в реальных проектах

---

## Лицензия

MIT © 2025 Lizard Team

См. [LICENSE](LICENSE).

**Note:** Модели, которые вы загружаете, могут иметь **свои** лицензии. Например, Qwen 2.5 — Apache 2.0, Llama 3.2 — Llama 3.2 Community License. Проверяйте перед коммерческим использованием.

---

<div align="center">

**Сделано с 🦎 и ненавистью к облачным API.**


</div>
