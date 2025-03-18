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
| Uniform     | Mean   | -0.12806728524443234  | -0.1280716508626938    | ~0         |
|             | Variance | 3327.7761193531333 | 3327.775390625   | ~0         |
|             | Std Dev | 57.68687995855846  | 57.68687438964844     | ~0         |
| Gaussian    | Mean   | 0.01904169921227771  |  0.019044388085603714    | ~0         |
|             | Variance | 99.31867004046917 | 99.31873321533203     | ~0         |
|             | Std Dev | 9.96587527718811  |  9.9658784866333   | ~0         |
| Exponential | Mean   | 9.972490312644073   | 9.97249698638916      | ~0         |
|             | Variance | 99.18580866104837 | 99.18612670898438     | ~0         |
|             | Std Dev | 9.95920723055045   | 9.959222793579102      | ~0         |

### Error Analysis

| Distribution | Mean Squared Error |
|-------------|-------------------|
| Uniform     | 9.171662042897382e-06          |
| Gaussian    | 2.653010924668988e-07           |
| Exponential |5.305564368932732e-07          |

The compression maintains statistical properties with minimal error, while reducing storage requirements by 25%.

## Optimal Compression Levels

When choosing the appropriate mantissa bit reduction level for float compression, it's essential to carefully balance precision requirements against storage benefits:

### High-Precision Computing

The goal here is maintain numerical accuracy while reducing storage. For example:

    - Climate modeling where a slight change in floating-point precision could produce incorrect long-term projections.

So it's very important to ensure accuracy and prevent data corruption and try to reduce storage at the same time. The methods available can be very CPU-intensive but offer good compression and fast decompression.

## Limited Storage Resources

The goal here is reduce file size for small storage capacities and lightweight processing. For example:

    -  A mobile game with compressed textures and audio files to minimize install size.

So  it's very important to drastically reduce file size but it comes with some cons like quality degradation.

## Real-Time Processing (Like Gaming, Streaming and Cloud Services)

The goal here is to prioritize speed over maximum compression. For example:

    -  Cloud gaming services (e.g., Google Stadia) where real-time encoding and decoding must happen with minimal latency.

So it's very important to have very fast decompression speed and also have adaptive quality based on network conditions which comes with the price of reducing fidelity (prioritize motion efficiency over exact pixel reproduction).
    
