# Research, define and expand on, current deficiencies in VTT. 

by [Michael Duggan](https://twitter.com/mickduggan)

## Introduction

VisualTrueType 6.35 includes support to handle all aspects of hinting Variable fonts. There are however some areas where the code output from the autohinter does not work optimally. There is also limited support in the Autohinter for Non Latin fonts.

The following notes are intended to open up discussion among the wider type community as well as to serve as pointers for any future open source development work on the VTT Autohinter. All of the areas of our discussion on hinting variable fonts are where I spend the time, both refining the code and checking for consistent and reliable code out. The eventual goal is to hopefully achieve a greater level of consistency, correctness and confidence in the code output, resulting in less time spent making manual adjustments to the code. The notes are broken out into a few rough categories.

**Existing bugs or unexpected behaviours while using VTT 6.35**

**Ideas for improvements to the current VTT Hinting visual hinting interface**

**Ideas, notes and code for future hinting of Variable fonts**

_These ideas will require more discussion and full testing. Once they are fully developed, and tested, they can be documented and made available for use in hinting Variable fonts. These ideas cover areas of hinting variable fonts._ 

**Not currently possible or supported in VTT** 

**Supported but needs deveopment work to refine to fit into the current hinting workflow**

_- Addition of Global deltas to cvt’s for variation instances or ranges_

_- Function to centre middle bars_

**Future ideas for VTT autohinter development work** 

**New ideas for approaches to hinting for modern rendering environments**
 
*All notes are based on the publicly available current version of Visual TrueType 6.35, running on Surface Laptop  / Windows 11

 
## Accent positioning and hinting 

**Note: bug / suggestion**

The current version of the VTT Autohinter outputs positioning code for both x and y direction positioning of accents above base glyphs. 

**X-Positioning code**

The autohinter has an option to disable all x-direction hinting for the entire font under (settings insert exact ui path). However the x-positioning code for accents is still generated. 

**Recommendation:** Disable x-positioning code output from Autohinter for accents positioning in x-direction [SVTCAX]

The code refers to a function (note number) which results in badly position accents in Variable fonts

[Insert graphic] with x-code v no x-code [note also existing script to deal with this, and link out to repo by Chris]

**Y-Positioning code**

The autohinter also generated y-axis positioning code to position accents above base glyphs.

[add discussion and text from Hinting Variable fonts, describing the problem]

**Current solution**
[Insert graphic] with y-code v custom y-code

Show example [ insert graphics and code explanations as in VTT Hinting Variable fonts]

**Recommendation:** Refine function to output corrected positioning code. 

**Recommendation:**  Future Autohinter development 

Better default Autohinting of accents

The autohinter does not use any special method to when adding hinting to base unique accent glyphs. The result is often glyph shapes that collapse to one pixel in height, rendering the accents at smaller screen sizes unreadable. The base set of accents in a typical font, are used in many composite glyphs, in some case as much as half the glyphs set. The work to fine tune and make readable accents is critical in achieving a well hinted fonts. 

Refine the autohinter appraoch for hinting accents, smarter auto hinting, add new rules  based on current best practices and approachs to hinting accents (ie always maintain a minimum distance of two pixels etc] 



## Middle bar centering

**Use case**

In Type design it is common practice to design the middle bar of the H and other characters such as E F, to be optically centred rather than mathematically centred. There are many glyphs in a typical Latin font, including glyphs in the Latin, Greek and Cyrillic designs that share this design characteristic.

A middle bar that is designed to be mathematically centred will appear optically too low. This optical efffect is adjusted for in the high resolution design, and in the case of the H bar for example, the bar is moved up slightly so that it _‘appears’_ to be centered.

One of the benefits of hinting Variable fonts is one simple set of hints can be applied to all variations in the font. This approach however has drawbacks. Once the hinting code had been added, and compiled, there are not many other options to adjust any less than ideal visual results across the variation space.

Lets have a look at the following example of the Hinting for the H in Google Sans Variable. The middle bar of the H has already been designed to be slightly higher than the mathematical centre, and this works well for high resolution output such as in print or on higher resolution screens. For lower resolutions, screens, where there are less pixels, rounding issues in different weights across the variation design space can cause the middle bar to appear too low at certain sizes and in certain weights variations. 

There are only a couple of ways to approach the hinting for this glyph

<img width="100%" height="100%" src="Images/LOW11GSH.png">

1. YIP Anchor the bottom point of the middle bar between the grid fitted points on the baseline and Cap height, and control the weight of the bar vertically from the bottom to the top of the bar.

<img width="100%" height="100%" src="Images/LOW12GSH.png">

2. YIP Anchor the top point of the middle bar between the grid fitted points on the baseline and Cap height, and control the weight of the bar vertically from the top to the bottom of the bar. [insert graphic]

Both of these approaches result in the middle bar appearing too low at certain sizes in certain weight instances.

When hinting static fonts, whenever rounding causes the bar to appear too low, an _inline delta command_ can be added to adjust the position of the bar at a specific size to set it to correctly appear centred. In variable fonts it is not possible to add delta commands as there is no one command that will work for all weight variations.

**Middle Bar centering Function**

Another approach to solve this problem is to use an already existing function [insert number] This function is supported in the current Font program generated by VTT when autohinting a font. 

Let’s have a look at how it works. [Insert graphics and explantation of the code and visual example of how it solves the problem]

<img width="100%" height="100%" src="Images/CENTER12.png">

<img width="100%" height="100%" src="Images/CENTER18.png">

Using this method and function results in a middle bar that is optically and visually centred accross all weight variations and sizes. One effect of this approach is that one side of the middle bar does not always fall on a sharp pixel boundary. The result of this is slightly more blur in the rendering of the horizontal bar. However this may be better for the visual output, than having the middle bar appearing to low or too high.

The Function [insert function number] supports two methods of use

1. Uses a cvt to control the weight of the middle bar

2. uses a dist to control the weight of the middle bar

Unfortunately neither of these methods fit into the current workflow and methods for hinting Variable fonts. Cvt’s are not typically used any longer to control the weights of  vertical features (ie: horizontal bars) 

The second method that uses a dist to control the weight is also not suitable for all variable fonts, especially any that support lighter weight variations. The dist command by its nature, always rounds to a full pixel. This causes too much distortion in lighter weights and is not recommend for use.

[Insert graphic example]

**Recommendation**

Refine the current function to support ’shift’ to control the weight of the vertical bar feature, while keeping the centering aspect of the function. Document the function and make it available for hinting Variable fonts. 