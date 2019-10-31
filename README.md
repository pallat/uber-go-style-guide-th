<!--

Editing this document:

- Discuss all changes in GitHub issues first.
- Update the table of contents as new sections are added or removed.
- Use tables for side-by-side code samples. See below.

Code Samples:

Use 2 spaces to indent. Horizontal real estate is important in side-by-side
samples.

For side-by-side code samples, use the following snippet.

~~~
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
BAD CODE GOES HERE
```

</td><td>

```go
GOOD CODE GOES HERE
```

</td></tr>
</tbody></table>
~~~

(You need the empty lines between the <td> and code samples for it to be
treated as Markdown.)

If you need to add labels or descriptions below the code samples, add another
row before the </tbody></table> line.

~~~
<tr>
<td>DESCRIBE BAD CODE</td>
<td>DESCRIBE GOOD CODE</td>
</tr>
~~~

-->

# Uber Go Style Guide

## Table of Contents

- [Introduction](#introduction)
- [Guidelines](#guidelines)
  - [Pointers to Interfaces](#pointers-to-interfaces)
  - [Receivers and Interfaces](#receivers-and-interfaces)
  - [Zero-value Mutexes are Valid](#zero-value-mutexes-are-valid)
  - [Copy Slices and Maps at Boundaries](#copy-slices-and-maps-at-boundaries)
  - [Defer to Clean Up](#defer-to-clean-up)
  - [Channel Size is One or None](#channel-size-is-one-or-none)
  - [Start Enums at One](#start-enums-at-one)
  - [Error Types](#error-types)
  - [Error Wrapping](#error-wrapping)
  - [Handle Type Assertion Failures](#handle-type-assertion-failures)
  - [Don't Panic](#dont-panic)
  - [Use go.uber.org/atomic](#use-gouberorgatomic)
- [Performance](#performance)
  - [Prefer strconv over fmt](#prefer-strconv-over-fmt)
  - [Avoid string-to-byte conversion](#avoid-string-to-byte-conversion)
  - [Prefer Specifying Map Capacity Hints](#prefer-specifying-map-capacity-hints)
- [Style](#style)
  - [Be Consistent](#be-consistent)
  - [Group Similar Declarations](#group-similar-declarations)
  - [Import Group Ordering](#import-group-ordering)
  - [Package Names](#package-names)
  - [Function Names](#function-names)
  - [Import Aliasing](#import-aliasing)
  - [Function Grouping and Ordering](#function-grouping-and-ordering)
  - [Reduce Nesting](#reduce-nesting)
  - [Unnecessary Else](#unnecessary-else)
  - [Top-level Variable Declarations](#top-level-variable-declarations)
  - [Prefix Unexported Globals with _](#prefix-unexported-globals-with-_)
  - [Embedding in Structs](#embedding-in-structs)
  - [Use Field Names to Initialize Structs](#use-field-names-to-initialize-structs)
  - [Local Variable Declarations](#local-variable-declarations)
  - [nil is a valid slice](#nil-is-a-valid-slice)
  - [Reduce Scope of Variables](#reduce-scope-of-variables)
  - [Avoid Naked Parameters](#avoid-naked-parameters)
  - [Use Raw String Literals to Avoid Escaping](#use-raw-string-literals-to-avoid-escaping)
  - [Initializing Struct References](#initializing-struct-references)
  - [Initializing Maps](#initializing-maps)
  - [Format Strings outside Printf](#format-strings-outside-printf)
  - [Naming Printf-style Functions](#naming-printf-style-functions)
- [Patterns](#patterns)
  - [Test Tables](#test-tables)
  - [Functional Options](#functional-options)

## Introduction

สไตล์เป็นเหมือนข้อตกลงที่ช่วยจัดระเบียบโค้ดของเรา แต่คำว่าสไตล์ก็อาจจะทำให้สับสนนิดหน่อย เพราะ ข้อตกลงนี้มันครอบคลุมไปมากกว่าแค่เรื่องไฟล์ซอสโค้ด เพราะถ้าเป็นอย่างนั้น gofmt ก็จัดการให้เราได้อยู่แล้ว

เป้าหมายของคำแนะนำชุดนี้ คือการลดความซับซ้อนด้วยการอธิบายว่าที่ Uber เราทำ หรือไม่ทำอะไรตอนที่เราเขียน Go กันบ้าง และกฎนี้มีไว้เพื่อช่วยให้โค้ดมันดูแลจัดการได้ง่าย ในขณะที่ก็ยอมให้วิศกรซอฟท์แวร์ใช้มันได้อย่างมีประสิทธิภาพด้วย

คำแนะนำชุดนี้เดิมถูกเขียนขึ้นโดย [Prashant Varanasi] และ [Simon Newton] เพื่อช่วยให้เพื่อนร่วมงานเริ่มต้นเขียน Go กันได้เร็วขึ้น แต่หลังจากผ่านไปหลายปี มันก็ถูกแก้ไขเพิ่มเติมจากข้อเสนอแนะต่างๆที่ได้รับ

  [Prashant Varanasi]: https://github.com/prashantv
  [Simon Newton]: https://github.com/nomis52

สำนวนการเขียน Go ในเอกสารนี้เป็นแบบฉบับที่ใช้กันที่ Uber ซึ่งปกติก็เป็นแนวทางเดียวกับการเขียน Go ทั่วไปอยู่แล้ว ซึ่งถ้าจะมีเพิ่มเติมจากภายนอกก็มาจากที่เหล่านี้:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [The Go common mistakes guide](https://github.com/golang/go/wiki/CodeReviewComments)

โค้ดทั้งหมดควรจะต้องไม่มี error ใดๆจาก `golint` และ `go vet` เราแนะนำให้คุณตั้งค่าใน editor ตามนี้:

- Run `goimports` on save
- Run `golint` and `go vet` to check for errors

คุณสามารถหาข้อมูลเพิ่มเติมเกี่ยวกับการเครื่องมือช่วยใน editors ได้จากที่นี่:
<https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins>

## Guidelines

### Pointers to Interfaces

คุณแทบไม่จำเป็นต้องใช้พอยเตอร์เพื่อใส่ใน interface คุณแค่ส่งค่าตรงๆเข้าไป แต่จะส่งเป็นพอยเตอร์ก็ได้เช่นกัน

interface ประกอบไปด้วยสองสิ่ง:

1. พอยเตอร์ ชี้ไปที่ type ของสิ่งที่เก็บ คุณจะคิดซะว่ามันเป็น "type" เลยก็ได้
2. พอยเตอร์ ของสิ่งที่เก็บ ถ้าสิ่งนั้นเป็นพอยเตอร์ ก็จะเก็บตรงๆ แต่ถ้ามันเป็นค่าใดๆก็ตาม มันจะเก็บเป็นพอยเตอร์ของค่านั้นแทน
You almost never need a pointer to an interface. You should be passing
interfaces as values—the underlying data can still be a pointer.

ถ้าคุณต้องการให้เมธอดแก้ไขค่าในตัวมันเองได้ด้วย นั่นคุณถึงจะต้องใช้พอยเตอร์

### Receivers and Interfaces

เมธอดที่มีตัวรับเป็นค่าปกติ สามารถเรียกใช้บนตัวแปรพอยเตอร์ ได้เลย

For example,

```go
type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// คุณอเรียกใช้ Read ได้อย่างเดียว
sVals[1].Read()

