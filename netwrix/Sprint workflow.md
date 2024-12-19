# Командный процесс
Процесс нашей команды разработчиков основан на методологии Scrumban. Методология Scrumban объединяет лучшие черты Scrum и Kanban в гибридную структуру управления проектами. Разработка разбита на спринты или итерации — фиксированные, повторяемые фазы продолжительностью в месяц. Каждый спринт имеет специальный рабочий процесс и включает в себя несколько событий, таких как обработка бэклога, оценка пользовательских историй, планирование спринта, еженедельное планирование, синхронизация с контролем качества, техническая встреча, обзор спринта. На картинке вы можете найти каждый из них:

# Оценка пользовательских историй
Члены команды используют пронумерованные карточки (1, 2, 3, 5, 8, 12, ?, ∞), чтобы назначать баллы историям пользователей и определять сложность рабочих элементов. Команда разработчиков обсуждает свои оценки на основе предыдущего опыта или мнения экспертов, и преобладающее число становится окончательной оценкой объекта.
Если какая-то пользовательская история недостаточно ясна (карточка «?»), команда разработчиков должна уточнить детали у владельца (обычно у менеджера по продукту).
Если какая-то пользовательская история оценена как «∞», команда разработчиков должна разделить ее на несколько элементов и оценить их.
Сеанс оценки может проводиться онлайн или оффлайн примерно за 7 рабочих дней до завершения текущего спринта.
Предполагаемые пользовательские истории следует помещать в столбец «Очередь» (** Предварительно одобренные элементы**) на доске невыполненных работ.

# Backlog grooming
Уборка бэклога, также называемая уточнением бэклога, является регулярной деятельностью команды разработчиков. Основная цель — поддерживать актуальность бэклога и готовить элементы бэклога для будущих спринтов. Груминг включает в себя декомпозицию функций, которая позволяет разделить большую работу на несколько задач, имеющих ценность для бизнеса — пользовательские истории. Могут быть некоторые регулярные задачи, такие как обслуживание конвейера DevOps, которые также можно определить как пользовательскую историю.
Уборку бэклога можно проводить в рамках любого подходящего мероприятия, например, при еженедельном планировании, синхронизации с контролем качества или планировании спринта. Бэклог может быть заполнен не только во время сеансов подготовки, но и в любое время любым членом команды, если можно определить пользовательскую историю.
Новые созданные пользовательские истории следует помещать в столбец «Новые» на [доске бэклога](./Backlog%20board.md).

# Планирование спринта
Планирование спринта — это событие, с которого начинается спринт. Цель планирования спринта — определить, что можно выполнить за спринт и как эта работа будет выполнена. Данное мероприятие проводится в последний понедельник месяца в рамках Еженедельного планирования.
При планировании спринта нам необходимо:

- Определить потенциал каждого члена команды в текущих проектах
- Учитывать работу, связанную с исправлением ошибок и поддержкой.
- В столбце «Очередь» выберите список пользовательских историй, которые будут взяты в спринте. Сумма сюжетных баллов не должна превышать общую скорость команды разработчиков. Истории пользователей следует переместить в столбец «Готово к старту» (журнал спринта) на доске журнала невыполненных работ.
- Сформулируйте цель спринта текущих проектов и подцели каждого направления, над которым мы планируем работать (сборщики/модули).

Планирование спринта — важная часть процесса, но не следует предполагать, что всю запланированную работу необходимо выполнить. Это не процесс «Чистого Scrum» со строгими планами спринтов. Мы должны мыслить в духе Agile, быть более гибкими и быть готовыми менять планы.

# Еженедельное планирование
Каждый понедельник у нас есть еженедельное собрание по планированию. Целью данного мероприятия является планирование задач на текущую неделю и синхронизация с текущей работой.

