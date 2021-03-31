---
layout: post
title: (Javascript ES6) String Object
summary: 인프런 강좌_자바스크립트 ES6+ 기본
author: devhtak
date: '2020-12-10 22:41:00 +0900'
category: Javascript ES6+
---

### Unicode

- 유니코드는 U+0031 형태
  - 코드 포인트
    - 0031이 코드 포인트
    - 문자 코드라고도 부른다
    - 코드 포인트로 문자/이모지 등을 표현
    - 4자리 이상의 UTF-16 진수 형태
  - 110 만개 정도 표현
    - U+0000 ~ U+10FFFF
    
- 유니코드 용어
  - plane (평면)
    - 코드 포인트 전체를 17개 plane으로 나눔
    - 하나의 plane은 65535(U+FFFF)개
    - 첫 번째 plane
      - BMP(Basic Multillingual Plane)라고 부른다.
      - 일반적인 문자(영문자, 숫자)가 여기에 속한다.
      - 한글의 코드 포인트도 여기에 속한다.
    - 첫 번째 plane을 제외한 plane
      - Supplementary plane, Astral plane이라고 부른다.
      - 5자리 이상의 코드 포인트를 표현할 수 있다.
        - ES6+ 에서 지원
  - Unicode
    - 이스케이프 시퀀스(Escape Sequence)
      - 역슬래시와 16진수로 값을 작성
        ```javascript
        const escape = "\x31\x32\x33";
        console.log(escape); // 123
        console.log("\\"); // \
        ```
        - 역슬래시가 에디터에 "\" 형태로 표시된다.
        - x를 소문자로 작서해야 한다.
        - JS 코드에서 역슬래시를 표시하려면 역슬래시를 2개 작성해야 한다.
      - 이를 16진수 이스케이프 시퀀스라고 부른다.
    - Unicode Escape Sequence
      - 이스케이프 시퀀스를 유니코드로 작성한 형태
        ```javascript
        const escape = "\x31\x32\x33";
        const unicode = "\u0034\u0035\u0036";
        console.log(escape); // 123
        console.log(unicode); // 456
        ```
        - 역슬래시 다음 u를 작성
    - UES 값 범위
      - UTF-16 진수로 U+0000에서 U+FFFF까지 사용 가능
        
- ES5 문제
  - U+FFFF 보다 큰 코드 포인트는 어떻게 작성할 것인가?
  - Unicode Code Point Escape
    - 코드 포인트 값에 관계없이 사용할 수 있는 형태 필요
      ```javascript
      const unicode = "\u0031\u0032\u0033";
      const ex6 = "\u{34}\u{35}\u{36}";
      console.log(unicode); // 123
      console.log(ex6); // 456
      ```
      - \u{31}, \u{1f418} 형태

- ES5 호환성
  - Surrogate pair
    - \u{1f418} 형태를 ES5에서 사용 불가
    - ES5에서는 두 개의 Unicode Escape Sequence 사용
    - 이를 Surrogate pair라고 한다.
      ```javascript
      const pair = "uD8SD\uDC18"; // = \u{1f418}
      console.log(pair); 
      ```
      - "\uD83D" 와 "\uDC18"을 연결하여 작성한다.
  - Surrogate pair 계산 방법
  - 유니코드 사용의 참조 사항
    - 브라우저, 스마트폰에 따라표시되는 문자 모습이 다르다.
    
### Unicode 함수

- fromCodePoint()
  - 형태: String.fromCodePoint()
  - 파라미터: 코드 포인트, num1 [, ...[, numN]]
  - 반환: 코드 포인트에 해당하는 문자로 반환
  
  - 유니코드의 코드 포인트에 해당하는 문자 반환
  - 파라미터에 다수의 코드 포인트 작성 가능
    - 문자를 연결하여 반환
      ```javascript
      console.log(String.fromCodePoint(49, 50, 51)); // 123
      console.log(String.fromCodePoint(44032, 44033)); // 가각
      console.log(String.fromCodePoint(0x31, 0x32, 0x33)); //123
      ```
  - ES5의 fromCharCode() 사용
    - Surrogate pair로 작성
      ```javascript
      console.log(String.fromCharCode(0xD83D, oxDC18)); // == String.fromCodePoint(ox1f418);
      ```

