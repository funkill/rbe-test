# Пример: map-reduce

Rust makes it very easy to parallelise data processing, without many of the headaches traditionally associated with such an attempt.

The standard library provides great threading primitives out of the box.
These, combined with Rust's concept of Ownership and aliasing rules, automatically prevent
data races.

Правила алиасинга (одна уникальная ссылка на запись или много ссылок на чтение) автоматически не позволяет вам манипулировать 
состоянием, которое видно другим потокам. (Где синхронизация необходима,
есть примитивы синхронизации, такие как `mutex` (мьютексы) или `channel` (каналы).)

In this example, we will calculate the sum of all digits in a block of numbers.
We will do this by parcelling out chunks of the block into different threads. Each thread will sum
its tiny block of digits, and subsequently we will sum the intermediate sums produced by each
thread.

Обратите внимание на то, что хоть мы и передаём ссылки через 
границы потоков, Rust понимает, что мы только передаём 
неизменяемые ссылки, которые можно только читать, и что из-за 
этого не может быть никакой небезопасности и гонок данных. Так 
как мы перемещаем (`move`) сегменты данных в 
поток, Rust также уверен, что данные будут жить до тех пор, пока 
поток не завершится, и висящих указателей не появится.

```rust,editable
use std::thread;

// This is the `main` thread
fn main() {

    // This is our data to process.
    // We will calculate the sum of all digits via a threaded  map-reduce algorithm.
    // Each whitespace separated chunk will be handled in a different thread.
    //
    // TODO: see what happens to the output if you insert spaces!
    let data = "86967897737416471853297327050364959
11861322575564723963297542624962850
70856234701860851907960690014725639
38397966707106094172783238747669219
52380795257888236525459303330302837
58495327135744041048897885734297812
69920216438980873548808413720956532
16278424637452589860345374828574668";

    // Make a vector to hold the child-threads which we will spawn.
    let mut children = vec![];

    /*************************************************************************
     * "Map" phase
     *
     * Divide our data into segments, and apply initial processing
     ************************************************************************/

    // split our data into segments for individual calculation
    // each chunk will be a reference (&str) into the actual data
    let chunked_data = data.split_whitespace();

    // Iterate over the data segments.
    // .enumerate() adds the current loop index to whatever is iterated
    // the resulting tuple "(index, element)" is then immediately
    // "destructured" into two variables, "i" and "data_segment" with a
    // "destructuring assignment"
    for (i, data_segment) in chunked_data.enumerate() {
        println!("data segment {} is \"{}\"", i, data_segment);

        // Process each data segment in a separate thread
        //
        // spawn() returns a handle to the new thread,
        // which we MUST keep to access the returned value
        //
        // 'move || -> u32' is syntax for a closure that:
        // * takes no arguments ('||')
        // * takes ownership of its captured variables ('move') and
        // * returns an unsigned 32-bit integer ('-> u32')
        //
        // Rust is smart enough to infer the '-> u32' from
        // the closure itself so we could have left that out.
        //
        // TODO: try removing the 'move' and see what happens
        children.push(thread::spawn(move || -> u32 {
            // Calculate the intermediate sum of this segment:
            let result = data_segment
                        // iterate over the characters of our segment..
                        .chars()
                        // .. convert text-characters to their number value..
                        .map(|c| c.to_digit(10).expect("should be a digit"))
                        // .. and sum the resulting iterator of numbers
                        .sum();

            // println! locks stdout, so no text-interleaving occurs
            println!("processed segment {}, result={}", i, result);

            // "return" not needed, because Rust is an "expression language", the
            // last evaluated expression in each block is automatically its value.
            result

        }));
    }


    /*************************************************************************
     * "Reduce" phase
     *
     * Collect our intermediate results, and combine them into a final result
     ************************************************************************/

    // collect each thread's intermediate results into a new Vec
    let mut intermediate_sums = vec![];
    for child in children {
        // collect each child thread's return-value
        let intermediate_sum = child.join().unwrap();
        intermediate_sums.push(intermediate_sum);
    }

    // combine all intermediate sums into a single final sum.
    //
    // we use the "turbofish" ::<> to provide sum() with a type hint.
    //
    // TODO: try without the turbofish, by instead explicitly
    // specifying the type of final_result
    let final_result = intermediate_sums.iter().sum::<u32>();

    println!("Final sum result: {}", final_result);
}


```

### Assignments

Не стоит позволять числу наших потоков быть зависимом от 
введённых пользователем данных. Что если пользователь решит 
вставить много пробелов? Мы *действительно* хотим 
создать 2000 потоков? Измените программу так, чтобы данные 
разбивались на ограниченное число блоков, объявленных 
статической константой в начале программы.

### See also:

- [Threads](../threads.md)
- [vectors](../../std/vec.md) and [iterators](../../trait/iter.md)
- [замыкания](../../fn/closures.md), [семантика передачи владения](../../scope/move.md) и [перемещения (`move`) в замыканиях](https://doc.rust-lang.org/book/ch13-01-closures.html#closures-can-capture-their-environment)
- [деструктуризация](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-to-break-apart-values) при присвоениях
- [turbofish notation](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect) to help type inference
- [unwrap vs. expect](../../error/option_unwrap.md)
- [enumerate](https://doc.rust-lang.org/book/loops.html#enumerate)