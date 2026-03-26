---
name: fpga
description: |
  VHDL, Verilog, SystemVerilog ve Vivado icin
  kod stili, anti-pattern, CDC, AXI, timing ve testbench kurallari.
globs:
  - "**/*.vhd"
  - "**/*.vhdl"
  - "**/*.v"
  - "**/*.sv"
  - "**/*.svh"
  - "**/*.xdc"
  - "**/*.tcl"
  - "**/hdl/**/*"
  - "**/rtl/**/*"
  - "**/ip/**/*"
  - "**/constraints/**/*"
alwaysApply: false
---

# FPGA RTL KURALLARI

Bu kurallar VHDL, Verilog, SystemVerilog ve constraint dosyalari icin gecerlidir.

---

## 1. TESTBENCH AYIRIMI (KRITIK)

| DURUM | EYLEM |
|-------|-------|
| "debounce kodu yaz" | SADECE debounce.vhd |
| "testbench de ekle" | debounce.vhd + debounce_tb.vhd (AYRI) |
| "simulasyon kodu" | Testbench AYRI dosya |

### Dosya Isimlendirme

```
TASARIM:    module_name.vhd     veya  module_name.v
TESTBENCH:  module_name_tb.vhd  veya  module_name_tb.v
PACKAGE:    module_name_pkg.vhd
```

---

## 2. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Entity/Module | snake_case | axi_lite_controller |
| Port | snake_case | data_valid, clk_100mhz |
| Signal | snake_case | state_reg, next_state |
| Constant/Generic | UPPER_SNAKE_CASE | DATA_WIDTH, CLK_FREQ |
| Type | snake_case_t | state_t, data_array_t |
| FSM State | ST_PREFIX | ST_IDLE, ST_PROCESS |
| Clock | clk_ prefix | clk_sys, clk_axi |
| Reset | rst_ / rstn_ | rst_sys, rstn_periph |
| Enable | en_ prefix | en_data, en_process |

### Port Sirasi

1. Clock sinyalleri (`clk_*`)
2. Reset sinyalleri (`rst_*`, `rstn_*`)
3. Enable/control sinyalleri
4. Input data sinyalleri
5. Output data sinyalleri
6. Status/flag sinyalleri

### Format

- Girinti: 2 veya 4 space (tutarli)
- Tab KULLANMA
- Satir uzunlugu: max 120 karakter
- Her process/always basinda yorum

---

## 3. TASARIM KALIPLARI

### FSM - Two-Process (VHDL)

```vhdl
-- State register (sequential)
process(clk)
begin
    if rising_edge(clk) then
        if rst = '1' then
            state_reg <= ST_IDLE;
        else
            state_reg <= state_next;
        end if;
    end if;
end process;

-- Next state + output logic (combinational)
process(state_reg, inputs)
begin
    state_next <= state_reg;  -- Default: hold
    case state_reg is
        when ST_IDLE =>
            if start = '1' then
                state_next <= ST_PROCESS;
            end if;
        when others =>
            state_next <= ST_IDLE;
    end case;
end process;
```

### Synchronous Reset Tercihi

- Aktif-high (`rst`) veya aktif-low (`rstn`) — TUTARLI ol
- Asenkron reset sadece ozel durumlar (BRAM, DSP)

### Pipeline Register

```vhdl
process(clk)
begin
    if rising_edge(clk) then
        if ce = '1' then
            stage1 <= input_data;
            stage2 <= stage1;
            stage3 <= stage2;
        end if;
    end if;
end process;
```

### Parametrik/Generic Tasarim

```vhdl
entity data_buffer is
    generic (
        DATA_WIDTH : integer := 8;
        DEPTH      : integer := 16
    );
    port (
        clk     : in  std_logic;
        rst     : in  std_logic;
        wr_en   : in  std_logic;
        wr_data : in  std_logic_vector(DATA_WIDTH-1 downto 0);
        rd_en   : in  std_logic;
        rd_data : out std_logic_vector(DATA_WIDTH-1 downto 0);
        full    : out std_logic;
        empty   : out std_logic
    );
end entity;
```

Generic'leri kullanarak yeniden kullanilabilir modeller tasarla.

---

## 4. CDC (CLOCK DOMAIN CROSSING)

### Karar Agaci

```
Sinyal clock domain gecisi yapacak mi?
├─ Tek bit sinyal mi?
│  ├─ Yavas → Hizli: 2-stage synchronizer YETERLI
│  └─ Hizli → Yavas: 2-stage + pulse stretch veya toggle sync
├─ Coklu bit (bus/veri) mi?
│  ├─ Dusuk hiz, kayip tolere edilebilir: Gray code + 2-stage sync
│  ├─ Veri butunlugu kritik: Async FIFO (dual-clock FIFO)
│  └─ Kontrol sinyali + veri: Handshake protokolu (req/ack)
└─ Reset sinyali mi?
   → Async assert, sync deassert (reset synchronizer)
```

