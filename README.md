# **Исследование методов разрешения противоречий между источниками данных в RAG**

Данный репозиторий содержит экспериментальные ноутбуки, посвящённые **дообучению** (fine-tuning) и **оценке** (evaluation) моделей для разрешения противоречий между источниками в Retrieval-Augmented Generation (**RAG**).

## **Структура репозитория**

📂 `fine-tuning/` – файлы для дообучения моделей:

- `roberta_fine_tuning.ipynb` – дообучение **RoBERTa** для проверки фактов.
- `flan_t5_fine_tuning.ipynb` – дообучение **FLAN-T5** для генерации ответов.

📂 `evaluation/` – файлы для оценки моделей:

- `baseline_evaluation.ipynb` – анализ уверенности моделей до дообучения.
- `fine_tuned_evaluation.ipynb` – оценка качества моделей после дообучения.

---

## **1. Концепция экспериментов**

### **1.1. Fine-tuning (дообучение)**

Задача: улучшить точность моделей **RoBERTa** и **FLAN-T5**, обучив их на верификации фактов и генерации текстов с учётом retrieved context.

**🔹 RoBERTa (фактчекинг)**

- **Модель**: `roberta-large-mnli`.
- **Датасет**: **FEVER** ([ссылка](https://fever.ai/data.html)), содержит утверждения и подтверждающие/опровергающие факты.
- **Процесс обучения**:
  - Классификация утверждений на `SUPPORTS`, `REFUTES`, `NOT ENOUGH INFO`.
  - Использование **cross-entropy loss**.
  - Оптимизатор **AdamW**, `learning_rate = 2e-5`, `batch_size = 16`, `epochs = 3`.
  - Балансировка классов в датасете для корректного обучения.

**🔹 FLAN-T5 (генерация ответов)**

- **Модель**: `google/flan-t5-base`.
- **Датасет**: **SQuAD v2** ([ссылка](https://rajpurkar.github.io/SQuAD-explorer/)) + **FEVER**.
- **Процесс обучения**:
  - Формирование входных последовательностей в формате `question + retrieved context`.
  - Loss-функция **cross-entropy** для текстовой генерации.
  - Использование **beam search** и **temperature scaling**.
  - Оптимизатор **AdamW**, `learning_rate = 3e-5`, `batch_size = 8`, `epochs = 1`.
  - Оценка модели через **validation loss** и сохранение чекпоинтов.

---

### **1.2. Evaluation (оценка моделей)**

Задача: анализ точности предсказаний, уверенности моделей и влияние retrieved context на генерацию ответов.

**🔹 Baseline Evaluation (до дообучения)**

- Проверка начальных характеристик моделей **T5** и **RoBERTa**.
- Метрики: **accuracy, precision, recall, F1-score**.
- Анализ уверенности модели через **логиты softmax**.
- Визуализация зависимости `confidence vs self-assessment`.

**🔹 Fine-Tuned Evaluation (после дообучения)**

- Анализ изменений точности предсказаний.
- Метрики: **BLEU, ROUGE, BERTScore** для генеративной модели.
- Проверка случаев **галлюцинаций** модели.
- Визуализация изменений в калибровке уверенности после fine-tuning.

---

## **2. Используемый стек**

### **2.1. LLM (Языковые модели)**

- `roberta-large-mnli` для фактчекинга.
- `google/flan-t5-base` для генерации ответов.

### **2.2. Датасеты**

- **FEVER** ([FEVER AI](https://fever.ai/data.html)) – фактчекинг.
- **SQuAD v2** ([SQuAD Explorer](https://rajpurkar.github.io/SQuAD-explorer/)) – вопросно-ответная генерация.

### **2.3. Retrieval-Augmented Generation (RAG)**

Используется **Google Custom Search API** для поиска релевантных фрагментов:

- Получение `top_k` результатов по запросу.
- Извлечение `snippets` из первых `k` документов.
- Конкатенация контекстов перед подачей в модель.

Пример вызова API:

```python
from googleapiclient.discovery import build

def retrieve_context(query, top_k=5):
    service = build("customsearch", "v1", developerKey="YOUR_API_KEY")
    result = service.cse().list(q=query, cx="YOUR_CSE_ID", num=top_k).execute()
    snippets = [item["snippet"] for item in result.get("items", [])]
    return " ".join(snippets)
```

---

## **3. Запуск экспериментов**

### **3.1. Дообучение моделей**

Открыть ноутбук в Google Colab, запустить ячейки с загрузкой датасетов и запуском обучения:

```python
!python fine-tuning/roberta_fine_tuning.ipynb
```

```python
!python fine-tuning/flan_t5_fine_tuning.ipynb
```

### **3.2. Оценка моделей**

Запустить анализ точности и визуализации:

```python
!python evaluation/baseline_evaluation.ipynb
```

```python
!python evaluation/fine_tuned_evaluation.ipynb
```

---

## **4. Выводы и перспективы**

Методы дообучения моделей на специализированных датасетах и использование retrieved context из Google Search API позволяют:
✅ Повысить точность фактчекинга и генерации ответов.
✅ Снизить уровень галлюцинаций в RAG-системе.
✅ Оценить уверенность моделей и её влияние на качество ответов.

**Дальнейшие исследования** включают:

- Оптимизацию retrieved context (например, reranking).
- Улучшение калибровки уверенности моделей.
- Внедрение дополнительных механизмов разрешения противоречий.