## Регулярная рутина каждого члена команды перед встречей:
- Убедитесь, что рабочий элемент, над которым вы работаете, находится в активном состоянии.
- Если вы все еще работаете над своим элементом «Работа» — актуализируйте «Выполненные часы».
- Если вы завершили свою работу, переместите элемент «Работа» в состояние «Выполнено», установите значение «Выполнено часов» полностью.
- Если вы используете Задача для отслеживания своей работы, вам также следует обновить Завершенные часы родительского рабочего элемента (например, Заявка или История пользователя). Рекомендуется обновиться в Закрытии Задачи (Решено - Закрыто). Это может сделать менеджер по разработке или назначенный член команды.
## Стендап-вопросы и действия на встрече
- Что ты делал на прошлой неделе? - Обзор прогресса
- Что ты будешь делать на этой неделе? - При необходимости назначьте новую работу (выберите один или несколько рабочих элементов из текущего журнала спринта).
- Есть ли какие-либо препятствия или препятствия, мешающие вам выполнять свою работу? - Запланируйте специальную встречу с заинтересованными сторонами, чтобы устранить блокировку.

## Notes:
- Что касается присутсвтвия всех на этом митинге, то это не обязательно. Я бы руководствовался вкладкой Capacity из текущего спринта. У кого Capacity больше 0, то предполагается участие в этом спринте над проектом Netwrix Auditor, а значит, и участи в митинге требуется. Также можно использовать эту информацию при решени участия или не участия в митинге с QC.
- То что запланировано на неделю, не должно быть сдвинуто новыми задачами (с одним исключением, см. ниже).

# Работа и приоритеты
- Рабочий элемент можно переместить из «Активного» в «Предложенный», если необходимо принять рабочий элемент с более высоким приоритетом. Рабочий элемент можно закрыть или удалить, если задача уже бесполезна.

- Не рекомендуется работать более чем над двумя объектами одновременно.

- Член команды должен получить новый рабочий элемент по приоритету. Если есть задача с наивысшим приоритетом, ее следует запустить первой.

- DevManager/Teamlead могу указать Priority в момент возникновения новой задачи не дожидаясь митинга. Передача высокоприоритетной задачи разработчику в середине недели, крайне не приветствуется. Задача может быть обозначена
как приоритетная и потенциальный испольнитель, после завершения текущей задачи, должен руководствоваться приоритетом беря в работу или следующую запланированную на текущей неделе задачу, или новую высокоприоритетную задачу
(при разнице 2 и более, например 3 приоритет проигрывает 1).

- Каждый из участников команды при досрочном выполнении всех запланированных задач (а вдруг?), может сам пойти в пулл и взять себе задачу, у которой проставлен Priority и StoryPoint (в случае с User Story). Если есть сложность с
выбором, то можно обратиться к DevManager/Teamlead-у с просьбой порекомендовать задачу 😉.

- Разработчику разрешается брать новосозданные баги в рамках фичи, над которой он(а) работает, без уведомления DevManager/Teamlead-а и процесса с пуллом. В этом случае Priority будет считаться как Severity.

- нет смысла оставлять ворк айтем в предыдущем спринте если он не был выполнен. Задача в том чтобы зафиксировать объем работы, который мы сделали за спринт. А если работа не закончена, то и функция, которую запланировали в спринт нет. Значит работа не считается сделанной в спринте и переходит в другой спринт

- Написание модульных тестов как обязательная задача для фичевых сторей
Наполнение задач можно проводить на одно из двух митингов либо спринт планинг или викли планинг

# Обзор спринта
Обзор спринта — это совместная встреча, которая обычно проводится после завершения спринта. Это мероприятие проводится в первую среду месяца вместо технической встречи. Менеджер по разработке объясняет, какую часть цели спринта им удалось достичь. Члены команды демонстрируют истории и задачи, которые они выполнили, и обсуждают незавершенные, а также препятствия, с которыми они столкнулись. Еще один показатель – отклонение от плана. Это следует проанализировать и учесть в будущих спринтах.
Иногда обзор спринта можно расширить с помощью ретроспективы.
Менеджер разработчиков использует следующие панели мониторинга для просмотра предыдущего спринта:

## Регулярная рутина каждого члена команды перед встречей
Каждый член команды должен реализовать выполненную работу предыдущего спринта.
Если вы все еще работаете над своим рабочим элементом, убедитесь, что Итерация установлена ​​как текущая.

# Синхронизируется с контролем качества
Это регулярные встречи с QC. Обсуждаем текущую работу и синхронизируем со следующими планами.

# Техническая встреча
Это обычная встреча команды разработчиков, на которой мы обсуждаем какую-то техническую тему или проблему.