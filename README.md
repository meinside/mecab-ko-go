# mecab-ko-go

A Golang wrapper for [mecab-ko](https://bitbucket.org/eunjeon/mecab-ko).

Forked from [mecab-golang](https://github.com/bluele/mecab-golang).

## Dependencies

`mecab-ko-go` needs [mecab-ko](https://bitbucket.org/eunjeon/mecab-ko) and [mecab-ko-dic](https://bitbucket.org/eunjeon/mecab-ko-dic).

### Install mecab-ko

Clone repository,

```bash
$ git clone git@bitbucket.org:eunjeon/mecab-ko.git
```

build, and install it:

```bash
$ cd mecab-ko
$ ./configure
$ make
$ make check
$ umask 0022
$ sudo make install
```

### Install mecab-ko-dic

Download dictionary file,

```bash
$ wget https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.0.1-20150920.tar.gz
```

unzip it,

```bash
$ tar -xzvf mecab-ko-dic-2.0.1-20150920.tar.gz
```

and install it:

```bash
$ cd mecab-ko-dic-2.0.1-20150920
$ ./configure
$ make
$ sudo make install
```

If it fails with ``automake``, run ``autogen.sh`` before ``configure`` like this:

```bash
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
```

(Only on Linux) if it still fails with ``libmecab.so.2``, do the following and try again:

```bash
$ sudo ldconfig
```

## Build/Install

### If mecab-config is installed,

If you have `mecab-config` installed, do the following:

```bash
$ export CGO_LDFLAGS="`mecab-config --libs`"
$ export CGO_FLAGS="`mecab-config --inc-dir`"
```

### If not,

Do the following:

```bash
$ export CGO_LDFLAGS="-L/path/to/lib -lmecab -lstdc++"
$ export CGO_CFLAGS="-I/path/to/include"
```

You should change `/path/to` to yours, like:

```bash
# (example)
$ export CGO_LDFLAGS="-L/usr/local/lib -lmecab -lstdc++"
$ export CGO_CFLAGS="-I/usr/local/include"
```

Now install `mecab-ko-go` with following command:

```bash
$ go get -u github.com/meinside/mecab-ko-go
```

## Examples

```go
package main

import (
	"fmt"

	"github.com/meinside/mecab-ko-go"
)

const (
	sampleSentence = "아버지, 지금이 몇 시인지나 알고 가방에 들어가십니까?"
)

func main() {
	m, err := mecab.New()
	if err != nil {
		panic(err)
	}
	defer m.Destroy()

	tg, err := m.NewTagger()
	if err != nil {
		panic(err)
	}
	defer tg.Destroy()

	lt, err := m.NewLattice(sampleSentence)
	if err != nil {
		panic(err)
	}
	defer lt.Destroy()

	node := tg.ParseToNode(lt)
	for {
		feature := node.Feature()
		tag, _ := mecab.NewTag(feature)

		if tag.Tag == "NNG" {
			fmt.Printf("[명사] %s (%s)\n", node.Surface(), feature)
		} else {
			fmt.Printf("[기타/%s] %s (%s)\n", tag.Description, node.Surface(), feature)
		}

		if node.Next() != nil {
			break
		}
	}
}
```

It will output:

```
[기타/문장 시작, 끝]  (BOS/EOS,*,*,*,*,*,*,*)
[명사] 아버지 (NNG,*,F,아버지,*,*,*,*)
[기타/구분자] , (SC,*,*,*,*,*,*,*)
[기타/일반 부사] 지금 (MAG,성분부사/시간부사,T,지금,*,*,*,*)
[기타/관형사] 이 (MM,~명사,F,이,*,*,*,*)
[기타/관형사] 몇 (MM,~가산명사,T,몇,*,*,*,*)
[기타/단위를 나타내는 명사] 시 (NNBC,*,F,시,*,*,*,*)
[기타/알 수 없음] 인지 (VCP+EC,*,F,인지,Inflect,VCP,EC,이/VCP/*+ᆫ지/EC/*)
[기타/보조사] 나 (JX,*,F,나,*,*,*,*)
[기타/동사] 알 (VV,*,T,알,*,*,*,*)
[기타/연결 어미] 고 (EC,*,F,고,*,*,*,*)
[명사] 가방 (NNG,*,T,가방,*,*,*,*)
[기타/부사격 조사] 에 (JKB,*,F,에,*,*,*,*)
[기타/동사] 들어가 (VV,*,F,들어가,*,*,*,*)
[기타/알 수 없음] 십니까 (EP+EF,*,F,십니까,Inflect,EP,EF,시/EP/*+ᄇ니까/EF/*)
[기타/마침표, 물음표, 느낌표] ? (SF,*,*,*,*,*,*,*)
[기타/문장 시작, 끝]  (BOS/EOS,*,*,*,*,*,*,*)
```

## Original Author

**Jun Kimura**

* <http://github.com/bluele>
* <junkxdev@gmail.com>

## License

MIT