// This will not compile:
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// คุณเรียกใช้ได้ทั้ง Read และ Write ผ่านพอยเตอร์
sPtrs[1].Read()
sPtrs[1].Write("test")
```

และในทางกลับกัน interface ยอมให้คุณแทนที่ด้วยพอยเตอร์ได้ แม้ว่าเมธอดจะใช้ตัวรับเป็นแค่ค่าปกติ

```go
type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// โค้ดด้านล่างนี้ไม่สามารถทำงานได้ เนื่องจาก s2Val เป็นค่าปกติ ในขณะที่ตัวรับในเมธอดไม่ใช่ค่าปกติแต่เป็นพอยเตอร์
```

Effective Go เขียนเรื่องนี้ไว้ได้ดีมากในเรื่อง [Pointers vs. Values]

  [Pointers vs. Values]: https://golang.org/doc/effective_go.html#pointers_vs_values

### Zero-value Mutexes are Valid

ค่า zero-value ของ `sync.Mutex` และ `sync.RWMutex` เป็นแบบสัมบูรณ์ นั่นแปลว่าคุณแทบไม่ต้องใช้พอยเตอร์กับ mutex เลย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
mu := new(sync.Mutex)
mu.Lock()
```

</td><td>

```go
var mu sync.Mutex
mu.Lock()
```

</td></tr>
</tbody></table>

ถ้าคุณใช้ struct ด้วยพอยเตอร์ mutex จะสามารถเป็นแบบ ไม่มีพอยเตอร์ให้

struct ที่ไม่ได้เปิดเผยสู่ภายนอกที่ใช้ mutex ปกป้องฟิลด์ในตัวมันเอง อาจจะฝัง mutext ไว้แบบนี้

<table>
<tbody>
<tr><td>

```go
type smap struct {
  sync.Mutex // ใช้เฉพาะ type ที่ไม่เปิดเผยสู่ภายนอก

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

</td><td>

```go
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

</td></tr>

</tr>
<tr>
<td>การฝัง ใช้กับ type ที่อยู่ภายใน หรือ type ที่ต้องการทำตัวเองเป็น Mutext interface</td>
<td>สำหรับ type ที่ต้องการเปิดเผยสู่ภายนอก ให้ใช้แบบ ฟิลด์ ภายใน struct</td>
</tr>

</tbody></table>

### Copy Slices and Maps at Boundaries

Slices และ maps เก็บของเป็นพอยเตอร์ ดังนั้นให้ระมัดระวังเวลาที่จะ copy ค่าเหล่านี้

#### Receiving Slices and Maps

