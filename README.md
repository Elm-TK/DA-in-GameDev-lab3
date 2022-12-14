# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:
- Ельмуратов Темирлан Кеулимжаевич
- РИ-210913

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.

Установил все нужное ПО, пакеты, расширения. (в том числе: mlagents 0.28.0,	torch 1.7.1)

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/11.png)

Добавил в сцену все нужные объекты и добавил скрипт для шара:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```

Добавил в директорию проекта файл yaml c таким содержанием:

```yaml
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

Запустил процесс обучения. Количество итераций достигло 210000.

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/1.png)

Присвоил шару файл, полученный в результате обучения.

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/3.png)

Результат: шар не падает с платформы и достигает куба достаточно плавно, точно и по оптимальной траектории.

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/2.gif)


## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.

```yaml
behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
```

### trainer_type
Тип обучения. 
ppo — обучение с поощрением

### batch_size

Количество выборок на обновление градиента. Если значение не указано, по умолчанию будет установлено значение 32.

### buffer_size

Количество опыта, который необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы приступим к какому-либо изучению или обновлению модели. Это должно быть в несколько раз больше, чем batch_size. Обычно больший размер буфера соответствует более стабильным обновлениям обучения.

### learning_rate

Параметр задает величину каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если обучение нестабильно, а вознаграждение не увеличивается последовательно.

### beta

Соответствует силе регуляции энтропии, что делает политику «более случайной». Параметр гарантирует, что агенты должным образом исследуют пространство действия во время обучения. Это приведет к более случайным действиям. Бету нужно отрегулировать так, чтобы энтропия медленно уменьшалась вместе с увеличением вознаграждения. Если энтропия падает слишком быстро, увеличьте бета-версию. Если энтропия падает слишком медленно, уменьшите бету.


### epsilon

Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении градиентного спуска. Установка
этого значения мала приведет к более стабильным обновлениям, но также замедлит процесс обучения.

### lambd

Параметр регуляризации (лямбда), используемый при расчете Обобщенной оценки преимущества (GAE). Это можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при расчете обновленной оценки стоимости. Низкие значения соответствуют большей зависимости от текущей оценки ценности (что может быть большой погрешностью), а высокие значения соответствуют большей зависимости от фактических вознаграждений, полученных в окружающей среде (что может быть высокой дисперсией). Параметр обеспечивает компромисс между ними, и правильное значение может привести к более стабильному процессу обучения. 

### num_epoch

Количество проходов, которые необходимо выполнить через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер пакета, тем больше это допустимо сделать. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.

### learning_rate_schedule

Определяет, как скорость обучения меняется с течением времени.

### normalize

Применяется ли нормализация к входным данным векторного наблюдения. Эта нормализация основана на текущем среднем значении и дисперсии векторного наблюдения. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна при более простых задачах дискретного управления.

### hidden_units

Соответствует количеству единиц в каждом полностью подключенном слое нейронной сети. Для простых задач, где правильным действием является простая комбинация входных данных наблюдения, это должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это должно быть больше.

### num_layers

Соответствует количеству скрытых слоев, присутствующих после ввода наблюдения или после кодирования CNN визуального наблюдения. Для простых задач меньшее количество слоев, скорее всего, будет тренироваться быстрее и эффективнее. Для более сложных задач управления может потребоваться больше уровней.

### gamma

Соответствует коэффициенту скидки для будущих вознаграждений. Это можно рассматривать как то, как далеко в будущем агент должен заботиться о возможных вознаграждениях. В ситуациях, когда агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем, это значение должно быть большим. В тех случаях, когда вознаграждение более немедленное, оно может быть меньше.

### strength.

Фактор, на который можно умножить вознаграждение, получаемое от окружающей среды. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения.

### max_steps

Общее количество шагов, которые должны быть выполнены в среде (или во всех средах, если используется несколько параллельно) до завершения процесса обучения.

### time_horizon

Соответствует количеству шагов сбора опыта для каждого агента перед добавлением его в буфер опыта.

### summary_freq

Количество опыта, которое необходимо собрать перед созданием и отображением статистики обучения.

## Задание 3
### Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и в первом задании, случайно изменять координаты на плоскости. 

Доработал код для того, чтобы шар перемещался между двумя кубами:
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public GameObject Target;
    public GameObject Target1;

    private bool targetAchieved;
    private bool target1Achieved;

    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.transform.localPosition = new Vector3(Random.value * 8 - 4, 0.5f, Random.value * 8 - 4);
        Target1.transform.localPosition = new Vector3(Random.value * 8 - 4, 0.5f, Random.value * 8 - 4);
        Target.SetActive(true);
        Target1.SetActive(true);
        targetAchieved = false;
        target1Achieved = false;
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.transform.localPosition);
        sensor.AddObservation(Target1.transform.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(targetAchieved);
        sensor.AddObservation(target1Achieved);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }

    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget1 = Vector3.Distance(this.transform.localPosition, Target.transform.localPosition);
        float distanceToTarget2 = Vector3.Distance(this.transform.localPosition, Target1.transform.localPosition);

        if (!targetAchieved & distanceToTarget1 < 1.42f)
        {
            targetAchieved = true;
            Target.SetActive(false);
        }

        if (!target1Achieved & distanceToTarget2 < 1.42f)
        {
            target1Achieved = true;
            Target1.SetActive(false);
        }

        if (targetAchieved & target1Achieved)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```

Обучил модель двигаться между двумя целями:

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/5.png)

Результат:

![](https://github.com/Elm-TK/DA-in-GameDev-lab3/blob/main/4.gif)

## Выводы
В ходе этой лабораторной работы я получил начальные знания и опыт использования MLAgent-a в Unity.

Игровой баланс - попытка привести все параметры, объекты и части игры в состояние равновесия. Например: середина между временем прохождения / затянутостью, монотонностью и усталостью от прохождения, уравновешивание параметров игровых персонажей или объектов, расчёт экономических аспектов игры, сложность уровней и т. д.

Для расчёта параметров, которые бы приближали игру к состоянию сбалансированности, нужно изучать, сравнивать и рассчитывать огромное количество параметров. В данной ситуации могут очень пригодиться системы машинного обучения, которые на основании определенных начальных данных будут вычислять эти самые параметры: возможности объектов / доход, траты и цены / сложность и другие аспекты игры.


