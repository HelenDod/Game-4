# Лабораторная работа № 5 Создание индивидуальной системы достижения пользователя и ее интеграция в пользовательский интерфейс.
Отчет по лабораторной работе #5 выполнила:
- Додонова Елена Игоревна
- РИ-300002
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

## Цель работы
Cоздание интерактивного приложения с рейтинговой системой пользователя и интеграция игровых сервисов в готовое приложение.

## Задание 1
### Используя видео-материалы практических работ 1-5 повторить реализацию приведенного ниже функционала:
– 1 Практическая работа «Интеграции авторизации с помощью Яндекс SDK»

– 2 Практическая работа «Сохранение данных пользователя на платформе Яндекс Игры»

– 3 Практическая работа «Сбор данных об игроке и вывод их в интерфейсе»

– 4 Практическая работа «Интеграция таблицы лидеров»

– 5 Практическая работа «Интеграция системы достижений в проект»


### – 1 Практическая работа «Интеграции авторизации с помощью Яндекс SDK»
Ход работы:
1) Напишем скрипт, который будет осуществлять проверку авторизации пользователя.

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using YG;

public class CheckConnectYG : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += CheckSDK;
    private void OnDisable() => YandexGame.GetDataEvent -= CheckSDK;
    void Start()
    {
        if(YandexGame.SDKEnabled == true)
        {
            CheckSDK();
        }
    }

    public void CheckSDK()
    {
        if(YandexGame.auth == true)
        {
            Debug.Log("User authorization ok");
        }
        else
        {
            Debug.Log("User not authorization");
            YandexGame.AuthDialog();
        }
    }
}
```

2) Создадим пустой проект YandexManager. К нему будем прикреплять все скрипты, связанные с Yandex SDK. В данном случае прикрепим скрипт CheckConnectYG к этому объекту.

3) Сделаем билд и загрузим на платформу Яндекс Игры, чтоб проверить авторизацию игрока. Также поставим галочку о включении возможности авторизации и лидерборда.

4) Просмотрев страницу кода, мы можем наблюдать, что наш пользователь авторизован

5) Если сейчас выйти из своей учетной записи и снова зайти в игру, нас платформа попросит авторизоваться.

6) И зайдя обратно мы увидим следующие действия, что пользователь был не авторизован, а сейчас снова авторизован


### – 2 Практическая работа «Сохранение данных пользователя на платформе Яндекс Игры»
Ход работы:
1) Напишем скрипты, с помощью которых будет осуществляться подсчет и сохранение очков игрока.

```
namespace YG
{
    [System.Serializable]
    public class SavesYG
    {
        public bool isFirstSession = true;
        public string language = "ru";
        public bool feedbackDone;
        public bool promptDone;

        // Ваши сохранения
        public int score;
    }
}
```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using YG;
using TMPro;

public class DragonPicker : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += GetLoadSave;
    private void OnDisable() => YandexGame.GetDataEvent -= GetLoadSave;

    public GameObject energyShieldPrefab;
    public int numEnergyShield = 3;
    public float energyShieldBottomY = -6f;
    public float energyShieldRadius = 1.5f;
    public List<GameObject> shieldList;
    public TextMeshProUGUI scoreGT;
    void Start()
    {
        if(YandexGame.SDKEnabled == true)
        {
            GetLoadSave();
        }
        shieldList = new List<GameObject>();
        for (int i = 1; i <= numEnergyShield; i++){
            GameObject tShiedGo = Instantiate<GameObject>(energyShieldPrefab);
            tShiedGo.transform.position = new Vector3(0, energyShieldBottomY, 0);
            tShiedGo.transform.localScale = new Vector3(1*i, 1*i, 1*i);
            shieldList.Add(tShiedGo);
        }
        
    }

    void Update()
    {
        
    }

    public void DragonEggDestroyed(){
        GameObject[] tDragonEggArray = GameObject.FindGameObjectsWithTag("Dragon Egg");
        foreach (var tGO in tDragonEggArray)
        {
            Destroy(tGO);
        }
        int shieldIndex = shieldList.Count - 1;
        GameObject tShieldGo = shieldList[shieldIndex];
        shieldList.RemoveAt(shieldIndex);
        Destroy(tShieldGo);
        if (shieldList.Count == 0){
            GameObject scoreGO = GameObject.Find("Score");
            scoreGT = scoreGO.GetComponent<TextMeshProUGUI>();
            UserSave(int.Parse(scoreGT.text));
            SceneManager.LoadScene("_0Scene");
            GetLoadSave();
        }
    }
    public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
    }

    public void UserSave( int currentScore)
    {
        YandexGame.savesData.score = currentScore;
        YandexGame.SaveProgress();
    }
}
```

2) Сделаем билд проекта и выгрузим на Яндекс Игры. Запустим игру и проверим, как будут записываться набранные очки. Когда впервые запустили игру, у нас изначально 0 очков. После 3 набранных очков проигрываем. Сохраняются эти 3 очка. При перезапуске игры мы все так же видим эти 3 очка, что говорит о том, что результат пользователя сохранился на платформе.


### – 3 Практическая работа «Сбор данных об игроке и вывод их в интерфейсе»
Ход работы:
1) Сделаем так, чтобы у нас отображалось лучшее значение очков, которые набрал пользователь. Для этого на начальной сцене создадим объект TextMeshPro и настроим его местоположение в правом нижнем углу экрана.


2) Напишем скрипт, чтоб наилучшие набранные игроком очки могли записываться в Best Score.

