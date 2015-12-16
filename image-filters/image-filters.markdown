title: Making Your Own Image Filters
author: Cate Huston

_Cate Huston is a developer and entrepreneur focused on mobile. She’s lived and worked in the UK, Australia, Canada, China and the United States, as an engineer at Google, an Extreme Blue intern at IBM, and a ski instructor. Cate speaks internationally on mobile development and her writing has been published on sites as varied as Lifehacker, The Daily Beast, The Eloquent Woman and Model View Culture. She co-curates Technically Speaking, blogs at Accidentally in Code and is @catehstn on Twitter._

## A Story of a Brilliant Idea That Wasn’t All That Brilliant

In Chinese art, there is often a series of four paintings showing the same
place in different seasons. Color - the cool whites of winter, pale hues of
spring, lush greens of summer, and red and yellows of fall is the
differentiation. Sometime around 2011, I had what I thought was a brilliant
idea. I wanted to be able to visualize a photo series, as a series of colors. I
thought it would show travel, and progression through the seasons.

I didn’t know how to calculate the dominant color from an image, and I thought
about scaling the image down to a 1x1 square and seeing what was left, but that
seemed like cheating. I knew how I wanted to display it though, in a layout
called the [Sunflower
layout](http://www.catehuston.com/applets/Sunflower/index.html). It’s the most
efficient way to layout circles.

I left this project for years, distracted by work, life, travel, talks.
Eventually I returned to it, and figured out how to calculate the dominant
color and finally [finished my
visualization](http://www.catehuston.com/blog/2013/09/02/visualising-a-photo-series/).
And that is when I discovered that this idea wasn’t, in fact, brilliant.
Because the progression wasn’t as clear as I hoped, the dominant color
extracted wasn’t generally the most appealing shade, the creation took a long
time (a couple of seconds per image), and required hundreds of images to make
something cool.

You might think this would be discouraging, but by the time I had got to this
point I had learned so many things that hadn’t come my way before - about color
spaces, pixel manipulation, and I had started making these cool partially
colored images, of the kind you find on postcards of London - the red bus, or
phone booth, but everything else is in gray-scale.

I used a framework called [Processing](https://processing.org/) because I was
familiar with it from developing programming curricula, and because I knew it
made it really easy to create visual applications. It’s a tool originally
designed for artists, so it abstracts away much of the boilerplate. It allowed
me to play, and to experiment.


University, and later work, had filled up my time with other people’s ideas and
priorities. Part of "finishing" this project was learning how to carve out time
to make progress on my own ideas in the time that was left to me. I calculated
this to be about 4 hours of good mental time a week. A tool that allowed me to
move faster was therefore really helpful, even necessary. Although it came with
it’s own set of problems, especially around writing tests. I felt thorough
tests were especially important for validating how it was working, and for
making it easier to pick up and resume a project that was often on ice for
weeks, even months at a time. Tests (and blogposts!) formed the documentation
for this project. I could leave failing tests to document what should happen
that I hadn’t quite figured out yet, make changes with more confidence that if
I changed something that I had forgotten was critical, the tests would remind
me.

This chapter will cover some details about Processing, talk you through color
spaces, decomposing an image into pixels and manipulating them, and unit
testing something that wasn’t designed with testing in mind. But I hope it will
also prompt you to go and make some progress on whatever idea you haven’t made
time for lately, because even if your idea turns out to be as terrible as mine
was, you may make something cool, and learn something fascinating in the
process.

## The App

This chapter will show you how to create your own image filter application
using Processing (a programming language and development environment built on
Java, used as a tool for artists to create with), where you can load in your
digital images and manipulate them yourself using filters that you create.
We’ll cover aspects of color, setting up the application in Processing, some of
the features of Processing, how to create color filters (mimicking what used to
be used in old-fashioned photography) and also a special kind of filter that
can only be done digitally - extracting the dominant hue from an image, and
showing or hiding it, to create eerie partially colored images.

We’ll also be adding a thorough test suite, and covering how to handle some of
the limitations of Processing when it comes to testability.

## Background

Today we can take a photo, manipulate it, and share it with all our friends in
a matter of seconds. However, a long long time ago (in digital terms anyway),
it used to be a process that would take weeks.

We would take the picture, then when we had used a whole roll of film, we would
take it in to be developed (often at the pharmacy), pick it up some days later,
and then discover that there was something wrong with many of the pictures -
hand not steady enough? Random person/thing that we didn’t remember seeing at
the time. Of course by then it was too late to remedy it.

Then next time we had friends over, we could show them our carefully curated
album of that trip we took, or alternatively, just tip the pictures out of the
shoebox we were keeping them in, onto the coffee table.

The process that would turn the film into the picture was one that most people
didn’t understand. Light was a problem, so you had to be careful with the film.
There was some process featuring darkened rooms and chemicals that they
sometimes showed bits of in films or on TV.

Which actually sounds familiar. Few people understand how we get from the point
and click on our smartphone camera to an image on instagram. But actually there
are many similarities.

### Photographs, the Old Way

Photographs are created by the effect of light on a light sensitive surface.
Photographic film is covered in silver halide crystals (extra layers are used
to create color photographs - for simplicity let’s just stick to
black-and-white photography here.) 

When talking an old fashioned style photograph - with film - the light hits the
film according to what you’re pointing at, and the crystals at those points are
changed in varying degrees - according to the amount of light. Then, the
[development
process](http://photography.tutsplus.com/tutorials/step-by-step-guide-to-developing-black-and-white-t-max-film--photo-2580)
converts the silver salts to metallic silver, creating the negative. The
negative has light and dark areas of the image inverted to their opposite. Once
the negatives have been developed, there is another series of steps that
reverse the image back and print it.

### Photographs, the Digital Way

When taking pictures using our smartphones or digital cameras, there is no
film. There is something called an Active Pixel Sensor which functions in a
similar way. Where we used to have silver crystals, now we have pixels - tiny
squares (in fact, pixel is short for "picture element"). Digital images are
made up of pixels, and the higher the resolution the more pixels there are.
This is why low-resolution images are described as "pixelated" - you can start
to see the squares. These pixels are just stored in an array, which the number
in each array "box" containing the color.

In FIXME Figure 1, we see some brightly colored blow up animals taken at MOMA
in NYC at high resolution. FIXME Figure 2 is the same image blown-up, but with
just 24 x 32 pixels.

See how it is so blurry? The lines of the animals aren’t as smooth? We call
that pixelation, which means the image is too big for the number of pixels it
contains, and the squares become visible. Here we can use it to get a better
sense of an image made up of squares of color.

What do these pixels look like? If we print out the colors of some of the
pixels in the middle (10,10 to 10,14) using the handy `Integer.toHexString` in
Java, we get hex colors:

```
FFE8B1
FFFAC4
FFFCC3
FFFCC2
FFF5B7
```

Hex colors are 6 characters long. The first two are the red value, the second
two the green value, and the third two the blue value. Sometimes there are an
extra two characters which are the alpha value. In this case `FFFAC4` means:

- red = FF (hex) = 255 (base 10)
- green = FA (hex) = 250 (base 10)
- blue = C4 (hex) = 196 (base 10)

## Running The App 

In FIXME Figure 3, we have a picture of our app running. It’s very much
developer-designed, I know, but we only have 500 lines of Java here so
something had to suffer! You can see the list of commands on the right. Some
things we can do:

- Adjust the RGB filters.
- Adjust the “hue tolerance”.
- Set the dominant hue filters, to either show or hide the dominant hue.
- Apply our current setting (it is infeasible to run this every key press).
- Reset the image.
- Save the image we have made.

We’re using Processing, as it makes it really simple to create a little
application, and do image manipulation. Processing is an IDE (Integrated
Development Environment) and a set of libraries to make visual applications. It
was originally created as a way for designers and artists to create digital
apps, so it has a very visual focus. 

We’ll focus on the Java-based version although Processing has now been ported
to other languages, including Javascript, which is awesome if you want to
upload your apps to the internet.

For this tutorial, I use it in Eclipse by adding core.jar to my build path. If
you want, you can use the Processing IDE, which removes the need for a lot of
boilerplate Java code. If you later want to port it over to Processing.js and
upload it online, you need to replace the file chooser with something else.

There are detailed instructions with screenshots in the project's
[repository](https://github.com/aosabook/500lines/blob/master/image-filters/SETUP.MD).
If you are familiar with Eclipse and Java already you may not need them.

## Processing Basics

### Size and Color

We don’t want our app to be a tiny grey window, so the two essential methods
that we will start by overriding (implementing in our own class, instead of
using the default implementation in the superclass, in this case `PApplet`) are
[`setup()`](http://processing.org/reference/setup_.html), and
[`draw()`](http://processing.org/reference/draw_.html). `setup()` is only
called when the app starts, and is where we do things like set the size.
`draw()` is called for every animation, or after some action can be triggered
by calling `redraw()` (as covered in the Processing Documentation, `draw()`
should not be called explicitly). 

Processing is designed to work nicely to create animated sketches, but in this
case we don’t want animation[^noanim], we want to respond to key presses. To prevent
animation (this would be a drag on performance) we will want to call
[`noLoop()`](http://www.processing.org/reference/noLoop_.html) from setup. This
means that `draw()` will only be called immediately after `setup()`, and
whenever we call `redraw()`.

[^noanim]: If we wanted to create an animated sketch we would not call
`noLoop()` (or, if we wanted to start animating later, we would call `loop()`).
The frequency of the animation is determined by `frameRate()`.

```java
    private static final int WIDTH = 360;
    private static final int HEIGHT = 240;

    public void setup() {
        noLoop();

        // Set up the view.
        size(WIDTH, HEIGHT);
        background(0);
    }
        
    public void draw() {
        background(0);
    }
```

[^noanim]: If we wanted to create an animated sketch we would not call noLoop()
(or, if we wanted to start animating later, we would call loop()). The
frequency of the animation is determined by frameRate().

These don’t really do much yet, but run the app again adjusting the constants
in `WIDTH` and `HEIGHT` to see different sizes.

`background(0)` specifies a black background. Try changing the number passed
into `background()` and see what happens - it’s the alpha value, and so if you
only pass one number in, it is always greyscale. Alternatively, you can call
`background(int r, int g, int b)`.
