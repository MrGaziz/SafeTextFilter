﻿# text-moderation-server

## Обзор
Этот проект предназначен для управления и фильтрации плохих слов.
В проекте требуется api key для Google Perspective API он бесплатный а для OpenAPI платный
## Контроллеры

### Контроллер в папке `version1`
Контроллер в папке `version1` использует Google Perspective API для анализа и фильтрации текста. Подробнее о Google Perspective API можно прочитать на [официальном сайте](https://perspectiveapi.com/).

### Контроллер в папке `version2`
Контроллер в папке `version2` использует Java и OpenAPI для интеграции с GPT-3, предоставляя фильтрацию текста на основе искусственного интеллекта. Подробнее о Java и OpenAI можно прочитать на [Baeldung](https://www.baeldung.com/java-openai-api-client).

### Контроллер в папке `version3`
Контроллер в папке `version3` отправляет запросы в базу данных, которые мы сами добавляем через файл json.
Для улучшения фильтрации текста и проверки схожести слов, проект может использовать алгоритм Левенштейна. Алгоритм Левенштейна вычисляет минимальное количество операций, необходимых для преобразования одного слова в другое. Это позволяет эффективно обнаруживать и фильтровать вариации плохих слов.

## Зависимости

Убедитесь, что вы включили следующие зависимости в ваш файл `build.gradle`:

```groovy
dependencies {
    // Spring Boot стандартные зависимости
    implementation 'org.springframework.boot:spring-boot-starter:3.3.1'
    testImplementation 'org.springframework.boot:spring-boot-starter-test:3.3.1'
    implementation 'org.springframework.boot:spring-boot-starter-web:3.3.1'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa:3.3.1'

    // PostgreSQL JDBC драйвер
    implementation 'org.postgresql:postgresql:42.2.18'

    // Apache Commons Text для работы с алгоритмом Левенштейна
    implementation 'org.apache.commons:commons-text:1.12.0'

    // Jackson для работы с JSON
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.12.3'

    // OkHttpClient для HTTP операций
    implementation 'com.squareup.okhttp3:okhttp:4.9.0'

    // Gson для разбора JSON
    implementation 'com.google.code.gson:gson:2.8.6'

    // OpenAI GPT-3 Java SDK для взаимодействия с AI
    implementation 'com.theokanning.openai-gpt3-java:service:0.18.2'
    implementation 'com.theokanning.openai-gpt3-java:api:0.18.2'
    implementation 'com.theokanning.openai-gpt3-java:client:0.18.2'
}
```

## Как начать

### Требования
- Git
- PostgreSQL
- Java Development Kit (JDK)
- Maven (если не включен в проект) или build.gradle я предоставил выше

### Шаги установки

1. **Клонируйте репозиторий:**
    ```bash
    git https://github.com/MrGaziz/SafeTextFilter.git
    ```

2. **Создание таблицы в базе данных:**

   Вам нужно создать таблицу для хранения списка плохих слов. Используйте следующую SQL команду для создания таблицы:
    ```sql
    CREATE TABLE reject_reasons (
        id INT PRIMARY KEY,  -- Идентификатор причины отказа
        name VARCHAR(50) NOT NULL,  -- Код причины отказа
        reason_kazakh VARCHAR(255),  -- Причина отказа на казахском языке
        reason_russian VARCHAR(255)  -- Причина отказа на русском языке
    );
   
   CREATE TABLE prohibited_words (
        id UUID PRIMARY KEY,
        word VARCHAR(255) NOT NULL
    );
   
   CREATE TABLE  moderated_messages (
        id UUID PRIMARY KEY,
        msisdn VARCHAR(11) NOT NULL,
        message VARCHAR(300) NOT NULL,
        created_at TIMESTAMP NOT NULL,
        status INT NOT NULL,
        CONSTRAINT fk_status FOREIGN KEY (status) REFERENCES reject_reasons(id)
    );
    
   INSERT INTO reject_reasons (id, name, reason_kazakh, reason_russian) VALUES
    (1, 'accept', '-', '-'),
    (2, 'profanity', 'Хабарыңыз балағат сөздер қамтиды', 'Ваш текст содержит маты.'),
    (3, 'toxicity', 'Хабарыңыз зиянды немесе уытты мазмұнды қамтиды.', 'Ваш текст содержит токсичные выражения.'),
    (4, 'severe_toxicity', 'Хабарыңыз аса зиянды және уытты сөздер қамтиды.', 'Ваш текст содержит тяжелые токсичные выражения.'),
    (5, 'threat', 'Хабарыңыз қауіп төндіретін мазмұн қамтиды.', 'Ваш текст содержит угрозы.'),
    (6, 'propaganda', 'Хабарыңыз насихат мазмұнын қамтиды.', 'Ваш текст содержит пропаганду.'),
    (7, 'agitation', 'Хабарыңыз үгіт-насихат мазмұнын қамтиды.', 'Ваш текст содержит агитацию.'),
    (8, 'fraud', 'Хабарыңыз алаяқтық мазмұнды қамтиды.', 'Ваш текст содержит мошеннические выражения.'),
    (9, 'sexual_explicit', 'Хабарыңыз ересектерге арналған мазмұнды қамтиды.', 'Ваш текст содержит контент для взрослых.'),
    (10, 'insult', 'Хабарыңыз қорлау сөздерді қамтиды.', 'Ваш текст содержит оскорбления.'),
    (11, 'identity_attack', 'Хабарыңыз жеке тұлғаға бағытталған мәлімдемелерді қамтиды.', 'Ваш текст содержит высказывания, направленные против личности.'),
    (12, 'default_reject', 'Хабарыңыз абоненттермен өзара әрекеттесу саясатын бұзатын мәлімдемелерді қамтиды.', 'Ваш текст содержит высказывания, которые нарушают политику взаимодействия с абонентами.');
    ```

