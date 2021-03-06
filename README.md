# 개요
SIPSkia(Simple Image Processing by Skia) 는 [Skia](https://skia.org/) 기반의 간단한 이미지
resize, crop 을 수행하는 파이썬 라이브러리입니다. 모씨 앱의 카드
이미지 변환에 사용되고 있습니다. 현재는 OSX Sierra 와 Ubuntu 14.04 에서
테스트 되었습니다.

# 목적
Skia 는 다른 이미지 처리 라이브러리와 비교하여 굉장히 빠른 성능과 낮은
CPU 사용률을 보입니다. 본 개발 작업을 진행하며 느낀 Skia 의 성능과
작은 노하우를 다른 분들과 공유하기 위한 목적으로 코드를 공개 하였습니다.

그런 관계로 본 프로젝트는 모씨 앱 서버 관리 이슈가 발생할 때 관리됩니다.

# 얼마나 빠른가?
Skia chrome/m52 브랜치와 [ImageMagick](https://www.imagemagick.org/) +
[Wand](http://docs.wand-py.org/en/0.4.4/) 3.x 때에는 5 ~ 6 배 이상의
성능 차이가 있었습니다만, chrome/m58 과 ImageMagick + Wand 4.x 기준으로
약 1.8 ~ 2배 정도의 성능 차이를 보입니다.

Skia 의 성능은 기존과 큰 차이가 없습니다만 ImageMagick, Wand 의 성능이
매우 좋아진 것으로 보입니다.

# 빌드 하기
## 의존성
sipskia 는 다음 버전에서 테스트 되었습니다. 아래 설치 방법이나 테스트 방법 모두
해당 버전을 기준으로 설명합니다.
* [boost](http://www.boost.org/) 1.63.0
* skia chrome/m58

# 준비
## OSX
### 1. boost-python 설치
brew 를 이용하면 깔끔하게 설치할 수 있습니다.
```bash
$ brew install boost-python
```

### 2. Skia 빌드
```bash
$ git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'
$ export PATH="${PWD}/depot_tools:${PATH}"
$ git clone https://skia.googlesource.com/skia.git
$ cd skia
$ python tools/git-sync-deps
```

위와 같이 사전 준비를 진행한 후 [ninja](https://ninja-build.org/) 를 통해 Skia 를 Static Library
로 빌드합니다.
```bash
$ bin/gn gen out/Static --args='is_official_build=true skia_enable_gpu=true skia_use_fontconfig=false cc="clang" cxx="clang++"'
$ ninja -C out/Static
```

정상적으로 빌드가 완료되면 out/Static 디렉토리에 libskia.a 파일이 생성됩니다.

## Ubuntu 14.04
사용자 명을 ubuntu 로, 홈 디렉토리를 /home/ubuntu 로 가정합니다.

### 1. boost-python 설치
boost-python 을 홈페이지에서 다운로드 받습니다. 그 후 부트스트래핑을 해 줍니다.
```bash
$ ./bootstrap.sh --with-libraries=python --prefix=/home/ubuntu/boost/
```

빌드합니다.
```bash
$ ./b2 cxxflags=-fPIC install
```
빌드가 성공적으로 완료되면 /home/ubuntu/boost/lib/ 에 libboost_python.a 와 libboost_python.so
가 생기는데 libboost_python.so 파일을 모두 삭제합니다. 현재 빌드 중 so 파일을 참조해 버리는 문제가
있습니다.

### 2. Skia 빌드
Skia 를 클론 받고 설정하는 방법은 동일합니다.
```bash
$ git clone 'https://chromium.googlesource.com/chromium/tools/depot_tools.git'
$ export PATH="${PWD}/depot_tools:${PATH}"
$ git clone https://skia.googlesource.com/skia.git
$ cd skia
$ python tools/git-sync-deps
```

위와 같이 사전 준비를 진행한 후 ninja 를 통해 Skia 를 Static Library
로 빌드합니다. Skia 빌드는 OSX 과 거의 비슷하나 ninja 설정에 약간의 차이가 있습니다.
```bash
$ bin/gn gen out/Static --args='is_official_build=true skia_enable_gpu=false skia_use_fontconfig=false skia_use_system_expat=false skia_use_system_freetype2=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false'
$ ninja -C out/Static
```

정상적으로 빌드가 완료되면 out/Static 디렉토리에 libskia.a 파일이 생성됩니다.

# sipskia
## 빌드
sipskia 를 클론 받은 후 setup.py 파일의 상단에 있는 사용자 경로 설정 부분을 알맞게 고쳐줍니다.
그 후 라이브러리를 생성합니다.
```bash
$ python setup.py build_ext
```

정상적으로 빌드가 완료되면 libsipskia.so 파일을 확인할 수 있습니다.

## 테스트
테스트를 진행하여 올바르게 이미지가 변환되는지 확인합니다.
```bash
$ python test.py convert
```

output 디렉토리에 정상적으로 변환된 파일들이 생성되는지 확인합니다.

이번엔 메모리 누수 테스트입니다.
```bash
$ python test.py leak
```

해당 프로세스가 돌고 있는 동안 프로세스 모니터링 툴을 띄워 메모리 점유율에 유의미한 변화가
발생하는지를 확인합니다.

## 벤치마크
ImageMagick + Wand 와 주로 벤치마크 비교를 합니다. Wand 홈페이지에 가서
운영체제에 맞는 Wand 를 설치합니다.

```bash
$ python benchmark.py
```

최신 버전을 기준으로 sipskia 는 ImageMagick + Wand 보다 이미지 리사이징, 크롭의 성능이
약 80 ~ 100% 정도 빠릅니다.

# 기타
앞서 언급한 것 처럼 별도의 pull request 나 issue 는 처리 되지 않습니다. 자유롭게 사용해 주세요.<br />
문의사항은 moonsoo.kim at nrise.net 으로 해 주시면 됩니다.