ต้องจำไว้นะว่า map หรือ slice ที่คุณรับเข้ามาเป็นอากิวเม้นต์ ก็ถูกคนที่ใช้มันแก้ไขได้ ถ้าคุณเก็บข้อมูลชนิดที่มันอ้างถึงกัน

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// คุณต้องการจะแก้ไขค่า d1.trips หรือเปล่า?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// ตอนนี้เราก็สามารถแก้ไขค่า trips[0] โดยไม่กระทบ d1.trips ได้แล้ว
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps

ในทางกลับกัน ให้ระมัดระวังการแก้ไขค่าไปที่ map หรือ slices ที่เปิดเผยสู่ภายนอกในระดับภายใน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot คืน ค่า ณ เวลาปัจจุบัน
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot ไม่ถูกป้องกันโดย mutex ดังนั้น
// ใครก็ตามที่เข้ามาถึง snapshot อาจจะเกิดการแย่งของกัน
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Defer to Clean Up

ใช้ defer เพื่อทำความสะอาดพวกทรัพยากรณ์ต่างๆเช่น ไฟล์ และ การล็อคต่างๆ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// มันง่ายที่ลืมแก้ล็อค เวลาที่มีการรีเทิร์นหลายๆที่
```

</td><td>

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

</td></tr>
</tbody></table>

Defer ใช้เวลาทำงานน้อยมาก ถ้าจะไม่ใช้มันก็ต่อเมื่อคุณมั่นใจแล้วว่าฟังก์ชั่นคุณจะทำงานเร็วในระดับ nanoseconds ถ้าคุณใช้ defer มันอ่านง่ายแน่นอนและคุ้มค่าที่จะใช้ โดยเฉพาะอย่างยิ่งเมื่อคุณมีเมธอดขนาดใหญ่ที่มีการใช้หน่วยความจำแบบท่ายาก และมีการคำนวณอย่างอื่นที่สำคัญกว่า การใช้ `defer`

### Channel Size is One or None

Channels ปกติควรมีขนาดอยู่ที่ 1 หรือไม่มีบัพเฟอร์เลย โดยค่าตั้งต้น channels จะเป็นแบบไม่มีบัพเฟอร์ และมีขนาดเป็นศูนย์ ขนาดอื่นๆ ขึ้นอยู่กับวิจารณญาณ ขึ้นอยู่กับว่า จะป้องกันการเติมของ ในขณะที่กำลังโหลด และมีการเขียน อย่างไร

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// หวังว่าจะพอสำหรับทุกคนนะ!
c := make(chan int, 64)
```

</td><td>

```go
// ให้ขนาดเป็นหนึ่ง
c := make(chan int, 1) // or
// ไม่มีตัวกันชนเลย หรือมีขนาดเท่ากับศูนย์
c := make(chan int)
```

</td></tr>
</tbody></table>

### Start Enums at One

วิธีมาตรฐานในการทำ enum ใน go คือการ สร้าง type ขึ้นมาเอง หรือประกาศเป็นกลุ่ม `const` ด้วยการใช้ `iota` ซึ่งโดยปกติตัวแปรจะมีค่าตั้งต้นเป็น 0 เสมอ เพราะฉะนั้นเวลาที่คุณจะทำ enum ควรจะเริ่มด้วยค่าที่ไม่ใช่ศูนย์นะ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

มันก็มีบางกรณีเหมือนกันที่การใช้ศูนย์อาจจะเหมาะสมกว่า ขึ้นอยู่กับสถานการณ์

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

<!-- TODO: section on String methods for enums -->

### Error Types

การสร้าง error ทำได้หลายวิธี:

- [`errors.New`] เมื่อสร้างจากสตริงง่ายๆ
- [`fmt.Errorf`] เมื่อต้องการจัดรูปแบบข้อความ
- สร้าง type ที่ implement `Error` เมธอด
- หุ้ม error ด้วยการใช้ [`"pkg/errors".Wrap`]

เมื่อจะทำการคืน errors ทางเลือกไหนถึงจะดีที่สุด ลองตั้งคำถามดูว่า:

