> [!note|badge]
> Mock функции позволяют перехватывать ее вызовы (и передаваемые аргументы), экземпляры (если эту функцию используют как конструктор), а также настраивать возвращаемое значение.

Хотим протестировать эту функцию:

```ts
function forEach(items: any[], callback: (...args: any[]) => void) {
  for (const item of items) {
    callback(item);
  }
}
```

```ts
const mockCallback = jest.fn((x) => 42 + x);

test('forEach mock function', () => {
  forEach([0, 1], mockCallback);
  
  // The mock function was called twice
  expect(mockCallback.mock.calls).toHaveLength(2);

  // The first argument of the first call to the function was 0
  expect(mockCallback.mock.calls[0][0]).toBe(0);

  // The first argument of the second call to the function was 1
  expect(mockCallback.mock.calls[1][0]).toBe(1);

  // The return value of the first call to the function was 42
  expect(mockCallback.mock.results[0].value).toBe(42);
});
```

# `.mock` свойство mock функции

- `calls[callIndex][argIndex]` - аргументы вызовов 
- `results[callIndex].value` - возвращенное значение вызовов
- `contexts[callIndex]` - значение `this` внутри вызова
- `instances.lenght` - сколько раз функция была вызвана как конструктор 
- `instances[instanceIndex]` - экземпляры
- `lastCall[argIndex]` - последний вызов 

> [!tip|badge]
> Также сохраняется значение `this` для каждого вызова.

```ts
const myMock1 = jest.fn();
const a = new myMock1();
a.prop = 'test1';
expect(myMock1.mock.instances[0].prop).toBe('test1');

const myMock2 = jest.fn();
const b = { prop: 'test2' };
const bound = myMock2.bind(b);
bound();
console.log(myMock2.mock.contexts);
// [ { prop: 'test2' } ]
```

# Можно настраивать возвращаемое значение

```ts
const myMock = jest.fn();
console.log(myMock());
// undefined

myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);
  
console.log(myMock(), myMock(), myMock(), myMock());
// 10, 'x', true, true
```

```ts
const filterTestFn = jest.fn();

filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter((num) => filterTestFn(num));
console.log(result);
// [11]
```

# Mocking (имитация) модуля

Хотим протестировать вызов статического метода данного класса:

```ts
import axios from 'axios'; 

class Users {
  static async all() {
    const resp = await axios.get('/users.json');
    return resp.data;
  }
}

export default Users;
```

> [!tip|badge]
> Суть теста заключается в том, чтобы не делать реальный запрос к API, для этого надо использовать функцию `jest.mock` для автоматической имитации модуля axios. Затем надо имитировать реализацию функции `axios.get` с помощью метода `.mockImplementation()`:

```ts
// 1. Имитируем модуль
jest.mock('axios');

test('should fetch users', async () => {
  const users = [{ name: 'Bob' }];
  const resp = { data: users };

  // 2. Имитируем axios.get
  axios.get.mockResolvedValue(resp);
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  const data = await Users.all();
  return expect(data).toEqual(users);
});
```

> [!tip|badge]
> `.mockResolvedValue(value)` это сокращение для `.mockImplementation(() => Promise.resolve(value))`. Документация - https://jestjs.io/docs/mock-function-api#mockfnmockresolvedvaluevalue.

# Имитация подмножества модуля

> [!tip|badge]
> Можно имитировать только подмножество модуля, а остальная часть модуля может сохранять свою реальную реализацию.

```ts
export const foo = 'foo';
export const bar = () => 'bar';
export default () => 'baz';
```

```ts
import defaultExport, { bar, foo } from '../src/index';

jest.mock('../src/index', () => {
  const originalModule = jest.requireActual('../src/index');

  //Mock the default export and named export 'foo'
  return {
    __esModule: true,
    ...originalModule,
    default: jest.fn(() => 'mocked baz'),
    foo: 'mocked foo',
  };
});

test('should do a partial mock', () => {
  const defaultExportResult = defaultExport();
  expect(defaultExportResult).toBe('mocked baz');
  expect(defaultExport).toHaveBeenCalled();

  expect(foo).toBe('mocked foo');
  expect(bar()).toBe('bar');
});
```

# Подробнее про имитацию реализации mock функции (`.mockImplementation`)

> [!tip|badge]
> Как было видно в примере с модулем `axios`, `jest.mock` автоматически сделал все экспортируемые функции - mock функциями. Затем с помощью вызова `.mockImplementation` на этой функции, мы имитируем ее реализацию.

> [!tip|badge]
> Если нужно воссоздать сложное поведение mock функции так, чтобы несколько вызовов функции приводили к разным результатам, надо использовать метод `.mockImplementationOnce`:

```ts
const myMockFn = jest
  .fn()
  .mockImplementationOnce((cb) => cb(null, true))
  .mockImplementationOnce((cb) => cb(null, false));

myMockFn((err, val) => console.log(val));
// true

myMockFn((err, val) => console.log(val));
// false
```

> [!tip|badge]
> Когда реализации заданные этим методом закончатся, mock функция будет использовать реализацию по умолчанию, заданную в `jest.fn` (если она определена):

```ts
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// 'first call', 'second call', 'default', 'default'
```

> [!note|badge]
> Чтобы mock функция возвращала `this`, можно использовать метод сокращение - `.mockReturnThis`. Документация - https://jestjs.io/docs/mock-function-api#mockfnmockreturnthis.

```ts
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// просто сокращение для:
const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

# Name

# Дополнительные полезные сопоставления