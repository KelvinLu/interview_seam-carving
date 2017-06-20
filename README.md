# Seam Carving ("the Photoshop question")

## Context and Overview

A simple technique involving some simple dynamic programming and differentiation/convolution can be used to implement content-aware image resizing. We will ask the interviewee to implement this technique (in an extremely constrained context) to shrink several images.

## Question-specific Terminology

- "Text file representation"
    - A CSV-delimited text file containing integers (bounded between 0-255 for RGBa values) and floating numbers (0-1 for intensity values) to represent each pixel in an image.
    - One pixel is represented by a tuple of RGBa values and a pre-calculated intensity value.
- "Energy mask"
    - A CSV-delimited text file that follows the same format as regular text file representations.

## Bootstrapping

List of resources:

- Several color PNG images.
- Energy masks for adjusting cropping.
- Samples for “correctly”-shrunken and “correctly”-cropped images.

## List of tools/scripts:

- A script to convert PNG images to text file representations.
- A script to convert text file representations to PNG images.
- A script to check the syntax/correctness of text file representations.
- A script (or compiled executable) that implements the seam carving over text file representations.

## The Question

### 1 - Introduction

This is your chance to equip your interviewee with the information they need to understand and succeed in implementing a solution.

Introduce the problem to your interviewee.

- Explain how images can be easily represented by mathematical constructs (2D and 3D matrices). Go over the basics of the RGBa colorspace.
- Give the gist of how seam-carving does its work (transforms an image to an energy representation to represent “edges”, does dynamic programming by memoization to find best-cost path through the energy domain).
- Give examples of applications. You might wish to skip around the SIGGRAPH presentation video for visual examples.

Explain the interview question format.

- Let the interviewee know what they are expected and not expected to do
    - No need for error checking. Assume that all images provided to the interviewee’s application are valid.
    - Go through each part of the question, making sure to explain how each part fits into the general problem of seam carving.
- Ensure that this question does not become a “riddle” question.
    - Offer to explain what differentiation and convolution is; build a natural intuition for finding image “edges”.
    - Offer to explain how the dynamic programming works in this context.

Show the interview environment.

- Let the interviewee know that they are free to use any programming language, IDE, and internet resources that they feel comfortable with, as allowed by the interviewee’s machine and appropriate context.
- List out the tools and resources that will be used to check progress along the question.

### 2 - Write a parser for reading text file representations

The interviewee should be comfortable reading files or standard input, storing the data as they see fit. This can be accomplished with basic programming primitives and basic data structures (e.g.; arrays) in a simple and quick manner. This part is expected to be relatively uninteresting and should be done quickly by virtually all interviewees.

Hints:

- They should already be expecting to only implement vertical seam carving from your introduction.
- Suggest they use a data structure that will be friendly for lots of small mathematical operations (a three-dimensional array-of-arrays-of-arrays is perfectly fine).

### 3 - Write a serializer for writing text file representations

The interviewee should be comfortable writing files or standard output in order to implement basic input and output for the text-based problem framework. Again, this can be accomplished in a simple and quick manner. This part is expected to be relatively uninteresting and should be done quickly by virtually all interviewees.

Deliverables:

- Use the parser to load a text file representation into the interviewee’s data structure.
- Use the serializer to recreate the text file representation from the interviewee’s data structure.
- Confirm that the reproduced image is visually identical.
- Ad-hoc crop the image (in whatever manner) within the interviewee’s application. Confirm that the reproduced image is visually sound.

### 4 - Implement an image intensity function

Have your interviewee implement a secondary data structure to store a mapping of a vector-to-scalar transformation (e.g.; image intensity).

It's important to explain to your interviewee the process of normalizing values in matrix applications. The vector-to-scalar transformation should implicitly or explicitly normalize values between zero and one.

For image intensity, one can use this sample function:

```
Y = floor(min(1.0, (
  0.2126 * (R/255.0)^gamma +
  0.7152 * (G/255.0)^gamma +
  0.0722 * (B/255.0)^gamma
)))

    where Y is intensity and gamma ~= 2.2
```

Hints:

