# Teaching ERA5 Reanalysis & GIS

## การเรียนรู้ข้อมูล ERA5 Reanalysis ด้วย Python, Xarray, Cartopy และ Google Colab

Repository นี้จัดทำขึ้นเพื่อใช้ในการเรียนการสอนการวิเคราะห์ข้อมูลบรรยากาศจาก **ERA5 reanalysis** ร่วมกับแนวคิดทางอุตุนิยมวิทยา ภูมิศาสตร์กายภาพ และภูมิสารสนเทศ โดยเน้นให้นิสิตเรียนรู้ตั้งแต่โครงสร้างข้อมูล NetCDF ไปจนถึงการอ่านแผนที่อากาศ การวิเคราะห์บรรยากาศหลายระดับความกดอากาศ การวิเคราะห์ space–time ด้วย Hovmöller diagram การสร้าง climatology และ anomaly และการสังเคราะห์เป็นกรณีศึกษาแบบ mini research project

> เป้าหมายของรายวิชาไม่ใช่เพียง “รันโค้ดให้ได้” แต่ให้นิสิตสามารถเชื่อมโยง  
> **ข้อมูล → วิธีวิเคราะห์ → แผนที่ → หลักการทางฟิสิกส์ → การตีความ → ข้อจำกัด**

Repository: [https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS](https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS)

---

## 1. เหมาะสำหรับใคร

เนื้อหาออกแบบสำหรับนิสิตระดับปริญญาตรีชั้นปีที่ 3–4 ในสาขา เช่น

- วิทยาศาสตร์สิ่งแวดล้อม
- ภูมิศาสตร์
- ภูมิสารสนเทศ
- อุตุนิยมวิทยาประยุกต์
- ทรัพยากรธรรมชาติและสิ่งแวดล้อม

รวมถึงสามารถใช้เป็นพื้นฐานสำหรับนิสิตระดับบัณฑิตศึกษาที่ต้องการเริ่มต้นวิเคราะห์ atmospheric reanalysis ด้วย Python

ความรู้พื้นฐานที่ช่วยให้เรียนได้ง่ายขึ้น ได้แก่ Python เบื้องต้น, latitude/longitude, weather map และความเข้าใจพื้นฐานเรื่องชั้นบรรยากาศ แต่ Notebook ได้ออกแบบให้ทบทวนแนวคิดที่จำเป็นระหว่างบทเรียน

---

## 2. ERA5 คืออะไร

**ERA5** คือ atmospheric reanalysis รุ่นที่ 5 ของ European Centre for Medium-Range Weather Forecasts (ECMWF)

Reanalysis เป็นข้อมูลที่เกิดจากการผสาน

```text
Atmospheric observations
        +
Numerical weather prediction model
        +
Data assimilation
        ↓
Spatially and temporally consistent atmospheric dataset
```

ดังนั้นต้องจำว่า

```text
Reanalysis ≠ direct observation
Reanalysis ≠ weather forecast
```

Reanalysis มีประโยชน์มากในการศึกษาสภาพบรรยากาศย้อนหลัง เพราะให้ตัวแปรหลายชนิดบน grid ที่ต่อเนื่องทั้งในอวกาศและเวลา

เอกสารอ้างอิงหลัก:

