# Teaching ERA5 Reanalysis & GIS

## คู่มือก่อนเรียน: จาก ERA5 Reanalysis → Python/Xarray → Weather Maps → Climatology → Anomaly → Case Study

Repository นี้เป็นชุดการเรียนการสอนสำหรับนิสิตด้าน **วิทยาศาสตร์สิ่งแวดล้อม ภูมิศาสตร์ ภูมิสารสนเทศ และอุตุนิยมวิทยาประยุกต์** โดยใช้ข้อมูล **ERA5 reanalysis** และ Python บน Google Colab

เป้าหมายไม่ใช่เพียงให้ “รันโค้ดได้” แต่ให้นิสิตเข้าใจว่า

```text
ข้อมูลคืออะไร
→ ตัวแปรมีความหมายทางกายภาพอย่างไร
→ สมการคำนวณอะไร
→ code ทำขั้นตอนใด
→ figure แสดงอะไร
→ ตีความได้แค่ไหน
→ มีข้อจำกัดอะไร
```

> **แนวคิดหลักของรายวิชา: Data → Calculation → Map → Physical interpretation → Limitation**

---

# 1. เริ่มจากลิงก์สำคัญ

## 1.1 Repository และ Google Colab

- GitHub repository: [Teaching_ERA5_Reanalysis_GIS](https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS)
- Google Colab: [https://colab.research.google.com/](https://colab.research.google.com/)

## 1.2 แหล่งข้อมูล ERA5

ข้อมูลที่ใช้ในรายวิชามาจาก **ERA5 reanalysis** ของ ECMWF / Copernicus Climate Change Service และถูกเตรียมเป็นไฟล์ขนาดเล็กสำหรับการเรียน

- [ECMWF ERA5 Documentation](https://confluence.ecmwf.int/display/CKB/ERA5%3A+data+documentation)
- [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/)
- [ERA5 overview — ECMWF](https://www.ecmwf.int/en/forecasts/dataset/ecmwf-reanalysis-v5)

บทความอ้างอิงหลัก:

Hersbach, H., Bell, B., Berrisford, P., et al. (2020). The ERA5 global reanalysis. *Quarterly Journal of the Royal Meteorological Society, 146*(730), 1999–2049. https://doi.org/10.1002/qj.3803

## 1.3 Library ที่ใช้

| Library / Platform | หน้าที่ในรายวิชา | Documentation |
|---|---|---|
| **Google Colab** | รัน Jupyter Notebook บน cloud | [Colab](https://colab.research.google.com/) |
| **Python** | ภาษาหลักในการวิเคราะห์ | [Python](https://docs.python.org/3/) |
| **Xarray** | เปิดและวิเคราะห์ NetCDF / multidimensional atmospheric data | [Xarray](https://docs.xarray.dev/) |
| **netCDF4** | backend สำหรับอ่าน/เขียน NetCDF | [netCDF4-python](https://unidata.github.io/netcdf4-python/) |
| **h5netcdf** | NetCDF/HDF5 backend | [h5netcdf](https://h5netcdf.org/) |
| **Dask** | ช่วยจัดการ array ขนาดใหญ่แบบ lazy/chunked | [Dask](https://docs.dask.org/) |
| **NumPy** | คำนวณ array และสมการเชิงตัวเลข | [NumPy](https://numpy.org/doc/) |
| **Pandas** | ตาราง, วันที่/เวลา, CSV | [Pandas](https://pandas.pydata.org/docs/) |
| **Matplotlib** | plot graph และ scientific figure | [Matplotlib](https://matplotlib.org/stable/) |
| **Cartopy** | แผนที่, projection, coastline, coordinate transform | [Cartopy](https://cartopy.readthedocs.io/) |

---

# 2. ERA5 และ Reanalysis คืออะไร

ERA5 ไม่ใช่สถานีตรวจอากาศโดยตรง และไม่ใช่ weather forecast

แนวคิดแบบง่าย:

```text
ข้อมูลตรวจวัดจริง
      +
แบบจำลองบรรยากาศ
      +
Data assimilation
      ↓
Reanalysis
```

ดังนั้น

```text
Reanalysis ≠ Observation
Reanalysis ≠ Forecast
```

ข้อดีของ reanalysis คือได้ข้อมูลบรรยากาศที่มี spatial grid และ time step สม่ำเสมอ เหมาะกับการวิเคราะห์ย้อนหลัง

---

# 3. Atmospheric Data Cube

ข้อมูลบรรยากาศใน ERA5 สามารถคิดเป็น “กล่องข้อมูลหลายมิติ”

```math
X = X(t,p,\phi,\lambda)
```

อ่านว่า:

```text
ตัวแปร X ขึ้นกับ
t = เวลา
p = ระดับความกดอากาศ
φ = ละติจูด
λ = ลองจิจูด
```

ตัวอย่าง:

```text
Temperature
→ เลือกวันที่
→ เลือก 850 hPa
→ เหลือแผนที่ latitude × longitude
```

นี่คือเหตุผลที่ Xarray เหมาะกับข้อมูล ERA5 เพราะเราสามารถเลือกข้อมูลตาม “ชื่อ coordinate” ได้โดยตรง

---

# 4. Teaching Datasets ที่ใช้

ไฟล์ต้นฉบับ ERA5 มีขนาดใหญ่ จึงเตรียมเป็น compact datasets เพื่อให้นิสิตใช้งานบน Colab ได้เร็วขึ้น

Folder บน GitHub:

```text
00_prepared_data/
```

## 4.1 EVENT: Pressure-Level Data

```text
era5_noul_20111002_05_SEAsia_pressure_levels_0.5deg.nc
```

ครอบคลุม:

```text
2–5 October 2011
00 UTC ของแต่ละวัน
```

มี 4 time steps เท่านั้น

> **สำคัญ: เป็น 00-UTC synoptic snapshots ไม่ใช่ daily mean**

Pressure levels:

```text
1000, 925, 850, 700, 500, 300, 250, 200 hPa
```

Variables:

| ชื่อ | ความหมาย |
|---|---|
| `t` | Temperature |
| `r` | Relative humidity |
| `q` | Specific humidity |
| `z` | Geopotential |
| `u` | Zonal wind component |
| `v` | Meridional wind component |
| `vo` | Relative vorticity |
| `d` | Horizontal divergence |

ใช้ใน Notebook 01–05 และ 09

---

## 4.2 SPATIOTEMPORAL: 850-hPa Zonal Wind

```text
era5_u850_6hourly_20110901_20111031_Asia_0.5deg.nc
```

ช่วงเวลา:

```text
1 September–31 October 2011
00, 06, 12, 18 UTC
```

ดังนั้น time interval คือ

```text
6 ชั่วโมง
```

มี variable เดียว:

```text
u = zonal wind component
```

การอ่านเครื่องหมาย:

```text
u > 0 → ลมองค์ประกอบไปทางตะวันออก
u < 0 → ลมองค์ประกอบไปทางตะวันตก
```

> `u` ไม่ใช่ wind speed

ใช้ใน Notebook 06 และ 09

---

## 4.3 MONTHLY BASELINE: January/July 1981–2020

```text
era5_850hPa_JanJul_monthlymeans_1981_2020_Asia_0.5deg.nc
```

ประกอบด้วย:

```text
January 1981–2020
July 1981–2020
```

แต่ละ time step เป็น **monthly mean**

ไฟล์นี้ยังไม่ใช่ climatology โดยตัวมันเอง

```text
monthly mean archive
        ↓ average same month across years
climatology
```

> ไม่มี October ใน dataset นี้ จึงไม่มี October climatology สำหรับคำนวณ October 2011 anomaly

ใช้ใน Notebook 07–08

---

# 5. ลำดับการเรียน

นิสิตควรเรียนตามลำดับ

```text
00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09
```

`00A` เป็น Notebook สำหรับผู้สอน ใช้เตรียม teaching datasets จาก ERA5 source files

## Open directly in Google Colab

| Notebook | เนื้อหา | Open in Colab |
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

# 6. วิธีอ่านสมการใน README นี้

เพื่อให้ GitHub แสดงผลได้สม่ำเสมอ README นี้ใช้ **GitHub math block**

ตัวอย่าง:

```math
V = \sqrt{u^2+v^2}
```

และจะมีคำอธิบายแบบข้อความกำกับเสมอ:

```text
Wind speed = square root of (u² + v²)
```

ดังนั้น แม้พื้นฐานคณิตศาสตร์ยังไม่มาก ให้เริ่มจาก “อ่านข้อความ” ก่อน แล้วค่อยดูสมการ

---

# 7. Notebook 00 — Download, Setup & Data Audit

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/00_Get_ERA5_Teaching_Data_and_Audit.ipynb)

## วัตถุประสงค์

เตรียม Colab, mount Google Drive, ดาวน์โหลดข้อมูลจาก GitHub และตรวจว่าไฟล์พร้อมสำหรับการเรียน

## แนวคิด

ก่อนวิเคราะห์ต้องตอบให้ได้ว่า:

```text
ไฟล์อยู่ที่ไหน?
ชื่อไฟล์อะไร?
ขนาดเท่าใด?
มี dimensions อะไร?
มี coordinates อะไร?
มี variables อะไร?
ช่วงเวลาอะไร?
```

## Code ทำอะไร

### 1. Mount Google Drive

```python
from google.colab import drive
drive.mount("/content/drive")
```

ความหมาย:

```text
เชื่อม Colab กับ Google Drive
เพื่อบันทึกไฟล์และรูปไม่ให้หายเมื่อ runtime ปิด
```

### 2. สร้าง folder

```python
Path(...).mkdir(parents=True, exist_ok=True)
```

ใช้จัด workspace ให้ reproducible

### 3. ดาวน์โหลดไฟล์

ใช้ URL ของ GitHub `raw` แล้วบันทึกลง `00_source_github`

### 4. เปิด NetCDF

```python
ds = xr.open_dataset(file)
```

### 5. Audit

ตรวจ:

```python
ds.dims
ds.coords
ds.data_vars
ds.attrs
```

## สิ่งที่ต้องเข้าใจ

```text
ดาวน์โหลดสำเร็จ
≠
ข้อมูลถูกต้องทางวิทยาศาสตร์
```

ต้อง audit metadata ทุกครั้ง

---

# 8. Notebook 01 — NetCDF & Xarray Fundamentals

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/01_NetCDF_Xarray_Fundamentals.ipynb)

## วัตถุประสงค์

เข้าใจ `Dataset`, `DataArray`, dimension, coordinate และการเลือกข้อมูลด้วย Xarray

## สมการพื้นฐาน

Atmospheric cube:

```math
T=T(t,p,\phi,\lambda)
```

อ่านแบบง่าย:

```text
Temperature เปลี่ยนตาม
เวลา + pressure level + latitude + longitude
```

### Kelvin → Celsius

```math
T_{^\circ C}=T_K-273.15
```

ตัวอย่าง:

```text
T = 300 K
T = 300 − 273.15
T = 26.85 °C
```

## Code สำคัญ

### เปิดไฟล์

```python
ds = xr.open_dataset(EVENT_FILE)
```

### เลือกด้วยตำแหน่ง index

```python
ds["t"].isel(time=0)
```

`isel` = index selection

### เลือกด้วยค่าจริงของ coordinate

```python
ds["t"].sel(
    time="2011-10-02",
    level=850,
)
```

`sel` = label/coordinate selection

ควรใช้ `sel(level=850)` เมื่อเราต้องการความหมายทางกายภาพ 850 hPa

### เลือก grid point ใกล้ตำแหน่ง

```python
.sel(
    latitude=13.75,
    longitude=100.50,
    method="nearest",
)
```

หมายถึงเลือก **grid center ที่ใกล้ที่สุด**

ไม่ใช่ค่าจากสถานีกรุงเทพโดยตรง

## สิ่งที่ต้องจำ

```text
nearest grid ≠ station observation
```

---

# 9. Notebook 02 — Spatial Subsetting, Cartopy & Map Projections

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/02_Spatial_Subsetting_Cartopy_and_Map_Projections.ipynb)

## วัตถุประสงค์

เข้าใจความสัมพันธ์ระหว่าง atmospheric grid, latitude/longitude และ map projection

## แนวคิดสำคัญ

```text
Data CRS
≠
Map projection
```

ข้อมูล ERA5 อยู่บน latitude/longitude grid

Cartopy สามารถนำข้อมูลเดียวกันไปแสดงด้วยหลาย projection เช่น:

```text
PlateCarree
Mercator
LambertConformal
```

## Code สำคัญ

### Spatial subset

```python
subset = field.sel(
    latitude=slice(...),
    longitude=slice(...),
)
```

หมายถึง “ตัดข้อมูลจริงให้เหลือพื้นที่ที่สนใจ”

### Map extent

```python
ax.set_extent(
    [90, 115, 0, 30],
    crs=ccrs.PlateCarree(),
)
```

หมายถึง “เปลี่ยนบริเวณที่แสดงบนแผนที่”

ไม่ได้ลดขนาด DataArray

### Data transform

```python
transform=ccrs.PlateCarree()
```

บอก Cartopy ว่า coordinate ต้นทางของข้อมูลคือ longitude/latitude

## แยกคำศัพท์

```text
Subsetting
= ตัดพื้นที่ข้อมูล

Subsampling
= เลือกบาง grid point

Coarsening
= รวมหลาย grid cells เป็น cell ที่หยาบขึ้น

Regridding
= คำนวณข้อมูลไปยัง grid ใหม่
```

> Grid spacing ไม่เท่ากับ effective physical resolution

---

# 10. Notebook 03 — Pressure-Level Atmospheric Structure

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/03_Pressure_Level_Atmospheric_Structure.ipynb)

## วัตถุประสงค์

เข้าใจบรรยากาศในแนวดิ่งจาก pressure levels

## Pressure levels

```text
1000 / 925 hPa → lower troposphere
850 hPa        → lower troposphere
700 hPa        → lower-middle
500 hPa        → middle troposphere
300–200 hPa    → upper troposphere
```

ความสูงจริงของ pressure surface เปลี่ยนตามสภาพบรรยากาศ จึงไม่ใช่ geometric height คงที่

## Hydrostatic balance

```math
\frac{\partial p}{\partial z}=-\rho g
```

อ่านแบบง่าย:

```text
เมื่อสูงขึ้น ความกดอากาศลดลง
เพราะน้ำหนักของอากาศที่อยู่ด้านบนลดลง
```

สัญลักษณ์:

| Symbol | ความหมาย |
|---|---|
| `p` | pressure |
| `z` | height |
| `ρ` | air density |
| `g` | gravitational acceleration |

## Geopotential → Geopotential Height

ERA5 variable `z` เป็น geopotential ไม่ใช่ metre

```math
Z=\frac{\Phi}{g_0}
```

โดย

```math
g_0=9.80665\ \mathrm{m\,s^{-2}}
```

ตัวอย่าง:

```text
ถ้า geopotential = 14,700 m² s⁻²

Z = 14,700 / 9.80665
≈ 1,499 m
```

## Code สำคัญ

```python
height = ds["z"] / 9.80665
```

### เลือกหลายระดับ

```python
for level in [1000, 850, 700, 500, 300, 200]:
    field = ds["t"].sel(level=level)
```

## สิ่งที่ต้องจำ

> 850 hPa อาจอยู่ต่ำกว่าพื้นดินเหนือพื้นที่ภูเขาสูง จึงต้องตีความอย่างระมัดระวัง

---

# 11. Notebook 04 — Wind & Vector Analysis

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/04_Wind_and_Vector_Analysis.ipynb)

## วัตถุประสงค์

เข้าใจองค์ประกอบของลมและการสร้าง vector map

## Wind components

```text
u = east–west component
v = north–south component
```

เครื่องหมาย:

```text
u > 0 → eastward
u < 0 → westward

v > 0 → northward
v < 0 → southward
```

## Wind speed

```math
V=\sqrt{u^2+v^2}
```

อ่านแบบง่าย:

```text
Wind speed
= √(u² + v²)
```

ตัวอย่าง:

```text
u = 3 m/s
v = 4 m/s

V = √(3² + 4²)
  = √25
  = 5 m/s
```

## Code

```python
speed = np.hypot(u, v)
```

เทียบเท่ากับ

```python
speed = np.sqrt(u**2 + v**2)
```

### วาด vector

```python
ax.quiver(
    lon,
    lat,
    u,
    v,
)
```

`quiver` ใช้ทิศและขนาดจาก `u` และ `v`

## Vector skip

```python
u.values[::4, ::4]
```

แปลว่าแสดงลูกศรทุก 4 grid cells เพื่อไม่ให้รูปแน่นเกินไป

> นี่เป็น visualization choice ไม่ใช่การเปลี่ยน resolution ของ ERA5

---

# 12. Notebook 05 — Synoptic Weather Maps & Atmospheric Dynamics

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/05_Synoptic_Weather_Maps_and_Atmospheric_Dynamics.ipynb)

## วัตถุประสงค์

รวมหลาย atmospheric fields เพื่ออ่าน synoptic weather map

## 12.1 Pressure Gradient Force

```math
\vec{F}_{PGF}
=
-\frac{1}{\rho}\nabla p
```

อ่านแบบง่าย:

```text
อากาศมีแนวโน้มถูกเร่งจากบริเวณ pressure สูง
ไปยัง pressure ต่ำ
```

## 12.2 Coriolis parameter

```math
f=2\Omega\sin\phi
```

หมายถึงผลของการหมุนโลกขึ้นกับ latitude

## 12.3 Relative Vorticity

```math
\zeta
=
\frac{\partial v}{\partial x}
-
\frac{\partial u}{\partial y}
```

อ่านแบบง่าย:

```text
ζ ใช้อธิบาย local rotation ของ horizontal flow
```

ใน Northern Hemisphere:

```text
ζ > 0 → cyclonic relative rotation
ζ < 0 → anticyclonic relative rotation
```

> Positive vorticity ไม่ได้แปลว่าอากาศลอยตัวขึ้นโดยตรง

## 12.4 Horizontal Divergence

```math
D
=
\frac{\partial u}{\partial x}
+
\frac{\partial v}{\partial y}
```

อ่านแบบง่าย:

```text
D > 0 → horizontal flow แผ่ออกจากกัน
D < 0 → horizontal flow มารวมกัน
```

> 200-hPa divergence ไม่ใช่ vertical velocity

## Code สำคัญ

### แปลง vorticity เพื่ออ่าน colorbar ง่าย

```python
vort_scaled = vort_500 * 1.0e5
```

ถ้าค่าจริงเป็น

```text
0.00002 s⁻¹
```

เมื่อคูณ `10⁵` จะได้

```text
2
```

ดังนั้น colorbar ใช้หน่วย

```text
10⁻⁵ s⁻¹
```

### Overlay หลาย field

```python
pcolormesh(...)   # shading
contour(...)      # geopotential height
quiver(...)       # wind
```

นี่คือพื้นฐานของ integrated synoptic map

---

# 13. Notebook 06 — Hovmöller & Spatiotemporal Analysis

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/06_Hovmoller_and_Spatiotemporal_Analysis.ipynb)

## วัตถุประสงค์

เรียนรู้การเปลี่ยนจากแผนที่ `space × space` ไปสู่ `space × time`

## เปรียบเทียบ

```text
Map
= latitude × longitude ที่เวลาเดียว

Time series
= time ที่ตำแหน่ง/พื้นที่เดียว

Hovmöller
= space × time
```

## Latitude-weighted Time–Longitude Hovmöller

```math
\overline{u}(\lambda,t)
=
\frac{
\sum_{\phi}u(\lambda,\phi,t)w(\phi)
}{
\sum_{\phi}w(\phi)
}
```

โดย

```math
w(\phi)=\cos\phi
```

อ่านแบบง่าย:

```text
1. เลือก latitude band
2. ให้แต่ละ latitude มีน้ำหนัก cos(latitude)
3. average latitude ออกไป
4. เหลือ longitude × time
```

## ทำไมต้อง cos(latitude)?

บน latitude–longitude grid พื้นที่ของ grid cell ลดลงเมื่อ latitude สูงขึ้น

โดยประมาณ:

```math
A\propto\cos\phi
```

## Apparent propagation speed

```math
c\approx\frac{\Delta x}{\Delta t}
```

อ่านง่าย:

```text
speed = distance / time
```

ระยะตาม longitude โดยประมาณ:

```math
\Delta x
\approx
111.32\cos\phi\,\Delta\lambda
```

ตัวอย่างง่าย:

```text
feature เปลี่ยนจาก 100°E → 105°E
ที่ latitude 15°N
ใช้เวลา 24 ชั่วโมง

distance ≈ 111.32 × cos(15°) × 5
≈ 538 km

apparent speed ≈ 538 / 24
≈ 22.4 km/h
```

> เรียกว่า apparent propagation เพราะ Hovmöller pattern ไม่จำเป็นต้องเป็นวัตถุเดียวที่เคลื่อนจริง

## Code

```python
weights = np.cos(np.deg2rad(latitude))
field.weighted(weights).mean("latitude")
```

---

# 14. Notebook 07 — Climatology & Seasonal Circulation

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/07_Climatology_and_Seasonal_Circulation.ipynb)

## วัตถุประสงค์

สร้าง climatology จาก monthly means หลายปี

## Monthly mean vs Climatology

```text
January 2011 monthly mean
= ค่าเฉลี่ยของ January 2011

January climatology
= ค่าเฉลี่ย January ของหลายปี
```

## January climatology

```math
\overline{X}_{Jan}
=
\frac{1}{N_y}
\sum_{y=1981}^{2020}X_{y,Jan}
```

อ่านแบบง่าย:

```text
เอา January ของทุกปี 1981–2020
มาบวกกัน
แล้วหารด้วยจำนวนปี
```

ถ้ามี 40 ปี:

```text
N_y = 40
```

## ตัวอย่างง่าย

สมมติ January temperature 3 ปี:

```text
Year 1 = 20 °C
Year 2 = 22 °C
Year 3 = 21 °C
```

Climatological mean:

```text
(20 + 22 + 21) / 3
= 21 °C
```

## July − January seasonal difference

```math
\Delta X_{Jul-Jan}
=
\overline{X}_{Jul}
-
\overline{X}_{Jan}
```

ตัวอย่าง:

```text
July climatology = 24 °C
January climatology = 20 °C

July − January
= 24 − 20
= +4 °C
```

หมายความว่า July climatological value สูงกว่า January 4 °C

> นี่คือ seasonal difference ไม่ใช่ anomaly

## Code สำคัญ

### เลือกเดือนด้วย datetime coordinate

```python
jan = ds.sel(
    time=(ds["time"].dt.month == 1)
)
```

### average ข้าม time

```python
JAN_CLIM = jan.mean(dim="time")
```

## Interannual variability

นิสิตจะเห็นว่าแม้ climatology เป็นค่าเฉลี่ย แต่แต่ละปียังแตกต่างกัน

```text
climatology ≠ ทุกปีเหมือนกัน
interannual variability ≠ trend
```

---

# 15. Notebook 08 — Anomaly Concepts & Baseline Matching

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/08_Anomaly_Concepts_and_Baseline_Matching.ipynb)

## วัตถุประสงค์

เรียนรู้ anomaly และตรวจว่า baseline match ก่อนลบ

## 15.1 Absolute anomaly

```math
X'
=
X-\overline{X}_{clim}
```

อ่านแบบง่าย:

```text
Anomaly
= Target value − Climatological mean
```

ตัวอย่าง:

```text
July 2011 temperature = 25 °C
July climatology      = 23 °C

Anomaly
= 25 − 23
= +2 °C
```

แปลว่า July 2011 สูงกว่า climatological July 2 °C

## 15.2 Standardized anomaly

```math
Z
=
\frac{X-\mu}{\sigma}
```

อ่านแบบง่าย:

```text
Standardized anomaly
= anomaly / historical standard deviation
```

ตัวอย่าง:

```text
Target = 25 °C
Climatological mean = 23 °C
Standard deviation = 1 °C

Z = (25 − 23) / 1
  = +2
```

แปลว่า target สูงกว่าค่าเฉลี่ยประมาณ 2 standard deviations

> Z = 2 ไม่ได้แปลว่า “มีนัยสำคัญทางสถิติ” โดยอัตโนมัติ

## 15.3 Wind-component anomaly

```math
u'
=
u-\overline{u}
```

```math
v'
=
v-\overline{v}
```

Vector-anomaly magnitude:

```math
|\vec{V}'|
=
\sqrt{u'^2+v'^2}
```

ตัวอย่าง:

```text
u' = 3 m/s
v' = 4 m/s

|V'| = √(3² + 4²)
     = 5 m/s
```

## Baseline matching

ก่อน subtraction ต้องถาม:

```text
Variable ตรงกันหรือไม่?
Month ตรงกันหรือไม่?
Pressure level ตรงกันหรือไม่?
Grid ตรงกันหรือไม่?
Units ตรงกันหรือไม่?
Temporal meaning ตรงกันหรือไม่?
```

ตัวอย่างที่ถูก:

```text
July 2011 monthly-mean T850
−
July 1981–2020 climatological T850
```

ตัวอย่างที่ไม่ถูก:

```text
2 October 2011 00-UTC snapshot
−
January climatology
```

## Code

### สร้าง climatology

```python
JUL_MEAN = jul_archive.mean(dim="time")
```

### anomaly

```python
JUL_ANOM = JUL_2011 - JUL_MEAN
```

### standardized anomaly

```python
JUL_T_Z = JUL_ANOM["t"] / JUL_STD["t"]
```

### Zero-centered color scale

```python
TwoSlopeNorm(
    vmin=-absmax,
    vcenter=0,
    vmax=absmax,
)
```

ทำให้

```text
0 anomaly
```

อยู่ตรงกลาง color scale

## สิ่งที่ต้องจำ

```text
anomaly ≠ trend
anomaly ≠ extreme automatically
standardized anomaly ≠ statistical significance
```

และ

```text
ไม่มี October climatology
→ ไม่คำนวณ October 2011 climatological anomaly
```

---

# 16. Notebook 09 — Integrated Noul 2011 Case Study

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nattaponm/Teaching_ERA5_Reanalysis_GIS/blob/main/09_Integrated_Noul_2011_Case_Study.ipynb)

## วัตถุประสงค์

รวมแนวคิดทั้งหมดเข้าสู่ mini atmospheric case study

## Integrated framework

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

## Code ทำอะไร

### 1. Loop ข้าม 4 event snapshots

```python
for time_value in event_times:
    ...
```

ใช้สร้าง maps วันที่ 2, 3, 4 และ 5 October 2011 เวลา 00 UTC

### 2. 850 hPa

```python
r + z + u + v
```

ใช้ดู:

```text
moisture
lower-level circulation
geopotential structure
```

### 3. 500 hPa

```python
vo + z + u + v
```

ใช้ดู mid-tropospheric rotational structure

### 4. 200 hPa

```python
d + u + v
```

ใช้ดู upper-level divergence และ flow

### 5. Hovmöller

ใช้ข้อมูล 6-hourly `u850` เพื่อเพิ่ม temporal context ที่ละเอียดกว่า EVENT dataset

### 6. Regional diagnostics

คำนวณค่าเฉลี่ยเหนือ Thailand box สำหรับ:

```text
T850
RH850
wind850
ζ500
D200
```

## วิธีเขียนผล

ใช้ framework:

```text
Observation
→ Physical interpretation
→ Limitation
```

ตัวอย่าง:

```text
Observation:
RH850 is relatively high over ...

Physical interpretation:
This indicates a relatively moist lower-tropospheric environment.

Limitation:
High RH alone does not demonstrate rainfall or deep convection.
```

---

# 17. สรุปสมการที่นิสิตควรรู้

## Temperature conversion

```math
T_{^\circ C}=T_K-273.15
```

```text
300 K → 26.85 °C
```

## Geopotential height

```math
Z=\frac{\Phi}{g_0}
```

## Wind speed

```math
V=\sqrt{u^2+v^2}
```

## Relative vorticity

```math
\zeta
=
\frac{\partial v}{\partial x}
-
\frac{\partial u}{\partial y}
```

## Horizontal divergence

```math
D
=
\frac{\partial u}{\partial x}
+
\frac{\partial v}{\partial y}
```

## Latitude weighting

```math
w(\phi)=\cos\phi
```

## Climatological mean

```math
\overline{X}
=
\frac{1}{N}
\sum_{i=1}^{N}X_i
```

## Anomaly

```math
X'=X-\overline{X}_{clim}
```

## Standardized anomaly

```math
Z=\frac{X-\mu}{\sigma}
```

## Apparent propagation speed

```math
c\approx\frac{\Delta x}{\Delta t}
```

---

# 18. วิธีคิดเลขสำหรับนิสิตที่ยังไม่ถนัดคณิตศาสตร์

หลักสูตรนี้ไม่ได้ต้องการให้ท่องสมการ แต่ต้องการให้เข้าใจ “สิ่งที่สมการทำ”

| สมการ | อ่านแบบง่าย |
|---|---|
| `T_C = T_K − 273.15` | เปลี่ยน Kelvin เป็น Celsius |
| `V = √(u²+v²)` | รวมลมสองแกนให้เป็นความเร็วลม |
| `Z = Φ/g` | เปลี่ยน geopotential เป็น height |
| `mean = sum/N` | บวกทุกค่าแล้วหารจำนวนข้อมูล |
| `anomaly = target − mean` | เปรียบเทียบปีที่สนใจกับค่าปกติ |
| `z-score = anomaly/SD` | ดูว่า anomaly ใหญ่เมื่อเทียบ variability แค่ไหน |
| `speed = distance/time` | ประมาณความเร็วการเคลื่อน |

เมื่อเจอสมการ ให้ถาม 3 ข้อ:

```text
1. Input คืออะไร?
2. คำนวณอะไร?
3. Output มีหน่วยอะไร?
```

---

# 19. วิธีอ่าน Code โดยไม่ต้องเก่ง Python มาก

ให้แบ่ง code เป็น 6 กลุ่ม

## 1. Import

```python
import xarray as xr
import numpy as np
```

แปลว่าเรียก library มาใช้

## 2. Read

```python
ds = xr.open_dataset(...)
```

แปลว่าเปิดข้อมูล

## 3. Select

```python
.sel(...)
.isel(...)
```

แปลว่าเลือกเวลา ระดับ หรือพื้นที่

## 4. Calculate

```python
np.hypot(...)
.mean(...)
.std(...)
.weighted(...)
```

แปลว่าคำนวณค่าทางสถิติหรือฟิสิกส์

## 5. Plot

```python
pcolormesh(...)
contour(...)
quiver(...)
```

แปลว่าสร้าง figure

## 6. Export

```python
to_csv(...)
to_netcdf(...)
savefig(...)
```

แปลว่าบันทึกผล

ดังนั้น flow ของ code ส่วนใหญ่คือ:

```text
READ
→ SELECT
→ CALCULATE
→ PLOT
→ INTERPRET
→ EXPORT
```

---

# 20. วิธีอ่าน Figure

ทุกครั้งให้ดูตามลำดับ:

```text
1. Variable อะไร?
2. Pressure level เท่าไร?
3. วันและเวลาอะไร?
4. Units คืออะไร?
5. สีหมายถึงอะไร?
6. Contour หมายถึงอะไร?
7. Vector หมายถึงอะไร?
8. Pattern อยู่ตรงไหน?
9. เปลี่ยนจากวันอื่นอย่างไร?
10. สรุปอะไรได้ และอะไรยังสรุปไม่ได้?
```

---

# 21. Scientific Cautions ที่ต้องจำ

```text
Reanalysis ≠ direct observation
Reanalysis ≠ forecast

grid spacing ≠ effective physical resolution

data CRS ≠ display projection

u/v ≠ wind speed

ERA5 z ≠ geopotential height
ก่อนหารด้วย g₀

positive vorticity ≠ ascent directly

upper-level divergence ≠ vertical velocity

high RH ≠ rainfall automatically

Hovmöller ≠ full spatial field

monthly mean ≠ climatology

seasonal difference ≠ anomaly

anomaly ≠ trend

standardized anomaly ≠ statistical significance

00-UTC snapshot ≠ daily mean
```

---

# 22. Suggested Learning Workflow

ก่อนกด Run ให้ทำตามนี้:

```text
1. อ่านวัตถุประสงค์ Notebook
2. อ่าน Theory
3. อ่านสมการแบบข้อความง่าย
4. ดู input file
5. Run ทีละ cell
6. อ่าน output
7. อ่าน figure
8. เขียน Observation
9. เขียน Physical interpretation
10. เขียน Limitation
11. ทำ Exercise
```

ไม่แนะนำให้ `Run all` ครั้งแรกโดยไม่อ่าน Markdown

---

# 23. Repository Structure

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

# 24. Learning Outcome หลังจบ Repository

เมื่อนิสิตจบ Notebook 00–09 ควรสามารถ:

```text
อ่าน NetCDF metadata
↓
เลือก atmospheric variables
↓
เลือก pressure level
↓
subset พื้นที่
↓
สร้าง weather map
↓
คำนวณ wind / height / climatology / anomaly
↓
สร้าง Hovmöller
↓
เปรียบเทียบหลาย atmospheric levels
↓
ตีความผลเชิงกายภาพ
↓
ระบุข้อจำกัด
↓
ส่งออก figure และ table แบบ reproducible
```

เป้าหมายสุดท้ายคือให้นิสิตตอบคำถามนี้ได้:

> **ข้อมูลบอกอะไร และข้อมูลยังไม่สามารถบอกอะไร?**

---

## Repository

[Teaching ERA5 Reanalysis & GIS](https://github.com/nattaponm/Teaching_ERA5_Reanalysis_GIS)
