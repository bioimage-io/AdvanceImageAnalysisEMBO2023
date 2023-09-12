# Advanced methods in bioimage analysis

EMBO Practical Course, Heidelberg, 2023

# Preliminary installations (local workstations that need to be setup)
## Fiji 
From session one you should already have:
- Download Fiji on your computer (https://imagej.net/software/fiji/downloads)
- Install the required plugins in Fiji:
  - Choose Help → Update and wait for it to complete;
  - In the ImageJ Updater, press Manage Update Sites;
  - Check the following sites and press Close:
    - IJB-Plugins
    - DeepImageJ
  - Press Apply Changes.

## ilastik

* Download the latest beta version of ilastik from [our download page](https://www.ilastik.org/download.html#beta).
* Windows users please install, Mac and linux users please extract the downloaded file.

Useful links:
* [ilastik documentation](https://www.ilastik.org/documentation/)
* [ilastik on forum.image.sc](https://forum.image.sc/tag/ilastik)
* [ilastik source code](https://github.com/ilastik/ilastik)

## BioImage Model Zoo

Download and install [Google Chrome](https://www.google.com/chrome/)

# BioImage Model Zoo exercises

## Exercise 1: Explore the BioImage Model Zoo content
(User guide: https://bioimage.io/docs/#/user_guide/README)
1. Go to the BioImage Model Zoo and identify where to find:
  -	The Documentation
  - “Ask for help”
  - Bug reports
  - Contact us
2. Look for a model to segment *Bacillus Subtilis* bacteria.
3. The model is displayed as a model card. If you click, it will open. Take a look at the information given (e.g., name of the authors, the publications, and the ID of the model.)
**What is the (animal) name of this model?**
4.	In the model card, find the Jupyter notebook that you could use to retrain the model. The link will open a new card with the information about this notebook. In this card, find a way to open it in Google Colab.
5.	Back in the model card, find the information about the dataset used to train the model. The link to the dataset will open a new card with the information about the dataset and the link to download it from Zenodo.
6.	In the model card, find a spot where you could ask for help with the model.

## Exercise 2: Run a model in the browser with the BioEngine
Browser prediction: Use the model `shivering-raccoon`. 
- Click on the “play” button to start the BioEngine. 
- Download the test image left column in the BioEngine describing the model.   
- Click on Select, upload the image and choose it.
- Choose the model format (`torchscript` is recommended)
- Choose the correct order for the channels (look at the description of the model). For 2D, it is usually `bcyx`.
- Click on submit.
 
### Exercise 2.1
Play with the prediction in the browser with the any of the following models:
- Hiding-tiger
-	Affable-shark
-	Powerful-chipmunk
-	Determined-chipmunk
-	Hiding-blowfish
-	Willing-hedgehog

# DeepImageJ:

## Exercise 3: Download a model from the BioImage Model Zoo and install it with deepImageJ in Fiji

1.	Look for a model to segment Mitochondria in 2D Transmission Electron Microscopy Images (e.g., `shivering-raccoon`).
2.	Find a way to download the model for deepImageJ (we will use them later).
3.	Install it in Fiji: Plugins > DeepImageJ > DeepImageJ Install Model
    -	Choose “Install Private Model”. There write the entire path to the model zip file that you just downloaded. (In Linux, click on the right button and search for properties. There you will find the path to the folder and the name of the model)
    - Read the note and click on Install. If there is no error message, the model is installed and you can close the window.
4.	Run a demo of the model: Plugin > DeepImageJ > DeepImageJ Run. Choose the model and click on Run on an example image.

## Exercise 4: Deep-learning boosted instance segmentation of cells.
1.	Download the model `LiveCellSegmentationBoundaryModel` from the Bioimage Model Zoo
    - The simplest way is to use the plugin deepImageJ Install Model in Fiji. This could take one or two minutes.
2.	Run a demo of the model in Fiji:
    -	Open deepImageJ Run.
    - In Model, choose LiveCellSegmentationBoundaryModel
    -	Click on “Run an example image”.
    You will get the raw output of a model trained to predict cell boundaries and cell masks.
3.	Save both images (input and output) to use them in the next steps.
4.	Create instances with the model output using Watershed segmentation and the boundaries.
    - Split the output channels in independent images: Image → Color → Split channels.
    - Binarize both channels: For each image, run Image → Adjust → Threshold…
        -	Click on “Dark background”
        -	Click on “Set” and choose
          - Edges: 0.35 and 1 as the lower and upper threshold respectively. Click “Ok”.
          - Binary mask: 0.5 and 1 as the lower and upper threshold respectively. Click “Ok”.
        -	Click “Apply” and “Convert to Mask”.
        - It may be that your Look Up Table (LUT) is inverted (the background displayed in white). You can change the display by going to Image → Look up Tables → Invert LUT
    - Create cell landmarks to run the marker-controlled watershed:
        - Subtract the edges to the binary mask: Process → Image → Calculator. Choose the subtraction: (binary mask) – (edge mask).
        - Marker Controlled Watershed: Plugins → MorpholibJ → Segmentation → Marker-controlled Watershed.
            -	Input: Binary segmentation (thresholded output of the model)
            -	Marker: The result of the subtraction (the previous step)
            -	Mask: Binary segmentation (thresholded output of the model)
            -	Compactness: 0.00.
 
### IJ Macro for automation

The previous code can also be run automatically with an ImageJ macro:

<details>

<summary>Macro code for above exercise</summary>

```javascript
name = "model_output";
rename(name);
run("Split Channels");
// Channel 1 represents the binary masks
selectWindow("C1-"+name);
setAutoThreshold("Default dark");
//run("Threshold...");
setThreshold(0.5000, 1.00);
run("Convert to Mask");
run("Invert LUT");
 
// Channel 2 represents the edges of the cells
selectWindow("C2-"+name);
setAutoThreshold("Default dark");
//run("Threshold...");
setThreshold(0.3500, 1.00);
setOption("BlackBackground", false);
run("Convert to Mask");
 
run("Invert LUT");
imageCalculator("Subtract create", "C1-"+name,"C2-"+name);
selectWindow("Result of C1-"+name);
 
rename("markers");
run("Marker-controlled Watershed", "input=C1-"+name + " marker=[markers] mask=C1-" + name + " compactness=0 binary calculate use");
run("Set Label Map", "colormap=Spectrum background=Black shuffle");
```

</details>

# ilastik

data: TODO: insert link :)

## Exercise 5 - Boundary segmentation with the BioimageIO Model Zoo

**data**:
* `neural_network_workflow/N6_root_z130.h5`

1. Open ilastik.
2. Create a new _Neural Network Classification_ Workflow.
3. load the hdf5 image file.
4. Proceed to the next workflow step (_NN Prediction_).
5. Go to https://bioimage.io and try to find a network you think would fit this data.
6. Copy the _nickname_ or the _doi_ of the model and paste it into the textfield on the left in ilastik.
7. Alternatively, use the network `2d-unet-arabidopsis-ovules_torchscript.zip` that is in this folder.
8. Export the result.

Bonus:
* Try any of the other images in the `neural_network_workflow` folder and see how the network performs.
* Try the autocontext workflow on the same image.

## Exercise 6 - Boundary-based instance segmentation Multicut

**data**:
* raw_data: `neural_network_workflow/N6_root_z130.h5`
* probabilities: exported result from Exercise 5.

Boundary-based instance segmentation

**Part 1:**
1. From Edges to Objects in the _Boundary-Based Segmentation with Multicut_ workflow (create a new workflow).
2. Load the dataset along with the corresponding probability map (you can use pixel classification, autocontext, or the **neural network workflow result from the previous exercise**).
3. In the DT Watershed, try different values for smoothing. Try to avoid joined segments - those cannot be separated anymore in later steps.
4. You can quickly see the raw data by pressing the letter `i` on the keyboard.
   Pressing `i` again will show you all overlay images again.

**Part2:**
1. Proceed on to training and Multicut.
2. Annotate edges you want to keep by _right clicking_ on them (red), and edges that should be left out of the solution with the _left mouse button_ (green). Click _live update_.
3. Check out the Multicut solution by clicking _update now_.
   Activate the Multicut Segmentation layer by clicking on the "eye" icon to see the label image.
   Are you happy with the result?
4. Is it enough to change the _Threshold_ value to improve the segmentation?
