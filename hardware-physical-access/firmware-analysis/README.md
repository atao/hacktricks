# Firmware Analysis

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* HackTricks에서 **회사 광고를 보거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)을 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

## **소개**

펌웨어는 하드웨어 구성 요소와 사용자가 상호 작용하는 소프트웨어 간의 통신을 관리하고 운영체제의 시작으로 이어지는 중요한 소프트웨어입니다. 펌웨어는 영구 메모리에 저장되어 장치가 전원을 켤 때부터 필수적인 지침에 액세스할 수 있도록 하여 운영체제를 시작합니다. 펌웨어를 조사하고 수정하는 것은 보안 취약점을 식별하는 중요한 단계입니다.

## **정보 수집**

**정보 수집**은 장치의 구성과 사용하는 기술을 이해하기 위한 중요한 초기 단계입니다. 이 과정은 다음과 같은 데이터 수집을 포함합니다:

* CPU 아키텍처 및 실행 중인 운영체제
* 부트로더 세부 정보
* 하드웨어 레이아웃 및 데이터 시트
* 코드베이스 메트릭 및 소스 위치
* 외부 라이브러리 및 라이선스 유형
* 업데이트 기록 및 규정 준수 인증
* 아키텍처 및 플로우 다이어그램
* 보안 평가 및 식별된 취약점

