# API для внешних площадок НТО
Внешние площадки могут и должны загружать нам баллы за свои курсы самостоятельно. В интерфейсе ментора будет предусмотрена возможность просмотра баллов своих учеников по каждой задаче отдельно, для внешних площадок есть два варианта работы:

- Загрузка задач и баллов по каждой задаче
- Загрузка баллов за активность (возможно только если у активности не загружено ни одной задачи, иначе возможно несоответствие загруженных и высчитанных баллов активности)
## Prerequests

Для каждой активности внешней площадки необходимо указать в поле `client_id` ID OAuth клиента в таланте. Все запросы валидируются по наличию этого `client_id` в авторизационном токене.

Задачи сгруппированы в уроки, уроки в свою очередь сгруппированы в попытки. У каждой попытки есть дата начала и дата окончания.


# Получение информации о команде
    GET /api/activity/{activity_id}/user/{talent_user_id}/team


# Создание попыток

POST `/api/activity/{activity_id}/attempt`
PATCH `/api/activity/{activity_id}/attempt/{attempt_id}`

BODY: 

    title: str
    start_at: str (YYYY-mm-dd HH:MM:SS)
    end_at: str (YYYY-mm-dd HH:MM:SS)

RESP:

    id: int
    title: str
    start_at: datetime
    end_at: datetime
    stepik_section_id: null // Актуально только для попыток на степике
    activity: Activity

ERRORS:

    404 "activity_does_not_exist" - неправильный id активности
    400 "not_allowed_for_client" - у активности не указан(указан другой) client_id


# Создание уроков

POST `/api/activity/{activity_id}/lesson`
PATCH `/api/activity/{activity_id}/lesson/{lesson_id}`

BODY: 

    title: str
    attempt_id: int

RESP:

    id: int
    title: str
    attempt: Attempt
    stepik_lesson_id: null // Актуально только для уроков на степике

ERRORS:

    404 "activity_does_not_exist" - неправильный id активности
    404 "attempt_does_not_exist" - неправильный id попытки
    400 "not_allowed_for_client" - у активности не указан(указан другой) client_id



# Загрузка задач

POST `/api/activity/{activity_id}/task`
PATCH `/api/activity/{activity_id}/task/{task_id}`
DELETE `/api/activity/{activity_id}/task/{task_id}`

BODY: 

    // Текст задачи
    description: str
    
    // id урока, задачи группируются по урокам и попыткам, для суммрования баллов и зачисления только лучшей попытки
    lesson_id: int
    
    // Позиция (порядковый номер) задачи в уроке
    position: int

RESP:

    id: int
    step_id: null // Актуально только для задач на степике
    lesson: Lesson
    position: int

ERRORS:

    404 "activity_does_not_exist" - неправильный id активности
    404 "lesson_does_not_exist" - неправильный id урока
    400 "not_allowed_for_client" - у активности не указан(указан другой) client_id
# Баллы за задачу

POST  `/api/score/task`

BODY:

    task_id: int
    score: float
    talent_user_id: int // ID юзера в таланте

ERRORS:

    404 "task_not_found" - неправильный id задачи
    400 "not_allowed_for_client" - у активности к которой относится задача не указан(указан другой) client_id
    400 "user_has_no_suitable_profile" - у юзера не выбран профиль содержащий активность задачи
    400 "user_has_no_participations" - юзер не участвует в НТО


# Баллы за активность

Возможно использовать только если у активности нет задач

POST  `/api/score/activity`

BODY:

    activity_id: int
    score: float
    talent_user_id: int // ID юзера в таланте

ERRORS:

    404 "activity_not_found" - неправильный id активности
    400 "not_allowed_for_client" - у активности не указан(указан другой) client_id
    400 "activity_has_tasks" - у активности есть задачи, используйте позадачную загрузку
    400 "user_has_no_suitable_profile" - у юзера не выбран профиль содержащий активность
    400 "user_has_no_participations" - юзер не участвует в НТО

