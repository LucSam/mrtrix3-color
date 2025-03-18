# MRTrix3 Color Mapping Enhancement

This fork of MRTrix3 implements a custom color mapping system for fiber tractography visualization. It replaces the standard RGB directional encoding with a perceptually optimized color scheme via a lookup table.

## Implementation Details

### Overview

The implementation transforms the standard RGB directional encoding of fiber tracts into a custom color space using a 3D lookup table stored as a BMP file. This enhances visual differentiation between fiber bundles with similar orientations.

### Technical Implementation

The color transformation works through these key steps:

1. **RGB to Index Conversion**
   Each fiber direction's RGB components (ranging from 0-1) are converted to a single integer index:
   ```
   index = ⌊R×255⌋ + ⌊G×255⌋×256 + ⌊B×255⌋×65536
   ```
   
   This creates a unique index for each possible RGB combination, where each component occupies 8 bits in a 24-bit integer.

2. **Color Lookup**
   The calculated index is used to retrieve a new color from the lookup table:
   ```
   R_new = LookupTable[3×(index+offset)]
   G_new = LookupTable[3×(index+offset)+1]
   B_new = LookupTable[3×(index+offset)+2]
   ```

3. **Implementation in Fragment Shader**
   The shader code for this transformation:
   ```glsl
   int pos = int(colour.r*255)+int(colour.g*255)*256+int(colour.b*255)*65536;
   colour.r = texelFetch(u_color_mapper, 3*(pos+18)).r;
   colour.g = texelFetch(u_color_mapper, 3*(pos+18)+1).r;
   colour.b = texelFetch(u_color_mapper, 3*(pos+18)+2).r;
   ```

### Source Code Modifications

The changes are primarily implemented in `src/gui/mrview/tool/tractography/tractogram.cpp`:

1. Added texture buffer uniform for the color map:
   ```cpp
   uniform samplerBuffer u_color_mapper;
   ```

2. Modified fragment shader to perform color lookup:
   ```cpp
   source += "int pos = int(colour.r*255)+int(colour.g*255)*256+int(colour.b*255)*65536;\n";
   source += "colour.r = texelFetch(u_color_mapper, 3*(pos+18)).r;\n";
   source += "colour.g = texelFetch(u_color_mapper, 3*(pos+18)+1).r;\n";
   source += "colour.b = texelFetch(u_color_mapper, 3*(pos+18)+2).r;\n";
   ```

3. Added code to load the BMP file:
   ```cpp
   char *colors = new char[3*4096*4096+18];
   std::ifstream infile(getenv("MRTRIX_IMAGE"));
   infile.read(colors, 3*4096*4096+18);
   ```

4. Created and bound OpenGL buffers for the color map:
   ```cpp
   gl::GenBuffers(1, &color_m);
   gl::BindBuffer(gl::TEXTURE_BUFFER, color_m);
   gl::BufferData(gl::TEXTURE_BUFFER, 3*4096*4096+18, colors, gl::STATIC_DRAW);
   gl::GenTextures(1, &color_mapper);
   ```

## Usage

1. Clone the repository
   ```bash
   git clone https://github.com/yourusername/mrtrix3.git
   cd mrtrix3
   ```

2. Build MRTrix3 following the standard instructions
   ```bash
   ./configure
   ./build
   ```

3. Set the environment variable to point to the color map BMP
   ```bash
   export MRTRIX_IMAGE=/path/to/out2.bmp
   ```

4. Launch mrview and visualize your tractography data
   ```bash
   ./bin/mrview your_tractography.tck
   ```

The color map (out2.bmp) should be located in the repository root directory after cloning.

## Benefits

This enhanced color mapping provides several advantages:
- Improved visual differentiation between fiber bundles
- Better perception of fiber orientations
- Enhanced visualization for research and clinical applications
- Real-time performance without preprocessing
