# Research, define and expand on, current deficiencies in VTT. 

by [Michael Duggan](https://twitter.com/mickduggan)

## Introduction

VisualTrueType 6.35 includes support to handle all aspects of hinting Variable fonts. There are however areas where the code output from the autohinter does not work optimally. There is also very limited support in the Autohinter for a comprehensive solution for hinting complex script fonts.

The following notes are intended to open up discussion among the wider type community as well as to serve as pointers for any future open source development work on the VTT Autohinter. The areas discussed below regarding hinting variable fonts, are where I spend the most time, both refining the code and checking for consistent and reliable code output. The eventual goal is to achieve a greater level of consistency, global support for scripts other than Latin, as well as correctness and confidence in the Autohinter code output, resulting in less time spent making manual adjustments. The notes are broken out into a few rough categories.

**1. VTT 6.35 bugs and unexpected behaviours. Suggested ideas for improvements**

**2. Ideas, and code for future hinting of Variable fonts**  

_These ideas are currently supported but need full development work and testing, to refine, and fit into the autohinter and hinting workflow_

_- Addition of Global deltas to cvt’s for variation instances or ranges_

_- Function to centre middle bars_

_- Accent positioning and hinting_

_The ideas require full discussion, development work and full testing. Once fully developed, and tested, they can be documented and made available for use in hinting Variable fonts. These ideas discussed specifically reference hinting variable fonts._ 

**3. Future ideas for VTT autohinter development work** 

_Built in support in the autohinter for scripts other than Latin_

**4. New ideas for approaches to hinting Variable fonts targeting modern rendering environments**
 
_All notes are based on the publicly available current version of Visual TrueType 6.35, running on Surface Laptop  / Windows 11_

## VTT 6.35 Bugs / Suggested improvements

_Bugs and suggestions for improvements to the VTT GUI and Autohinter_

## VTT 6.35 Bugs

**Compile VTT talk via menu option, not working**

Changes made manually to the code in the VTT Talk window, are not compiled when using the menu option Tools > Compile > VTT talk. 

**Repo:** Open a font in VTT. Add autohinting. Make a manual edit to the VTT Talk code. Use the menu option to compile the changes via, Tools > Compile > VTT talk

**Expected:** Code should compile. The usual and faster method of using CTRL R (Compile) is working as expected

**X-Positioning composite code is broken in Variable fonts**

The autohinter has an option to disable all x-direction hinting for the entire font under _(Tools > options > Autohinter Tab > Disable X-Direction Hints)_  X-positioning code for accents is still generated however when this option is set.

**Recommendation:** Disable x-positioning code output from Autohinter for accent positioning in x-direction [SVTCAX].

**Importing xml files into larger fonts showing spinning disk**

Importing an xml file into a larger font, shows a spinning disk. It appears that the application has hung, with no indication of progress.

**Repo:** Open a larger font in VTT. (Inter Variable font) Add autohinting. Export XML file File > Export > All code to XML. Make a change to the xml file. Import file, File > Import > All code from XML. For the Inter Variable font example the code can take up to 1 + minutes, and looks like the application has hung. For larger fonts such as any CJK, it is difficult to know how long this action will take. 

**Recommendation:** Show progress bar for import of xml.


## VTT Suggested improvements

**Compile everything for all glyphs and save on Import of XML file**

When changes are made to an exported XML file, and the XML file is imported in VTT, the code needs to be compiled. Tools > Compile > Everything for all glyphs. Add option in the VTT UI, to Import, Compile everything for all glyphs and save. 

**Change the Default output from the Autohinter from ‘ResYDist’ to ‘YShift’ for Variable font hinting**

The current recommendation for hinting Variable fonts is to replace all ‘ResYDist’ instructions with a ‘YShift’ instruction. Using the ‘YShift’ command in place of the ‘ResYDist’ for controlling stem weights, helps to render the outlines of all weight Variations from lightest to heaviest, using a more natural rendering from the outline. _The shift command is typically used to constrain a distance in the y-direction between two points, when no CVT is used or needed (i.e. when a distance is too small to warrant using a CVT to control the distance)_

The ‘ResYDist’ command, causes stems to round to full pixels at smaller sizes. This causes distortions, particularly in lighter outlines. Using the YShift command in place of ResYDist corrects these distortions and renders the outlines more accurately.

**Current solution:** Export all hinting code from the font to XML, edit the XML in an external text editor, replace all ResYDist references with YShift, re-import XML back to your original font, with modifications, or, use python script found [here](https://github.com/source-foundry/vtt-hinting-scripts) 

**Recommended solution:** Modify the Autohinter to output the YShift command, by default, when hinting Variable fonts. 

**Investigate adding support to add RES commands via the Hinting GUI in VTT**

The Res addition to the command ResYAnchor, for example, stands for _Rendering Environment Specific,_ and ensures that the appropriate rounding happens, for various rendering environments. This saves adding additional Hinting commands if Hinting is required to work in a variety of rendering environments, and is recommended when hinting Variable fonts. _The Res command calls a Function, that is designed to also allow for more subtle rendering of features such as undershoots and overshoots_

**Current VTT Support:** The ‘Res’ (Resolution Environment Specific) command is not supported when using the Graphical Hinting tools. This additional code must be added manually, in the VTTtalk window.

**Recommended solution:** Investigate adding support to add RES commands via the Hinting GUI in VTT. When rehinting any existing glyph from scratch, for example, having the ability to add the RES commands via the hinting GUI saves time, and results in an improved workflow. Currently the RES commands can only be added manually to the VTT Talk, after the glyph has been hinted using the Graphical Hinting tools. 

**GASP Table recommendations for Variable fonts**

When following the recommendations for hinting variable fonts described [here](https://github.com/googlefonts/how-to-hint-variable-fonts) the following ‘gasp’ table settings are recommended and apply to all Variation Instances. For static fonts, the ‘gasp’ table can be set to have different values for each weight of the font. In Variable fonts, the ‘gasp’ table values are set once to cover all Instances.

**Recommended default values for all Variable fonts**

Disable hinting, and enable symmetric smoothing below 9ppem

Enable hinting, and enable symmetric smoothing @ 9ppem and above

**Current workflow practice**

The VTT Autohinter automatically generates a ‘gasp’ table for a Variable font. The current ‘gasp’ values that are generated in Visual TrueType 6.35 do not match the best practice recommendations, and have to be edited manually in VTT. (Edit menu, > Edit Gasp Table)

**Recommended solution:** Modify the Autohinter to output the best practice recommended values for the GASP table settings, when autohinting a Variable font. 

In every font there are certain conditions that will always apply. These conditions are controlled through code placed in the pre-program. When hinting instructions are used and when they are not is a global condition controlled in the pre-program. The size at which hinting is turned on and off, specified in the pre-program should be coordinated with the GASP table values. 

**Recommended solution:** Modify the default pre-program default template.

## Ideas, and code for future hinting of Variable fonts

## Instance Range CVT Delta Function

_Targeted cvt deltas for design space instances or instance ranges_

A common feature in typeface design, is that heights _(commonly x-heights)_ are often designed to vary across weights. For example the x-height in the heavier weights, is often designed to be slightly larger than the Regular. When different weights are typeset together in print or at high resolution, the x-heights ‘appear’ optically balanced. Set together, in running text, when heavier weights are used for emphasis, for example, the weights ‘appear’ to be optically aligned, even though the x-height is larger in the bolder weights. Without this optical adjustment the bolder weights can appear to be too small when set next to the Regular.

This works well when the typeface is printed or used on a high resolution screen when there are enough pixels to ensure that this subtle difference does not look obviously incorrect, but instead looks optically correct. 

At smaller screen sizes on lower resolution devices however, this difference in x-height can be a full pixel. With an x-height of 10 pixels for example in the Regular, and 11 pixels in a Bolder weight, the appearance is jarring and looks incorrect, as there are not enough pixels to show this subtle difference.

For static fonts, each weight of the font contains its own set of hints and cvt values. In static fonts, the heavier weights use the correct and larger cvt value in the cvt table. For smaller sizes, where rounding of the x-height may differ from Roman to Bold, for example, [inheritance](https://github.com/googlefonts/how-to-hint-variable-fonts#cvt---control-value-table) can be used, to force the Bold weight to be equal to the Regular until a size where its appropriate to allow for this subtle difference to be shown, usually a higher sizes where there are more pixels available. This means that in static fonts, the weights can be synchronised for smaller screen sizes when needed.

In variable fonts however, one set of hints and one cvt is used for heights in the font. The difference in x-height (for example) when there is a difference, is reflected in the [CVAR](https://github.com/googlefonts/how-to-hint-variable-fonts#cvar--cvt-variations-table) table. The cvt for the x-height is set in the CVT table, for the default instance of the font, and edits are then made to this cvt in the CVAR table for any heights that change across the design space. This is currently the limit for what can be done to control heights. The CVAR edits must be made to reflect accurately the measurements in the high resolution design. What this means in practice is that rounding of heights can vary between weights at smaller screen sizes on lower screen resolutions. 

**Note:** There is no method currently to balance or synchronize the weights in Variable fonts, as is commonly done for static fonts using inheritence.

The functions documented here goes some way towards being able to adress this problem.

<img width="100%" height="100%" src="Images/opensans14point.png">

**Figure(a)**

Showing x-height measurement in font units of Open Sans Variable, Regular and Extrabold

**Figure(b)**

Showing x-height Control Value Table for Open Sans Variable Regular and x-height CVAR adjustment for Extrabold

**Figure(c)**

Showing hinted screen rendering at 19ppem / 14 point at 96 dpi, combining Regular and Extrabold in text. **Top** showing the cvt rounding of the Regular x-height (10 pixels) and the cvt rounding of the ExtraBold CVAR adjusted x-height (11 pixels) **Bottom** Showing x-height of the ExtraBold adjusted to be equal to the Regular, using FN 195.

**Details on Function 195:**

For the example shown above the call to the function is added (inside an ASM statement) to the cvt table directly after the cvt for the x-height, cvt 6 

6:   1096 /* x height */
ASM("CALL[], 6, 192, 12, 13, 16384, 16384, 1, 195") 

**Calls Function 195**

**FDEF[], 195**
/* Function 195 takes 7 arguments */
/* VERSION 1.0 20200128 */
 
/* This function moves a CVT if it is between a PPEM range AND an AXIS range. High and low values can be the same. */
/* DEPENDANCY: FN 190 */
/* CALL[],<CVT>,<amount>,<low PPEM>,<high PPEM>, <low norm AXIS>, <high norm AXIS>, <axis number>, 150 */
/* <CVT> CVT to be modified */
/* <amount> Amount to change CVT, in ± 64ths */
/* <low ppem> Lowest PPEM range to be modified (inclusive) */
/* <high ppem> Highest PPEM range to be modified (inclusive) */
/* <low AXIS> Lowest normalized axis value to be modified (inclusive) */
/* <high AXIS> Highest normalized axis value to be modified (inclusive) */
/* <axis number> The axis number (one based) */
/* 195 Function number */
/* NOTE: If the axis number is not present in the font, it will return an axis value of zero. If this value is within the low AXIS to high AXIS range, the change WILL be applied. */
 
The first parameter after the call is the CVT number.

The second parameter is the change, in 1/64th of a pixel. So, making the CVT two pixels smaller would be -128. A full pixel higher woudl be 64.

The third and fourth parameters are the size range to apply this change. It is inclusive, so 12, 13 would be 12 and 13 PPEM. 12,12 would be just 12 PPEM. 12,30 would be 12 through 30. In the example above 19,19, adjusts the Extrabold x-height at 19ppem only

The fifth and sixth parameters are the ranges to along a variable font axis to apply the change. The ranges must be expressed in what are called by variable fonts, “normalized coordinates”. Normalized coordinates are a mathematical scaling of the coordinate space where zero is the default outline, plus one is the maximum for the axis and minus one is the minimum for the axis. 

Glyph #379 in the Selawik Variable font shows an example of normalized values for the weight axis. (Glyph #382 is a hexadecimal version of the same). 
 
For this function to work, it needs the equivalent of FN 190, which for a given axis number returns the current axis value. (If the equivalent function number is not 190, FN 195 — or whatever it might be renumbered to — needs to be updated to reference the different function number for FN 190).

The VTT assembler can also work with hexadecimal numbers. Glyph 382 shows the hexadecimal value for the axis. In this sample use of the function as shown above:
ASM("CALL[], 6, -64, 19, 19, 16384, 16384, 1, 195")
Instead of using 16834, you could use a hexadecimal value of 4000. The sample would look like this:
ASM("CALL[], 6, -64, 19, 19, 0x4000, 0x4000, 1, 195")
…where “0x” indicates the number is hexadecimal and it would perform exactly the same as the previous sample.
 

**Other example**
Superior letters need to to use a cvt to control the size on screen and uses one cvt for all weights. Bolder weights could be adjusted in ranges or individual instances to be larger at smaller sizes to ensure clarity.

**Issues and suggestions:** More testing of the New Functions and documentation is needed. Make the Functions available for use for hinting Variable fonts. Longer term have the Autohinter output these function as part of the default Font Program output. 

## Middle bar centering

**Use case**

In Type design it is common practice to design the middle bar of the H and other characters such as E F, to be optically centered rather than mathematically centered. There are many glyphs in a typical Latin font, including glyphs in the Latin, Greek and Cyrillic designs that share this design characteristic.

A middle bar that is designed to be mathematically centered will appear optically too low. This optical effect is adjusted for in the high resolution design. In the case of the H bar for example, the bar is moved up slightly so that it _‘appears’_ to be centered.

One of the benefits of hinting Variable fonts is one simple set of hints can be applied to all variations in the font. This approach however has drawbacks. Once the hinting code had been added, and compiled, there are not many other options to adjust any less than ideal visual results across the variation space.

Lets have a look at the following example of the Hinting for the H in the Google Sans Variable Font. The middle bar of the H has already been designed to be slightly higher than the mathematical centre, and this works well for high resolution output such as in print or on higher resolution screens. For lower resolution screens, where there are less pixels, rounding issues in different weights across the variation design space can cause the middle bar to appear too low at certain sizes and in certain weights variations. 

There are only a couple of ways to approach the hinting to center the middle bar. 

<img width="100%" height="100%" src="Images/LOW11GSH.png">

1. YIP Anchor the bottom point of the middle bar between the grid fitted points on the baseline and Cap height, and control the weight of the bar vertically from the bottom to the top of the bar.

<img width="100%" height="100%" src="Images/LOW12GSH.png">

2. YIP Anchor the top point of the middle bar between the grid fitted points on the baseline and Cap height, and control the weight of the bar vertically from the top to the bottom of the bar.

Both of these approaches result in the middle bar appearing too low at certain sizes in certain weight instances, in these examples in the Bold Variation at 11,13,18 and 20ppem

When hinting static fonts, whenever rounding causes the bar to appear too low, an _inline delta command_ can be added to adjust the position of the bar at a specific size to set it to correctly appear centered. 

**Note:** In variable fonts it is not possible to add delta commands as there is no one command that will work for all weight variations.

**Middle Bar centering Function**

Another approach to solve this problem is to use an already existing function [Function 116 as output by an older version of the VTT Autohinter] This function is supported in the current Font program generated by VTT when autohinting a font, but is not output by default when using the Light Latin Autohinter.

Let’s have a look at how it works to help in centering the middle bar of the H

<img width="100%" height="100%" src="Images/CENTER12.png">

<img width="100%" height="100%" src="Images/CENTER18.png">

Using this method and function results in a middle bar that is optically and visually centered accross all weight variations and sizes. One effect of this approach is that one side of the middle bar does not always fall on a sharp pixel boundary. The result of this is slightly more blur in the rendering of the horizontal bar. However this may be better for the visual output, than having the middle bar appearing to low or too high.

The Function supports two methods of use

1. Uses a cvt to control the weight of the middle bar

2. Uses a Dist to control the weight of the middle bar

Unfortunately neither of these methods fit into the current workflow and methods for hinting Variable fonts. Cvt’s are typically not used to control the weights of vertical features (ie: horizontal bars) 

The second method that uses a dist to control the weight is also not suitable for all variable fonts, especially any that support lighter weight variations. The dist command by its nature, always rounds to a full pixel. This causes too much distortion in lighter weights and is not recommend for use.

**Recommendation**

Refine the current function used for centering, to support ’Yshift’, to control the weight of the vertical bar feature, while keeping the centering aspect of the function. This will maintain a consistency with the current recommended hinting method, that uses the shift command to control the weight of horizontal features. Document the functions and make it available in the VTT Font Program for hinting Variable fonts. 

## Accent positioning and hinting

**Autohinter output for x-axis Accent positioning is incorrect in Variable fonts**

The current version of the VTT 6.35 Autohinter outputs positioning code for both x-direction and y-direction positioning of accents above base glyphs. 

**X-Positioning composite code**

The autohinter has an option to disable all x-direction hinting for the entire font under _(Tools > options > Autohinter Tab > Disable X-Direction Hints)_  
X-positioning code for accents is still generated however when this option is set. 

The code generated by the VTT Version 6.35 Autohinter, x-positioning code, calls a Function 87. The function is designed to space accents above and below base glyphs. The results however in Variable fonts are accents that are incorrecly positioned. Currently all accented glyphs need to be carefully checked to make sure the automatic code is correct.

**Current solution in VTT 6.35:** Manually check and or delete code output from Autohinter for accents positioning in x-direction [SVTCAX], or, use python script found [here](https://github.com/source-foundry/vtt-hinting-scripts) 

_The script removes all of the SVTCA[X] glyph assembly source blocks from the VTT XML export.  It is a single pass replacement over the full glyph set._

**Accent Positioning code**

X-direction positioning Code

<img width="100%" height="100%" src="Images/xaccentposition.png">

**Top:** Function 87, used to position accents, causes the circumflex accent, to be uncentered.

**Bottom:** X-direction code removed, ([SVTCAX] code block), Glyph program compiled and saved. Accent is now centered and positioned correctly, using **x-offset code only**, for all Variation instances.

The x-offset code is sufficient to position accents correctly for all Variation Instances in the x-direction. By removing the x-direction code, accents will be positioned correctly. 

This also reduces the overall font file size.

**Recommendation:** Modify the Autohinter to respect, Disable all x-direction hinting for the entire font under _(Tools > options > Autohinter Tab > Disable X-Direction Hints)_ to disable code output for accents positioning in x-direction [SVTCAX]

**Y-Positioning code**

The autohinter also generates y-axis positioning code to position accents above base glyphs, calling a Function 86 designed for this purpose

For details on the problem with this in Variable fonts, and current workarounds and solutions, see [Positioning accents](https://github.com/googlefonts/how-to-hint-variable-fonts#positioning-accents)

**Recommendation:** Refine function to output corrected positioning code. 

**Recommendation:**  Future Autohinter development 

**Better default Autohinting of accents**

The Autohinter does not use any special method when adding hinting to base unique accent glyphs. The result is often glyph shapes that collapse to one pixel in height, rendering the accents at smaller screen sizes unreadable. The base set of accents in a typical Latin font, are used in many composite glyphs, in some case as much as half the glyphs set. The work to fine tune and make readable accents is critical in achieving a well hinted font. 

**Recommendation** Refine the autohinter approach for hinting accents, smarter auto hinting, add new rules based on current best practices for hinting accents (i.e. always maintain a minimum distance of two pixels] 

**Accent Hinting details**
 
The VTT Autohinter uses a lightweight hinting strategy that focuses on fitting common heights, such as x-height, Cap Height, Ascender, Descender to CVT heights. This strategy, takes advantage of Windows symmetric rendering modes and anchors these key heights to the grid, thereby maintaining consistency in heights across a font family. This method of grid-fitting also helps to reduce blur, and minimizes distortion.
 
When the autohinter runs on glyphs that do not have char group data associated with them, such as accents, it reverts to a basic hinting strategy of anchoring points, _(usually without a reference to a cvt)_ at the y-min and y-max of the glyph. The hinting approach works well for glyphs that reference Height cvt’s, but fails when this basic anchoring is applied to hinting accents. The autohinter approach does not take into account the structure of the accented glyphs. Accents are special case glyphs, and do not normally require any height cvt’s. The Autohinting approach results in points at the y-min and y-max, grid-fitting independently of one another. This can result in either collapsed hinted outlines in some cases or hinted accents that only appear as one pixel tall. Most accents need to be at least two pixels tall for a correctly described their appearance at smaller sizes on lower resolution screens.
 
See the following examples. _(Font Inconsolata Variable font)_

<img width="100%" height="100%" src="Images/MacronHinting.png">

**Macron accent hinting**

**Left:** Unhinted outline / no Gridfit

**Middle:** Autohinted outline / Gridfit

**YAnchor (0)** Moves point 0 to the grid, without a reference to a cvt.

**ResYAnchor (1, 16)** Moves point 1 to the control value listed in the ‘Control Program’, that corresponds to an automatically generated ‘other’, (cvt #16) and rounds this point to a grid line.

Both points on the outline, 0, and 1 are rounded to the grid independantly of one another. In this case at 12ppem, point zero and point 1 both round to the same pixel height, resulting in a collapsed hinted outline. The only reason there are any pixels displaying in this case is due to drop-out control being activated.

**Right:** Manually hinted outline / Gridfit

**YAnchor (0)** Moves point 0 to the grid, without a reference to a cvt.

**YShift (0,1)** Shifts point 1, to a new position on the grid, relative to point 0’s new position on the grid, while also maintaining a minumum distance. This is the correct hinting solution. 

<img width="100%" height="100%" src="Images/AcuteHinting.png">

**Acute accent hinting**

**Left:** Unhinted outline / no Gridfit

**Middle:** Autohinted outline / Gridfit

**YAnchor (0)** Moves point 0, (point at y-min) to the grid, without a reference to a cvt.

**YAnchor (2)** Moves point 2 (point at y-max) to the grid, without a reference to a cvt.

Both points on the outline, 0, and 2 are rounded to the grid independently of one another. In this case at 12ppem, point 0, rounds up to grid, point 2 rounds down to grid, resulting in a hinted outline that is one pixel in height. To correctly describe an acute accent, a minimum of two pixels in height is needed. The autohinted outline in case will appear as a dot or flat line. 

**Right:** Manually hinted outline / Gridfit

**YAnchor (0)** Moves point 0, (point at y-min) to the grid, without a reference to a cvt.

**YDist(0,2,>=2)** Moves point 2, to a new position on the grid, relative to point 0’s new position on the grid, while also maintaining a minumum distance of 2 pixels at all sizes for all variation instance. _(width and weight)_
 
**Recommend** Extend / modify VTT Autohinter to have built in special intelligence specific to accented glyphs. The approach is based on generic common hinting strategies for accents. See illustrations and methods described above. The two basic approachs described, can be applied to a wide variety of accent types and will work consistently in displaying clear and readable accents at all sizes across all variation instances. _(width and weight for example)_

## Extending Character group support

_Extending Character group support for addition glyph sets, and scripts besides Latin_

The CharGrp.txt file for the font assigns each glyph to a character group: uppercase, lowercase, figure, or other. This file is imported into VTT when preparing a font for hinting. This data is also used internally by the VTT Autohinter, when Autohinting a font, to automaticaly build a cvt table, with correct height information, as well as to correctly identify glyphs and assign appropriate cvt height values.

The ‘character group’ tells Visual TrueType which group of values to use from the control value table (‘cvt’) for the glyph. The VTT Autohinter uses character group data to assign cvt’s, to glyphs, identifying common heights for Latin, Greek and Cyrillic. _(WGL4 glyph set that is currently supported internally in the Autohinter)_

Glyphs that fall outside of this specified glyph range, are autohinted, but are not assigned cvt’s. A YAnchor command is typically applied to the y-min and y-max of these glyphs, without a reference to a cvt. 

VTT does currently allow user definable character groups and feature categories, for glyph set that fall outside of the four hard-wired categories "uppercase", "lowercase", "figure", and "other". Further character groups and feature categories can be added into the new control program, along with the sequence of glyphs needed to visualize the spacing. See the link below for an example of adding Indic character group to [Noto Devanagari Serif Variable font](https://github.com/googlefonts/how-to-hint-complex-script-variable-fonts#adding-character-group-information-and-cvts-for-devanagari) 

**Note:** The addition of further character groups supports manual hinting, and speeds up the hinting process, by allowing VTT to automatically choose the correct cvt’s as the glyph is hinted using the graphical hinting tools. It is not possible currently to have the Autohinter recognise these new character groups, and apply the correct cvt’s automatically. 

**Problem:** Global adjustments _(to key Heights cvt's for example,)_ are often made in the cvt table to adjust the overall hinted proportions of the font. These adjustments only apply to glyphs that have been assigned height cvt’s. Glyphs that fall outside of the VTT supported glyph set, require manual hinting and appropriate cvt’s assigned to ensure consistency with the main glyph set.

**Example:** Ligature glyphs, _(ffi, ffl, ft, fft,)_ are not recognised as lowercase, by the Autohinter and are assigned into the default ‘other’ character group, without cvt’s being assigned for heights. Global adjustments made to change the x-height or ascender height for the main font, will not be reflected in the ligature glyphs until they have been assigned cvt’s to match the lowercase. Many newer fonts contain much larger glyph sets, including extensive Opentype sets _(ligatures, Small Caps, with extended accented sets)._ Currently these glyphs require manual adjustments, and appropriate cvt assignment, a time consuming process.

**Recommendation**
Investigate extending the Autohinter to recognise additional glyph sets outside of the currently supported WGL4 glyph range.

Investigate extending the Autohinter to recognise addition glyph ranges for other scripts. This would require additional script specific typographic height information to be built into the autohinter. There is a possibility to leverage some relevant script specific work already [completed for building the TTFAutohinter](https://freetype.org/ttfautohint/doc/ttfautohint.html#scripts)


## Simplify CVT, Pre and Font Program templates 

When the preparing a font for hinting or when running VTT Autohinter VTT creates three global tables.

**Control Value & cvar (cvt variations) tables)**

The CVT table is created automatically for Latin fonts, with pre-populated font measurements

**Pre-Program**

In every font there are certain conditions that will always apply. In a TrueType font these conditions are controlled through code placed in the pre-program.

**Font Program**

This table stores functions that can be called from the ‘Pre-Program’ or from the glyph’s instructions. Functions are used to eliminate repetitive code from being used in multiple glyphs. 

Both the CVT table and the Font Program are based on older templates, and contain unnecessary information. (e.g: Older functions related to Italic control that are no longer needed) The default Font program generated by the Autohinter contains a total of 73 Functions, many of which are no longer relevant or needed. 

The CVT table also contains many entries that are not needed for hinting in Variable fonts

**Recommendation:**

Research and review the above table templates. Remove all unnecessary cvt entries from the default CVT Table template. Remove all unneeded functions from the Font program template. Modify the VTT Autohinter to generate these new simplified templates, for a modern hinting workflow.

**Benefits**

Potential for some font file size savings. A simpler CVT Table and Font program would be useful also for an easier workflow.

## Future Hinting research

As discussed in the [Variable font hinting document](https://github.com/googlefonts/how-to-hint-variable-fonts#hinting-and-new-rendering), and here in [Respecting the outlines](https://github.com/googlefonts/how-to-hint-variable-fonts/tree/main/Article) hinting has been reduced to a simple set of instructions, to control important heights, using cvt’s as a reference, and to control stem weights, using either a ResYdist, or YShift command. Maintaining consistent heights, and reducing blur on horizontals, are the main benefits of hinting for modern rendering environments.

The basic hinting commands needed to hint a Variable font are as follows
ResYAnchor _(with and without cvt references)_, ResYDist, YShift and YInterpolate

In addition to Anchors with height cvt references, and stem weight control, using the Dist or Shift commands, the next important hinting command is ‘Interpolate’. _(Interpolate moves untouched points between two moved key points or ‘parents points’, keeping the relation the same as the original position of all points)_

When preparing any Variable font, a large part of the post Autohinter work, is to manually add ‘Interpolate’ code. The Autohinter does a poor job of interpolation, mostly not generating any code at all. Glyphs that typically need the code, A B K, M, N, Q, R, S, V, W, X, Y, Z for example, need manually editeding to add the ‘Interpolate’ code. This is just the Uppercase in Latin. In any typical font, there are many additional glyphs that need this code, to ensure the correct weight of the hinted glyph is maintained. 

Taking a step back now to look at what hinting is useful for, and what is missing, the ‘Interpolate’ code stands out as a missing piece. Perhaps this can be revisted,and engineered in a simpler way. As part of this investigation, I did some experimental hinting, to try to understand how a simpler approach may be the answer to many of the glyphs that currently need ‘Interpolation’ code added. Lets have a look at the following. 

<img width="100%" height="100%" src="Images/AutohintM.png">

**YAnchor (8, 8)** Moves point 8 to the control value listed in the ‘Control Program’, that corresponds to the baseline, (cvt #8) and rounds this point to a grid line.

**YShift (8, 16) YShift (8, 23)** Shifts point 16 and 23, to a new position on the grid, relative to point 5’s new position on the grid, ensuring point 16 and 23 will also be grid-fit to the baseline. **Note** _Because points 16 and 23 are at the same y coordinates as point 8, its not clear that that these additional commands are needed_ 

**YAnchor (10, 2)** Moves point 10 to the control value listed in the ‘Control Program’, that corresponds to the Cap height, (cvt #2) and rounds this point to a grid line.

**YShift (10, 14)** Shifts point 14, to a new position on the grid, relative to point 10’s new position on the grid, ensuring point 14 will also be grid-fit to the Cap Height. **Note** _Because points 14 is the same y-coordinate as point 10, it is not clear that that this additional command is needed_

This is the code as output for the Capital M by the VTT Autohinter. In this case the outline is scaled to the desired size and Point 10 is rounded to the grid, using the scaled and rounded height of cvt 2. The remaining points, 1,2,11,12,22 and 21, are still not positioned correctly relative to point 8 and point 10’s new position on the grid. The result is that at this size, points 1,2,22 and 21 are too high, and points 11 and 12 are too low in both the Regular and Light weight variation of the font. This causes the diagonals in the hinted outline to appear to be too light. These points now need to be Interpolated, so as they are properly positioned between Baseline and Cap Height.

**Note** _The last command in the VTT talk, Smooth(), generates low level code of IUP[ Y ] and IUP [ X ], which according to the TrueType specification, ‘Interpolates Untouched Points through the outline’. 

**Note** This does not appear to be the case in this example. More investigation is needed to inderstand why this is the case._

<img width="100%" height="100%" src="Images/InterpolateM.png">

**YInterpolate(8,2,12,22,10)** Untouched points are interpolated between Parent points, to ensure the relationship of these points is kept the same as the original position of all points in the high resolution outline. This corrects the weight of diagonals. **Note** This code is a manual addition and has to be added to many glyphs in any typical font. 

<img width="100%" height="100%" src="Images/MinimalM.png">

In this final example, only 2 points on the outline are touched. Point 8 is positioned on the Baseine and point 2 on the Cap Height. When this code is compiled, the result is equivalent to the second example shown above. The untouched points appear to be interpolated correctly, using just the Smooth() command. Points 23 and 16 on the baseline, are aligned correctly at all sizes. Point 14 at the Cap Height is also correct grid-fitted and aligned at all sizes. Points 1,2,11,12,22, are all positioned correctly in relation to points 2 and 8. 

**Recommendation:** This last example points to a newer simpler method of hinting, which would be another step towards fully automated hinting of Variable fonts. In addition to a simpler approach, this would also save on file size.

More detailed research is required to fully understand and document, how the Gridfitting, Interpolate and Smooth all work together. Given the new simpler set of instructions, it is possible that bulding a new Autohinter or extending TTF Autohint for example, might use this new information. 


## Relevant external discussions

[Open sourcing VTT](https://typedrawers.com/discussion/2846/open-sourcing-vtt)

[“Constraints” instead of PS or TT hints](https://github.com/unified-font-object/ufo-spec/issues/43)