# Анализ данных и искуственный интеллект Лабараторная № 1 сбор, обработка и визуализация тестового набора данных
Отчет по лабораторной работе #1 выполнил:
- Устинов Никита Валерьевич
- РИ210910

Отметки о выполнении задания
(undone-#, done-*)

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

Структура отчета


## Цели работы
Ознакомиться с основными операторами языка Python на примере реализации линейной регрессии.

## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity. При выполнении задания используйте видео-материалы и исходные данные, предоставленные преподавателя курса.
- В облачном сервисе google console подключить API для работы с google sheets и google drive.
- Реализовать запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
- Создать новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.
- Написать функционал на Unity, в котором будет воспризводиться аудио-файл в зависимости от значения данных из таблицы.


## Задание 2
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1

## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2

Ход работы:
### Задание 1
- В облачном сервисе google console подключил API для работы с google sheets и google drive(скриншот 1)
- Релизовал запись данных из скрипта на python в google-таблицу, данные которой описывают изменение темпа инфляции на протяжении 10 отчетных периодов с учетом стоимости игрового объекта в каждый период

```py
import gspread
import numpy as np
gc = gspread.service_account(filename="unitylabadatasciense-792809ea8ff5.json")
sh = gc.open("UnityLabaDataSCienseSheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1, 11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('c' + str(i)), str(tempInf))
        print(tempInf)
```

- Создал проект в Unity, который получал данные из google-таблицы, в которую были записаны данные в предыдущем пункте(скриншон 2)
- Написал функционал на Unity, в котором воспроизводился аудио-файл в зависимости от значения данных из таблицы

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip GoodSpeak;
    public AudioClip NormalSpeak;
    public AudioClip BadSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;


    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioMode(GoodSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioMode(NormalSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioMode(BadSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1Iji8SOdY2JcdGTF19Nd0O2aQLkdRdeoAxntz2xZPICE/values/Лист1?key=AIzaSyAxo_3QPnebOrbOnnwIJxtKjy83Wz9t4lE");
        yield return curentResp.SendWebRequest();
        string RawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(RawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioMode(AudioClip mode)
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = mode;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

### Задание 2
Добавил функцию линейной регрессии в инициализацию данных в таблицу

```py
import gspread
import inline as inline
import matplotlib
import numpy as np
import matplotlib.pyplot as plt
gc = gspread.service_account(filename="unitylabadatasciense-792809ea8ff5.json")
sh = gc.open("UnityLabaDataSCienseSheets")
priceX = np.random.randint(2000, 10000, 11)
priceX = np.array(priceX)
priceY = np.random.randint(2000, 10000, 11)
priceY = np.array(priceY)
mon = list(range(1, 11))


a = np.random.rand(1)
print(a)
b = np.random.rand(1)
print(b)
Lr = 0.000001


def model(a, b, x):
  return a * x + b


def loss_function(a, b, x, y):
  num = len(x)
  prediction = model(a, b, x)
  return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
  num = len(x)
  prediction = model(a, b, x)
  da = (1.0 / num) * ((prediction - y) * x).sum()
  db = (1.0 / num) * (prediction - y).sum()
  a = a - Lr * da
  b = b - Lr * db
  return a, b

def iterate (a, b, x, y, times):
  for i in range(times):
    a, b = optimize(a, b, x, y)
  return a, b


a, b = iterate(a, b, priceX, priceY, 1)
prediction = model(a, b, priceX)
loss = loss_function(a, b, priceX, priceY)
print(a, b, loss)
plt.scatter(x,y)
plt.plot(x,prediction)


i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = ((prediction[i-1]-prediction[i-2])/prediction[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(prediction[i-1]//1))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(prediction)
```


### Задание 3
- написал функцию проверки уровня инфляции, добавил новые сценарии, зависящие от уровня инфляции

```cs
void Update()
    {
        if (CheckInflation(int.MinValue, 10))
        {
            StartCoroutine(PlaySelectAudioMode(GoodSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (CheckInflation(10, 100))
        {
            StartCoroutine(PlaySelectAudioMode(NormalSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (CheckInflation(100, int.MaxValue))
        {
            StartCoroutine(PlaySelectAudioMode(BadSpeak));
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    bool CheckInflation(int min, int max)
    {
        return dataSet["Mon_" + i.ToString()] >= min &
            dataSet["Mon_" + i.ToString()] <= max &
            statusStart == false &
            i != dataSet.Count;
    }
```

## Выводы
В ходе работы ознакомился с работой сервисов google console API, google-таблиц. Смоделировал изменение цены на товар и инфляцию, исходя из разницы цен в разные промежутки итерации. Создал в Unity скрипт, который в зависимости от принимаемых значений инфляции выдает не только её текущий показатель, но и звуковой эффект-отчет.

## Приложение
### Основная литература:
Рекомендуемая литература
1.	Геймдизайн: как создать игру, в которую будут играть все. / Шелл Дж. - М.: Альпина Паблишер, 2021. - 640 с. : ил. - ISBN 978-5-9614-1209-3.
2.	Unity и C#. Геймдев от идеи до реализации. / Бонд Д. Г. - 2-е изд. - СПб.: Питер, 2021. - 928 с. : ил. - ISBN 978-5-4461-0715-5.
3.	Грас Дж. Data Science. Найка о данных с нуля: Пер. с англ. – СПб.: БХВ-Петербург, 2020. – 336 с.
