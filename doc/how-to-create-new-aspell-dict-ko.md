Aspell: 새로운 언어 사전 만들기
================================


사전 템플릿 만들기
--------------------------------

### aspell-lang 다운로드

aspell 사전을 만들기 위해서 필수적인 파일들이 있는데 직접 하나씩 만들어도
되지만 `aspell-lang`을 이용하여 기본적인 템플릿을 생성한 후에 내용을 채워넣으면
좀더 편리하다.

우선 aspell-lang을 다운받아야 한다. 최신 배포 파일을 찾아서 다운받아 압축을
풀면 된다. 예를 들어, 현재 최신 파일은 다음과 같다.

    $ curl -O ftp://ftp.gnu.org/gnu/aspell/aspell-lang-20101122.tar.gz
    $ tar xzvf aspell-lang-20101122.tar.gz

또는 다음처럼 CVS 저장소를 내려받아도 된다. 

    $ cvs -z3 -d:pserver:anonymous@cvs.savannah.gnu.org:/sources/aspell co aspell-lang


### pre: 기본 디렉토리 생성

이제 `aspell-lang` 디렉토리로 들어가서 `pre LANG CHARSET` 명령을 내린다. 예를
들어, 만주어의 경우에 LANG은 `mnc`, CHARSET은 `iso-8859-1`을 쓴다고 하면 다음과
같이 명령을 내리면 된다.

    $ cd aspell-lang
    $ ./pre mnc iso-8859-1

이제 `mnc`라는 디렉토리가 새로 만들어진 것을 확인할 수 있다. 이 안에는 다음과
같은 파일들이 들어 있다. 

    $ cd mnc
    $ ls
    Copyright info      mnc.dat   mnc.wl    proc

이 파일에 적절한 내용을 채워넣어야한다. 어떤 내용을 채워넣을지는 다음 섹션에서
설명하겠다.

### LANG: 언어 코드

위에서 만주어를 위해 `mnc`라는 이름을 사용했다. 이 이름은 ISO 639의 언어명
코드를 따른다. 영어는 `en`, 한국어는 `ko` 등으로 두 글자짜리가 있으면 그것을
사용한다. 만주어의 경우에는 두 글자짜리 코드가 없기 때문에 세 글자짜리 코드
`mnc`를 사용한 것이다.

경우에 따라서는 같은 언어의 사전이기는 하지만 차이를 구별하려고 할 수도 있다.
이때 사전 이름의 형식은 "언어명_지역-변이형-크기"를 따른다. 예를 들어, 영어는
`en`인데 미국 영어는 `en_US`, 영국 영어는 `en_GB`, 캐나다 영어는 `en_CA`로
이름을 붙인다. 독일어는 `de`인데 옛날 철자법을 따르는 사전의 이름은
`de-alt`로 붙이는 식이다.

### CHARSET: 문자 세트

위에서 만주어를 위해 `iso-8859-1` 문자세트를 사용했다. 이것은 유니코드의 첫
256글자와 같다. 영문자 알파벳만 사용하는 언어를 위해 사전을 만들 것이라면
`iso-8859-1`을 사용하면 된다.

aspell에서 말하는 문자 세트와 인코딩은 다른 것이다. aspell은 유니코드도
문제없이 다룰 수 있다. 다만 내부적으로는 8비트로 처리된다. 기본적으로 지원되는
문자세트는 `aspell-lang/maps` 디렉토리에 개별 파일로 들어있다.

지원하려는 언어가 사용하는 문자세트를 새로 만들어야 할 수도 있다. 유니코드
문자를 사용하는 경우에는 `u-<LANG>.cmap`, `u-<LANG>.cset` 파일, 영문자를
기본으로 하면서 추가적으로 몇개 문자가 더 필요한 경우에는 `l-<LANG>.cmap`,
`u-<LANG>.cset` 파일로 문자세트를 정의해야한다.

예들 들어 살펴보자. 만주어를 표현하기 위해서 영문자 `a-z` 외에 두 개의 유니코드
문자 š (U+0161), ū (U+016B)가 필요하다고 하자. `asepll-lang/maps` 아래 다음과
같이 `l-mnc.txt`라는 이름의 파일을 만들자.

    include iso-8859-1.txt
    
    0x80 U+0161
    0x81 U+016B

여기서 `include iso-8859-1.txt`는 만주어 문자세트에 모든 아스키 문자를
포함하겠다는 의미이다. 그리고 아래쪽에 추가로 사용할 문자 목록을 써준다.
`0x80`, `0x81`은 aspell에서 내부적으로 사용할 코드이다.

이제 `aspell-lang` 디렉토리에서 다음 명령으로 `l-mnc.cmap`과 `l-mnc.cset`
파일을 생성할 수 있다. 이 파일들을 `mnc` 디렉토리로 복사해 넣으면 된다.
`l-mnc.txt`는 `mnc/misc` 아래 넣어주는 것이 일반적이다.

    $ ./mkchardata maps/l-mnc.txt    
    $ cp maps/l-mnc.* mnc/
    $ mkdir mnc/misc
    $ mv mnc/l-mnc.txt mnc/misc/