### 2-Stage Synchronizer

```vhdl
process(clk_dst)
begin
    if rising_edge(clk_dst) then
        sync_ff1 <= signal_src;
        sync_ff2 <= sync_ff1;
    end if;
end process;
signal_dst_safe <= sync_ff2;
```

### 3-Stage Synchronizer (Yuksek guvenilirlik)

```vhdl
-- Metastability olasiligi azaltmak icin (MTBF artisi)
process(clk_dst)
begin
    if rising_edge(clk_dst) then
        sync_ff1 <= signal_src;
        sync_ff2 <= sync_ff1;
        sync_ff3 <= sync_ff2;
    end if;
end process;
signal_dst_safe <= sync_ff3;
```

### Async FIFO (Bus transfer icin)

```vhdl
-- Dual-clock FIFO kullan (Xilinx FIFO Generator IP veya manual)
-- Write side: clk_wr domain
-- Read side: clk_rd domain
-- Gray code pointer'lar ile full/empty uretimi
-- Minimum derinlik: Burst boyutu + CDC latency
```

### CDC Anti-Patterns

```vhdl
-- YANLIS: Direkt baglanti (CDC ihlali)
signal_clkB <= signal_clkA;

-- YANLIS: Combinational logic CDC cikisinda
output <= sync_ff2 and some_signal;  -- Glitch riski

-- DOGRU: Synchronized sinyal sadece register'da kullan
process(clk_dst)
begin
    if rising_edge(clk_dst) then
        output_reg <= sync_ff2 and some_signal;
    end if;
end process;
```

---

## 5. TIMING CLOSURE STRATEJILERI

### Critical Path Kisaltma

```
1. Combinational logic zincirini bol → Pipeline stage ekle
2. Buyuk MUX/decoder → One-hot encoding veya binary tree
3. Carry chain uzunlugu → DSP/BRAM inference kullan
4. Fan-out azalt → Register duplication (Vivado otomatik yapabilir)
```

### Constraint Yazimi (XDC)

```tcl
# Clock tanimlari
create_clock -name sys_clk -period 10.0 [get_ports clk_100mhz]
create_clock -name axi_clk -period 8.0 [get_ports clk_125mhz]

# Clock domain arasi
set_clock_groups -asynchronous -group [get_clocks sys_clk] -group [get_clocks axi_clk]

# CDC synchronizer register'larini ASYNC_REG olarak isaretle
set_property ASYNC_REG TRUE [get_cells -hier -regexp {.*sync_ff[12].*}]

# False path'i ilk senkronizer kademesinin D pinine hedefli uygula
set_false_path -to [get_pins -hier -regexp {.*sync_ff1.*/D}]

# Multicycle path (ornek: 2 cycle)
set_multicycle_path 2 -setup -from [get_cells src_reg] -to [get_cells dst_reg]
set_multicycle_path 1 -hold -from [get_cells src_reg] -to [get_cells dst_reg]

# IO timing
set_input_delay -clock sys_clk -max 2.0 [get_ports data_in]
set_output_delay -clock sys_clk -max 1.5 [get_ports data_out]
```

### Pipelining Pattern

```
Timing karsilanmiyorsa:
1. WNS (Worst Negative Slack) kontrol et
2. Critical path'i Vivado'da incele
3. En uzun combinational logic'i bul
4. Pipeline register ekle (throughput artar, latency artar)
5. Retiming enable et (Vivado synthesis option)
```

---

## 6. AXI INTERFACE TASARIMI

### AXI4-Lite Slave Register Map

```vhdl
-- Register adresleri (ornek)
constant REG_CTRL    : integer := 16#00#;  -- Control register
constant REG_STATUS  : integer := 16#04#;  -- Status register (RO)
constant REG_DATA_IN : integer := 16#08#;  -- Data input
constant REG_DATA_OUT: integer := 16#0C#;  -- Data output (RO)

-- Write process
process(s_axi_aclk)
begin
    if rising_edge(s_axi_aclk) then
        if s_axi_aresetn = '0' then
            ctrl_reg <= (others => '0');
        elsif wr_en = '1' then
            case to_integer(unsigned(wr_addr)) is
                when REG_CTRL    => ctrl_reg <= s_axi_wdata;
                when REG_DATA_IN => data_in_reg <= s_axi_wdata;
                when others      => null;
            end case;
        end if;
    end if;
end process;
```

### AXI4-Stream Handshake

```vhdl
-- Master: TVALID = veri hazir, TDATA = veri
-- Slave: TREADY = alabiliyorum
-- Transfer: TVALID='1' AND TREADY='1' olan clock edge'de gerceklesir

-- KURALLAR:
-- 1. TVALID assert edildikten sonra TREADY gelene kadar LOW'a cekilemez
-- 2. TREADY herhangi bir zamanda assert/deassert edilebilir
-- 3. TLAST son transfer'i isaretler (paket sonu)
```

### AXI Tasarim Kurallari

