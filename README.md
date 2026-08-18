
수정 내역은 아래로 뽑아서 채워 넣으시면 됩니다.

git remote add upstream https://github.com/appintheair/MRZScanner.git
git fetch upstream
git log --oneline upstream/develop..HEAD
// MRZScanner/Package.swift
// Patched fork: Gregorian-calendar fix for MRZ date century inference (see MRZFieldFormatter).
.package(url: "https://github.com/WOWPASS-APP/MRZParser.git", .revision("09791f10122e840fc05d89ed8b5596425c58985f"))

즉 의존 사슬이 앱 → MRZScanner(fork) → MRZParser(fork) 라서, MRZParser가 사라지면 MRZScanner 해석 단계에서 먼저 깨집니다.

## 이 저장소에 대해

[appintheair/MRZParser](https://github.com/appintheair/MRZParser)를 fork 한 저장소입니다.
MRZ 날짜의 세기 추론을 그레고리력 기준으로 처리하도록 수정한 버전이며,
WOWPASS iOS 앱의 여권 인식 기능에 사용합니다. (`MRZFieldFormatter` 참조)

## 이 조직에 유지되어야 하는 이유

이 저장소는 앱이 직접 참조하지 않고, [MRZScanner](https://github.com/WOWPASS-APP/MRZScanner) fork의
`Package.swift`가 참조하는 **전이 의존성**입니다.

    WOWPASS iOS 앱 → MRZScanner (fork) → MRZParser (fork)

**특정 커밋(`09791f10122e840fc05d89ed8b5596425c58985f`)에 고정**되어 있어,
저장소가 삭제되거나 주소가 바뀌면 MRZScanner 해석 단계에서 실패하고
**로컬·CI 양쪽 모두 빌드가 즉시 깨집니다.**

앱 쪽에서는 아래 파일에 잠금 정보가 기록되어 있습니다.

- `Tuist/Dependencies/Lockfiles/Package.resolved`
- `WOWPASS.xcworkspace/xcshareddata/swiftpm/Package.resolved`

여권 촬영 플로우에는 [NFCPassportReader](https://github.com/WOWPASS-APP/NFCPassportReader) fork도 함께 쓰입니다.
세 저장소 중 하나만 옮기거나 지워도 빌드가 깨집니다.

## 비공개로 전환할 수 없는 이유

fork 저장소는 원본이 public인 경우 private으로 전환할 수 없습니다.
GitHub의 제약이라 설정으로 우회할 수 없고, fork 관계를 해제(detach)해야만 가능합니다.
해제는 GitHub Support 요청이 필요합니다.

## 정리 계획

앱 저장소로 코드를 편입해 이 fork 자체를 제거하는 방향으로 정리할 예정입니다.
그 전까지는 **삭제 · 이름 변경 · 소유자 이전을 하지 말아주세요.**
변경이 필요하면 모바일팀에 먼저 공유 부탁드립니다.
 ## 이 저장소에 대해

---

[![Build and test](https://github.com/appintheair/MRZParser/actions/workflows/Build%20and%20test.yml/badge.svg)](https://github.com/appintheair/MRZParser/actions/workflows/Build%20and%20test.yml)
[![codecov](https://codecov.io/gh/appintheair/MRZParser/branch/develop/graph/badge.svg?token=XS5F9MtSfq)](https://codecov.io/gh/appintheair/MRZParser)
[![spm](https://img.shields.io/badge/SPM-compatible-brightgreen.svg)](https://github.com/appintheair/MRZParser/blob/develop/Package.swift)

# MRZParser
[MRZ](https://en.wikipedia.org/wiki/Machine-readable_passport) code parser for TD1(ID cards), TD2, TD3 (Passports), MRVA (Visas type A), MRVB (Visas type B) types.

## Fields Distribution of Official Travel Documents:
![image](https://raw.githubusercontent.com/appintheair/MRZParser/develop/docs/img/Fields_Distribution.png)
#### Fields description
Field | TD1 description | TD2 description | TD3 description | MRVA description | MRVB description
----- | --------------- | --------------- | --------------- | ---------------- | ----------------
Document type | The first letter shall be 'I', 'A' or 'C' |  <- | Normally 'P' for passport | The First letter must be 'V' | <- |
Country code | 3 letters code (ISO 3166-1) or country name (in English) | <- | <- | <- | <- |
Document number | Document number | <- | <- | <- | <- |
Birth date | Format: YYMMDD | <- | <- | <- | <- |
Sex | Genre. Male: 'M', Female: 'F' or Undefined: 'X', "<" or "" | <- | <- | <- | <- |
Expiry date  | Format: YYMMDD | <- | <- | <- | <- |
Nationality | 3 letters code (ISO 3166-1) or country name (in English) | <- | <- | <- | <- |
Surname | Holder primary identifier(s) | <- | Primary identifier(s) | <- | <- |
Given names | Holder secondary identifier(s) | <- | Secondary identifier(s) | <- | <- |
Optional data | Optional personal data at the discretion of the issuing State. Non-mandatory field. | <- | Personal number. In some countries non-mandatory field. | Optional personal data at the discretion of the issuing State. Non-mandatory field. | <- |
Optional data 2 | Optional personal data at the discretion of the issuing State. Non-mandatory field. | X | X | X | X |

## Installation guide
### Swift Package Manager
```swift
dependencies: [
    .package(url: "https://github.com/appintheair/MRZParser.git", .upToNextMajor(from: "1.1.2"))
]
```
## Usage
The parser is able to validate the MRZ string and parse the MRZ code. Let's start by initializing our parser.
```swift
let parser = MRZParser()
```
For parsing, we use the `parse` method which returns the `MRZResult` structure with all the necessary data.
```swift
parser.parse(mrzString: mrzString)
```
## Example
### TD1 (ID card)
#### Input
```
I<UTOD231458907<<<<<<<<<<<<<<<
7408122F1204159UTO<<<<<<<<<<<6
ERIKSSON<<ANNA<MARIA<<<<<<<<<<
```
#### Output
Field | Value
----- | -----
Document type | I
Country code | UTO
Document number | D23145890
Birth date | 1974.08.12
Sex | FEMALE
Expiry date  | 2012.04.15
Nationality | UTO
Surname | ERIKSSON
Given names | ANNA MARIA
Optional data | ""
Optional data 2 | ""

### TD2
#### Input
```
I<UTOERIKSSON<<ANNA<MARIA<<<<<<<<<<<
D231458907UTO7408122F1204159<<<<<<<6
```
#### Output
Field | Value
----- | -----
Document type | I
Country code | UTO
Document number | D23145890
Birth date | 1974.08.12
Sex | FEMALE
Expiry date  | 2012.04.15
Nationality | UTO
Surname | ERIKSSON
Given names | ANNA MARIA
Optional data | ""

### TD3 (Passport)
#### Input
```
P<UTOERIKSSON<<ANNA<MARIA<<<<<<<<<<<<<<<<<<<
L898902C36UTO7408122F1204159ZE184226B<<<<<10
```
#### Output
Field | Value
----- | -----
Document type | P
Country code | UTO
Document number | L898902C3
Birth date | 1974.08.12
Sex | FEMALE
Expiry date  | 2012.04.15
Nationality | UTO
Surname | ERIKSSON
Given names | ANNA MARIA
Optional data | ZE184226B

### MRVA (Visa type A)
#### Input
```
V<UTOERIKSSON<<ANNA<MARIA<<<<<<<<<<<<<<<<<<<
L8988901C4XXX4009078F96121096ZE184226B<<<<<<
```
#### Output
Field | Value
----- | -----
Document type | V
Country code | UTO
Document number | L8988901C
Birth date | 1940.09.07
Sex | FEMALE
Expiry date  | 1996.12.10
Nationality | XXX
Surname | ERIKSSON
Given names | ANNA MARIA
Optional data | 6ZE184226B

### MRVB (Visa type B)
#### Input
```
V<UTOERIKSSON<<ANNA<MARIA<<<<<<<<<<<
L8988901C4XXX4009078F9612109<<<<<<<<
```
#### Output
Field | Value
----- | -----
Document type | V
Country code | UTO
Document number | L8988901C
Birth date | 1940.09.07
Sex | FEMALE
Expiry date  | 1996.12.10
Nationality | XXX
Surname | ERIKSSON
Given names | ANNA MARIA
Optional data | ""

## License

The library is distributed under the MIT [LICENSE](https://opensource.org/licenses/MIT).
