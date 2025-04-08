# **Исследование методов разрешения противоречий между источниками данных в RAG**

Данный репозиторий содержит экспериментальные ноутбуки, посвящённые **дообучению**, **оценке** и **разрешению противоречий** в Retrieval-Augmented Generation (**RAG**).

## **Структура репозитория**

📂 `fine-tuning/` – файлы для дообучения моделей:

- `roberta_fine_tuning.ipynb` – дообучение **RoBERTa** для проверки фактов.
- `flan_t5_fine_tuning.ipynb` – дообучение **FLAN-T5** для генерации ответов.

📂 `evaluation/` – файлы для оценки моделей:

- `baseline_evaluation.ipynb` – анализ уверенности моделей до дообучения.
- `fine_tuned_evaluation.ipynb` – оценка качества моделей после дообучения.

📂 `contradiction_resolution/` – разрешение противоречий:

- `RAG_Ambiguity_Resolution.ipynb` – основной эксперимент по ранжированию, анализу доверия к источникам и выявлению противоречий.

---

## **1. Концепция экспериментов**

### **1.1. Fine-tuning (дообучение)**

**🔹 RoBERTa (фактчекинг)**

- Классификация утверждений: `SUPPORTS`, `REFUTES`, `NOT ENOUGH INFO`.
- Использован датасет **FEVER**.

**🔹 FLAN-T5 (генерация ответов)**

- Ответ на вопрос с учётом контекста.
- Использованы **SQuAD v2** и **FEVER**.

### **1.2. Evaluation (оценка моделей)**

**🔹 До дообучения:** метрики качества, анализ логитов, визуализация уверенности.

**🔹 После дообучения:** сравнение метрик, анализ галлюцинаций и калибровки уверенности.

---

## **2. Разрешение противоречий в RAG**

В ноутбуке `RAG_Ambiguity_Resolution.ipynb` реализована система, включающая:

### **2.1. Контекстуальное ранжирование**

- Ранжирование сниппетов по формуле: `0.6 * семантика + 0.25 * доверие + 0.15 * актуальность`.
- Используется sentence-transformer + cosine similarity + оценка домена.

### **2.2. Байесовская модель доверия**

- Для каждого домена храним `(α, β)` как параметры бета-распределения.
- Обновляем после каждого эпизода:
  - Успех → увеличиваем α
  - Ошибка → увеличиваем β
- Получаем `trust score = α / (α + β)`.

### **2.3. Механизм согласия между источниками**

- Отдельно проверяем каждый сниппет с помощью RoBERTa.
- Сравниваем метки: если есть и `SUPPORTS`, и `REFUTES` → `has_contradiction = True`.
- В случае противоречия доверие к источникам снижается.

### **2.4. Автоматическая генерация утверждений**

- Утверждение формируется из ответа:
  - `"The answer to the question '...' is ..."`
- Проверка согласованности утверждения с контекстом.

### **2.5. Визуализация результатов**

- Матрицы ошибок, boxplot по категориям, распределения по логитам.
- Анализ доверия к доменам и противоречий в топ-3 источниках.

---

## **3. Используемый стек**

- `roberta-large-mnli`, `google/flan-t5-base`, `sentence-transformers`.
- Датасеты: **FEVER**, **SQuAD v2**.
- `Google Custom Search API` для извлечения сниппетов.

---

## **4. Запуск экспериментов**

### **Дообучение моделей:**

```python
!python fine-tuning/roberta_fine_tuning.ipynb
!python fine-tuning/flan_t5_fine_tuning.ipynb
```

### **Оценка:**

```python
!python evaluation/baseline_evaluation.ipynb
!python evaluation/fine_tuned_evaluation.ipynb
```

### **Разрешение противоречий:**

```python
!python contradiction_resolution/RAG_Ambiguity_Resolution.ipynb
```

---

## **5. Выводы и перспективы**

✅ Улучшено разрешение неоднозначностей в retrieved context.<br>
✅ Снижено влияние недостоверных источников.<br>
✅ Повышена интерпретируемость за счёт визуализации доверия и голосов источников.<br>

📌 Перспективы:

- Поддержка пользовательской обратной связи.
- Улучшение reranking моделей и учёт метаинформации.
- Объединение retrieved и verified данных в единый граф знаний.