```
namespace YG
{
    [System.Serializable]
    public class SavesYG
    {
        public bool isFirstSession = true;
        public string language = "ru";
        public bool feedbackDone;
        public bool promptDone;

        // Ваши сохранения
        public int score;
        public int bestScore = 0;
    }
}
```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using YG;
using TMPro;

public class DragonPicker : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += GetLoadSave;
    private void OnDisable() => YandexGame.GetDataEvent -= GetLoadSave;

    public GameObject energyShieldPrefab;
    public int numEnergyShield = 3;
    public float energyShieldBottomY = -6f;
    public float energyShieldRadius = 1.5f;
    public List<GameObject> shieldList;
    public TextMeshProUGUI scoreGT;
    void Start()
    {
        if(YandexGame.SDKEnabled == true)
        {
            GetLoadSave();
        }
        shieldList = new List<GameObject>();
        for (int i = 1; i <= numEnergyShield; i++)
        {
            GameObject tShiedGo = Instantiate<GameObject>(energyShieldPrefab);
            tShiedGo.transform.position = new Vector3(0, energyShieldBottomY, 0);
            tShiedGo.transform.localScale = new Vector3(1*i, 1*i, 1*i);
            shieldList.Add(tShiedGo);
        }
        
    }

    void Update()
    {
        
    }

    public void DragonEggDestroyed()
    {
        GameObject[] tDragonEggArray = GameObject.FindGameObjectsWithTag("Dragon Egg");
        foreach (var tGO in tDragonEggArray)
        {
            Destroy(tGO);
        }
        int shieldIndex = shieldList.Count - 1;
        GameObject tShieldGo = shieldList[shieldIndex];
        shieldList.RemoveAt(shieldIndex);
        Destroy(tShieldGo);
        if (shieldList.Count == 0)
        {
            GameObject scoreGO = GameObject.Find("Score");
            scoreGT = scoreGO.GetComponent<TextMeshProUGUI>();
            UserSave(int.Parse(scoreGT.text), YandexGame.savesData.bestScore);
            SceneManager.LoadScene("_0Scene");
            GetLoadSave();
        }
    }
    public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
    }

    public void UserSave(int currentScore, int currentBestScore)
    {
        YandexGame.savesData.score = currentScore;
        if (currentScore > currentBestScore)
        {
            YandexGame.savesData.bestScore = currentScore;
        }
        YandexGame.SaveProgress();
    }
}
```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using YG;
using TMPro;

public class CheckConnectYG : MonoBehaviour
{
    private void OnEnable() => YandexGame.GetDataEvent += CheckSDK;
    private void OnDisable() => YandexGame.GetDataEvent -= CheckSDK;
    private TextMeshProUGUI scoreBest;
    void Start()
    {
        if(YandexGame.SDKEnabled == true)
        {
            CheckSDK();
        }
    }

    public void CheckSDK()
    {
        if (YandexGame.auth == true)
        {
            Debug.Log("User authorization ok");
        }
        else
        {
            Debug.Log("User not authorization");
            YandexGame.AuthDialog();
        }
        GameObject scoreBO = GameObject.Find("BestScore");
        scoreBest = scoreBO.GetComponent<TextMeshProUGUI>();
        scoreBest.text = YandexGame.savesData.bestScore.ToString();
        scoreBest.text = "Best Score: " + YandexGame.savesData.bestScore.ToString();
    }
}
```

3) Снова делаем билд и загружаем на Яндекс Игры. После проигрыша у нас отображается максимально набранное количество очков.

4) Добавим имя игрока на экран. Создадим TextMeshPro PlayerName и зададим расположение над магом.

5) Напишем код в скрипте Dragon Picker, отвечающий за отображение имени на экране.

 ```
 public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
        GameObject playerNamePrefabGUI = GameObject.Find("PlayerName");
        playerName = playerNamePrefabGUI.GetComponent<TextMeshProUGUI>();
        playerName.text = YandexGame.playerName;
    }
 ```
При запуске мы можем видеть имя на экране.

### – 4 Практическая работа «Интеграция таблицы лидеров»
Ход работы:
1) Добавим в скрипт Dragon Picker строчку кода в метод DragonEggDestroyed. Благодаря этому данные после проигрыша будут передаваться в таблицу лидеров.

```
UserSave(int.Parse(scoreGT.text), YandexGame.savesData.bestScore);
```

2) Сделаем билд проекта, загрузим его на Яндекс Игры. Во вкладке "Лидерборды" добавим техническое и отображаемое названия.

3) После проигрыша в лидерборде мы можем видеть топ игроков.

### – 5 Практическая работа «Интеграция системы достижений в проект»
Ход работы:



## Задание 2
### Описать не менее трех дополнительных функций Яндекс SDK, которые могут быть интегрированы в игру.
Ход работы:

## Задание 3
### Доработать стилистическое оформление списка лидеров и системы достижений, реализованных в задании 1
Ход работы:

## Выводы
В ходе данной лабораторной работы мы смогли подготовить нашу игру к сборке для публикации. 
Мы создали стартовый экран игры с главным меню, тем самым сделав ее полноценнее. 
Поработали с настройкой анимации. 
Осуществили кликабельность кнопок, сделали паузу и выход из игры, настроили слайдер громкости, благодаря чему игра стала более интерактивная.
Добавили в игру музыкальные дорожки, произвели их настройку.
А так же для атмосферности включили персонажа, сделали ему анимацию движения.
