# Maintainer: Duncan Hill <duncan.hill@redkingdomlabs.com>
pkgname=hacash-miner-git
pkgver=r236.91a9388
pkgrel=1
pkgdesc="Full node miner for Hacash"
arch=("x86_64")
url="https://github.com/hacash/miner"
license=("MIT")
groups=()
depends=()
makedepends=("git" "go")
provides=("hacash-miner-git" "hacash-miner")
conflicts=("hacash-miner-git" "hacash-miner")
replaces=("hacash-miner-git" "hacash-miner")
backup=("etc/hacash-miner.config.ini")
options=()
#install=
source=("github.com/hacash/chain::git+https://github.com/hacash/chain#commit=02d82f10b2927100ba19b1745f61ecd066848e3e"
        "github.com/hacash/core::git+https://github.com/hacash/core#commit=7921381a286223a23c5de4088627b877b8fe0d5c"
        "github.com/hacash/miner::git+https://github.com/hacash/miner#commit=91a93888609a41c9fd09ef3b605423406fd6a7c6"
        "github.com/hacash/mint::git+https://github.com/hacash/mint#commit=b234dcd67c6c9b73e8511e83ad9fd3c06b969f9d"
        "github.com/hacash/node::git+https://github.com/hacash/node#commit=52f036b41517b64bcb993afbbd81a20f5574e8ae"
        "github.com/hacash/service::git+https://github.com/hacash/service#commit=0b23aa2820f855d0c2d58492157b2cf786d0c6e8"
        "github.com/hacash/x16rs::git+https://github.com/hacash/x16rs#commit=99ee0a9f43b59a71afa29c0af96731648e4b9805"
        "hacash-miner.config.ini"
        "PKGLICENSE"
       )
noextract=()
md5sums=("SKIP"
         "SKIP"
         "SKIP"
         "SKIP"
         "SKIP"
         "SKIP"
         "SKIP"
         "ff272542fdaaeb8ea7f3d868db16819e"
         "dccb2e40b964265cfb0691aad1e29971")

pkgver() {
    cd "$srcdir/miner"

    # Git, no tags available.
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

build() {
    # Move the directory tree to be Go compatible.
    prefix="$srcdir/src/github.com/hacash"
    rm -rf "$prefix"
    mkdir -p "$prefix"
    for repo in chain core miner mint node service x16rs; do
        mv "$srcdir/$repo" "$prefix/$repo"
    done

    # Build the x16rs library.
    cd "$prefix/x16rs"

    cd sha3
    for f in *.c; do
        ofile=$(basename ${f})

        # md_helper.c is always broken and not needed; or so it seems...
        if [ "$ofile" != "md_helper.c" ]; then
            gcc -c -o ${ofile}.o -Ofast -march=native ${f}
        fi
    done

    cd ../sha3_256
    for f in *.c; do
        ofile=$(basename ${f})
        gcc -c -o ${ofile}.o -Ofast -march=native ${f}
    done

    cd ..
    ar qs libx16rs_hash.a sha3/*.o sha3_256/*.o

    # Build the actual miner executable.
    cd "$prefix"
    GO111MODULE=off GOPATH="$srcdir" go build \
        -ldflags '-w -s' \
        -trimpath \
        -o hacash-miner miner/run/main/main.go

    # Strip debug symbols.
    strip -s hacash-miner
}

package() {
    # Copy over the executable.
    dest="$pkgdir/usr/local/bin"
    mkdir -p "$dest"
    cp "$srcdir/src/github.com/hacash/hacash-miner" "$dest/hacash-miner"

    # Copy over the configuration file.
    dest="$pkgdir/etc"
    mkdir -p "$dest"
    cp "$srcdir/hacash-miner.config.ini" "$dest/hacash-miner.config.ini"

    # Copy over the package license.
    dest="$pkgdir/usr/share/licenses/hacash-miner"
    mkdir -p "$dest"
    cp "$srcdir/PKGLICENSE" "$dest/LICENSE"
}
