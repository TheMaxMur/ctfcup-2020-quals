# Отборочный этап Кубка CTF России 2020

## Crypto | Alone

### Описание

> Вино и сигареты — это всё, что нам осталось...

### Хинт

> Не нужно пытаться решать задачу о рюкзаке "в лоб". Попробуйте придумать, как восстановить секретные параметры (`t`, `g` и `d`), зная только массив `key`.

### Выдаваемые файлы

- [alone.sage](task/alone.sage)
- [output.txt](task/output.txt)

### Решение

В задании реализована [криптосистема Шора-Ривеста](https://ru.wikipedia.org/wiki/Ранцевая_криптосистема_Шора-Ривеста), основанная на [задаче о рюкзаке](https://ru.wikipedia.org/wiki/Задача_о_рюкзаке), точнее на частном её случае — [задаче о сумме подмножеств](https://ru.wikipedia.org/wiki/Задача_о_сумме_подмножеств).

- `{m[0], m[1], ..., m[p-1]}, m[i] in {0, 1}` — биты нашего сообщения, а также флаги принадлежности к подмножеству
- `{key[0], key[1], ..., key[p-1]}, key[i] in {0, ..., p^h}` — элементы множества
- `sum(m[i] * key[i] for i in {0, ..., p-1}) = ciphertext in {0, ..., p^h}` — сумма подмножества

Мы знаем сумму подмножества, но не знаем конкретные элементы, которые были просуммированы. Если мы решим эту задачу, то есть восстановим нужные элементы, то мы восстановим биты сообщения и следовательно расшифруем флаг. Для начала проверим, сможем ли мы решить рюкзак "в лоб". Для этого посчитаем плотность упаковки рюкзака:

`density = p / log2(p^h) = 2.3587034204437374`

Плотность сильно больше `1`, это значит, что известные алгоритмы для решения рюкзака, основанные на [LLL](https://ru.wikipedia.org/wiki/Алгоритм_Ленстры_—_Ленстры_—_Ловаса) (такие как LO и CJLOSS), здесь не сработают. Тогда попробуем поискать уязвимости в реализации криптосистемы.

Уязвимость находится в генерации ключа, а именно в следующей строке:

```python
c = [(d + (t + i).log(g)) % q for i in range(p)]
```

Можно заметить, что используется тождественная перестановка, а значит, что мы можем реализовать атаку известной перестановки (описанную в упомянутой статье Википедии или в [исследовании Serge Vaudenay](https://dx.doi.org/10.1007/BFb0055732)). 

В решении используется алгоритм LLL, при этом размерность матрицы может получиться достаточно большой, поэтому стоит уменьшить количество элементов при восстановлении `t`.

__Пример решения: [solver.sage](task/solver.sage)__

### Флаг

`ctfcup{pr41s3_th3_LLL_4lg0r1thm}`
