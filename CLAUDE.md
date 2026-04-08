# BI Dashboards — Lehavot

**Спецификация BI дашбордов для Lehavot**

---

## 🔗 Связь с Todo Hub

| Ресурс | Ссылка |
|--------|--------|
| **Hub проект** | `C:\Users\Yuri Katz\RiderProjects\Work\Todo` |
| **Planner** | https://planner.cloud.microsoft/lehavot.com/Home/PlanViews/S8Lh7Ry61ECnLuZe8USRSZcAD2i2 |
| **Bucket** | `BI Dashboards` — 9 задач |
| **Task folder** | `Todo/tasks/011-bi-dashboards/` |

---

## Файлы

| Файл | Назначение |
|------|------------|
| `SPEC.md` | **Source of truth** — полная спецификация дашбордов |
| `BI.pbix` | **Power BI файл** — рабочий прототип (OData → Priority) |
| `sales.html` | HTML mockup — Sales Dashboard |
| `orders.html` | HTML mockup — Orders Dashboard |
| `orders-by-due-date.html` | HTML mockup — Orders by Due Date Dashboard |
| `index.html` | Landing page |

---

## BI.pbix — текущее состояние (анализ 11/02/2026)

### Источники данных
| Таблица | OData Entity | Фильтр |
|---------|-------------|--------|
| AINVOICES | `lehavot-pri.abra-it.cloud/.../leh1010/AINVOICES` | `STATDES = "סופית"` |
| Orders | `lehavot-pri.abra-it.cloud/.../leh1010/ORDERS` | — |
| DateTable | Auto-generated | — |

### DAX Measures (3)
| Мера | Формула |
|------|---------|
| Total_Sales | `SUM(AINVOICES[TOTPRICE])` |
| Sales_LastYear | `CALCULATE(SUM(AINVOICES[TOTPRICE]), SAMEPERIODLASTYEAR(DateTable[Date]))` |
| Sales_YoY_Percent | `DIVIDE(SUM(...) - [Sales_LastYear], [Sales_LastYear], 0)` |

### Визуалы (1 страница: "מבט כללי")
- Card: סה"כ מכירות (Total_Sales)
- Card: כמות (TOTQUANT)
- Card: רווח הזמנות (QPROFIT)
- Bar Chart: מכירות לפי סוכן (TOTPRICE by AGENTNAME)
- Bar Chart: מכירות לפי לקוח (TOTPRICE by CDES)
- Line Chart: מכירות לאורך זמן (Year/Month)
- Slicer: Date filter

### Что есть vs что нужно по SPEC
| Компонент | Статус |
|-----------|--------|
| AINVOICES (חשבוניות) | ✅ |
| Orders (הזמנות) | ✅ |
| DOCUMENTS (תעודות משלוח) | ❌ |
| AGENTS (отдельная таблица) | ❌ (inline) |
| CUSTOMERS (отдельная) | ❌ (inline) |
| PART / FAMILY | ❌ |
| Budget (Excel) | ❌ |
| Сегментация | ❌ |
| Sales Dashboard | ⚠️ частично |
| Orders Dashboard | ❌ |
| Orders by Due Date | ❌ |

---

## Ключевые участники

| Кто | Роль |
|-----|------|
| **Элад (Elad)** | Заказчик, mockups, требования |
| **Рувен (Reuven)** | Данные Priority, маппинг сегментов |
| **Алик** | Финальное утверждение |

---

## Текущее состояние (Feb 2026)

- SPEC.md активен, основные вопросы закрыты
- Маппинг сегментов: через связь **סוכן (Agent) → לקוח (Customer)**
- Открытый вопрос: есть ли в Priority встроенная связь или нужна таблица маппинга
- MTD Budget убран (решение Элада 09/02)
- Детали: `Todo/tasks/011-bi-dashboards/ACTION_PLAN.md`
- **הצעת מחיר от Алик (Элиэзер Гольдберг):** 28,000 ₪ модель + 1,000 ₪/мес поддержка. Бעלות файлов — להבות
- **BI.pbix** — есть рабочий прототип с OData подключением к Priority

---

## Правила

1. **SPEC.md = source of truth** — все изменения фиксировать там
2. **Вопросы/решения** — добавлять в SPEC.md (секция שאלות פתוחות)
3. **Язык:** иврит для бизнес-терминов, английский для технических
4. **Priority данные:** проверять в `PriortyAgent/sdk/schema/`

---

## AI Tools — чтение PBIX файлов

### PBIXRay MCP Server
- **Статус:** ✅ установлен и подключён к Claude Code
- **Как работает:** MCP сервер даёт Claude прямой доступ к содержимому PBIX
- **Сервер:** `Work/_tools/pbixray-mcp-server/src/pbixray_server.py`
- **Python:** 3.13 (`C:\Users\Yuri Katz\AppData\Local\Programs\Python\Python313\python.exe`)
- **Конфиг:** `~/.claude.json` → mcpServers → pbixray

**Доступные инструменты:**
| Tool | Описание |
|------|----------|
| `load_pbix_file` | Загрузить PBIX файл |
| `get_tables` | Список таблиц |
| `get_dax_measures` | DAX меры |
| `get_power_query` | M код (Power Query) |
| `get_schema` | Схема таблиц |
| `get_relationships` | Связи модели |
| `get_table_contents` | Данные таблицы (с пагинацией) |
| `get_statistics` | Статистика модели |
| `get_model_summary` | Общий обзор |

### PBIXRay Python (fallback)
Если MCP недоступен, можно вызвать напрямую:
```bash
py -3.13 -c "
from pbixray import PBIXRay
model = PBIXRay(r'path\to\file.pbix')
print(model.tables)
print(model.dax_measures.to_string())
print(model.power_query.to_string())
print(model.statistics.to_string())
print(model.relationships.to_string())
"
```

### Зависимости
- `pbixray` 0.5.0 (Python 3.13, не работает на 3.14)
- `xpress9` 0.3.8 — декомпрессия DataModel
- `apsw`, `kaitaistruct`, `mcp`, `numpy`