기본 정보 작성
--------------------------------

이제 `mnc` 디렉토리에는 다음과 같은 파일들이 존재하는 상태가 되었다.

    $ ls
    Copyright  info       mnc.dat    mnc.wl     proc 

여기서 `info`와 `LANG.dat`에 해당 사전에 대한 정보를 넣어주어야 한다.
`LANG.wl`은 단어 목록(word list)인데 이에 대해서는 다음 섹션에서 설명하겠다.

### `info`

가장 먼저 `info`파일의 내용을 작성해 보자. 이 파일에는 해당 사전에 대한 정보가
기록된다. 예를 들어, 만주어의 경우 다음과 같이 만들 수 있다.

    name_english Manchu
    lang mnc
    data-file mnc_affix.dat
    author:
      name You Hyun Jo
      email you at cpan.org
    copyright GPLv3
    url https://github.com/youhyunjo/manchu-spell
    version 20130123-0
    source-version 20130123
    complete false
    accurate true
    alias mnc manchu
    dict:
      name mnc
      add mnc

이때 `data-file`에 명시한 파일들로 필수적인 파일이 된다. 단순히 단어목록만 모든
것이 해결되는 언어라면 추가적인 데이터 파일은 필요없다. 만주어의 경우에는
접사가 다양하게 변하는 언어이므로 접사 규칙을 사용할 필요가 있으므로
`mnc_affix.dat`를 추가하였다.

### `LANG.dat`

이제 `LANG.dat` 파일의 내용을 작성해보자. 포함될 내용은 내용은 간단하다.
만주어의 경우에는 `mnc.dat`가 되고 다음과 같은 내용이 들어간다.

    name mnc
    charset iso-8859-1
    data-encoding utf-8
    affix mnc
    affix-compress true
    partially-expand true
    repl-table mnc_affix.dat
    special ' -**

아스키 문자만 사용한다면 `charset iso-8859-1`이 되고 별도의 문자 세트를
만들었다면 문자세트의 이름을 써주면 된다. 예를 들어, 앞서 만든 문자세트를
이용한다면 `charset l-mnc`라고 할 수 있다. 마지막 행의 `special`은 특수한
문자를 단어의 일부로 사용하기 위한 장치이다. Aspell 매뉴얼에서 자세한 설명이
나온다.

### 라이선스와 저작권

다음으로 저작권을 담은 Copyright 파일을 작성하자. 적당히 작성하면 된다. 널리
사용되는 라이선스를 위한 `COPYING` 파일은 자동으로 제공해 준다.

### `proc` 스크립트

이제 `proc` 스크립트를 이용하여 필요한 파일들을 자동으로 생성할 수 있다. 그런데
`data-file`로 추가한 `mnc_affix.dat`가 없는 상태이다. 일단은 빈 파일을 만들어보자
그리고 `proc` 스크립트를 실행하자

    $ touch mnc_affix.dat
    $ ./proc

여러 가지 필요한 파일들이 만들어진다. `README`, `COPYING` 등이 제공되고
`configure`, `Makefile.pre` , `.alias`, `.multi` 등이 만들어진다.

단어 목록과 접사 규칙 만들기
--------------------------------

모든 준비가 되었다. 이제 진짜 사전을 만들 차례이다. 단어 목록으로 해결되는
언어라면 `LANG.wl`을 만드는 것으로 충분하다. 단어목록 파일 `LANG.wl`은 한 행에
한 단어를 담고 있는 파일이다. 단어의 형태가 변하지 않는다면 그냥 단어 목록을
만드는 것으로 충분하다. 단어가 모양이 변한다면 단어목록과 접사규칙을
만들어야한다. 대개의 경우에는 접사 규칙이 필요하므로 `LANG_affix.dat`를 만들
필요가 있다.

만주어의 예를 들어 살펴보자. 만주어의 명사 beye는 beyei, beyede, beyebe로
변한다. 이때 -i, -de, -be는 접미사가 된다. 동사 arambi는 arame, arafi, araha,
arara, araci로 변한다. ara-가 어간이고 -mbi, -me, -fi, -ha, -ra, -ci가
어미이다. 한편, 동사 genembi는 geneme, genefi, genehe, genere, geneci로
변한다고 하자. gene-가 어간이고 -mbi, -me, -fi, -he, -re, -ci가 어미이다.
-ha/-he, -ra/-re는 어간의 형태에 따라 선택되는 것을 알 수 있다.

이때 `mnc.wl`을 다음과 만들어보자. 

    beye/N
    ara/V
    gene/V

여기에서 `beye`, `ara`, `gene`는 어간이다. `N`과 `V`는 접사의 유형을 구별하는
장치이다. 하나의 그룹을 이루는 접사는 하나의 문자로 나타낼 수 있다.
`mnc_affix.dat`에는 `N`, `V`라는 접사 그룹에 대한 규칙을 기술해야한다. 예를
들면 다음과 같다.

    SFX N Y  3
    SFX N 0 i .
    SFX N 0 de .
    SFX N 0 be .

    SFX V Y 8
    SFX V 0 mbi .
    SFX V 0 me .
    SFX V 0 fi .
    SFX V 0 ha a
    SFX V 0 he e
    SFX V 0 ra a
    SFX V 0 re e
    SFX V 0 ci .