- codePointAt()
  - 형태: String.prototype.codePointAt()
  - 파라미터: 유니코드로 변환할 문자열의 인덱스, 1나만 작성 가능하다.
  - 반환: 코드 포인트 값
  
  - 대상 문자열에서 파라미터에 작성한 인덱스 번째 문자를 유니코드 코드 포인트로 변환하여 반환
    ```javascript
    const result = "가나다".codePointAt(2);
    console.log(result); // 45796
    console.log(typeof result); // number
    
    console.log("가나다".codePointAt(3)); // undefined
    console.log(String.fromCodePoint(result)); // 다
    ```
    - "가나다".codePointAt(2)
      - 문자열 "가나다"의 3번째에 코드 포인트를 구해 반환한다.
    - 반환된 코드 포인트 타입은 number이다.
    - 인덱스 번째에 문자가 없으면 undefined를 반환한다.
    - "가나다".codePointAt(2) 에 값은 45796이고, String.fromCodePoint("45796")은 "다"dlek.
  - 코드 포인트는 UTF-16으로 인코딩된 값

- normalize()
  - 형태: String.prototype.normalize()
  - 파라미터: 정규화 형식. 디폴트: NFC
  - 반환: 변환된 문자열
  
  - 대상 문자열을 파라미터에 지정한 유니코드 정규화 형식으로 변환하여 반환
    ```javascript
    console.log("ㄱ".codePointAt().toString(16)); // 3131
    console.log("ㅏ".codePointAt().toString(16)); // 314f
    console.log("\u{3131}\u{314f}"); // ㄱ ㅏ
    ```
    - ㄱ 과 ㅏ 의 코드 포인트를 16진수로 구한다.
    - ㄱ 과 ㅏ 의 코드 포인트를 연결하여 작성
  - 유니코드 정규화 형식
    - NFC, NFD, NFKC, NFKD
    - http://www.unicode.org/reports/tr15
      ```javascript
      const point = "\u{3131}\u{314F}";
      console.log(point.normalize("NFC")); // ㄱ ㅏ
      console.log(point.normalize("NFD")); // ㄱ ㅏ
      console.log(point.normalize("NFKD")); // 가
      console.log(point.normalize("NFKC")); // 가
      ```
      - NFC와 NFD는 단지 연결하여 어색하지만 NFKD와 NFKC는 모아 쓴 형태로 표시된다.

### String 추가 함수
- startsWith
  - 형태: String.prototype.startsWith()
  - 파라미터: 비교 문자열. 비교 시작 인덱스(option) 디폴트: 0
  - 반환: 시작하면 true, 아니면 false

  - 대상 문자열이 첫 번째 파라미터의 문자열로 시작하면 true, 아니면 false 반환
    - 정규 표현식 사용 불가
      ```javascript
      const target = "ABC";
      console.log(target.startsWith("AB")); // true
      console.log(target.startsWith("BC")); // false
      console.log(/^AB/.test(target)); // true
      ```
      - "AB" 로 시작하므로 true 반환 / "BC" 가 있지만 시작이 아니므로 false 반환
      - 정규표현식의 ^ 과 같습니다.
  - 두 번째 파라미터
    - 선택이며, 비교 시작 인덱스 작성
      ```javascript
      const target = "ABC";
      console.log(target.startsWith("AB")); // true
      console.log(target.startsWith("BC", 1)); // true
      ```
    
- endsWith
  - 형태: String.prototype.endsWith()
  - 파라미터: 비교 문자열. 사용 문자열 길이(option) 디폴트: 0
  - 반환: 끝나면 true, 아니면 false

  - 대상 문자열이 첫 번째 파라미터의 문자열로 끝나면 true, 아니면 false 반환
    ```javascript
    const target = "ABC";
    console.log(target.endsWith("BC")); // true
    console.log(target.endsWith("AB")); // false
    console.log(/BC$/.test(target)); // true
    ```
    - "BC" 로 끝나므로 true 반환 / "AB" 가 있지만 끝이 아니므로 false 반환
    - 정규표현식의 $ 과 같습니다.
    
  - 두 번째 파라미터 선택이며, 사용할 문자열 길이 지정
    ```javascript
    const target = "ABC";
    console.log(target.endsWith("BC")); // true
    console.log(target.endsWith("AB", 2)); // true
    ```
    - ABC 세글자를 2글자(AB)로 사용하겠다고 설정했기 때문에 true가 된다.

- repeat()
  - 형태: String.prototype.repeat()
  - 파라미터: 복제(반복)할 수(option) 디폴트 : 0
  - 반환: 복제하여 만든 문자열
  
  - 대상 문자열을 파라미터에 작성한 수 만큼 복제, 연결하여 반환
    ```javascript
    const target = "ABC";
    console.log(target.repeat(3)); // ABCABCABC
    console.log(target.repeat(0)); // ""
    console.log(target.repeat()); // ""
    console.log(target.repeact(2.7)); // ABCABC
    ```
    - 소수를 넣는 경우(2.7) 버림하여 계산