- [ECMWF ERA5 Documentation](https://confluence.ecmwf.int/display/CKB/ERA5%3A+data+documentation)
- [Hersbach et al. (2020), The ERA5 global reanalysis](https://doi.org/10.1002/qj.3803)

---

## 3. แนวคิดสำคัญของหลักสูตร

ข้อมูลบรรยากาศสามารถมองเป็น atmospheric data cube

$$
X = X(t,p,\phi,\lambda)
$$

โดย

- $t$ = time
- $p$ = pressure level
- $\phi$ = latitude
- $\lambda$ = longitude

เส้นทางการเรียนใน repository นี้คือ

```text
Where are the data?
        ↓
How are the data structured?
        ↓
Where are they on Earth?
        ↓
How does the atmosphere vary vertically?
        ↓
How does the wind behave?
        ↓
How do we build synoptic weather maps?
        ↓
How does the pattern evolve in space and time?
        ↓
What is the normal climatological state?
        ↓
How different is a target period from that baseline?
        ↓
How do we integrate all evidence into a case study?
```

---

## 4. ข้อมูลสำหรับการเรียน

ข้อมูลขนาดใหญ่ถูกเตรียมให้อยู่ในรูป compact teaching datasets เพื่อให้สามารถใช้งานบน Google Colab และดาวน์โหลดจาก GitHub ได้สะดวก

Folder:

```text
00_prepared_data/
```

ประกอบด้วยข้อมูลหลัก 3 ชุด

### 4.1 EVENT — Pressure-Level Dataset

```text
era5_noul_20111002_05_SEAsia_pressure_levels_0.5deg.nc
```

ลักษณะสำคัญ:

- 2–5 October 2011
- มี 4 time steps
- ทุก time step คือ **00 UTC synoptic snapshot**
- ไม่ใช่ daily mean
- pressure levels: 1000, 925, 850, 700, 500, 300, 250 และ 200 hPa
- variables หลัก: `t`, `r`, `q`, `z`, `u`, `v`, `vo`, `d`
- horizontal grid spacing ประมาณ 0.5°

ใช้หลักใน Notebook 01–05 และ 09

### 4.2 SPATIOTEMPORAL — 850-hPa Zonal Wind

```text
era5_u850_6hourly_20110901_20111031_Asia_0.5deg.nc
```

ลักษณะสำคัญ:

- 1 September–31 October 2011
- temporal interval = 6 ชั่วโมง
- เวลา 00, 06, 12 และ 18 UTC
- variable = `u`
- pressure level = 850 hPa ตาม source/preparation metadata

ข้อควรจำ:

```text
u ≠ wind speed
```

`u` คือ zonal component ของลมเท่านั้น

ใช้หลักใน Notebook 06 และ 09

### 4.3 MONTHLY BASELINE — January/July Monthly Means

```text
era5_850hPa_JanJul_monthlymeans_1981_2020_Asia_0.5deg.nc
```

ลักษณะสำคัญ:

- 1981–2020
- มีเฉพาะ January และ July
- ข้อมูลแต่ละ time step เป็น monthly mean
- variables: `z`, `r`, `t`, `u`, `v`, `vo`
- pressure level = 850 hPa ตาม source/preparation metadata

ข้อควรจำ:

> ไฟล์นี้เป็น **monthly-mean archive** ไม่ใช่ precomputed climatology

Climatology จะถูกคำนวณใน Notebook 07 โดยเฉลี่ยเดือนเดียวกันข้ามหลายปี

และเนื่องจากไม่มี October:

```text
No October climatology
→ No October 2011 climatological anomaly
```

---

## 5. วิธีเริ่มเรียน

สำหรับนิสิต ให้เริ่มจาก Notebook 00 แล้วเรียนต่อเรียงตามลำดับ

```text
00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09
```

Notebook `00A` เป็น **Instructor / Data Preparation Notebook** ใช้สำหรับเตรียม compact teaching datasets จาก ERA5 source files ขนาดใหญ่ จึงไม่จำเป็นสำหรับการเรียนปกติของนิสิต

### เปิดใน Google Colab

| Notebook | หัวข้อ | Colab |
|---|---|---|
| 00A | Prepare ERA5 Teaching Datasets — Instructor | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/00A_Prepare_ERA5_Teaching_Datasets.ipynb) |
| 00 | Download, Setup & Data Audit | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/00_Get_ERA5_Teaching_Data_and_Audit.ipynb) |
| 01 | NetCDF & Xarray Fundamentals | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/01_NetCDF_Xarray_Fundamentals.ipynb) |
| 02 | Spatial Subsetting, Cartopy & Map Projections | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/02_Spatial_Subsetting_Cartopy_and_Map_Projections.ipynb) |
| 03 | Pressure-Level Atmospheric Structure | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/03_Pressure_Level_Atmospheric_Structure.ipynb) |
| 04 | Wind & Vector Analysis | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/04_Wind_and_Vector_Analysis.ipynb) |
| 05 | Synoptic Weather Maps & Atmospheric Dynamics | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/05_Synoptic_Weather_Maps_and_Atmospheric_Dynamics.ipynb) |
| 06 | Hovmöller & Spatiotemporal Analysis | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/06_Hovmoller_and_Spatiotemporal_Analysis.ipynb) |
| 07 | Climatology & Seasonal Circulation | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/07_Climatology_and_Seasonal_Circulation.ipynb) |
| 08 | Anomaly Concepts & Baseline Matching | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/08_Anomaly_Concepts_and_Baseline_Matching.ipynb) |
| 09 | Integrated Noul 2011 Case Study | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/09_Integrated_Noul_2011_Case_Study.ipynb) |

