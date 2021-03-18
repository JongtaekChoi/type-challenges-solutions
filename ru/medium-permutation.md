---
id: 296
title: Permutation
lang: ru
level: medium
tags: union
---

## Проблема

Реализовать тип `Permutation<U>`, который принимает объединение типов и возвращает возможные перестановки элементов из этого объединения.
Например:

```typescript
type perm = Permutation<'A' | 'B' | 'C'>;
// expected ['A', 'B', 'C'] | ['A', 'C', 'B'] | ['B', 'A', 'C'] | ['B', 'C', 'A'] | ['C', 'A', 'B'] | ['C', 'B', 'A']
```

## Решение

Одно из моих любимых.
Эта проблема выглядит как сложная, на первый взгляд, но это не так.

Чтобы понять решение, проникнитесь одним из подходов к решению задач - ["разделяй и властвуй"](https://ru.wikipedia.org/wiki/Разделяй_и_властвуй_(информатика)).
Если решение к проблеме найти не получается, потому что проблема слишком сложная - разделяйте её и решайте проблемы поменьше.
Вместо того, чтобы искать возможные перестановки всех элементов, начнём с задач, где этих элементов нету или есть только один.

Начнём с пустого объединения.
Если объединение пустое, без элементов в нём, это значит что переставлять нечего - результатом будет пустой массив.
Иначе, исходим из того, что в объединении один элемент.
А перестановка из одного элемента - это сам элемент.
Выразим эту ситуацию, используя условные типы:

```typescript
type Permutation<T> = T extends never ? [] : [T]
```

С этим решением мы даже проходим один тест.
Тест, который проверяет перестановки одного элемента.
Всё как и планировалось!

Но, как мы можем найти перестановки по двум элементам?
Подход "разделяй и властвуй" всё ещё актуален.
Например, чтобы узнать `Permutation<'A' | 'B'>`, возьмём первый элемент `'A'` и найдём перестановки по остальным элементам, то есть `Permutation<'B'>`.
То же проделываем и со вторым элементом.
Берём элемент `'B'` и ищем перестановки по остальному объединению, то есть `Permutation<'A'>`.
Но мы же знаем как искать перестановки по объединениям с одним элементом!

В итоге, получаем простой алгоритм.
Достаём один элемент и рекурсивно ищём перестановки без этого элемента.
До тех пор, когда не доходим до базового случая рекусии - один элемент в объединении.
Рекурсия заканчивается, а благодаря вариативным типам, результат схлопывается в один кортеж.

```text
Permutation<‘A’ | ‘B’> -> [‘A’, ...Permutation<‘B’>] + [‘B’, ...Permutation<‘A’>] -> [‘A’, ‘B’] + [‘B’, ‘A’]
```

Смоделируем такое поведение на уровне системы типов.
Напомню, что условные типы в TypeScript дистрибутивные на объединениях.
Поэтому, когда вы пишете `T extends Some<T>`, где `T` это объединение, что TypeScript делает?
Он берёт каждый элемент из объединения и применяет к нему условный тип.

Воспользуемся такой особенностью, чтобы перебрать элементы из объединения.
Конструкция `T extends infer U ? U : never` будет присваивать в тип параметр `U` элемент из объединения.
А значит, мы будем знать, какой элемент исключить из рекурсии.

Заменим `[T]`, на реализацию нашего "разделяй и властвуй":

```typescript
type Permutation<T> = T extends never ? [] : T extends infer U ? [U, ...Permutation<Exclude<T, U>>] : []
```

Мы близки к решению.
В теории, это должно даже работать, но нет.
Что пошло не так?
Вместо перестановок всегда получаем `never`.
После непродолжительной отладки, я понял, что нужно обернуть наш условный тип в кортежи.

```typescript
type Permutation<T> = [T] extends [never] ? [] : T extends infer U ? [U, ...Permutation<Exclude<T, U>>] : []
```

Решение всё ещё не рабочее, но после второго захода на отладку, я нашел проблему.
Честно, я так и не понял в чем она заключалась.
В месте `T extends infer U`, в нашем случае, это не работало так, как я ожидал.
И я был откровенно удивлен, когда обычное копирование тип параметра `T` в другой тип параметр `C` решило проблему.

```typescript
type Permutation<T, C = T> = [T] extends [never] ? [] : C extends infer U ? [U, ...Permutation<Exclude<T, U>>] : []
```

## Что почитать

- [Условные типы](https://www.typescriptlang.org/docs/handbook/advanced-types.html#conditional-types)
- [Выведение типов в условных типах](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-inference-in-conditional-types)
- [Рекурсивные условные типы](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#recursive-conditional-types)
- [Вариативные типы](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html#variadic-tuple-types)