첫 행의 `SFX`는 접미사(suffix)라는 의미이다. `N`는 접사유형을 구분하기 위한
이름이다. `Y`는 접두사(prefix)와 함께 나타날 수 있다는 의미이다. `3`은 접미사
`N` 그룹에 3개의 규칙이 있다는 의미이다. 

두번째 행부터는 실제 접미사 규칙이다. `SFX N`까지는 접미사 그룹 `N`이라는
의미이고 이어지는 3개 컬럼이 각각 어간에서 뺄 것, 더할 것, 조건를 나타낸다.
`0 i .`에서 `0`은 어간에서 아무것도 빼지 않는다는 의미아다. `i`는 i를
덧붙인다는 의미이다. `.`은 조건에 제약이 없다는 의미이다.

이제 접미사 `V` 그룹을 보자. `SFX V 0 ha a`는 어간이 a로 끝날 때 아무 것도 빼지
않고 ha를 덧붙이라는 의미이다. `SFX V 0 he e`는 어간이 e로 끝날 때 아무 것도
빼지 않고 he를 덧붙이라는 의미이다. 따라서 ara/V는 araha로 확장되고 gene/V는
genehe로 확장된다.

설치
--------------------------------

이제 모든 파일이 준비되었다. 설치를 하기 위해서는 우선 설치용 사전 파일을
만들어야 한다. 다음 명령으로 `LANG.cwl`을 만들고 `LANG.rws`를 만들 수 있다.
이것이 설치될 사전 파일이다.

    $ ./configure
    $ make

설치는 다음과 같이 한다.

    $ make install

물론 지금까지 만든 사전을 사용하기 위해서는 이를 위해서는 aspell 0.60 이후
버전이 설치되어 있어야한다. 

사용하기
--------------------------------

다음 명령은 현재 시스템에 설치되어 있는 사전의 목록을 출력해준다. 

    $ aspell dump dicts

설치한 사전의 어휘 목록을 출력해보자. `-l <lang>` 옵션으로 언어를 지정한다.
만주어라면:

    $ aspell -l mnc dump master
    ara/V
    beye/N
    gene/V

설치된 사전의 접사 규칙이 제대로 작동하는지 알아보기 위하여 `expand`와
`munch`를 사용해 볼 수 있다.

    $ echo 'ara/v' | aspell -l mnc expand 
    ara araci arara araha arafi arame arambi
    $ echo 'araha' | aspell -l mnc munch
    araha ara/V

철자법 검사가 어떻게 작동하는지 확인하고 싶다면:

    $ echo 'arahe' | aspell -l mnc -a
    @(#) International Ispell Version 3.1.20 (but really Aspell 0.60.6.1)
    & arahe 2 0: araha, arame

배포하기
--------------------------------

사전이 제대로 만들어졌다고 확신한다면 배포할 수 있다. 배포할 파일은 사전
디렉토리에서 다음 명령으로 간단히 만들 수 있다.

    $ make dist

이 명령의 결과 `aspell6-mnc-2013012309.tar.bz2` 파일이 만들어진다. 이것을
배포하면 된다.

Aspell 설치
--------------------------------

Debian 계열 리눅스에서는 다음 명령으로 설치할 수
있다.

    $ sudo apt-get install

MacOSX에서는 Homebrew를 이용하는 것이 가장 손쉬운 방법인 듯하다.

    $ brew install aspell


MS Windows의 경우
--------------------------------

MS Windows를 위한 것으로는 Aspell 0.5 버전까지만 공식적으로 지원된다. 이후
버전을 설치하는 것은 여러가지 문제가 있다. 2013년 1월 현재 다음 파일을 다운로드
받아 설치하는 것이 가장 간편한 듯하다.

* http://nosq.com/download/aspell-0.60.3.exe

그리고 사전들이 들어있는 디렉토리를 찾아서 필요한 파일들을 모두 복사해서
넣어주면 된다. 위 설치 파일을 이용했다면 사전 디렉토리는 다음과 같다.

* `C:\Program Files\Aspell\lib\aspell-0.60`

이 디렉토리에 `.alias`, `.multi`, `.dat`, `.rws`, `.cmap`, `.cset` 파일을 모두
복사해서 넣으면 된다.

그리고 명령프롬프트에서 사용하려면 환경변수 `PATH`에 `C:\Program
Files\Aspell\bin`을 추가해주는 것이 좋다.

### Notepad++의 경우

Notepad++의 Spell-Checker 플러그인은 aspell 0.5 버전만 지원한다. aspell 0.6
버전을 설치하면 작동하지 않는다. DLL파일의 이름이 0.5버전에서
`aspell-15.dll`인데 aspell 0.60에서는 `libaspell-15.dll`로 바뀌었기 때문이다.

`C:\Program Files\Aspell\bin` 디렉토리를 열어보면 `libaspell-15.dll`라는 파일이
있다. 이것을 복사하여 `aspell-15.dll`라는 이름으로 하나 더 만든 후에 넣어주면
Notepad++에서도 잘 작동한다.


