```go
package main

import (
	"context"
	"errors"
	"fmt"
	"net/http"
	"os"
	"os/signal"

	"golang.org/x/sync/errgroup"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	_, _ = w.Write([]byte("hello world"))
}

var interrupt chan os.Signal

//
func registerSignal(ctx context.Context, s1 *http.Server) error {
	interrupt = make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)

	select {
	case msg := <-interrupt:
		signal.Stop(interrupt)
		//通知http server 结束
		if err := s1.Shutdown(ctx); err != nil {
			return err
		}
		return errors.New(msg.String())
	case <-ctx.Done():
		return context.Canceled
	}
}

func main() {
	group, ctx := errgroup.WithContxxt(context.Background())
	s1 := &http.Server{Addr: "127.0.0.1:8080", Handler: nil}
	group.Go(func() error {
		return s1.ListenAndServe()
	})

	group.Go(func() error {
		return registerSignal(ctx, s1)
	})

	if err := group.Wait(); err != nil {
		fmt.Println(err)
	}
}

```