- นี่เป็น error ที่ต้องการข้อมูลเพิ่มเป็นพิเศษไหม ถ้าไม่ ก็ใช้ [`errors.New`] ก็น่าจะพอแล้ว
- คนที่จะเอา error นี้ไปใช้ต่อ เขาต้องการจะสืบหาไหมว่า นี่เป็นความผิดพลาดแบบไหน ถ้าใช่ คุณควรสร้าง type ที่มีเมธอด `Error()` ขึ้นมาใช้เองจะดีกว่า
- คุณต้องการจะบอกคนอื่นไหมว่า นี่เป็น error ที่เกิดขึ้นตรงไหน ถ้าใช่ ลองใช้ตัวนี้ดู [section on error wrapping](#error-wrapping)
- ในกรณีอื่นๆ [`fmt.Errorf`] ก็เป็นตัวเลือกที่ดี

  [`errors.New`]: https://golang.org/pkg/errors/#New
  [`fmt.Errorf`]: https://golang.org/pkg/fmt/#Errorf
  [`"pkg/errors".Wrap`]: https://godoc.org/github.com/pkg/errors#Wrap

ถ้าผู้เรียก ต้องการสืบว่านี่เป็น error อะไร และคุณอยากจะสร้างมันด้วย [`errors.New`] ก็ขอให้ ทำให้มันเป็นตัวแปรดีกว่า

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

ถ้าคุณมี error ที่ผู้เรียกต้องการสืบหาว่าเป็นแบบไหน แต่คุณก็อยากจะเพิ่มข้อมูลลงไปในนั้น (ไม่ใช่ค่าคงที่) ถ้างั้นคุณก็น่าจะสร้าง type มาใช้เอง

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

ขอให้ระมัดระวังการเปิดเผย error type ที่คุณสร้างมันขึ้นมาออกสู่ภายนอกโดยตรง เราแนะทำให้คุณเปิดฟังก์ชั่นที่ใช้เช็ค type ของ error นี้ออกไปแทนจะดีกว่า

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

<!-- TODO: Exposing the information to callers with accessor functions. -->

### Error Wrapping

มีสามวิธีที่จะบอกให้ผู้ที่เรียกใช้รู้ว่าการทำงานผิดพลาด:

- คืน error เดิมๆออกไปเลย ถ้าคุณไม่ต้องการเพิ่มคำอธิบายใดๆ และอยากให้เห็น error ดิบๆแบบนั้น
- เพิ่มคำอธิบายลงไปด้วยการใช้ [`"pkg/errors".Wrap`] และใช้ [`"pkg/errors".Cause`] เวลาที่ต้องการถอดเอาเฉพาะ error เดิมออกมา
- ใช้ [`fmt.Errorf`] ถ้าผู้เรียกไม่อยากรู้ว่าเป็น error แบบไหนให้ชัดเจน

เราแนะนำให้เพิ่มคำอธิบายลงไปถ้าทำได้ แทนที่จะให้เห็น error แบบคลุมเครือเช่น "connection refused" แล้วเพิ่มคำอธิบายให้มีประโยชน์มากกว่าลงไป เช่น "call service foo: connection refused"

เวลาที่คุณจะเพิ่มคำอธิบายใน error ให้ใช้ประโยคที่กระชับ แล้วไม่ต้องใส่คำเวิ่นเว้อเช่น "failed to" ไม่งั้นเวลามันผ่านหลายๆชั้นแล้วมันจะดูเป็นคำขยะ:

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

</td><td>

```go
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

<tr><td>

```
failed to x: failed to y: failed to create new store: the error
```

</td><td>

```
x: y: new store: the error
```

</td></tr>
</tbody></table>

แต่ไม่ว่ายัง เวลาที่ error ถูกส่งไปที่ระบบอื่น มันควรมีความชัดเจนในข้อความ (ตัวอย่างเช่น ติดป้ายว่า `err` หรือใช้คำนำหน้า "Failed" ตอนที่ลง logs)

See also [Don't just check errors, handle them gracefully].

  [`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
  [Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### Handle Type Assertion Failures

การรับค่าเดียวตอนที่ทำ [type assertion] มันอาจจะ panic ถ้า type มันไม่ถูก ดังนั้นให้ใช้สำนวนแบบ "comma ok" เสมอ

  [type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

<!-- TODO: There are a few situations where the single assignment form is
fine. -->

### Don't Panic

โค้ดที่จะขึ้น Production อย่าใช้ panics เพราะ Panic เป็นตัวหลักของการเกิด [cascading failures] ถ้ามันเกิด error ขึ้น ก็ให้ฟังก์ชั่นคืน error ออกไป ให้คนที่เรียกเขาไปตัดสินใจจัดการเอาเองเถิด

  [cascading failures]: https://en.wikipedia.org/wiki/Cascading_failure

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

Panic/recover ไม่ใช่วิธีการจัดการ error เพราะโปรแกรมจะ panic เฉพาะเมื่อเกิดเหตุที่คาดไม่ถึงเช่น ไปอ้างถึงอะไรก็แล้วแต่ กับค่า nil เว้นแค่จะเป็นช่วงเตรียมของก่อนเริ่มโปรแกรม ถ้าเกิดเหตุที่ไม่คาดคิดก็ควรจะหยุดการทำงานของโปรแกรมไปเลย

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

แม้กระทั่งใน tests ก็แนะนำให้่ใช้ `t.Fatal` หรือ `t.FailNow` มากกว่าการทำให้มัน panic เพื่อบอกให้เทสรู้ว่าเกิดข้อผิดพลาด

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

</td><td>

```go
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

</td></tr>
</tbody></table>

<!-- TODO: Explain how to use _test packages. -->

### Use go.uber.org/atomic

ตัวทำ automic ในแพ็คเก็จ [sync/automic] ใช้ได้กับ type ดิบๆ (`int32`, `int64`, etc.) เราเลยลืมที่จะใช้มันเวลาจะอ่านหรือแก้ไขค่าตัวแปร

[go.uber.org/atomic] ได้เพิ่ม type ที่ปลอดภัยเข้าไปอีก โดยซ่อน type จริงๆไว้ข้างล่าง นอกจากนี้ยังเพิ่ม type `atomic.Bool` เพื่อให้สะดวกขึ้นอีก

  [go.uber.org/atomic]: https://godoc.org/go.uber.org/atomic
  [sync/atomic]: https://golang.org/pkg/sync/atomic/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

</td><td>

```go
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

</td></tr>
</tbody></table>

## Performance

คำแนะนำโดยตรงเกี่ยวกับประสิทธิภาพ คือทำเฉพาะส่วนที่เป็น hot path (ส่วนที่ถูกเรียกใช้งานหนักๆ)

### Prefer strconv over fmt

เมื่อต้องการแปลงชนิดไปมา กับสตริง ให้ใช้ `strconv` จะเร็วกว่าใช้ `fmt`

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Avoid string-to-byte conversion

อย่าสร้าง slices ของ byte จากสตริงในลูป ให้ทำครั้งเดียวพอ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Prefer Specifying Map Capacity Hints

ถ้าทำได้ ให้บอกใบ้ขนาดให้กับ map ตอนที่เรียก `make()`

```go
make(map[T1]T2, hint)
```

การใบ้ค่าความจุกับ `make()` อย่างน้อยพยายามให้มันใกล้เคียงที่สุด ตอนที่สร้าง map จะช่วยลดเวลาตอนที่ต้องเพิ่มขนาดมันทีหลัง ซึ่งอันที่จริงการใส่ความจุแบบนี้ก็ไม่รับประกันว่ามันจะไม่เสียเวลา เพราะบางทีการเพิ่มของเข้าไปก็อาจจะเกิดกระบวนการจองหน่วยความจำได้ ทั้งๆที่ก็ได้ให้ความจุไปก่อนแล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` ถูกสร้างโดยไม่ใบ้ขนาดให้มัน ซึ่งมันอาจจะทำให้ต้องเสียเวลาจังหวะที่จะจองหน่วยความจำ

</td><td>

`m` ถูกสร้างโดยใบ้ขนาดให้ด้วย ซึ่งจะเสียเวลาจองหน่วยความจำนิดเดียว

</td></tr>
</tbody></table>

## Style

### Be Consistent

คำแนะนำบางส่วนที่ระบุในเอกสารชุดนี้ วัดผลได้จริง เว้นไว้แต่เพียง พฤติกรรม บริบท หรือหัวข้อต่างๆ

นอกเหนือจากที่กล่าวมาก็คือ **ทำให้เป็นจังหวะเดียวกัน**

โค้ดที่ลายมือเดียวกัน มันดูแลรักษาง่าย มันง่ายที่จะเข้าใจ ไม่ทำให้เสียเวลาต้องมานั่งแกะ แล้วถ้าแก้ไขย้ายที่มันก็ยังทำได้ง่ายกว่า รวมถึงตอนแก้บั๊กด้วย

ตรงกันข้าม ถ้าเขียนมาคนละแบบ หรือสไตล์ไม่เข้ากันทั้งๆที่โค้ดชุดเดียวกัน มันจะทำให้เสียเวลาในการดูแล เปราะบาง และไม่เข้ากัน ทั้งหมดทั้งมวลนี้จะทำให้ทำงานได้ช้า รีวิวโค้ด จะเหนื่อยมาก และเต็มไปด้วยบั๊ก

เวลาจะนำเอาคำแนะนำชุดนี้ไปปรับใช้จริง แนะนำว่าให้ทำกันในระดับแพ็คเก็จ (หรือใหญ่กว่า): ถ้าทำแค่ในแพ็คเก็จย่อยๆ มันจะขัดกับสิ่งที่กล่าวมาข้างต้น เพราะมันจะมีหลายสไตล์ในโค้ดชุดเดียว

### Group Similar Declarations

Go สนับสนุนการจัดกลุ่มการการประกาศที่เป็นพวกเดียวกัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import "a"
import "b"
```

</td><td>

```go
import (
  "a"
  "b"
)
```

</td></tr>
</tbody></table>

การทำแบบนี้ยังสามารถทำได้กับการประกาศ constant ตัวแปร และการประกาศ type

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

</td></tr>
</tbody></table>

จัดกลุ่มเฉพาะสิ่งที่สัมพันธ์กัน อย่าไปทำกับอะไรที่ไม่เกี่ยวข้องกัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

การจัดกลุ่มสามารถทำในฟังก์ชั่นก็ได้ ดังแสดงในตัวอย่าง

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f() string {
  var red = color.New(0xff0000)
  var green = color.New(0x00ff00)
  var blue = color.New(0x0000ff)

  ...
}
```

</td><td>

```go
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )

  ...
}
```

</td></tr>
</tbody></table>

### Import Group Ordering

แบ่งกลุ่มการอิมพอร์ตเป็นสองชุด:

- Standard library
- Everything else

การจัดกลุ่มนี้ goimports ทำให้โดยปกติอยู่แล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Package Names

เวลาจะประกาศชื่อแพ็คเก็จ ให้เลือกแบบนี้:

- ใช้ตัวอักษรเล็กทั้งหมด ไม่มีตัวใหญ่ หรือขีดล่าง
- ไม่เปลี่ยนชื่อมันตอนที่ผู้ใช้ import มันเข้าไป
- สั้นและกระชับ เพราะมันจะถูกอ้างถึงในทุกที่จะมาเรียกใช้
- ไม่ต้องทำเป็นพหูพจน์
- อย่าตั้งชื่อ "common", "util", "shared" ชื่อพวกนี้มันห่วย และไม่ได้ช่วยให้เรารู้อะไรเลย

ดูเพิ่มเติมได้ที่ [Package Names] และ [Style guideline for Go packages]

  [Package Names]: https://blog.golang.org/package-names
  [Style guideline for Go packages]: https://rakyll.org/style-packages/

### การตั้งชื่อฟังก์ชั่น

เราทำแบบเดียวกับที่ชุมชนคนเขียน go ทำกันด้วยการใช้ [MixedCaps for function
names] (การผสมตัวอักษรเล็กและใหญ่) ยกเว้นเฉพาะเวลาเขียนเทส สามารถใช้ขีดล่างได้ เพื่อจัดกลุ่มการทดสอบที่สัมพันธ์กัน ตัวอย่างเช่น
`TestMyFunction_WhatIsBeingTested`.

  [MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Import Aliasing

การตั้งชื่อแฝงให้แพ็คเก็จที่ import ทำเมื่อชื่อแพ็คเก็จที่นำเข้ามาไม่ตรงกับส่วนสุดท้ายของพาร์ท

```go
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

ในกรณีอื่นๆ การตั้งชื่อแฝงให้การ import ไม่ควรทำ เว้นเสียแต่ว่ามันจะไปซ้ำกันกับแพ็คเก็จอื่น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"


  nettrace "golang.net/x/trace"
)
```

</td><td>

```go
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

