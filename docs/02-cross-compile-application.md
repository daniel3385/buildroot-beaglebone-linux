# BeagleBone Black + Buildroot SDK Guide

This guide shows how to generate a Buildroot SDK, configure the cross-compilation environment, and build a simple application for the BeagleBone Black (BBB).

---

## 1. Generate the SDK (Buildroot)

Inside your Buildroot directory:

```bash
make sdk
```

This will generate a file like:

```bash
output/images/arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz
```

---

## 2. Install the SDK

Extract and install it to a directory of your choice:

```bash
cd output/images
./arm-buildroot-linux-gnueabihf_sdk-buildroot.tar.gz
```

Example install path:

```bash
/home/user/bbb_sdk
```

---

## ⚙️ 3. Activate the toolchain environment (manual export)

In theory the SDK provides an `environment-setup-*` script to configure environment, but it was not my case, so I had to manually configure the environment using exported variables.

```bash
export PATH=$HOME/bbb-sdk/bin:$PATH
```
---

## 4. Create a simple application

Create `hello.c`:

```c
#include <stdio.h>

int main() {
    printf("Hello from BeagleBone Black (Buildroot)!\n");
    return 0;
}
```

---

## 5. Compile the application

Use the cross compiler provided by the SDK:

```bash
arm-buildroot-linux-gnueabihf-gcc hello.c -o hello
```

---

## 6. Check the binary

```bash
file hello
```

Expected output:

```
ELF 32-bit LSB executable, ARM, EABI5 ...
```
---