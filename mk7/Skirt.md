# Skirt Design and Workflow

## Intro

FullSkirtGroup contains the full bodies for the form, skirt, seam, and clamp from which various
sections will be sliced out.

The form is the outside part of the gluing jig.
The clamp in the inside part of the gluing jig.
The seam is a 20mm wide band that is glued on the inside of the seam between skirt panels.
The skirt is the skirt panel.

A skirt panel may be sliced into parts to fit on the 3D printer's bed.  For example:
* SternUpperLeftBody
* SternLowerLeftBody
* SternUpperRightBody
* SternUpperRightBody
* SternSliceSeamLeftBody
* SternSliceSeamLeftBody

The stern does not need any new jigs, it will use the stern quarter and beam jigs.

### Non-parametric workflow

This workflow basically creates a 3D model of the skirt then slices into sections that are then flattened (unwrapped) into panels that can be printed in TPU.  The panels are then assembled using E6000 glue and some jigs into the full skirt.  A 20mm seam piece provides gluing surface across adjacent panels.

The issue is in the flattening.  The workflow is to extract the faces of the skirt panel as a shell, then convert to a mesh that is then unwrapped.  The resulting shape is then aligned to the XY axis and converted to a sketch that is then modified (tabs and holes added) and extruded.  The best I can tell, parametric continuity is lost when the face shell is created.  So when any changes are made to the original 3D skirt model, everything from the flattening process has to be manually recreated.  I know, bummer...

## FullSkirtGroup

The FullSkirtGroup is the modeling of the four layers of the skirt:
* Form - the outside of the gluing jig
* Clamp - the inside of the gluing jig
* Skirt - the hovercraft TPU skirt that will be divided into panels
* Seam - overlapping TPU strips for gluing the skirt panels together

### Workflow

* Create the FullSkirtGroup.
* Create the FormBody and move into the FullSkirtGroup.
* Create the PathSketch which is the upper deck perimeter.
* Create the SkirtCrossSectionSketch with construction geometry for the 4 layers (form, skirt, seam, clamp).  Make sure the PathSketch touches the cross section where the top of the arc intersects the top horizontal line.
* Create the SkirtAdditivePath
* Duplicate the FormBody to SkirtBody, SeamBody, ClampBody
* Edit the SkirtCrossSectionSketch in each of the bodies to have the correct normal/construction line types
* Move FormBody, SkirtBody, SeamBody, ClampBody to the FullSkirtGroup

## Panel Groups

There will be four unique skirt panel profiles:  Bow, Beam, Stern Quarter, Stern

The skirt panel profiles will be sliced off of the full model layers.  The process is basically add two datum planes where the model will be sliced.  If the two dataum planes do not converge withing the model, then add a third datum plane to cut off the section of the model opposite of the desired section.  Next a sketch is created for each datum plane and contains an rectangle larger than the model. This sketch then is used to create a pocket that cuts of the undesirable portions of the model, eventually leaving just the desired section.

The skirt panels will then be unrolled and modified with seam tabs and holes so they can be printed flat.

The other panels are printed on their sides.

Bow and Beam panels will have 10mm holes for injecting air into center air cushion.

Stern and Stern Quarter will not have air injection holes to reduce flooding of skirt over water.

All panels will have 3mm drain holes at the skirt's apogee (i.e. closest to the ground).

### Workflow

Duplicate the FullSkirtGroup to the following panel groups:  BowGroup, BeamGroup, SternQuarterGroup, SternGroup


In each of the panel groups, slice each of the bodies by:

* add two datum planes where the slice needs to be.
* attach a sketch to each datum plane with centered rectangle (2 * HullLength x 2 * HullLength)
* create a pocket from each sketch that removes the unwanted portions of the full skirt shape

The FormBody, SeamBody, and ClampBody parts are finished.

## Skirt Panel

The SkirtBody need to be unrolled and sliced as it will be printed flat on the print bed.
SeamMasterSketch - create a master sketch with all the seam information to be merged into panel sketches

### Workflow

for name in [Bow, Beam, SternQuarter, Stern]:

