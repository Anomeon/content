---
title:  "Пишем первый тест для платформы Доки"
description: "Добавим первый тест в платформу Доки, разберёмся, как настроить Jest для кода, который выполняется Node.js."
authors:
  - hellsquirrel
tags:
  - article
related:
  - /tools/how-to-test-and-why
  - /tools/testing-and-fake-objects
---

## Немного очевидностей
Пишите тесты для вашего кода. Когда вы пишите тесты, вы глубже анализируете поведение вашего приложения. Тест документирует поведение вашего кода понятным вашим коллегам-разработчикам языком. Ваше приложение становится надёжным и гибким. Вы не боитесь рефакторингов. Тесты на CI позволят всей вашей команде спать спокойно. Тесты на `git pre-commit hook` не дадут запушить сломанный код в репозиторий. Зелёные галочки успокаивают.

## Как начать писать тесты?
Сначала нужно понять какие именно тесты вы хотите написать и выбрать подходящий для них фреймворк. Разобраться в тестах и фреймворках помогут эти статьи:
* [Как и зачем писать тесты](/tools/how-to-test-and-why/)
* [Фиктивные объекты и данные, моки, стабы](/tools/testing-and-fake-objects/)

Если вы не любите читать, но любите смотреть, предлагаем вам три коротких видео: [раз](https://www.loom.com/share/ed81362e0cb24a4da396419e75ceba0f), [два](https://www.loom.com/share/8a01f3821bb44ad4bea7682c99ced7a9), [три](https://www.loom.com/share/48698cd6abf947089c42b3427649a5ff). В них все что мы будем делать в этом рецепте.

Сейчас мы напишем несколько тестов, для разных кусочков [платформы доки](https://github.com/doka-guide/platform).
Для тестов будем использовать [Jest](https://jestjs.io/)

## Настраиваем Jest
У фреймворка _Jest_ есть отличная документация, всю необходимую информацию по настройке можно [найти там](https://jestjs.io/docs/getting-started)

Чтобы правильно настроить _Jest_ на платформе Доки, нужно научить его выполнять тесты для двух разных окружений:
* для браузера, там мы будем тестировать странички Доки
* для Node.js, в этом окружении мы будем тестировать сборку платформы Доки

Jest может поддерживать различные окружения, но нам понадобится специальный [трансформер](https://jestjs.io/docs/code-transformation) `babel-jest`, который поможет удобно использовать как _нативные ES модули_ так и старый-добрый _CommonJS_.

Наш файл конфигурации будет выглядеть так:
```js
module.exports = {
  testEnvironment: 'jest-environment-node',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  transform: {
    '\\.[jt]sx?$': 'babel-jest',
  },
}
```

Ключик `testEnvironment` говорит, что по умолчанию мы будем тестировать код сборки.
`setupFilesAfterEnv` запустит перед тестами специальный скрипт. Не будем подробно останавливаться на том что это за скрипт, любознательные читатели всегда могут заглянуть в [исходник](https://github.com/doka-guide/platform/blob/main/jest.setup.js).

## Запускаем тесты, которых пока нет
Чтобы запустить тесты, мы создадим специальный скрипт в файле [`package.json` нашей платформы](https://github.com/doka-guide/platform/blob/09ac9232e199f802e92c52143733edfb990180ec/package.json#L33):
```json
{
  "scripts": {
    "test": "jest"
  }
}
```

В реальных приложениях конфигурация тестов [более затейливая](https://github.com/apollographql/apollo-client/blob/78f6d27d2d926c56cefd54d6f3e2371eb7e890d1/package.json#L53), вам может понадобиться несколько скриптов для запуска разных тестов. Или вам может понадобиться запускать тесты в разных конфигурациях.

## Настало время написать наш первый тест
Мы будем тестировать функцию [форматирования заголовков](https://github.com/doka-guide/platform/blob/main/src/libs/title-formatter/title-formatter.js)

Код которой выглядит так:

```js
function titleFormatter(segments) {
  return segments.filter(Boolean).join(' — ')
}
```

Нам нужно убедиться что эта функция... форматирует заголовки :) Для этого не нужно думать, нужно просто написать тест.

Создадим папку `__tests__` где-нибудь поближе к файлу с функцией форматирования заголовков. И положим в эту папку наш первый тест.

```js
// src/libs/__tests__/title-formatter.js
import { titleFormatter } from '../title-formatter/title-formatter'

describe('titleFormatter', () => {
  it('форматирует заголовки', () => {
    const formattedTitle = titleFormatter(['test', 'test2'])
    expect(formattedTitle).toEqual('test — test2')
  })
})
```

Запустим наш тест
```bash
npm run test
```

Весёлые зелёные галочки сообщают, что все получилось.

![Зеленые галочки, которые показывают что все тесты прошли](images/pass.png)

Если вы хотите перезапускать тесты по мере изменения кода, используйте флаг `--watch`:
```bash
npm run test -- --watch
```

Возможно вы задаётесь вопросом: зачем писать тест для такой простой функции? Или думаете "Хм, написать семь строчек кода чтобы проверить однострочную функцию это не продуктивно". Представьте себе что кто-то решил изменить вашу функцию и добавить к ней ещё один параметр, например вот так:

```js
function titleFormatter(separator = ' — ', segments) {
  return segments.filter(Boolean).join(separator)
}
```

Тесты сразу же начнут падать. Это заставит ваших коллег проверить везде ли используется правильная сигнатура этой функции. Семь строк кода защитят вас от ошибки `Uncaught TypeError: Cannot read properties of undefined (reading 'filter')` в боевом приложении.


## Попробуем что-то посложнее.
Для нашего второго упражнения попробуем потестировать функционал поиска. Он живет в файлике [src/scripts/core/search-api-client.js](https://github.com/doka-guide/platform/blob/main/src/scripts/core/search-api-client.js) платформы доки. Мы будет тестировать функцию `search()`.

Давайте посмотрим, что она делает:
```js
 search(query, filters = []) {
    let url = new URL(this.url)
    let params = new URLSearchParams(url.search)
    params.append('search', query.replaceAll('+', '%2B').replaceAll('-', '%2D'))
    filters.forEach((f) => {
      params.append(f.key, f.val)
    })
    return fetch(url.toString() + '?' + params.toString(), {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        Origin: 'https://doka.guide',
      },
    }).then((response) => response.json())
  }
```

Метод `search()` использует асинхронную фукнцию `fetch()` и нам нужно учесть это в тесте. Мы уже знаем как начать: создаем папку `__tests__`, закидываем в нее `search-api-client.js`. Так как поиск асинхронный тест у нас тоже будет анисхронный.

```js
import searchClient from '../core/search-api-client.js'

describe('searchClient', () => {
    it('должен что-то искать', async () => {
        const searchResult = await searchClient.search('test')
        const expected = {
          title: 'Как и зачем писать тесты',
          link: '/tools/how-to-test-and-why/',
          category: 'tools',
        }

        expect(searchResult).toEqual(expected);
    })
})
```
Запустим наши тесты и, ожидаемо, увидим, что они падают.
![Падающие тесты](images/fetch-failing.png)

Похоже, тестирующая функция ничего не знает о существовании функции `fetch()`. Есть несколько способов решить эту проблему. Например, можно добавить в тестовое окружение полифил для функции `fetch()` и делать реальные запросы к API Доки. При этом мы не сможем запускать наши тесты в offline-режиме и будем привязаны к конкретной реализации API. Для некоторых систем это абсолютно нормально, но для нашего простого случая мы поступим иначе:
Определим функцию `fetch()` прямо внутри нашего теста.


```js
import searchClient from '../core/search-api-client.js'

describe('searchClient', () => {
  it('должен что-то искать', async () => {
    global.fetch = jest.fn(() => Promise.resolve(42))
    const searchResult = await searchClient.search('test')
    const expected = {
      title: 'Как и зачем писать тесты',
      link: '/tools/how-to-test-and-why/',
      category: 'tools',
    }

    expect(searchResult).toEqual(expected)
  })
})
```
Наша заглушка для `fetch()` всегда возвращает `Promise`, который релозолвится числом `42`. Наши тесты по-прежнему не проходят.
![Тесты продолжают падать](images/fetch-keep-failing.png)

На этот раз `Jest` не доволен значением, c которым мы релозолвим наш промис. У нас есть статья которая [подскажет что же должен возвращать `fetch()`](/js/fetch/). Прочитав ее мы уверенно правим наш тест:

```js
describe('searchClient', () => {
  it('должен что-то искать', async () => {
    const expectedResult = {
      title: 'Как и зачем писать тесты',
      link: '/tools/how-to-test-and-why/',
      category: 'tools',
    }

    const json = jest.fn(() => Promise.resolve(expectedResult))
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json,
      })
    )

    const searchResult = await searchClient.search('test')
    expect(searchResult).toEqual(expectedResult)
  })
})
```

Запускаем и видим, что он проходит.
![Тесты проходят](images/fetch-passing.png)

Осталось разобраться с двумя непонятностями:
* Что вообще мы тестируем?
* Зачем нам этот странный `jest.fn()`?

Полезное упражнение попробовать пересказать тест словами. Сейчас мы проверяем, что функция `search()` возвращает ожидаемое значение при условии что глобальная функция `fetch()` работает так как мы это определили. В текущей реализации поиск всегда будет возвращать одно и то же значение для любых запросов. Это не то, как работает поиск на самом деле.

Давайте добавим дополнительную проверку, чтобы убедиться, что мы используем правильный `URL` запроса. Заодно разберемся c [`jest.fn()`](https://jestjs.io/docs/mock-functions). Эта функция позволяет заменить (замокать) реализацию модулей или функций. Она следит за тем, сколько раз и с какими параметрами была вызвана наша функция и предлоставляет удобный доступ к этой информации. Например, мы можем проверить что вызвали `fetch()` только один раз `expect(global.fetch).toHaveBeenCalledTimes(1)`. Или посмотреть что параметр запроса передается так как нужно. Вот наш финальный тест:

```js
describe('searchClient', () => {
  it('должен что-то искать', async () => {
    const expectedResult = {
      title: 'Как и зачем писать тесты',
      link: '/tools/how-to-test-and-why/',
      category: 'tools',
    }

    const json = jest.fn(() => Promise.resolve(expectedResult))
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json,
      })
    )

    const searchResult = await searchClient.search('test')
    expect(searchResult).toEqual(expectedResult)
    expect(global.fetch.mock.calls[0][0]).toContain('search=test')
  })
})
```
И он проходит :)
![Финальный тест проходит](images/fetch-passing-2.png)


Мы постарались вас убедить, что писать тесты просто и весело. Если в вашем проекте нет тестов, попробуйте добавить хотя бы один. Через некоторое время вы будете удивляться как это вы раньше работали без тестов 🤓