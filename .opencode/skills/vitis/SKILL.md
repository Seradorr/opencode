---
name: vitis
description: |
  Xilinx Vitis, bare-metal C/C++ ve BSP gelistirme icin
  kod stili, driver kullanimi, DMA, interrupt, boot ve anti-pattern kurallari.
globs:
  - "**/*.c"
  - "**/*.h"
  - "**/src/**/*.c"
  - "**/src/**/*.h"
  - "**/bsp/**/*"
  - "**/platform/**/*"
  - "**/ps7_cortexa9_0/**/*"
  - "**/psu_cortexa53_0/**/*"
  - "**/psu_cortexr5_0/**/*"
  - "**/Makefile"
  - "**/CMakeLists.txt"
  - "**/lscript.ld"
alwaysApply: false
---

# VITIS EMBEDDED C KURALLARI

Bu kurallar Xilinx Vitis IDE, bare-metal C ve BSP gelistirme icin gecerlidir.

---

## 1. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Fonksiyon | snake_case | init_dma, read_register |
| Degisken | snake_case | data_buffer, byte_count |
| Macro | UPPER_SNAKE_CASE | MAX_BUFFER_SIZE |
| Struct | PascalCase veya _t | DmaConfig, dma_config_t |
| Global | g_ prefix | g_dma_instance |
| Static | s_ prefix | s_init_done |
| ISR handler | _isr suffix | dma_rx_isr, gpio_isr |

### Header Guard

```c
#ifndef PROJECT_MODULE_H
#define PROJECT_MODULE_H

/* Content */

#endif /* PROJECT_MODULE_H */
```

### Temel Kurallar

- Her public fonksiyon hata durumunda anlamli status donmeli
- Magic number yerine macro veya sabit kullan
- Register-level kodda volatile ve alignment ihtiyaclarini acikca dusun
- ISR icinde uzun is ve heap kullanimindan kacin

---

## 2. XILINX DRIVER PATTERN

### Standart Initialization

```c
#include "xgpio.h"
#include "xparameters.h"

XGpio gpio_instance;

int init_gpio(void)
{
    XGpio_Config *config;

    config = XGpio_LookupConfig(XPAR_AXI_GPIO_0_DEVICE_ID);
    if (config == NULL) {
        return XST_FAILURE;
    }

    if (XGpio_CfgInitialize(&gpio_instance, config, config->BaseAddress) != XST_SUCCESS) {
        return XST_FAILURE;
    }

    XGpio_SetDataDirection(&gpio_instance, 1, 0x00);
    return XST_SUCCESS;
}
```

### Interrupt Setup

```c
#include "xscugic.h"
#include "xil_exception.h"

XScuGic gic_instance;

int setup_interrupts(void)
{
    XScuGic_Config *config;

    config = XScuGic_LookupConfig(XPAR_SCUGIC_SINGLE_DEVICE_ID);
    if (config == NULL) {
        return XST_FAILURE;
    }

    if (XScuGic_CfgInitialize(&gic_instance, config, config->CpuBaseAddress) != XST_SUCCESS) {
        return XST_FAILURE;
    }

    Xil_ExceptionRegisterHandler(
        XIL_EXCEPTION_ID_INT,
        (Xil_ExceptionHandler)XScuGic_InterruptHandler,
        &gic_instance);
    Xil_ExceptionEnable();

    return XST_SUCCESS;
}

int connect_interrupt(u32 intr_id, Xil_InterruptHandler handler, void *callback)
{
    if (XScuGic_Connect(&gic_instance, intr_id, handler, callback) != XST_SUCCESS) {
        return XST_FAILURE;
    }

    XScuGic_Enable(&gic_instance, intr_id);
    return XST_SUCCESS;
}
```

---

## 3. DMA PATTERN'LERI

### Simple DMA Transfer

```c
#include "xaxidma.h"

XAxiDma dma_instance;

int init_dma(void)
{
    XAxiDma_Config *config;

    config = XAxiDma_LookupConfig(XPAR_AXIDMA_0_DEVICE_ID);
    if (config == NULL) {
        return XST_FAILURE;
    }

    if (XAxiDma_CfgInitialize(&dma_instance, config) != XST_SUCCESS) {
        return XST_FAILURE;
    }

    XAxiDma_IntrDisable(&dma_instance, XAXIDMA_IRQ_ALL_MASK, XAXIDMA_DEVICE_TO_DMA);
    XAxiDma_IntrDisable(&dma_instance, XAXIDMA_IRQ_ALL_MASK, XAXIDMA_DMA_TO_DEVICE);

    return XST_SUCCESS;
}

int dma_transfer(u8 *tx_buf, u8 *rx_buf, u32 len)
{
    /* TX: CPU yazdi, DMA okuyacak */
    Xil_DCacheFlushRange((UINTPTR)tx_buf, len);

    /* RX: stale line kalmamasi icin transferden once invalidate */
    Xil_DCacheInvalidateRange((UINTPTR)rx_buf, len);

    if (XAxiDma_SimpleTransfer(&dma_instance, (UINTPTR)rx_buf, len, XAXIDMA_DEVICE_TO_DMA)
        != XST_SUCCESS) {
        return XST_FAILURE;
    }

    if (XAxiDma_SimpleTransfer(&dma_instance, (UINTPTR)tx_buf, len, XAXIDMA_DMA_TO_DEVICE)
        != XST_SUCCESS) {
        return XST_FAILURE;
    }

    while (XAxiDma_Busy(&dma_instance, XAXIDMA_DEVICE_TO_DMA)) {
    }

    while (XAxiDma_Busy(&dma_instance, XAXIDMA_DMA_TO_DEVICE)) {
    }

    /* DMA tamamlandiktan sonra CPU okumadan once tekrar invalidate et */
    Xil_DCacheInvalidateRange((UINTPTR)rx_buf, len);
    return XST_SUCCESS;
}
```

