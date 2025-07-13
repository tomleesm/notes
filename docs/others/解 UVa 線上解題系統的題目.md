# 解 UVa 線上解題系統的題目

原文在 2023-08-02 完成

依照之前的經驗，寫程式分成以下的步驟：

1. 定義需求
2. 需求轉換成執行步驟
3. 步驟翻譯成程式碼
4. 電腦執行程式碼

我覺得第二點最困難，業界還給這種轉換能力取名 Computational Thinking，不過我怎麼看都覺得類似數學思考的方法。我目前最缺少的就是這種思考解題能力，所以解 UVa 題目[^1]，把解題過程和程式碼都寫成文章，以為憑證。

第三點所需的程式語言和框架語法並不難學，前幾天複習 PHP 和 Laravel 官方文件，其實大多還記得。UVa 題目最後的程式碼用 Python 語法，因為 PHP 適合寫網站，不是終端機程式[^2]，而且最近學了一點 Python，正好寫題目來練習。

## Q100 3n + 1

已知

n = 22，產生數列 22, 11, 34, 17, 52, 26, 13, 40, 20, 10, 5, 16, 8, 4, 2, 1。cycle length 為 16

i, j => 例如 1, 5

所求：輸入 i 和 j，例如 1 和 5，求 1 到 5 之間的最大的 cycle length

| n | 數列 | cycle length |
| -- | ---- | ------------ |
| 1 | 1 | 1 |
| 2 | 2, 1 | 2 |
| 3 | 3, 10, 5, 16, 8, 4, 2, 1 | 8 |
| 4 | 4, 2, 1 | 3 |
| 5 | 5, 16, 8, 4, 2, 1 | 6 |

最大的 cycle length 為 8

輸入 1 5，輸出 1 5 8

MURMUR：輸出 1 10 或 10 1，輸出 1 10 20 或 10 1 20

所以有可能 i > j 或 i < j

AHA: 輸入值為 x, y，然後 i = 最小值(x, y)，j = 最大值(x, y)，這樣保證 i < 或等於 j

``` python
s = input().split()
x, y = int(s[0]), int(s[1])
i, j = min(x, y), max(x, y)

length = 1
max_length = 1
for n in range(i, j):
    # 求 n 的 cycle length
    # 找出最大的 cycle length
```

``` python
def cycle_length(n):
    cycle_length = 1
    if n == 1:
        return cycle_length
    elif n 是奇數:
        n = 3 * n + 1
    else:
        n = n / 2

    cycle_length += 1
```

while 終止的條件是 `n == 1`，所以

``` python
def cycle_length(n):
    cycle_length = 1
    while n != 1:
        if (n % 2) == 1:
            n = 3 * n + 1
        else:
            n = n / 2

        cycle_length += 1

    return cycle_length
```

最後的程式碼

``` python
def cycle_length(n):
    cycle_length = 1
    while n != 1:
        if (n % 2) == 1:
            n = 3 * n + 1
        else:
            n = n / 2

        cycle_length += 1

    return cycle_length


while True:
    s = input().split()
    x, y = int(s[0]), int(s[1])
    i, j = min(x, y), max(x, y)

    max_cycle_length = 0
    for n in range(i, j + 1):
        length = cycle_length(n)
        max_cycle_length = max(length, max_cycle_length)

    print(i, j, max_cycle_length)
```

## Q272 TeX Quotes

```
"To be or not to be," quoth the Bard, "that
is the question".
The programming contestant replied: "I must disagree.
To `C' or not to `C', that is The Question!"
```

輸入若干列文字(所以用 `input()` 會有問題)，...以 end-of-file 做結束(所以會讀取檔案)...  

```
把每一個字元抓出來
if 字元是雙引號
    if 第一個雙引號
        把雙引號換成兩個 ` 字元
    else
        把雙引號換成 2 個單引號
else
    字元存到字串的尾端 (result )

print(result)
```

STUCK：怎麼知道是第一個雙引號，還是第二個？  
AHA：用一個變數紀錄 `first_double_quote = True`

