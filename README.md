# "Doodle Labeller"

> Daniel Buscombe, Marda Science daniel@mardascience.com

> Significant code contribution from LCDR Brodie Wells, Naval Postgraduate school Monterey
> Sample image provided by Christine Kranenburg , USGS St. Petersburg Coastal and Marine Science Center

This is a tool for partially supervised image segmentation and is based on code previously contained in the "dl tools" repository, https://github.com/dbuscombe-usgs/dl_tools

## Rationale
There are many great tools for exhaustive image labeling for segmentation tasks, using polygons. Examples include [makesense.ai](www.makesense.ai) and [cvat](https://cvat.org). However, for high-resolution imagery with large spatial footprints and complex scenes, such as aerial and satellite imagery, exhaustive labeling using polygonal tools is prohibitively time-consuming.

The approach taken here is to freehand label only some of the scene, then use a model to complete the scene. Sparse annotations are provided to a Conditional Random Field (CRF) model, that develops a scene-specific model for each class and creates a dense (i.e. per pixel) label image based on the information you provide it. This approach can reduce the time required for detailed labeling of large and complex scenes by an order of magnitude or more.

## How to use

This is python software that is designed to be used from within a `conda` environment. After setting up that environment, the user places imagery in a folder and creates a `config` file that tells the program where the imagery is and what classes will be labeled. The minimum number of classes is 2. There is no limit to the maximum number of classes.

### create environment
```
conda env create -f doodler.yml
```

### Activate environment
```
conda activate doodler
```

### Add pictures
You will need separate folders for:
* Where to get the images that you want to label `data/images`. The program will assume you want to label all these images in one go. It's usually best to put one image in there at a time at first while you get a feel for how long it takes
* Where to put a labeled images (annotation files in `.npy` format, the label images in png format, and a plot showing the image and label in png format)

### Make a config.json file
Example is provided

`
{
  "image_folder" : "data/images",
  "label_folder": "data/label_images",
  "max_x_steps": 4,
  "max_y_steps": 4,
  "ref_im_scale": 0.8,
  "lw": 5,
  "im_order": "RGB",
  "theta": 40,
  "n_iter": 10,
  "compat_col": 40,
  "scale": 5,
  "prob": 0.4,
  "classes": {
   "Surf_Swash": "#fcf9f9",
   "Water": "#3b81d0",
   "BareSand": "#c0d03b",
   "Grass_Shrub": "#62894d",
   "WoodyVeg": "#395c27",
   "MarshPlatform": "#94925c",
   "Road": "#888c8c",
   "Anthro": "#23ad96"
 }
}
`
where

"image_folder" : ordinarily this would be "data/images" but could be a relative path to any relative directory
"label_folder": see the above but for "data/label_images"
"max_x_steps": number of tiles to split the image up in x direction
"max_y_steps": number of tiles to split the image up in y direction
"ref_im_scale": the proportion of the detected screen size to make the window
"lw": initial pen width in pixels. this can be changed on the fly
"im_order": either "RGB" or "BGR", depending on what type of imagery you have (RGB is more common)
"theta_col": 40,
"compat_col": 40,
"scale": 5,
"prob": 0.4,
"n_iter": 10,
"classes": a dictionary of class names and associated colors as hex codes. there are various online color pickers including this one: https://htmlcolorcodes.com/

This file must be called `config.json`

### Run
If you are running this through command line, you will need to cd to the
directory you will be working in, for example:

```
cd /Documents/doodle_labeller
```

Then run it with:

```
python doodler.py -c config_file.json
```

It will select the next image the next time you run it.

### Draw on the image
The title of the window is the label that will be associated with the pixels
you draw on. After you are done with label press escape. You can increase and
decrease the brush width with + / -. You can also undo a mistake with z.

* Use s to skip a square
* Use b to go back a square
* Use number keys to switch label

## Dense labeling happens after a manual annotation session
After you have labeled each image tile for each image, the program will automatically use the CRF algorithm to
carry out dense (i.e. per-pixel) labelling based on the labels you provided and the underlying image


## compiling doodler.py
- conda activate doodler
- pip install --upgrade 'setuptools<45.0.0'
- pip install python-opencv-headless
- pip install pyinstaller
- pyinstaller --onefile --noconfirm doodler.py --clean --hidden-import pydensecrf.eigen
- conda deactivate
- ./dist/doodler

## Improvements coming soon
* support for geotiffs
* compiled executables
* (maybe) add water masking model to automatically mask water prior to manual annotation
