# Шпаргалка по SageMath: Еліптичні криві над скінченними полями

## 1. Створення поля та кривої

```python
# Скінченне поле F_q
F = GF(17)

# Еліптична крива y² = x³ + ax + b
E = EllipticCurve(F, [3, 2])    # a=3, b=2
print(E)
# Elliptic Curve defined by y^2 = x^3 + 3*x + 2 over Finite Field of size 17

# Дискримінант (Δ ≠ 0 — інакше не еліптична крива)
print(E.discriminant())
```

## 2. Порядок кривої

```python
n = E.order()
print(n)    # 14

# Перевірка нерівності Гассе: |n - (q+1)| ≤ 2√q
q = 17
print(abs(n - (q + 1)), "<=", 2*sqrt(RR(q)))
# 4 <= 8.2462...
```

## 3. Всі точки кривої

```python
pts = E.points()
print(len(pts))   # = порядку кривої

for P in pts:
    print(P)
# (0 : 1 : 0)      ← точка на нескінченності O
# (0 : 7 : 1)
# (3 : 4 : 1)
# ...
```

Точка на нескінченності:
```python
O = E(0)           # або E(0, 1, 0)
print(O)           # (0 : 1 : 0)
```

## 4. Арифметика точок

```python
P = E(3, 4)
Q = E(0, 7)

# Додавання
R = P + Q
print(R)

# Подвоєння
print(2 * P)

# Скалярний добуток
print(5 * P)

# Обернена точка
print(-P)

# Перевірка: nP = O
print(E.order() * P)    # (0 : 1 : 0)
```

## 5. Порядок точки

```python
P = E(3, 4)
print(P.order())    # 7

# Знайти генератор (точку максимального порядку)
for P in E.points():
    if P != E(0):
        print(P, "ord =", P.order())
```

## 6. Структура групи

```python
G = E.abelian_group()
print(G)
# Additive abelian group isomorphic to Z/14

inv = G.invariants()
print(inv)    # (14,) — циклічна група

# Якщо група не циклічна:
E2 = EllipticCurve(GF(17), [1, 7])
print(E2.abelian_group())
# Additive abelian group isomorphic to Z/2 + Z/10

# Генератори
gens = G.gens()
for g in gens:
    print(g, "порядок", g.order())
```

Інтерпретація `invariants()`:
- `(n,)` — циклічна: E(F_q) ≅ Z/nZ
- `(n₁, n₂)` — E(F_q) ≅ Z/n₁Z ⊕ Z/n₂Z, причому n₁ | n₂

## 7. j-інваріант

```python
j = E.j_invariant()
print(j)    # 2

# Ручне обчислення: j = 1728 · 4a³ / (4a³ + 27b²)
a, b = 3, 2
disc = F(4)*F(a)^3 + F(27)*F(b)^2
j_manual = F(1728) * F(4) * F(a)^3 / disc
print(j_manual)
```

Властивості j-інваріанта:
- Дві криві ізоморфні ⟺ однаковий j-інваріант (над алгебраїчним замиканням)
- j = 0: крива y² = x³ + b (a = 0) — має додаткові автоморфізми
- j = 1728: крива y² = x³ + ax (b = 0) — має додаткові автоморфізми

## 8. Всі криві з даним j-інваріантом

```python
# Побудова кривої з заданим j-інваріантом
E_from_j = EllipticCurve_from_j(F(2))
print(E_from_j)
```

## 9. Ізоморфізми між кривими

Дві криві над F_q ізоморфні, якщо мають однаковий j-інваріант **і** пов'язані перетворенням:
- x' = u²x, y' = u³y
- a' = u⁴a, b' = u⁶b

```python
E1 = EllipticCurve(F, [3, 2])
u = F(2)
a2 = u^4 * F(3)    # = 16·3 mod 17
b2 = u^6 * F(2)    # = 64·2 mod 17
E2 = EllipticCurve(F, [a2, b2])

print(E1.j_invariant(), E2.j_invariant())   # однакові

# Перевірка ізоморфізму (Sage)
print(E1.is_isomorphic(E2))   # True

# Отримати явний ізоморфізм
iso = E1.isomorphism_to(E2)
print(iso)

# Застосування ізоморфізму до точки
P = E1(3, 4)
P2 = iso(P)
print(P2)
print(P2 in E2.points())    # True
```

