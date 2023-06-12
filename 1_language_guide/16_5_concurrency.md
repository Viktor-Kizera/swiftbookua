# Конкурентність

🚧 Наразі триває переклад даного розділу. 🚧 

Swift має вбудовану підтримку написання асинхронного та паралельного коду структурованим чином. *Асинхронний код* може бути призупинений і продовжений пізніше, хоч лише одна частина програми виконується в одиницю часу. Призупинення і продовження виконання коду у вашій програмі дозволяє їй продовжувати прогрес короткострокових операцій, на кшталт оновлення інтерфейсу користувача, при цьому продовжуючи працювати над довгостроковими операціями, наприклад над викачуванням даних із мережі чи над парсингом файлів. *Паралельний код* – означає, що декілька частин коду можуть виконуватись одночасно. Наприклад, комп'ютер з чотириядерним процесором може виконувати чотири шматки коду в один і той же час, виділяючи на кожну задачу по ядру. Програми, що використовують паралельний та асинхронний код, виконують декілька операцій одночасно; вони призупиняють операції, котрі очікують на відповідь від зовнішніх систем, і це робить написання такого коду з урахуванням безпеки пам'яті легшим. 

Втім, додаткова гнучкість планування паралельного та асинхронного коду має свою ціну, що полягає у збільшеній складності. Swift дозволяє вам виражати свої наміри у спосіб, що має певні перевірки часу компіляції. Наприклад, ви можете послуговуватись акторами для безпечного звернення до змінюваного стану. Однак, додавання конкурентності до повільного чи забагованого коду не гарантує, що він стане швидшим чи коректнішим. Ймовірніше за все додавання конкурентності може навіть нашкодити, зробивши ваш код складним для зневадження. Однак, використання підтримки конкурентності на рівні мови Swift у тому вашому коді, який повинен бути конкурентним, допоможе вам знаходити проблеми під час компіляції.

Решта цього розділу буде використовувати термін *конкурентність* для позначення комбінації асинхронного і паралельного коду.

> **Примітка** 
>
> Якщо ви писали конкурентний код раніше, ви, певне, звикнути працювати з потоками. Модель конкурентності у Swift побудована поверх потоків, але ви не взаємодієте з ними напряму. Асинхронна функція у Swift може поступитись потоком, на якому вона виконується, що дозволяє іншій асинхронній функції працювати на цьому потоці допоки перша функція заблокована. Коли асинхронний код продовжиться, Swift не дає жодних гарантій щодо того, на якому потоці функція працюватиме далі.

Хоч і можливо писати конкурентний код без підтримки на рівні мови Swift, цей код як правило складніше читати. Наприклад, код нижче завантажує список назв фотографій, потім завантажує перше фото зі списку, та показує це фото користувачу:

```swift
listPhotos(inGallery: "Літня відпустка") { photoNames in
    let sortedNames = photoNames.sorted()
    let name = sortedNames[0]
    downloadPhoto(named: name) { photo in
        show(photo)
    }
}
```

Навіть у цьому простому випадку, код потрібно писати як послідовність обробників завершень, призводячи до написання вкладених замикань. У цьому стилі, складніший код із глибшим рівнем вкладення може швидко стати непідйомним. 

## Визначення та виклик асинхронних функцій

*Асинхронна функція* чи *асинхронний метод* – це особливий вид функцій чи методів, які можна призупиняти під час виконання. Це відрізняється від звичайних, тобто синхронних функцій чи методів, котрі або виконуються до завершення, або викидують помилку, або ніколи не повертають результат. Асинхронні функції чи методи також роблять одну з цих трьох речей, але вони також можуть стати на паузу посеред виконання, коли вони чогось очікують. Кожне з таких місць у тілі методу чи функції, де він або вона можуть бути призупинені, слід позначати явно. 

Щоб вказати, що функція або метод є асинхронним, використовується ключове слово `async`. Його слід вказувати в оголошенні після параметрів, аналогічно до того, як ключове слово `throws` позначає функцію, що може викидати помилку. Якщо функція чи метод повертає значення, слід писати `async` до стрілочки повернення (`->`). Наприклад, ось як можна оголосити функцію для отримання назв фотографій у галереї:

