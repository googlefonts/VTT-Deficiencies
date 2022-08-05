# Research, define and expand on, current deficiencies in VTT. New ideas for Variable font hinting in VTT

by [Michael Duggan](https://twitter.com/mickduggan)

## Introduction

VisualTrueType 6.35 includes support to handle all aspects of hinting Variable fonts. There are however some areas where the code output from the autohinter does not work optimally. There is also limited support in the Autohinter for Non Latin fonts.
 
The following *notes, document the areas where the workflow may be improved by adding new capabilities and improving upon existing features, both in the Autohinter and the Visual hinting UI 
 
The documentation can be used to further discussion as well as help in future development work on the Autohinter, with the eventual goal of to speed up the overall workflow, and to achieve a more robust and consistent output from the autohinter.
 
*All notes are based on the publicly available current version of Visual TrueType 6.35, running on Surface Laptop  / Windows 11

 

