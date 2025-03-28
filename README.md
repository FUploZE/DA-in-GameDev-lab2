# DA-in-GameDev-lab2
# Отчет по лабораторной работе #2 выполнил(а):
- Аржавицин Захар Денисович
- НМТ-233929

## Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

Знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.
- 
---

## Цель работы

Ознакомиться 

---

## Задание 1
### Анализ игровой переменной

Выбранная игровая переменная: **[HP]**

**Роль в игре:** Данная пепременная играет важную роль в игре.При падении Health point'ов до нуля игра заканчивется. Ограниченность здоровья создает напряжение и требует стратегического планирования..

**Условия изменения:** Данная переменная изменяется при получение урона, вампиризма .

**Диапазон значений:** от 0 до 30 ( может быть выше 30 если игрок будет получать лвл).

**Схема экономической модели:** Построить схему с указанием связей выбранной переменной с другими ресурсами в игре.

---

## Задание 2
### Заполнение Google-таблицы и визуализация данных

Ссылка на таблицу с диаграммиой https://docs.google.com/spreadsheets/d/1viWXKnXLKsJK7_cb4AWK1Y62fHdkI1NUmSjtyn4pzC0/edit?usp=sharing

**Код скрипта на Python:**
```python
import gspread
import numpy as np


MAX_HP = 30
DAMAGE = 10 
VAMPIRISM_MIN = 1  
VAMPIRISM_MAX = 3  

gc = gspread.service_account(filename='unity-ad-454702-607b1f51f4dd.json')
sh = gc.open("Unity Workshop 2")
worksheet = sh.worksheet("Лист2")
worksheet.clear()
worksheet.update('A1:C1', [['№', 'HP', 'Изменение']])

current_hp = MAX_HP
event_count = 10  # Количество событий

for event in range(1, event_count + 1):

    will_heal = np.random.random() > 0.5 and current_hp > 0
    
    if will_heal:
        heal = np.random.randint(VAMPIRISM_MIN, VAMPIRISM_MAX + 1)
        new_hp = min(MAX_HP, current_hp + heal)
        change = f"+{heal}"
    else:
        new_hp = max(0, current_hp - DAMAGE)
        change = f"-{DAMAGE}"
    worksheet.update(f'A{event + 1}', [[event]])
    worksheet.update(f'B{event + 1}', [[new_hp]])
    worksheet.update(f'C{event + 1}', [[change]])
    current_hp = new_hp
print("Данные успешно записаны в таблицу!")
```

**Анализ данных и предложения по улучшению:**
Проблемы:
Слишком быстрая смерть (3 удара).
Вампиризм на lvl 1 слабый (+3 HP vs -10 за удар).
Улучшения:
Вариант 1: Уменьшить урон до -7 HP.
Вариант 2: Добавить Аптечки, покупаемые у торговца
---

## Задание 3
### Настройка звукового сопровождения в Unity

**Шаги выполнения:**
1. Добавлены звуковые файлы в Unity.
2. Реализован скрипт, реагирующий на изменение переменной.
3. При изменении переменной проигрываются соответствующие звуки (например, звук удара при снижении здоровья ).

**Код на C# для воспроизведения звука:**
```csharp
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;
using SimpleJSON;

public class HealthSystem : MonoBehaviour
{
    public AudioClip damageSound;
    public AudioClip healSound;
    public AudioClip criticalSound;
    public AudioClip deathSound;

    private AudioSource audioSource;
    private int currentHP = 30;
    private bool isDead;

    void Start()
    {
        audioSource = GetComponent<AudioSource>();
        if (audioSource == null)
        {
            audioSource = gameObject.AddComponent<AudioSource>();
        }

        StartCoroutine(LoadHealthData());
    }

    IEnumerator LoadHealthData()
    {
        string url = "https://sheets.googleapis.com/v4/spreadsheets/1viWXKnXLKsJK7_cb4AWK1Y62fHdkI1NUmSjtyn4pzC0/values/Лист2?key=AIzaSyAcP5xI3mtlIAMqGaay8AViHpdtVPKA9mY";
        UnityWebRequest request = UnityWebRequest.Get(url);
        yield return request.SendWebRequest();

        if (request.result != UnityWebRequest.Result.Success)
        {
            Debug.LogError($"Ошибка загрузки данных: {request.error}");
            yield break;
        }

        var json = JSON.Parse(request.downloadHandler.text);
        if (json == null || json["values"] == null)
        {
            Debug.LogError("Ошибка: JSON пустой или не содержит 'values'");
            yield break;
        }

        for (int i = 1; i < json["values"].Count; i++)
        {
            var row = json["values"][i];

            if (row.Count < 3)
            {
                Debug.LogWarning($"Ошибка в строке {i + 1}: недостаточно данных");
                continue;
            }

            string hpStr = row[1];
            string changeStr = row[2];

            if (!int.TryParse(hpStr, out int hpValue))
            {
                Debug.LogWarning($"Ошибка парсинга HP в строке {i + 1}: '{hpStr}'");
                continue;
            }

            ProcessHPChange(hpValue, changeStr);
            yield return new WaitForSeconds(1f);
        }
    }

    void ProcessHPChange(int newHP, string change)
    {
        if (isDead) return;

        currentHP = newHP;
        Debug.Log($"Обработка: HP={currentHP}, Изменение={change}");

        if (change.Contains("-"))
        {
            if (damageSound != null) audioSource.PlayOneShot(damageSound);
            if (currentHP <= 10 && criticalSound != null) audioSource.PlayOneShot(criticalSound);
        }
        else if (change.Contains("+"))
        {
            if (healSound != null) audioSource.PlayOneShot(healSound);
        }

        if (currentHP <= 0)
        {
            if (deathSound != null) audioSource.PlayOneShot(deathSound);
            isDead = true;
        }
    }
}

```

---

## Выводы
В ходе выполнения лабораторной работы:
- Проанализирована игровая переменная (ХП) и её место в экономической модели игры "СПАСТИ РТФ: Выживание".
- Разработан и выполнен скрипт на Python для сбора и визуализации данных.
- Проведен анализ динамики изменения переменной и предложены варианты её улучшения.
- Реализовано звуковое сопровождение изменений переменной в Unity.
- Отчет оформлен в виде документации на GitHub с использованием Markdown.
