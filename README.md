### **Постановка завдання: Впровадження елементів DevSecOps у цикл розробки програмного забезпечення за допомогою GitHub Actions**  

#### **Мета:**  
Розробити та впровадити інтеграцію автоматизованих процесів безпеки на основі DevSecOps у процес CI/CD публічного репозиторію:

Приклад репозіторія: [juice-shop-github-actions](https://github.com/dimdimuzun/juice-shop-github-actions). 
Інтеграція повинна охоплювати статичний аналіз коду (SAST), аналіз залежностей (Software Composition Analysis) та динамічне тестування безпеки (DAST).  

---

### **Завдання:**  

#### **0. Установлення JuiceShop:**  
- **Опис:** Установити вразливий застосунок JuiceShop та налаштувати GitHub репозиторій.  
- **Дії:**  
  - Установити JuiceShop.  
  - Під'єднати його за допомогою SSH до репозиторію GitHub.

| Крок 1 | Крок 2 |
| :---: | :---: |
| ![Знімок 1](Знімок_20260504_190417.png) | ![Знімок 2](Знімок_20260505_011546.png) |

  - Запустити застосунок, за допомогою `npm install` -> `npm start`.  

![Знімок 3](Знімок_20260505_011359.png)

---

#### **1. Налаштування SAST за допомогою SonarCloud:**  
- **Опис:** Інтегрувати статичний аналіз коду для виявлення вразливостей на ранніх етапах розробки.  
- **Дії:**  
  - Налаштувати SonarCloud проєкт.  

![Налаштування SonarCloud](Pasted%20image%2020260505145427.png)

  - Додати токен доступу `SONAR_TOKEN` до секретів GitHub.  

![Додавання токена](Pasted%20image%2020260505145701.png)

  - Створити GitHub Actions workflow для запуску аналізу при кожному push або pull request у репозиторій.  

![Workflow 1](Pasted%20image%2020260505145934.png)
![Workflow 2](Pasted%20image%2020260505153652.png)

```yml
name: SAST

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

env:
  NODE_DEFAULT_VERSION: 20
  ANGULAR_CLI_VERSION: 17

jobs:
  sonarcloud:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v6

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_DEFAULT_VERSION }}
        cache: npm

    - name: Install dependencies
      run: npm install --legacy-peer-deps --no-audit --no-fun --ignore-scripts

    - name: SonarCloud Scan
      uses: sonarsource/sonarcloud-github-action@v3
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

  - Переконатися, що результати аналізу відображаються у вкладці Security.  

![Security 1](Pasted%20image%2020260505152640.png)
![Security 2](Pasted%20image%2020260505152734.png)
![Security 3](Pasted%20image%2020260505152815.png)

---

#### **2. Налаштування Software Composition Analysis за допомогою Snyk:**  
- **Опис:** Перевірити зовнішні залежності застосунку на наявність відомих вразливостей.  
- **Дії:**  
  - Зареєструвати репозиторій у Snyk.  

![Реєстрація Snyk](Pasted%20image%2020260505153415.png)

  - Додати токен доступу `SNYK_TOKEN` до секретів GitHub.  

![Токен Snyk](Pasted%20image%2020260505145701.png)

  - Налаштувати GitHub Actions workflow для регулярної перевірки залежностей та відправлення звітів у PR.  

![Snyk Workflow 1](Pasted%20image%2020260505153451.png)
![Snyk Workflow 2](Pasted%20image%2020260505153900.png)
![Snyk Workflow 3](Pasted%20image%2020260505154430.png)

```yml
name: SOFTWARE COMPOSITION ANALYSIS using Snyk

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  snyk:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

  - Пофіксити вразливість.

![Фікс 1](Pasted%20image%2020260505154749.png)
![Фікс 2](Pasted%20image%2020260505154929.png)
![Фікс 3](Pasted%20image%2020260505155858.png)


---

#### **3. Налаштування DAST за допомогою ZAProxy:**  
- **Опис:** Виконати динамічне тестування безпеки вебзастосунку після розгортання.  
- **Дії:**  
  - Додати конфігурацію для запуску контейнера ZAProxy у GitHub Actions. Створити сценарії сканування вебінтерфейсу проєкту. Налаштувати автоматичне виконання тестів після розгортання на тестовому сервері.

![ZAP Actions](Pasted%20image%2020260505155959.png)

```yml
name: DAST

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  pull-requests: write
  issues: write

jobs:
  zap_scan:
    runs-on: ubuntu-latest

    name: Scan the web applications
    steps:
      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: 'https://demo.owasp-juice.shop'
          cmd_options: '-a'
```

  - Проаналізувати звіт.  

![Звіт ZAP 1](Pasted%20image%2020260505161054.png)
![Звіт ZAP 2](Pasted%20image%2020260505161250.png)
![Звіт ZAP 3](Pasted%20image%2020260505161331.png)

---

### **Очікувані результати:**  
- **Автоматизація процесів безпеки:** Усі етапи перевірки безпеки виконуються автоматично під час CI/CD циклу.  
- **Звітування:** Отримання звітів про вразливості у GitHub Actions та у вкладці Security GitHub.  
- **Покращення процесу розробки:** Зменшення кількості вразливостей на різних етапах розробки через раннє виявлення ризиків.  

---

### **Критерії успішності:**  
- Успішна інтеграція та запуск усіх трьох компонентів (SAST, SCA, DAST).  
- Автоматичне формування звітів і візуалізація результатів у GitHub.  
- Мінімальна кількість помилок і збоїв при виконанні сценаріїв тестування безпеки.  

---

### **Додаткові вимоги:**  
- Оформити документацію з описом процесу налаштування кожного компонента.  
- Створити детальні інструкції для майбутньої підтримки та масштабування проєкту.
