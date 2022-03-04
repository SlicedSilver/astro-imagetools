# Astro ImageTools

`astro-imagetools` comes with the set of tools to perform image optimization and handle responsive images generation for the **Astro JS** framework.

## Installation

To install the package, run the following command:

```bash
npm install astro-imagetools

# yarn
yarn add astro-imagetools

# pnpm
pnpm add astro-imagetools
```

The `astro-imagetools` package comes with an `<Image />` component and a _Vite_ plugin. The component depends on the plugin to be able to optimize and generate responsive images.

It also exports the `renderImage` function used by the component interally to allow the user to programmatically generate image sets.

To use the component, first register the plugin in your `astro.config.js` file:

```js
import astroImagePlugin from "astro-imagetools/plugin";

export default {
  vite: {
    plugins: [astroImagePlugin],
  },
};
```

Then, you'll be able to use the component inside your Astro components as shown below:

```astro
---
import Image from "astro-imagetools";
---

<html>
  <body>
    <h1>Hello World!</h1>
    <Image
      src="/src/images/landscape.jpg"
      alt="alt text"
      artDirectives={[
        {
          media: "(max-aspect-ratio: 3/2)",
          src: "/src/images/portrait.jpg",
        },
      ]}
    />
  </body>
</html>
```

If you want to generate optimize image sets programmatically, you can use the `renderImage` function as shown below:

```js
import renderImage from "astro-imagetools/renderImage";

export async function getRenderedImage({ src, alt, ...rest }) {
  const { link, style, image } = await renderImage({ src, alt, ...rest });

  return link + style + image;
}
```

The `renderImage` function takes the same arguments as the props of the `<Image />` component. The function returns a promise that resolves to an `ImageHTMLData` object which contains the following properties:

```ts
export interface ImageHTMLData {
  link: string;
  style: string;
  image: string;
}
```

## Configuration Options

Both the `<Image />` component and the `renderImage` function supports a total of 40 config options. You can pass them directly to the component as props and to the function as propertise of an object.

### Example Usage

#### `<Image />` Component

```jsx
<Image src="/src/images/landscape.jpg" alt="alt text" />
```

#### `renderImage` Function

```js
const { link, style, image } = await renderImage({
  src: "/src/images/landscape.jpg",
  alt: "alt text",
});
```

### Interface

The `ImageConfig` interface below describes the props that the component and the `renderImage` function accepts. The props are passed to the component as a JSX attribute.

This section is only for quick reference and for astronomers familiar with _TypeScript_. If you are not comfortable with `TypeScript` or need more information, you can skip this section and move on to the next section for a more detailed explanation with examples.

**Note:** The `<Image />` component and the plugin fallback to `@astropub/codecs` for processing images if the environment is unable to install `sharp`. Most of the properties defined in the `ImagetoolsConfig` interface won't be available in this case.