``` python
if first_double_quote
    # 把雙引號換成 ` 字元
    first_double_quote = False
else:
    # 把雙引號換成兩個單引號
    first_double_quote = True
```

最後結果

``` python
# 讀取檔案為字串
f = open("Q272_input.txt", "rt")
first_double_quote = True
result = ""

for line in f:
    # 把每一個字元抓出來
    for s in line:
        # if 字元是雙引號
        if s == '"':
            # 第一個雙引號
            if first_double_quote:
                # 把雙引號換成兩個 `` 字元
                result += "``"
                first_double_quote = False
            # 第二個雙引號
            else:
                # 把雙引號換成兩個單引號
                result += "''"
                first_double_quote = True
        # 不是雙引號
        else:
            # 字元存到字串尾端
            result += s

f.close()
print(result)
```

Q272_input.txt

```
"To be or not to be," quoth the Bard, "that
is the question".
The programming contestant replied: "I must disagree.
To `C' or not to `C', that is The Question!"
```

## Q458 The Decoder

a -- +2 --> c -- -2 --> a  
p ---> r ---> p  
p ---> r ---> p  
l ---> n ---> l  

K 是多少？

1 --- -K --> `*`  
J ---> C  
K ---> D  
J ---> C  

AB*C*DEFGHI*J*K

所以 K = 7

```
while True
    輸入一列字串
    抓取每一個字元
    字元減掉 K(=7)
    把解密後的字元存到字串
    print 字串
```

``` python
while True:
    password = input()
    result = ""
    for c in password:
```

STUCK：字元是字串，不能直接減掉 7  
AHA 字元 ---> ASCII Code ---> 減掉 7 ---> 字元  
A ---- `ord('A')` --> 65 ---> 58 ---- `chr(58)` --> `:`

``` python
ascii_code = ord(c) - 7
decoded_c = chr(ascii_code)
result += decoded_c

print(result)
```

最後結果

``` python
K = 7
while True:
    # 輸入一列字串
    password = input()
    # 抓取每個字元
    decoded_password = ''
    for c in password:
        # 字元減掉 K(= 7)
        #   字元 --> ASCII Code --> 減掉 K --> 字元
        ascii_code = ord(c)
        decoded_ascii_code = ascii_code - K
        decoded_char = chr(decoded_ascii_code)
        # 把解密後的字元存到字串
        decoded_password += decoded_char
    # print 字串
    print(decoded_password)
