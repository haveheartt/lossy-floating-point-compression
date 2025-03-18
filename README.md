# Lossy Floating-Point Compression

There is two jupyter notebook files, the first, unpacked is the initial version where I was experimenting with the LSB compression. There I zero outed the mantissa bits from the floating point numbers of the three distributions (uniform, Gaussian, and exponential) and saved it in bin files. Then i realized that filesize of the full-precision bin file and the compressed bin file were the same. 

Silly as I was i did not understood why, then I went to my good old friend internet and realized that:
    
- Both the original and compressed data are stored as 32-bit floats (4 bytes per value). And even though the compressed data has reduced precision (due to zeroing out LSBs), it is still stored in the same 32-bit format, so the file size remains unchanged.

So I could do two things:

- Convert the compressed data to 16-bit floats (2 bytes per value) before saving it to the binary file, which would reduce the filesize by 50% but would also reduce the precision.

- Or I could extract the bits we want to keep (exponent and reduced mantissa), pack them into a more compact data type and save this compact representation to bin files and that is what I did on the packed file. 
 
## How It Works

The compression system operates in two stages:

1. **Precision Reduction**: Zeroes out the least significant bits of the mantissa in IEEE 754 single-precision floats
2. **Bit Packing**: Extracts and packs only the significant bits (sign, exponent, and reduced mantissa) into a more compact representation

IEEE 754 single-precision floats use 32 bits structured as:
- 1 sign bit
- 8 exponent bits
- 23 mantissa bits

Our compression preserves the sign and exponent bits while reducing the mantissa bits (e.g., from 23 to 12 bits).

## Results

Using a compression level of 12 mantissa bits, we observed:

### Compression Ratio

| Distribution | Original Size | Compressed Size | Ratio |
|-------------|--------------|----------------|-------|
| Uniform     | 400,000 bytes | 300,000 bytes  | 1.33x |
| Gaussian    | 400,000 bytes | 300,000 bytes  | 1.33x |
| Exponential | 400,000 bytes | 300,000 bytes  | 1.33x |

### Statistical Preservation

| Distribution | Metric | Original | Compressed | Difference |
|-------------|--------|----------|------------|------------|
| Uniform     | Mean   | -0.1587  | -0.1587    | ~0         |
|             | Variance | 3327.43 | 3327.43   | ~0         |
|             | Std Dev | 57.684  | 57.684     | ~0         |
| Gaussian    | Mean   | -0.0741  | -0.0741    | ~0         |
|             | Variance | 101.01 | 101.01     | ~0         |
|             | Std Dev | 10.051  | 10.051     | ~0         |
| Exponential | Mean   | 9.981   | 9.981      | ~0         |
|             | Variance | 99.802 | 99.802     | ~0         |
|             | Std Dev | 9.990   | 9.990      | ~0         |

### Error Analysis

| Distribution | Mean Squared Error |
|-------------|-------------------|
| Uniform     | 9.20e-6           |
| Gaussian    | 2.76e-7           |
| Exponential | 5.37e-7           |

The compression maintains statistical properties with minimal error, while reducing storage requirements by 25%.

## Optimal Compression Levels

When choosing the appropriate mantissa bit reduction level for float compression, it's needed to carefully balance precision requirements against storage benefits:
