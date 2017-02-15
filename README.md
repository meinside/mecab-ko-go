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
	"strings"

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
		features := strings.Split(node.Feature(), ",")

		// XXX - 품사 태그 설명:
		// https://docs.google.com/spreadsheets/d/1-9blXKjtjeKZqsf4NzHeYJCrr49-nXeRF6D80udfcwY/edit#gid=589544265
		if features[0] == "NNG" {
			fmt.Printf("[명사] %s (%s)\n", node.Surface(), node.Feature())
		} else {
			fmt.Printf("[기타] %s (%s)\n", node.Surface(), node.Feature())
		}

		if node.Next() != nil {
			break
		}
	}
}
```

It will output:

```
[기타]  (BOS/EOS,*,*,*,*,*,*,*)
[명사] 아버지 (NNG,*,F,아버지,*,*,*,*)
[기타] , (SC,*,*,*,*,*,*,*)
[기타] 지금 (MAG,성분부사/시간부사,T,지금,*,*,*,*)
[기타] 이 (MM,~명사,F,이,*,*,*,*)
[기타] 몇 (MM,~가산명사,T,몇,*,*,*,*)
[기타] 시 (NNBC,*,F,시,*,*,*,*)
[기타] 인지 (VCP+EC,*,F,인지,Inflect,VCP,EC,이/VCP/*+ᆫ지/EC/*)
[기타] 나 (JX,*,F,나,*,*,*,*)
[기타] 알 (VV,*,T,알,*,*,*,*)
[기타] 고 (EC,*,F,고,*,*,*,*)
[명사] 가방 (NNG,*,T,가방,*,*,*,*)
[기타] 에 (JKB,*,F,에,*,*,*,*)
[기타] 들어가 (VV,*,F,들어가,*,*,*,*)
[기타] 십니까 (EP+EF,*,F,십니까,Inflect,EP,EF,시/EP/*+ᄇ니까/EF/*)
[기타] ? (SF,*,*,*,*,*,*,*)
[기타]  (BOS/EOS,*,*,*,*,*,*,*)
```

## Original Author

**Jun Kimura**

* <http://github.com/bluele>
* <junkxdev@gmail.com>

## License

MIT

