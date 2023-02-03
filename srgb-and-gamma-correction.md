# sRGB and gamma correction

Today, most images and monitors use the sRGB color space. It is convenient that they use the same color space, since it means that the RGB values from an image can be fed directly to the display. However, there is a downside to using sRGB which not enough people realize: sRGB encodes the RGB components in a nonlinear way to provide a set of colors that is perceptually uniform.
![Encoding vs linear intensity](./images/intensities.png "Encoding vs linear intensity" =606x118)

The fact that sRGB is a nonlinear color space means that adding or multiplying sRGB colors is **wrong**. This means that things like linear interpolation, fade-outs, and bilinear filtering are wrong when they are done on sRGB colors. They should be done only on *linear* sRGB color representations.

It is not always noticeable when sRGB colors are used incorrectly, but the effect is obvious in some edge cases. For example, consider this example where I places a red circle onto a green background and applied a CSS blur filter:
![Comparison of incorrect vs correct gamma](./images/incorrect.png "Comparison of incorrect vs correct gamma" =256x256)

If you think the blurred edges of the circle look too dark, you are right. When I make the same picture in GIMP, it looks like this:
![Comparison of incorrect vs correct gamma](./images/correct.png =256x256)

Usually, the difference is more subtle. In general, images with correct gamma handling have softer shadows and less intensive highlights. For example, consider the following example from [the "Gamma-correct lighting" article on Wolfire.com](http://blog.wolfire.com/2010/02/Gamma-correct-lighting):
![Comparison of incorrect vs correct gamma](./images/incorrect_gamma_comparison.jpg "Comparison of incorrect vs correct gamma" =2560x1024)

The following, more pronounced example is from [the article about gamma correction on learnopengl.com](https://learnopengl.com/Advanced-Lighting/Gamma-Correction):
![Comparison of incorrect vs correct gamma](./images/gamma_correction_srgbtextures.png "Comparison of incorrect vs correct gamma" =800x313)


## Gamma correction

If we want to apply any transformation, we should convert the RGB of sRGB colors to *linear* RGB components. This is also known as [gamma correction](https://en.wikipedia.org/wiki/Gamma_correction) and is done by applying the conversion
$$ C_\text{linear} = f(C_\text{sRGB}) $$

where
$$ f(x) = \begin{cases} \frac{x}{12.92} & \text{if $x \in [0, 0.04045]$} \\ \left( \frac{C_\text{sRGB} + 0.055}{1.055} \right)^{2.4} & \text{if $x \in (0.04045, 1]$} \end{cases} $$

This conversion should be done to each of the R, G, and B components. If there is an alpha component it should *not* be converted.

Before displaying the colors, we should convert the linear sRGB color back to a proper sRGB representation by applying the inverse transformation
$$ C_\text{sRGB} = g(C_\text{linear}) $$

with
$$ g(x) = \begin{cases} 12.92 x & \text{if $x \in [0, 0.0031308]$} \\ 1.055 x^{\frac{1}{2.4}} - 0.055 & \text{if $x \in (0.0031308, 1]$} \end{cases} $$

This is a an odd transformation. It is chosen to be very close to the mapping $y = x^{2.2}$ but to be linear for very dark values. So $C_\text{sRGB} \approx (C_\text{linear})^{2.2}$ so that $C_\text{linear} \approx (C_\text{sRGB})^{\frac{1}{2.2}}$. As a simplification or optimization, you could use this mapping as well, and get very close results (although the results might be off for small values).

If you have a graphics application and you're not sure if it uses gamma correctly, I'd first suggest to investigate if you happen to use a library or framework that already handles gamma for your. In particular, OpenGL has support for sRGB as outlined in [the article about gamma correction on learnopengl.com](https://learnopengl.com/Advanced-Lighting/Gamma-Correction).

If you really need to do it yourself, I'd suggest to transform the input images to linear floating point values when you load them. Then, you perform one final conversion back to sRGB before displaying or saving the picture.
