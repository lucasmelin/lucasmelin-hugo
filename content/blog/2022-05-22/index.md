---
date: "2022-05-22"
title: "Installing Go tools globally"
tags: ["go", "til"]
summary: "A quick how-to for installing Go tools globally."
slug: install-go-tools-globally
---

There are several useful command-line tools in the Go ecosystem such as [`delve`](https://github.com/go-delve/delve),
[`godoc`](https://pkg.go.dev/golang.org/x/tools/cmd/godoc) and [`staticcheck`](https://staticcheck.io/) to name a few.

These can all be installed using Go, however there's a slight trick to ensure that the tools are installed _globally_ - you need to first change to a directory that is both outside
of your `GOPATH` and outside of a module (a temp directory works fine for this).
Then, you can simply use `go install` to install the tools like so:

```sh
go install github.com/go-delve/delve/cmd/dlv@latest
go install golang.org/x/tools/cmd/godoc
go install honnef.co/go/tools/cmd/staticcheck@latest
```
