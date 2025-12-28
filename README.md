openapi: 3.0.3
info:
  title: Сервис управления меню и производства
  description: API системы производства \"Робот и точка\". Микросервис управления приготовлением блюд в роботизированном ресторане. Отвечает за мониторинг роботов-поваров, выполнение рецептов по технологическим картам и взаимодействием с системой управления запасами.
  version: 1.0.0
  contact:
    name: Elisey Bauer
    email: severjanen@rambler.ru
  license:
    name: Proprietary
    url: https://restaurant.com/license

servers: 
  - url: "https://github.com/Elisey1986/API_specification/blob/54e8b39c20cce676da65900e0d9ce00142cdfd4d/README.md"
    description: Production server
  - url: https://api.robot&dot.com/v1
    description: "prod"
  - url: https://api-test.robot&dot.com/v1
    description: "test"

paths:
  # ---------- ЭНТПОЙНТЫ ЗАКАЗОВ ----------
  /orders:
    get:
      tags: 
        - Заказы
      summary: Получить список заказов в производстве
      parameters:
        - name: OrderStatus
          in: query
          schema:
            type: string
            enum: [готовится, ожидание, отменен, готов, сбой]
          description: Фильтр по статусу
        - name: по дате
          in: query
          schema:
            type: string
            default: 01.01.2025
            format: дд.мм.гггг
          description: Фильтр по дате создания
        - name: количество записей
          in: query
          schema:
            type: integer
            default: 20
          description: Лимит записей
        - name: Начальная запись №
          in: query
          schema:
            type: integer
            default: 0
          description: Начать с записи №
      responses:
        '200':
          description: Список заказов
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderListResponse'
        '500':
          $ref: '#/components/responses/InternalServerError'

    post:
      tags: 
        - Заказы
      summary: Создать новый заказ на приготовление
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateProductionOrderRequest'
      responses:
        '201':
          description: Заказ успешно создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionOrderResponse'
        '400':
          $ref: '#/components/responses/BadRequestError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /orders/{orderId}:
    get:
      tags: 
        - Заказы
      summary: Получить детали заказа
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
            example: "00001_01_12_2025"
          description: Номер заказа на производство
      responses:
        '200':
          description: Детали заказа
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionOrderResponse'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

    put:
      tags: 
        - Заказы
      summary: Обновить статус заказа
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
            example: "00001_01_12_2025"
          description: Номер заказа
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateOrderRequest'
      responses:
        '200':
          description: Заказ обновлен
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionOrderResponse'
        '400':
          $ref: '#/components/responses/BadRequestError'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /orders/{orderId}/cancel:
    post:
      tags: 
        - Заказы
      summary: Отменить приготовление заказа
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
            example: "00001_01_12_2025"
          description: Номер заказа
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                reason:
                  type: string
                  description: Причина отмены
      responses:
        '200':
          description: Приготовление отменено
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionOrderResponse1'
        '400':
          description: Невозможно отменить, так как заказ уже завершен
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  # ---------- ЭНТПОЙНТЫ РОБОТОВ ----------
  /robots:
    get:
      tags: [Данные о роботах]
      summary: Получить список всех роботов
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [готов, занят, ошибка, техническое обслуживание, отключен]
          description: Фильтр по статусу
        - name: type
          in: query
          schema:
            type: string
            enum: [Фритюр, Жарщик, Тостер, Салат, Напитки]
          description: Фильтр по типу робота
      responses:
        '200':
          description: Список роботов
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RobotListResponse'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /robots/{robotId}:
    get:
      tags: [Данные о роботах]
      summary: Получить информацию о роботе
      parameters:
        - name: robotId
          in: path
          required: true
          schema:
            type: string
          description: Идентификатор робота
      responses:
        '200':
          description: Информация о роботе
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RobotResponse'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

    put:
      tags: [Данные о роботах]
      summary: Обновить настройки робота
      parameters:
        - name: robotId
          in: path
          required: true
          schema:
            type: string
          description: Идентификатор робота
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateRobotRequest'
      responses:
        '200':
          description: Настройки обновлены
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RobotResponse'
        '400':
          $ref: '#/components/responses/BadRequestError'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'


  /robots/{robotId}/emergency-stop:
    post:
      tags: [Данные о роботах]
      summary: Аварийная остановка робота
      parameters:
        - name: robotId
          in: path
          required: true
          schema:
            type: string
          description: Идентификатор робота
      responses:
        '200':
          description: Команда аварийной остановки отправлена
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  timestamp:
                    type: string
                    format: date-time
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  # ========== Техногогические карты ==========
  /tech_map:
    get:
      tags: [Информация о тех.картах]
      summary: Получить список тех.карт
      parameters:
        - name: по ID блюда
          in: query
          schema:
            type: string
            example: "burger_001"
          description: Фильтр по ID блюда
        - name: по совместимости с роботом
          in: query
          schema:
            type: string
            example: "Robot_001"
          description: Фильтр по совместимости с роботом
      responses:
        '200':
          description: Список рецептов
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TechMapListResponse'
        '500':
          $ref: '#/components/responses/InternalServerError'

    post:
      tags: [Информация о тех.картах]
      summary: Загрузить техкарту
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTechMapRequest'
      responses:
        '201':
          description: Рецепт создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TechMapResponse'
        '400':
          $ref: '#/components/responses/BadRequestError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /tech_map/{tech_map_Id}:
    get:
      tags: [Информация о тех.картах]
      summary: Получить детали тех.карты
      parameters:
        - name: tech_map_Id
          in: path
          required: true
          schema:
            type: string
            example: "tec_map_001_01122025"
          description: Фильтр по ID карты
      responses:
        '200':
          description: Детали рецепта
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TechMapResponse'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

    put:
      tags: [Информация о тех.картах]
      summary: Обновить тех.карту
      parameters:
        - name: tech_map_Id
          in: path
          required: true
          schema:
            type: string
            example: "tec_map_001_01122025"
          description: Фильтр по ID карты
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateTechMapRequest'
      responses:
        '200':
          description: Тех.карта обновлена
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TechMapResponse'
        '400':
          $ref: '#/components/responses/BadRequestError'
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /tech_map/{tech_map_Id}/delete:
    delete:
      tags: [Информация о тех.картах]
      summary: Удалить рецепт
      parameters:
        - name: tech_map_Id
          in: path
          required: true
          schema:
            type: string
            example: "tec_map_001_01122025"
          description: Фильтр по ID карты
        - name: force
          in: query
          schema:
            type: boolean
            default: false
          description: Принудительное удаление (hard delete)
      responses:
        '200':
          description: Рецепт успешно удален
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DeleteRecipeResponse'
        '400':
          description: |
            Невозможно удалить рецепт:
            - Рецепт используется в активных заказах
            - Это единственный активный рецепт для блюда
            - Некорректный replacementRecipeId
        '404':
          $ref: '#/components/responses/NotFoundError'
        '500':
          $ref: '#/components/responses/InternalServerError'
    

  # ========== КОНЕЧНЫЕ ТОЧКИ ПРОИЗВОДСТВА ==========
  /production/stats:
    get:
      tags: [Статистика]
      summary: Получить статистику производства
      parameters:
        - name: period
          in: query
          schema:
            type: string
            enum: [час, день, неделя, месяц]
            default: день
          description: Период агрегации
      responses:
        '200':
          description: Статистика производства
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionStatsResponse'
        '500':
          $ref: '#/components/responses/InternalServerError'

  /production/queue:
    get:
      tags: [Статистика]
      summary: Получить текущую очередь производства
      responses:
        '200':
          description: Очередь производства
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductionQueueResponse'
        '500':
          $ref: '#/components/responses/InternalServerError'

  # ========== События ==========
  /webhooks/robot-status:
    post:
      tags: [События]
      summary: Webhook для обновления статуса робота (вызывается роботом)
      description: |
        Роботы отправляют сюда свои статусы/логи через MQTT/HTTP.
        Аутентификация через API ключ или сертификат.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/RobotStatusUpdate'
      responses:
        '200':
          description: Статус принят
          content:
            application/json:
              schema:
                type: object
                properties:
                  received:
                    type: boolean
        '400':
          $ref: '#/components/responses/BadRequestError'
        '401':
          $ref: '#/components/responses/UnauthorizedError'

  
