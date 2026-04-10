# 049. PDF-DB 불일치 검사 결과

**날짜**: 2026-03-29

## 개요

966개 PDF의 OCR 결과(첫 페이지)와 DB 논문 메타데이터(제목, 연도)를 비교하여
잘못 연결된 PDF를 탐지. 연도 차이 3년 이상인 경우를 의심 대상으로 분류.

**의심 대상: 14건**

## 불일치 목록

| ID | DB 연도 | OCR 연도 | 차이 | DB 제목 | OCR 내용 (첫 페이지) |
|---:|--------:|---------|-----:|---------|---------------------|
| 3469 | 2021 | 1905 | 116년 | Redlichia nobilis Walcott, 1905, the oldest trilobite in Sou | Article https://doi.org/10.1007/s12303-021-0036-0pISSN 1226-4806 eISSN 1598-7477 |
| 2715 | 2019 | 1937 | 82년 | A new lightfish,† Vinciguerria orientalis, sp. nov.(Teleoste | Taylor &amp; FrancisTaylor &amp; Francis Group Journal of Vertebrate Paleontolog |
| 2712 | 2018 | 1937 | 81년 | Fossil prowfish, Zaprora koreana, sp. nov.(Pisces, Zaprorida | Taylor &amp; FrancisTaylor &amp; Francis Group Journal of Vertebrate Paleontolog |
| 3400 | 1981 | 1904 | 77년 | Stratigraphy of the Pyeongan System | 지질학회지 제17권 제2호 p.109—111, 1981년 6월 平安系의 層序 Stratigraphy of the Pyeongan System 鄭 |
| 2463 | 1991 | 2011 | 20년 | Conodonts from the Machari Formation (Middle ? - Upper Cambr | 284 J. Paleont. Soc. Korea. Vol. 27, No. 2, 2011 Furongian conodonts from the Ma |
| 2261 | 2001 | 2019, 2019 | 18년 | Dinosaur track-bearing deposits in the Cretaceous Jindong Fo | 지질학회지 제 55권 제 5호, p. 495-511, (2019년 10월)J. Geol. Soc. Korea, v. 55, no. 5, p. 4 |
| 2365 | 1999 | 2013, 2014 | 14년 | 펭귄의 진화 | J. Paleont. Soc. Korea. Vol. 29-30, No.1-2, (2013-2014): p. 37-44 37  펭권의 진화   박 |
| 2820 | 2006 | 1986, 1986, 1993 | 11년 | 백악기 영동층군에서 산출된 구과류 화석의 특징과 고기후적 의미 | 한국지구과학회 2006년 춘계학술발표회2006년 2월 16일 - 17일 중 북 대 학교 백악기 영동충군에서 산출된 구과류 화석의특징과 고기후적  |
| 2423 | 1986 | 1994 | 8년 | Emended stratigraphy of the Miocene Formations in the Pohang | J. Paleont. Soc. Korea, V. 10, NO.1, (1994) : pp. 99~116. EMENDED STRATIGRAPHY O |
| 3343 | 1989 | 1984 | 5년 | 江原道 三陟炭田地域의 化折層에서 產出된 上部 캄브리아紀 코노돈트 化石群의 生物區 特性 | 53  江原道 三陟炭田地域의 花折層에서 產出된 上部 캠프리아紀 코노돈트 化石群의 生物區 特性   李 秉 洙   延世大學校 地質學科   要 約   |
| 3607 | 1987 | 1992 | 5년 | Fossils of Korea (2): Mesozoic | 조선의 화석 2 과학기술출판사 1992 |
| 2933 | 1996 | 1993 | 3년 | CONODONTS FROM THE MANITOU FORMATION, COLORADO, U.S.A. | DBpia J. Paleont. Soc. Korea. V. 9, NO.1, (1993) : pp. 77~92. CONODONTS FROM THE |
| 3533 | 1993 | 1986, 1990, 1981 | 3년 | On some acritarchs from the Tosong Formation in Tosongli Zho | 우리는 유도된 해석식을 추판주의 인상속도계산에 적용하여 최대질량추판주의 첫인상속도들을 결정하였다(표).   맺는 말   1. 추판주를 인상할 때 |
| 3599 | 2017 | 2014 | 3년 | 신의주주층의 지질시대에 대한 새로운 견해 | 34 연구론  정확도가 달라지는 부족점을 극복할 수 있다. 참고 문헌 [1] Chabitha, D. et al., ISPRS J. Photogr |

## 상세

### [3469] 연도 차이 116년
- **DB 연도**: 2021
- **OCR 연도**: 1905
- **DB 영문제목**: Redlichia nobilis Walcott, 1905, the oldest trilobite in South Korea: age and morphologic restoration by strain analysis
- **DB 한글제목**: (없음)
- **파일**: references/2021/3469.pdf
- **OCR 첫 페이지**:
  > Article https://doi.org/10.1007/s12303-021-0036-0pISSN 1226-4806 eISSN 1598-7477 Geosciences Journal GJ Redlichia nobilis Walcott, 1905, the oldest trilobite in South Korea: age and morphologic restoration by strain analysis Woon Sang Yoon1, Dong-Cha

### [2715] 연도 차이 82년
- **DB 연도**: 2019
- **OCR 연도**: 1937
- **DB 영문제목**: A new lightfish,† Vinciguerria orientalis, sp. nov.(Teleostei, Stomiiformes, Phosichthyidae), from the middle Miocene of
- **DB 한글제목**: (없음)
- **파일**: references/2019/2715.pdf
- **OCR 첫 페이지**:
  > Taylor &amp; FrancisTaylor &amp; Francis Group Journal of Vertebrate Paleontology ISSN: 0272-4634 (Print) 1937-2809 (Online) Journal homepage: https://www.tandfonline.com/loi/ujvp20 A new lightfish, †Vinciguerria orientalis, sp. Nov.(Teleostei, Stomi

### [2712] 연도 차이 81년
- **DB 연도**: 2018
- **OCR 연도**: 1937
- **DB 영문제목**: Fossil prowfish, Zaprora koreana, sp. nov.(Pisces, Zaproridae), from the Neogene of South Korea
- **DB 한글제목**: (없음)
- **파일**: references/2018/2712.pdf
- **OCR 첫 페이지**:
  > Taylor &amp; FrancisTaylor &amp; Francis Group Journal of Vertebrate Paleontology ISSN: 0272-4634 (Print) 1937-2809 (Online) Journal homepage: https://www.tandfonline.com/loi/ujvp20 Fossil prowfish, Zaprora koreana, sp. nov. (Pisces, Zaproridae), fro

### [3400] 연도 차이 77년
- **DB 연도**: 1981
- **OCR 연도**: 1904
- **DB 영문제목**: Stratigraphy of the Pyeongan System
- **DB 한글제목**: (없음)
- **파일**: references/1981/3400.pdf
- **OCR 첫 페이지**:
  > 지질학회지 제17권 제2호 p.109—111, 1981년 6월 平安系의 層序 Stratigraphy of the Pyeongan System 鄭昌熙(Chang Hi Cheong)* 1886년 독일의 지질학자 Gottsche가 현 大成단과 부근에서 Neuropteris라는 식물화석을 채취하고 그 곳과 慶尙道 일대에 石炭系가 분포한다고 하여 韓國에 石炭系의 존재를 처음으로 밝혔다. 그러나 慶尙道 일대에 石炭系가 없음은 곧(1904) 명백하게 되었다

### [2463] 연도 차이 20년
- **DB 연도**: 1991
- **OCR 연도**: 2011
- **DB 영문제목**: Conodonts from the Machari Formation (Middle ? - Upper Cambrian) in the Yeongweol area, Kangweon-Do, Korea
- **DB 한글제목**: (없음)
- **파일**: references/1991/2463.pdf
- **OCR 첫 페이지**:
  > 284 J. Paleont. Soc. Korea. Vol. 27, No. 2, 2011 Furongian conodonts from the Machari Formationin the Gokgeum section, Yeongwol, Korea Byung-Su Lee* Department of Earth Science Education, Chonbuk National University, 561-756, Jeonju, Korea This is a 

### [2261] 연도 차이 18년
- **DB 연도**: 2001
- **OCR 연도**: 2019, 2019
- **DB 영문제목**: Dinosaur track-bearing deposits in the Cretaceous Jindong Formation, Korea
- **DB 한글제목**: (없음)
- **파일**: references/2001/2261.pdf
- **OCR 첫 페이지**:
  > 지질학회지 제 55권 제 5호, p. 495-511, (2019년 10월)J. Geol. Soc. Korea, v. 55, no. 5, p. 495-511, (October 2019)DOI http://dx.doi.org/10.14770/jgsk.2019.55.5.495 ISSN 0435-4036 (Print)ISSN 2288-7377 (Online) 경북 청송군 신성리 백악기 사곡층의 공통발자국화석 퇴적층:산상 및 고환경 김현주1 · 황구근2

### [2365] 연도 차이 14년
- **DB 연도**: 1999
- **OCR 연도**: 2013, 2014
- **DB 영문제목**: (없음)
- **DB 한글제목**: 펭귄의 진화
- **파일**: references/1999/2365.pdf
- **OCR 첫 페이지**:
  > J. Paleont. Soc. Korea. Vol. 29-30, No.1-2, (2013-2014): p. 37-44 37 
펭권의 진화
 
박진영* · 허 민
 
전남대학교 지구환경과학부 &amp; 한국공룡연구센터
 
요 약: 펭권의 기원과 이들이 어떻게 남반구의 모든 대륙에서 번성하게 되었는지에 대한 것은 비교적 최근에 발견된 화석증거들을 통해 해석이 가능해졌다. 가장 오래된 펭권류인 와이마누 만네링이(뉴질랜드, 팔레오세 그린스톤층)는 펭권

### [2820] 연도 차이 11년
- **DB 연도**: 2006
- **OCR 연도**: 1986, 1986, 1993, 1995, 1927
- **DB 영문제목**: (없음)
- **DB 한글제목**: 백악기 영동층군에서 산출된 구과류 화석의 특징과 고기후적 의미
- **파일**: references/2006/2820.pdf
- **OCR 첫 페이지**:
  > 한국지구과학회 2006년 춘계학술발표회2006년 2월 16일 - 17일 중 북 대 학교 백악기 영동충군에서 산출된 구과류 화석의특징과 고기후적 의미 서지혜*, 김종현 공주대학교 지구과학교육학과 요약 충청북도 영동 지역에 분포하는 영동충군은 옥천변성대에 있는 소규모 퇴적분지중의 하나이다. 영동충군의 지질과 고생물에 관한 연구는 옥천변성대의 조구조운동과 관련한 퇴적분지 발달 규명과 아울러 경상분지와와의 상호대비가 가능케 함으로써 이 시기의 한반도의

### [2423] 연도 차이 8년
- **DB 연도**: 1986
- **OCR 연도**: 1994
- **DB 영문제목**: Emended stratigraphy of the Miocene Formations in the Pohang Basin, Part 1
- **DB 한글제목**: (없음)
- **파일**: references/1986/2423.pdf
- **OCR 첫 페이지**:
  > J. Paleont. Soc. Korea, V. 10, NO.1, (1994) : pp. 99~116. EMENDED STRATIGRAPHY OF THE MIOCENE FORMATIONSIN THE POHANG BASIN, PART II : SOUTH OF THE HYONGSAN FAULT Hyesu YUN* * Department of Geology, Chungnam National University, Daejon 305-764, Korea

### [3343] 연도 차이 5년
- **DB 연도**: 1989
- **OCR 연도**: 1984
- **DB 영문제목**: 江原道 三陟炭田地域의 化折層에서 產出된 上部 캄브리아紀 코노돈트 化石群의 生物區 特性
- **DB 한글제목**: (없음)
- **파일**: references/1989/3343.pdf
- **OCR 첫 페이지**:
  > 53 
江原道 三陟炭田地域의 花折層에서 產出된 上部 캠프리아紀 코노돈트 化石群의 生物區 特性
 
李 秉 洙
 
延世大學校 地質學科
 
要 約
 
白雲山向斜帶 南翼인 江原道 三陟炭田 地域의 花折層에서 산출된 上部 캠프리아紀 코노돈트 化石群에 對한 生物區 特性을 연구하였다. 花折層의 코노돈트 化石群은 Phakelodus, Furnishina, Westergaardodina 및 Cordylodus proavus 등 많은 凡世界的 分布屬과 함께 Pro

### [3607] 연도 차이 5년
- **DB 연도**: 1987
- **OCR 연도**: 1992
- **DB 영문제목**: Fossils of Korea (2): Mesozoic
- **DB 한글제목**: 조선의 화석 2: 중생대편
- **파일**: references/1987/3607.pdf
- **OCR 첫 페이지**:
  > 조선의 화석 2 과학기술출판사 1992

### [2933] 연도 차이 3년
- **DB 연도**: 1996
- **OCR 연도**: 1993
- **DB 영문제목**: CONODONTS FROM THE MANITOU FORMATION, COLORADO, U.S.A.
- **DB 한글제목**: (없음)
- **파일**: references/1996/2933.pdf
- **OCR 첫 페이지**:
  > DBpia J. Paleont. Soc. Korea. V. 9, NO.1, (1993) : pp. 77~92. CONODONTS FROM THE MANITOU FORMATION, COLORADO, U. S. A. Kwang-Soo SEO* and Raymond L. ETHINGTON** * Department of Geological Science, Kongju National University,Kongju 314-701, Korea ** D

### [3533] 연도 차이 3년
- **DB 연도**: 1993
- **OCR 연도**: 1986, 1990, 1981, 1936, 1935
- **DB 영문제목**: On some acritarchs from the Tosong Formation in Tosongli Zhonggang District, Korea
- **DB 한글제목**: 중강군 토성리 토성층에서 나온 일부 의원류에 대하여 (상원계 acritarch)
- **파일**: references/1993/3533.pdf
- **OCR 첫 페이지**:
  > 우리는 유도된 해석식을 추판주의 인상속도계산에 적용하여 최대질량추판주의 첫인상속도들을 결정하였다(표).
 
맺는 말
 
1. 추판주를 인상할 때 시추갈구리에서의 출력원전이용조건에 의하여 인상기계시간을 최소로 할 수 있는 최대질량추판주의 첫 인상속도를 해석식(8), (9)에 의하여 결정할 수 있다.
 
2. 제기된 방법에 의하여 추판주의 합리적인 인상속도들과 시추기계의 권양동력을 정확히 결정할 수 있다.
 
참고문헌
 

(1) 박승철, 시추기계

### [3599] 연도 차이 3년
- **DB 연도**: 2017
- **OCR 연도**: 2014
- **DB 영문제목**: (없음)
- **DB 한글제목**: 신의주주층의 지질시대에 대한 새로운 견해
- **파일**: references/2017/3599.pdf
- **OCR 첫 페이지**:
  > 34 연구론 
정확도가 달라지는 부족점을 극복할 수 있다.
참고 문헌
[1] Chabitha, D. et al., ISPRS J. Photogram. Remot. Sens., 89, 13(2014).
 
Precise Geometric and Terrain Rectification of High Resolution Optical Satellite Images Using Ground Control Line
 
Jong Hak Chol
 
In t

