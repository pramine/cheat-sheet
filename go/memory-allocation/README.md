# Go Memory allocations

In order to inspect how much memory is being allocated and the source of those allocations we’ll use [pprof](https://golang.org/pkg/runtime/pprof/). 
pprof is an absolutely amazing tool and one of the main reasons that I personally use go. In order to use it we’ll have to first enable it, and then take some snapshots. If you’re already using http, enabling it is literally as easy as:

`import _ "net/http/pprof"`

Once pprof is enabled we’ll periodically take heap snapshots throughout the life of process memory growth. Taking a heap snapshot is just as trivial:

```
curl http://localhost:8080/debug/pprof/heap > heap.0.pprof
sleep 30
curl http://localhost:8080/debug/pprof/heap > heap.1.pprof
sleep 30
curl http://localhost:8080/debug/pprof/heap > heap.2.pprof
sleep 30
curl http://localhost:8080/debug/pprof/heap > heap.3.pprof
```

The goal is to get an idea of how memory is growing throughout the life of the program. Let’s inspect the most recent heap snapshot:

`$ go tool pprof pprof/heap.3.pprof`

Local symbolization failed for main: open /tmp/go-build598947513/b001/exe/main: no such file or directory
Some binary filenames not available. Symbolization may be incomplete.
Try setting PPROF_BINARY_PATH to the search path for local binaries.
File: main
Type: inuse_space
Time: Jul 30, 2018 at 6:11pm (UTC)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) svg
Generating report in profile002.svg
(pprof) top20
Showing nodes accounting for 410.75MB, 99.03% of 414.77MB total
Dropped 10 nodes (cum <= 2.07MB)
      flat  flat%   sum%        cum   cum%
  408.97MB 98.60% 98.60%   408.97MB 98.60%  bytes.Repeat
    1.28MB  0.31% 98.91%   410.25MB 98.91%  main.(*RequestTracker).Track
    0.50MB  0.12% 99.03%   414.26MB 99.88%  net/http.(*conn).serve
         0     0% 99.03%   410.25MB 98.91%  main.main.func1
         0     0% 99.03%   410.25MB 98.91%  net/http.(*ServeMux).ServeHTTP
         0     0% 99.03%   410.25MB 98.91%  net/http.HandlerFunc.ServeHTTP
         0     0% 99.03%   410.25MB 98.91%  net/http.serverHandler.ServeHTTP

This is absolutely amazing. pprof defaults to Type: inuse_space which displays all the objects that are currently in memory at the time of the snapshot. We can see here that bytes.Repeat is directly responsible for 98.60% of all of our memory!!!

The line below the bytes.Repeat entry shows:

1.28MB  0.31% 98.91%   410.25MB 98.91%  main.(*RequestTracker).Track
This is really interesting, it shows that Track itself has1.28MB or 0.31% but is responsible for 98.91% of all in use memory!!!!!!!!!!!!! Further more we can see that http has even less memory in use but is responsible for even more than Track (since Track is called from it).

pprof exposes many ways to introspect and visualize memory (in use memory size, in use number of objects, allocated memory size, allocated memory objects), it allows listing the track method and showing how much each line is responsible for:


(pprof) list Track
Total: 414.77MB
ROUTINE ======================== main.(*RequestTracker).Track in /vagrant_data/go/src/github.com/dm03514/grokking-go/simple-memory-leak/main.go
    1.28MB   410.25MB (flat, cum) 98.91% of Total
         .          .     19:
         .          .     20:func (rt *RequestTracker) Track(req *http.Request) {
         .          .     21:   rt.mu.Lock()
         .          .     22:   defer rt.mu.Unlock()
         .          .     23:   // alloc 10KB for each track
    1.28MB   410.25MB     24:   rt.requests = append(rt.requests, bytes.Repeat([]byte("a"), 10000))
         .          .     25:}
         .          .     26:
         .          .     27:var (
         .          .     28:   requests RequestTracker
         .          .     29:

This directly pinpoints the culprit:

1.28MB   410.25MB     24:   rt.requests = append(rt.requests, bytes.Repeat([]byte("a"), 10000))
pprof also allows visual generation of the textual information above:

(pprof) svg
Generating report in profile003.svg

This clearly shows the current objects occupying the process memory. Now that we have the culprit Track we can verify that it is allocating memory to a global without ever cleaning it up, and fix the root issue.

Resolution: Memory was being continually allocated to a global variable on each HTTP request.