```ts
// The formats supported by the `<Image />` component and the plugin.
declare type format =
  | "heic"
  | "heif"
  | "avif"
  | "jpg"
  | "jpeg"
  | "png"
  | "tiff"
  | "webp"
  | "gif";

// Check https://github.com/tooolbox/node-potrace for more info on the properties of the PotraceOptions interface.
declare type PotraceOptions = TraceOptions | PosterizeOptions;

declare interface SharedTracingOptions {
  turnPolicy?: "black" | "white" | "left" | "right" | "minority" | "majority";
  turdSize?: number;
  alphaMax?: number;
  optCurve?: boolean;
  optTolerance?: number;
  threshold?: number;
  blackOnWhite?: boolean;
  color?: "auto" | string;
  background?: "transparent" | string;
}

declare interface TraceOptions {
  function?: "trace";
  options?: SharedTracingOptions;
}

declare interface PosterizeOptions {
  function?: "posterize";
  options?: SharedTracingOptions & {
    fill?: "spread" | "dominant" | "median" | "mean";
    ranges?: "auto" | "equal";
    steps?: number | number[];
  };
}

declare interface FormatOptions {
  format?: format | format[] | [] | null;
  // The image format or formats to generate image sets for. If `format` is set to `null` or `[]`, no image will be generated.

  // **Note:** Passing `[]` or `null` does not necessarily mean that no image will be generated. If `includeSourceFormat` is set to `true`, then the source format and the format specified in the `fallbackFormat` prop will still be generated.
  fallbackFormat?: boolean;
  // The format the browser will fallback to if the other formats are not supported by it. If not provided, the format of the source image will be used.
  includeSourceFormat?: boolean;
  // Whether to generate image set for the source format or not.
  formatOptions?: Record<format, ImageToolsConfigs> & {
    tracedSVG?: PotraceOptions;
    // Check the format type and ImageToolsConfigs & PotraceOptions interfaces for the supported properties
  };
}

declare interface ImageToolsConfigs {
  flip?: boolean;
  flop?: boolean;
  invert?: boolean;
  flatten?: boolean;
  normalize?: boolean;
  grayscale?: boolean;
  hue?: number;
  saturation?: number;
  brightness?: number;
  w?: number;
  h?: number;
  ar?: number;
  width?: number;
  height?: number;
  aspect?: number;
  background?: string;
  tint?: string;
  blur?: number | boolean;
  median?: number | boolean;
  rotate?: number;
  quality?: number;
  fit?: "cover" | "contain" | "fill" | "inside" | "outside";
  kernel?: "nearest" | "cubic" | "mitchell" | "lanczos2" | "lanczos3";
  position?:
    | "top"
    | "right top"
    | "right"
    | "right bottom"
    | "bottom"
    | "left bottom"
    | "left"
    | "left top"
    | "north"
    | "northeast"
    | "east"
    | "southeast"
    | "south"
    | "southwest"
    | "west"
    | "northwest"
    | "center"
    | "centre"
    | "cover"
    | "entropy"
    | "attention";
}

declare interface ArtDirective
  extends PrimaryProps,
    FormatOptions,
    ImageToolsConfigs {
  media: string;
  // The media query for the intended media of the image.

  // Check the PrimaryProps, FormatOptions and ImageToolsConfigs interface for the other supported options.
}

declare type sizesFunction = {
  (breakpoints: number[]): string;
};

declare interface PrimaryProps {
  src: string;
  sizes?: string | sizesFunction;
  objectPosition?: string;
  objectFit?: "fill" | "contain" | "cover" | "none" | "scale-down";
  placeholder?: "dominantColor" | "blurred" | "tracedSVG" | "none";
  breakpoints?:
    | number[]
    | {
        count?: number;
        minWidth?: number;
        maxWidth?: number;
      };
}

export interface ImageConfig
  extends PrimaryProps,
    FormatOptions,
    ImageToolsConfigs {
  alt: string;
  // The value of the `alt` attribute of the `<img />` element.
  preload?: boolean | format;
  // Whether to preload the image or not or what format of image to preload.
  loading?: "lazy" | "eager" | "auto" | null;
  // The value of the `loading` attribute of the `<img />` element.
  decoding?: "async" | "sync" | "auto" | null;
  // The value of the `decoding` attribute of the `<img />` element.
  layout?: "constrained" | "fixed" | "fullWidth" | "fill";
  // The layout mode of the image.

  // In `constrained` mode, the image will occupy full width of the container with `max-width` set to its width. The height of the image will be calculated based on the aspect ratio of the image. The image will be scaled down to fit the container but won't be enlarged.

  // In `fixed` mode, the image will have a fixed width and height. The `width` and `height` props will be used to set the width and height of the image. The image won't be scaled down nor enlarged.

  // In `fullWidth` mode, the image will be scaled up or down to occupy the full width of the container. The height of the image will be calculated based on the aspect ratio of the image.

  // In `fill` mode, the image will be scaled up or down to fill the entire width and height of the container.
  artDirectives?: ArtDirective[];
  // Check the ArtDirective interface for more details.
}
```

### `PrimaryProps`

The properties described in the `PrimaryProps` interface are some of the main props and they are shared with the `ArtDirectives` too. All the props of the `PrimaryProps` interface are optional except for the `src` and `alt` properties.

#### src

**Type:** `string`

**Default:** `undefined`

The absolute path to the source image.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" />
```

#### alt

**Type:** `string`

**Default:** `undefined`

The value of the `alt` attribute of the `<img />` element.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" />
```

#### sizes

**Type:** `string` or `(breakpoints: number[]) => string`

**Default:** `` (breakpoints) => `(min-width: ${breakpoints.at(-1)}px) ${breakpoints.at(-1)}px, 100vw ``

A string or function that returns a string suitable for the value of the `sizes` attribute of the `<img />` element. The final calculated breakpoints are passed to the function as a parameter.

**Code example:**

```astro
<!-- string type -->
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  sizes="(min-width: 400px) 400px, 100vw"
/>

