# GIFTmacro – ImageJ Macro for Automated Fiber Diameter Analysis

This repository contains an ImageJ/Fiji macro (`GIFTmacro`) that automates the measurement of electrospun fiber diameters from SEM images.  

The macro is based on the **General Image Fiber Tool (GIFT)** method described and validated in:

> A. Götz, V. Senz, W. Schmidt, J. Huling, N. Grabow, S. Illner, *Measurement* 177 (2021) 109265.

The macro provides:

- A **toolbar menu tool** in ImageJ/Fiji (“GIFTmacro Menu Tool”)
- **Single-image** analysis mode
- **Batch** analysis mode for folders of images
- Optional **fiber orientation analysis** via existing plugins

---

## Features

- Automatic detection and measurement of fiber diameters from SEM images
- Rotational analysis (0–180°) to improve sampling of fiber orientations
- Edge detection and automatic thresholding based on a user-defined **percent of included pixels**
- Morphological filtering using a horizontal line structuring element
- **Gaussian fitting** of the diameter histogram to estimate mean fiber diameter and standard deviation
- Optional **orientation analysis** using:
  - [OrientationJ](https://imagej.net/plugins/orientationj)
  - [Directionality](https://imagej.net/plugins/directionality)
- Single-image mode (interactive) and batch mode (automated for many images)
- Exportable results tables and histogram plots

---

## Requirements

### Software

- **ImageJ** or **Fiji** (recommended: Fiji, as many plugins are pre-bundled)

### Required Plugins

The macro calls the following plugins:

1. **Morphological Filters**  
   Used for erosion/dilation with a horizontal line structuring element.  
   - Menu name: `Morphological Filters`
   - Operation: `Erosion` and `Dilation`
   - Element: `Horizontal Line`

2. **Directionality** (optional, for orientation analysis)  
   - Menu name: `Directionality`
   - Method used: `Fourier components`

3. **OrientationJ** (optional, for orientation analysis)  
   - Menu name: `OrientationJ Distribution`

Make sure these plugins are installed and accessible from the ImageJ/Fiji menu before running the macro.

---

## Installation

1. Open **ImageJ/Fiji**.
2. Go to **`Plugins › Macros › New…`** (or **Edit…** if you want to paste into an existing macro window).
3. Paste the entire macro code into the editor.
4. Save the file as something like:

   - `GIFTmacro.ijm` in the `macros/toolsets` folder of your ImageJ/Fiji installation.

5. In ImageJ/Fiji, load the toolset via:

   - **`More Tools (>> icon on toolbar) › Install…`**  
     or  
   - **`Plugins › Macros › Install…`** and select `GIFTmacro.ijm`.

6. After installation, you should see a **toolbar button** named:

   > `GIFTmacro Menu Tool`

Clicking this tool opens a small menu with two options:

- `Process Single Open Image`
- `Batch Process`

---

## Usage

### 1. Process Single Open Image

This mode processes the currently active image in ImageJ/Fiji.

**Steps:**

1. Open a SEM image containing fibers (e.g. `.tif`, `.jpg`, `.png`, `.bmp`).
2. Click the **“GIFTmacro Menu Tool”** on the toolbar.
3. Choose **`Process Single Open Image`**.
4. A dialog titled **“New Analysis”** will appear.

#### Single Image – Dialog Parameters

**GIFT Parameters:**

- **Degrees of rotation**  
  - Step size (in degrees) for rotating the image between 0 and 180°.  
  - Smaller steps → more measurements, longer runtime.

- **Line Length**  
  - Length (in pixels) of the horizontal structuring element used in morphological erosion/dilation.  
  - Should roughly match or slightly exceed the typical fiber diameter.

- **Enter the threshold value (percent of pixels to include)**  
  - Percentage of pixels (based on edge image) used to determine the threshold.  
  - The macro computes a threshold so that approximately this fraction of pixels is kept as “edge” pixels.

- **Or check to manually set image thresholding in the next step**  
  - If checked, the macro shows a threshold window where you manually adjust the threshold on the processed image.  
  - The macro then converts your manual setting into an equivalent percentage.

- **Set bin size for diameter histograms**  
  - Bin width for the histogram of measured distances (fiber diameters) in your chosen unit.

---

**Image Parameters:**

- **Distance in pixels / Known distance**  
  - Used to set the **scale factor**.  
  - Example: if 100 pixels correspond to 1 µm, then:
    - *Distance in pixels* = 100  
    - *Known distance* = 1  
  - If both are 1, the measurements are kept in **pixels**.

- **Manually set scale in next step**  
  - If checked, you will be prompted to **draw a line on the scalebar** in your image and then enter its actual length (e.g. in µm).  
  - The macro will compute and use the corresponding pixel-to-unit conversion.

---

**Cropping Options:**

The macro can optionally restrict analysis to a sub-region (for example, to exclude a scalebar or labels):

- **Define image area to be analysed by known height**  
  - If checked, you enter the **height of the region (in pixels)** measured from the **top** of the image.
  - Only this top region is processed.

- **Check to manually select the area to be analysed in next step**  
  - If checked, you will manually draw a rectangle on the image.
  - The macro uses this selection as the analysis region.
  
> ⚠ **Important:** Do **not** select both cropping methods at once. The macro will exit with an error if both are enabled.

---

**Fiber Orientation Measurement (Optional):**

- **Choose plugin:**  
  - `None` – no orientation analysis.  
  - `OrientationJ` – uses OrientationJ Distribution.  
  - `Directionality` – uses the Directionality plugin (Fourier components).

If an orientation plugin is selected, the macro will:

- Run it once (at 0° rotation),
- Fit a Gaussian to the orientation histogram,
- Extract:
  - **Orientation Mean**
  - **Orientation SDEV**
  - **Orientation R²**
- Add these values to the **Results** table.

---

#### Single Image – Processing Steps (Summary)

Internally, the macro:

1. Optionally crops the image.
2. Converts to **8-bit**.
3. Runs **Find Edges**.
4. Applies either automatic thresholding (based on a percentile) or your manual threshold.
5. Converts to a **binary mask**.
6. For each rotation angle from 0° to <180°:
   - Rotates the thresholded image.
   - Applies **morphological erosion + dilation** with a horizontal line.
   - Scans each column for peaks along the vertical pixel profile.
   - Computes distances between successive peaks and converts them to physical units using the scale.
   - Accumulates all distances into a single array.

7. Builds a **histogram** of all distances (fiber diameters).
8. Fits a **Gaussian** to the histogram to obtain:
   - Mean diameter
   - Standard deviation
   - R² of the fit
9. Writes results and (optionally) orientation statistics to the ImageJ **Results** table and to the **CompleteDataFile / CompleteFrequencyData** tables.

---

### 2. Batch Process

Batch mode processes **all images in a given folder** and saves outputs to a chosen output directory.

**Steps:**

1. Click the **“GIFTmacro Menu Tool”** on the toolbar.
2. Choose **`Batch Process`**.
3. A “New Analysis” dialog will appear.

#### Batch Mode – Dialog Parameters

**Input/Output and Saving:**

- **Choose Input Directory**  
  - Folder containing your SEM images.  
  - ⚠ Avoid placing non-image files in this folder; they may cause the macro to fail.

- **Choose Output Directory**  
  - Folder where results and plots will be saved.

- **Save image processing intermediates (edge detection & morphological filtering)**  
  - If checked, the macro saves intermediate images (edges and filtered images) for each rotation angle.

- **Add the following label to all saved file names:**  
  - Optional label prefix (e.g. `Sample1_`).  
  - The macro automatically adds an underscore.

---

**GIFT Parameters, image parameters, cropping, and orientation options** are equivalent to those in single-image mode, with the difference that:

- **Manual scale selection** (if enabled) is done **once** on the **first image** in the input directory.
- **Manual cropping** (if enabled) is also defined **once** using the first image.
- **Manual thresholding** (if enabled) is defined only once on the first image and used for the entire batch (as a percentage).

---

#### Batch Mode – Outputs

For each image in the input directory, the macro:

- Computes diameter distributions and Gaussian fits.
- Appends per-image results to the ImageJ **Results** table:
  - Diameter Mean
  - Diameter SDEV
  - Diameter R²
  - # of Observations
  - Label (filename)
  - Optional orientation metrics (if selected)

- Updates the global tables:
  - `CompleteDataFile`
  - `CompleteFrequencyData`
  - `CompleteOrientationData` (if orientation plugins are used)

Additionally, for each image:

- Saves two histogram plots:
  - **Tiff**:  
    `labelString + "Histogram_modifiable_" + filename.tif`  
    (modifiable in ImageJ)
  - **PNG**:  
    `labelString + "Histogram_" + filename.png`  
    (for direct use in publications/presentations)

At the end of batch processing, a final dialog asks how to save the tables:

- **Don’t Save**
- **Save as .txt**
- **Save as .CSV**
- **Save as .xls**

Depending on your choice, the macro writes:

- `BatchResults.(txt/csv/xls)`
- `CompleteDataFile.(txt/csv/xls)`
- `CompleteFrequencyFile.(txt/csv/xls)`
- `CompleteOrientationFile.(txt/csv/xls)` (if orientation analysis is enabled)

into the **output directory**, prefixed by the optional label string.

---

## Interpreting the Results

### Results Table (per-image)

Columns include:

- **Diameter Mean**  
  Gaussian mean of the measured fiber diameters (in your chosen units or pixels).

- **Diameter SDEV**  
  Gaussian standard deviation of the fiber diameter distribution.

- **Diameter Rsqr**  
  Coefficient of determination (R²) of the Gaussian fit.

- **# of Observations**  
  Total number of distance measurements contributing to the histogram.

- **Label**  
  Original image filename.

If orientation analysis is enabled:

- **Orientation Mean (degree)**  
- **Orientation SDEV (degree)**  
- **Orientation Rsqr**

---

## Troubleshooting & Tips

- **Macro exits with “User selected both cropping methods”**  
  → In the dialog, enable **either** “Define image area by known height” **or** “manually select area”, not both.

- **Macro exits with “Invalid value … (must be positive)”**  
  → Check numeric fields (degrees, line length, threshold percent, bin size, scale, height). All must be ≥ 0, and for height > 0.

- **No line / crop area selected when prompted**  
  → When the macro pauses and asks you to draw a line or rectangle, use the appropriate selection tool and **confirm** with OK.

- **Batch crashes with non-image files**  
  → Ensure the input directory only contains supported image formats.

- **Scale and units**  
  - If you want diameters in µm, set the scale such that the known distance is in µm.
  - If you keep both values as 1, results remain in pixels.

---

## Citation

If you use this macro or its underlying method in your work, please cite:

> A. Götz, V. Senz, W. Schmidt, J. Huling, N. Grabow, S. Illner, “Automated measurement of fiber diameters of electrospun fibers in SEM images,” *Measurement* 177 (2021) 109265.

---

## Acknowledgements

- Original method and GIFT concept developed at the **Institute for Bioengineering**, University of Rostock Medical Center.
- This macro adapts the GIFT workflow into an ImageJ/Fiji macro with both single-image and batch processing modes.

---
