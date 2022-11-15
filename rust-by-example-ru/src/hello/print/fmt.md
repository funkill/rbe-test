# Форматирование

Мы видели, что форматирование задаётся *макросом форматирования*:

- `format!("{}", foo)` -&gt; `"3735928559"`
- `format!("0x{:X}", foo)` -&gt;[`"0xDEADBEEF"`]
- `format!("0o{:o}", foo)` -&gt; `"0o33653337357"`

Одна и та же переменная (`foo`) может быть отображена по разному в зависимости от используемого *типа аргумента*: `X`, `o` или *неопределённый*.

Функционал форматирования реализован благодаря типажу, и для каждого типа аргумента существует свой. Наиболее распространённый типаж для форматирования — `Display`, который работает без аргументов: например `{}`.

```rust,editable
use std::fmt::{self, Formatter, Display};

struct City {
    name: &'static str,
    // Широта
    lat: f32,
    // Долгота
    lon: f32,
}

impl Display for City {
    // `f` — это буфер, данный метод должен записать в него форматированную строку
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        let lat_c = if self.lat >= 0.0 { 'N' } else { 'S' };
        let lon_c = if self.lon >= 0.0 { 'E' } else { 'W' };

        // `write!` похож на `format!`, только он запишет форматированную строку
        // в буфер (первый аргумент функции)
        write!(f, "{}: {:.3}°{} {:.3}°{}",
               self.name, self.lat.abs(), lat_c, self.lon.abs(), lon_c)
    }
}

#[derive(Debug)]
struct Color {
    red: u8,
    green: u8,
    blue: u8,
}

fn main() {
    for city in [
        City { name: "Дублин", lat: 53.347778, lon: -6.259722 },
        City { name: "Осло", lat: 59.95, lon: 10.75 },
        City { name: "Ванкувер", lat: 49.25, lon: -123.1 },
    ].iter() {
        println!("{}", *city);
    }
    for color in [
        Color { red: 128, green: 255, blue: 90 },
        Color { red: 0, green: 3, blue: 254 },
        Color { red: 0, green: 0, blue: 0 },
    ].iter() {
        // Поменяйте {:?} на {}, когда добавите реализацию
        // типажа fmt::Display
        println!("{:?}", *color)
    }
}
```

Вы можете посмотреть [полный список типажей форматирования] и их типы аргументов в документации к [`std::fmt`].

### Задание

Добавьте реализацию типажа `fmt::Display` для структуры `Color`, чтобы вывод отображался вот так:

```text
RGB (128, 255, 90) 0x80FF5A
RGB (0, 3, 254) 0x0003FE
RGB (0, 0, 0) 0x000000
```

Пара подсказок, если вы не знаете, что делать:

- Возможно, [вам потребуется перечислить каждый цвет несколько раз](https://doc.rust-lang.org/std/fmt/#argument-types).
- Вы можете [выровнять нулями до ширины 2] с помощью `:0>2` .

### Смотрите также:

[`std::fmt`](https://doc.rust-lang.org/std/fmt/)


[`"0xDEADBEEF"`]: https://en.wikipedia.org/wiki/Deadbeef#Magic_debug_values
[`std::fmt`]: https://doc.rust-lang.org/std/fmt/
[полный список типажей форматирования]: https://doc.rust-lang.org/std/fmt/#formatting-traits
[выровнять нулями до ширины 2]: https://doc.rust-lang.org/std/fmt/#width