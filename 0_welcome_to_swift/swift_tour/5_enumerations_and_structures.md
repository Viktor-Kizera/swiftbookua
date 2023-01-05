---
title: Перечислення і Структури
layout: default
parent: Тур по Swift
grand_parent: Ласкаво просимо до Swift
nav_order: 5
has_children: false
has_toc: false
---

# Перечислення і Структури

Щоб створити перечислення, використовуємо ключове слово `enum`. Як класи та всі інші іменовані типи, перечислення можуть мати асоційовані з ними методи.

```swift
// Ранг
enum Rank: Int {
    case ace = 1                                                // Туз
    case two, three, four, five, six, seven, eight, nine, ten   // два, три, чотири, п'ять, шість, сім, вісім, дев'ять, десять
    case jack, queen, king                                      // валет, дама, король
    func simpleDescription() -> String {
        switch self {
        case .ace:
            return "туз"
        case .jack:
            return "валет"
        case .queen:
            return "дама"
        case .king:
            return "король"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.ace
let aceRawValue = ace.rawValue
```

> **Експеримент**
>
> Створіть функцію, що порівнює два елементи `Rank` порівнюючи їх сирі значення \(`rawValue`\).

За замовчанням, мова Swift присвоює сирі значення починаючи з нуля і збільшує їх щоразу на одиницю, але цю поведінку можна змінити, явно вказавши значення. У прикладі вище, випадку `Ace` явно присвоюється сире значення`1`, а інші сирі значення присвоюються далі по черзі. Можна використовувати рядки чи числа з рухомою комою як тип сирого значення перечислення. Щоб отримати сире значення, використовуємо властивість `rawValue`.

Щоб створити екземпляр перечислення із сирого значення, використовуємо ініціалізатор `init?(rawValue:)`.

```swift
if let convertedRank = Rank(rawValue: 3) {
    let threeDescription = convertedRank.simpleDescription()
}
```

Елемент перечислення є значенням сам по собі, а не просто способом запису сирого значення. Фактично, у випадку, коли немає виразного сирого значення, не обов'язково його вказувати.

```swift
// Масть
enum Suit {
    case spades, hearts, diamonds, clubs    // піка, чирва, бубна, трефа
    func simpleDescription() -> String {
        switch self {
        case .spades:
            return "spades"
        case .hearts:
            return "hearts"
        case .diamonds:
            return "diamonds"
        case .clubs:
            return "clubs"
        }
    }
}
let hearts = Suit.hearts
let heartsDescription = hearts.simpleDescription()
```

> **Експеримент**
>
> Додайте метод `color()` до `Suit`, котрий поверне “чорний” для піки та трефи, і "червоний" для бубни і чирви.

Зверніть увагу на два способи, якими звертаємось вище до елементу перечислення `hearts`. Коли константі `hearts` просвоюється значення, до елементу перечислення `Suit.hearts` звертаємось за допомогою повного імені, бо константа інакше константа не буде мати явно вказаного типу. Всередині інструкції `switch` до елементу перечислення звертаємось за допомогою скороченої форми `.hearts`, бо відомо, що значення `self` має тип `Suit`. Можемо використовувати скорочену форму щоразу коли тип значення вже відомий.

Якщо перечислення має сирі значення, ці значення визначаються в оголошенні, тому кожен екземпляр певного випадку перечислення завжди має одне і те ж саме значення. Альтернативою для випадків перечислення є асоційовані значення. Ці значення визначаються під час створення екземпляру, і вони можуть бути різними для кожного екземпляру випадку перечислення. Можете вважати, що асоційовані значення поводяться як властивості, що зберігаються, випадку перечислення. Наприклад, уявімо собі сервер, що повідомляє час сходу і заходу сонця. Сервер може у відповідь на запит надати або інформацію про схід і захід, або опис помилки.

```swift
enum ServerResponse {
    case result(String, String)
    case failure(String)
}

let success = ServerResponse.result("6:00 am", "8:09 pm")
let failure = ServerResponse.failure("Out of cheese.")

switch success {
case let .result(sunrise, sunset):
    print("Sunrise is at \(sunrise) and sunset is at \(sunset).")
case let .failure(message):
    print("Failure...  \(message)")
}
```

> **Експеримент**
>
> Додайте третій випадок до перечислення `ServerResponse` і до перемикача `switch`

Помітимо, як час сходу і заходу витягується зі значення `ServerResponse` як частина зіставлення значення у перемикачі.

Щоб створити структури використовуємо `struct`. Структури підтримують багато можливостей класів, в тому числі методи та ініціалізатори. Одна з найбільш важливих відмінностей між структурами та класами полягає в тому, що структури завжди копіюються при передачі у коді, в той час як класи передаються за посиланням.

```swift
struct Card {
    var rank: Rank
    var suit: Suit
    func simpleDescription() -> String {
        return "The \(rank.simpleDescription()) of \(suit.simpleDescription())"
    }
}
let threeOfSpades = Card(rank: .three, suit: .spades)
let threeOfSpadesDescription = threeOfSpades.simpleDescription()
```

> **Експеримент**
>
> Додайте до `Card` метод, що створює повну колоду карт, тобто по одній карті кожної масті і рангу.