이를 위해 **오픈 소스 인텔리전스 (OSINT)** 도구와 수동 및 자동 검토 프로세스를 통해 사용 가능한 오픈 소스 소프트웨어 구성 요소를 분석하는 것이 매우 중요합니다. [Coverity Scan](https://scan.coverity.com) 및 [Semmle’s LGTM](https://lgtm.com/#explore)과 같은 도구는 잠재적인 문제를 찾기 위해 활용할 수 있는 무료 정적 분석을 제공합니다.

## **펌웨어 획득**

펌웨어를 얻는 방법은 복잡성에 따라 다양하게 접근할 수 있습니다:

* **직접** 소스(개발자, 제조업체)로부터
* 제공된 지침에 따라 **빌드**하기
* 공식 지원 사이트에서 **다운로드**하기
* 호스팅된 펌웨어 파일을 찾기 위해 **Google 도크** 쿼리 사용
* [S3Scanner](https://github.com/sa7mon/S3Scanner)와 같은 도구를 사용하여 **클라우드 스토리지**에 직접 액세스하기
* 중간자 공격 기술을 사용하여 **업데이트** 가로채기
* **UART**, **JTAG**, 또는 **PICit**와 같은 연결을 통해 장치에서 **추출**하기
* 장치 통신 내에서 업데이트 요청을 **스니핑**하기
* **하드코딩된 업데이트 엔드포인트** 식별 및 사용하기
* 부트로더 또는 네트워크에서 **덤프**하기
* 모든 것이 실패한 경우 적절한 하드웨어 도구를 사용하여 저장 칩을 **제거하고 읽기**

## 펌웨어 분석

이제 **펌웨어를 가지고 있으므로**, 어떻게 처리할지에 대한 정보를 추출해야 합니다. 이를 위해 다음과 같은 도구를 사용할 수 있습니다:

```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```

만약 이러한 도구로 많은 것을 찾지 못한다면 `binwalk -E <bin>` 명령으로 이미지의 **엔트로피**를 확인해보세요. 엔트로피가 낮다면 암호화되지 않았을 가능성이 높습니다. 엔트로피가 높다면 (또는 어떤 방식으로 압축되었다면) 암호화되었을 가능성이 높습니다.

또한, 다음 도구를 사용하여 **펌웨어에 내장된 파일을 추출**할 수 있습니다:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

또는 [**binvis.io**](https://binvis.io/#/) ([code](https://code.google.com/archive/p/binvis/))를 사용하여 파일을 검사할 수 있습니다.

### 파일 시스템 가져오기

이전에 언급한 `binwalk -ev <bin>`과 같은 도구를 사용하여 **파일 시스템을 추출**할 수 있어야 합니다.\
보통 binwalk는 **파일 시스템의 이름으로 된 폴더**에 추출합니다. 이 폴더의 이름은 보통 다음 중 하나입니다: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### 수동 파일 시스템 추출

가끔씩 binwalk는 **파일 시스템의 매직 바이트를 식별하지 못할 수 있습니다**. 이러한 경우, binwalk를 사용하여 파일 시스템의 오프셋을 찾고 바이너리에서 압축된 파일 시스템을 **수동으로 추출**한 다음 아래 단계에 따라 파일 시스템을 해당 유형에 맞게 수동으로 추출하면 됩니다.

```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```

다음 **dd 명령어**를 실행하여 Squashfs 파일 시스템을 추출하십시오.

```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```

또한 다음 명령어를 실행할 수도 있습니다.

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* squashfs에 대한 경우 (위의 예시에서 사용됨)

`$ unsquashfs dir.squashfs`

파일은 이후 "`squashfs-root`" 디렉토리에 있을 것입니다.

* CPIO 아카이브 파일

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* jffs2 파일 시스템의 경우

`$ jefferson rootfsfile.jffs2`

* NAND 플래시를 사용하는 ubifs 파일 시스템의 경우

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

## 펌웨어 분석

펌웨어를 얻은 후, 그 구조와 잠재적인 취약점을 이해하기 위해 분석해야 합니다. 이 과정에서 다양한 도구를 사용하여 펌웨어 이미지에서 가치 있는 데이터를 분석하고 추출합니다.

### 초기 분석 도구

바이너리 파일(`<bin>`으로 지칭됨)을 초기 검사하기 위해 일련의 명령어가 제공됩니다. 이 명령어는 파일 유형을 식별하고 문자열을 추출하며, 바이너리 데이터를 분석하고, 파티션 및 파일 시스템 세부 정보를 이해하는 데 도움이 됩니다:

```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```

이미지의 암호화 상태를 평가하기 위해 **엔트로피**는 `binwalk -E <bin>`으로 확인됩니다. 낮은 엔트로피는 암호화 부재를 나타내며, 높은 엔트로피는 가능한 암호화 또는 압축을 나타냅니다.

**내장된 파일**을 추출하기 위해, **file-data-carving-recovery-tools** 문서와 파일 검사를 위한 **binvis.io**와 같은 도구와 자원을 추천합니다.

### 파일시스템 추출

`binwalk -ev <bin>`을 사용하면 일반적으로 파일시스템을 추출할 수 있으며, 종종 파일시스템 유형에 따라 이름이 지정된 디렉토리에 추출됩니다(예: squashfs, ubifs). 그러나 **binwalk**가 마법 바이트가 누락되어 파일시스템 유형을 인식하지 못하는 경우 수동 추출이 필요합니다. 이는 `binwalk`를 사용하여 파일시스템의 오프셋을 찾은 다음 `dd` 명령을 사용하여 파일시스템을 추출하는 것을 포함합니다:

```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```

그 후에, 파일 시스템 유형에 따라 (예: squashfs, cpio, jffs2, ubifs), 내용을 수동으로 추출하기 위해 다른 명령을 사용합니다.

### 파일 시스템 분석

파일 시스템을 추출한 후에는 보안 취약점을 찾기 시작합니다. 보안 취약한 네트워크 데몬, 하드코딩된 자격 증명, API 엔드포인트, 업데이트 서버 기능, 컴파일되지 않은 코드, 시작 스크립트 및 오프라인 분석을 위한 컴파일된 이진 파일에 주의를 기울입니다.

검사해야 할 **주요 위치** 및 **항목**은 다음과 같습니다:

* 사용자 자격 증명을 위한 **etc/shadow** 및 **etc/passwd**
* **etc/ssl**에 있는 SSL 인증서 및 키
* 잠재적인 취약점을 위한 구성 및 스크립트 파일
* 추가 분석을 위한 포함된 이진 파일
* 일반적인 IoT 장치 웹 서버 및 이진 파일

파일 시스템 내에서 민감한 정보와 취약점을 발견하기 위해 여러 도구를 사용할 수 있습니다:

* 민감한 정보 검색을 위한 [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) 및 [**Firmwalker**](https://github.com/craigz28/firmwalker)
* 포괄적인 펌웨어 분석을 위한 [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core)
* 정적 및 동적 분석을 위한 [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go) 및 [**EMBA**](https://github.com/e-m-b-a/emba)

### 컴파일된 이진 파일의 보안 검사

파일 시스템에서 찾은 소스 코드와 컴파일된 이진 파일은 취약점을 조사해야 합니다. Unix 이진 파일에 대한 **checksec.sh** 및 Windows 이진 파일에 대한 **PESecurity**와 같은 도구를 사용하여 악용될 수 있는 보호되지 않은 이진 파일을 식별할 수 있습니다.

## 동적 분석을 위한 펌웨어 에뮬레이션

펌웨어를 에뮬레이션하여 장치의 동작 또는 개별 프로그램을 **동적 분석**할 수 있습니다. 이 접근 방식은 하드웨어 또는 아키텍처 종속성으로 인해 도전을 겪을 수 있지만, 루트 파일 시스템이나 특정 이진 파일을 Raspberry Pi와 같은 일치하는 아키텍처와 엔디안을 가진 장치로 전송하거나 미리 구축된 가상 머신에 전송하여 추가 테스트를 용이하게 할 수 있습니다.

### 개별 이진 파일 에뮬레이션

개별 프로그램을 검사하기 위해서는 프로그램의 엔디안과 CPU 아키텍처를 식별하는 것이 중요합니다.

#### MIPS 아키텍처 예제

MIPS 아키텍처 이진 파일을 에뮬레이션하기 위해 다음 명령을 사용할 수 있습니다:

```bash
file ./squashfs-root/bin/busybox
```

그리고 필요한 에뮬레이션 도구를 설치하십시오:

```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```

MIPS (big-endian)의 경우 `qemu-mips`를 사용하고, little-endian 바이너리의 경우 `qemu-mipsel`을 선택해야 합니다.

#### ARM 아키텍처 에뮬레이션

ARM 바이너리의 경우, 에뮬레이션을 위해 `qemu-arm` 에뮬레이터를 사용합니다.

### 전체 시스템 에뮬레이션

[Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit) 등의 도구를 사용하면 전체 펌웨어 에뮬레이션을 수행하여 프로세스를 자동화하고 동적 분석을 돕습니다.

## 실제 동적 분석 기법

이 단계에서는 실제 또는 에뮬레이션된 장치 환경을 사용하여 분석합니다. OS와 파일 시스템에 대한 쉘 액세스를 유지하는 것이 중요합니다. 에뮬레이션은 하드웨어 상호작용을 완벽하게 모방하지 못할 수 있으므로 때로는 에뮬레이션을 다시 시작해야 할 수도 있습니다. 분석은 파일 시스템을 다시 검토하고 노출된 웹 페이지 및 네트워크 서비스를 이용하며 부트로더 취약점을 탐색해야 합니다. 펌웨어 무결성 테스트는 잠재적인 배후 취약점을 식별하는 데 중요합니다.

## 런타임 분석 기법

런타임 분석은 gdb-multiarch, Frida, Ghidra와 같은 도구를 사용하여 프로세스나 바이너리와 상호작용하며 퍼징 및 기타 기법을 통해 취약점을 식별하는 것을 의미합니다.

## 바이너리 공격과 PoC

식별된 취약점에 대한 PoC를 개발하려면 대상 아키텍처에 대한 깊은 이해와 저수준 언어로의 프로그래밍 능력이 필요합니다. 임베디드 시스템에서의 바이너리 런타임 보호 기능은 드물지만, 존재할 경우 Return Oriented Programming (ROP)과 같은 기법이 필요할 수 있습니다.

## 펌웨어 분석을 위한 준비된 운영 체제

[AttifyOS](https://github.com/adi0x90/attifyos)와 [EmbedOS](https://github.com/scriptingxss/EmbedOS)와 같은 운영 체제는 필요한 도구가 탑재된 펌웨어 보안 테스트를 위한 사전 구성된 환경을 제공합니다.

## 펌웨어 분석을 위한 준비된 운영 체제

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS는 사물 인터넷(IoT) 장치의 보안 평가 및 침투 테스트를 수행하는 데 도움이 되는 배포판입니다. 필요한 도구가 로드된 사전 구성된 환경을 제공하여 시간을 절약해 줍니다.
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): 펌웨어 보안 테스트 도구가 탑재된 Ubuntu 18.04 기반의 임베디드 보안 테스트 운영 체제입니다.

## 연습용 취약한 펌웨어

펌웨어에서 취약점을 발견하기 위해 다음과 같은 취약한 펌웨어 프로젝트를 시작점으로 활용할 수 있습니다.

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* The Damn Vulnerable Router Firmware Project
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## 참고 자료

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## 교육 및 인증

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 제로에서 영웅까지 AWS 해킹 배우기<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**구독 플랜**](https://github.com/sponsors/carlospolop)을 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**를** 팔로우하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **자신의 해킹 기법을 공유**하세요.

</details>