</td></tr>
</tbody></table>

### Function Grouping and Ordering

- ควรเรียงฟังก์ชั่นตามลำดับการเรียกใช้
- ฟังก์ชั่นในไฟล์ควรจัดกลุ่มตาม receiver

ฟังก์ชั่นที่เปิดเผยไปข้างนอกควรอยู่ในส่วนแรกๆของไฟล์ หลังการประกาศ `struct`, `const`, `var`

ฟังก์ชั่นแบบนี้ `newXYZ()`/`NewXYZ()` ควรอยู่หลังการประกาศ type แต่อยู่ก่อนเมธอดที่ใช้ type นี้เป็นตัว receiver

พอฟังก์ชั่นถูกจัดกลุ่มแบบนี้ พวกฟังก์ชั่นที่ใช้งานทั่วไปก็ควรไปอยู่ส่วนท้ายๆของไฟล์

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduce Nesting

โค้ดควรลดความยุ่งเหยิงด้วยการจัดการ error ก่อนแล้วรีเทิร์นออกไป หรือไปเริ่มต้นลูปใหม่ ให้เร็วที่สุด เพื่อลดโค้ดที่ซ้อนกันหลายๆชั้น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Unnecessary Else

ถ้าตัวแปรจะถูกกำหนดค่าทั้งใน if และ else มันควรจะเหลือแค่ if ก็ได้

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Top-level Variable Declarations

