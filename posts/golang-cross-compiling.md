# Golang 交叉编译

```sh
$ env GOOS=windows GOARCH=amd64 go build github.com/major1201/grabber
```

查看目前支持的交叉编译列表：

```bash
go tool dist list
```

GOOS - Target Operating System|GOARCH - Target Platform
---|---
android|arm
android|arm64
android|386
android|amd64
darwin|386
darwin|amd64
darwin|arm
darwin|arm64
dragonfly|amd64
freebsd|386
freebsd|amd64
freebsd|arm
linux|386
linux|amd64
linux|arm
linux|arm64
linux|ppc64
linux|ppc64le
linux|mips
linux|mipsle
linux|mips64
linux|mips64le
netbsd|386
netbsd|amd64
netbsd|arm
openbsd|386
openbsd|amd64
openbsd|arm
plan9|386
plan9|amd64
solaris|amd64
windows|386
windows|amd64

**注意** ：编译 Android 平台需要安装 `Android NDK`，并且需要一些额外的步骤

## 编译 Android 平台程序

1. 下载 `Android NDK`：<https://github.com/android-ndk/ndk/wiki>
2. 解压包
3. 创建 standalone toolchain:
```bash
cd android-ndk-r18b/build/tools
for i in arm64 arm x86_64 x86; do
    python make_standalone_toolchain.py --arch $i --install-dir /opt/ndk/$i
done
```
4. 设置 `PATH`:
```
PATH=$PATH:/opt/ndk/arm64/bin:/opt/ndk/arm/bin:/opt/ndk/x86_64/bin:/opt/ndk/x86/bin
```
5. 开始编译：
```bash
env CGO_ENABLED=1 GOOS=android GOARCH=arm64 CC=aarch64-linux-android-clang CCX=aarch64-linux-android-clang++ go build .
env CGO_ENABLED=1 GOOS=android GOARCH=arm   CC=arm-linux-androideabi-clang CCX=arm-linux-androideabi-clang++ go build .
env CGO_ENABLED=1 GOOS=android GOARCH=386   CC=i686-linux-android-clang    CCX=i686-linux-android-clang++    go build .
env CGO_ENABLED=1 GOOS=android GOARCH=amd64 CC=x86_64-linux-android-clang  CCX=x86_64-linux-android-clang++  go build .
```

## 生成 release 脚本

示例（请根据自己情况更改 `package` 和 `version` 变量）：

```bash
#!/bin/bash

package='github.com/major1201/dohproxy'
version='0.1.1'

package_split=(${package//\// })
package_name=${package_split[-1]}

ndk_root=/opt/ndk

platforms=(
    "android/arm"
    "android/arm64"
    "android/386"
    "android/amd64"
    # "darwin/386"
    "darwin/amd64"
    # "darwin/arm"
    # "darwin/arm64"
    "dragonfly/amd64"
    # "freebsd/386"
    "freebsd/amd64"
    # "freebsd/arm"
    # "linux/386"
    "linux/amd64"
    "linux/arm"
    "linux/arm64"
    # "linux/ppc64"
    # "linux/ppc64le"
    # "linux/mips"
    # "linux/mipsle"
    # "linux/mips64"
    # "linux/mips64le"
    # "netbsd/386"
    # "netbsd/amd64"
    # "netbsd/arm"
    # "openbsd/386"
    # "openbsd/amd64"
    # "openbsd/arm"
    # "plan9/386"
    # "plan9/amd64"
    # "solaris/amd64"
    "windows/386"
    "windows/amd64"
)

mkdir -p release

export PATH=$PATH:${ndk_root}/arm/bin:${ndk_root}/arm64/bin:${ndk_root}/x86/bin:${ndk_root}/x86_64/bin

function print_error() {
    echo "An error has occurred! Aborting the script execution... (platform=${platform})"
    exit 1
}

for platform in "${platforms[@]}"
do
    platform_split=(${platform//\// })
    GOOS=${platform_split[0]}
    GOARCH=${platform_split[1]}

    output_name=${package_name}'-'${GOOS}'_'${GOARCH}'-'${version}
    output_path="${output_name}"
    if [ "${GOOS}" = "windows" ]; then
        output_path+='.exe'
    fi

    echo "Building ${package_name} for ${platform}..."
    case "${GOOS}" in
    "windows")
        if env GOOS="${GOOS}" GOARCH="${GOARCH}" go build -o "${output_path}" "${package}"; then
            rm -f "release/${output_name}.zip"
            zip "release/${output_name}.zip" "${output_path}"
        else
            print_error
        fi
        ;;
    "android")
        case "${GOARCH}" in
        "arm")
            CC=arm-linux-androideabi-clang
            CCX=arm-linux-androideabi-clang++
            ;;
        "arm64")
            CC=aarch64-linux-android-clang
            CCX=aarch64-linux-android-clang++
            ;;
        "386")
            CC=i686-linux-android-clang
            CCX=i686-linux-android-clang++
            ;;
        "amd64")
            CC=x86_64-linux-android-clang
            CCX=x86_64-linux-android-clang++
            ;;
        *)
            echo "Unknown GOARCH: ${GOARCH}"
            exit 2
            ;;
        esac

        if env CGO_ENABLED=1 GOOS="${GOOS}" GOARCH="${GOARCH}" CC="${CC}" CCX="${CCX}" go build -o "${output_path}" "${package}"; then
            rm -f "release/${output_name}.tar.gz"
            tar czvf "release/${output_name}.tar.gz" "${output_path}"
        else
            print_error
        fi
        ;;
    *)
        if env GOOS="${GOOS}" GOARCH="${GOARCH}" go build -tags netgo -o "${output_path}" "${package}"; then # `-tags netgo` enables binaryies run in Alpine Docker image
            rm -f "release/${output_name}.tar.gz"
            tar czvf "release/${output_name}.tar.gz" "${output_path}"
        else
            print_error
        fi
        ;;
    esac
    rm -f "${output_path}"
done
```

参考：

- <https://www.digitalocean.com/community/tutorials/how-to-build-go-executables-for-multiple-platforms-on-ubuntu-16-04>
- <https://github.com/jedisct1/dnscrypt-proxy/wiki/Building-the-Android-version-on-non-Android-OS>
