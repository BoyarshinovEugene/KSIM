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
Було реалізовано механізм, за яким народжуваність (plantiong-rate) збільшується у клітинах, розташованих близько до електростанцій.
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



![Скріншот моделі в процесі симуляції](example-model.png)


## Обчислювальні експерименти
*// тут повинен бути наведений опис одного експерименту, за аналогією з першої л/р.* 
### 1. 