การใช้คียเวิร์ด `var` ไม่ต้องบอก type ก็ได้ เว้นเสียแต่ว่ามันจะคืน type ไม่ตรงกับที่ต้องการ

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

ระบุ type ถ้า type ที่ได้รับมาไม่ตรงกับที่อยากได้

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError but we want error.
```

### Prefix Unexported Globals with _

ตั้งชื่อขึ้นต้นด้วย ขีดล่าง เวลาประกาศด้วย `var`s และ `const`s ให้ตัวแปรที่ไม่เปิดเผยสู่ภายนอก เพื่อทำให้ชัดเจว่ามันถูกใช้เป็น global อยู่ภายในแพ็คเก็จ

ข้อยกเว้น: ตัวแปร error ที่ไม่เปิดเผยสู่ภายนอก ควรตั้งขื่อขึ้นต้นด้วย `err`

หลักการและเหตุผล: ตัวแปรที่ประกาศไว้ตั้งแต่ต้น และพวก constants มีขอบเขตในแพ็คเก็จ เพราะฉะนั้น การตั้งชื่อแบบกลางๆ มันจะทำให้เกิดเรื่องไม่คาดคิดได้ ทำให้ได้ค่าผิดในไฟล์อื่นได้ง่ายมาก

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

</td><td>

```go
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

</td></tr>
</tbody></table>

### Embedding in Structs

type ที่ถูกฝังไว้ (เช่น mutexes) ควรอยู่บนสุดของรายการใน struct และควรเว้นบรรทัดว่างๆไว้สักบรรทัด

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>