```swift
func listPhotos(inGallery name: String) async -> [String] {
    let result = // ... якийсь асинхронний код, що працює з мережею ...
    return result
}
```

Якщо функція або метод є одночасно асинхронними та викидають помилку, ключове слово `async` слід писати перед `throws`.

При виклику асинхронного методу, виконання призупиняється до того моменту, поки цей метод не поверне значення. Слід писати ключове слово `await` перед таким викликом, щоб явно відмітити можливу точку призупинення. Це схоже на написання `try` при виклику функції, що викидає помилку, для позначення можливої зміни у ході виконання програми при помилці. Всередині асинхронного методу потік виконання призупиняється *лише* при виклику іншого асинхронного методу чи функції: призупинення ніколи не є неявним або превентивним. Це означає, що кожне можливе місце призупинення є завжди відмічене ключовим словом `await`.

Наприклад, у коді нижче отримуються назви усіх фотографій у галереї й після цього показується перша фотографія:

```swift
let photoNames = await listPhotos(inGallery: "Літня відпустка")
let sortedNames = photoNames.sorted()
let name = sortedNames[0]
let photo = await downloadPhoto(named: name)
show(photo)
```

Оскільки функції `listPhotos(inGallery:)` та `downloadPhoto(named:)` обидві мають зробити мережеві запити, вони можуть виконуватись відносно тривалий час. Перетворення їх обох на асинхронні функції шляхом додавання ключового слова `async` перед стрілочкою повернення дозволяє решті коду додатка виконуватись, в той час, як цей код чекає, поки зображення не стане готовим. 

Для кращого розуміння конкурентної природи прикладу вище, розглянемо один з можливих порядків виконання:

1. Код розпочинає виконуватись з першого рядка і виконується до першого `await`. Він викликає функцію `listPhotos(inGallery:)` і призупиняє своє виконання допоки він очікує на вихід цієї функції. 
2. Допоки виконання цього коду призупинене, якийсь інший конкурентний код у цій же програмі виконується. Наприклад, можливо якесь довгоривале фонове завдання продовжує оновлювати список нових фотогалерей. Це завдання також виконується до наступної точки призупинення, позначеної за допомогою `await`, або до свого завершення. 
3. Після виходу з методу `listPhotos(inGallery:)`, код у прикладі продовжує виконуватись, починаючи з цієї точки. Він присвоює значення, що було повернуто, константі `photoNames`.
4. Рядки, що визначають константи `sortedNames` та `name`, є звичайним, синхронним кодом. Оскільки ніщо не позначено ключовим словом `await` у цих рядках, в них нема жодних можливих точок призупинення. 
5. Наступний `await` позначає виклик функції `downloadPhoto(named:)`. Цей код знову призупиняє виконання, допоки не відбудеться повернення з цієї функції, що дає можливість виконуватись конкурентному коду.
6. Після повернення з функції `downloadPhoto(named:)`, значення, котра вона повернула, присвоюється константі `photo` і після цього передається як аргумент для виклику функції `show(_:)`.