Перебір всіх можливих u:
```python
F = GF(17)
for u in F:
    if u != 0:
        a2 = u^4 * F(3)
        b2 = u^6 * F(2)
        print(f"u={u}: a'={a2}, b'={b2}")
```

## 10. Криві кручення (twists)

Квадратичне кручення: якщо d — квадратичний нелишок в F_q, то крива
y² = x³ + d²·a·x + d³·b є кручення E.

```python
E = EllipticCurve(F, [3, 2])

# Знайти квадратичний нелишок
d = None
for x in F:
    if x != 0 and not x.is_square():
        d = x
        break
print("Нелишок d =", d)    # наприклад, 3

# Побудова кручення
Et = EllipticCurve(F, [d^2 * F(3), d^3 * F(2)])
print(Et)
print("#E =", E.order(), ", #E_t =", Et.order())
print("Сума:", E.order() + Et.order(), "= 2(q+1) =", 2*(17+1))
# #E + #E_t = 2(q+1) — завжди!
```

Альтернативно — вбудована функція:
```python
Et = E.quadratic_twist()
print(Et)
print(E.is_isomorphic(Et))    # False (якщо j ≠ 0, 1728)
```

Всі кручення кривої (до 6 для j=0, до 4 для j=1728, 2 для решти):
```python
twists = E.twists()
for Ei in twists:
    print(Ei, " j =", Ei.j_invariant(), " #E =", Ei.order())
```

## 11. Точки кручення (torsion)

```python
# n-кручення: точки P такі що nP = O
E = EllipticCurve(GF(13), [2, 5])

# 2-кручення (точки порядку 1 або 2)
R.<x> = GF(13)[]
cubic = x^3 + 2*x + 5
print(cubic.roots())    # корені → точки (r, 0)

# Точки порядку 2 на кривій
for P in E.points():
    if P.order() == 2:
        print(P)

# Підгрупа n-кручення
print(E.torsion_order())    # порядок підгрупи кручення (= порядку кривої для F_q)
```

Поліноми ділення:
```python
# n-й поліном ділення
E = EllipticCurve(GF(17), [3, 2])
psi = E.division_polynomial(3)
print(psi)
print(psi.roots())    # x-координати точок 3-кручення
```

## 12. Перетворення у форму Монтгомері

Форма Монтгомері: By² = x³ + Ax² + x, де B(A² − 4) ≠ 0.

**Необхідна умова**: крива має точку порядку 2 (корінь x³+ax+b), причому
для обраного кореня r значення 3r²+a має бути квадратичним лишком.

```python
F = GF(17)
a, b = 3, 2
E = EllipticCurve(F, [a, b])

# Крок 1: Знайти корені x³ + ax + b
R.<x> = F[]
cubic = x^3 + a*x + b
roots = cubic.roots()
print("Корені:", roots)
print("Факторизація:", cubic.factor())

# Крок 2: Обрати корінь r, де 3r²+a — квадратичний лишок
r = None
for root, mult in roots:
    val = 3*root^2 + F(a)
    print(f"  r={root}: 3r²+a = {val}, QR = {val.is_square()}")
    if val != 0 and val.is_square():
        r = root
        break

if r is None:
    print("Перетворення у форму Монтгомері неможливе!")
else:
    print(f"Обрано r = {r}")

    # Крок 3: Заміна x = X + r → Y² = X³ + 3r·X² + (3r²+a)·X
    alpha = F(3) * r           # коефіцієнт при X²
    beta = F(3) * r^2 + F(a)   # коефіцієнт при X
    print(f"Y² = X³ + {alpha}·X² + {beta}·X")

    # Крок 4: λ = √(3r²+a)
    lam = beta.sqrt()
    print(f"λ = √{beta} = {lam}")

    # Крок 5: Коефіцієнти Монтгомері
    A_m = alpha / lam           # A = 3r/λ
    B_m = F(1) / lam^3          # B = 1/λ³
    print(f"Форма Монтгомері: {B_m}·y² = x³ + {A_m}·x² + x")

    # Перевірка
    check = B_m * (A_m^2 - 4)
    print(f"B(A²-4) = {check} ≠ 0: {check != 0}")
```

## 13. Перетворення Монтгомері → Едвардс

