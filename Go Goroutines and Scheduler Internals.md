---
tags: Go TODO
---

# Scheduler

Go uses an `M`:`N` scheduler - distributes `M` goroutines onto `N` threads

## Work Stealing

## Per Context Run-queue

# References

- [proc.go](https://github.com/golang/go/blob/master/src/runtime/proc.go)
- [Concurrency in Go. Katherine Cox-Buday](References.md#Concurrency%20in%20Go.%20Katherine%20Cox-Buday)
- [GopherCon 2018: Kavya Joshi - The Scheduler Saga - YouTube](https://youtu.be/YHRO5WQGh0k?si=ZHiNxNxG1GAOff9D)
- [Scalable Go Scheduler Design Doc - Google Docs](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)
- [Dmitry Vyukov — Go scheduler: Implementing language with lightweight concurrency - YouTube](https://youtu.be/-K11rY57K7k?si=goeOdKxqd-cXBZE1)
- [Continuation-passing style - Wikipedia](https://en.wikipedia.org/wiki/Continuation-passing_style)
- [runtime: tight loops should be preemptible · Issue #10958 · golang/go · GitHub](https://github.com/golang/go/issues/10958)
- [dotGo 2017 - JBD - Go's work stealing scheduler - YouTube](https://youtu.be/Yx6FBsGNOp4?si=3Onn5gRHTaPvlbtb)
- [Go's work-stealing scheduler · rakyll.org](https://rakyll.org/scheduler/)
- [All Go: Concurrency](https://www.andy-pearce.com/blog/posts/2023/Jun/all-go-concurrency/)
