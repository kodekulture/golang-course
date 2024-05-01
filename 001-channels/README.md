Here are all the codes used in the [video]()! 

Thanks for watching :-)

# Explanation 🖊

### Channel Read+Write

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	go func() { ch <- 1 }()

	fmt.Printf("Got %d\n", <-ch)
}
```

### Channel Read+Write Buffered

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 1)

	ch <- 1

	fmt.Printf("Got %d\n", <-ch)
}
```

### Channel + range + close

```go
package main

import "fmt"

func main() {
  message := "I'll be splitted to runes"
  ch := make(chan rune)

  go func(){
    for _, r := range message {
      ch <- r
    }
    close(ch)
  }()

  var runes []rune

  for r := range ch {
    runes = append(runes, r)
  }

  fmt.Println(string(runes))
}
```

### Channel + ok

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	go func() {
		ch <- 1
		close(ch)
	}()

	var (
		value int
		ok    bool
	)

	value, ok = <-ch
	fmt.Printf("Got %d : %t\n", value, ok)

	value, ok = <-ch
	fmt.Printf("Got %d : %t\n", value, ok)
}
```

### Channel + select

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Hello"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "World"
    }()

    select {
    case msg1 := <-ch1:
        fmt.Println("Received:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received:", msg2)
    }
}
```

### Channel nil + read

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var c chan int

	go func() {
		<-c // block
		fmt.Println("Hello, World")
	}()

	time.Sleep(1 * time.Second)

	fmt.Println("Done")
}
```

### Channel nil + write

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var c chan int

	go func() {
		fmt.Println("Channel is listening")
		i := <-c // blocks
		fmt.Printf("Channel got %d\n", i)
	}()

	time.Sleep(1 * time.Second)

	select {
	case c <- 1: // blocks
		fmt.Println("Wrote value into channel")
	default:
		fmt.Println("Skip read from channel")
	}

	time.Sleep(1 * time.Second)

	fmt.Println("Done")
}
```

### Channel nil + close

```go
package main

import "fmt"

func main() {
  var c chan int

  close(c) // panics

  fmt.Println("Done")
}
```

### Channel worker-pool

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d is processing job %d\n", id, job)
		time.Sleep(1 * time.Second)
		fmt.Printf("Worder %d finished job %d\n", id, job)
	}
}

func main() {
	var (
		numWorkers = 3
		numJobs    = 10

		jobs = make(chan int, numJobs)

		wg sync.WaitGroup
	)

	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go worker(i, jobs, &wg)
	}

	for i := 1; i <= numJobs; i++ {
		jobs <- i
	}

	close(jobs)

	wg.Wait()
}
```

# Interview 📑

### Double read

#### Problem

```go
package main

func main() {
    timeStart := time.Now()
    _, _ = <-worker(), <-worker()
    println(int(time.Since(timeStart).Seconds())) // What will this output?
}

func worker() chan int {
    ch := make(chan int)
    go func() {
        time.Sleep(3 * time.Second)
        ch <- 1
    }()
    return ch
}
```

#### Solution

```
6
```

#### Explanation

In simple word, It's because the following 2 blocks of code are similar to the compiler

```go
package main

func main() {
  _, _ = <-worker(), <-worker()
  // OR
  _, = <-worker()
  _, = <-worker()
}
```

### Unclosed channel

#### Problem

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var (
		ch    = make(chan *int, 4)
		wg    = sync.WaitGroup{}

    arr = []int{1, 2, 3, 4}
	)

	wg.Add(1)

	go func() {
		for _, value := range arr {
			ch <- &value
		}
	}()

	go func() {
		for value := range ch {
			fmt.Println(*value) // What will be printed here?
		}
		wg.Done()
	}()

	wg.Wait()
}
```

#### Solution

Change the for loop in the first goroutine to

```go
for _, value := range arr {
  v := value
  ch <- &v
}
close(ch)
```

#### Explanation

- The range statement uses the same var for all iterations (i.e. the pointer is the same)
- Closing the channel is necessary for our application to execute completely, otherwise it will hang forever 

### Merge channels 

#### Problem

```go
package main

import "fmt"

func main() {
  var (
    c1 = make(chan int)
    c2 = make(chan int)
    c3 = make(chan int)
  )

  // write value & close channel 
  go func(){ c1 <- 1; close(c1) }()
  go func(){ c2 <- 2; close(c2) }()
  go func(){ c3 <- 3; close(c3) }()

  // merge channels into signle channel
  out := mergeChannels(c1, c2, c3)

  // read channels output
  for value := range out {
    fmt.Printf("Got %d\n", value)
  }

  fmt.Println("Done")
}

func mergeChannels(chans ...<-chan int) <-chan int {
  panic("implement me")
}
```

#### Solution

```go
package main

import (
  "fmt"
  "sync"
)

func main() {
  var (
    c1 = make(chan int)
    c2 = make(chan int)
    c3 = make(chan int)
  )

  // write value & close channel 
  go func(){ c1 <- 1; close(c1) }()
  go func(){ c2 <- 2; close(c2) }()
  go func(){ c3 <- 3; close(c3) }()

  // merge channels into single channel
  out := mergeChannels(c1, c2, c3)

  // read channels output
  for value := range out {
    fmt.Printf("Got %d\n", value) // NOTICE: Order is not guaranteed
  }

  fmt.Println("Done")
}

func mergeChannels(chans ...<-chan int) <-chan int {
  // Create a new channel that will
  // collect message from passed channels
	out := make(chan int)

	var wg sync.WaitGroup
	for _, c := range chans {
		wg.Add(1)
    // For each channel we create a goroutine
    // that reads it's values and pass it to the
    // `out` channel.
		go func(c <-chan int) {
			defer wg.Done()
			for v := range c {
				out <- v
			}
		}(c)
	}

	go func() {
    // Upon closing the passed channels, we need
    // to close our out channel since it can't receive
    // any messages anymore. Otherwise, consumers of the
    // `out` channel will hang forever
		wg.Wait()
		close(out)
	}()

	return out
}
```

#### Explanation

The explanation is added to the code block above

# Channel cycle

#### Problem

```go
package main

import "fmt"

func main() {
	const size = 10000
	list := []chan int{make(chan int)}

	for i := 0; i < size; i++ {
		list = append(list, make(chan int))
		go func(i int) {
			go func() {
				for v := range list[i] {
					list[i-1] <- v + 1
				}
				go func() {
					close(list[i-1])
				}()
			}()
		}(i + 1)
	}

	go func() {
		list[len(list)-1] <- 1
		close(list[len(list)-1])
	}()

	fmt.Println(<-list[0])
	fmt.Println(<-list[0])
}
```

#### Solution

```go
package main

import "fmt"

func main() {
	const size = 10000
	list := []chan int{make(chan int)}

	for i := 0; i < size; i++ {
		list = append(list, make(chan int))
		go func(i int) {
			go func() {
				for v := range list[i] {
					list[i-1] <- v + 1
				}
				go func() {
					close(list[i-1])
				}()
			}()
		}(i + 1)
	}

	go func() {
		list[len(list)-1] <- 1
		close(list[len(list)-1])
	}()

	fmt.Println(<-list[0])
	fmt.Println(<-list[0])
}
```

#### Explanation

The explanation is added to the code block above