### Use Field Names to Initialize Structs

คุณควรระบุชื่อฟิลด์เสมอเมื่อประกาศตัวแปรจาก struct ซึ่งตอนนี้เวลานี้ถูกบังคับโดย [`go vet`] เรียบร้อยแล้ว

  [`go vet`]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

ข้อยกเว้น: ชื่อฟิลด์ *อาจจะ* ละไว้ได้ในตารางการทดสอบถ้ามันมี 3 ฟิลด์หรือน้อยกว่า

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### Local Variable Declarations

การประกาศตัวแปรแบบสั้น (`:=`) ควรถูกใช้เมื่อต้องการกำหนดค่าให้ตัวแปรอยู่แล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

อย่างไรก็ดี บางกรณีการปล่อยให้มันเป็นค่าเริ่มต้นก็อาจจะชัดเจนกว่า ด้วยการใช้คีย์เวิร์ด `var` [Declaring Empty Slices] ตัวอย่างเช่น

  [Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### nil is a valid slice

`nil` เป็นค่าที่เหมาะสมที่จะใช้แทน slice ขนาด 0 หมายความว่า

- คุณไม่ควรคืน slice ที่มีขนาดเท่ากับศูนย์ออกไปตรงๆ แต่ให้คืน `nil` ออกไปแทน

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  if x == "" {
    return []int{}
  }
  ```

  </td><td>

  ```go
  if x == "" {
    return nil
  }
  ```

  </td></tr>
  </tbody></table>

- การตรวจสอบว่า slice นั้นว่างเปล่าหรือไม่ ให้ใช้ `len(s) == 0` อย่าไปตรวจสอบ `nil`

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  func isEmpty(s []string) bool {
    return s == nil
  }
  ```

  </td><td>

  ```go
  func isEmpty(s []string) bool {
    return len(s) == 0
  }
  ```

  </td></tr>
  </tbody></table>

- zero value (slice ที่ประกาศด้วย `var`) สามารถใช้งานได้เลย โดยไม่ต้อง `make()` ก่อน

  <table>
  <thead><tr><th>Bad</th><th>Good</th></tr></thead>
  <tbody>
  <tr><td>

  ```go
  nums := []int{}
  // or, nums := make([]int)

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td><td>

  ```go
  var nums []int

  if add1 {
    nums = append(nums, 1)
  }

  if add2 {
    nums = append(nums, 2)
  }
  ```

  </td></tr>
  </tbody></table>

### Reduce Scope of Variables

ถ้ามีโอกาสลดขอบเขตของตัวแปรก็ควรทำ แต่อย่าไปลดมันถ้ามันขัดแย้งกับ [Reduce Nesting](#reduce-nesting)

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

ถ้าคุณต้องการผลลัพธ์ของฟังก์ชั่นไปใช้หลัง if ต่อ งั้นคุณก็ไม่ควรลดขอบเขตมัน

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Avoid Naked Parameters

พารามิเตอร์เปลือยๆที่ใส่ไปตอนที่เรียกฟังก์ชั่น มันอ่านยาก ให้เพิ่มคอมเม้นท์แบบ C-style ลงไป (`/* ... */`) ให้ความหมายชัดเจนขึ้น

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

แต่มันก็ยังไม่ดีที่สุด เราควรแทนที่ type `bool` ที่เปลือยๆอยู่นี้ด้วยการสร้าง type ขึ้นมาให้มันอ่านง่ายขึ้น และยังรองรับหากในอนาคตต้องการมีมากกว่าสองสถานะ (true/false)

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Use Raw String Literals to Avoid Escaping

Go สนับสนุน [raw string literals](https://golang.org/ref/spec#raw_string_lit) ซึ่งสามารถใส่ได้หลายบรรทัดรวมทั้งเครื่องหมายคำพูดได้ด้วย ซึ่งการใช้แบบนี้เพื่อป้องกันการทำ hand-escaped เพราะมันจะทำให้อ่านยาก

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Initializing Struct References

ใช้ `&T{}` แทนการใช้ `new(T)` เมื่อต้องการสร้างตัวแปรแบบอ้างอิงจาก struct จะดูดีกว่า

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Initializing Maps

เสนอให้ใช้ `make(..)` เพื่อสร้าง maps ว่างๆ และเอาไปเขียนโปรแกรมต่อได้ ซึ่งมันทำการประกาศตัวแปรให้พร้อมใช้งานดูมีความต่างจากการประกาศเฉยๆ และมันยังทำให้ง่ายต่อการเพิ่มการใบ้ขนาดให้ในภายหลังด้วย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

การประกาศให้พร้อมใช้งาน กับการประกาศแล้วยังไม่พร้อมใช้งาน ดูคล้ายๆกัน

</td><td>

การประกาศให้พร้อมใช้งาน กับการประกาศแล้วยังไม่พร้อมใช้งาน ดูแตกต่างกัน

</td></tr>
</tbody></table>

ถ้าทำได้ ก็ให้ใบ้ความจุตอนที่ประกาศ maps ด้วยคำสั่ง `make()` ลองดูที่ [Prefer Specifying Map Capacity Hints](#prefer-specifying-map-capacity-hints) สำหรับข้อมูลเพิ่มเติม

หรือในทางกลับกัน ถ้า map นั้นจะต้องเก็บค่าที่แน่นอน ก็ให้ใช้การประกาศด้วยปีกกาได้เลย

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

กฎพื้นฐานของนิ้วหัวแม่มือก็คือ ใช้ปีกกาประกาศเมื่อต้องใส่ค่าคงที่ลงไปตั้งแต่ต้น ไม่เช่นนั้นก็ใช้ `make` (และใส่การใบ้ความจุถ้าทำได้)

### Format Strings outside Printf

ถ้าคุณประกาศการจัดรูปแบบสตริงสำหรับใช้กับฟังก์ชั่น `Printf`-style ให้ทำเป็น `const`

มันจะช่วยให้ `go vet` ได้วิเคราะห์การจัดรูปแบบให้

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td><td>

```go
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

</td></tr>
</tbody></table>

### Naming Printf-style Functions

เมื่อคุณประกาศฟังก์ชั่น `Printf`-style ช่วยทำให้มั่นใจว่า `go vet` จะสามารถตรวจเจอมันและจะได้ตรวจสอบรูปแบบสตริงได้

หมายความว่า คุณควรใช้ชื่อที่ตั้งเผื่อไว้แล้วตามสไตล์ `Printf` ถ้าทำได้ `go vet` จะได้ตรวจสอบได้เอง ดูรายละเอียดเพิ่มเติมได้ที่ [Printf family]

  [Printf family]: https://golang.org/cmd/vet/#hdr-Printf_family

ถ้าการใช้ชื่อที่ตั้งเผื่อไว้ ไม่ใช่ทางเลือกของคุณ งั้นก็ให้ตั้งชื่อลงท้ายด้วย f: เช่น `Wrapf` ไม่ใช่ `Wrap` เฉยๆ โดยสามารถบอกให้ `go vet` ตรวจสอบฟังก์ชั่นสไตล์ `Printf` ได้ แต่มันจะต้องลงท้ายด้วยตัว f เท่านั้น

```shell
$ go vet -printfuncs=wrapf,statusf
```

See also [go vet: Printf family check].

  [go vet: Printf family check]: https://kuzminva.wordpress.com/2017/11/07/go-vet-printf-family-check/

## Patterns

### Test Tables

ใช้การทดสอบที่ขับเคลื่อนด้วยตาราง ด้วย [subtests] เพื่อหลีกเลี่ยงการเขียนโค้ดซ้ำๆ เวลาที่เราเทสด้วยลอจิกแบบเดิมหลายๆครั้ง

  [subtests]: https://blog.golang.org/subtests

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

</td><td>

```go
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

</td></tr>
</tbody></table>

ตารางการทดสอบมันจะง่ายเวลาที่จะเพิ่มบริบทให้ข้อความ error แล้วยังลดโค้ดซ้ำๆ และเพิ่ม test cases ง่าย

เราปฏิบัติตามประเพณีนิยมด้วยการใช้ slice ของ struct แล้วตั้งชื่อว่า `tests` และแต่ละ test case ให้ชื่อ `tt` และระบุชื่อให้ input และ output ในแต่ละ test case ด้วยการตั้งชื่อขึ้นต้นว่า `give` และ `want`

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

### Functional Options

Functional options เป็นรูปแบบที่ใช้ประกาศ type `Option` เพื่อบันทึกข้อมูลลงไปใน struct ภายใน จากนั้นให้รับตัวแปรแบบ varidic เข้ามาเป็นตัวเลือก และจัดการตามข้อมูลที่บันทึกไว้ใน options ที่เป็น struct ภายใน

ใช้รูปแบบนี้สำหรับอาร์กิวเม้นต์ที่เป็นตัวเลือกและ APIs สาธารณะอื่นๆที่คุณคาดเดาได้ว่าจะต้องถูกขยาย โดยเฉพาะอย่างยิ่งถ้าคุณมีอาร์กิวเม้นต์ 3ตัว หรือมากกว่าอยู่แล้ว

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

</td><td>

```go
type options struct {
  timeout time.Duration
  caching bool
}

// Option overrides behavior of Connect.
type Option interface {
  apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
  f(o)
}

func WithTimeout(t time.Duration) Option {
  return optionFunc(func(o *options) {
    o.timeout = t
  })
}

func WithCaching(cache bool) Option {
  return optionFunc(func(o *options) {
    o.caching = cache
  })
}

// Connect creates a connection.
func Connect(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    timeout: defaultTimeout,
    caching: defaultCaching,
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}

// Options must be provided only if needed.

db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
  addr,
  db.WithCaching(false),
  db.WithTimeout(newTimeout),
)
```

</td></tr>
</tbody></table>

See also,

- [Self-referential functions and the design of options]
- [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis

<!-- TODO: replace this with parameter structs and functional options, when to
use one vs other -->