---

# 6. รายละเอียดแต่ละ Notebook

## Notebook 00 — Download, Setup & Data Audit

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/00_Get_ERA5_Teaching_Data_and_Audit.ipynb)

### วัตถุประสงค์

Notebook แรกใช้เตรียม working environment บน Google Colab และ Google Drive ดาวน์โหลด teaching datasets จาก GitHub และตรวจว่าข้อมูลพร้อมสำหรับบทเรียนถัดไป

นิสิตจะเรียนรู้ว่า ก่อนวิเคราะห์ข้อมูลควรถามก่อนว่า

```text
ข้อมูลมาจากไหน?
มีไฟล์อะไร?
มี dimensions อะไร?
มีตัวแปรอะไร?
ช่วงเวลาใด?
พื้นที่ครอบคลุมเท่าใด?
หน่วยคืออะไร?
```

### แนวคิดสำคัญ

- reproducible data workflow
- file organization
- NetCDF metadata
- provenance
- coordinate range
- temporal coverage
- pressure-level availability
- data audit ก่อนวิเคราะห์

โครงสร้าง working directory:

```text
MyDrive/
└── Teaching_ERA5_Reanalysis_GIS/
    ├── 00_source_github/
    ├── 01_data/
    ├── 02_metadata/
    ├── 03_output/
    ├── 04_figures/
    └── 99_logs/
```

### สิ่งที่ต้องจำ

```text
File downloaded successfully
≠
Scientific data verified
```

---

## Notebook 01 — NetCDF & Xarray Fundamentals

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/01_NetCDF_Xarray_Fundamentals.ipynb)

### วัตถุประสงค์

ทำความเข้าใจโครงสร้าง NetCDF และวิธีใช้ Xarray เพื่อเลือกข้อมูลจาก atmospheric data cube

### แนวคิดสำคัญ

```text
Dataset
DataArray
Dimensions
Coordinates
Variables
Attributes
```

แนวคิด atmospheric cube:

$$
T=T(t,p,\phi,\lambda)
$$

การแปลง Kelvin เป็น Celsius:

$$
T_{^\circ C}=T_K-273.15
$$

### สิ่งที่นิสิตจะทำ

- เปิด NetCDF ด้วย `xarray`
- ตรวจ dimensions/coordinates/variables
- ใช้ `.isel()` และ `.sel()`
- เลือกเวลาและ pressure level
- เลือก grid point ใกล้ตำแหน่งจริง
- สร้างแผนที่ 850-hPa temperature
- เปรียบเทียบ 2–5 October 2011

### ข้อควรจำ

```text
.isel(level=5)
≠
เลือก 500 hPa โดยอัตโนมัติ
```

---

## Notebook 02 — Spatial Subsetting, Cartopy & Map Projections

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/02_Spatial_Subsetting_Cartopy_and_Map_Projections.ipynb)

### วัตถุประสงค์

เชื่อม atmospheric grid เข้ากับแนวคิด GIS และ cartographic visualization

### แนวคิดสำคัญ

```text
Data CRS
≠
Map projection
```

และ

```text
Subsetting
≠
Subsampling
≠
Coarsening
≠
Regridding
```

นิสิตจะเรียนรู้:

- latitude/longitude coordinate ordering
- spatial subset
- map extent
- Plate Carrée
- Mercator
- Lambert Conformal
- `transform=` ใน Cartopy
- grid-center visualization

### หลักการสำคัญ

การเปลี่ยน projection เปลี่ยนวิธีที่ข้อมูลถูก **แสดงบนแผนที่** ไม่ได้เปลี่ยน atmospheric values ของข้อมูลต้นฉบับ

