% Golang
% Jonhnny Weslley
% 20/09/2014

# Golang

## O que é Go?

- Linguagem de programação open-source
- Compilada (compilação rápida)
- Estaticamente tipada   \\o/
- Cross-compilation (32-bit, 64-bit, ARM)
    - FreeBSD, Linux, Mac OS X, Windows e android (go 1.4)
- Garbage collector
- Sistema de tipos simples
- Possui primitivas de concorrência

## Origem

- Criada pelo Google
- Iniciada em 2007 por Robert Griesemer, Rob Pike e Ken Thompson.
- Liberada em 2009 sob BSD-license
- Versão 1.0 em Março de 2012

![Gopher](images/gopher.jpg)

## Quem usa?

- Google (YouTube, dl.google.com, ...)
- dotCloud (Docker)
- SoundCloud
- Canonical
- CloudFlare
- Mozilla
- Hashicorp
- ...
- [http://golang.org/wiki/GoUsers]()

## Quem usa?

![Google Trends](images/trends.png)

## Por que eu gosto de Go?

- Simplicidade
- Desempenho
- Binários estáticos (facilita deploys)
- Excelente biblioteca padrão
- Orientada a objetos ??
- Interfaces implicitas
- Ferramentas
    - go build, got fmt, go test, go cover, ...

## O que não é tão legal (ainda)?

- GC
- Gerenciamento de dependências
- Tratamento de erros

# Use the source, Luke!

## Hello World!

```golang
package main

import "fmt"

func main() {
    quem := "Gurupi"
    fmt.Printf("Fala galera do %s!\n", quem)
}
```

```sh
go run hello.go
```

## Tipos

- Tipos primitivos

```
int, uint, int8, uint8, ...
bool, string
float32, float64
complex64, complex128
```

- slices, arrays e maps

```
[]int, [3]string, []struct{ Name string }
map[string]int
```

- structs

```golang
type Proxy struct {
  httputil.ReverseProxy
  servers Servers
  tld     string
}
```

## Tipos

- pointers

```
*int, *Proxy
```

- funções

```golang
func AddrPort(addr string) (int, error)
```

- channels

```
chan bool
```

- interfaces

```golang
type Server interface {
  Name() string
  Port() int
}
```

# Interfaces

## Duck typing

- São implicitas

```golang
// Interface do pacote `fmt`
type Stringer interface {
    String() string
}
```

```golang
type server struct {
  name string
  port int
}
func (s *server) Name() string {
  return s.name
}
func (s *server) Port() int {
  return s.port
}
func (s *server) String() string {
  return fmt.Sprintf("%s:%d", s.name, s.port)
}
```

```golang
  s := server{"localhost", 3000}
  fmt.Printf("Started at %s\n", s)
```

## Composição de interfaces

```golang
package io
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {
    Reader
    Writer
}
```

## Encadeamento de io.Readers

```golang
package main

import (
    "compress/gzip"
    "encoding/base64"
    "io"
    "os"
    "strings"
)

func main() {
    var r io.Reader
    r = strings.NewReader(data)
    r = base64.NewDecoder(base64.StdEncoding, r)
    r, _ = gzip.NewReader(r)
    io.Copy(os.Stdout, r)
}
```

## Programação para Web

```golang
package main

import (
 "log"
 "net/http"
)

func Collector(w http.ResponseWriter, req *http.Request) {
  defer req.Body.Close()
  monit := MonitFromXml(req.Body)
  queue <- &monit
}

func main() {
    http.HandleFunc("/collector", Collector)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## Http handlers

```golang
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

```golang
type dogpack struct {}

func (h *dogpack) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Processamento da requisição
}

func main() {
    http.Handle("/", &dogpack{})
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## HTTP HandlerFunc

```golang
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

## Compondo funcionalidades

HTTP error handlers

```golang
type errorHandler func(http.ResponseWriter, *http.Request) error

func handleError(f errorHandler) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        err := f(w, r)
        if err != nil {
            log.Printf("%v", err)
            http.Error(w, "Oops!", http.StatusInternalServerError)
        }
    }
}
```

## HTTP error handlers

```golang
func handler(w http.ResponseWriter, r *http.Request) error {
    name := r.FormValue("name")
    if name == "" {
        return fmt.Errorf("empty name")
    }
    fmt.Fprintln(w, "Hi,", name)
    return nil
}
```

```golang
http.HandleFunc("/form", handleError(handler))
```

# Concorrência

--------

> Don't communicate by sharing memory.
> Instead, share memory by communicating.

## Concorrência

- Goroutines e channels
- Inspirada por CSP

## Goroutines

- Não são threads
- Muitas goroutines podem executar na mesma thread

```golang
i := pivot(s)
go sort(s[:i])
go sort(s[i:])
```

## Channels

- Servem para sincronização e comunicação entre goroutines
- Operador `<-` é utilizado para enviar e receber valores

```golang
func compute(ch chan int) {
    ch <- someComputation()
}

func main() {
    ch := make(chan int)
    go compute(ch)
    result := <-ch
}
```

## Sincronização

- (Un)buffered channels

```golang
done := make(chan bool)
doSort := func(s []int) {
 sort(s)
 done <- true
}

i := pivot(s)
go doSort(s[:i])
go doSort(s[i:])
<-done
<-done
```

## Comunicação

```golang
type Work struct { x, y, z int }

func worker(in <-chan *Work, out chan <- *Work) {
    for w := range in {
        w.z = w.x * w.y
        out <- w
    }
}

func Run() {
    in, out := make(chan *Work), make(chan *Work)
    for i := 0; i < 10; i++ {
        go worker(in, out)
    }
    go sendLotsOfWork(in)
    receiveLotsOfResults(out)
}
```

# Workspace

## Ferramentas

- Compila e executa

```
go run hello.go
```

- Executa os testes

```
go test
```

- Gera binário e formata código

```
go build
go fmt
```

- Baixa e instala dependências

```
go get github.com/jweslley/procker
```

## Organização do Workspace

- Organizado automaticamente pelas ferramentas de build

```
workspace/
  bin # executable binaries
  pkg # compiled object files
  src # source code
    github.com/jweslley/
      procker/

```

```golang
import "github.com/jweslley/procker"
```

## Criando um workspace

Crie o diretorio em local de preferência

```
mkdir -p $HOME/gocode/src
```

Exporte a variavel `GOPATH`

```
export GOPATH=$HOME/gocode
```

Adicione o `GOPATH/bin` ao `PATH`

```
export PATH=$PATH:$GOPATH/bin
```

## Trabalhando no workspace

- Escolha um *namespace*
    `github.com/jweslley` no meu caso

- Crie um diretório e programe alguma coisa

```
mkdir $GOPATH/src/github.com/jweslley/hello
cp hello.go $GOPATH/src/github.com/jweslley/hello
```

- Crie e instale o binário

```
go install github.com/jweslley/hello
```

- Execute-o

```
$GOPATH/bin/hello
```

## Mais informações

- [http://golang.org/]()
- [Documentation](http://golang.org/doc/)
- [A Tour of Go](http://tour.golang.org/)
- [Go playground](https://play.golang.org/)
- [How to Write Go Code](http://golang.org/doc/code.html)
- [Effective Go](http://golang.org/doc/effective_go.html)
- [Go by example](https://gobyexample.com/)
- [Go Bootcamp book](http://www.golangbootcamp.com/book/)


## Obrigado!

- Jonhnny Weslley
- @jweslley
- [http://jweslley.github.io/talks/gurupi-golang]()
