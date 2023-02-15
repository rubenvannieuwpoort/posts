# Compositing

Compositing is combining multiple (possibly partially transparent) layers of image data into a single image. If you have ever used Photoshop or GIMP, compositing is how all the layers are combined into a final image that can be displayed on screen.

The image data is assumed to be saved in the form of a grid of pixels, so that compositing can be done per pixel. In this post I will derive the composition equation, which describes how to compose one pixel onto another.


## Notation

I will write a pixel $P$ as
$$ P = ((R, G, B), \alpha) $$

$R$, $G$, $B$, and $\alpha$ are values in $[0, 1]$. The RGB components should correspond to the color of the pixel, expressed in a linear color space. This is important: as I wrote before, the default color space sRGB is not linear, so it needs to be converted! The alpha component corresponds to the opacity of the pixel, if $\alpha = 0$ the pixel is fully transparent (e.g. invisible) and if $\alpha = 1$ it is fully opaque (e.g. if it is composited on top of another pixel, that other pixel will not be visibly anymore).

I will often write $C = (R, G, B)$ because in the equations we don't need to distinguish between the individual RGB components. The composition of a pixel $X$ on top of a pixel $Y$ is denoted as $X \rightarrow Y$, and the RGB and alpha components of a pixel $X$ are denoted as $C_X$ and $\alpha_X$.


## The compositing equation

First, we consider the case where we composite a pixel $X$ on a pixel $Y$ which is opaque (so $\alpha_Y = 1$). If $\alpha_X = 1$ we expect the resulting color to be equal to $C_X$, and if $\alpha_X = 0$ we expect the resulting color to be equal to $C_Y$. Assuming that the transition is linear we derive
$$ \begin{equation} \begin{aligned} \alpha_{X \rightarrow Y} &= 1 \\ C_{X \rightarrow Y} &= \alpha_X C_X + (1 - \alpha_X) C_Y \end{aligned} \end{equation} $$

The generic case is a bit more complicated. We start with the alpha component. Physically, transparent object lets a certain fraction of light pass through it. If we have two objects on top of each other, and the first object lets a fraction $(1 - \alpha_X)$ of light through, and the second lets a fraction $(1 - \alpha_Y)$ of light through, we expect a fraction of $(1 - \alpha_X)(1 - \alpha_Y)$ to go through both objects. Applying the same principles, we expect that $1 - \alpha_{X \rightarrow Y} = (1 - \alpha_X)(1 - \alpha_Y)$. More elegantly put,
$$ \alpha_{X \rightarrow Y} = \alpha_X + \alpha_Y - \alpha_X \alpha_Y $$

For the RGB components it's not clear where to start. It turns out to be informative to evaluate the RGB components of $X \rightarrow Y \rightarrow Z$ where $Z$ is opaque. We can evaluate this as $(X \rightarrow Y) \rightarrow Z$ or as $X \rightarrow (Y \rightarrow Z)$. We find
$$ C_{(X \rightarrow Y) \rightarrow Z} = \alpha_{X \rightarrow Y} C_{X \rightarrow Y} + (1 - \alpha_{X \rightarrow Y})C_Z $$

and
$$ \begin{aligned} C_{X \rightarrow (Y \rightarrow Z)} &= \alpha_X C_X + (1 - \alpha_X) C_{Y \rightarrow Z} \\ &= \alpha_X C_X + (1 - \alpha_X)(\alpha_Y C_Y + (1 - \alpha_Y) C_Z) \end{aligned} $$

Since there really is just one result of compositing three pixels, we expect that both orders of evaluation give the same result (in mathspeak, the composition operator $\rightarrow$ is [associative)[https://en.wikipedia.org/wiki/Associative_property]). That is, $(X \rightarrow Y) \rightarrow Z = X \rightarrow (Y \rightarrow Z)$.

Now, setting $C_{(X \rightarrow Y) \rightarrow Z} = C_{X \rightarrow (Y \rightarrow Z)}$ and simplifying a bit using the formula for $\alpha_{X \rightarrow Y}$ we found before, we find
$$ \alpha_{X \rightarrow Y} C_{X \rightarrow Y} = \alpha_X C_X + (1 - \alpha_X) \alpha_Y C_Y $$

This finally gives us the full, generic composition equation
$$ \begin{equation} \begin{aligned} \alpha_{X \rightarrow Y} &= \alpha_X + \alpha_Y - \alpha_X \alpha_Y \\ C_{X \rightarrow Y} &= \frac{\alpha_X C_X + (1 - \alpha_X) \alpha_Y C_Y}{\alpha_{X \rightarrow Y}} \end{aligned} \end{equation} $$


## Premultiplied alpha

It is worth noting that the composition equation we derived in the last section contains a division. This is not ideal as divisions are very expensive to compute. The astute reader might have noticed that for any pixel we really only use the product of the RGB components and the alpha component. So instead of storing the RGB components, we can store the product of the RGB components and the alpha component (we still need to store the alpha component itself since it is needed in the composition equation).

This technique is known as **premultiplied alpha** or **associated alpha** (and not using it is sometimes called **straight alpha** or **unassociated alpha**). It makes computing compositions much nicer; instead of computing $C_{X \rightarrow Y}$ we compute $\alpha_{X \rightarrow Y} C_{X \rightarrow Y}$ and the composition equation becomes

$$ \begin{equation} \begin{aligned} \alpha_{X \rightarrow Y} &= \alpha_X + \alpha_Y - \alpha_X \alpha_Y \\ \alpha_{X \rightarrow Y} C_{X \rightarrow Y} &= \alpha_X C_X + (1 - \alpha_X) \alpha_Y C_Y \end{aligned} \end{equation} $$

It is worth noting that using premultiplied alpha throws away information in the color components for which $\alpha \neq 1$. So you should never store images with premultiplied alpha if you intend on making changes to the alpha channel. So in the context of a project, it's probably best to have images with straight alpha under version control, and then convert them to premultiplied alpha as a build step. Alternatively, your application can convert to premultiplied alpha when it imports images (this is also a good place to convert to a linear color space, if it is necessary).


## Sources

[1] https://ciechanow.ski/alpha-compositing/

[2] https://en.wikipedia.org/wiki/Alpha_compositing