### ข้อควรจำ

```text
Grid spacing
≠
effective physical resolution
```

และ

```text
set_extent()
≠
subsetting the DataArray
```

---

## Notebook 03 — Pressure-Level Atmospheric Structure

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/03_Pressure_Level_Atmospheric_Structure.ipynb)

### วัตถุประสงค์

เรียนรู้ vertical structure ของ atmosphere ผ่าน pressure levels

```text
1000
925
850
700
500
300
250
200 hPa
```

### Hydrostatic relationship

$$
\frac{\partial p}{\partial z}
=
-\rho g
$$

### Geopotential height

ERA5 `z` เป็น geopotential $\Phi$ หน่วย $\mathrm{m^2\,s^{-2}}$

ดังนั้น

$$
Z
=
\frac{\Phi}{g_0}
$$

โดย

$$
g_0=9.80665\ \mathrm{m\,s^{-2}}
$$

### สิ่งที่นิสิตจะวิเคราะห์

- temperature หลาย pressure levels
- relative humidity
- specific humidity
- geopotential height
- vertical profile ที่ตำแหน่งหนึ่ง
- lower / middle / upper troposphere

### ข้อควรจำ

Pressure level ไม่ใช่ geometric height คงที่ทุกพื้นที่ และ 850 hPa อาจอยู่ต่ำกว่าพื้นดินเหนือภูเขาสูง

---

## Notebook 04 — Wind & Vector Analysis

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/04_Wind_and_Vector_Analysis.ipynb)

### วัตถุประสงค์

เข้าใจ wind components และการแสดงลมเป็น vector

### Wind components

```text
u > 0 → eastward component
u < 0 → westward component

v > 0 → northward component
v < 0 → southward component
```

### Wind speed

$$
V
=
\sqrt{u^2+v^2}
$$

### สิ่งที่นิสิตจะทำ

- คำนวณ wind speed
- แสดง `u`, `v`
- สร้าง wind vectors ด้วย `quiver`
- เปรียบเทียบ 850, 500 และ 200 hPa
- ดู vertical change ของ atmospheric flow

### ข้อควรจำ

```text
u ≠ wind speed
v ≠ wind speed
```

vector density บนแผนที่เป็น visual sampling choice ไม่ใช่ observational resolution

---

## Notebook 05 — Synoptic Weather Maps & Atmospheric Dynamics

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/05_Synoptic_Weather_Maps_and_Atmospheric_Dynamics.ipynb)

### วัตถุประสงค์

เริ่มสร้าง weather maps แบบบูรณาการหลายตัวแปร

### Pressure Gradient Force

$$
\vec{F}_{PGF}
=
-\frac{1}{\rho}\nabla p
$$

### Coriolis parameter

$$
f
=
2\Omega\sin\phi
$$

### Relative vorticity

$$
\zeta
=
\frac{\partial v}{\partial x}
-
\frac{\partial u}{\partial y}
$$

Northern Hemisphere:

```text
ζ > 0
→ cyclonic relative rotation
```

### Horizontal divergence

$$
\nabla_h\cdot\vec{V}
=
\frac{\partial u}{\partial x}
+
\frac{\partial v}{\partial y}
$$

### Synoptic framework

```text
850 hPa
→ temperature + RH + height + wind

500 hPa
→ geopotential height + relative vorticity + wind

200 hPa
→ divergence + wind
```

### ข้อควรจำ

```text
positive vorticity
≠ rising motion directly

upper-level divergence
≠ proof of severe convection
```

---

## Notebook 06 — Hovmöller & Spatiotemporal Analysis

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/06_Hovmoller_and_Spatiotemporal_Analysis.ipynb)

### วัตถุประสงค์

เปลี่ยนจากการดูแผนที่ ณ เวลาเดียว ไปสู่การวิเคราะห์ **space × time**

### Map

```text
latitude × longitude
at one time
```

### Time series

```text
time
at one location or region
```

### Hovmöller

```text
space × time
```

### Time–Longitude Hovmöller

