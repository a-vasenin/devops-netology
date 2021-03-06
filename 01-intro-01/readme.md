### Задание №2 - Описание жизненного цикла задачи (разработки нового функционала)

Чтобы лучше понимать предназначение дальнейших инструментов, с которыми нам предстоит работать, давайте 
составим схему жизненного цикла задачи в идеальном для вас случае.

#### Описание истории

Представьте, что вы работаете в стартапе, который запустил интернет-магазин.
Ваш интернет-магазин достаточно успешно развивался, и вот пришло время налаживать процессы: у вас стало больше конечных клиентов, менеджеров и разработчиков
Сейчас от клиентов вам приходят задачи, связанные с разработкой нового функционала.
Задач много, и все они требуют выкладки на тестовые среды, одобрения тестировщика, проверки менеджером перед показом клиенту.
В случае необходимости, вам будет необходим откат изменений. 

#### Решение задачи

Вам необходимо описать процесс решения задачи в соответствии с жизненным циклом разработки программного обеспечения.
Использование какого-либо конкретного метода разработки не обязательно.
Для решения главное - прописать по пунктам шаги решения задачи (релизации в конечный результат) с участием менеджера, разработчика (или команды разработчиков),
тестировщика (или команды тестировщиков) и себя как DevOps-инженера.

## Решение

1. Менеджер берет откуда-то требования. Например:
- клиент говорит нам, что хочет какую-то штуку
- или мы сами анализируем рынок, и решаем, что какая-то штука будет нам полезна

2. Собираем команду - людей, которые эту фичу будут реализовывать, обсуждаем с ними фичу. Сюда входят менеджеры, разработчики, тестировщики, девопсы. 

3. Делим, кто что будет делать. Например, разработчик - кодировать бизнес-логику, писать юнит тесты, выбирать данные для мониторинга. Тестировщик - писать системные тесты, проводить исследовательское тестирование. Девопс - подготавливать инфраструктурку, писать CI/CD скрипты, подключать мониторинг. Менеджер - разговаривать с людьми, или что они там делают, я не знаю.

4. Через какое-то время разработчик коммитит код, где-то крутятся тесты. Проходит ревью. Если тесты или ревью не прошли - фича возвращается на доработку.

5. Если прошло, фича едет в тестовое окружение. Гоняются, например, приемочные тесты. Мониторинг мониторит. Если тесты не прошли, или мониторинг ругается - билд откатывается, фича возвращается на доработку.

6. Тестировщик тестирует. Если тесты не прошли, возвращает на доработку.

7. Фича катится по кнопочке какой-нибудь в продакшн (кнопочке выкатки, или кнопочке мержа). Можно на небольшое количество пользователей сначала, чтобы снизить масштаб вероятноых дефектов.

8. Запускаем тесты, мониторим поведение фичи в продакшене. Если что-то выглядит плохо, от билд откатываем, возвращаем на доработку.
