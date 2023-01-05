---
title: Обробка Помилок
layout: default
parent: Тур по Swift
grand_parent: Ласкаво просимо до Swift
nav_order: 7
has_children: false
has_toc: false
---

# Обробка Помилок

Помилки представляємо як будь-який тип, що відповідає до протоколу `Error`.

```swift
enum PrinterError: Error {
    case outOfPaper
    case noToner
    case onFire
}
```

Використовуємо `throw` щоб викинути помилку, і `throws` щоб позначити функцію, котра може викинути помилку. Якщо викинути помилку всередині функції, одразу відбувається вихід із функції, і код, котрий викликав функцію, обробляє помилку.

```swift
func send(job: Int, toPrinter printerName: String) throws -> String {
    if printerName == "Never Has Toner" {
        throw PrinterError.noToner
    }
    return "Job sent"
}
```

Існує декілька способі обробки помилок. Одним із них є конструкція `do`-`catch`. Всередині блоку `do` позначаємо код, що може викинути помилку, написавши перед ним `try`. Всередині блоку `catch` помилці автоматично надається ім'я `error`, окрім випадку, коли ви явно надаєте їй інше ім'я.

```swift
do {
    let printerResponse = try send(job: 1040, toPrinter: "Bi Sheng")
    print(printerResponse)
} catch {
    print(error)
}
```

> **Експеримент**
>
> Змініть ім'я принтеру на `"Never Has Toner"`, щоб функція `send(job:toPrinter:)` викинула помилку.

Можна вказати кілька блоків `catch` щоб обробляти різні види помилок по-різному. В блоці `catch` вказується точно такий же патерн, що і в блоці `case` в перемикачі `switch`.

```swift
do {
    let printerResponse = try send(job: 1440, toPrinter: "Gutenberg")
    print(printerResponse)
} catch PrinterError.onFire {
    print("I'll just put this over here, with the rest of the fire.")
} catch let printerError as PrinterError {
    print("Printer error: \(printerError).")
} catch {
    print(error)
}
```

> **Експеримент**
>
> Додайте код, який викидує помилку всередині блоку `do`. Який вид помилки потрібно викинути, щоб помилка оброблялась в першому блоці `catch`? У другому блоці? У третьому блоці?

Іншим способом обробки помилок є конструкція `try?`, що перетворює результат на опціонал. Якщо функція викинула помилку, специфічне значення помилки відкидається, а результатом виклику буде `nil`. В іншому випадку результатом буде опціонал, що містить значення, котре повернула функція.

```swift
let printerSuccess = try? send(job: 1884, toPrinter: "Mergenthaler")
let printerFailure = try? send(job: 1885, toPrinter: "Never Has Toner")
```

Інструкцію `defer` зручно використовувати, щоб створити блок коду, котрий виконається після всього іншого коду функції, і перед тим, як відбудеться вихід із функції. Даний код буде виконано незалежно від того, викинула функція помилку чи ні. `defer` зручно використовувати, щоб розмістити код налаштування і код очищення поруч, всупереч тому, що ці блоки коду буде виконано в різний час.

```swift
var fridgeIsOpen = false
let fridgeContent = ["milk", "eggs", "leftovers"]

func fridgeContains(_ food: String) -> Bool {
    fridgeIsOpen = true
    defer {
        fridgeIsOpen = false
    }

    let result = fridgeContent.contains(food)
    return result
}
fridgeContains("banana")
print(fridgeIsOpen)
```