### Double Buffering Pattern

```c
#define BUF_SIZE 4096

static u8 buf_a[BUF_SIZE] __attribute__((aligned(32)));
static u8 buf_b[BUF_SIZE] __attribute__((aligned(32)));
static volatile int active_buf = 0;

void dma_rx_isr(void *callback)
{
    XAxiDma *dma = (XAxiDma *)callback;
    u32 irq_status = XAxiDma_IntrGetIrq(dma, XAXIDMA_DEVICE_TO_DMA);

    XAxiDma_IntrAckIrq(dma, irq_status, XAXIDMA_DEVICE_TO_DMA);
    active_buf ^= 1;

    u8 *next_buf = (active_buf == 0) ? buf_a : buf_b;
    Xil_DCacheInvalidateRange((UINTPTR)next_buf, BUF_SIZE);
    XAxiDma_SimpleTransfer(dma, (UINTPTR)next_buf, BUF_SIZE, XAXIDMA_DEVICE_TO_DMA);
}
```

### DMA Kurallari

- RX buffer'i CPU okumadan once invalidate et
- TX buffer'i DMA okumadan once flush et
- Buffer'lari cache line alignment ile ayarla
- Timeout, error bit ve reset path'lerini goz ardi etme

---

## 4. PS-PL HABERLESME

### Port Tipleri

| PORT | BANT GENISLIGI | KULLANIM |
|------|----------------|----------|
| AXI GP | Dusuk | Register erisimi, kontrol |
| AXI HP | Yuksek | DMA, video, buyuk veri |
| AXI ACP | Yuksek + coherent | Cache tutarli veri paylasimi |
| EMIO | Dusuk hiz | GPIO, SPI, I2C, UART |

### Interrupt Mapping

```c
/* Interrupt ID'leri tasarima ve SoC ailesine gore degisir.
 * Sabit ID ezberleme; BSP tarafinda uretilen macro'lari referans al.
 */
#define PL_INTR_ID  XPAR_FABRIC_AXI_GPIO_0_IP2INTC_IRPT_INTR
```

Kurallar:
- xparameters.h ve generated BSP macro'larini birincil referans kabul et
- Interrupt mapping'i tasarim bazli dogrula
- Shared interrupt yapilarinda ack sirasi ve source temizligini kontrol et

---

## 5. BOOT SEQUENCE

```text
FSBL
  -> PS init
  -> DDR ve clock init
  -> Bitstream yukleme
  -> U-Boot veya bare-metal application
```

Boot hatalarinda kontrol et:
- Boot mode pinleri
- DDR ve clock init
- Bitstream boyutu ve yukleme akisi
- Stack, heap ve exception yapilandirmasi

---

## 6. LINKER SCRIPT

```ld
MEMORY {
    ps7_ddr_0 : ORIGIN = 0x00100000, LENGTH = 0x1FF00000
    ps7_ram_0 : ORIGIN = 0x00000000, LENGTH = 0x00030000
}

_HEAP_SIZE  = 0x10000;
_STACK_SIZE = 0x8000;
```

Kurallar:
- DMA buffer'lari alignment gereksinimine uygun olmali
- ISR veya kritik veri gerekliyse OCM kullanimi dusunulmeli
- Heap ve stack boyutlari gercek kullanim senaryosuna gore ayarlanmalı

---

## 7. DEBUG STRATEJILERI

### UART Debug

```c
#include "xil_printf.h"

xil_printf("DMA status: 0x%08x\r\n", XAxiDma_ReadReg(base, offset));
```

### Register Dump

```c
void dump_regs(u32 base, int count)
{
    for (int i = 0; i < count; i++) {
        xil_printf("REG[0x%02x] = 0x%08x\r\n", i * 4, Xil_In32(base + i * 4));
    }
}
```

Kurallar:
- JTAG, UART ve memory window birlikte dusunulmeli
- Reproduce edilebilir hata durumunda once register ve interrupt state al
- Debug print'i production yoluna yerlestirme

---

## 8. ANTI-PATTERNS

### ISR Icinde Uzun Is

```c
volatile int data_ready = 0;

void my_isr(void *callback)
{
    data_ready = 1;
}
```

### Hata Kontrolsuz Cagri

```c
if (XGpio_Initialize(&gpio, DEVICE_ID) != XST_SUCCESS) {
    return XST_FAILURE;
}
```

### Magic Number

```c
#define MY_IP_BASE  XPAR_MY_IP_0_BASEADDR
#define REG_CTRL    0x00
#define CTRL_EN     0x1F
```

### Bare-Metal malloc

- Heap fragmentasyonu ve kontrol zorlugu nedeniyle varsayilan tercih olmasin
- Uzun omurlu buffer ve DMA alanlarinda static allocation kullan

---

## 9. MULTI-CORE VE PAYLASILAN BELLEK

```c
#define SHARED_MEM_BASE  0xFFFF0000

typedef struct {
    volatile u32 flag;
    volatile u32 data[256];
} SharedMem;

#define SHARED ((SharedMem *)SHARED_MEM_BASE)
```

Kurallar:
- Shared memory protokolunu netlestir
- Cache coherency ihtiyacini acikca dusun
- IPI ve polling stratejilerini karistirma

---

## 10. KRITIK DOSYALAR

Bu dosyalarda degisiklik yapmadan once onay iste:

- `lscript.ld`
- FSBL kaynak dosyalari
- ISR fonksiyonlari
- `Makefile`, `CMakeLists.txt`
- `xparameters.h` (otomatik uretilir, manuel degistirme)
