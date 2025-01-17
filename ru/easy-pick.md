---
id: 4
title: Pick
lang: ru
level: easy
tags: union built-in
---

## Проблема

Реализовать встроенный `Pick<T, K>` не используя его.
Этот тип возвращает новый тип, в котором перечисляются только те свойства из `T`, которые указаны в `K`.
Например:

```typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = MyPick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

## Решение

Чтобы решить эту проблему, нам сильно помогут типы поиска и сопоставляющие типы.

Типы поиска извлекают тип из другого типа по имени.
Это похоже на получение значения из объекта по ключу.
Например, нас интересует какого типа `description` в интерфейсе `Todo`.
Мы можем легко его получить используя типы поиска - `Todo['description']` вернёт `string`.

Сопоставляющие типы перебирают ключи в интерфейсе или объекте и заменяют их на новый тип.
Например, мы можем сделать все ключи в `Todo` неизменяемыми.
Для этого достаточно использовать конструкцию `{ readonly [K in keyof Todo]: Todo[K] }`.

Зная, что у TypeScript есть типы поиска и сопоставляющие типы, мы можем реализовать `Pick<T, K>`.
Для решения нам нужно взять все элементы из `K` и вернуть новый тип, в котором ключами будут только поля из `K`.
Как раз то, что делают сопоставляющие типы.

А значениями этих ключей будут типы из `T` без изменений.
Чтобы взять типы отдельно взятых свойств из `T`, воспользуемся типами поиска.

```typescript
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
```

Таким образом, мы говорим "возьми всё, что есть в `K`, сопоставь это к `P` и сделай ключом нашего нового объекта, а значение возьми без изменений из `T`".

## Что почитать

- [Типы поиска](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#keyof-and-lookup-types)
- [Сопоставляющие типы](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
