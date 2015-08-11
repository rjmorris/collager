# collager

collager is a Javascript library for computing the size and position of images to produce a collage arrangement inside a fixed-size container. (An image collage may also be called an image gallery, image wall, or any of several other names.) collager fits an image collection into a container under the following assumptions:

- Images may be resized as long as their aspect ratios are preserved.
- The image sort order may be altered.
- Images must not be cropped.
- Extra whitespace is allowed in the container if necessary to meet the other assumptions.

collager isn't an all-in-one solution for creating a collage or image gallery. It doesn't add captions, transitions, or lightbox effects, and it doesn't even place the images on the page. It merely computes a new size for each image and where it should be placed. All the rest is up to you, using the tools of your choice.

collager produces two arrangements. One is a "regular" arrangement where the images line up nicely with equal spacing between them. Another is a "perturbed" arrangement, where the images are randomly displaced from their regular positions.

One goal of the project is to achieve the optimal collage arrangement (where optimal is defined as the least amount of wasted space), but this isn't guaranteed in the current version. The current version first makes an initial pass through the images, placing them using a simple algorithm. Then it repeatedly swaps random pairs of images, keeping the swaps that improve the arrangement and discarding those that don't.

Even though the library was written with images in mind, it will work on any array of objects as long as they have a defined width and height. For example, you could use it to compute the positions for a collage of `<div>` elements.

## Dependencies

collager uses several utility functions from [d3](http://d3js.org), so you must include d3 to use collager.

## Example

First, import collager and d3 into your document, either within the document's `<head>` or at the end of its `<body>`:

    <script src="third-party/d3-3.5.6/d3.min.js"></script>
    <script src="third-party/collager-1/collager.js"></script>

Somewhere in your Javascript code, define an array of images (or other objects):

    var images = [
        { path: 'images/a.png', width: 180, height: 260 },
        { path: 'images/b.png', width:  80, height: 130 },
        { path: 'images/c.png', width: 240, height: 180 },
        { path: 'images/d.png', width: 320, height: 150 },
        { path: 'images/e.png', width: 220, height: 220 },
        { path: 'images/f.png', width: 100, height: 240 }
    ];

Call collager's `arrange()` method to assign each image a position within a 400x300 pixel container:

    collager().arrange({
        objects: images,
        containerWidth: 400,
        containerHeight: 300
    });

`arrange()` adds new properties to each image object, which now look something like this:

    {
      path: 'images/a.png',
      width: 180,
      height: 260,
      collagerWidth: 76.9811320754717,
      collagerHeight: 111.19496855345913,
      collagerTop: 165.04081359561084,
      collagerLeft: 88.42767295597483,
      collagerTopPerturbed: 167.51688728089744,
      collagerLeftPerturbed: 97.92943204457302
    },

Finally, use the new collager properties to create the image elements within the container. This example does so using d3:

    d3.select('.collage-container')
        .style('position', 'relative')
        .selectAll('img')
        .data(images)
        .enter()
        .append('img')
        .attr('src', function(d) { return d.path; })
        .attr('width', function(d) { return d.collagerWidth; })
        .attr('height', function(d) { return d.collagerHeight; })
        .style('position', 'absolute')
        .style('top', function(d) { return d.collagerTop + 'px'; })
        .style('left', function(d) { return d.collagerLeft + 'px'; })
    ;

## API

Create a collager object by calling `collager()`. The collager object supports the following properties and methods:

### version

Return the current collager version.

### arrange(obj)

Arrange a set of objects into a collage formation. `obj` is a required argument. It must be an object containing the following keys:

- `objects`: An array of objects, each of which must contain a `width` and `height` property. Note that `arrange()` will modify this array by sorting it and by inserting properties into each object.
- `containerWidth`: The width of the container into which the collage should fit.
- `containerHeight`: The height of the container into which the collage should fit.

The `obj` argument may optionally contain the following keys:

- `numRows`: Number of rows to split the objects across. If `null`, `arrange()` will determine the best number of rows. (default: `null`)
- `hPaddingInner`: Amount of padding to insert between images in a row. (default: 20)
- `vPaddingInner`: Amount of padding to insert between rows. (default: 20)
- `hPaddingOuter`: Amount of padding to insert between the left/right boundaries of the container and the first/last image of each row. (default: 0)
- `vPaddingOuter`: Amount of padding to insert between the top/bottom boundaries of the container and the first/last row. (default: 0)
- `perturbRate`: When randomly perturbing the final positions of the objects, proportion of the padding area eligible to contain the perturbed position. Higher values lead to a greater degree of perturbation. A value of 0.5 produces the greatest degree of perturbation while still guaranteeing that the perturbed positions don't produce overlapping objects. (default: 0.5)
- `optimizeSteps`: Number of attempts to improve the object placement after making an initial guess. Higher values lead to more optimal arrangements at the cost of runtime. (default: 100)

After calling `arrange()`, the following new properties will be added to each object in the `objects` array:

- `collagerWidth`: Scaled width of the object in the collage arrangement.
- `collagerHeight`: Scaled height of the object in the collage arrangement.
- `collagerTop`: Top position of the object in the collage arrangement, relative to its container.
- `collagerLeft`: Left position of the object in the collage arrangement, relative to its container.
- `collagerTopPerturbed`: Top position of the object after its position has been randomly moved up or down.
- `collagerLeftPerturbed`: Left position of the object after its position has been randomly moved left or right.