$$
\overline{u}(\lambda,t)
=
\frac{
\sum_\phi u(\lambda,\phi,t)w(\phi)
}{
\sum_\phi w(\phi)
}
$$

โดย

$$
w(\phi)=\cos\phi
$$

### Apparent propagation speed

$$
c
\approx
\frac{\Delta x}{\Delta t}
$$

และโดยประมาณ

$$
\Delta x
\approx
111.32\cos\phi\Delta\lambda
$$

### ข้อควรจำ

```text
diagonal Hovmöller feature
≠ proof of atmospheric wave
```

ควรย้อนกลับไปตรวจ maps ทุกครั้ง

---

## Notebook 07 — Climatology & Seasonal Circulation

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/07_Climatology_and_Seasonal_Circulation.ipynb)

### วัตถุประสงค์

เรียนรู้การสร้าง climatology จาก monthly-mean archive

### January climatology

$$
\overline{X}_{Jan}
=
\frac{1}{N_y}
\sum_{y=1981}^{2020}
X_{y,Jan}
$$

### July climatology

$$
\overline{X}_{Jul}
=
\frac{1}{N_y}
\sum_{y=1981}^{2020}
X_{y,Jul}
$$

### Seasonal difference

$$
\Delta X_{Jul-Jan}
=
\overline{X}_{Jul}
-
\overline{X}_{Jan}
$$

### สิ่งที่นิสิตจะวิเคราะห์

- January circulation
- July circulation
- temperature difference
- RH difference
- wind-vector difference
- relative-vorticity difference
- interannual variability

### ข้อควรจำ

```text
monthly mean ≠ climatology
July − January ≠ anomaly
interannual variability ≠ trend
```

---

## Notebook 08 — Anomaly Concepts & Baseline Matching

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/08_Anomaly_Concepts_and_Baseline_Matching.ipynb)

### วัตถุประสงค์

เรียนรู้ว่า anomaly ที่ถูกต้องต้องเริ่มจาก baseline ที่ match กัน

### Absolute anomaly

$$
X'
=
X
-
\overline{X}_{clim}
$$

### Standardized anomaly

$$
Z
=
\frac{X-\mu}{\sigma}
$$

### Wind-vector anomaly

$$
u'
=
u-\overline{u}
$$

$$
v'
=
v-\overline{v}
$$

$$
|\vec{V}'|
=
\sqrt{u'^2+v'^2}
$$

### Baseline matching checklist

ก่อน subtraction ต้องตรวจ:

```text
Variable
+ Calendar month
+ Pressure level
+ Grid
+ Units
+ Temporal semantics
```

Notebook ใช้ January 2011 และ July 2011 เป็นตัวอย่าง เพราะมี matching climatology

### ข้อควรจำ

```text
anomaly ≠ trend
standardized anomaly ≠ statistical significance
large anomaly ≠ extreme automatically
```

และ

```text
No October climatology
→ Do not compute October 2011 climatological anomaly
```

---

## Notebook 09 — Integrated Noul 2011 Case Study

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/09_Integrated_Noul_2011_Case_Study.ipynb)

### วัตถุประสงค์

นำแนวคิดจาก Notebook 00–08 มาสังเคราะห์เป็น integrated atmospheric case study

### Framework

```text
850 hPa
RH + height + wind
        ↓
500 hPa
relative vorticity + height + wind
        ↓
200 hPa
divergence + wind
        ↓
6-hourly u850
Hovmöller + regional time series
        ↓
Integrated interpretation
```

นิสิตจะวิเคราะห์:

- lower-tropospheric evolution
- mid-tropospheric rotational structure
- upper-level divergence
- vertical synthesis
- event-window Hovmöller
- Thailand-area diagnostics
- limitations of the available datasets

### Scientific writing framework

```text
Observation
        ↓
Physical interpretation
        ↓
Limitation
```

### ข้อควรจำ

```text
EVENT = four 00-UTC snapshots
≠ daily means

u850 ≠ total wind speed

positive vorticity ≠ ascent directly

upper-level divergence ≠ severe convection automatically

regional mean ≠ replacement for maps
```

ชื่อไฟล์หรือชื่อ case study ไม่ใช่การยืนยัน storm track โดยอิสระ หากไม่มี track dataset สำหรับ verification