- Make sure their function lies in the range [0, 255].
- Have them consider the (vector) domain and (scalar) range of the function.
- Suggest they may use their serializer to produce a single-channel “debugging” image (with 255 alpha), showing their transform at work. This will useful later down the line if the interviewee wishes to debug other mathematical structures, such as the memoized path-cost structure.

Deliverables:

- Produce an image showing the energy transform of some other image. It should resemble the subjects in the original photo!
- Check the transformation result against a known, example image.

### 5 - Implement differentiation/convolution over the energy domain

Have your interviewee implement the Sobel operator, with both horizontal and vertical kernels over their image intensity data structure. The resulting transformation should be that of the magnitude of the convolution of both kernels.

This can be applied with two for-loops. It is okay to just do this, optimizations for arithmetic aren’t really the focus point here.

> Ask your interviewee about “edge cases” in convolution strides and introduce the concept of padding if they do not know about it.

Convolution, in this application, should be done without any padding. The convolution result dimensions should be two pixels less than that of the convolved image. This also has the semantic effect of never cropping or considering the “outermost” border pixels.

Deliverables:

- Produce an image showing the energy-differential transform of some other image. This should clearly show the (grayscale-ish) “edges” of the input image!
- Check the differentiation result against a known, example image.

### 6 - Implement (vertical) seam-finding via dynamic programming

> Ask your interviewee about how they would take on the problem of finding the best seam/path.
>
> Converse about the naïve solution (a series of recursive path cost-additions for each possible path from top-to-bottom) or other classic pathfinding graph algorithms (Dijkstra’s, Bellman-Ford, Floyd-Warshall) and why they may or may not work well.
>
> Hint/lead them towards the dynamic programming approach (memoization with another matrix).
>
> Consider both time and space complexities with your approaches.

Have your interviewee implement a seam “finding” function that operates over the result of the convolution of the energy domain. This involves a simple least-cost pathfinding solution, which is done via memoization (of minimal path costs) and backtracking.

At this particular step, we will constrain the interview question to only consider vertical seam carving (read: seams that go from top-to-bottom). We will also be using 8-connected paths, so one row will have exactly one seam point.

> Ask your interviewee about what values they would initialize their memoization matrix with. How would they implement the concept of infinity in the case of least-cost path-finding algorithms?

Given an input image of dimensions &#8477;<sup>r&#10799;c</sup>, the produced convolution should be that of &#8477;<sup>(r-2)&#10799;(c-2)</sup>. Thus, the vertical or horizontal seam produced as a function of the convolution result will only traverse `r-2` rows or `c-2` columns. Ask your interviewee about what they would do in this edge case and see how they respond; any reasonable solution will do, the simplest is to just pad/copy/extend the edge points.

Deliverables:

- Produce some debugging output to verify that a reasonable seam path is found.

### 7 - Implement (vertical) image-carving

Have your interviewee use the seam path found and remove that seam from the image! Output the image.

Deliverables:

- Produce a seam-carved output image. It must be one pixel less in length (w.r.t. the seam orientation) than the original.

### 8 - Make an end-to-end iterative seam-carving script

Have your interviewee loop their seam carving procedure.

It is okay to hard-code the number of iterations in their code.

It is also okay to iterate over the same image’s matrix representation, but not over the actual file (e.g; the code shouldn't "save" and "reload" the image with each seam).

> Ask your interviewee about failure cases for seam carving! You can easily demonstrate this with the included examples or by grabbing a .PNG image off the internet.

#### Hints:

- The intensity function is a spatially-invariant, local feature. You can also apply seam carving to the intensity transform to avoid unnecessary recomputation.

#### Deliverables:

- Produce a shrunken image, that reasonably preserves the content of the image, via seam-carving.
- Check the transformation result against a known, example image.

### Bonus Rounds

- Bonus #1:
    - Write another or modify an existing parser to read a “energy mask” file
    - Use the “energy mask” file to modify the energy representation of an image, implementing selective cropping
- Bonus #2:
    - Implement end-to-end seam carving horizontally
- Bonus #3:
    - Use seam carving to extend (instead of shrinking) images
- Bonus #4:
    - Replace the energy and differentiation transform with another cost function (such as taking the Euclidean distance between points in RGB colorspace)
