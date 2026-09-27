# microsvg.h

> A single-header C99 SVG 2 decoder and rasterizer.

*should be complete excluding scripting support (not planned)*

## Origins & Credits

Originally forked from [NanoSVG](https://github.com/memononen/nanosvg)

> Copyright (c) 2013-14 Mikko Mononen memon@inside.org

The NanoSVG-derived parser and rasterizer code has since been rewritten and expanded to implement the SVG 2 specification; this is no longer a drop-in replacement for NanoSVG and its API is its own.

The SVG parser is based on:
- Anti-Grain Geometry 2.4 SVG example - Copyright (C) 2002-2004 Maxim Shemanarev (McSeem) (http://www.antigrain.com/)

Other attributions:
- Arc calculation code based on [canvg](https://code.google.com/p/canvg/)
- Bounding box calculation based on http://blog.hackers-cafe.net/2009/06/how-to-calculate-bezier-curves-bounding.html
- The polygon rasterization is heavily based on stb_truetype rasterizer by Sean Barrett - http://nothings.org/

## License

zlib License

This software is provided 'as-is', without any express or implied warranty. In no event will the authors be held liable for any damages arising from the use of this software.

Permission is granted to anyone to use this software for any purpose, including commercial applications, and to alter it and redistribute it freely, subject to the following restrictions:

1. The origin of this software must not be misrepresented; you must not claim that you wrote the original software. If you use this software in a product, an acknowledgment in the product documentation would be appreciated but is not required.
2. Altered source versions must be plainly marked as such, and must not be misrepresented as being the original software.
3. This notice may not be removed or altered from any source distribution.

## User API and Workflow

`microsvg.h` is a single-header implementation of the SVG 2 specification. It grew out of a fork of NanoSVG, but the parser and rasterizer have been rewritten to implement SVG 2 and its API is its own; it is not a drop-in replacement for `nanosvg.h` / `nanosvgrast.h`.

The output of the parser is a list of cubic-bezier shapes. The shapes in the SVG image are transformed by the viewBox and converted to the specified units, so you get the same looking data as designed in your favorite app.

The units passed in should be one of: `px`, `pt`, `pc`, `mm`, `cm`, `in` (`px` with a dpi of 96 is a good default). DPI controls unit conversion.

### Example Workflow (Parse + Rasterize)

```c
// Load SVG
MSVGimage* image = msvgParseFromFile("test.svg", "px", 96);
printf("size: %f x %f\n", image->width, image->height);

// Create rasterizer (can be reused for many images)
MSVGrasterizer* rast = msvgCreateRasterizer();

// Rasterize to an RGBA buffer (non-premultiplied alpha, 4 bytes/px)
unsigned char* img = malloc((int)image->width * (int)image->height * 4);
msvgRasterize(rast, image, 0, 0, 1, img,
              (int)image->width, (int)image->height, (int)image->width * 4);

// Work with the shapes directly if desired
for (MSVGshape* shape = image->shapes; shape; shape = shape->next)
    for (MSVGpath* path = shape->paths; path; path = path->next)
        for (int i = 0; i < path->npts-1; i += 3)
            drawCubicBez(path->pts[i*2], path->pts[i*2+1], ...);

free(img);
msvgDeleteRasterizer(rast);
msvgDelete(image);
