# Promise API

В классе `Promise` есть 6 статических методов.

## Promise.all

Запускает нескольких промисов параллельно.

Синтаксис:

```javascript
let promise = Promise.all(iterable);
```

Метод `Promise.all` принимает массив промисов и возвращает новый промис.

Новый промис завершится, когда завершится весь переданный список промисов, и его результатом будет массив их результатов.

Пример:

```javascript
let names = ["iliakan", "remy", "jeresig"];

let requests = names.map((name) =>
  fetch(`https://api.github.com/users/${name}`)
);

Promise.all(requests)
  .then((responses) => {
    // все промисы успешно завершены
    for (let response of responses) {
      alert(`${response.url}: ${response.status}`); // покажет 200 для каждой ссылки
    }

    return responses;
  })
  // преобразовать массив ответов response в response.json(),
  // чтобы прочитать содержимое каждого
  .then((responses) => Promise.all(responses.map((r) => r.json())))
  // все JSON-ответы обработаны, users - массив с результатами
  .then((users) => users.forEach((user) => alert(user.name)));
```

> [!IMPORTANT]  
> Если любой из промисов завершится с ошибкой, то промис, возвращённый `Promise.all`, немедленно завершается с этой ошибкой.

Например:

```javascript
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]).catch(alert); // Error: Ошибка!
```

## Promise.allSettled

Синтаксис:

```javascript
let promise = Promise.allSettled(iterable);
```

Метод `Promise.allSettled` всегда ждёт завершения всех промисов. В массиве результатов будет

- `{status:"fulfilled", value:результат}` для успешных завершений,
- `{status:"rejected", reason:ошибка}` для ошибок.

Например, мы хотели бы загрузить информацию о множестве пользователей. Даже если в каком-то запросе ошибка, нас всё равно интересуют остальные.

Пример:

```javascript
let urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/remy",
  "https://no-such-url",
];

Promise.allSettled(urls.map((url) => fetch(url))).then((results) => {
  // (*)
  results.forEach((result, num) => {
    if (result.status == "fulfilled") {
      alert(`${urls[num]}: ${result.value.status}`);
    }
    if (result.status == "rejected") {
      alert(`${urls[num]}: ${result.reason}`);
    }
  });
});
```

Массив results в строке (\*) будет таким:

```javascript
[
  {status: 'fulfilled', value: ...объект ответа...},
  {status: 'fulfilled', value: ...объект ответа...},
  {status: 'rejected', reason: ...объект ошибки...}
]
```

То есть, для каждого промиса у нас есть его статус и значение/ошибка.

## Promise.race

Метод очень похож на `Promise.all`, но ждёт только первый выполненный промис, из которого берёт результат (или ошибку).

Синтаксис:

```javascript
let promise = Promise.race(iterable);
```

Например, тут результат будет `1`:

```javascript
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]).then(alert); // 1
```

Быстрее всех выполнился первый промис, он и дал результат. После этого остальные промисы игнорируются.

## Promise.any

Метод очень похож на `Promise.race`, но ждёт только первый успешно выполненный промис, из которого берёт результат.

Если ни один из переданных промисов не завершится успешно, тогда возвращённый объект `Promise` будет отклонён с помощью `AggregateError` – специального объекта ошибок, который хранит все ошибки промисов в своём свойстве `errors`.

```javascript
let promise = Promise.any(iterable);
```

Например, здесь, результатом будет `1`:

```
Promise.any([
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Ошибка!")), 1000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```

## Promise.resolve/reject

### Promise.resolve

`Promise.resolve(value)` создаёт успешно выполненный промис с результатом `value`.

То же самое, что:

```javascript
let promise = new Promise((resolve) => resolve(value));
```

### Promise.reject

`Promise.reject(error)` создаёт промис, завершённый с ошибкой `error`.

То же самое, что:

```javascript
let promise = new Promise((resolve, reject) => reject(error));
```
