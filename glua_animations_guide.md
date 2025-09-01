# Руководство по анимациям в GLua (Garry's Mod)

Этот гайд объясняет тему **анимаций в Garry’s Mod (GLua)** на практических примерах.
Мы разберём:

- Что такое анимация и плавные переходы
- Как использовать `Lerp`, `FrameTime`, `math.Approach`
- Как делать анимации для **HUD**, **меню** и других интерфейсов
- Несколько **кейсов сложнее**: плавное появление, скейлы, слайдеры и пр.

---

## Теория

В GLua **анимация** — это постепенное изменение значения (позиции, размера, цвета и т.д.) во времени.
Мы не «телепортируем» объект из точки A в точку B, а изменяем его свойства маленькими шагами.

### Основные функции:

- `Lerp(t, from, to)` — линейная интерполяция. Позволяет получить значение между `from` и `to` при коэффициенте `t` (от `0` до `1`)
- `FrameTime()` — возвращает время, прошедшее за кадр. Используется для плавности, чтобы FPS не влиял
- `math.Approach(cur, target, step)` — плавно приближает число `cur` к `target` с указанным шагом

---

## Пример 1. Плавное изменение цвета HUD

```lua
-- Этот пример создаёт плавный переход цвета текста на HUD
-- Мы будем менять цвет от красного к зелёному и обратно

local color_target = Color(255, 0, 0) -- текущая цель цвета
local color_current = Color(255, 255, 255) -- текущий цвет текста
local state = true -- направление (true — зелёный, false — красный)

hook.Add("HUDPaint", "CustomHUD.AnimatedText", function()
    local ft = FrameTime() * 5 -- множитель скорости
    -- плавно меняем компоненты RGB
    color_current.r = Lerp(ft, color_current.r, color_target.r)
    color_current.g = Lerp(ft, color_current.g, color_target.g)
    color_current.b = Lerp(ft, color_current.b, color_target.b)

    draw.SimpleText("Hello Animation!", "Trebuchet24", ScrW() / 2, ScrH() / 2, color_current, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
end)

-- Каждые 2 секунды меняем цель цвета
timer.Create("CustomHUD.ColorSwitch", 2, 0, function()
    if state then
        color_target = Color(0, 255, 0)
    else
        color_target = Color(255, 0, 0)
    end
    state = not state
end)
```

### Объяснение:

1. Мы храним `color_current` (текущий цвет) и `color_target` (цель)
2. На каждом кадре через `Lerp` делаем маленький шаг от текущего к целевому
3. Таймер меняет цель каждые 2 секунды → цвет начинает «течь» к новой цели
4. Таким образом создаётся плавный эффект «переливания»

---

## Пример 2. Анимация появления панели меню

```lua
-- Создаём окно, которое плавно выезжает сверху вниз
-- Используем math.Approach для изменения Y позиции

concommand.Add("open_menu_anim", function()
    local frame = vgui.Create("DFrame")
    frame:SetSize(400, 300)
    frame:SetTitle("Анимированное меню")
    frame:Center()
    frame:MakePopup()

    local startY = -300 -- начальная Y позиция (за экраном)
    local targetY = ScrH() / 2 - 150 -- финальная позиция (центр экрана)
    local curY = startY

    frame.Think = function(pnl)
        curY = math.Approach(curY, targetY, 500 * FrameTime()) -- скорость
        pnl:SetPos(ScrW() / 2 - 200, curY)
    end
end)
```

### Объяснение:

1. Панель сначала создаётся за пределами экрана (`-300`)
2. В `Think` мы каждый кадр приближаем `curY` к `targetY`
3. За счёт `FrameTime()` движение одинаково плавное на 60 FPS и 200 FPS
4. Получается красивая анимация выезда меню сверху

---

## Пример 3. Плавное появление (альфа-анимация)

```lua
-- Панель плавно появляется за счёт альфа-канала (прозрачность)

concommand.Add("open_menu_fade", function()
    local frame = vgui.Create("DFrame")
    frame:SetSize(400, 300)
    frame:Center()
    frame:SetTitle("Fade меню")
    frame:MakePopup()

    frame:SetAlpha(0) -- начинаем с прозрачности 0
    local curAlpha = 0

    frame.Think = function(pnl)
        curAlpha = math.Approach(curAlpha, 255, 300 * FrameTime())
        pnl:SetAlpha(curAlpha)
    end
end)
```

### Объяснение:

- Мы изменяем прозрачность панели (`alpha`)
- Вместо резкого появления → плавный fade-in
- Можно использовать так же для исчезновения (менять цель на `0`)

---

## Пример 4. Сложный HUD с анимированным баром

```lua
-- Создаём анимированный бар здоровья (плавное уменьшение/увеличение)

local hp_smooth = 100

hook.Add("HUDPaint", "CustomHUD.HealthBar", function()
    local ply = LocalPlayer()
    if not IsValid(ply) then return end

    local hp = ply:Health()
    hp_smooth = Lerp(FrameTime() * 5, hp_smooth, hp) -- плавная интерполяция

    local w, h = 200, 25
    local x, y = 50, ScrH() - 100

    surface.SetDrawColor(50, 50, 50)
    surface.DrawRect(x, y, w, h)

    surface.SetDrawColor(200, 50, 50)
    surface.DrawRect(x, y, (hp_smooth / ply:GetMaxHealth()) * w, h)

    draw.SimpleText("HP: " .. math.Round(hp_smooth), "Trebuchet18", x + w / 2, y + h / 2, color_white, TEXT_ALIGN_CENTER, TEXT_ALIGN_CENTER)
end)
```

### Объяснение:

- `hp_smooth` всегда чуть-чуть «догоняет» реальное HP
- При получении урона полоса плавно спадает вниз, а не резко дёргается
- Классический эффект «анимированного худа»

---

## Пример 5. Расширение меню с анимацией

```lua
-- Панель-список, которая разворачивается при нажатии кнопки

concommand.Add("open_expand_menu", function()
    local frame = vgui.Create("DFrame")
    frame:SetSize(400, 500)
    frame:Center()
    frame:SetTitle("Расширяемое меню")
    frame:MakePopup()

    local panel = vgui.Create("DPanel", frame)
    panel:Dock(TOP)
    panel:SetTall(0)

    local expanded = false
    local curHeight = 0
    local targetHeight = 200

    local btn = vgui.Create("DButton", frame)
    btn:Dock(BOTTOM)
    btn:SetTall(40)
    btn:SetText("Развернуть")

    btn.DoClick = function()
        expanded = not expanded
        if expanded then
            btn:SetText("Свернуть")
        else
            btn:SetText("Развернуть")
        end
    end

    panel.Think = function(pnl)
        local goal = expanded and targetHeight or 0
        curHeight = Lerp(FrameTime() * 5, curHeight, goal)
        pnl:SetTall(curHeight)
    end
end)
```

### Объяснение:

- У нас есть скрытая панель с высотой `0`
- При нажатии кнопки она плавно разворачивается до `200 px`
- При повторном нажатии → плавно схлопывается обратно
- Такой приём часто используют для выпадающих списков

---

## Заключение

Анимации в GLua строятся на **плавных изменениях свойств** (позиция, размер, цвет, прозрачность и т.д.).
Основные инструменты:

- `Lerp` — интерполяция (плавный переход)
- `math.Approach` — приближение к цели с шагом
- `FrameTime()` — стабильная скорость на любом FPS

---

_Автор: darkfated_