- includes()
  - 형태: String.prototype.includes()
  - 파라미터: 존재 여부 비교 문자열, 비교 시작 인덱스(option) 디폴트: 0
  - 반환: 존재하면 true, 아니면 false
  
  - 대상 문자열에 첫 번째 파라미터의 문자열이 있으면 true, 없으면 false
    ```javascript
    const target = "123";
    console.log(target.includes("1")); // true
    console.log(target.includes(12)); // true
    console.log(target.includes('13')); // false
    ```
    - "1" 과 12는 존재하며 "13"은 존재하지 않는다.
  - 첫 번째 파라미터
    - 숫자이면 문자열로 변환하여 체크
  - 두 번째 파라미터(선택)
    - 비교 시작 인덱스 작성
      ```javascript
      const target = "ABC";
      console.log(target.includes("A"), 1); // false
      try {
          result = target.includes("/^A/"); 
      } catch(e) {
          console.log("정규 표현식 사용 불가");
      }
      ```
      - "A"가 있지만 0번 인덱스에 있기 때문에 false
      - 정규표현식 사용이 불가하다.
      
- padStart()
  - 형태: String.prototype.padStart()
  - 파라미터: 늘릴 길이 값, 설정할 값(option)
  - 반환: 길이를 늘리고 값을 채운 결과
  
  - 첫번째 파라미터 값만큼 길이를 늘리고 늘어난 끝에 대상 문자열을 설정한 후 앞의 남은 공간에 두 번째 파라미터를 채운다.
  - 두번째 파라미터에 채울 문자열 작성
    ```javascript
    console.log("ABC".padStart(10, "123")); // 1231231ABC
    console.log("ABC".padStart(6, "123456")); //123ABC
    console.log("ABCDE".padStart(3, "123")); // ABCDE
    console.log("ABC".padStart(6).length); // 6
    ```
    
- padEnd()
  - 형태: String.prototype.padEnd()
  - 파라미터: 늘릴 길이 값, 설정할 값(option)
  - 반환: 길이를 늘리고 값을 채운 결과
  
  - 첫번째 파라미터 값만큼 길이를 늘리고 늘어난 끝에 대상 문자열을 설정한 후 뒤의 남은 공간에 두 번째 파라미터를 채운다.
  - 두번째 파라미터에 채울 문자열 작성
    ```javascript
    console.log("ABC".padEnd(10, "123")); // ABC1231231
    console.log("ABC".padEnd(6, "123456")); //ABC123
    console.log("ABCDE".padEnd(3, "123")); // ABCDE
    console.log("ABC".padEnd(6).length); // 6
    ```

- trimStart()
  - 형태: String.prototype.trimStart(), ES2019
  - 파라미터: 없음
  - 반환: 앞 공백을 삭제한 결과
  
  - 문자열 앞의 공백을 삭제
    ```javascript
    const value = "  123";
    console.log(value.length); // 5
    console.log(value.trimStart().length); // 3
    
    const split = "a, b, c".split(",");
    for(let value of split) {
        console.log(value, value.length); // a, 1/ b, 2/ c, 2
        console.log(value.trimStart().length); // 1/1/1
    }
    ```

- trimEnd()
  - 형태: String.prototype.trimEnd(), ES2019
  - 파라미터: 없음
  - 반환: 뒤 공백을 삭제한 결과
  
  - 문자열 뒤의 공백을 삭제
    ```javascript
    const value = "123  ";
    console.log(value.length); // 5
    console.log(value.trimStart().length); // 3
    
    const split = "a ,b ,c".split(",");
    for(let value of split) {
        console.log(value, value.length); // a , 2/ b , 2/ c , 1
        console.log(value.trimEnd().length); // 1/1/1
    }
    ```
    
### Template Literal

- Template Literal
  - Syntax: `문자열`, `문자열${표현식}문자열`, tag`문자열${표현식}문자열`  
  - template literal
    - 문자열 처리를 위한 리터럴
    - 표현식을 포함할 수 있다.
    
  - backtick 안에 표현식 작성
    - 표현식을 ${표현식} 형태로 작성
      ```javascript
      console.log(`ABC`); // ABC
      const one = 1, two = 2;
      const result = `${one} + ${two} 는 ${one + two}이 된다.`;
      console.log(result); // 1 + 2 는 3이 된다.
      ```

