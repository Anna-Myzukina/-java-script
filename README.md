# -java-script
Вопрос: Как понять батут в JavaScript?

Вот код:

  
    function repeat(operation, num) {
    return function() {
      if (num <= 0) return
      operation()
      return repeat(operation, --num)
    }
    }

    function trampoline(fn) {
    while(fn && typeof fn === 'function') {
      fn = fn()
    }
    }

    module.exports = function(operation, num) {
    trampoline(function() {
      return repeat(operation, num)
    })
    }


Я прочиталa, что батут используется для решения проблемы переполнения, поэтому функция не просто сохранит сам вызов и стек.

Но как работает этот фрагмент? Особенно trampoline функционировать? Что именно это делало while и как он достиг своей цели?

______

while цикл будет продолжать работать, пока условие не будет ложным.

fn && typeof fn === 'function' будет ложным, если fn само по себе является ложным, или если fn это нечто иное, чем функция.

Первая половина фактически избыточна, так как значения фальшивки также не являются функциями.

_______
while цикл будет продолжать работать, пока условие не будет ложным.

fn && typeof fn === 'function' будет ложным, если fn само по себе является ложным, или если fn это нечто иное, чем функция.

Первая половина фактически избыточна, так как значения фальшивки также не являются функциями.

______________________________________________________________________________

В других ответах описывается, как работает батут. У данной реализации есть два недостатка, хотя один из них даже вреден:

Протокол трамплина зависит только от функций. Что делать, если результат рекурсивной операции также является функцией?
Вы должны применить рекурсивную функцию с функцией батута в вашем кодовом коде. Это деталь реализации, которая должна быть скрыта.
По сути, метод батута относится к ленивой оценке на нетерпеливом языке. Ниже приведен подход, позволяющий избежать упомянутых выше недостатков:

// a tag to uniquely identify thunks (zero-argument functions)

const $thunk = Symbol.for("thunk");

//  eagerly evaluate a lazy function until the final result

const eager = f => (...args) => {
  let g = f(...args);
  while (g && g[$thunk]) g = g();
  return g;
};

// lift a normal binary function into the lazy context

const lazy2 = f => (x, y) => {
  const thunk = () => f(x, y);
  return (thunk[$thunk] = true, thunk);
};

// the stack-safe iterative function in recursive style

const repeat = n => f => x => {
  const aux = lazy2((n, x) => n === 0 ? x : aux(n - 1, f(x)));
  return eager(aux) (n, x);
};

const inc = x => x + 1;

// and run...

console.log(repeat(1e6) (inc) (0)); // 1000000
Ленькая оценка происходит локально внутри repeat, Следовательно, ваш код вызова не должен беспокоиться об этом.
