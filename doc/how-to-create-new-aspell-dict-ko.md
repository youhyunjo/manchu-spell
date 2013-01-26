Aspell: 새로운 언어 사전 만들기
================================


기본 절차
--------------------------------

### aspell-lang

우선 aspell-lang을 다운받아야 한다. 다음처럼 다운받아 압축을 풀거나

    $ curl -O ftp://ftp.gnu.org/gnu/aspell/aspell-lang-20101122.tar.gz
    $ tar xzvf aspell-lang-20101122.tar.gz

또는 CVS 저장소를 내려받는다.

    $ cvs -z3 -d:pserver:anonymous@cvs.savannah.gnu.org:/sources/aspell co aspell-lang

유니코드 문자를 사용하고 있다면 `u-.cmap`, `u-.cset` 파일을 만들어야한다.


### 사전 디렉토리

우선 새로운 언어 사전 디렉토리를 만들어야한다. 언어 코드는 ISO 639를 따른다. 두
글자짜리가 있으면 사용하고 없으면 세 글자짜리를 쓴다. 예를 들어, 만주어는
`mnc`이므로 `aspell6-mnc`라는 디렉토리를 만들면 된다.

    $ mkdir aspell6-mnc

이 디렉토리 안에 모든 파일을 만들어 넣으면 된다. 필수적으로 필요한 파일은 다음과 같다.

* `COPYING` - 라이선스 파일. 
* `Copyright` - 저작권.
* `info` 
* `<lang>.dat` - 예를 들어, 만주어라면 `mnc.dat`
* `<lang>.wl` - 단어 목록. 예를 들어, 만주어라면 `mnc.wl`
* `<lang>_affix.dat` - 접사 목록. 꼭 필수는 아니지만 대개 필요하다. 만주어의 경우에는 `mnc_affix.dat`

### `info`

가장 먼저 만들 것은 `info`파일이다.  이 파일에는 해당 사전에 대한 정보가 기록된다. 예를 들어, 만주어의 경우 다음과 같이 만들 수 있다.

    name_english Manchu
    lang mnc
    data-file mnc_affix.dat
    author:
      name You Hyun Jo
      email youhyunjo at gmail com
    copyright GPLv3
    url https://github.com/youhyunjo/manchu-spell/aspell-mnc
    version 20130123-0
    source-version 20130123
    complete false
    accurate true
    alias mnc manchu
    dict:
      name mnc
      add mnc

이때 `data-file`에 명시한 파일도 추가적으로 준비해야한다. 단어목록으로 모든 것이 해결되는 언어라면 데이터 파일을 추가할 필요없다.


### 라이선스와 저작권

다음으로 라이선스를 담은 `COPYING` 파일을 넣는다. 예를 들어, GPLv3를 다운받아 넣으면 된다. 
저작권을 담은 `Copyright` 파일도 있어야한다. 적당히 작성하면 된다.

### `<lang>.dat`

이제 `.dat` 파일을 넣어야한다. 포함될 내용은 내용은 간단하다. 만주어의 경우에는
`mnc.dat`가 되고 다음과 같은 내용이 들어간다.

    name mnc
    charset iso-8859-1
    affix mnc
    affix-compress true
    partially-expand true
    repl-table mnc_affix.dat


### `proc`

이 파일들이 모두 준비되었다면 앞서 다운로드 받은 aspell-lang에서 `proc` 파일을
복사하여 현재 디렉토리에 넣고 다음 명령을 내리자.

    $ ./proc create

여러 가지 필요한 파일들이 만들어진다. `README`, `configure`, `Makefile.pre` 등
여러 파일이 만들어진다.


### 설치

진짜 사전이 되려면 물론 단어 목록(word list)을 담은 `<lang>.wl` 파일과
접사목록을 담은 `<lang>_affix.dat`를 추가해야한다. 만주어의 경우에는 `mnc.wl`과
`mnc_affix.dat`가 된다. 일단 두 파일이 이미 있다고 가정하자. 다음 명령을 내리면
`.wl` 파일에서 `.cwl` 파일을 만들어 준다. `.cwl` 확장자는 압축된 단어
목록(compressed word list)라는 의미이다.

    $ ./configure
    $ make

만들어진 사전을 설치하려면 다음 명령을 내리면 된다.

    $ make install




단어 목록과 접사 규칙 만들기
--------------------------------

단어목록 파일 `<lang>.wl`은 한 행에 한 단어를 담고 있는 파일이다. 단어의 형태가
변하지 않는다면 그냥 단어 목록을 만드는 것으로 충분하다. 단어가 모양이 변한다면
단어목록과 접사규칙을 만들어야한다.


예를 들어, 만주어 단어 arambi가 arame, arafi, araha, arara, araci로 변한다고
하자. ara-가 어간이고 -mbi, -me, -fi, -ha, -ra, -ci가 어미이다. 또, genembi는
geneme, genefi, genehe, genere, geneci로 변한다고 하자. 이때 `mnc.wl`을

    ara/v
    gene/v

와 같이 만든다. 여기서 `/v`는 접사의 유형을 나타내기 위한 장치이다. 한 글자로
접사의 유형을 나타내면된다. `mnc_affix.dat`에는 `v`라는 접사에 대한 규칙을
기술해야한다. 예를 들면 다음과 같다.

    SFX v Y 6
    SFX v 0 mbi .
    SFX v 0 me .
    SFX v 0 fi .
    SFX v 0 ha a
    SFX v 0 he e
    SFX v 0 ra a
    SFX v 0 re e
    SFX v 0 ci .

첫 행의 `SFX`는 접미사(suffix)라는 의미이다. `v`는 접사유형을 구분하기 위한
이름이다. `Y`는 접미사와 함께 쓰일 수 있다는 의미이다. `6`은 해당 접사유형에
6행의 정보가 있다는 의미이다.

두번째 행에서 `SFX v` 다음에 세 가지 항목이 기술된다. 뺄 것, 더할 것, 조건.
이렇게 세가지이다. `0`은 어간의 끝에서 아무 것도 빼지않는다는 의미이다. `mbi`는
어간에 어미 mbi를 붙이라는 의미이다. `.`은 `mbi` 어미가 붙을 수 있는 조건에
제약이 없다는 의미이다.

다섯번째 행에서 `0 ha a`는 어간이 a로 끝나는 조건에서 아무 것도 빼지 않고 ha를
덧붙이라는 의미이다. 이 규칙에 따라 ara는 araha로 변한다. 여섯번째 행에서 `0
he e`는 어간이 e로 끝나는 조건에서 아무 것도 빼지 않고 he를 덧붙이라는
의미이다. 이 규칙에 따라 gene는 genehe가 된다.


사용하기
--------------------------------

다음 명령은 설치되어 있는 사전의 목록을 출력해준다. 

    $ aspell dump dicts

설치한 사전의 어휘 목록을 출력해보자. `-l <lang>` 옵션으로 언어를 지정한다. 만주어라면:

    $ aspell -l mnc dump master

설치된 사전이 작동하는지 알아보기 위하여 `expand`와 `munch`를 사용해 볼 수 있다.

    $ echo 'ara/v' | aspell -l mnc expand 
    $ echo 'araha' | aspell -l mnc munch

철자법 검사를 해보고 싶다면:

    $ echo 'arahe' | aspell -l mnc -a