Форма Едвардса: a·x² + y² = 1 + d·x²y², де a ≠ d, a ≠ 0, d ≠ 0.

```python
# Продовжуємо з A_m, B_m з попереднього кроку
a_ed = (A_m + 2) / B_m
d_ed = (A_m - 2) / B_m
print(f"Форма Едвардса: {a_ed}·x² + y² = 1 + {d_ed}·x²y²")
print(f"a ≠ d: {a_ed != d_ed}, a ≠ 0: {a_ed != 0}, d ≠ 0: {d_ed != 0}")
```

## 14. Переведення точок між формами

```python
# Вейєрштрас → Монтгомері
P = E(3, 4)
px, py = F(P[0]), F(P[1])
x_m = (px - r) / lam
y_m = py
print(f"Монтгомері: ({x_m}, {y_m})")

# Перевірка: B·y² = x³ + A·x² + x
lhs = B_m * y_m^2
rhs = x_m^3 + A_m * x_m^2 + x_m
print(f"Перевірка: {lhs} = {rhs} → {lhs == rhs}")

# Монтгомері → Едвардс (потрібно y_m ≠ 0, x_m ≠ -1)
x_e = x_m / y_m
y_e = (x_m - 1) / (x_m + 1)
print(f"Едвардс: ({x_e}, {y_e})")

# Перевірка: a·x² + y² = 1 + d·x²·y²
lhs = a_ed * x_e^2 + y_e^2
rhs = 1 + d_ed * x_e^2 * y_e^2
print(f"Перевірка: {lhs} = {rhs} → {lhs == rhs}")
```

## 15. Ендоморфізм Фробеніуса

```python
E = EllipticCurve(GF(17), [3, 2])
phi = E.frobenius_endomorphism()
print(phi)

# Характеристичний поліном Фробеніуса: t² - (q+1-n)t + q
t_trace = 17 + 1 - E.order()
print("Слід Фробеніуса:", t_trace)

# Застосування до точки
P = E(3, 4)
print(phi(P))    # = (3^17 mod 17, 4^17 mod 17) = (3, 4) для F_17-раціональних точок
```

## 16. Суперсингулярність

```python
E = EllipticCurve(GF(17), [3, 2])

print(E.is_supersingular())    # True/False
print(E.is_ordinary())         # протилежне

# Суперсингулярна ⟺ слід Фробеніуса ≡ 0 (mod p)
trace = 17 + 1 - E.order()
print("Слід:", trace, ", SS:", trace % 17 == 0)
```

## 17. Корисні команди

```python
# Випадкова точка на кривій
P = E.random_point()

# Перевірка належності точки кривій
print(E(3, 4) in E.points())

# Координати точки
P = E(3, 4)
print(P[0], P[1])    # x, y (афінні)
print(P.xy())         # (x, y) кортеж

# Факторизація порядку
print(factor(E.order()))

# Всі криві над F_q (перебір)
F = GF(11)
for a in F:
    for b in F:
        if 4*a^3 + 27*b^2 != 0:
            E = EllipticCurve(F, [a, b])
            print(f"y²=x³+{a}x+{b}, j={E.j_invariant()}, #E={E.order()}")
```

## 18. Швидка довідка

| Команда | Опис |
|---------|------|
| `GF(q)` | Створити поле F_q |
| `EllipticCurve(F, [a, b])` | Крива y²=x³+ax+b над F |
| `E.order()` | Порядок групи точок |
| `E.points()` | Список всіх точок |
| `E.abelian_group()` | Структура абелевої групи |
| `E.j_invariant()` | j-інваріант |
| `E.discriminant()` | Дискримінант |
| `E.is_isomorphic(E2)` | Перевірка ізоморфізму |
| `E.isomorphism_to(E2)` | Явний ізоморфізм |
| `E.quadratic_twist()` | Квадратичне кручення |
| `E.twists()` | Всі кручення |
| `E.is_supersingular()` | Перевірка суперсингулярності |
| `E.frobenius_endomorphism()` | Ендоморфізм Фробеніуса |
| `E.division_polynomial(n)` | n-й поліном ділення |
| `E.random_point()` | Випадкова точка |
| `P.order()` | Порядок точки |
| `P.xy()` | Афінні координати |
| `factor(n)` | Факторизація числа |
| `EllipticCurve_from_j(j)` | Крива з даним j-інваріантом |