```

測試資料

```
# 輸入
1JKJ'pz'{ol'{yhklthyr'vm'{ol'Jvu{yvs'Kh{h'Jvywvyh{pvu5
# 輸出
*CDC is the trademark of the Control Data Corporation.
```

## Q476 Points in Figures: Rectangles

```
r 8.5 17.0 25.5 -8.5
類別 左上x 左上y 右下x 右下y (x1 y2 x2 y1)
所以 x 在 8.5 ~ 25.5, y 在 -8.5 ~ 17.0

*
2.0 2.0
x   y
```

點的 x 要在矩形的 x 範圍內且 y 在矩形的 y 的範圍內，則點在此矩形內。剛好落在邊上的點不視為落在該矩形內，所以 `矩形的 x1 < 點的 x < 矩形的 x2`，`矩形的 y1 < 點的 y < 矩形的 y2`，不含`x` 等於 `x1` 或 `x2` 的邊界情況

x = 2.0 不在 8.5 ~ 25.5 內，所以第一個點不在第一個矩形內

1. 輸入矩形座標
2. 輸入點座標
3. 檢查點是否在矩形內
   4. 抓出一個點
   5. 抓出一個矩形
   6. if 點的x在矩形的x1 ~ x2 且 y 在矩形的y1 ~ y2，則輸出 Point i is contained in figure j
   7. 如果所有矩形檢查過都沒有，則輸出 Point i is not contained in any figure

輸入矩形座標

讀取 terminal 輸入 --> `input()` 一次讀取一行

```
if 只有一個一個字元且為 `*`
    矩形輸入結束
```

所以
``` python
while True:
    s = input()
    if len(s) == 1 and s == "*":
        break
    else:
```
抓取矩形座標，放到 dict ，再放到 list

``` python
rectangles = []
r = s.split() # --> ['r', '8.5', '17.0', '25.5', '-8.5']
rectangle = {
        "x1": float(r[1]),
        "y2": float(r[2]),
        "x2": float(r[3]),
        "y1": float(r[4]),
    }
    rectangles.append(rectangle)
```

輸入點座標，和輸入矩形座標差不多

``` python
while True:
    s = input()
    p = s.split() # --> ['2.0', '2.0']
    if len(p) == 2 and p[0] == "9999.9" and p[1] == "9999.9":
        break
    else:
        points = []
        point = { "x": float(p[0]), "y": float(p[1]) }
        points.append(point)
```

檢查點是否在矩形內

``` python
i = 0, j = 0
for point in points:
    i = i + 1
    is_point_in_rectangles = False
    for rectangle in recangles:
        j = j + 1
        if rectangle['x1'] < point['x'] < rectangle['x2']:
            print('Point ' + i + ' is contained in figure ' + j)
            is_point_in_rectangle = True

    if not is_point_in_rectangle:
        print('Point ' + i + ' is not contained in any figure')
```

最後結果：有修改小地方

``` python
# 新增矩形
rectangles = []
while True:
    s = input()
    if len(s) == 1 and s == "*":
        break

    r = s.split()
    rectangle = {
        "x1": float(r[1]),
        "y2": float(r[2]),
        "x2": float(r[3]),
        "y1": float(r[4]),
    }
    rectangles.append(rectangle)

# 新增點
points = []
while True:
    s = input()
    p = s.split()
    if len(p) == 2 and p[0] == "9999.9" and p[1] == "9999.9":
        break

    point = { "x": float(p[0]), "y": float(p[1]) }
    points.append(point)

# 判斷點是否在矩形內
for i, p in enumerate(points):
    contained_in_figure = False
    for j, r in enumerate(rectangles):
        if r["x1"] < p["x"] < r["x2"] and r["y1"] < p["y"] < r["y2"]:
            print("Point {} is contained in figure {}".format(i + 1, j + 1))
            contained_in_figure = True

    if not contained_in_figure:
        print("Point {0} is not contained in any figure".format(i + 1))
```

## Q477 Points in Figures: Rectangles and Circles

輸入有兩種形狀：矩形和圓形

r 8.5 17.0 25.5 -8.5 --> 矩形  
c 20.2 7.3 5.8 --> 圓形  
  20.2 7.3 是圓心座標， 5.8 是圓半徑  

STUCK：怎麼知道點是否在圓內？  
AHA：(畫圖：一個圓和幾個點) 點到圓心的距離小於圓半徑  

STUCK：點到圓心的距離怎麼計算？  
AHA：如果點 p = (3, 4), 圓心 c = (10, 20)，距離 = 開根號( (3 - 10)的平方 + (4 - 20)的平方 )  
附註：上面這一行在紙上實際上是數學公式，只是不方便輸入到電腦上，所以這樣寫

輸入形狀

```
if 形狀是矩形
    抓 4 個座標值
else if 形狀是圓形
    抓圓心座標和半徑
```

輸入點：同 Q476

檢查是否在圓形內

```
抓一個點
    抓一個圖形
        if 圖形是矩形
            矩形 x1 < 點 x < 矩形 x2, 矩形 y1 < 點 y < 矩形 y2
        else if 圖形是圓形
            開根號( (點x - 圓心x)的平方 + (點y - 圓心y)的平方 ) < 圓半徑
```

開根號 `math.sqrt()`，平方 `math.pow(點x - 圓心x, 2)`

``` python
import math

# 新增矩形
shapes = []
while True:
    s = input()
    if s == "*":
        break

    r = s.split()
    # 輸入矩形
    if r[0] == "r":
        shape = {
            "type": "rectangle",
            "x1": float(r[1]),
            "y2": float(r[2]),
            "x2": float(r[3]),
            "y1": float(r[4]),
        }
    # 輸入圓形
    elif r[0] == "c":
        shape = {
            "type": "circle",
            "x": float(r[1]),
            "y": float(r[2]),
            "r": float(r[3])
        }
    shapes.append(shape)

# 新增點
points = []
while True:
    s = input()
    p = s.split()
    if p[0] == "9999.9" and p[1] == "9999.9":
        break

    point = {"x": float(p[0]), "y": float(p[1])}
    points.append(point)

# 判斷點是否在矩形內
contained_in_figure = False
for i, point in enumerate(points):
    for j, shape in enumerate(shapes):
        if (
            shape["type"] == "rectangle"
            and shape["x1"] < point["x"] < shape["x2"]
            and shape["y1"] < point["y"] < shape["y2"]
        ):
            print("Point {} is contained in figure {}".format(i + 1, j + 1))
            contained_in_figure = True
        elif (
            shape["type"] == "circle"
            and math.sqrt(math.pow(shape["x"] - point["x"], 2) + math.pow(shape["y"] - point["y"], 2))
            < shape["r"]
        ):
            print("Point {} is contained in figure {}".format(i + 1, j + 1))
            contained_in_figure = True

    if not contained_in_figure:
        print("Point {0} is not contained in any figure".format(i + 1))

    contained_in_figure = False
```

## Q488 Triangle Wave

2 --> 有 2 組測試資料

3 --> A 振幅 (A <= 9)  
2 --> F 頻率

5  
3

STUCK: 什麼是振幅？  
AHA: 波上下起伏的大小

STUCK: 什麼是頻率？  
AHA: 例如每秒幾次(幾個波)。相反的，週期是指做一次(一個波)要多久

F = 2 --> 輸出 2 個波，每個波振幅的水平高度為 A --> A = 3 --> 波最大為 333

STUCK: 要怎麼輸入資料？  
``` python
n = int(input())
groups = []
for i in range(0, n):
    A = int(input())
    F = int(input())
    group = { 'A': A, 'F': F}
    groups.append(group) # for 抓出一個 group['A'] 為 A 的值(振幅)
```

STUCK: 如果 A = 3, 要怎麼產生波如下？

```
1
22
333
22
1
```

1) 先產生 333

``` python
for i in range(0, 3):
    print(3, end="") # 不換行
```

2) 產生

```
1
22
333
```

``` python
for A in range(1, 4):
    for i in range(0, A):
        print(A, end="")
    print()
```

3) 產生

```
22
1
```

愈來愈少(2, 1)

``` python
for A in range(2, 0, -1):
    for i in range(0, A):
        print(A, end="")
    print()
```

產生一個波：2) + 3)

``` python
A = 3
# 2)
for i in range(1, A + 1):
    for j in range(0, i):
        print(i, end="")
    print()
# 3)
for i in range(A - 1, 0, -1):
    for j in range(0, i):
        print(i, end="")
    print()
```

產生多個波就包上一個 for 迴圈，外面再包上一個 for 表示有多筆測試資料

``` python
for group in groups:
    # 共產生 F 個波
    for f in range(0, F):
        # 帶入上面的產生一個波
    print()
```

最後組裝起來，調整一下

``` python
# 輸入有 n 組測試資料
n = int(input())
print()

# 開始輸入測試資料
groups = []
for i in range(0, n):
    A = int(input())
    F = int(input())
    group = {"A": A, "F": F}
    groups.append(group)
    print()

for group in groups:
    A = group['A']
    F = group['F']
    # 產生幾個波
    for f in range(0, F):

        # 產生一個波
        # 如果 A = 3
        # 1
        # 22
        # 333
        for i in range(1, A + 1):
            for j in range(0, i):
                print( i , end="" )
            print()
        # 22
        # 1
        for i in range(A - 1, 0, -1):
            for j in range(0, i):
                print(i, end="")
            print()
        # 產生一個波後，換行
        print()
```

## Q494 Kindergarten Counting Game

## 亂七八糟的開始

> 一開始急著解答，所以寫的亂七八糟，沒什麼用。寫出來是為了呈現真實的情況：寫程式的過程並不是都那麼順遂的

Meep Meep! 2  
I tot I taw a putty tat. 7  
I did! I did! I did taw a putty tat. 10  
Shssssssssssh ... I am hunting wabbits. Heh Heh Heh Heh ... 9  

Meep

M: 開始連續的英文字母  
p: 停止連續的英文字母

1. 把一行字串的每一個字串抓出來
2. M 是在 A ~ Z 或 a ~ z 嗎？(正規表示法？)  
  if yes, 連續英文字母 = True  
  e if yes, 連續英文字母 = True  
  e if yes, 連續英文字母 = True  
  p if yes, 連續英文字母 = True  
  (空白) else 連續英文字母 = False  
從 yes --> No, word 數 +1    

~~一開始，連續英文字母 = False~~  
~~把一行字串的每一個字元抓出來~~  
~~if 字元在 A ~ Z 或 a ~ z 之間~~  
  ~~連續英文字母 = True~~  

``` python
line_str = input()
word_start == False
for s in line_str
    if s 在 A ~ z, a ~ z 之間
        word_start = True # 開始一個新的字(word)
```

AHA: 所以有四種情況

|  | 原來的 word_start = True | 原來的 word_start = False |
| -- | --- | --- |
| s A ~ Z, a ~ z True | | |
| s A ~ Z, a ~ z False | | |

## 有耐心的特殊化

> 從這裡開始，有耐心的特殊化，就好多了

一開始, word 數 = 0

| 字元 | word_start | 字元是英文字母 | word 數 | |
| -- | --------- | ------------------ | ---- | -- |
| M | ✓ | <-- ✓ | 0 | |
| e | ✓ | <-- ✓ | 0 | |
| e | ✓ | <-- ✓ | 0 | (1) |
| p | ✓ | <-- ✓ | 0 | |
| (空白) | x | <-- x | 1 | (2) |
| M | ✓ | <-- ✓ | 1 | (3) |

``` python
if word_start == True and not (s 是英文字母)
    word_count += 1
    word_start = False

if word_start == False and s 是英文字母
    word_start = True
```

----------

``` python
word_start = False
word 數 = 0
for s in 一行字串
    if word_start:
        if s 是英文字母
            # (1)
        else:
            # (2)
            word 數 += 1
            word_start = False
    else:
        if s 是英文字母:
            # (3)
            word_start = True
        else:
            # (4) <-- 字元是 ! 或 .

print(word_count)
```

把上面的 code 包在 while True 內，for 迴圈跑完, `word_start = False, word_count = 0`

STUCK: 如果結尾沒有標點符號，會少算一個 word

例如 This is a test  
顯示為 3，正確值 4  

改成 not s 是英文字母，或已到字串結尾

``` python
for i, s in 一行字串
    (i + 1) == len(一行字串)
```

最後的結果

``` python
import re

while True:
    word_start = False
    word_count = 0

    line_of_string = input()
    for i, c in enumerate(line_of_string):
        match = re.match(r'[a-zA-Z]', c)
        # match is None: 字元不是英文字母
        # i + 1 == len(line_of_string): 單純是該行結束了
        if word_start and (match is None or (i + 1) == len(line_of_string)):
            word_count += 1
            word_start = False
        if not word_start and not( match is None ):
            word_start = True

    print(word_count)

```

[^1]: [UVa線上解題系統](https://zh.wikipedia.org/zh-tw/UVa%E7%B7%9A%E4%B8%8A%E8%A7%A3%E9%A1%8C%E7%B3%BB%E7%B5%B1)
[^2]: 我知道有 [STDIN](https://www.php.net/manual/en/features.commandline.io-streams.php) 這個東西，只是...