# gopls 바이너리 빌드

gopls 는 공식 바이너리 배포가 없어 Go 툴체인으로 교차 빌드해 동봉합니다 (순수 Go, CGO 없음).

```sh
mkdir gopls-build && cd gopls-build
go mod init athena/gopls-build
go get golang.org/x/tools/gopls@v0.23.0
for t in darwin/arm64 darwin/amd64 linux/arm64 linux/amd64 windows/arm64 windows/amd64; do
  os=${t%/*}; arch=${t#*/}
  key=$os; [ "$os" = windows ] && key=win32
  a=$arch;  [ "$arch" = amd64 ] && a=x64
  ext="";   [ "$os" = windows ] && ext=.exe
  CGO_ENABLED=0 GOOS=$os GOARCH=$arch go build -trimpath -ldflags="-s -w" \
    -o "<repo>/plugins/lsp-go/platforms/$key-$a/gopls$ext" golang.org/x/tools/gopls
done
```

버전을 올릴 때는 `go get` 의 태그와 `.athena-plugin/plugin.json` 의 `version` 을 함께 바꿉니다.
