ifndef GOPATH
$(error GOPATH is undefined)
endif

UNAME_S := $(shell uname -s)
ifeq ($(UNAME_S),Darwin)
	GOOS := darwin
	LDFLAGS :=
ifdef STATIC
$(error Static linking not possible on OS X)
endif
else ifeq ($(UNAME_S),Linux)
	GOOS := linux
ifdef STATIC
	SUFFIX := -static
	LDFLAGS := --ldflags '-extldflags "-static -ltinfo -lgpm"'
else
	SUFFIX :=
	LDFLAGS :=
endif
endif

ifneq ($(shell uname -m),x86_64)
$(error "Build on $(UNAME_M) is not supported, yet.")
endif

SOURCES   := $(wildcard *.go */*.go)
BINDIR    := ../bin

BINARY32  := fzf-$(GOOS)_386$(SUFFIX)
BINARY64  := fzf-$(GOOS)_amd64$(SUFFIX)
VERSION    = $(shell fzf/$(BINARY64) --version)
RELEASE32  = fzf-$(VERSION)-$(GOOS)_386$(SUFFIX)
RELEASE64  = fzf-$(VERSION)-$(GOOS)_amd64$(SUFFIX)

all: release

release: build
	-cd fzf && cp $(BINARY32) $(RELEASE32) && tar -czf $(RELEASE32).tgz $(RELEASE32)
	cd fzf && cp $(BINARY64) $(RELEASE64) && tar -czf $(RELEASE64).tgz $(RELEASE64) && \
	rm -f $(RELEASE32) $(RELEASE64)

ifndef STATIC
build: test fzf/$(BINARY32) fzf/$(BINARY64)
else
build: test fzf/$(BINARY64)
endif

test:
	go get
	go test -v ./...

install: $(BINDIR)/fzf

uninstall:
	rm -f $(BINDIR)/fzf $(BINDIR)/$(BINARY64)

clean:
	cd fzf && rm -f fzf-*

fzf/$(BINARY32): $(SOURCES)
	cd fzf && GOARCH=386 CGO_ENABLED=1 go build -o $(BINARY32)

fzf/$(BINARY64): $(SOURCES)
	cd fzf && go build $(LDFLAGS) -o $(BINARY64)

$(BINDIR)/fzf: fzf/$(BINARY64) | $(BINDIR)
	cp -f fzf/$(BINARY64) $(BINDIR)
	cd $(BINDIR) && ln -sf $(BINARY64) fzf

$(BINDIR):
	mkdir -p $@

docker-arch:
	docker build -t junegunn/arch-sandbox - < Dockerfile.arch

docker-ubuntu:
	docker build -t junegunn/ubuntu-sandbox - < Dockerfile.ubuntu

arch: docker-arch
	docker run -i -t -v $(GOPATH):/go junegunn/$@-sandbox \
		sh -c 'cd /go/src/github.com/junegunn/fzf/src; /bin/bash'

ubuntu: docker-ubuntu
	docker run -i -t -v $(GOPATH):/go junegunn/$@-sandbox \
		sh -c 'cd /go/src/github.com/junegunn/fzf/src; /bin/bash'

linux: docker-arch
	docker run -i -t -v $(GOPATH):/go junegunn/arch-sandbox \
		/bin/bash -ci 'cd /go/src/github.com/junegunn/fzf/src; make'

linux-static: docker-ubuntu
	docker run -i -t -v $(GOPATH):/go junegunn/ubuntu-sandbox \
		/bin/bash -ci 'cd /go/src/github.com/junegunn/fzf/src; make STATIC=1'

.PHONY: all build release test install uninstall clean docker \
	linux linux-static arch ubuntu docker-arch docker-ubuntu