<!-- function type -->
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  sizes={(breakpoints) => {
    const maxWidth = breakpoints.at(-1);
    return `(min-width: ${maxWidth}px) ${maxWidth}px, 100vw`;
  }}
/>
```

#### objectPosition

**Type:** `string`

**Default:** `50% 50%`

The value of the `object-position` CSS property of the `<img />` element.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  objectPosition="top left"
/>
```

#### objectFit

**Type:** `"fill" | "contain" | "cover" | "none" | "scale-down"`

**Default:** `"cover"`

The value of the `object-fit` CSS property of the `<img />` element.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  objectFit="contain"
/>
```

#### placeholder

**Type:** `"dominantColor" | "blurred" | "tracedSVG" | "none"`

**Default:** `"blurred"`

The placeholder to be displayed while the image is loading. If `placeholder` is set to `"dominantColor"`, the dominant color of the image will be used as the placeholder. If it is set to `"blurred"`, a very low-resolution version of the provided image will be enlarged and used as the placeholder. If it is set to `"tracedSVG"`, a traced SVG of the image will be used as the placeholder. If it is set to `"none"`, no placeholder will be displayed.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  placeholder="dominantColor"
/>
```

#### breakpoints

**Type:** `number[] | { count?: number; minWidth?: number; maxWidth?: number }`

**Default:** `undefined`

An array of widths in pixels to generate image sets for. If not provided, the breakpoints will be calculated automatically based on the width of the provided image.

If an object is passed then the breakpoints will be calculated based on `count`, `minWidth`, and `maxWidth` properties. The `count` property is to specify the number of breakpoints to generate. The `minWidth` and `maxWidth` properties are to specify the widths to generate in the range between their values.

