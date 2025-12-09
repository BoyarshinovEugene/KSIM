## Комп'ютерні системи імітаційного моделювання
## СПм-24-2, **Бояршинов Євгеній Валентинович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 4, модель у середовищі NetLogo:
[Urban Suite - Pollution](https://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Curricular%20Models/Urban%20Suite/Urban%20Suite%20-%20Pollution.nlogo)
<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Позитивний вплив електростанцій на ймовірність появи нових людей:** поблизу power-plant народжуваність збільшується, кількість power-plant на полі підвищює ймовірність появи людей.
Було реалізовано механізм, за яким народжуваність (birth-rate) збільшується в залежності від кількості power-plant на полі, а також, у клітинах, розташованих близько до електростанцій.
Для цього:
1. Додати обчислення відстані до найближчої електростанції
Додано процедуру:
<pre>
to-report distance-to-powerplant
  let plants patches with [is-power-plant?]
  if any? plants [
    report min [distance myself] of plants
  ]
  report 999
end
</pre>
2. Додаємо процедуру для обчислення коефіцієнту підвищення народжуваності залежно від кількості електростанцій
Додано процедуру:
<pre>
to-report birth-boost-from-powerplants
  ;; формула: boost = 1 + (power-plants / 10)
  ;; при 2 power-plants → 1.2
  ;; при 10 power-plants → 2
  ;; при 20 power-plants → 3
  report 1 + (power-plants / 10)
end
</pre>
3. Змінюємо процедуру reproduce:
Змінюємо існуючу процедуру:
<pre>
to reproduce  ;; person procedure
  if health > 4 and random-float 1 < birth-rate [
    hatch-people 1 [
      set health 5
    ]
  ]
end
</pre>
на нову:
<pre>
to reproduce  ;; person procedure
  let dist distance-to-powerplant
  let boost 1

  ;; підсилення народжуваності біля електростанцій
  if dist < 5 [ set boost boost * 2 ]
  if dist < 3 [ set boost boost * 3 ]

  ;; додатковий бонус залежно від кількості power-plants
  set boost boost * birth-boost-from-powerplants

  ;; народження
  if health > 4 and random-float 1 < (birth-rate * boost) [
    hatch-people 1 [
      set health 5
      set color black
    ]
  ]
end
</pre>
Це гарантує, що люди активно з’являються саме навколо електростанцій.
 
**Збільшено вірогідність висадки дерев у клітинах поблизу електростанцій:** чим ближче до power-plant, тим більша імовірність посадки дерева.
Було реалізовано механізм, за яким народжуваність (planting-rate) збільшується у клітинах, розташованих близько до електростанцій.
Для цього:
Змінюємо процедуру maybe-plant таким чином додавая створену нову процедуру, що обчислує відстань до електростанції:
<pre>
to maybe-plant  ;; person procedure
  let dist distance-to-powerplant
  let chance planting-rate
  
  ;; збільшуємо шанс посадки дерева, якщо близько до power-plant
  if dist < 5 [ set chance planting-rate * 2 ]
  if dist < 3 [ set chance planting-rate * 3 ]

  if random-float 1 < chance [
    hatch-trees 1 [
      set health 5
      set color green
    ]
  ]
end
</pre>

<br>

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Реалізуємо логіку "міграції" людей:** якщо людина знаходится в клітині з забрудненям то вона знайде найближчу клітинку на якій не має забруднення.
Якщо людина знаходиться у зоні з високим pollution → вона намагається перейти у більш чистий сусідній патч.
Якщо всі сусідні патчі такі ж брудні або гірші → людина рухається випадково (як раніше).
Це створює поведінковий реалізм, подібний до міграції населення у забруднених районах.
Для цього:
1. Додаємо нову процедуру clean-migrate, яка перевіряє сусідні клітини на забруднення:
<pre>
to clean-migrate  ;; person procedure
  let current-pollution pollution

  ;; шукаємо всі сусідні патчі
  let neighbor-patches neighbors

  ;; обираємо патчі, де pollution менше, ніж тут
  let cleaner-patches neighbor-patches with [ pollution < current-pollution ]

  ;; якщо таких немає — еміграція неможлива, продовжуємо звичайну поведінку
  if not any? cleaner-patches [ stop ]

  ;; обираємо один найбільш чистий патч
  let best-patch min-one-of cleaner-patches [ pollution ]

  ;; рухаємось у бік цього патчу
  face best-patch
  fd 1
end
</pre>
2. Модифікуємо wander, щоб люди спочатку намагалися емігрувати
<pre>
to wander  ;; person procedure
  ;; Якщо pollution високий — намагаємось емігрувати у чистішу зону
  if pollution > 1 [
    clean-migrate
    set health health - 0.1
    stop
  ]

  ;; звичайна поведінка для “здорових” зон
  rt random-float 50
  lt random-float 50
  fd 1
  set health health - 0.1
end
</pre>
![Скріншот моделі в процесі симуляції](example-model.png)


## Обчислювальні експерименти

### 1. 
