# Руководство по эффектам и частицам в GLua (Garry's Mod)

## Содержание

1. [Введение](#введение)
2. [Основы системы эффектов](#основы-системы-эффектов)
3. [Простые эффекты](#простые-эффекты)
4. [Система частиц](#система-частиц)
5. [Продвинутые техники](#продвинутые-техники)
6. [Стандартные эффекты и материалы](#стандартные-эффекты-и-материалы)
7. [Оптимизация](#оптимизация)
8. [Полезные ссылки](#полезные-ссылки)

## Введение

GLua - это модифицированная версия языка Lua, используемая в Garry's Mod. Система эффектов и частиц - одна из самых зрелищных возможностей игры, позволяющая создавать впечатляющие визуальные эффекты: от простых искр до сложных взрывов и световых шоу.

**Для кого это руководство:**

- Разработчиков аддонов для Garry's Mod
- Создателей серверов, желающих добавить уникальные визуальные эффекты
- Энтузиастов, изучающих программирование в GLua

**Что вы узнаете:**

- Как работает система эффектов в Garry's Mod
- Как использовать встроенные эффекты
- Как создавать собственные эффекты с нуля
- Лучшие практики и методы оптимизации

## Основы системы эффектов

### Типы эффектов

В Garry's Mod существует два основных типа эффектов:

1. **Встроенные эффекты** - готовые эффекты, доступные через функцию `util.Effect`. Например, взрывы, искры, дым.

2. **Пользовательские эффекты** - созданные вами с нуля эффекты с использованием системы частиц.

### Клиент-серверная архитектура

Важно понимать: **все эффекты выполняются на стороне клиента**, но часто инициируются сервером. Это ключевой момент для понимания работы с эффектами.

Типичная схема работы:

1. Сервер определяет, что нужно создать эффект (например, произошел взрыв)
2. Сервер отправляет клиентам информацию о том, где и какой эффект нужно создать
3. Клиенты получают эту информацию и локально создают эффект

```lua
-- На сервере
if SERVER then
    util.AddNetworkString("CreateEffect")

    function TriggerEffect(pos)
        -- Отправляем всем клиентам
        net.Start("CreateEffect")
        net.WriteVector(pos)
        net.Broadcast()
    end
end

-- На клиенте
if CLIENT then
    net.Receive("CreateEffect", function()
        local pos = net.ReadVector()

        -- Создаем эффект взрыва
        local effectData = EffectData()
        effectData:SetOrigin(pos)
        util.Effect("Explosion", effectData)
    end)
end
```

Это базовая схема, которая позволяет серверу указать, где должен появиться эффект, а клиенту - отобразить его.

## Простые эффекты

### Использование встроенных эффектов

Garry's Mod включает множество готовых эффектов, которые вы можете использовать немедленно, без необходимости создавать что-то с нуля.

#### Как использовать встроенный эффект

**Шаг 1:** Создайте объект EffectData
**Шаг 2:** Настройте его параметры (позиция, масштаб и т.д.)
**Шаг 3:** Вызовите util.Effect с нужным именем эффекта

Вот простой пример создания взрыва:

```lua
-- Эта функция создает эффект взрыва в указанной позиции
function CreateExplosion(position, scale)
    local effectData = EffectData()
    effectData:SetOrigin(position)  -- Где создать эффект
    effectData:SetScale(scale or 1) -- Размер эффекта
    util.Effect("Explosion", effectData)
end

-- Вызываем функцию для создания взрыва (например, в консоли)
-- CreateExplosion(LocalPlayer():GetPos(), 1.5)
```

#### Популярные встроенные эффекты для демонстрации:

```lua
-- Создание брызг крови
function CreateBloodSplash(position, normal)
    local effect = EffectData()
    effect:SetOrigin(position)
    effect:SetNormal(normal or Vector(0, 0, 1))
    util.Effect("BloodImpact", effect)
end

-- Создание электрических искр
function CreateSparks(position, normal)
    local effect = EffectData()
    effect:SetOrigin(position)
    effect:SetNormal(normal or Vector(0, 0, 1))
    effect:SetMagnitude(2) -- Интенсивность
    effect:SetScale(1.5)   -- Размер
    util.Effect("ElectricSpark", effect)
end

-- Создание дымового следа
function CreateSmokeTrail(entity, duration)
    local effect = EffectData()
    effect:SetEntity(entity)
    effect:SetScale(duration or 2) -- Длительность в секундах
    util.Effect("smoke_trail", effect)
end
```

### Основные параметры EffectData

EffectData позволяет настраивать эффекты через следующие методы:

| Метод                | Описание                      | Типичное использование                   |
| -------------------- | ----------------------------- | ---------------------------------------- |
| SetOrigin(vector)    | Устанавливает позицию эффекта | Где появится эффект                      |
| SetNormal(vector)    | Устанавливает направление     | Для направленных эффектов (искры, кровь) |
| SetEntity(entity)    | Привязывает эффект к сущности | Для эффектов, следующих за объектом      |
| SetScale(number)     | Устанавливает масштаб         | Размер или длительность эффекта          |
| SetMagnitude(number) | Устанавливает силу            | Интенсивность эффекта                    |
| SetRadius(number)    | Устанавливает радиус          | Область действия эффекта                 |
| SetAngles(angle)     | Устанавливает угол            | Направление эффекта                      |

## Система частиц

Система частиц - это основа для создания сложных эффектов в Garry's Mod. Она позволяет создавать и управлять множеством маленьких объектов (частиц), которые вместе формируют визуальный эффект.

### Как создать эмиттер и частицы

Чтобы использовать систему частиц, необходимо:

1. Создать эмиттер в нужной позиции
2. Добавить частицы с помощью эмиттера
3. Настроить их свойства
4. Завершить работу эмиттера

Вот небольшой пример создания огненных частиц:

```lua
-- Создание простого огненного эффекта
function CreateFireEffect(pos, duration)
    -- Создаем эмиттер
    local emitter = ParticleEmitter(pos)

    -- Время начала для отслеживания длительности
    local startTime = CurTime()

    -- Будем создавать частицы в течение указанного времени
    timer.Create("FireEffect"..tostring(pos), 0.05, duration/0.05, function()
        -- Если прошло нужное время, останавливаем
        if CurTime() > startTime + duration then
            return
        end

        -- Создаем огненную частицу
        local particle = emitter:Add("particles/flamelet"..math.random(1,5),
                                     pos + VectorRand() * 5)

        if particle then
            -- Настраиваем частицу
            particle:SetVelocity(Vector(0, 0, math.Rand(20, 30)))
            particle:SetLifeTime(0)
            particle:SetDieTime(math.Rand(0.5, 1.0))
            particle:SetStartAlpha(200)
            particle:SetEndAlpha(0)
            particle:SetStartSize(math.Rand(5, 10))
            particle:SetEndSize(2)
            particle:SetColor(255, 180, 100)
            particle:SetRoll(math.Rand(0, 360))
        end
    end)

    -- Завершаем эмиттер через указанное время
    timer.Simple(duration, function()
        if emitter and emitter:IsValid() then
            emitter:Finish()
        end
    end)

    return emitter
end

-- Использование:
-- CreateFireEffect(Vector(0, 0, 0), 5) -- Огонь в течение 5 секунд
```

### Важные свойства частиц

При создании частиц вы можете настраивать множество их свойств:

| Свойство      | Описание               | Пример использования                    |
| ------------- | ---------------------- | --------------------------------------- |
| SetVelocity   | Направление и скорость | particle:SetVelocity(Vector(0, 0, 20))  |
| SetLifeTime   | Начальное время жизни  | particle:SetLifeTime(0)                 |
| SetDieTime    | Время до исчезновения  | particle:SetDieTime(1.5)                |
| SetStartAlpha | Начальная прозрачность | particle:SetStartAlpha(255)             |
| SetEndAlpha   | Конечная прозрачность  | particle:SetEndAlpha(0)                 |
| SetStartSize  | Начальный размер       | particle:SetStartSize(10)               |
| SetEndSize    | Конечный размер        | particle:SetEndSize(2)                  |
| SetColor      | Цвет                   | particle:SetColor(255, 100, 50)         |
| SetRoll       | Угол вращения          | particle:SetRoll(math.Rand(0, 360))     |
| SetRollDelta  | Скорость вращения      | particle:SetRollDelta(math.Rand(-1, 1)) |
| SetGravity    | Гравитация             | particle:SetGravity(Vector(0, 0, -100)) |
| SetCollide    | Включение столкновений | particle:SetCollide(true)               |
| SetBounce     | Упругость (0-1)        | particle:SetBounce(0.3)                 |

## Продвинутые техники

Теперь, когда мы освоили основы, перейдем к более интересным и продвинутым техникам создания эффектов.

### Анимированные частицы

Вы можете создавать частицы, которые меняются со временем, используя функцию `SetThinkFunction`. Вот пример простого анимированного огненного эффекта:

```lua
function CreateAnimatedFire(pos, duration)
    local emitter = ParticleEmitter(pos)
    local startTime = CurTime()

    -- Создаем несколько базовых частиц
    for i = 1, 5 do
        local particle = emitter:Add("particles/flamelet"..math.random(1,5), pos)

        if particle then
            particle:SetVelocity(VectorRand() * 5 + Vector(0, 0, 20))
            particle:SetLifeTime(0)
            particle:SetDieTime(duration)
            particle:SetStartSize(15)
            particle:SetEndSize(5)
            particle:SetStartAlpha(200)
            particle:SetEndAlpha(0)
            particle:SetRoll(math.Rand(0, 360))

            -- Начальные цвета
            local startColor = Color(255, 180, 100)
            local endColor = Color(255, 50, 0)
            particle:SetColor(startColor.r, startColor.g, startColor.b)

            -- Устанавливаем функцию думания (обновление каждый кадр)
            particle:SetNextThink(CurTime())

            -- Эта функция будет вызываться каждый кадр для частицы
            particle:SetThinkFunction(function(part)
                -- Расчет прогресса жизни частицы (0-1)
                local progress = (CurTime() - startTime) / 2

                -- Плавно меняем цвет от оранжевого к красному
                local r = Lerp(progress, startColor.r, endColor.r)
                local g = Lerp(progress, startColor.g, endColor.g)
                local b = Lerp(progress, startColor.b, endColor.b)
                part:SetColor(r, g, b)

                -- Планируем следующее обновление
                part:SetNextThink(CurTime())
            end)
        end
    end

    timer.Simple(duration, function()
        if emitter and emitter:IsValid() then
            emitter:Finish()
        end
    end)
end

-- Использование:
-- CreateAnimatedFire(Vector(0, 0, 0), 5) -- Анимированный огонь на 5 секунд
```

### Создание следов (трейлов)

Следы за объектами - эффектный способ подчеркнуть движение. Garry's Mod предоставляет простую функцию для их создания:

```lua
-- Создание следа за объектом
function AttachTrailToEntity(entity, color, width, duration)
    -- Параметры по умолчанию
    color = color or Color(255, 50, 50)
    width = width or 10
    duration = duration or 1

    -- Используем встроенную функцию для создания следа
    local trail = util.SpriteTrail(
        entity,           -- Родительская сущность
        0,                -- Аттачмент (0 для центра)
        color,            -- Цвет следа
        false,            -- Добавить как потомка?
        width,            -- Стартовая ширина
        0,                -- Конечная ширина
        duration,         -- Продолжительность
        1 / width * 0.5,  -- Скорость затухания
        "trails/plasma"   -- Материал следа
    )

    return trail
end

-- Пример использования:
-- AttachTrailToEntity(LocalPlayer(), Color(0, 255, 0), 15, 2) -- Зеленый след
```

### Комбинированные эффекты

Настоящая мощь системы эффектов раскрывается при комбинировании разных типов частиц. Вот пример небольшого комбинированного эффекта взрыва:

```lua
function CreateMiniExplosion(pos, scale)
    scale = scale or 1
    local emitter = ParticleEmitter(pos)

    -- 1. Огненный шар (центр взрыва)
    for i = 1, 5 * scale do
        local particle = emitter:Add("particles/flamelet" .. math.random(1, 5), pos)

        if particle then
            local vel = VectorRand() * 50 * scale

            particle:SetVelocity(vel)
            particle:SetLifeTime(0)
            particle:SetDieTime(math.Rand(0.2, 0.4) * scale)
            particle:SetStartAlpha(255)
            particle:SetEndAlpha(0)
            particle:SetStartSize(10 * scale)
            particle:SetEndSize(5 * scale)
            particle:SetColor(255, 180, 100)
            particle:SetRoll(math.Rand(0, 360))
        end
    end

    -- 2. Дым (следует после огня)
    for i = 1, 5 * scale do
        local offset = VectorRand() * 5 * scale
        local particle = emitter:Add("particles/smokey", pos + offset)

        if particle then
            local vel = VectorRand() * 30 * scale + Vector(0, 0, 20 * scale)

            particle:SetVelocity(vel)
            particle:SetLifeTime(0.1)
            particle:SetDieTime(math.Rand(0.5, 1.0) * scale)
            particle:SetStartAlpha(150)
            particle:SetEndAlpha(0)
            particle:SetStartSize(10 * scale)
            particle:SetEndSize(20 * scale)
            particle:SetColor(100, 100, 100)
            particle:SetRoll(math.Rand(0, 360))
        end
    end

    -- 3. Искры (разлетаются во все стороны)
    for i = 1, 15 * scale do
        local particle = emitter:Add("effects/spark", pos)

        if particle then
            local vel = VectorRand() * 100 * scale

            particle:SetVelocity(vel)
            particle:SetLifeTime(0)
            particle:SetDieTime(math.Rand(0.3, 0.6))
            particle:SetStartAlpha(255)
            particle:SetEndAlpha(0)
            particle:SetStartSize(2 * scale)
            particle:SetEndSize(0)
            particle:SetColor(255, 200, 50)
            particle:SetGravity(Vector(0, 0, -400))
            particle:SetCollide(true)
            particle:SetBounce(0.3)
        end
    end

    emitter:Finish()

    -- 4. Добавляем звуковой эффект
    sound.Play("ambient/explosions/explode_" .. math.random(1, 4) .. ".wav", pos, 75, 100, scale * 0.5)

    -- 5. Добавляем временный динамический свет
    local light = DynamicLight(0)
    if light then
        light.Pos = pos
        light.Size = 150 * scale
        light.Decay = 1000
        light.R = 255
        light.G = 180
        light.B = 100
        light.DieTime = CurTime() + 0.2 * scale
        light.Brightness = 2
    end
end

-- Пример использования:
-- CreateMiniExplosion(LocalPlayer():GetEyeTrace().HitPos, 1.5)
```

## Стандартные эффекты и материалы

Garry's Mod включает множество встроенных эффектов и материалов, которые вы можете использовать для создания собственных эффектов.

### Популярные встроенные эффекты

Вот таблица с наиболее часто используемыми эффектами:

| Название      | Описание            | Ключевые параметры              |
| ------------- | ------------------- | ------------------------------- |
| Explosion     | Стандартный взрыв   | Origin, Scale                   |
| BloodImpact   | Брызги крови        | Origin, Normal                  |
| ElectricSpark | Электрические искры | Origin, Normal, Magnitude       |
| MuzzleEffect  | Эффект выстрела     | Origin, Angles, Scale           |
| WaterSplash   | Всплеск воды        | Origin, Normal                  |
| StriderBlood  | Кровь страйдера     | Origin, Normal                  |
| smoke_trail   | Дымовой след        | Entity, StartPos, EndPos, Scale |
| GlassImpact   | Разбитое стекло     | Origin, Normal                  |

### Полезные материалы для частиц

Когда вы создаете частицы, вам потребуются материалы для них. Вот список популярных материалов:

#### Огонь и взрывы:

- `particles/fire` - базовый огонь
- `particles/flamelet1` - `particles/flamelet5` - варианты огня
- `effects/fire_cloud1` - огненное облако
- `effects/fire_embers1` - угли и искры
- `particle/particle_glow_01` - простое свечение

#### Дым и туман:

- `particles/smokey` - стандартный дым
- `particles/smoke1` - густой дым
- `particle/particle_smokegrenade` - дым как от гранаты
- `effects/dust_puff001` - облако пыли

#### Искры и энергия:

- `effects/spark` - базовая искра
- `effects/bluespark` - синяя искра
- `effects/energysplash` - всплеск энергии
- `effects/energyball` - энергетический шар
- `effects/stunstick` - эффект оглушающей дубинки

#### Следы:

- `trails/plasma` - плазменный след
- `trails/electric` - электрический след
- `trails/smoke` - дымовой след
- `trails/laser` - лазерный след
- `trails/physbeam` - луч физпушки

## Оптимизация

Эффекты могут сильно влиять на производительность игры, особенно при большом количестве частиц. Вот несколько важных советов по оптимизации:

### Техники оптимизации эффектов

1. **Ограничивайте количество частиц**

```lua
-- Пример функции с динамическим количеством частиц
function CreateOptimizedEffect(pos, quality)
    -- Quality от 0 до 1
    quality = quality or 1

    -- Количество частиц зависит от качества
    local particleCount = math.floor(30 * quality)

    local emitter = ParticleEmitter(pos)
    for i = 1, particleCount do
        -- Создание частиц...
    end
    emitter:Finish()
end
```

2. **Используйте LOD (Level of Detail)**

```lua
function CreateDistanceBasedEffect(pos)
    -- Проверяем расстояние до игрока
    local distance = pos:Distance(LocalPlayer():GetPos())
    local quality = 1

    -- Уменьшаем детализацию с расстоянием
    if distance > 500 then
        quality = 0.75
    elseif distance > 1000 then
        quality = 0.5
    elseif distance > 2000 then
        quality = 0.25
    elseif distance > 3000 then
        -- Очень далеко, эффект не создаём
        return
    end

    -- Создаем эффект с нужным качеством
    CreateOptimizedEffect(pos, quality)
end
```

3. **Повторно используйте эмиттеры**

```lua
-- Глобальный эмиттер для повторного использования
local globalEmitter = nil

function GetSharedEmitter(pos)
    if not IsValid(globalEmitter) then
        globalEmitter = ParticleEmitter(pos)
    end

    globalEmitter:SetPos(pos)
    return globalEmitter
end
```

4. **Используйте ConVar для настройки качества**

```lua
-- Создаём переменную для настройки качества
local effectQuality = CreateClientConVar("cl_effect_quality", "1", true, false, "Качество эффектов (0-1)", 0, 1)

-- Использование переменной для настройки
function CreateQualityAwareEffect(pos)
    local quality = effectQuality:GetFloat()
    -- Создаем эффект с заданным качеством
    CreateOptimizedEffect(pos, quality)
end
```

## Полезные ссылки

Для более глубокого изучения системы эффектов вам пригодятся следующие ресурсы:

- [Официальная Wiki GLua](https://wiki.facepunch.com/gmod/) - основная документация
- [Раздел EFFECT на Wiki](https://wiki.facepunch.com/gmod/EFFECT_Hooks) - документация по системе эффектов
- [ParticleEmitter](https://wiki.facepunch.com/gmod/CLuaEmitter) - документация по эмиттеру частиц
- [EffectData](https://wiki.facepunch.com/gmod/CEffectData) - документация по данным эффектов
- [Render Functions](https://wiki.facepunch.com/gmod/~search?q=render) - функции рендеринга для продвинутых эффектов
- [DynamicLight](https://wiki.facepunch.com/gmod/Global.DynamicLight) - документация по динамическому освещению

## Заключение

Система эффектов в Garry's Mod - мощный инструмент для создания впечатляющих визуальных элементов в ваших проектах. От простых искр до сложных взрывов, анимированного огня и следов - всё это доступно при правильном использовании системы частиц.

Помните о балансе между визуальным качеством и производительностью, особенно если ваш аддон или сервер предназначен для широкой аудитории с разными компьютерами.

Экспериментируйте с параметрами, комбинируйте разные типы частиц и не бойтесь создавать уникальные эффекты - это одна из самых творческих частей разработки для Garry's Mod!

---

_Автор: darkfated_
