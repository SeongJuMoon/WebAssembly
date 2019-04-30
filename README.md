### WebAssembly

그냥 심심해서 튜로리얼 레포로 끄적이고 있는 고랭으로 웹 어셈블리 한번 해보겠습니다?

issue && PR은 언제든지 환영합니다..!!

요구 사항

Go 1.11 버전 이상

 - Go 설치

```bash
    sudo apt-get update
    sudo apt-get -y upgrade

    cd /tmp
    wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz


    sudo tar -xvf go1.11.linux-amd64.tar.gz
    sudo mv go /usr/local
```

1. 웹 어셈블리란?

브라우저의 퍼포먼스 향상을 제고하기 위해 네이티브의 작성한 소스코드를 .wasm로 작성하여 자바스크립트 인터페이스에서 wasm 바이너리에서 호출하는 것을 의미한다..

## 웹 어셈블리의 목표

 1. (동작이) 빠르고, (성능이) 효과적이고, (코드가)이식성이 좋을 것 
    웹어셈블리 코드는 일반적인 하드웨어들이 제공하는 기능을 활용하여 여러종류의 플랫폼 위에서 거의 네이티브에 가까운 속도로 실행될 수 있습니다.

2. 읽기 쉽고 디버깅이 가능할 것 — 웹어셈블리는 저수준 어셈블리 언어지만, 손으로 작성하고, 보고, 디버깅할 수는 있도록, 사람이 충분히 읽을 수 있는 수준의 텍스트 포맷을 갖고있습니다.

3. 안전함을 유지할 것 — 웹어셈블리는 **샌드박싱된 실행환경에서 안전하게** 돌아갈 수 있도록 설계되었습니다. 웹상의 다른 코드와 마찬가지로, 웹어셈블리 코드도 브라우저의 동일한 출처(same-origin)와 권한정책을 강제할 것입니다.

4. 웹을 망가뜨리지 않을 것 — 웹어셈블리는 다른 웹 기술과 마찰없이 사용되면서 하위호환성을 관리할 수 있도록 설계되었습니다.

### NOTE:: 자세한 사항은 다음에 [공식 홈페이지](https://developer.mozilla.org/ko/docs/WebAssembly/Concepts)을 참고하세요!


2. 빌드 방법
이 프로젝트에서는 간단한 println코드를 가지고 .wasm을 만들어 보는걸 목적으로 합니다.

우리의 main.go는 다음과 같이 작성되있습니다.

```go
    package main

    import "fmt"

    func main() {
        fmt.Println("hello webasm")
    }

```

wasm 패키지 경로에서 다음의 명령어 수행합니다

```bash
$ GOOS=js GOARCH=wasm go build -o main.wasm
```

이후 (go env GOROOT)에 존재하는 /misc/wasm/wasm_exec.js를 view 디렉토리로 복사해옵니다.


3. 실행 방법
view 디렉토리 안에 index.html을 작성합니다.

```html
<html>
	<head>
		<meta charset="utf-8">
		<script src="wasm_exec.js"></script>
		<script>
			const go = new Go();
			WebAssembly.instantiateStreaming(fetch("main.wasm"), go.importObject).then((result) => {
				go.run(result.instance);
			});
		</script>
	</head>
	<body></body>
</html>
```
그 다음 같은 디렉토리 안에 http.go를 작성합니다.

```go
package main

import (
	"flag"
	"log"
	"net/http"
)

var (
	listen = flag.String("listen", ":8080", "listen address")
	dir    = flag.String("dir", ".", "directory to serve")
)

func main() {
	flag.Parse()
	log.Printf("listening on %q...", *listen)
	err := http.ListenAndServe(*listen, http.FileServer(http.Dir(*dir)))
	log.Fatalln(err)
}

```

이후 http://localhost:8080 에 접속해보면 콘솔에서 fmt.Println()의 코드가 실행되고 있을것입니다.!!


4. 더 알아보기
[MDN 예제](https://github.com/mdn/webassembly-examples)
[golang wiki](https://github.com/golang/go/wiki/WebAssembly)
