# BMP280 sensor API

## Table of Contents

- [Introduction](#Intro)
- [Version and date](#Ver)
- [Driver files information](#file)
- [Sensor interfaces](#interface)
- [Integration Examples](#examples)
 
### Introduction<a name = Intro></a>

This package contains the Bosch Sensortec's BMP280 sensor driver (sensor API)

The sensor driver package includes bmp280.h, bmp280.c and bmp280_defs.h files

### Version and date<a name = Ver></a>

Driver files  | Version | Date
--------------|---------|------------
bmp280.c      |  3.0.0  | 3 Apr 2018
bmp280.h      |  3.0.0  | 3 Apr 2018
bmp280_defs.h |  3.0.0  | 3 Apr 2018

### Driver files information<a name = file></a>

- `bmp280.c` : This source file contains the function definitions of the sensor API interfaces.
- `bmp280.h` : This header file contains the function declarations of the sensor API interfaces.
- `bmp280_defs.h` : This header file has the constants, macros and data type declarations.

## Sensor interfaces<a name = interface></a>

- SPI 4-wire, SPI 3-wire interface
- I2C interface

### Integration Examples<a name = examples></a>

#### Initializing the sensor

To initialize the sensor, you will first need to create a device structure.
You can do this by creating an instance of the structure bmp280_dev. 
Then update the various parameters in structure as shown below.

##### Initialization for SPI 4-Wire interface

```c
int8_t rslt;
struct bmp280_dev bmp;

/* Sensor interface over SPI with native chip select line */
bmp.dev_id  =  0;
bmp.intf = BMP280_SPI_INTF;
bmp.read = user_spi_read;
bmp.write = user_spi_write;
bmp.delay_ms = user_delay_ms;

rslt = bmp280_init(&bmp);

if (rslt == BMP280_OK) {
  /* Sensor chip ID will be printed if initialization was successful */
  printf("\n Device found with chip id 0x%x", bmp.chip_id);
}
```

##### Initialization for I2C interface

``` c
int8_t rslt;
struct bmp280_dev bmp;
/* Sensor interface over I2C with primary slave address  */
bmp.dev_id = BMP280_I2C_ADDR_PRIM;
bmp.intf = BMP280_I2C_INTF;
bmp.read = user_i2c_read;
bmp.write = user_i2c_write;
bmp.delay_ms = user_delay_ms;

rslt = bmp280_init(&bmp);

if (rslt == BMP280_OK) {
  /* Sensor chip ID will be printed if initialization was successful */
  printf("\n Device found with chip id 0x%x", bmp.chip_id);
}

```

#### Sensor Configuration settings

> Pre-requisite : Initialize the sensor in SPI or I2C

```c
struct bmp280_config conf;

/* Always read the current settings before writing, especially when
 * all the configuration is not modified 
 */
rslt = bmp280_get_config(&conf, &bmp);
/* Check if rslt == BMP280_OK, if not, then handle accordingly */

/* Overwrite the desired settings */
conf.filter = BMP280_FILTER_COEFF_2;
conf.os_pres = BMP280_OS_16X;
conf.os_temp = BMP280_OS_4X;
conf.odr = BMP280_ODR_1000_MS;

rslt = bmp280_set_config(&conf, &bmp);
/* Check if rslt == BMP280_OK, if not, then handle accordingly */

/* Always set the power mode after setting the configuration */
rslt = bmp280_set_power_mode(BMP280_NORMAL_MODE, &bmp);
/* Check if rslt == BMP280_OK, if not, then handle accordingly */
```

#### Example for data read out

> Pre-requisite : Initialize the sensor in SPI or I2C and set sensor mode and sensor settings

```c
struct bmp280_uncomp_data ucomp_data;
uint8_t meas_dur = bmp280_compute_meas_time(&bmp);

printf("Measurement duration: %dms\r\n", meas_dur);

/* Loop to read out 10 samples of data */ 
for (uint8_t i = 0; (i < 10) && (rslt == BMP280_OK); i++) {
    bmp.delay_ms(meas_dur); /* Measurement time */

    rslt = bmp280_get_uncomp_data(&ucomp_data, &bmp);
    /* Check if rslt == BMP280_OK, if not, then handle accordingly */

    int32_t temp32 = bmp280_comp_temp_32bit(ucomp_data.uncomp_temp, &bmp);
    uint32_t pres32 = bmp280_comp_pres_32bit(ucomp_data.uncomp_press, &bmp);
    uint32_t pres64 = bmp280_comp_pres_64bit(ucomp_data.uncomp_press, &bmp);
    double temp = bmp280_comp_temp_double(ucomp_data.uncomp_temp, &bmp);
    double pres = bmp280_comp_pres_double(ucomp_data.uncomp_press, &bmp);

    printf("UT: %d, UP: %d, T32: %d, P32: %d, P64: %d, P64N: %d, T: %f, P: %f\r\n", \
      ucomp_data.uncomp_temp, ucomp_data.uncomp_press, temp32, \
      pres32, pres64, pres64 / 256, temp, pres);

    bmp.delay_ms(1000); /* Sleep time between measurements = BMP280_ODR_1000_MS */
}
```