- AXI clock ve reset adlandirmasi: `s_axi_aclk`, `s_axi_aresetn`
- Reset aktif-low (AXI standardi)
- WSTRB (write strobe) byte enable olarak kullan
- BRESP/RRESP hata kodlari: OKAY, SLVERR, DECERR

---

## 7. RESOURCE OPTIMIZATION

### DSP Inference

```vhdl
-- DSP48E2 inference icin:
-- Multiply-accumulate pattern
process(clk)
begin
    if rising_edge(clk) then
        mult_result <= signed(a) * signed(b);   -- DSP multiplier
        accum <= accum + mult_result;            -- DSP accumulator
    end if;
end process;

-- KURALLAR:
-- 1. Pipeline register kullan (DSP icindeki register'lar)
-- 2. Signed/unsigned tutarli ol
-- 3. Bit genisligi DSP limitinde tut (25x18 veya 27x18)
```

### BRAM Inference

```vhdl
-- BRAM inference icin:
-- 1. Sync read + sync write
-- 2. Yeterli derinlik (genellikle 32+ adres)
-- 3. Array tanimla

type mem_array_t is array(0 to DEPTH-1) of std_logic_vector(WIDTH-1 downto 0);
signal mem : mem_array_t;

process(clk)
begin
    if rising_edge(clk) then
        if we = '1' then
            mem(to_integer(unsigned(addr))) <= din;
        end if;
        dout <= mem(to_integer(unsigned(addr)));  -- Read-first
    end if;
end process;
```

### LUT Optimization

- Gereksiz logic'i sadelelestir
- Boolean ifadeleri minimize et
- Resource sharing: ayni arithmetic unit'i time-multiplex et

---

## 8. ANTI-PATTERNS

### Latch Olusturan Kod

```vhdl
-- YANLIS: else yok = LATCH
process(sel, a, b)
begin
    if sel = '1' then
        output <= a;
    end if;
end process;

-- DOGRU: Tum dallar tanimli
process(sel, a, b)
begin
    if sel = '1' then
        output <= a;
    else
        output <= b;
    end if;
end process;
```

### Eksik Sensitivity List

```vhdl
-- YANLIS
process(a)  -- b eksik!
begin
    output <= a and b;
end process;

-- DOGRU (VHDL-2008)
process(all)
begin
    output <= a and b;
end process;
```

### Blocking vs Non-Blocking (Verilog)

```verilog
// YANLIS: Sequential'da blocking
always @(posedge clk) begin
    a = b;   // YANLIS
    c = a;
end

// DOGRU: Non-blocking
always @(posedge clk) begin
    a <= b;
    c <= a;
end
```

### Multi-driven Net

```vhdl
-- YANLIS: Ayni sinyale iki process'ten atama
process(clk) begin
    output <= a;
end process;
process(clk) begin
    output <= b;  -- Multi-driven!
end process;

-- DOGRU: Tek process veya MUX ile secim
```

---

## 9. IP INTEGRATION

### Vivado IP Integrator Best Practices

- IP'leri modüler tut: her IP tek islev
- Clock ve reset'leri dogru bagla (Clocking Wizard kullan)
- AXI Interconnect'te adres haritasini kontrol et
- Custom IP paketlerken version numarasi ver

### Custom IP Paketleme

```
1. Tools → Create and Package New IP
2. Interface tanimla (AXI4-Lite, AXI4-Stream, custom)
3. Port ve register haritasini olustur
4. Driver dosyalarini (C header) otomatik uret
5. IP Catalog'a ekle
```

---

## 10. VIVADO SENTEZ UYARILARI

| UYARI | ANLAM | COZUM |
|-------|-------|-------|
| Inferred latch | Latch olusmus | Eksik else/default ekle |
| Multi-driven net | Birden fazla surucu | Tek process veya tri-state |
| Timing not met | Constraint saglanmamis | Pipeline, constraint guncelle |
| DSP inference failed | DSP cikarilamamis | Bit genisligi/pattern kontrol |
| BRAM inference failed | RAM olarak cikarilamamis | Sync read/write kontrol |

---

## 11. TESTBENCH YAPISI (Sadece Istendiginde)

```vhdl
-- Clock generation
clk_process: process
begin
    clk <= '0'; wait for CLK_PERIOD/2;
    clk <= '1'; wait for CLK_PERIOD/2;
end process;

-- Reset generation
rst_process: process
begin
    rst <= '1';
    wait for CLK_PERIOD * 10;
    rst <= '0';
    wait;
end process;

-- Stimulus
stim_process: process
begin
    wait until rst = '0';
    wait for CLK_PERIOD * 2;
    -- Test cases
    report "Test completed" severity note;
    wait;
end process;
```

---

## 12. KRITIK DOSYALAR

Bu dosyalarda degisiklik yapmadan once **ONAY ISTE**:
- `*.xdc` — Constraint dosyalari
- Top-level entity/module
- Clock/reset modulleri
- AXI interface modulleri
