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
![Снимок экрана (1851)](https://user-images.githubusercontent.com/90499063/205877943-21914cc3-e71e-40b5-8914-bbbfebfb0b54.png)

3) Сделаем билд и загрузим на платформу Яндекс Игры, чтоб проверить авторизацию игрока. Также поставим галочку о включении возможности авторизации и лидерборда.
![image (1)](https://user-images.githubusercontent.com/90499063/205878111-2714afe7-e245-4eef-9171-bff19a380e4d.png)

4) Просмотрев страницу кода, мы можем наблюдать, что наш пользователь авторизован
![image (2)](https://user-images.githubusercontent.com/90499063/205878204-dfbf7542-40c3-4e92-ac15-432983cbb340.png)

5) Если сейчас выйти из своей учетной записи и снова зайти в игру, нас платформа попросит авторизоваться.
![image (3)](https://user-images.githubusercontent.com/90499063/205878297-b6b14fc8-dcf9-4569-bd72-4156a6b96752.png)

6) И зайдя обратно мы увидим следующие действия, что пользователь был не авторизован, а сейчас снова авторизован
![image (4)](https://user-images.githubusercontent.com/90499063/205878382-90cb4c3c-08cf-4bd7-8913-cdd769b543a5.png)


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
![2 задание фото в конце](https://user-images.githubusercontent.com/90499063/205878572-48e57e45-abf9-4306-af10-8243bc135550.JPG)


### – 3 Практическая работа «Сбор данных об игроке и вывод их в интерфейсе»
Ход работы:
1) Сделаем так, чтобы у нас отображалось лучшее значение очков, которые набрал пользователь. Для этого на начальной сцене создадим объект TextMeshPro и настроим его местоположение в правом нижнем углу экрана.
![image (5)](https://user-images.githubusercontent.com/90499063/205879778-3735d091-621b-4dda-9360-d7bfa534e902.png)

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
![image (6)](https://user-images.githubusercontent.com/90499063/205879952-2e74cd63-b40c-4e14-9089-df8a4e855643.png)
![image (7)](https://user-images.githubusercontent.com/90499063/205879997-5085e65a-1039-48b9-8706-01b9870cf0b4.png)

4) Добавим имя игрока на экран. Создадим TextMeshPro PlayerName и зададим расположение над магом.
![image (8)](https://user-images.githubusercontent.com/90499063/205880074-ad0294a3-2f36-4378-8be2-7dd652f00c51.png)

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

![image (9)](https://user-images.githubusercontent.com/90499063/205880155-87c94e08-ea86-4cb4-aae3-6eddebed7ff4.png)


### – 4 Практическая работа «Интеграция таблицы лидеров»
Ход работы:
1) Добавим в скрипт Dragon Picker строчку кода в метод DragonEggDestroyed. Благодаря этому данные после проигрыша будут передаваться в таблицу лидеров.

```
UserSave(int.Parse(scoreGT.text), YandexGame.savesData.bestScore);
```

2) Сделаем билд проекта, загрузим его на Яндекс Игры. Во вкладке "Лидерборды" добавим техническое и отображаемое названия.
![image (10)](https://user-images.githubusercontent.com/90499063/205880251-cd69572a-324e-4142-a262-37855ce5b6d7.png)
![image (11)](https://user-images.githubusercontent.com/90499063/205880307-43ae84cb-19fd-44b7-9fb7-27f1fe1d6a81.png)

3) После проигрыша в лидерборде мы можем видеть топ игроков.
![Снимок экрана (1857)](https://user-images.githubusercontent.com/90499063/205881077-7e4f1179-f3d6-4d32-8902-37f8d6b6621b.png)

### – 5 Практическая работа «Интеграция системы достижений в проект»
Ход работы:
1) Добавим раздел в меню, который будет нас перенаправлять на список достижений в игре. Выполним настройки перехода между окнами. 
![image (12)](https://user-images.githubusercontent.com/90499063/205881188-61c9174b-42fc-4e5b-a585-212bdbc8900d.png)

2) Визуально настроим окно достижений.
![image (13)](https://user-images.githubusercontent.com/90499063/205881262-095e3ab3-2e1a-4b41-b827-2139cfe03c18.png)

3) Напишем скрипты, которые отвечают за обработку данных достижений пользователя. Так выглядят финальные строки.

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
        public string[] achivMent = new string[5];
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
    [SerializeField] private TextMeshProUGUI scoreBest;
    [SerializeField] private TextMeshProUGUI ListAchiv;
    [SerializeField] private TextMeshProUGUI achivList;
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
        scoreBest.text = "Best Score: " + YandexGame.savesData.bestScore.ToString();
        if ((YandexGame.savesData.achivMent)[0] == null & !GameObject.Find("ListAchiv"))
        {

        }
        else
        {
            foreach(string value in YandexGame.savesData.achivMent)
            {
                Debug.Log(value + "Nice!");
                GameObject ListAchiv = GameObject.Find("ListAchiv");
                achivList.text = value.ToString() + "\n";
                //GameObject.Find("ListAchiv").GetComponent<TextMeshProUGUI>().text = 
                //GameObject.Find("ListAchiv").GetComponent<TextMeshProUGUI>().text + value + '\n';
            }
        }
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

    [SerializeField] public GameObject energyShieldPrefab;
    [SerializeField] private int numEnergyShield = 3;
    [SerializeField] private float energyShieldBottomY = -6f;
    [SerializeField] private float energyShieldRadius = 1.5f;
    [SerializeField] private TextMeshProUGUI scoreGT;
    [SerializeField] private TextMeshProUGUI playerName;
    public List<GameObject> shieldList;
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
            string[] achivList;
            achivList = YandexGame.savesData.achivMent;
            achivList[0] = "Береги щиты!";
            Debug.Log(achivList[0]);
            UserSave(int.Parse(scoreGT.text), YandexGame.savesData.bestScore, achivList);
            YandexGame.NewLeaderboardScores("TOPPlayerScore", int.Parse(scoreGT.text));
            SceneManager.LoadScene("_0Scene");
            GetLoadSave();
        }
    }
    public void GetLoadSave()
    {
        Debug.Log(YandexGame.savesData.score);
        GameObject playerNamePrefabGUI = GameObject.Find("PlayerName");
        playerName = playerNamePrefabGUI.GetComponent<TextMeshProUGUI>();
        playerName.text = YandexGame.playerName;
    }

    public void UserSave(int currentScore, int currentBestScore, string[] currentAchiv)
    {
        YandexGame.savesData.score = currentScore;
        if (currentScore > currentBestScore)
        {
            YandexGame.savesData.bestScore = currentScore;
        }
        YandexGame.savesData.achivMent = currentAchiv;
        YandexGame.SaveProgress();
    }
}
```

https://user-images.githubusercontent.com/90499063/205886535-5726f337-e6d0-4216-9288-ab30c17f23bd.mp4

## Задание 2
### Описать не менее трех дополнительных функций Яндекс SDK, которые могут быть интегрированы в игру.
Ход работы:
Функции, которые можно дополнительно интегрировать в игру:

1) Покупки внутри игры. Пользователи могут вносить реальные деньги и получать за них дополнительные превилегии в виде скинов, уровней, жизней.

2) Баннерная реклама. Уже привычная пользователям реклама, с помощью которой можно получать доход. А так же пользователи после ее просмотра получают бонусы в игре.

3) Оценки и отзывы. Можно просить пользователя оценить игру и написать комментарий, за что он может получить какой-либо бонус внутри игры. Это поможет игре стать более популярной и узнаваемой на рынке.

## Задание 3
### Доработать стилистическое оформление списка лидеров и системы достижений, реализованных в задании 1
Ход работы:
![image (14)](https://user-images.githubusercontent.com/90499063/205894782-46a54414-f810-4168-b4c4-ffc0eb3c6599.png)

## Выводы
В ходе данной лабораторной работы мы смогли подготовить нашу игру к сборке для публикации. 
Мы создали стартовый экран игры с главным меню, тем самым сделав ее полноценнее. 
Поработали с настройкой анимации. 
Осуществили кликабельность кнопок, сделали паузу и выход из игры, настроили слайдер громкости, благодаря чему игра стала более интерактивная.
Добавили в игру музыкальные дорожки, произвели их настройку.
А так же для атмосферности включили персонажа, сделали ему анимацию движения.