- Tagged Template
  - 템플릿에 함수 이름을 작성한 형태
    ```javascript
    const one = 1, two = 2;
    const show = (text, value) => {
        console.log(`${text[0]}${value}`);
        console.log(text[1]);
    };
    show `1 + 2 = ${one + two}`; // 1 + 2 = 3, ""
    ```
    - template에서 문자열과 표현식을 분리
    - 1 + 2 = 까지가 문자열이고, ${one + two}가 표현식이며 평가하면 3이 된다.
    - show() 함수를 호출
    - 문자열을 배열로 넘겨 준다.
      - 왼쪽에서 오른쪽으로 배열 엘리먼트로 추가. 마지막에 빈 문자열을 엘리먼트로 추가
    - 표현식은 평가 결과를 넘겨 준다.
    - console.log(text[1]); 
      - 호출하는 함수에서 넘겨 준 빈 문자열 text[1]이 없으면 undefined가 출력
      
  - 호출되는 함수를 태그 함수(tag function)이라고 부른다.
  - show 함수를 호출하면서 문자열을 배열로 파라미터로 넘기고 표현식 결과를 하나씩 파라미터로 넘긴다.  
  - 호출하는 곳에서 
    - 표현식을 평가한 값을 다수 넘겨 줄 때 사용
    - 태그 함수에 대응하는 파라미터 이름을 작성한 형태
      ```javascript
      const one = 1, two = 2;
      const show = (text, plus, minus) => {
          console.log(`${text[0]}${plus}`);
          console.log(`${text[1]}${minus}`);
          console.log(`${text[2]}${text[3]}`);
      };
      show `1 + 2 = ${one + two} 고 1 - 2 = ${one - two} 이다.`; // 1 + 2 = 3, 고 1 - 2 = -1, 이다.undefined
      ```
  - 문자열을 분리하면 \["1 + 2 =", " 이고 1- 2=", " 이다."]
    - 3개의 배열 엘리먼트가 된다.
  - 표현식을 분리하면
    - ${one + two} 와 ${one - two}
  - show 태그 함수를 호출
  - Rest 파라미터 사용
    ```javascript
    const one = 1, two = 2;
    const show = (text, ...rest) => {
        console.log(`${text[0]}${rest[0]}`);
        console.log(`${text[1]}${rest[1]}${text[2]}`);
    };
    show `1 + 2 = ${one + two} 고 1 - 2 = ${one - two} 이다.`; // 1 + 2 = 3, 고 1 - 2 = -1 이다.
    ```
    
- raw()
  - 형태: String.raw \`template string`
  - 반환: 반환 형태 (option)
  
  - String.raw에
    - 이어서 template을 작성
      ```javascript
      const one = 1, two = 2;
      const result = String.raw `1+2=${one + two}`;
      console.log(result); // 1+2=3
      ```
      - 표현식을 평가하고 결과를 표현식 위치에 설정
    - 줄 바꿈을 문자로 처리
      ```javascript
      console.log(`one\ntwo`); // one, two
      console.log(String.raw `one\ntwo`); // one\ntwo
      ```
      - \n을 문자 그대로로 인식한다.
    - 유니코드의 코드 포인트 처리
      ```javascript
      console.log(`\u{31}\u{32}`); //12
      console.log(String.raw `\u{31}\u{32}`); // \u{31}\u{32}
      ```
      - 유니코드를 반환하지 않고 문자 그대로로 사용한다.
  
  - raw의 "문자열"을 문자 하나씩 전개하면서
    - 두번째 파라미터부터 조합하고 연결
    - 문자열
      ```javascript
      const one = 1, two = 2;
      console.log(String.raw({raw: "ABCD"}, one, two, 3)); // A1B2C3D
      ```
      - 첫번째 파라미터에 인스턴스는 항상 raw로 사용해야 한다.
      - 항상 마지막은 문자열로 만든다.
      - A를 반환 버퍼에 넣고 raw()의 2번째 파라미터 값을 버퍼에 첨부 -> one 변숫값인 1을 첨부하여 A1이 된다.
      - B를 반환 버퍼 끝에 첨부하고 raw()의 3번째 파라미터 값을 버퍼에 첨부 -> 즉, two 변숫값인 2를 첨부
      - raw()의 3번째 파라미터 값을 버퍼에 첨부 -> two 변숫값인 2를 첨부
      - 반복하여 A1B2C3D를 만들어 조합한 결과를 반환한다.
    - 배열
      ```javascript
      const rawValue = {raw: ["A", "B", "C"]};
      console.log(String.raw(rawValue, 1, 2, 3)); // A1B2C
      ```
      - \`A${1}B${2}C`
      - 마지막을 문자열로 생성해야 하기 때문에 3을 사용하지 않고 C에서 끝이 난다.
  - 첫 번째 파라미터는 {raw: 값} 형태
  - 두 번째 파라미터부터 조합할 값 작성
    - ({raw: "ABCD"}, 1, 2, 3)
    
  
** 출처1. 인프런 강좌_자바스크립트 ES6+
