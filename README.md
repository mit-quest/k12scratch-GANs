# Documentation For the Scratch GAN Paint Extension


## Contents
* Introduction
* What is GAN Paint? 
* How the GAN Paint Extension works
* How to interact with GAN Paint/Gandissect API
* The current state of the GAN Paint block
* Other notes


## Introduction
Hi, I'm Maya, the student who was working on this project during spring & fall 2019. The project is effectively to take [GAN Paint](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4) and put it in a Scratch block. I'm proud to say that I helped start the project, and also worked on getting it to a point where it's viable. There is still lots of room for improvement, which I will discuss later in this document. I will also explain the different components that allow this extension to work and how they work together, as well as how to interact with the GANPaint API. If you have any questions, feel free to contact me at my Kerberos email (mayigrin at mit dot-edu). I'd be happy to help! 

If you are going to work on the GAN Paint extension, please go through all sections of this document. If you're just curious about how to use the Gandissect API, read only "How to interact with GAN Paint/Gandissect API".

Also, you'll notice that I mention Phillip quite a bit throughout this document. He was the student to work on this project during spring 2019. You can see his documentation at the following link:
[https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md](https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md)


## What is GAN Paint? 
[GAN Paint](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4) is a tool that has used GANs (Generative Adversarial Networks) to generate 16 photorealistic images of churches. By using the various brushes provided, and then dragging your mouse across the image, you can tell GAN Paint to add or remove certain features to/from different parts of the image, and GAN Paint will alter the image accordingly. I recommend that you play around with GAN Paint for a few minutes to see what I'm talking about. 