---

# 7. Library ที่ใช้ในหลักสูตร

| Library / Platform | ใช้ทำอะไร | Documentation |
|---|---|---|
| Google Colab | Cloud notebook environment | [Google Colab](https://colab.research.google.com/) |
| Python | ภาษาหลักสำหรับการวิเคราะห์ | [Python](https://www.python.org/) |
| Xarray | วิเคราะห์ multidimensional labeled arrays / NetCDF | [Xarray Documentation](https://docs.xarray.dev/) |
| netCDF4 | อ่าน/เขียน NetCDF | [netCDF4-python](https://unidata.github.io/netcdf4-python/) |
| h5netcdf | NetCDF4/HDF5 backend | [h5netcdf](https://h5netcdf.org/) |
| Dask | lazy / chunked array computation | [Dask](https://docs.dask.org/) |
| NumPy | numerical array computation | [NumPy](https://numpy.org/doc/) |
| Pandas | ตาราง, time index และ CSV | [Pandas](https://pandas.pydata.org/docs/) |
| Matplotlib | visualization | [Matplotlib](https://matplotlib.org/stable/) |
| Cartopy | geographic maps and projections | [Cartopy](https://cartopy.readthedocs.io/) |

---

# 8. แนวทางการอ่าน Figure

Figures ใน Notebook ใช้ข้อความภาษาอังกฤษเพื่อให้เหมาะกับ scientific visualization และหลีกเลี่ยงปัญหา font compatibility

แนวทางการอ่าน figure:

```text
1. อ่าน variable
2. อ่าน pressure level
3. อ่าน date/time
4. อ่าน units
5. ดู spatial pattern
6. ดู vector/contour relationship
7. เปรียบเทียบกับ pressure level หรือเวลาอื่น
8. สรุปเฉพาะสิ่งที่ข้อมูลสนับสนุน
```

โดยทั่วไป figures ถูกออกแบบให้ export ที่ประมาณ 300 dpi เพื่อใช้ฝึก workflow ที่ใกล้เคียงงานวิจัย

---

# 9. หลักการสำคัญที่ควรจำตลอดรายวิชา

```text
Reanalysis ≠ observation
Reanalysis ≠ forecast

data CRS ≠ map projection

grid spacing ≠ physical resolution

u/v components ≠ wind speed

ERA5 z ≠ geopotential height
until divided by g₀

positive vorticity ≠ ascent directly

upper-level divergence ≠ vertical velocity

high RH ≠ rainfall automatically

Hovmöller ≠ full spatial field

climatology ≠ trend

seasonal difference ≠ anomaly

anomaly ≠ statistical significance

00-UTC snapshot ≠ daily mean
```

---

# 10. Suggested Learning Workflow

สำหรับแต่ละ Notebook แนะนำให้นิสิตทำตามลำดับ:

```text
1. อ่าน Learning Objectives
2. อ่าน Theory / Concept
3. อ่านสมการและความหมายของตัวแปร
4. ตรวจ input data
5. Run code ทีละ cell
6. อ่าน figure ก่อนอ่านคำอธิบาย
7. เขียน Observation ของตนเอง
8. เขียน Physical interpretation
9. ระบุ Limitation
10. ทำ Exercise
```

ไม่แนะนำให้เลือก `Runtime → Run all` ตั้งแต่ครั้งแรกโดยไม่อ่าน Markdown เพราะเป้าหมายของ repository นี้คือการเข้าใจ **เหตุผลของการวิเคราะห์** ไม่ใช่เพียงสร้างรูปให้สำเร็จ

---

# 11. จากการเรียนสู่ Mini Research Project

หลังจบ Notebook 09 นิสิตควรสามารถตั้ง workflow ของตนเองได้

```text
Research question
        ↓
Select ERA5 variables
        ↓
Choose pressure level / time period
        ↓
Audit metadata
        ↓
Spatial subset
        ↓
Build maps / time series / Hovmöller
        ↓
Compare atmospheric levels
        ↓
Interpret physical mechanisms
        ↓
State limitations
        ↓
Export reproducible figures and tables
```

ตัวอย่างคำถามที่สามารถพัฒนาต่อ:

- circulation ก่อนและหลัง heavy-rain event แตกต่างกันอย่างไร?
- low-level moisture transport เปลี่ยนตามฤดูกาลอย่างไร?
- 500-hPa vorticity pattern เชื่อมกับ synoptic evolution อย่างไร?
- upper-level wind และ divergence เปลี่ยนอย่างไรระหว่างเหตุการณ์?
- monsoon circulation ใน January และ July แตกต่างกันอย่างไร?
- atmospheric anomaly ของปีหนึ่งแตกต่างจาก climatological baseline มากเพียงใด?

---

# 12. Data and Scientific Caveats

### 12.1 0.5° teaching grid

Teaching datasets ถูกลดขนาดเพื่อให้ใช้งานง่ายใน Colab

```text
0.5° grid spacing
≠ claim of 0.5° effective atmospheric resolution
```

### 12.2 850 hPa over high terrain

850-hPa pressure surface อาจอยู่ต่ำกว่าพื้นดินเหนือภูมิประเทศสูง เช่น Tibetan Plateau และ Himalayas

prepared dataset ไม่มี surface pressure สำหรับทำ robust below-ground mask

### 12.3 EVENT temporal sampling

EVENT dataset มีเฉพาะ 00 UTC ของ 2–5 October 2011 จึงไม่สามารถใช้แทน full diurnal evolution ได้

### 12.4 October climatology

Monthly baseline มีเฉพาะ January และ July จึงไม่มี scientific basis สำหรับสร้าง October 2011 monthly anomaly จาก dataset นี้

---

# 13. Repository Structure

```text
Teaching_ERA5_Reanalysis_GIS/
│
├── 00_prepared_data/
│   ├── era5_noul_20111002_05_SEAsia_pressure_levels_0.5deg.nc
│   ├── era5_u850_6hourly_20110901_20111031_Asia_0.5deg.nc
│   └── era5_850hPa_JanJul_monthlymeans_1981_2020_Asia_0.5deg.nc
│
├── 00A_Prepare_ERA5_Teaching_Datasets.ipynb
├── 00_Get_ERA5_Teaching_Data_and_Audit.ipynb
├── 01_NetCDF_Xarray_Fundamentals.ipynb
├── 02_Spatial_Subsetting_Cartopy_and_Map_Projections.ipynb
├── 03_Pressure_Level_Atmospheric_Structure.ipynb
├── 04_Wind_and_Vector_Analysis.ipynb
├── 05_Synoptic_Weather_Maps_and_Atmospheric_Dynamics.ipynb
├── 06_Hovmoller_and_Spatiotemporal_Analysis.ipynb
├── 07_Climatology_and_Seasonal_Circulation.ipynb
├── 08_Anomaly_Concepts_and_Baseline_Matching.ipynb
├── 09_Integrated_Noul_2011_Case_Study.ipynb
└── README.md
```

---

# 14. Recommended Citation for ERA5

Hersbach, H., Bell, B., Berrisford, P., et al. (2020). The ERA5 global reanalysis. *Quarterly Journal of the Royal Meteorological Society, 146*(730), 1999–2049. https://doi.org/10.1002/qj.3803

---

# 15. Final Learning Message

เมื่อเรียนครบ repository นี้ สิ่งที่สำคัญที่สุดไม่ใช่การจำ syntax ของ Python แต่คือการตอบคำถามต่อไปนี้ให้ได้:

```text
ข้อมูลนี้คืออะไร?
มาจากไหน?
มีข้อจำกัดอะไร?
ตัวแปรนี้มีความหมายทางฟิสิกส์อย่างไร?
เหตุใดจึงเลือก pressure level นี้?
เหตุใดจึงเลือกแผนที่หรือ Hovmöller?
สิ่งที่เห็นใน figure สนับสนุนข้อสรุปอะไร?
และมีอะไรที่ยังสรุปไม่ได้?
```

> **Good atmospheric analysis = correct data handling + physical understanding + spatial thinking + cautious interpretation + reproducibility**

---

## Instructor / Repository

**Teaching ERA5 Reanalysis & GIS**

GitHub: [https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS](https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS)