When an object is passed or the `breakpoints` prop is not provided, the breakpoints are calculated using a simple formula or algorithm. Instead of explaining the complete algorithm here, I am linking to the [source code](https://github.com/RafidMuhymin/astro-imagetools/blob/main/src/component/utils/getBreakpoints.js) of it.

**Code example:**

```astro
<!-- number[] type -->
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  breakpoints={[200, 400, 800, 1600]}
/>

<!-- { count?: number; minWidth?: number; maxWidth?: number } type -->
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  breakpoints={{ count: 5, minWidth: 200, maxWidth: 1600 }}
  /* five breakpoints will be generated ranging from 200px to 1000px */
/>
```

### `ImageToolsConfigs`

The properties described in the `ImageToolsConfigs` interface are the directives
supported by the [`imagetools-core`](https://npmjs.com/package/imagetools-core) library. All the directives are documented in the [directives documentation](https://github.com/JonasKruckenberg/imagetools/blob/main/docs/directives.md) of the `imagetools-core` library. They are being documented here to reflect the changes made in the `astro-imagetools` package and for the component syntax.

> **Note:** The values passed in the `background` and `tint` property will be parsed by the [`color-string`](https://www.npmjs.com/package/color-string) library so all color values known from css like rgb, rgba or named colors can be used.
>
> The `format` property is not defined in the `ImageToolsConfigs` interface because it works differently in the context of the `<Image />` component.
>
> The values passed in the `width`, `height` and `aspect` properties are used to resize the image when loading. The final image widths will be calculated from the [`breakpoints`](#breakpoints) property.
>
> The `imagetools-core` package supports `number[]` values for a few directives. But the `<Image />` component doesn't support them because they don't make sense in the context of the component.

#### flip

**Type:** `boolean`

**Default:** `undefined`

Flip the image about the vertical axis. This step is always performed **after** any rotation.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" flip />
```

#### flop

**Type:** `boolean`

**Default:** `undefined`

Flop the image about the horizontal axis. This step is always performed **after** any rotation.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" flop />
```

#### invert

**Type:** `boolean`

**Default:** `undefined`

Produces a **negative** of the image.

#### flatten

**Type:** `boolean`

**Default:** `undefined`

This directive will remove the alpha channel of the image, reducing filesize. Transparent pixels will be merged with the color set by [`background`](#background).

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" flatten />
```

#### normalize

**Type:** `boolean`

**Default:** `undefined`

**Normalizes** the image by stretching its luminance to cover the full dynamic range. This Enhances the output image contrast.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" normalize />
```

#### grayscale

**Type:** `boolean`

**Default:** `undefined`

Converts the image to an 8-bit grayscale image.

> **Note:** This directive will convert the image to the `b-w` colorspace, meaning the resulting image will only have one channel.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" normalize />
```

#### hue

**Type:** `number`

**Default:** `undefined`

Adjusts the images `hue` rotation by the given number of degrees. Commonly used together with [`saturation`](#saturation) and [`brightness`](#brightness).

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" hue={-30} />
```

#### saturation

**Type:** `number`

**Default:** `undefined`

Adjusts the images `saturation` with the given saturation multiplier. Commonly used together with [`hue`](#hue) and [`brightness`](#brightness).

**Code example:**

<!-- prettier-ignore -->
```astro
<Image 
  src="https://picsum.photos/200/300" 
  alt="A random image" 
  saturation={0.5}
/>
```

#### brightness

**Type:** `number`

**Default:** `undefined`

Adjusts the images `brightness` with the given brightness multiplier. Commonly used together with [`hue`](#hue) and [`saturation`](#saturation).

**Code example:**

<!-- prettier-ignore -->
```astro
<Image 
  src="https://picsum.photos/200/300" 
  alt="A random image" 
  brightness={0.5}
/>
```

#### `width` | `w`

**Type:** `number`

**Default:** The width of the image

Resizes the image to be the specified amount of pixels wide. If not given the height will be scaled accordingly.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" width={400} />
```

#### `height` | `h`

**Type:** `number`

**Default:** The height of the image

Resizes the image to be the specified amount of pixels tall. If not given the width will be scaled accordingly.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" height={400} />
```

#### `aspect` | `ar`

**Type:** `number`

**Default:** The aspect ratio of the image

Resizes the image to be the specified aspect ratio. If height and width are both provided, this will be ignored. If height is provided, the width will be scaled accordingly. If width is provided, the height will be scaled accordingly. If neither height nor width are provided, the image will be cropped to the given aspect ratio.

**Code example:**

<!-- prettier-ignore -->
```astro
<Image 
  src="https://picsum.photos/200/300"
  alt="A random image"
  aspect={3/2}
/>
```

#### background

**Type:** `string`

**Default:** `undefined`

This instructs various directives (e.g. the [`rotate`](#rotate)) to use the specified color when filling empty spots in the image.

> **Note:** This directive does nothing on it's own, it has to be used in conjunction with another directive.
> You also cannot set multiple values.

**Code example:**

```astro
---
const src = "https://picsum.photos/200/300";
const alt = "A random image";
---

<Image {src} {alt} flatten background="#FFFFFFAA" />

<Image {src} {alt} rotate={90} background="hsl(360, 100%, 50%)" />
```

#### tint

**Type:** `string`

**Default:** `undefined`

Tints the image using the provided chroma while preserving the image luminance. If the image has an alpha channel it will be untouched.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  tint="rgba(10,33,127)"
/>
```

#### blur

**Type:** `number | boolean`

**Default:** `undefined`

Blurs the image. When no argument is provided it performs a fast blur. When an argument between _0.3 and 1000_ is provided it performs a more accurate gaussian blur.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" blur /* A fast blur will be performed */ />

<Image src="https://picsum.photos/200/300" alt="A random image" blur={100} /* A gaussian blur will be performed */ />
```

#### median

**Type:** `number | boolean`

**Default:** `undefined`

Applies a median filter. This is commonly used to remove noise from images.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" median />

<Image src="https://picsum.photos/200/300" alt="A random image" median={50} />
```

#### rotate

**Type:** `number`

**Default:** `undefined`

Rotate the image by the specified number of degrees.

> **Note:** You can change the background color the empty parts are filled with by setting the [`background`](#background) directive.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" rotate={90} />
```

#### quality

**Type:** `number`

**Default:** `undefined`

All formats (except `gif`) allow the quality to be adjusted by setting this directive.

The argument must be a number between 0 and 100.

> See sharps [Output options](https://sharp.pixelplumbing.com/api-output) for default quality values.

**Code example:**

```astro
<Image src="https://picsum.photos/200/300" alt="A random image" quality={50} />
```

#### fit

**Type:** `"cover" | "contain" | "fill" | "inside" | "outside"`

**Default:** `undefined`

When both `width` and `height` are provided, this directive can be used to specify the method by which the image should fit.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  width={100}
  height={100}
  fit="contain"
/>
```

#### kernel

**Type:** `"nearest" | "cubic" | "mitchell" | "lanczos2" | "lanczos3"`

**Default:** `undefined`

Use this directive to set a different interpolation kernel when resizing the image.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  kernel="lanczos3"
/>
```

#### position

**Type:** `"top" | "right top" | "right" | "right bottom" | "bottom" | "left bottom" | "left" | "left top" | "north" | "northeast" | "east" | "southeast" | "south" | "southwest" | "west" | "northwest" | "center" | "centre" | "cover" | "entropy" | "attention"`

**Default:** `undefined`

When both `width` and `height` are provided AND `fit` is is set to `cover` or `contain`, this directive can be used to set the position of the image.

See sharps [resize options](https://sharp.pixelplumbing.com/api-resize#resize) for more information.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  width={100}
  height={100}
  fit="contain"
  position="attention"
/>
```

<!--
```ts
declare interface FormatOptions {
  format?: format | format[] | [] | null;
  // The image format or formats to generate image sets for. If `format` is set to `null` or `[]`, no image will be generated.

  // **Note:** Passing `[]` or `null` does not necessarily mean that no image will be generated. If `includeSourceFormat` is set to `true`, then the source format and the format specified in the `fallbackFormat` prop will still be generated.
  fallbackFormat?: boolean;
  // The format the browser will fallback to if the other formats are not supported by it. If not provided, the format of the source image will be used.
  includeSourceFormat?: boolean;
  // Whether to generate image set for the source format or not.
  formatOptions?: Record<format, ImageToolsConfigs> & {
    tracedSVG?: PotraceOptions;
    // Check the format type and ImageToolsConfigs & PotraceOptions interfaces for the supported properties
  };
}
``` -->

### `FormatOptions`

The `FormatOptions` interface defines the configuration options supported by the `<Image />` component for generating image sets for different formats.

The formats supported by the `<Image />` component are:

```ts
declare type format =
  | "heic"
  | "heif"
  | "avif"
  | "jpg"
  | "jpeg"
  | "png"
  | "tiff"
  | "webp"
  | "gif";
```

The `<Image />` component supports the following format options:

#### format

**Type:** `format | format[] | [] | null`

**Default:** `["avif", "webp"]`

The image format or formats to generate image sets for. If `format` is set to `null` or `[]`, no _additional_ image set will be generated.

> **Note:** Passing `[]` or `null` does not necessarily mean that no image will be generated. Image sets will still be generated for the source format if `includeSourceFormat` is set to `true` (which is the default value) and for the format specified in the `fallbackFormat` prop (the default value is the source format).

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  format={["webp", "jpg"]}
/>
```

#### fallbackFormat

**Type:** `format`

**Default:** The source format of the image

The format the browser will fallback to if the other formats are not supported.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  format={["webp", "jpg"]}
  fallbackFormat="png"
/>
```

#### includeSourceFormat

**Type:** `boolean`

**Default:** `true`

Whether to generate image set for the source format or not.

**Code example:**

```astro
<Image
  src="/src/images/image.tiff"
  alt="A random image"
  fallbackFormat="png"
  includeSourceFormat={false}
/>
```

#### formatOptions

**Type:** `Record<format, ImageToolsConfigs> & { tracedSVG?: PotraceOptions }`

**Default:** The default values for the all the formats except `tracedSVG` are inherited from the props of the `<Image />` component defined in the [`ImageToolsConfigs`](#imagetoolsconfigs) interface. And for more information on `tracedSVG` see the [`PotraceOptions`](#potraceoptions) interface.

The configuration options for the different formats. The ten supported keys are `heic`, `heif`, `avif`, `jpg`, `jpeg`, `png`, `tiff`, `webp`, `gif` and `tracedSVG`. These configuration options will be respected when generating image sets for different formats. And the `tracedSVG` config options are used when the `placeholder` prop is set to `"tracedSVG"`.

**Code example:**

```astro
<Image
  src="https://picsum.photos/200/300"
  alt="A random image"
  format={["webp", "jpg"]}
  fallbackFormat="png"
  includeSourceFormat={false}
  formatOptions={{
    jpg: {
      quality: 80,
    },
    png: {
      quality: 80,
    },
    webp: {
      quality: 50,
    },
    tracedSVG: {
      background: "#fff",
      color: "#000",
      turnPolicy: "black",
      turdSize: 1,
      alphaMax: 1,
      optCurve: true,
      threshold: 100,
      blackOnWhite: false,
    },
  }}
/>
```

<!-- Color Values and non array values -->

<!--### `PotraceOptions`

The properties described in the `PotraceOptions` interface are the option -->

<!-- Acknowledgements -->

<!-- Investigate what the default values are in ImageToolsConfigs -->