components:
  schemas:
    ErrorResponse:
      type: object
      properties:
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            details:
              type: array
              items:
                type: string
        timestamp:
          type: string
          format: date-time

    # ========== Заказы ==========
    CreateProductionOrderRequest:
      type: object
      required:
        - order_id
        - dish_id
        - quantity
      properties:
        order_id:
          type: string
          description: ID заказа из Order Service
          example: "00001_01_12_2025"
        dish_id:
          type: string
          description: ID блюда из Menu Service
          example: "burger_001"
        tech_map_version:
          type: string
          description: Версия тех.карты
          example: "1_01022025"
        quantity:
          type: integer
          minimum: 1
          description: Количество порций
        priority:
          type: string
          enum: [низкий, нормальный, высокий]
          default: нормальный
          description: Приоритет приготовления

    ProductionOrderResponse:
      type: object
      properties:
        id:
          type: string
          format: uuid
          description: Идентификатор запроса
          example: "6c4b2bc0-337c-44fe-9f46-2508cde55169"
        order_id:
          type: string
          description: Идентификатор заказа
          example: "00001_01_12_2025"
        dish_id:
          type: string
          description: Уникальный идентификатор позиции заказа в корзине
          example: "burger_001"
        dish_name:
          type: string
          description: Наименование блюда
          example: "Бургер Троцкий"
        quantity:
          type: integer
          description: Количество заказов
          example: "1"
        status:
          $ref: '#/components/schemas/OrderStatus'
        robot_id:
          type: string
          description: Идентификатор робота
          example: "Robot_001"
        tech_map_version:
          type: string
          description: Идентификтор тех.карты
          example: "1_01022025"
        start_time:
          type: string
          format: date-time
          description: Время начала приготовления
        completion_time:
          type: string
          format: date-time
          description: Время окончания приготовления
    
    ProductionOrderResponse1:
      type: object
      properties:
        id:
          type: string
          format: uuid
          description: Идентификатор запроса
          example: "6c4b2bc0-337c-44fe-9f46-2508cde55169"
        order_id:
          type: string
          description: Идентификатор заказа
          example: "00001_01_12_2025"
        dish_id:
          type: string
          description: Уникальный идентификатор позиции заказа в корзине
          example: "burger_001"
        dish_name:
          type: string
          description: Наименование блюда
          example: "Бургер Троцкий"
        status:
          $ref: '#/components/schemas/OrderStatus1'
        cancellation_time:
          type: string
          format: date-time
          description: Время окончания приготовления

    OrderListResponse:
      type: object
      properties:
        orders:
          type: array
          items:
            $ref: '#/components/schemas/ProductionOrderResponse'
        pagination:
          $ref: '#/components/schemas/PaginationInfo'

    UpdateOrderRequest:
      type: object
      properties:
        priority:
          type: string
          enum: [низкий, нормальный, высокий]
          default: нормальный
        scheduled_for:
          type: string
          format: date-time
          nullable: true

    # ========== роботы ==========
    RobotResponse:
      type: object
      properties:
        robot_id:
          type: string
          description: Идентификатор робота
          example: "Robot_001"
        type:
          type: string
          enum: [Фритюр, Жарщик, Тостер, Салат, Напитки]
        model:
          type: string
          description: Ноименование модели
          example: "3ad4940"
        status:
          $ref: '#/components/schemas/RobotStatus'
        capabilities:
          type: array
          items:
            type: string
          description: Список поддерживаемых операций
        order_id:
          type: string
          nullable: true
          example: "00001_01_12_2025"
          description: идентификатор текущего заказа
        ingredient_levels:
          type: array
          items:
            $ref: '#/components/schemas/IngredientLevel'
        last_maintenance:
          type: string
          format: date-time
          nullable: true
        uptime:
          type: integer
          description: Время работы в секундах
          example: "1200"
        errors:
          type: array
          items:
            $ref: '#/components/schemas/RobotError'
        created_in_system:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time
    RobotListResponse:
      type: object
      properties:
        robots:
          type: array
          items:
            $ref: '#/components/schemas/RobotResponse'
        stats:
          type: object
          properties:
            total:
              type: integer
            ready:
              type: integer
            busy:
              type: integer
            error:
              type: integer

    UpdateRobotRequest:
      type: object
      properties:
        name:
          type: string
        status:
          $ref: '#/components/schemas/RobotStatus'
        maintenance_mode:
          type: boolean

    RobotStatusUpdate:
      type: object
      required:
        - robot_id
        - status
        - timestamp
      properties:
        robot_id:
          type: string
        status:
          $ref: '#/components/schemas/RobotStatus'
        timestamp:
          type: string
          format: date-time
        current_task:
          type: string
          nullable: true
        progress:
          type: number
          minimum: 0
          maximum: 100
        ingredient_levels:
          type: array
          items:
            $ref: '#/components/schemas/IngredientLevel'
        errors:
          type: array
          items:
            type: string
        temperature:
          type: number
          nullable: true
        power_consumption:
          type: number
          nullable: true

    # ========== Технологические карты ==========
    CreateTechMapRequest:
      type: object
      required:
        - dish_id
        - name
        - steps
      properties:
        dish_id:
          type: string
          example: "burger_001"
        name:
          type: string
          example: "Бургер Мечта поручика Ржевского"
        version:
          type: string
          default: "1.0.0"
        description:
          type: string
          example: "Бургер на ржаной булке с добавлением клюквенного соуса"
        compatible_robots:
          type: array
          items:
            type: string
          description: Список типов роботов, совместимых с рецептом
        steps:
          type: array
          items:
            $ref: '#/components/schemas/RecipeStep'
        estimated_time:
          type: integer
          example: 180
          description: Оценочное время приготовления в секундах
        difficulty:
          type: string
          enum: [easy, medium, hard]
          default: medium
        ingredients:
          type: array
          items:
            $ref: '#/components/schemas/RecipeIngredient'

    TechMapResponse:
      type: object
      properties:
        tec_map_id:
          type: string
          example: "tec_map_001_01122025"
        dish_id:
          type: string
          example: "burger_001"
        dish_name:
          type: string
          description: Наименование блюда
          example: "Бургер Троцкий"
        name:
          type: string
          example: "Бургер Троцкий"
        version:
          type: string
          example: "1.0.1"
        description: 
          type: string
        steps:
          type: array
          items:
            $ref: '#/components/schemas/RecipeStep'
        estimated_time:
          type: integer
          description: Время выполнения
          example: "180"
        ingredients:
          type: array
          items:
            $ref: '#/components/schemas/RecipeIngredient'
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time

    TechMapListResponse:
      type: object
      properties:
        recipes:
          type: array
          items:
            $ref: '#/components/schemas/TechMapResponse'
        pagination:
          $ref: '#/components/schemas/PaginationInfo'

    UpdateTechMapRequest:
      type: object
      properties:
        name:
          type: string
        description:
          type: string
        compatible_robots:
          type: array
          items:
            type: string
        steps:
          type: array
          items:
            $ref: '#/components/schemas/RecipeStep'
        estimated_time:
          type: integer
        difficulty:
          type: string
    
    DeleteRecipeResponse:
      type: object
      properties:
        tec_map_id:
          type: string
          example: "tec_map_001_01122025"
        status:
          type: string
          enum: [удалено, ошибка]
        message:
          type: string
          example: "Технологическая карта tec_map_001_01122025 успешно удалена"
        deleted_at:
          type: string
          format: date-time

    # ========== Продукция ==========
    ProductionStatsResponse:
      type: object
      properties:
        period:
          type: string
        orders:
          type: object
          properties:
            total:
              type: integer
            completed:
              type: integer
            failed:
              type: integer
            avg_completion_time:
              type: number
        robots:
          type: object
          properties:
            utilization_rate:
              type: number
            avg_uptime:
              type: number
            error_rate:
              type: number
        efficiency:
          type: object
          properties:
            on_time_rate:
              type: number
            resource_utilization:
              type: number
            waste_percentage:
              type: number
        top_dishes:
          type: array
          items:
            type: object
            properties:
              dish_id:
                type: string
              dish_name:
                type: string
                description: Наименование блюда
                example: "Бургер Троцкий"
              count:
                type: integer

    ProductionQueueResponse:
      type: object
      properties:
        pending:
          type: array
          items:
            $ref: '#/components/schemas/QueueItem'
        in_progress:
          type: array
          items:
            $ref: '#/components/schemas/QueueItem'
        scheduled:
          type: array
          items:
            $ref: '#/components/schemas/QueueItem'

    OrderStatus:
      type: string
      enum: [готовится, приостановлен, готов, ошибка]
    OrderStatus1:
      type: string
      enum: [отменен, ошибка]
    RobotStatus:
      type: string
      enum: [готов, занят, ошибка, техническое обслуживание, отключен]

    # ========== SUB-КОМПОНЕНТЫ ==========
    IngredientLevel:
      type: object
      properties:
        ingredient_id:
          type: string
          description: Идентификатор ингридиента
          example: "012g31"
        name:
          type: string
          description: наименование ингридиента
          example: "котлета говяжья"
        current_level:
          type: number
          minimum: 0
          maximum: 100
          description: Уровень заполнения в процентах

    RobotError:
      type: object
      properties:
        code:
          type: string
          enum: 
            - E01 — перегрузка двигателя;
            - E02 — перенапряжение;
            - E03 — пониженное напряжение;
            - E08 — тепловая защита, нужно дать машине остыть, проверить, не закупорены ли вентиляционные щели моторного блока;
            - E09 — предохранительные выключатели, необходимо нажать на кнопку выключения;
            - E12 — тепловая защита, нужно проверить кнопку выключения на предмет западания или дать машине остыть до 20 минут;
            - E13 — попытка запуска машины в то время, как она была в процессе торможения, необходимо дождаться полной остановки мотора, после чего приступить к перезапуску;
            - E14 — предохранительные выключатели (ILS) незакрыты, нужно проверить правильное положение выключателей защитных элементов (крышки, нажимных кнопок, защёлки крышки и т. д.).
        message:
          type: string
          example: "Рекомендуется замена робота"
        severity:
          type: string
          enum: [низкий, средний, высокий, критический]
        timestamp:
          type: string
          format: date-time
        resolved:
          type: boolean

    RecipeStep:
      type: object
      required:
        - action
        - duration
      properties:
        step_number:
          type: integer
          example: 1
        action:
          type: string
          description: Действие 
          example: 
            - Поместить индейку на смазанный маслом лист пекарской бумаги на горячую сковороду.
        duration:
          type: integer
          description: Длительность в секундах
          example: 5
        target_temperature:
          type: number
          nullable: true
          description: Температура приготовления
          example: 200
        target_weight:
          type: number
          nullable: true
          description: Вес продукта
          example: 50
        parameters:
          type: object
          additionalProperties: true
        dependencies:
          type: array
          items:
            type: integer
          description: Номера шагов, которые должны быть выполнены до этого

    RecipeIngredient:
      type: object
      required:
        - ingredient_id
        - quantity
        - unit
      properties:
        ingredient_id:
          type: string
          example: "cutlet_001"
        name:
          type: string
          example: "Говяжья котлета"
        quantity:
          type: number
          minimum: 1
        unit:
          type: string
          enum: [шт, г, кг, л, мл]

    QueueItem:
      type: object
      properties:
        order_id:
          type: string
          description: ID заказа из Order Service
          example: "00001_01_12_2025"
        dish_name:
          type: string
          description: Наименование блюда
          example: "Бургер Троцкий"
        priority:
          type: string
        estimated_start:
          type: string
          format: date-time
          nullable: true
        estimated_completion:
          type: string
          format: date-time
          nullable: true
        robot_id:
          type: string
          description: Идентификатор робота
          example: "Robot_001"

    PaginationInfo:
      type: object
      properties:
        total:
          type: integer
        limit:
          type: integer
          default: 10
        offset:
          type: integer
          default: 0

  responses:
    BadRequestError:
      description: Неверный запрос
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            error:
              code: "VALIDATION_ERROR"
              message: "Недопустимые параметры запроса"
              details: ["Количество должно быть больше 0"]
            timestamp: "2024-01-15T10:30:00Z"

    UnauthorizedError:
      description: Не авторизован
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    ForbiddenError:
      description: Доступ запрещен
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

    NotFoundError:
      description: Ресурс не найден
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            error:
              code: "ORDER_NOT_FOUND"
              message: "Заказ с идентификатором 123 не найден"
              details: []
            timestamp: "2024-01-15T10:30:00Z"

    InternalServerError:
      description: Внутренняя ошибка сервера
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: API ключ для внутренних сервисов

security:
  - ApiKeyAuth: []