* Set Refine to true on last operation in {name}SkirtBody
* {name}FaceShell - Part: Shape Builder, Shell from faces - create a face shell of all interior {name}SkirtBody faces.  Rename Shell to {name}FaceShell.  Move into {name}Group.
* {name}FaceShell (Meshed) - Mesh: Create Mesh from shape. Move into {name}Group.
* {name}UnwrappedShape - Mesh: Unwrap mesh.  Rename Shape to {name}UnwrappedShape.  Do not move into {name}Group.
* tmpSketch - Draft: Draft to sketch.  Measure the long edge (that connects to the top deck) and save the value in the spreadsheet.
* create body {name}UnwrappedBody
* {name}AlignmentSketch - create an alignment sketch on the XY plane with a square centered on y axis and bottom aligned to x axis with a width equal to the saved value of the long edge.
* delete the tmpSketch
* Select {name}AlignmentSketch, then control select {name}UnwrappedShape, Part: Alignment (select the two bottom corner points on both sketches.  Right click, Align.  The {name}UnwrappedShape should now have the long edge aligned to X axis and horizontally centered on the Y axis.
* {name}UnwrappedSketch - select {name}UnwrappedShape then Draft: Draft to sketch.  Rename SketchNNN to {name}AlignedSketch and move to {name}UnwrappedBody.
* move {name}UnwrappedShape to {name}Group.
* hide {name}AlignmentSketch, {name}UnwrappedShape, {name}FaceShell (Meshed)
* edit {name}AlignedSketch, select all the normal lines, add Block Constraints.  The sketch should now be fully constrained.
* Attach {name}AlignedSketch to {name}UnwrappedBody's XY plane and activate "Plane face" Map Mode.  Select and refresh the sketch.
* move {name}UnwrappedBody to {name}Group.
* merge {name}AlignedSketch and SliceNMasterSketch, where n is the number of pieces (2, 4, or 8) the panel needs to be divided into for printing (select sketches to merge then Sketcher: merge sketches)
* rename SketchNNN to {name}PanelSketch and move to {name}UnwrappedBody.
* attach {name}PanelSketch to the {name}UnwrappedBody.OriginNNN.XY-planeNNN.  Refresh {name}PanelSketch.
* edit {name}PanelSketch and attach seam construction geometry to the unwrapped panel sketch.

for 2 piece panels, the positions are ["Upper", "Lower", "Splice"].
for 4 piece panels, the positions are ["UpperLeft", "UpperRight", "LowerLeft", "LowerRight", "SpliceLeft", "SpliceRight"].
for 8 piece panels, the positions are ["UpperLeft", "UpperCenter", "UpperRight", "LowerLeft", "LowerCenter", "LowerRight", "SpliceLeft", "SpliceCenter", "SpliceRight"].  Note, the two center panels are identical so only one center panel model is necessary.

* create bodies for each of the appropriate positions:  {name}{position}Body
* duplicate the {name}PanelSketch into each body and rename to {name}{positon}BaseSketch.
* edit {name}{positon}BaseSketch and adjust the normal lines to construction lines so the correct perimeter for the panel section is normal lines.  Add any normal lines needed.  Use Split Edge to add a point on the perimeter for the line that slices the panel (hint: split edge, make new segments parallel, attach new point to slice construction line, coincident the two points, toggle the remaining line segment to construction).
* duplicate {name}{positon}BaseSketch to {name}{positon}TabSketch
* edit {name}{positon}TabSketch and change all normal lines to construction, then change the construction lines for the seam tabs to normal lines.
* adjust the position Z height of {name}{positon}TabSketch to SkirtThickness.
* Pad using SkirtThickness for {name}{position}BaseSketches and {name}{position}TabSketches

### Starboard vs Port Beam Panels

The port and starboard beam panels are mirrors of each other so simply select the bodies on the left and right side of the beam panel, then Part: Mirroring.  The default should be the XY Plane which is what is needed.  Finally move the mirrored bodies to the BeamGroup.

### No Print

Some of the bodies are not intended to be printed:  {name}SkirtBody and {name}UnwrappedBody.  As a reminder you may want to append " (no print)" to them: i.e.:  "{name}SkirtBody (no print)" and "{name}UnwrappedBody (no print)".  You may also want to so indicate other objects like {name}FaceShell, {name}FaceShell (Meshed), and {name}UnwrappedShape.

### Exporting

* Save all

To combine a set of bodies into a single STL file:

* select {name}{position}Body models and File|Export as Skirt-{name}Group or Skirt-{name}Jig

Or you can individually export each model to it's own STL file.  The former is a little more convient for importing into the slicer while the later allows reloading changes into the slicer.

