# ember
[![Go Report Card](https://goreportcard.com/badge/github.com/maja42/ember)](https://goreportcard.com/report/github.com/maja42/ember) [![GoDoc](https://godoc.org/github.com/maja42/ember?status.svg)](https://godoc.org/github.com/maja42/ember)

Ember is a lightweight library and tool for embedding arbitrary resources into a go executable at runtime.
The resources don't need to exist at compile time.

Embedding binary files (eg. zip-archives and executables) is supported.

## Use case

Applications often require runtime- or user-defined configuration and resources to be stored alongside
the executable. \
This forces the end-user to deal with multiple files when copying, moving or distributing the application. 
It also allows end users to manipulate those attachments, which is not always desirable.

The main use-case of ember is to bundle such configuration files and other resources with the application at runtime.
There is no need for setting up a go toolchain to (re-)build the application every time there is a new configuration.

## Cross platform

Ember is truly cross-platform. It supports any OS, and embedding resources can also be done cross-platform. \
This means that files can be attached to windows executables on both windows and linux and vice-versa.

## Usage

Ember consists of two parts. 
1. The `ember` package is imported by the application that receives attachments.
2. The `ember/embedding` package is used by a separate application that attaches files to the already-compiled target executable. \
This package can be used as a library. Alternatively there exists a CLI tool at `ember/cmd/embedder`.

## Example

The following example can also be found at `examples/list`.

### Access embedded files from within the target application

```go
package main

import (
	"fmt"
	"io"
	"log"
	"os"

	"github.com/maja42/ember"
)

func main() {
	attachments, err := ember.Open()
	if err != nil {
		log.Fatal(err)
	}
	defer attachments.Close()

	fmt.Printf("Executable contains %d attachments\n", attachments.Count())
	contents := attachments.List()

	for _, name := range contents {
		s := attachments.Size(name)
		fmt.Printf("\nAttachment %q has %d bytes:\n", name, s)
		
		r := attachments.Reader(name)
		io.Copy(os.Stdout, r)
		fmt.Println()
	}
}
```

### Embed files into a target executable

To embed files into a compiled go executable you can use the CLI tool at `cmd/embedder`. 
Alternatively, you can also integrate embedding-logic into your own application by importing `ember/embedding` (see the [GoDoc](https://godoc.org/github.com/maja42/ember/embedding) for more information).

To use `cmd/embedder`, first create an `attachments.json` file describing the files to embed:

```json
{
  "file A": "path/to/file A.txt",
  "file B": "path/to/file B.txt"
}
```

Afterwards, attach the files to an already-built executable:

```bash
cd cmd/embedder
go build
./embedder -attachments ./attachments.json -exe ./myApp -out ./myFinishedApp
```

## Contributions

Feel free to submit feature requests, bug reports and pull requests.
