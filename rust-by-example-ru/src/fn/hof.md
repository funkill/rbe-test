# Функции высшего порядка

Rust предоставляет Функции Высшего Порядка (ФВП). Это функции, которые получают на вход одну или несколько функций и/или выдают более полезную функцию. ФВП и ленивые итераторы придают языку Rust функциональный оттенок.

```rust,editable
fn is_odd(n: u32) -> bool {
    n % 2 == 1
}

fn main() {
    println!("Найти сумму всех квадратов нечётных чисел не больше 1000");
    let upper = 1000;

    // Императивный подход
    // Объявляем переменную-накопитель
    let mut acc = 0;
    // Итерируем: 0, 1, 2, ... до бесконечности
    for n in 0.. {
        // Возводим число в квадрат
        let n_squared = n * n;

        if n_squared >= upper {
            // Останавливаем цикл, если превысили верхний лимит
            break;
        } else if is_odd(n_squared) {
            // Прибавляем число, если оно нечётное
            acc += n_squared;
        }
    }
    println!("императивный стиль: {}", acc);

    // Функциональный подход
    let sum_of_squared_odd_numbers: u32 =
        (0..).map(|n| n * n)             // Все натуральные числа возводим в квадрат
             .take_while(|&n_squared| n_squared < upper) // Берём те, что ниже верхнего предела
             .filter(|&n_squared| is_odd(n_squared))     // Выбираем нечётные
             .sum(); // Складываем
    println!("функциональный стиль: {}", sum_of_squared_odd_numbers);
}
```

[Option] и [Iterator] реализуют значительную часть функций высшего порядка..


[Option]: https://doc.rust-lang.org/core/option/enum.Option.html
[Iterator]: https://doc.rust-lang.org/core/iter/trait.Iterator.html