## How the GAN Paint Extension works 
### Overview
The GAN Paint Extension is made up of 3 major components: the Javascript code for the Scratch block, the Python code for interaction with the API, and the dissected GAN and its associated API. You will need the following repositories in order to interact with those components and actually use the Scratch GAN Paint Extension: 
* [https://github.com/mit-quest/k12scratch-blocks](https://github.com/mit-quest/k12scratch-blocks)
* [https://github.com/mit-quest/k12scratch-gui](https://github.com/mit-quest/k12scratch-gui)
* [https://github.com/mit-quest/k12scratch-vm](https://github.com/mit-quest/k12scratch-vm)
* [https://github.com/mit-quest/k12scratch-render](https://github.com/mit-quest/k12scratch-render)
* [https://github.com/mit-quest/bridge_gandissect](https://github.com/mit-quest/bridge_gandissect)

The Javascript code for the Scratch block is located primarily in the following two documents:
* [https://github.com/mit-quest/k12scratch-blocks/blob/master/core/field_ganpaint.js](https://github.com/mit-quest/k12scratch-blocks/blob/master/core/field_ganpaint.js)
* [https://github.com/mit-quest/k12scratch-vm/blob/master/src/extensions/scratch3_ganpaint/index.js](https://github.com/mit-quest/k12scratch-vm/blob/master/src/extensions/scratch3_ganpaint/index.js)

The Python code for interaction with the API is in this document:
* [https://github.com/mit-quest/k12scratch-blocks/blob/master/core/GANpaint_request.py](https://github.com/mit-quest/k12scratch-blocks/blob/master/core/GANpaint_request.py)

And the dissected GAN and its associated API is in this repository:
* [https://github.com/mayigrin/proGAN_translation](https://github.com/mayigrin/proGAN_translation)

The way these components all work together is as follows:
* The user sets up the Scratch Dev environment and runs the GUI
* Next, the user runs the GAN Paint API server.
* The user should then start the Python server that serves as the middle man between the Javascript code and the GAN Paint API.
* When the user uses the Scratch GAN Paint Block and selects some pixels on the image using their mouse, a GET request is sent to the aforementioned Python server.
* That Python server does some necessary manipulation of the parameters that it receives, and then executes a specific choreography of GET and POST request with the GAN Paint API in order to get a base64png string representation of the new image.
* The Python code then saves that image to a specific location, overwriting the existing image that is show in the Scratch GAN Paint Block
* The Scratch GANPaint Block then updates, showing the newly modified image to the user.

I will now explain, for each of those steps, what code needs to be run and where in order to accomplish that task.

### Initial downloads and installations
First make sure to install Git and npm (Node Package Manager) on your computer, and then download my versions of the scratch-gui, scratch-vm, scratch-blocks, scratch-render, and gandissect repositories from GitHub. Make sure to save all 4 repositories in the same folder. Said repositories can be found here: 

* scratch-gui: [https://github.com/mit-quest/k12scratch-blocks](https://github.com/mit-quest/k12scratch-blocks)
* scratch-vm: [https://github.com/mit-quest/k12scratch-gui](https://github.com/mit-quest/k12scratch-gui)
* scratch-blocks: [https://github.com/mit-quest/k12scratch-vm](https://github.com/mit-quest/k12scratch-vm)
* scratch-render: [https://github.com/mit-quest/k12scratch-render](https://github.com/mit-quest/k12scratch-render)
* gandissect: [https://github.com/mit-quest/bridge_gandissect](https://github.com/mit-quest/bridge_gandissect)

### Pre-setup code modifications
Before you even setup your environment to run Scratch with the GAN Paint Extension, there are a few modifications you will have to make to your version of the code.

First, you will need to figure out the IP address of your k12scratch-blocks and bridge_gandissect directories. If you are running these all locally, you can simply use `localhost`.

Now, there are two places where you need to replace IP addresses with the IP values you found. 

First, in "k12scratch-blocks/blob/master/core/field_ganpaint.js", go to line 797, inside the ajax call Replace "34.70.177.61" with the IP address you found for k12scratch-blocks, or "localhost" if you're running everything locally.

Next, in "k12scratch-blocks/blob/master/core/GANpaint_request.py", go to lines 25, 32, 57, and 101. In each of those places, replace "34.74.168.113" with the IP address you found for k12scratch-blocks, or "localhost" if you're running everything locally.

### How to set up the Scratch Dev environment
Now, to run the modified version of Scratch, open the terminal, cd into the k12scratch-gui directory, and enter the command "npm start -- --no-inline --no-hot".

If the "npm start -- --no-inline --no-hot" command isn't working, see if [this tutorial](https://scratch.mit.edu/discuss/topic/336496/) can help you. 

Whenever you need to rebuild k12scratch-blocks (i.e. make sure that the changes you make to k12scratch-blocks get expressed), you will need to open the terminal, cd into the scratch-blocks directory, and enter the command "npm run prepublish". Wait until this has finished executing, then run the command "npm start -- --no-inline --no-hot" within scratch-gui to open the Scratch GUI again. Also, to execute the command "npm run prepublish" in scratch-blocks, you will need to be using Python 2 on your computer. This may require you setting up a virtual environment on your computer to run Python 2. 

Also, make sure to read the [Scratch 3 Extension documentation](https://github.com/LLK/scratch-vm/blob/develop/docs/extensions.md)--it is extremely helpful. 

### How to set up the GAN Paint API server
Open another terminal window, and cd into the k12bridge-gandissect directory. In order to run the GAN Paint API server, you need your Python version in this directory to be Python 3. If it's not, you may want to set up a virtual environment for this directory. Verify that your Python version in this directory is Python 3 by running "python -V". Finally, run "python -m netdissect.server --address 0.0.0.0".

### How to set up the Python server for GAN Paint API interaction
Open another terminal window, cd into the k12scratch-blocks directory, and then cd into the "core" folder. Verify that your Python version in this directory is Python 2 by running "python -V". Finally, run "python GANpaint_request.py".

### How to access the GUI
Once everything finishes compiling, go to "http://localhost:8601/" in your web browser (or instead of "localhost", the IP address of k12scratch-gui), and you should see the modified version of Scratch. From there, to load the GAN Paint extension, click on the "Add Extension" button in the bottom left corner of the Scratch GUI, scroll down to the extension labeled "GAN Paint", and click on it. 


## How to interact with GAN Paint/Gandissect API
I created the GAN Paint API used in this project by training a GAN on pictures of castles from the [Places dataset](http://places2.csail.mit.edu/), and then running [Gandissect](https://github.com/CSAILVision/gandissect) on it. If you'd like to learn more about how to train a GAN, you can look at the Github repo I used: [https://github.com/nashory/pggan-pytorch](https://github.com/nashory/pggan-pytorch). If you'd like to learn more about how to dissect a GAN using Gandissect, go here: [https://github.com/CSAILVision/gandissect](https://github.com/CSAILVision/gandissect).

Once you've run Gandissect on your GAN, you can run `python -m netdissect.server --address 0.0.0.0` to start the API. You can then go to [http://localhost:5001/api/ui](http://localhost:5001/api/ui) to see the API UI. If you click on "all", you'll see a list of possible GET and POST requests you can make to the API. Since the UI has very little documentation, I'll give a brief walkthrough of the requests I used, and what you need to use them. 

### GET /all_projects
*Takes in:* no parameters.

*Returns:* a list of jsons, one for each project you've dissected with Gandisect. In each json, there's two attributes: "project", which is the name of this JSON's project; and "info", which is a json containing the attribute "layers" which corresponds to a list of the layers available in this dissection.

I used this command to access the names of the layers I could use.

### GET /rankings
*Takes in:* `project`, which is the name of the project you want to use; and `layer`, the name of the layer you want to use.

*Returns:* a json containing different metrics from the dissection. Each metric has a list of scores, and the index of each score is the unit its associated with. You can therefore use this use to figure out the most highly-ranked units for each feature.

You can see what a response looks like if you try using the UI with one of the projects & layers you saw in the previous request.
I used the "iou" metric for the feature I was interested in, and then found the 10 lowest-ranking units, since all of the scores were negated.

### GET /levels
*Takes in:* `project`, which is the name of the project you want to use; `layer`, the name of the layer you want to use; and `quantiles`, the quantile for the ablation values it will return.

*Returns:* a json containing ablation values for each unit, given the quantile you inputted. Again, the index of each value corresponds to the unit associated with it.

I used this to find the ablation values for the 10 highest-ranked units that I found using the previous request.

### POST /generate
*Takes in:* a json of the following format:
```
{
            
      # image ids we're interested in
      "ids": [
        <image_id>
      ],
              
      "interventions": [
        {
          # abalations describe which units and values to use to 
          # change that area
          "ablations": <abalations>,
          
          # maskalpha describes the path that the user selected
          # to change
          "mask": {
            "bitbounds": [],
            "bitstring": <base64png_string>,
            "shape": []
          },
        }
      ],
      "project": <project>,
      "return_urls": 0,
    }
```
Where `<image_id>` is the image we're modifying; `<base64png_string>` is a base64png string that represents a 256x256 image with white in non-selected areas and #0BC6D4 in selected areas (essentially, the area of selected pixels we want to apply our feature of choice to); `<project>` is the name of the project you've dissected and want to use; 
and `<abalations>` refers to a list of dictionaries in the following format:
```
{
	"alpha": 1, 
	"layer": <layer>, 
	"unit": <unit>, 
	"value": <level>
}
```
Where `<layer>` is the layer we've been using in all of our requests; `<unit>` the index of one of the 10 highest-ranked units that we found earlier;
and `<level>` is the ablation value for that unit that we found using the `/levels` requests.

*Returns:* a json containing within it a base64png string that represents the image result we want.

This is the request that will actually generate the modified image that we want from this API.


You can also look at my code [here](https://github.com/mit-quest/k12scratch-blocks/blob/master/core/GANpaint_request.py) to see how I used the above requests to get the result I needed for the GAN Paint extension.


## The current state of the GAN Paint Extension

To see the state of the Scratch/GUI part of the extension, please read [Phillip's documentation](https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md)

The goal of my most recent semester working on this project was to modify the existing Scratch GAN Paint Block code so that it actually communicates with the GAN Paint API and updates the image in the block as one would expect it to.

I wrote Python code that automates the GAN Paint API communication that I explained above. I also wrote code to turn that Python code into a simple server, and then added a script to the Block's Javascript code that would send a GET request to my Python server. This way, all of the details of the GAN Paint API communication could be abstracted from the perspective of the Block code. This also means that in the future, if someone wants to add more functionality to the block, they can do so by mimicking the existing structure and therefore will not have to add an extreme amount of Javascript.

I also added code (both on the Javascript and Python side) to convert the information we know into the appropriate format for the GAN Paint API. This includes turning an array of selected-pixel-locations into a base64png string that represents a 256x256 image with white in non-selected areas and #0BC6D4 in selected areas. I also had to convert from filenames to each image's associated image ID in the API. 

All of these additions can be found in either [https://github.com/mit-quest/k12scratch-blocks/blob/master/core/field_ganpaint.js](https://github.com/mit-quest/k12scratch-blocks/blob/master/core/field_ganpaint.js) or [https://github.com/mit-quest/k12scratch-blocks/blob/master/core/GANpaint_request.py](https://github.com/mit-quest/k12scratch-blocks/blob/master/core/GANpaint_request.py)


### What the Block/Extension currently does: 

* The block displays a dropdown GUI for the GAN Paint editor. This dropdown field is of the type "ganpaint", a new field type that Phillip created for this project. 

* The dropdown GUI has: 11 buttons with text on its left hand side (We refer to these buttons as "text buttons"), the main image that is being edited in the middle, and 16 buttons on the right hand side for selecting which starting church image to use (We refer to these buttons as "church selection buttons"). 

* The first 7 of the text buttons (labeled "tree" through "dome") are used to select which brush you are using. They act as a set of radio buttons (so that only 1 of the 7 can be selected at any given time). A string called "brushState" records which of the 7 brush buttons is currently selected (see scratch-blocks/core/field_ganpaint.js). 

* The 8th and 9th text buttons (labeled "draw" and "remove", respectively) are for the user to select whether they are adding or removing a given feature from the main image. These two buttons act like radio buttons (so that only one of two can be selected at any given time). A string called "drawingState" records which of these two buttons is currently selected (see scratch-blocks/core/field_ganpaint.js). 

* When clicked, the 11th text button (labeled "reset") resets the main image to display the original version of the image currently being edited. 

* If the user drags the mouse across the main image, the coordinates of all points within the image that the mouse dragged over will be stored in an array called "selectedPoints". selectedPoints is reset every time that the main image is clicked on or dragged.

* The block code then turns selectedPixels into a 256x256-character string, where each character is 1 if the corresponding pixel in the 256x256 image has been selected, and 0 if not. That string is then sent in a GET request along with brushState and the file path of the main image to the address of the Python server.

* The Python server then turns the 256x256-character string into a base64png string that represents a 256x256 image with white in non-selected areas and #0BC6D4 in selected areas. It also converts the given file path to an image ID.

* It then uses that information as well as the brushState to make a series of GET and POST requests to the GAN Paint API, after which it will receive a base64png string. It will then convert that string to a png image and save that image in the "k12scratch-blocks/media/extensions/ganpaint_images/" directory

* The Javascript code then updates the main image to be the location of this new image, and the dropdown GUI updates its display.

* When any given church selection button is clicked, the main image will be set to display the corresponding image. 

### What the Block/Extension still needs to do: 

* The 10th text button (labeled "undo"), can be clicked, but currently doesn't do anything besides print the message "Undo button clicked. " to the console. It needs to be properly implemented as an undo button. This undo button will ideally function by storing previous versions of the main image on the user's web browser, and accessing them as necessary when the undo button is clicked. Of course, the actual implementation of the undo button will be up to you (assuming that you are the one who will finish this project). Insert the code that you write for the undo button within the function "Blockly.FieldGANPaint.prototype.onButtonClick" in the file "scratch-blocks/core/field_ganpaint.js", at the spot that says: 

```
// ***
// Undo button NOT YET IMPLEMENTED. Add the necessary code here. 
// ***
```

* When activated, the GAN Paint block should (it does not currently do this) save its main image as either a sprite costume or as a Stage backdrop. To accomplish this, you will need to add code to the GAN Paint block's opcode (i.e. the function "saveGANPaintImage" within the file "scratch-vm/src/extensions/scratch3_ganpaint/index.js"). Scratch does let its users import external images to be used as a costumes--if you can figure out what JavaScript function it is that does this, you can probably use that function to save the main GAN Paint image as a costume or backdrop. Neither I nor Phillip know what function it is though, so you'll have to find it. Or save the main GAN Paint image in some other way. Also, feel free to disregard Phillip's commented-out code in the saveGANPaintImage function in the file "scratch-vm/src/extensions/scratch3_ganpaint/index.js". 

* The SVG images that Phillip used to create the GAN Paint field don't display properly in Safari. In Safari, you just get this instead: 

![Dropdown in Safari](GAN%20Paint%20Dropdown%20Safari.png)

This needs to be fixed. You will need the GAN Paint extension to function properly in Firefox, Chrome, Safari, and Microsoft Edge (and possibly also in Internet Explorer and Opera--ask whoever is in charge (presumably your supervisor) about what browsers the GAN Paint extension needs to function properly in). Right now, the SVG images in question do display properly in Firefox though, I can guarantee that much. 

We have not tested the GAN Paint extension in Microsoft Edge, Internet Explorer, or Opera, so we don't know how well the extension currently works in those browsers. 

* On the [GAN Paint website](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4), the main image gets visibly shaded as the user drags their mouse over it. Currently, however, the ganpaint field in Scratch does not do so. So, if you have time, this shading feature should be implemented in the ganpaint field. To accomplish this, you will probably need to add code to the functions "Blockly.FieldGANPaint.prototype.onMouseDown" and "Blockly.FieldGANPaint.prototype.onMouseMove" in the file "scratch-blocks/core/field-ganpaint.js". 

* The images that the GAN Paint extension currently uses in the extension library (i.e. the pictures that you see when you click the "Add Extension" button in the Scratch GUI) are the same as the pictures used in [this tutorial](https://medium.com/@hiroyuki.osaki/how-to-develop-your-own-block-for-scratch-3-0-1b5892026421). Eventually, these pictures will need to be replaced with other, more suitable images. Replacing these images will involve replacing the two images stored in the folder "scratch-gui/src/lib/libraries/extensions/ganpaint", and modifying the file "scratch-gui/src/lib/libraries/extensions/index.jsx" accordingly. 

* Currently, even though the reset button erases the changes visually, and clicking any given church selection button will change the main image accordingly, it doesn't change the image and the pixels that are being sent to GAN Paint so even if you try to select pixels again, it just sends your initial selection. Once in a while, selecting another image and then drawing on it works, but I'm not sure why it works sometimes but not consistently.

* You also can't compound pixel selections currently, as in I can't pick one feature and draw with it, and then also pick another feature and draw with it and see both results.

* Additionally, the draw/remove toggle doesn't do anything; either way, you'll just be drawing onto the image.

* OPTIONAL: Depending on the circumstances for which it is to be used, the GAN Paint extension might eventually need to be viewable in other languages. This, however, is completely optional at this point, as it is not at all an immediate concern. 

Good luck! And feel free to contact me if you have any questions (especially if I explained something poorly and/or forgot to mention something important). 


## Other notes

### A list of potentially useful hyperlinks: 

* GAN Paint: [http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4)
* Phillip's documentaion on his part in this project: [https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md](https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md)

My modified versions of the scratch-gui, scratch-vm, scratch-blocks, scratch-render, and gandissect repositories: 

* scratch-gui: [https://github.com/mit-quest/k12scratch-blocks](https://github.com/mit-quest/k12scratch-blocks)
* scratch-vm: [https://github.com/mit-quest/k12scratch-gui](https://github.com/mit-quest/k12scratch-gui)
* scratch-blocks: [https://github.com/mit-quest/k12scratch-vm](https://github.com/mit-quest/k12scratch-vm)
* scratch-render: [https://github.com/mit-quest/k12scratch-render](https://github.com/mit-quest/k12scratch-render)
* gandissect: [https://github.com/mit-quest/bridge_gandissect](https://github.com/mit-quest/bridge_gandissect)


### Other helpful resources and tools: 

* Me: Feel free to contact me if you need help or clarity regarding something that I have done. See the "Introduction" section for my contact information. 
* Phillip! He did pretty much all of the Scratch/GUI work for this project, and you can see his contact info in [the "Introduction" section of his repository](https://github.com/PTegmark/Scratch-GANPaint/blob/master/README.md#introduction).
