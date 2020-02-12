## pytorch-debayer

Provides GPU demosaicing of images captured with Bayer color filter arrays (CFA) with batch support. This implementation relies on pure PyTorch functionality and thus avoids any extra build steps.

Currently, two modules based on bilinear interpolation are provided
 - `debayer.Debayer2x2` uses 2x2 convolutions. Trades speed for color accuracy.
 - `debayer.Debayer3x3` uses 3x3 convolutions. Slower but reconstruction results comparable with `OpenCV.cvtColor`.

### Usage
Usage is straight forward

```python
import torch
from debayer import Debayer3x3

f = Debayer3x3().cuda()

bayer = ...     # a Bx1xHxW, torch.float32 tensor of BG-Bayer images
rgb = f(bayer)  # a Bx3xHxW, torch.float32 tensor of RGB images
```

see [this example](debayer/apps/example.py) for elaborate code.

### Limitations

Currently **pytorch-debayer** requires BG-Bayer color filter array layout. According to OpenCV naming conventions (see [here](https://docs.opencv.org/4.2.0/de/d25/imgproc_color_conversions.html) towards end of file) that means your Bayer input image must be arranged in the following way
```
RGRGRG...
GBGBGB...
RGRGRG...
```

### Benchmark
Runtimes processing a 5 megapixel image.

Method | Device | Elapsed | Mode |
|:----:|:------:|:-------:|:----:|
| Debayer2x2 | GeForce GTX 1070 | 3.49 msec | time_upload=True |
| Debayer2x2 | GeForce GTX 1070 | 1.89 msec | time_upload=False |
| Debayer3x3 | GeForce GTX 1070 | 6.17 msec | time_upload=False |
| OpenCV 4.0.0 | CPU i5-6600 | 3.58 msec | transparent_api=False |
| OpenCV 4.0.0 | CPU i5-6600 | 15.77 msec | transparent_api=True |

See [benchmark code](debayer/apps/benchmark.py). Invoke with

```
> python -m debayer.apps.benchmark etc\test.bmp
```