Можливі точки призупинення у вашому коді, позначені за допомогою `await`, вказують на те, що поточна частина коду може призупинити виконання, допоки вона чекає на завершення виконання асинхронної функції чи методу. Це також називають *поступкою потоком*, оскільки за лаштунками, Swift призупиняє виконання вашого коду на даному потоці, та натомість виконує якийсь інший код на цьому ж потоці. Оскільки код з `await` повинен бути здатним призупинити виконання, лише певні місця у вашій програмі можуть викликати асинхронні функції чи методи:

 - Код у тілі асинхронної функції, методу чи властивості.
 - Код у статичному методі `main()` структури, класу чи перечислення, що позначається атрибутом `@main`.
 - Код у неструктурованій дочірній задачі, як описано у розділі [Неструктурована конкурентність]({% link _book/1_language_guide/16_5_concurrency.md %}#Неструктурована-конкурентність) нижче.

Код поміж можливими точками призупинення виконується послідовно, без можливості переривання з іншого конкурентного коду. Наприклад, код нижче переносить зображення з однієї галереї до іншої. 

```swift
let firstPhoto = await listPhotos(inGallery: "Літня відпустка")[0]
add(firstPhoto, toGallery: "Поїздка")
// At this point, firstPhoto is temporarily in both galleries.
remove(firstPhoto, fromGallery: "Літня відпустка")
```

Між викликами `add(_:toGallery:)` та `remove(_:fromGallery:)` не може виконуватись жоден інший код. В цей час, перше фото `firstPhoto` одночасно знаходиться у двох галереях, що тимчасово ламає один з інваріантів нашого додатка. Щоб зробити ще більш зрозумілим те, що до цієї частини коду не можна додавати `await` у майбутньому, можна винести його в одну синхронну функцію:

```swift
func move(_ photoName: String, from source: String, to destination: String) {
    add(photoName, toGallery: destination)
    remove(photoName, fromGallery: source)
}
// ...
let firstPhoto = await listPhotos(inGallery: "Літня відпустка")[0]
move(firstPhoto, from: "Літня відпустка", to: "Поїздка")
```

У прикладі вище, оскільки функція `move(_:from:to:)` є синхронною, гарантується, що вона ніколи не буде містити можливих точок призупинення. У майбутньому, якщо ви спробуєте додати конкурентний код до цієї функції шляхом додавання точки призупинення, ви отримаєте помилку часу компіляції, а не привнесете баг. 

> **Примітка** 
>
> Метод [`Task.sleep(until:tolerance:clock:)`](https://developer.apple.com/documentation/swift/task/sleep(until:tolerance:clock:)) може бути дуже зручним при написанні простого коду для вивчення того, як працює конкурентність. Цей метод не робить нічого, але чекає як мінімум задану кількість наносекунд перед поверненням. Ось версія функції `listPhotos(inGallery:)`, що за допомогою `sleep(until:tolerance:clock:)` симулює очікування на завершення мережевої операції:
>
> ```swift
> func listPhotos(inGallery name: String) async throws -> [String] {
>     try await Task.sleep(until: .now + .seconds(2), clock: .continuous)
>     return ["IMG001", "IMG99", "IMG0404"]
> }
> ```

## Асинхронні послідовності

Функція `listPhotos(inGallery:)` у попередньому розділі асинхронно повертає весь масив одразу, після того, як всі елементи масиву готові. Іншим підходом є очікування на готовність кожного елементу колекції окремо за допомогою *асинхронної послідовності*. Ось як виглядає ітерування по асинхронній послідовності: 

```swift
import Foundation

let handle = FileHandle.standardInput
for try await line in handle.bytes.lines {
    print(line)
}
```

Замість використання звичайного циклу `for`-`in`, у прикладі вище пишемо `for` з `await` після нього. Як і при виклику асинхронної функції чи методу, написання `await` позначає можливу точку призупинення. Цикл `for`-`await`-`in` потенційно призупиняє виконання на початку кожної ітерації, очікуючи на готовність кожного наступного елементу. 

Аналогічно до того, як ви можете використовувати ваші власні типи у циклі `for`-`in` за допомогою підпорядкування їх до протоколу [`Sequence`](https://developer.apple.com/documentation/swift/sequence), ви можете використовувати ваші власні типи у циклі `for`-`await`-`in` за допомогою підпорядкування їх до протоколу [`AsyncSequence`](https://developer.apple.com/documentation/swift/asyncsequence).

## Паралельний виклик асинхронних функцій

Виклик асинхронної функції за допомогою `await` виконує лише одну частину коду в момент часу. Допоки асинхронний код виконується, ми очікуємо на його завершення перед тим, як перейти до виконання наступного рядка коду. Наприклад, щоб отримати перші три фото з галереї, можна очікувати на три виклики функції `downloadPhoto(named:)`, як написано нижче:

```swift
let firstPhoto = await downloadPhoto(named: photoNames[0])
let secondPhoto = await downloadPhoto(named: photoNames[1])
let thirdPhoto = await downloadPhoto(named: photoNames[2])

let photos = [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

Цей підхід має одну важливу ваду: хоч завантаження і є асинхронним та дозволяє іншим задачам виконуватись під час своєї роботи, у будь-який момент виконується лише один з викликів `downloadPhoto(named:)`. Кожне фото повністю завантажується перед тим, як почне завантажуватись наступне. Однак, цим операціям необов'язково очікувати: кожне фото можна завантажувати незалежно одне від одного, і навіть одночасно.

Щоб викликати асинхронну функцію і дати їй можливість виконуватись паралельно із кодом довкола неї, слід писати `async` перед `let` при визначенні константи, і потім писати `await` при кожному використанні константи. 

```swift
async let firstPhoto = downloadPhoto(named: photoNames[0])
async let secondPhoto = downloadPhoto(named: photoNames[1])
async let thirdPhoto = downloadPhoto(named: photoNames[2])

let photos = await [firstPhoto, secondPhoto, thirdPhoto]
show(photos)
```

У цьому прикладі, всі три виклики функції `downloadPhoto(named:)` розпочинаються не очікуючи на завершення попереднього. Якщо у системі доступно достатньо ресурсів, вони можуть виконуватись одночасно. Жоден з цих викликів не позначено `await`, оскільки код не призупиняється в очікуванні на результат виклику. Натомість виконання продовжується до рядка з визначенням масиву `photos`. В цей момент програма потребує результатів від цих асинхронних викликів, тому слід писати `await` для призупинення виконання допоки всі три фотографії не будуть завантажені. 

Ось як можна думати про різницю між цими двома підходами:

 - Асинхронну функцію слід викликати за допомогою `await` тоді, коли код на наступних рядках залежить від результату виконання цієї функції. This creates work that is carried out sequentially. 
 - Асинхронну функцію слід викликати за допомогою `async`-`let` тоді, коли результат її виконання не потрібен одразу, а потрібен десь пізніше. Це створює задачі, котрі можуть виконуватись паралельно. 
- Як `await`, так і `async`-`let` дозволяє виконуватись іншому коду під час призупинення.
- В обох випадках можливу точку призупинення слід відмічати за допомогою `await`, щоб позначити, що виконання може призупинитись, якщо потрібно, допоки асинхронна функція не поверне результат.

Обидва ці підходи можна змішувати в одному коді. 

## Таски та таск-групи

*Таска*, або задача – це одиниця роботи, що може виконуватись асинхронно як частина вашої програми. Весь асинхронний код виконується як частина якоїсь таски. Описаний у розділі вище синтаксис `async`-`let` створює для вас дочірню таску. Ви також можете створити таск-групу та додавати дочірні таски до цієї групи, що дає вам більше контролю щодо пріоритетів та скасування, а також дозволяє створювати динамічну кількість задач. 

Таски впорядковуються в ієрархію. Кожна таска у таск-групі має одну й ту ж батьківську таску, а кожна таска може мати дочірні таски. Оскільки зв'язок між тасками та таск-групами є явним, цей підхід називається *структурованою конкурентністю*. Хоч на вас і лежить певна частина відповідальності за коректність коду, явні ієрархічні зв'язки між тасками дозволяють Swift керувати певними поведінками на кшталт поширення скасування за вас, і дозволяє Swift відловлювати деякі помилки під час компіляції. 

```swift
await withTaskGroup(of: Data.self) { taskGroup in
    let photoNames = await listPhotos(inGallery: "Літня відпустка")
    for name in photoNames {
        taskGroup.addTask { await downloadPhoto(named: name) }
    }
}
```

Детальніше з таск-групами можна ознайомитись в документації: [`TaskGroup`](https://developer.apple.com/documentation/swift/taskgroup).

### Неструктурована конкурентність

На додачу до описаних у розділах вище структурованих підходів до конкурентності, Swift також підтримує неструктуровану конкурентність. На відміну від тасків, що є частиною таск-групи, *неструктурована задача* не має батьківської задачі. Ви маєте повну гнучкість в управлінні неструктурованими тасками у будь-який потрібний у вашій програмі спосіб, але ви також повністю відповідаєте за їх коректність. Щоб створити неструктуровану таску, що виконується на поточному акторі, слід викликати ініціалізатор [`Task.init(priority:operation:)`](https://developer.apple.com/documentation/swift/task/3856790-init). Щоб створити неструктуровану таску, що не є частиною поточного актора, більш відому як *відокремлена таска*, слід викликати метод класу [`Task.detached(priority:operation:)`](https://developer.apple.com/documentation/swift/task/3856786-detached). Обидві ці операції повертають таску, з якою ви можете взаємодіяти: наприклад, почекати на її результат або скасувати її.

```swift
let newPhoto = // ... some photo data ...
let handle = Task {
    return await add(newPhoto, toGalleryNamed: "Весняні пригоди")
}
let result = await handle.value
```

Більше інформації про керування відокремленими тасками можна знайти у документації: [`Task`](https://developer.apple.com/documentation/swift/task).

### Скасування тасків

Конкурентність у Swift використовує кооперативну модель скасування. Кожна таска перевіряє, чи не було його скасовано у доречні моменти під час його виконання та відповідає на скасування у будь-який доречний спосіб. В залежності від того, яку роботу виконує таска, це як правило означає одне з наступного:

 - Викидання помилки на кшталт `CancellationError`
 - Повернення `nil` або порожньої колекції
 - Повернення частково завершеної роботи

Щоб перевірити чи не було скасовано поточну таску, слід викликати метод [`Task.checkCancellation()`](https://developer.apple.com/documentation/swift/task/3814826-checkcancellation), котрий викидає помилку `CancellationError` у випадку, якщо задачу було скасовано, або перевірити значення [`Task.isCancelled`](https://developer.apple.com/documentation/swift/task/3814832-iscancelled), та опрацювати скасування у вашому коді. Наприклад, таска, що завантажує фото з галереї, може потребувати видалення частково завантажених фотографій та закриття мережевих з'єднань. 

Щоб поширити скасування вручну, слід викликати метод [`Task.cancel()`](https://developer.apple.com/documentation/swift/task/3851218-cancel).

## Актори

Ви можете використовувати таски для розділення вашої програми на ізольовані, конкурентні частини. Таски є ізольованими одна від одної, і саме це робить безпечним їх одночасне виконання. Втім, іноді вам може бути потрібно передавати інформацію між тасками. Актори дозволяють вам безпечно ділитись інформацією між конкурентним кодом. 

Як і класи, актори є типами-посиланнями, тому порівняння типів-значень та типів-посилань у розділі [Класи як типи-посилання]({% link _book/1_language_guide/8_classes_and_structures.mdк%}#класи-як-типи-посилання) є справедливим і для них. На відміну від класів, актори дозволяють звертатись до їх змінюваного стану лише одній тасці за раз, що дозволяє для коду в кількох тасках взаємодіяти з одним і тим же екземпляром актору безпечно. Наприклад, ось актор, що записує значення вимірювання температури:

```swift
actor TemperatureLogger {
    let label: String
    var measurements: [Int]
    private(set) var max: Int

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }
}
```

Актори оголошуються за допомогою ключового слова `actor`, після чого йде його назва та оголошення у парі фігурних дужок. Актор `TemperatureLogger` має властивості, до котрих може звертатись код ззовні актора, та обмежує властивість `max` таким чином, що лише код всередині актора може оновлювати максимальне значення. 

Екземпляр актора створюється за допомогою такого ж синтаксису ініціалізації, як і для класів та структур. При доступі до властивості чи методу актора, слід використовувати `await` для позначення потенційної точки призупинення. Наприклад:

```swift
let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
print(await logger.max)
// Надрукує "25"
```

У цьому прикладі, доступ до `logger.max` є можливою точкою призупинення. Оскільки актор дозволяє лише одній задачі за раз звертатись до свого змінюваного стану, якщо код з іншої таски вже взаємодіє з `logger`, цей код повинен призупинитись в очікуванні доступу до властивості. 

Натомість код, що є частиною актора, не повинен використовувати `await` для доступу до властивостей актора. Наприклад, ось метод, що оновлює `TemperatureLogger` з новим виміром температури:

```swift
extension TemperatureLogger {
    func update(with measurement: Int) {
        measurements.append(measurement)
        if measurement > max {
            max = measurement
        }
    }
}
```

The `update(with:)` method is already running on the actor, so it doesn't mark its access to properties like `max` with `await`. This method also shows one of the reasons why actors allow only one task at a time to interact with their mutable state: Some updates to an actor's state temporarily break invariants. The `TemperatureLogger` actor keeps track of a list of temperatures and a maximum temperature, and it updates the maximum temperature when you record a new measurement. In the middle of an update, after appending the new measurement but before updating `max`, the temperature logger is in a temporary inconsistent state. Preventing multiple tasks from interacting with the same instance simultaneously prevents problems like the following sequence of events:

1. Your code calls the `update(with:)` method. It updates the `measurements` array first.
2. Before your code can update `max`, code elsewhere reads the maximum value and the array of temperatures.
3. Your code finishes its update by changing `max`.

In this case, the code running elsewhere would read incorrect information because its access to the actor was interleaved in the middle of the call to `update(with:)` while the data was temporarily invalid. You can prevent this problem when using Swift actors because they only allow one operation on their state at a time, and because that code can be interrupted only in places where `await` marks a suspension point. Because `update(with:)` doesn't contain any suspension points, no other code can access the data in the middle of an update.

If you try to access those properties from outside the actor, like you would with an instance of a class, you'll get a compile-time error. For example:

```swift
print(logger.max)  // Error
```

Accessing `logger.max` without writing `await` fails because the properties of an actor are part of that actor's isolated local state. Swift guarantees that only code inside an actor can access the actor's local state. This guarantee is known as *actor isolation*.

## Sendable Types

Tasks and actors let you divide a program into pieces that can safely run concurrently. Inside of a task or an instance of an actor, the part of a program that contains mutable state, like variables and properties, is called a *concurrency domain*. Some kinds of data can't be shared between concurrency domains, because that data contains mutable state, but it doesn't protect against overlapping access.

A type that can be shared from one concurrency domain to another is known as a *sendable* type. For example, it can be passed as an argument when calling an actor method or be returned as the result of a task. The examples earlier in this chapter didn't discuss sendability because those examples use simple value types that are always safe to share for the data being passed between concurrency domains. In contrast, some types aren't safe to pass across concurrency domains. For example, a class that contains mutable properties and doesn't serialize access to those properties can produce unpredictable and incorrect results when you pass instances of that class between different tasks.

You mark a type as being sendable by declaring conformance to the `Sendable` protocol. That protocol doesn't have any code requirements, but it does have semantic requirements that Swift enforces. In general, there are three ways for a type to be sendable:

- The type is a value type, and its mutable state is made up of other sendable data --- for example, a structure with stored properties that are sendable or an enumeration with associated values that are sendable.
- The type doesn't have any mutable state, and its immutable state is made up of other sendable data --- for example, a structure or class that has only read-only properties.
- The type has code that ensures the safety of its mutable state, like a class that's marked `@MainActor` or a class that serializes access to its properties on a particular thread or queue.

For a detailed list of the semantic requirements, see the [`Sendable`](https://developer.apple.com/documentation/swift/sendable) protocol reference.

Some types are always sendable, like structures that have only sendable properties and enumerations that have only sendable associated values. For example:

```swift
struct TemperatureReading: Sendable {
    var measurement: Int
}

extension TemperatureLogger {
    func addReading(from reading: TemperatureReading) {
        measurements.append(reading.measurement)
    }
}

let logger = TemperatureLogger(label: "Tea kettle", measurement: 85)
let reading = TemperatureReading(measurement: 45)
await logger.addReading(from: reading)
```

Because `TemperatureReading` is a structure that has only sendable properties, and the structure isn't marked `public` or `@usableFromInline`, it's implicitly sendable. Here's a version of the structure where conformance to the `Sendable` protocol is implied:

```swift
struct TemperatureReading {
    var measurement: Int
}
```
